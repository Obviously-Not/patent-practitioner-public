# Security policy

## Reporting a vulnerability

Please report security issues privately. Do not open a public GitHub issue.

Use GitHub's **private vulnerability reporting**: navigate to the Security tab of this repository and click **Report a vulnerability**. This routes the report directly to the maintainers without making it public.
Direct link: [Security tab](https://github.com/Obviously-Not/patent-practitioner-public/security).

## Scope

continuation-drafter is a local command-line tool and local web UI. It reads a
specification and a set of parent claims from your filesystem and sends them to
an inference backend you choose: a model on this machine (nothing leaves it) or a
remote endpoint you configure. There is no service operated on your behalf, so the
attack surface is the binary and the files it touches.

**Since 0.7.0 it can also reach three more places, each only when the operator asks or has
turned it on**, and none in offline mode or where an IT department sets the
`DisablePatentLookups` policy: Google Patents, to fetch a reference the examiner cited, by its
number and nothing else; and a patent search, either on the operator's own Google Cloud project
(BigQuery) or through SerpApi with the operator's own key. A search sends the query, which the
tool writes from the operator's request and shows, with where it will go, before it is sent
unless the operator has turned that confirmation off.

**On this machine means on this machine, and the binary decides that from the address
rather than from the name.** A local model can be pointed at another host through
`OLLAMA_BASE_URL` or `CD_LOCAL_OPENAI_BASE_URL`, and a host on your own network is
still across a network. In that case the tool tells you, naming the variable and the
destination, before anything is sent. The disclosure and the request are computed from
one resolver so they cannot disagree; they did disagree until 2026-09-25, which is the
kind of report this section is asking for.

**A model the local server runs elsewhere is refused.** Ollama can list a cloud model beside
the ones it runs and send each request for it on to another host, while the address the tool
dials is still this machine. Before sending to a local model the tool asks Ollama where it runs
that model, in its description of the model and in its list of models (the only place Ollama
names the host for a cloud model), refuses one Ollama forwards (naming the host), and treats a
server it cannot ask as one it will not send to. If an answer says another host generated it anyway, the tool withholds
that answer, says that one request left and where it went, and refuses every later call to that
model. It cannot see forwarding by a server other than Ollama, or by a proxy in front of one,
that says nothing about it.

**What the tool asks a local model server, and where.** Every such question goes to the address
configured for that server and nowhere else, and each answer is read for numbers only. Of
Ollama: its version and what it holds (`/api/version`, `/api/ps`), what it says about a model
(`/api/show`), and its list of models (`/api/tags`), the last to see where each model runs and to
work out what a window costs in memory before asking for it. Of an OpenAI-compatible server: its model list, and the tables in
which llama.cpp (`/props`, `/slots`) and LM Studio (`/api/v1/models`) say what window a model is
loaded at. Of a server that says nothing, the tool measures the window with three prompts of
generated text that carry nothing of the specification.

In scope:

- Anything that sends specification text to a destination the operator did not
  configure, or that widens the local-only guarantee of the on-machine path, including a
  model the local server forwards elsewhere reaching it without the refusal above
- A reference fetch that carries anything but the reference's number, a search sent without
  the confirmation the operator left on, or either one made in offline mode or under the
  `DisablePatentLookups` policy
- **A disclosure that disagrees with where the request goes**, in either direction: a
  run that reaches another host without saying so, and a run that claims your
  specification left the machine when it did not. The second is a real finding, not a
  cosmetic one, because a warning that fires on a local call teaches a reader to scroll
  past the one that matters
- Disclosure of the API key or license keys held in
  `~/.continuation-drafter/config.json`, including through logs, error text,
  process arguments, or a loosening of that file's 0600 permissions
- Vulnerabilities in the local web UI's server: request handling that reaches
  files outside the working set, or that lets another local user or a web page
  in the operator's browser drive the tool
- Flaws in licence renewal: a renewal response that can replace a valid token with one the
  client should have rejected, a payload that carries more than the licence key, or any way to
  make the client fail closed (stop working) when the endpoint is unreachable, which it must
  never do.
- Flaws in offline license verification, such as accepting a token that was not
  signed by the embedded key, or verifying with an attacker-supplied key
- Path traversal or symlink escape when reading spec and claim files, or when serving a kept
  PDF (a paper, the specification or a reference): one matter's request reaching another
  matter's files, a file its own record no longer names, or any file outside the matter's own
  folder
- A margin note, on a reply or on the claims, reaching an exported document
- Dependency vulnerabilities that materially affect any of the above

Out of scope:

- Issues in third-party services the tool can be pointed at, including Ollama
  itself and any remote inference provider
- The quality, correctness, or completeness of drafted claims. Output is
  R&D-grade drafting input for a licensed practitioner, never filing-ready and
  never a legal determination; a weak draft is a product limitation, not a
  vulnerability
- That all capabilities are currently available regardless of license tier.
  This is a documented, deliberate state: verification and reporting are
  implemented and feature gating is deferred, so an unlocked feature is not a
  bypass
- Social engineering, physical access, and theoretical issues without a
  reproducer

## What to expect

We will acknowledge a valid report within 5 business days and provide a
remediation plan within 14 days for in-scope issues. Disclosure timing is
coordinated with the reporter.
