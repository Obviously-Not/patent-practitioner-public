# patent-continuation

> **This is a distribution repository.** It carries the documentation and the
> released binaries. The source lives in a private repository and is not
> published here, so there is nothing to build or send a pull request against:
> every file here is generated from the source repo on each release and is
> overwritten by the next one. Bug reports and feature requests are welcome in
> this repo's issue tracker.

> Continuation-claim **drafting** pipeline for licensed patent practitioners,
> shipped as a **Go binary**. A specification plus a set of filed (parent) claims
> goes in; drafted continuation claims come out, refined by a cross-family critic.
> All inference runs locally by default.
>
> Intended audience: **licensed patent attorneys and agents** who run it as a
> drafting aid under their own professional judgment.
>
> Website, with the measured model comparison and the continuation-practice
> reference corpus: [patentcontinuation.com](https://patentcontinuation.com).

## What it is

A Go binary. A specification plus the parent claims go in; drafted continuation
claims come out. All inference is local by default (Ollama), or through any
OpenAI-compatible endpoint with your own key. Your specification is sent only to the
model provider you configure, and there is no server of ours in that path.

One exception, stated because it is the only one: if you activate a paid licence, the binary
renews that licence over the network. It sends the licence key and nothing else, roughly once
a month, on launch, when the licence is close to expiring. It carries no specification text, no
matter identifiers and no usage data, and `CD_NO_LICENSE_RENEWAL=1` switches it off entirely.
Without a paid licence the binary makes no such call at all.

| Piece | Role |
|---|---|
| `go/main.go` | CLI: `draft`, `export`, `critique`, `bestpractice`, `judge`, `panel`, `compare`, `bench`, `serve`, `models`, `version`, `license`, `activate`, `upgrade` |
| `go/internal/server/` | HTTP server for the local web UI (runs on localhost:9473) |
| `go/web/` | Alpine.js SPA served by the binary (no build step). HTML/CSS are embedded via go:embed; Alpine itself loads from a pinned CDN with a subresource-integrity hash, so the web UI needs network access on first load and is not air-gapped |
| `go/internal/pipeline/draft.go` | Spec analysis, target extraction, drafting, and the verify->revise loop |
| `go/internal/pipeline/critic.go` | Cross-family critic that flags per-claim defects for the revise loop |
| `go/internal/pipeline/judge.go` | Claim-quality rubric judge with spec-anchor validation |
| `go/internal/pipeline/panel.go` | Multi-model scoring panel behind the `panel` subcommand (R&D; cannot rank near-equals) |
| `go/internal/license/` | Offline Ed25519 license verification (multiple keys, multiple capability tiers) |
| `go/internal/llm/` | Provider layer: local Ollama plus an OpenAI-compatible remote |

## Setup

You need the binary (download it from the latest release, or see below to build
it), and **at least one inference backend**:
local **Ollama** (private: your specification never leaves your machine) or a key for one of
three cloud providers, **OpenRouter**, **OpenAI** or **Anthropic** (faster and stronger, but
the spec is sent to a third party). You can set up several and choose per run with
`--provider` and `--model`.

Each provider's key is read from the environment first, and otherwise from
`~/.continuation-drafter/config.json`, a file the binary creates mode 0600 (owner read/write
only). Keys are never read from a command-line flag, never written to a log, and never stored
in this repository. The web UI can save a key into that file for you; the CLI does not, so if
you use the CLI only, export the key in your shell. The binary does not read a `.env` file.

### Option A: local models with Ollama (private)

1. **Install Ollama** from https://ollama.com/download (macOS, Linux, Windows).
   On macOS you can also `brew install ollama`.
2. **Start the server.** The desktop app starts it for you; otherwise run
   `ollama serve`. It listens on `http://localhost:11434`. Point the tool at a
   different host (for example a beefier machine on your LAN) with
   `export OLLAMA_BASE_URL=http://that-host:11434`.

   **If you do that, your specification goes to that host.** It is still your
   hardware and no third party is involved, but the text crosses a network and the
   "nothing leaves this machine" guarantee is about *this* machine. The tool says so:
   the command line prints a note naming the variable and the host before anything is
   sent, and the web interface shows a banner beside the model picker. Leave the
   variable unset for unpublished or privileged material.
3. **Pull a model before you run.** The binary does **not** download models for
   you, and it assumes **no default model**: name one with `--model` or a
   `*_MODEL` env var, or the run fails fast with guidance. Pull one first:

   ```bash
   ollama pull qwen3:32b         # capable general model; 41k window
   ollama pull qwen3-coder:30b   # 262k window: fits a real (124k-char) spec
   ```

   Context is the first filter: a spec is ~1 token per 4 characters and the whole
   thing goes into the prompt, so the model's context window must exceed the spec.
   The tool sizes the request's context window to the prompt automatically, so any
   model gets its full window with no per-model tuning; you only need to pick a
   model whose window fits your spec. **Read [`docs/models.md`](docs/models.md)
   before choosing a model for real work** (which models fit real specs, which
   draft well, which fail outright). There is no single best local model, it is a
   tradeoff between window size and drafting quality.
4. **Verify:** `ollama list` shows the model you pulled; `./continuation-drafter
   models` shows the configuration the tool resolved.

### Option A2: a different local server (Ollama is not the only one)

If you already run **mlx-openai-server, vllm-mlx, llama.cpp's `llama-server`, LM Studio,
LocalAI or SGLang**, point the local path at it directly:

```bash
export CD_LOCAL_OPENAI_BASE_URL=http://127.0.0.1:1234/v1   # your server's address
./continuation-drafter draft --provider local-openai --model <name> \
  --spec spec.txt --parent parent.txt
```

No API key: a server you run does not issue you one. On Apple silicon several of these run
requests at the same time where Ollama's MLX engine takes them one at a time, which matters
for the panel and for `compare`. Run `bench` to see which yours does.

**Whether this counts as local is decided by where the request goes, not by the name.** If the
address you configure is not on this machine, the tool tells you before your specification is
sent and treats the run as remote. A machine on your own network is still across a network,
and the same is true of `OLLAMA_BASE_URL` pointed at another host.

### Option B: cloud models (faster, stronger)

Three providers, each with its own key. A single draft makes several model calls, so check
your balance before a big run.

| Provider | `--provider` | Key | Endpoint override |
|---|---|---|---|
| OpenRouter | `openrouter` | `OPENROUTER_API_KEY` | `OPENROUTER_BASE_URL` |
| OpenAI | `openai` | `OPENAI_API_KEY` | `CD_OPENAI_BASE_URL` |
| Anthropic | `anthropic` | `ANTHROPIC_API_KEY` | `CD_ANTHROPIC_BASE_URL` |

1. **Create a key** with your chosen provider and export it.
2. **Name the provider and the model:**
   `--provider anthropic --model claude-opus-4-8`, or
   `--provider openrouter --model anthropic/claude-opus-4-8`.

**With no `--provider`, the model string still chooses**, exactly as it did before: a
colon tag such as `qwen3:32b` runs on this machine, and a slash slug or a bare vendor prefix
goes to OpenRouter. Every command documented for earlier versions keeps working unchanged.

Whichever you choose, the tool prints a one-line note naming the provider and the endpoint
before your specification leaves the machine.

**OpenRouter is a router, not the company that runs the model.** It sends each call to one of
the companies hosting the model you chose, and may choose a different one each time: a Claude
model there is hosted by Anthropic, Google, Amazon and Microsoft. Two settings limit that, and
the note says which applies:

- `CD_OPENROUTER_PROVIDERS`: the hosts you allow, comma-separated, as OpenRouter names them
  (for example `anthropic`, or `anthropic,google-vertex`). Unset allows any host.
- `CD_OPENROUTER_ZDR=1`: only hosts that keep nothing (zero data retention).

If no host meets the limits, OpenRouter refuses the call before sending it anywhere and the
error lists the hosts that do serve that model. Both settings are also in the web interface's
Settings page.

In the web interface each provider is a tab in the model picker, showing its models when its
key is set and a key box when it is not. The lists are curated: text models only, no
purpose-built or billing variants, nothing much older than the previous generation, and on
OpenRouter only the frontier labs. That last one is an editorial judgement rather than a
popularity ranking, because no usage data is published. With an empty search box you see the
fifty most recent and a count of how many more there are; searching reaches the whole curated
list, and every pane has a box to name a model directly if the catalog does not list it.

`CD_OPENAI_BASE_URL` is namespaced and the OpenRouter one is not, which looks inconsistent and
is deliberate: `OPENAI_BASE_URL` already redirects the **local Ollama** endpoint in this
binary, and giving one variable two meanings, one of which quietly moves a privileged
specification, is not a trade worth making.

**Confidentiality:** a remote model sends the specification to a third party.
That is fine for published patents and **not** for anything unpublished or
privileged, use the local path (Option A) for those. The "nothing leaves your
machine" guarantee belongs to the local path only.

## Run it

### On a Mac, without a terminal

Download **`continuation-drafter-macos.dmg`** from the
[latest release](https://github.com/Obviously-Not/patent-practitioner-public/releases/latest),
open it, drag **Continuation Drafter** to Applications, and double-click it. The drafting
interface opens in your browser. macOS 11 or later; one download runs on Apple silicon and
Intel.

macOS asks once whether you are sure you want to open something downloaded from the
internet. The button is **Open**: the app is signed with a Developer ID certificate and
notarized by Apple, with the ticket stapled, so no network check is needed at first launch.

It has no Dock icon, because it runs no window of its own; the interface is the browser tab
it opens. Press **Stop** in that tab to shut it down.

### From the command line

Get the binary once, then run it from wherever your files are (paths resolve
from your current directory).

```bash
# Download a prebuilt binary (pick your OS and architecture: darwin-arm64,
# darwin-amd64, linux-amd64, linux-arm64, or windows-amd64.exe).
gh release download --repo Obviously-Not/patent-practitioner-public \
  --pattern 'continuation-drafter-darwin-arm64'
chmod +x continuation-drafter-darwin-arm64
mv continuation-drafter-darwin-arm64 continuation-drafter
```

The macOS binaries are signed with a Developer ID certificate and notarized by Apple as
of v0.1.4, so there is nothing else to do: no Gatekeeper dialog, and no quarantine
attribute to clear. Releases up to and including v0.1.3 were unsigned and needed
`xattr -d com.apple.quarantine`; that step is no longer required and should not be
copied from an older page.

The Windows binary is signed as of v0.2.3, by Geeks in the Woods, which is the name
Windows shows as the verified publisher. Releases before that were unsigned and Windows
Defender blocked them. Expect a warning on a new release anyway: Windows says a file
isn't commonly downloaded until enough people have taken it, and preselects Delete. The
Publisher line in that dialog is the check, and it is populated only when the signature
validates. SmartScreen scores a new certificate on
download reputation rather than on the signature alone, so a prompt is still possible; the
publisher line on it is what distinguishes this build, and the SHA-256 below is what
proves it.

A **downloaded** bare binary cannot be opened by double-clicking it, whatever its
signature: an HTTP download carries no execute bit and a bare file has no bundle, so the
Finder hands it to a text editor. That is what the `.dmg` above is for, and it is the
right download for anyone who was not going to use a terminal anyway.

What establishes that the file is the one published is its SHA-256, listed in
`checksums.txt` with each release.

**First, see it work with no files of your own.** The walkthrough specification the
published lessons use is built into the binary, so this runs from any directory and
never counts against any allowance:

```bash
./continuation-drafter models      # what this machine can run, and which model fits
DRAFT_MODEL=<model> ./continuation-drafter draft --demo
```

**Draft** (the main command). A spec plus the parent claims in, drafted
continuation claims out on stdout; live progress prints to stderr:

```bash
# Local model (your specification never leaves your machine)
DRAFT_MODEL=qwen3:32b ./continuation-drafter draft \
  --spec ../fixtures/micro/micro_spec.txt \
  --parent ../fixtures/micro/micro_parent_claims.txt

# Cloud model (spec leaves your machine; published material only)
ANTHROPIC_API_KEY=... ./continuation-drafter draft --provider anthropic \
  --spec spec.txt --parent parent.txt \
  --model claude-opus-4-8 --revise-loops 2
```

**Critique** an existing draft on its own (per-claim 112(a)/112(b) defects with
fixes), independent of the drafting loop:

```bash
./continuation-drafter critique --spec spec.txt --parent parent.txt \
  --draft claims.txt --model anthropic/claude-sonnet-5 --json
```

**Export to Word.** Claims are text, and the work continues in a word processor.
`export` writes a `.docx` you can open in Word, Pages or LibreOffice, either from a
claims file or straight off a `draft --json` run:

```bash
./continuation-drafter export --draft claims.txt --out claims.docx

DRAFT_MODEL=qwen3:32b ./continuation-drafter draft \
  --spec spec.txt --parent parent.txt --json \
  | ./continuation-drafter export --from-json - --out claims.docx
```

LaTeX is also supported, for drafters who keep matter documents that way. The
format follows the `--out` extension, so `--out claims.tex` writes LaTeX; state
`--format` explicitly to override it.

Claim numbering is preserved exactly as drafted, never renumbered, because a
continuation's claims are commonly numbered on from the parent. The document is
written user-only (mode 0600) since it holds client claim language, it refuses to
overwrite an existing file unless you pass `--force`, and it carries the maturity
notice inside the document, because an exported file gets read by people who never
saw the terminal that made it.

**Compare two drafts**, and find out how confident the answer is:

```bash
./continuation-drafter compare --spec spec.txt --parent parent.txt \
  --a draft-a.txt --b draft-b.txt --rounds 8
```

It asks the judge about the same pair twice, once each way round, because a judge can answer
differently depending on which draft it sees first. That is a property of the model, not of
your claims. It reports one of three things: that one draft is better, that it could not find
a difference, or that it could not tell, with the reason. **"Could not tell" is a real answer
and will be a common one.** It also reports how consistent the judge was with itself, which is
the number that says whether to trust the rest, and a small local model can score very badly
on it.

**Check whether your model server runs requests at the same time:**

```bash
./continuation-drafter bench
```

A multi-model panel is several requests. On a server that runs them together it finishes in a
fraction of the time; on one that queues them it does not. Which yours does depends on the
server and the model format rather than on this tool, so `bench` measures it and tells you
whether setting `CD_CONCURRENCY` is worth it.

**Other commands:** `judge` scores one draft against the rubric, returning
per-claim verdicts with spec-anchor validation (T1) and honest-null: a claim it
cannot anchor to the spec comes back `indeterminate`, not a low score (R&D; the
score is not a ranking, the support audit is the useful part). `panel` scores
candidate drafts with a multi-model judge (R&D: it separates gross differences
but cannot rank near-equals, so read it as a spread, not a ranking);
`bestpractice` reviews a draft for single-actor enforceability; `models`
prints the resolved model configuration; `version` prints build info. Run
`<command> --help` for every flag.

## Web UI

The binary includes a local web UI. Start it with `serve`:

```bash
./continuation-drafter serve           # opens http://localhost:9473 in your browser
./continuation-drafter serve --port 8080 --no-browser
```

The interface has two tabs.

**Draft** takes a whole patent application, specification and existing claims in one
document, and runs the drafting pipeline over it with progress streaming as it goes.

**Discuss** is a conversation over the same document. You attach the application once, then
ask in your own words, and the tool selects one of its passes and runs it:

| You ask about | It runs |
|---|---|
| what the specification discloses that the claims do not reach | Find what the parent left unclaimed |
| how to broaden what you already claim | Find broader scope in what you already claim |
| another application around the same claim center, framed differently | Keep this claim center and recast it |
| drafting claims to the directions you have chosen | Draft continuation claims |
| whether a claim set is correct | Find defects in a claim set, or Review drafting craft |
| whether a limitation is narrowing you for nothing | Find scope you are giving away |
| rewriting a set | Fix flagged defects, or Polish a clean set |

The broadening pass works through six moves against your existing claims: who performs the
reciprocal or downstream operation, which limitations are one implementation rather than the
invention, whether a per-element step can be an aggregate, whether the artifact the process
produces can be claimed instead, which phase of the system's life is unclaimed, and which other
disclosed product could perform the same role.

The continuation directions it finds are listed beside the conversation, each labelled with
whether its supporting quote was located verbatim in your specification, and each removable if
you disagree with it. Directions you keep are what the drafting pass writes claims to, so the
set of claims you get is the set you chose.

Both tabs share the rest: drag-drop or paste, model selection from what this machine can
actually reach, light and dark themes following your system preference, and licence-key
activation.

**What the tool never does** is tell you whether anything is patentable, novel, non-obvious or
eligible. It drafts claim language and reports on drafting quality against the specification
you supply. Every output is for a licensed practitioner to check and decide on.

The server binds to `127.0.0.1` only (localhost, not network-accessible). Your inputs
(spec, parent claims, drafts) stay on your machine; the only place they are sent is the
LLM provider you configure. One honest caveat: the web UI loads the Alpine.js library
from a pinned CDN (with an integrity hash) when the page opens, so the browser makes one
request to that CDN on load. That request carries no spec or claim data, but it does mean
the **web UI needs network access on first load and is not fully air-gapped**. The CLI has
no such dependency. With a local model and no paid licence it runs fully offline; with a paid
licence it additionally makes the monthly licence-renewal call described under Setup, which
`CD_NO_LICENSE_RENEWAL=1` disables. An air-gapped firm should run with that variable set and a
long-dated token issued out of band.

## Updating

Nothing checks for updates on its own unless you ask it to. When you want to know, press
**Check for updates** in Settings, or run:

```
continuation-drafter update --check
continuation-drafter update           # asks before installing
continuation-drafter update --yes     # installs without asking
```

The check is one request to github.com that carries nothing about you or your matters; GitHub
sees an ordinary web request. Offline mode (`CD_NO_LICENSE_RENEWAL=1`) refuses it.

Settings also has a switch, off until you turn it on, that makes the same check each time the
app opens and shows a link in the header when a newer release exists. It never installs anything
by itself.

Before anything is replaced, the release's `checksums.txt` must carry a valid signature from
the key built into your copy, the download must match its checksum, a Mac download must carry
Apple's signature for this publisher and pass Gatekeeper, and the new program must report the
version it was fetched as. If any check fails, nothing changes and the error says so; download
the new version from the releases page instead. The Mac app is replaced whole, and from the web
interface it restarts itself and the page reloads when the new version is running.

## For IT departments

A firm's IT department can enforce four settings rather than trust them: models on this machine
only, no update check, offline mode, and the firm's own model server. They are set through Group
Policy or Intune on Windows and a configuration profile on a Mac, and a practitioner cannot change
them. The keys and where to set them are on [patentcontinuation.com/security](https://patentcontinuation.com/security).

## License keys

Keys are Ed25519-signed, offline-verified, and stored locally in
`~/.continuation-drafter/config.json`. Verification stays offline: the signature is checked
against a public key compiled into the binary, with no network involved. What is networked is
ACQUISITION, not verification, and only for a subscription: the binary fetches a replacement
token when the current one nears expiry, so you run `activate` once rather than every month. Multiple keys can be activated; each key's tier is
verified and reported. **As of 2026-07-21 the tier is reported but does not yet gate any
feature: the free/paid paywall is deferred, so all features are available today.**

```bash
./continuation-drafter license             # show license status and capabilities
./continuation-drafter activate --key <your-key>   # activate a key
./continuation-drafter upgrade             # report what this build can be upgraded to
```

**Note:** release builds embed the licence verification public key (since v0.1.1);
dev builds do not, and skip verification. **In neither case is any feature gated:**
nothing is currently sold, `upgrade` says so rather than opening a payment page, and a
key verifies and reports a tier without changing what the tool will do.

### The knobs that matter

- `--model` selects the model and the routing (no default is assumed; name one).
  A **colon** tag (`qwen3:32b`) is a local Ollama model; a **slash** slug
  (`openai/gpt-5.5`,
  `anthropic/claude-opus-4-8`) or a recognized remote prefix (`claude-`, `gemini-`,
  `grok-`, `gpt-`, `chatgpt`, or an o-series name) routes to OpenRouter (a bare
  `gpt-5.5` is rewritten to `openai/gpt-5.5`). Ambiguous bare names that exist in
  both worlds (`llama`, `qwen`, `deepseek`, `mistral`) default to LOCAL; add the
  `vendor/name` slash form to force one remote. If a bare name is not a pulled
  local model, the error hints at the remote form. See
  [`docs/models.md`](docs/models.md).
- `--revise-loops N` (default 3) is how many critic/revise passes are ALLOWED; each pass
  costs two more model calls. `0` gives the raw first-pass draft. The loop stops early when
  the critic finds nothing, or when a pass stops reducing what it finds, so fewer passes than
  you allowed is the normal outcome. With `--json`, `revise_loops` reports how many revisions
  were actually adopted, alongside `revise_loops_run`, `revise_loops_discarded` and
  `revise_loops_configured`; a pass whose answer could not be parsed keeps the previous claims
- **The critic runs on its own model.** By default it is the draft model; set
  `CRITIC_MODEL` (or `critique --model`) to make the revise loop cross-family.
  Gotcha worth stating: if you draft on a large-window remote model but leave the
  critic on a small local default, the critic can choke on a big spec. Point the
  critic at a model that also fits the spec.
- `--max-tokens` / `MAX_COMPLETION_TOKENS` (default 16000) is the per-call
  completion cap. Two deadlines bound a run and they are related rather than set
  separately: `CD_REQUEST_TIMEOUT_SECONDS` (default 1200, twenty minutes) is each
  model call's, and the whole run's is derived from it so that one call and one
  retry always fit, which is 45 minutes at the default. `--timeout` and
  `CD_RUN_TIMEOUT_SECONDS` override the second. The error says which one stopped
  a run: "after 2 attempts, each timing out" is a call, and a bare "context
  deadline exceeded" is the run.
- Keys come from the environment first, then from `~/.continuation-drafter/config.json`
  (mode 0600), never from a flag and never from a log. Each cloud provider has its
  own key, `OPENROUTER_API_KEY`, `OPENAI_API_KEY` and `ANTHROPIC_API_KEY`, and you
  need only the ones you actually use. The environment always wins, so exporting a
  variable overrides whatever is saved in the file.
- `--json` emits a structured envelope on stdout instead of plain text. For `draft` it now
  also carries the **targets** the drafting was aimed at, each with the result of looking for
  its supporting quote: located, absent, quoted from the parent claims rather than the
  specification, and so on, with a sentence saying what that points at.
- **Every run reports how much of your specification it read**, as a percentage and the
  largest stretch nothing cited. The tool could always tell you a proposed direction was
  fabricated; this is what tells you it never looked at part of the document.
- `CD_CONCURRENCY` runs independent model calls together, and **defaults to 1**. Run
  `bench` first: on a server that queues requests, raising it buys no speed and raises peak
  memory on a machine that may already be near its limit.
- `CD_WINDOW_LARGE_SPECS=1` reads a specification too large for your model in overlapping
  windows instead of refusing. **Off by default**, because a model that can hold the whole
  document should get the whole document, and windowing costs attention that overlapping
  does not give back. It is for when the alternative is not running at all. A windowed run
  **says so while it runs** and reports how many windows produced nothing, because a degraded
  run should not be mistaken for an ordinary one and a window that quietly produces nothing
  leaves a result that looks clean and is smaller than it should be. If every window fails,
  the run fails: a model server that was unreachable throughout must not read as a
  specification with nothing left to claim.
- `CD_NO_BROWSER=1` stops the tool opening a browser window, which matters if you script it.
- `CD_NO_LICENSE_RENEWAL=1` disables the one outbound call this binary makes on its own,
  described under Setup, and refuses update checks (see Updating).

**Model choice matters more than it looks, and context is the first filter.**
A specification is ~1 token per 4 characters and the whole thing goes into the
prompt, so a model whose window is smaller than the spec **silently truncates
its input** and drafts from a spec it never fully saw. A 124k-character spec is
~31k tokens, which rules out every 32k-40k model no matter how well it drafts.

Measured results for the models available locally, including which ones fail
outright, are in [`docs/models.md`](docs/models.md). Thinking models (qwen3,
glm, deepseek-r1) are handled: the binary disables thinking so the completion
budget buys claim text rather than discarded reasoning.

Remote models via OpenRouter work too (`--model anthropic/claude-opus-4-8` with
`OPENROUTER_API_KEY` set), but the spec then leaves your machine. Use a local
model for anything unpublished or privileged. Note the local path costs more than
speed: on real specs, local models draft well-grounded but strategically flatter
claims than frontier models, so treat a local draft's claim aim as a first pass.

## Honest maturity

This is **R&D-grade**, not a finished product. The originating experiment's
conclusions still hold: the pipeline drafts real, well-formed, spec-grounded
continuation claims, but a **single AI judge cannot reliably rank drafters**, the
revise loop is a modest positive, and only a multi-model panel gives a stable
read. Treat output as drafting *input* for a practitioner, never as filing-ready
and never as a legal determination.

The licensing system's **verification and reporting are implemented; feature gating is
deferred**. All features are available in every build, including release builds.

**The reason changed with v0.1.1 and the distinction is worth stating plainly**, because
the conclusion is the same and the mechanism is not. Until then, release builds carried no
public key, and a build with no key treats every feature as available. Release builds now
embed the key. What keeps every feature available is no longer the missing key: it is that
**no feature is gated on a capability**, and that the free allowance is undecided rather
than set. A key verifies and reports a tier; it does not change what the tool will do.

## License

Proprietary. All rights reserved. Not open source. The binary is distributed
freely; the source is not published.
