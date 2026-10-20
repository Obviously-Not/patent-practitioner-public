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
| `go/web/` | Alpine.js SPA served by the binary (no build step). HTML, CSS, fonts and Alpine itself are embedded via go:embed and served from the binary, so the web UI makes no request to any other site |
| `go/internal/pipeline/draft.go` | Spec analysis, target extraction, drafting, and the verify->revise loop |
| `go/internal/pipeline/critic.go` | Cross-family critic that flags per-claim defects for the revise loop |
| `go/internal/pipeline/judge.go` | Claim-quality rubric judge with spec-anchor validation |
| `go/internal/pipeline/panel.go` | Multi-model scoring panel behind the `panel` subcommand (R&D; cannot rank near-equals) |
| `go/internal/license/` | Offline Ed25519 license verification (multiple keys, multiple capability tiers) |
| `go/internal/llm/` | Provider layer: local Ollama plus an OpenAI-compatible remote |

## Setup

You need the binary (download it from the latest release, or see below to build
it), and **at least one inference backend**:
local **Ollama** (private: a model Ollama runs on your machine never sends your specification off it) or a key for one of
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
LocalAI or SGLang**, point the local path at it directly. Ollama stays the recommended local
server, because the tool sets the context window on every call there and checks what that
window costs in memory; LM Studio is the alternative, since it says what window it loaded a model
at and refuses a prompt longer than it rather than cutting it.

```bash
export CD_LOCAL_OPENAI_BASE_URL=http://127.0.0.1:1234/v1   # your server's address
./continuation-drafter draft --provider local-openai --model <name> \
  --spec spec.txt --parent parent.txt
```

No API key: a server you run does not issue you one. On Apple silicon several of these run
requests at the same time where Ollama's MLX engine takes them one at a time, which matters
for the panel and for `compare`. Run `bench` to see which yours does.

**The window your server holds the model at is known before your specification is sent.** An
OpenAI-compatible request has no field for the context window: the server chose it when it
loaded the model, and a longer prompt is cut or refused. So the tool reads the window llama.cpp,
vLLM and LM Studio (for a loaded model) declare, or uses the one you state, and refuses a call
that does not fit it. A server that says nothing is measured first, with two short prompts and
one as long as the run needs, which carry no part of your specification and can take a few
minutes on a large model. When that says nothing either, the number is needed from you:

```bash
export CD_LOCAL_OPENAI_CONTEXT_TOKENS=16384   # LM Studio's context length, llama.cpp's -c / -np, vLLM's --max-model-len
```

or Settings in the web interface, for one model. With no window known the run is refused and
says where your server shows it. An Ollama address given here is run on Ollama's own API, which
sets the window on every call, so nothing has to be stated.

A server that declares no window, such as `mlx_lm.server`, reads a prompt until memory runs out,
and only on Ollama does the tool price a window in memory before asking for it. On such a server,
choose a model whose window your machine can hold; [`docs/models.md`](docs/models.md) has the
measurement.

A whole run (draft, critique and judge) has been measured through llama.cpp, vLLM and LM Studio.
LM Studio does not honour the tool's request that a model not think, so turn thinking off for the
model there. On the MLX servers a small model did not hold the JSON a scoring step needs: a draft
still completes, saying which review did not; choose a model that holds it on Ollama. [`docs/models.md`](docs/models.md) has the details.

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
machine" guarantee belongs to the local path only, and to a model the local server runs
itself: a cloud model that Ollama lists and forwards to another host is refused, and the
model list marks it.

## Run it

### On a Mac, without a terminal

Download **`continuation-drafter-macos.dmg`** from the
[latest release](https://github.com/Obviously-Not/patent-practitioner-public/releases/latest),
open it, drag **Patent Practitioner Tools** to Applications, and double-click it. The drafting
interface opens in your browser. macOS 13 Ventura or later; one download runs on Apple silicon and
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
# Local model (your specification stays on your machine)
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

LaTeX and plain text are also supported, LaTeX for drafters who keep matter documents
that way. The format follows the `--out` extension, so `--out claims.tex` writes LaTeX
and `--out claims.txt` plain text; state `--format` explicitly to override it.

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
whether to say in Settings how many requests your server runs at once. A remote provider runs
three at once already.

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

The interface starts with what you are working on: a reply to an Office action, or claims for
a continuation application. Each matter then runs in steps.

**A reply to an Office action** runs Papers, The action, Address the Rejections and Objections, and
The reply. A line
above the steps says what the matter is: the application, the latest action and its date, the
computed deadline and whether its mailing date is confirmed, and the claims in use. Under Papers you
add each paper as you have it, as a PDF, a Word file or text, one at a time or several at once: the
Office action, the current claims, the specification, the previous response, earlier papers or the
complete Patent Center file history, and any cited reference. Each paper's kind and date are read
from its own words, for you to confirm, and Papers says which claims the reply reads and what is
still missing. At a first action the application alone will do: given as the specification, its claims
under its claims heading are the claims in use, and once the action is read Papers names any claim the
action names that they do not hold, as when a preliminary amendment added claims. A Patent Center download carries the application as filed, and when the matter has no
specification yet that is read as its specification; one you add under Papers is kept. A history
over Patent Center's 100 MB download limit can be downloaded in parts and added one after another. Scanned pages
are read several at a time, as many as half the machine's cores and at most eight. A scanned claim listing is read claim by claim: a page whose claim numbers the scan
set apart from their claims is read again as one column, and what a scan makes of struck-through
words, and each page's header and footer, are left out of the claims. Check the claims in use
against the paper all the same, since a struck word the scan reads as an ordinary word cannot be told
from one the claim keeps. Each PDF you add, the specification and references included, is kept with the
matter on this machine, in the matter's own folder, as is a granted reference's PDF fetched for
its column citations, and each goes with the matter when you delete it; the record links each
paper, and a quotation of the action, the specification, the claims or a reference opens its PDF
at the page the quotation is on, a scanned paper read by OCR included. Every quotation is checked
against its paper and shown as the paper's own text: a quotation shortened with an ellipsis is found
part by part, in order and close together, and shown with each cut marked, and one that runs across a
page's header and footer is found across it. The action shows each item in the examiner's own words, grouped by what the examiner
applied, with the response deadline computed from the mailing date and a table of the claims, what
each depends from and every item stated against it. Beside the deadline it gives where today falls:
the cost of filing today, by month of extension and fee, and after a final action whether today is
within two months of it. When the file history holds a Notice of Abandonment or a letter of express
abandonment, the line above the steps names it, and the action it names shows the notice's own
sentences, any petition dated after it, and the rules on revival, quoted; the reply is still drafted,
as a petition to revive is filed with it. Check the deadline against your docket, which is the system
of record: the deadline is your responsibility. Today is this computer's date.

Address the Rejections and Objections (and Requirements, when the action states one) gives each
item of the action a workspace of its own: its state, the examiner's
citations limitation by limitation, a response approach you choose (arguments, amendments, both, cancelling the
claims, a terminal disclaimer where the item allows one, or your own words), and its drafted units.
Each unit is marked as a record fact (written from the record with no model), suggested text, or
text for review; you add it to your reply, edit it or dismiss it, and an added unit can be taken out
again. A unit saying what a reference does not describe names the claim's words the drafter did not
find, shows the passages it rests on, and is added only once you say what you reviewed. Arguments on
a section 103 rejection address the combination the examiner proposes, and on a section 102
rejection the single-reference standard. Adding a drafted amendment writes its words into the claim
it names in your proposed claims (filled with the claims in use if you have not typed any), and
choosing to cancel claims marks them canceled; either can be taken out again, and the claims keep a
history of each change. A drafted amendment that brings in a dependent claim's words names that
claim; one proposing words a claim already recites says so, and adding one to a claim your proposed
claims do not hold tells you its words were not written. A restriction requirement is always listed
as an item to answer. The reply holds your reply to that action and the checks over
it, led by their counts, and reads as a document with its section headings and what changed since
your last export. Reply in Word is laid out as a filed response: a running header, a title naming
the rule the reply is filed under (37 CFR 1.111, 1.116, or 1.114 with a request for continued
examination, as you choose after a final action), a caption filled from the file history with a bracketed place
for anything it lacks (the title is read from the papers that state it, or typed in *Matter facts*),
the amendments to the specification, title and abstract you adopted, marked up against the text they
replace, with the abstract on its own sheet, the claim listing marked up from your proposed claims with only what changed marked (a period that
becomes a semicolon is shown in double brackets, and the word before it stays plain), the remarks, a
conclusion and an unsigned signature block. Two sample layouts are built in, and each can be
downloaded as Word to adapt; a reply of yours uploaded as a template gives its own layout instead.
Notes on the reply, on a section or the whole of it, are resolved rather than
deleted, counted beside the export and in Next steps, and never written into the exported reply.

**Continuation claims** run Papers, Embodiments and Claims. With the parent's Patent Center download,
the line above the steps names the parent and its issue date, before or after today, and *Matter
facts* gives its notice of allowance with three months from it, the issue fee paid, and any notice of
abandonment, beside MPEP 211.01(b) on copendency. Embodiments reads the parent's
specification and its claims (or a whole application, read at its claims heading) and lists what
the specification discloses beyond them, each saying how it differs from the parent's claims, or
that it only narrows one, with the specification passage cited for it, for you to verify. With the
parent's file history attached, each is then related to the parent's prosecution record: which
limitations of the claims as allowed it builds on, whether each was in the claims as filed or added in
a round of prosecution, and the examiner's reasons for allowance, every claim, round and quotation
checked against the record. Draft first claims toward one (an
independent claim and two dependents), edit them or revise them with an instruction of your own,
then expand them into a full set of the size and claim types you choose, keeping your reviewed
claims first. Draft straight through finds the embodiments and drafts toward them in one run. Each
embodiment names the paragraph its passage was found at, and those found at one paragraph are listed
together. Critique findings are shown by weight: those the program checked in the text, those for
your review, and notes; a finding the claim text answers is set aside, with one line saying why.
As on a reply, Next steps lists what the matter has left, notes go on a claim or the whole set and
are never exported, the claims' download is recorded so what changed since can be shown, and the
claims keep a history of how they came to be.

**The conversation** sits below every step. You ask in your own words, and the tool selects one
of its passes and runs it:

| You ask about | It runs |
|---|---|
| what the specification discloses that the claims do not reach | Find what the specification discloses beyond the parent's claims |
| how to broaden what you already claim | Find broader scope in what you already claim |
| another application around the same claim center, framed differently | Keep this claim center and recast it |
| drafting claims to the directions you have chosen | Draft continuation claims |
| whether a claim set is correct | Find defects in a claim set, or Review drafting craft |
| whether a limitation is narrowing you for nothing | Find scope you are giving away |
| rewriting a set | Fix flagged defects, or Polish a clean set |

The broadening pass works through six moves against your existing claims: who performs the
reciprocal or downstream operation, which limitations are one implementation rather than the
invention, whether a per-element step can be an aggregate, whether the artifact the process
produces can be claimed instead, which phase of the system's life the claims do not recite, and which other
disclosed product could perform the same role.

The continuation directions it finds are listed beside the conversation, each labelled with
whether its supporting quote was located verbatim in your specification, and each removable if
you disagree with it. Directions you keep are what the drafting pass writes claims to, so the
set of claims you get is the set you chose.

Every step shares the rest: drag-drop or paste, model selection from what this machine can
actually reach, light and dark themes following your system preference, and licence-key
activation.

**What the tool never does** is tell you whether anything is patentable, novel, non-obvious or
eligible. It drafts claim language and reports on drafting quality against the specification
you supply. Every output is for a licensed practitioner to check and decide on.

The server binds to `127.0.0.1` only (localhost, not network-accessible). Your inputs
(spec, parent claims, drafts) stay on your machine; they are sent to the LLM provider you
configure, and a patent search you ask for sends its query, which describes the invention, to
the search service you chose (SerpApi, or your own Google Cloud project). The web UI loads everything it needs, including the Alpine.js
library, from this program, so the browser makes no request to any other site and the web
UI works with no network access. With a local model and no paid licence it runs fully offline,
apart from what you ask it to fetch or install (an update, a cited reference, a search, a model,
or OCR through your own Homebrew on a Mac); with a paid
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

A firm's IT department can enforce five settings rather than trust them: models on this machine
only, no update check, offline mode, the firm's own model server, and no fetching of cited
references or patent searches. They are set through Group
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
- `--revise-loops N` is how many critic/revise passes are ALLOWED; each pass costs two more
  model calls. It defaults to the count saved in the web interface under Settings, Drafting,
  which is 1 until changed, so the command line and the web interface revise alike. `0` gives the raw first-pass draft. The loop stops early when
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
- **How many model calls run at once** (a panel's rounds, a reply's items): one at a time on
  a model server you run, unless Settings says it runs more; three at once at a remote
  provider. A panel on your own server runs model by model, so each model loads once.
  `CD_CONCURRENCY` overrides both. Run `bench` before raising the local number: on a server
  that queues requests, more at once buys no speed and raises peak memory.
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
