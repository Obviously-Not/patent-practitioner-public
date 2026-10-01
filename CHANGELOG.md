# Changelog

All notable changes to continuation-drafter are recorded here. The format
follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/), and releases
are the `v*` tags in the source repository.

This file is the **only** source of release notes: the release pipeline
deliberately does not generate them from commit history, because the source
repository is private and its commit subjects are not published.

## [Unreleased]

## [0.6.1] - 2026-10-01

### Changed

- **Continuation Drafter is now Patent Practitioner Tools.** It reads Office actions as well as
  drafting continuation claims, and the old name described one of those. The command
  (`continuation-drafter`), the download file names and your saved settings are unchanged, and a
  copy you update from inside the app keeps working as before. On a Mac, an updated copy keeps its
  old name in the Applications folder; if you download the new version instead, you will have
  both, and the old one can be deleted.
- **The places an IT department sets its policies moved with the name**, on Windows and on a Mac.
  Policies set under the 0.6.0 locations no longer apply; the new ones are on
  [patentcontinuation.com/security](https://patentcontinuation.com/security).

## [0.6.0] - 2026-10-01

### Added

- **An optional update check when the app opens.** Settings has a new switch, off unless you turn
  it on. When it is on, opening the app makes the same request as pressing Check for updates and
  shows a link in the header if a newer release exists. It never installs anything by itself, and
  offline mode refuses it.
- **Settings your IT department can enforce.** A firm can now require models on this machine
  only, turn off the update check, force offline mode, or name its own model server, through
  Group Policy or Intune on Windows and a configuration profile on a Mac. A practitioner sees
  which settings their organization has set, and cannot change them.
- **Release notes in the app.** Check for updates now shows what changed in the new release, not
  only its version number.
- **An offer to move the Mac app out of the disk image.** Opened straight from the disk image, the
  app asks whether to copy itself into Applications and open from there, because a copy left
  running from the disk image cannot be updated.

### Changed

- **Advice for a model server that is not running fits your machine.** It used to say to run a
  terminal command everywhere. It now says to install Ollama if it is not installed, to open the
  Ollama app on a Mac or Windows machine where it is, and names the command only where that is the
  normal way to start it. The same advice appears on the command line, in the `models` report and
  in the browser.
- **Update is offered only where it can work.** A copy that cannot replace itself, because it is
  running from a disk image or from a folder you cannot write to, says so and why, instead of
  failing partway through an install.
- **Double-clicking the program on Windows no longer leaves a console window open**, and a problem
  at startup is shown in a dialog rather than printed to a window that closes.

### Fixed

- **A provider that declines to answer is reported as a refusal rather than as a bad answer.**
  Some endpoints answer a request they will not fulfil with an ordinary successful reply that
  simply contains no text. The tool accepted that as a completed generation, so the failure
  surfaced one step later as "no parseable targets" or "unparseable JSON", which reads as the
  model answering badly and sends you after the wrong remedy: the model, the format, the token
  limit. Both the local and the remote path now stop on an empty answer and report what the
  endpoint actually said, including the reason it gave for stopping and the prompt tokens it read,
  and the recovery steps no longer offer a larger token limit for a reply that was never
  generated. An empty answer is also no longer retried, because an endpoint that declined this
  request declines it again at the same price.
- **A scoring panel says when it was short.** A panel model that fails or declines is skipped and
  the run continues on the models that answered, which is the intended behaviour; but the report
  named every model that had been configured and gave no sign that one of them contributed to no
  score. The caveat now names any model that returned nothing and reports how many scoring calls
  produced a judgment, and the JSON payload carries the same facts: `panel_models` is the set that
  actually scored, with `panel_models_silent`, `judgments_scored` and `judgments_asked` beside it.
  No score changes; the means were always computed from the judgments that arrived.
- **`revise_loops` now reports the revisions that were actually taken.** It reported the number
  you asked for. So a run where three revise passes were requested said three whether three
  revisions were adopted, three were thrown away because the model answered in a shape the parser
  could not read, or the critique converged on the first look and none ran at all. Two drafters,
  one whose revisions are accepted and one whose are all discarded, produced identical JSON. The
  field is now the count adopted, with `revise_loops_run`, `revise_loops_discarded` and
  `revise_loops_configured` beside it, and a pass that produces nothing usable says so while the
  run is going rather than passing in silence. Same in the web interface, and the revise endpoint
  gained the two counts it was missing.
- **The revise prompt stopped showing the model a format it would then refuse.** It displayed the
  current claims as `[1] ... [2] ...` while the claims already carried their own numbers, so a set
  was shown double-numbered, and asked for "the full numbered claim set" with no example. A model
  that copied the format it had just been shown had its whole answer discarded and the prior claims
  kept, silently. The claim set is now shown the way the tool reads one back.
- **A model the server does not have is reported as that, and nothing else.** Asking for a model
  that has not been pulled produced `API error (status 404)` followed by the server's raw JSON. It
  now says which server was asked and which model it does not have, and the recovery steps are the
  ones for a missing model rather than advice worked out from the wording of the message. The same
  applies to an OpenAI-compatible server you run yourself. Three other kinds of 404 that are NOT
  about the model (a routing limit, a zero-data-retention limit, and a router's own temporary
  unavailability) keep the advice that is right for each of them.
- **The browser gets the same diagnosis the command line does.** When a failure is one the
  tool can name exactly, the web interface used to work it out again from the wording of the
  error message rather than from what the client had already established. Mostly it agreed;
  for one failure it did not. A server that reads only part of your specification was answered
  with "raise the token limit", which makes the problem worse, because the size of input a
  model accepts is its context window minus the output budget. It now leads with choosing a
  model whose window fits, and says that nothing in the output would have shown you the
  specification was cut.
- **A failure with a specific diagnosis carries its full set of recovery steps.** Failures the
  tool can name exactly (a model that does not fit in memory, a server that read only part of
  your specification, an endpoint that returned nothing) printed a single next step, where every
  other failure carries at least two.

## [0.5.1] - 2026-09-30

### Added

- **A public patent to try it on.** The `samples` folder carries a granted application's public
  record: the Patent Center file history of 18/541,216 (US 12,299,058), exactly as Patent Center
  serves it, with its specification and its allowed claims. They let you try the whole flow
  before any client material goes near the tool. The download page walks through it.

### Fixed

- **v0.5.0 was assembled from two runs of the release.** One upload failed and the release was
  finished by a second run, so it went out without its Linux arm64 binary (since restored, byte
  for byte, to the checksum it was signed with), and its checksums file does not match its macOS
  binaries or its software bills of materials, which came from the other run. Those macOS
  binaries are genuine and signed, but on a Mac the command-line `update` refuses them, as it
  should. The Mac app updates normally. This release is built in one run, and a release can no
  longer be published unless every file its checksums name is attached and matches.

## [0.5.0] - 2026-09-30

### Changed

- **The Draft tab shows the directions first, and drafts only the ones you keep.** Drafting is
  the slow and costly part of a run, so the first press now finds the directions the
  specification supports and stops there. Each is listed with whether its supporting passage was
  found in your specification. Remove the ones not worth drafting, edit a direction's wording in
  the Continuation targets panel, then draft; only the directions you kept are drafted. Drafting
  straight through, as before, is one click away.

- **The web interface looks like the rest of the ObviouslyNot family.** Main buttons are black
  and turn cyan when you point at them, secondary buttons are outlined, panels have softer corners
  and a light shadow, labels are rounded tags, headings are stronger, and the ObviouslyNot mark sits
  beside the name. The dark theme has matching versions of each.

### Fixed

- **A prosecution-history reading no longer lists the law as art.** Reading an Office Action, a
  small model could report a statute, a rule or a court decision the examiner cited as if it
  were a reference of record. Those are now withheld and named as withheld; patents and
  publications, including a patent cited for double patenting, are unaffected.

- **Buttons were drawn in the wrong typeface.** Almost every button in the web interface, Draft
  Claims included, used the browser's default font rather than the interface's own.
- **Two things were hard to read.** The label naming a defect's type was white text on amber, and
  a secondary button's outline was too faint to show where the button ended. Both now meet the
  contrast the rest of the interface does.
- **A reopened matter said it had no drafted claims.** Opening a matter you had already drafted
  showed "Drafted Claims (0)" above its claims. The count now counts the claims on the page.
- **Opening a matter's link scrolled the top of the page away.** A bookmark, a reload or a link
  straight to a matter opened partway down, with the model picker and the tabs out of sight.
- **A ring outlined the whole page after every move between tabs**, and the cost of a matter and
  of the last run sat against the edge of their panel. Both are gone.

## [0.4.0] - 2026-09-29

### Added

- **Every proposed direction now says what its supporting quote actually is.** The tool used
  to answer one question about a quote, whether it appears in your specification, and a "no"
  covered several different situations that call for different work from you. It now
  distinguishes them: the quote is in the specification; it is not there at all; its parts are
  each real but were never adjacent, so the model fused two true passages into a sentence your
  document does not contain; it appears in more than one place, so it does not locate anything;
  it is too short for a match to mean much; or it is in the parent claims rather than in the
  specification. Each points at different work, and the one that used to be invisible is the
  fused quote, because a practitioner can go and read both passages.

- **Drafted claims are checked for two things a program can decide without asking a model.** A
  claim reciting a number that appears in neither your specification nor the parent claims is
  flagged with the surrounding text, because an invented quantity reads perfectly well in
  otherwise correct prose. And a direction wholly contained in another is flagged as
  redundant, so that near-duplicates are not several readings of one decision. Both report
  what they found and where; neither is a verdict, and every judgement about what a finding
  means remains yours.

- **Small local models are asked for output in a shape the server enforces.** Where the server
  supports it, the request now carries the structure of the expected answer rather than only
  describing it in words. Measured while building it: a small local model returned an empty
  result on a weakly worded prompt and returned usable output for the same prompt once the
  shape was enforced. Servers that do not support it are unaffected.

- **A run now says what happened to it rather than only that it failed.** Before a long run
  starts, the tool checks whether your model server has a model wedged past its own unload
  time, which is the state in which every later call waits behind it and reports a timeout
  that looks like a slow model. It reports and never refuses, because a clock can drift. A
  reply that was cut off mid-answer is now identified as truncated rather than reported as a
  parsing failure, and failures are grouped by what you would do about them, so a network
  problem and a model refusing to answer no longer read the same.

- **Every drafting run now reports how much of your specification it actually read.** A line
  at the end says what percentage of the document the located supporting quotes touch, and
  where the largest stretch nothing cited begins. The tool could already tell you that a
  proposed direction was fabricated; it could not tell you that it never looked at column 7,
  and for a tool whose job is finding unclaimed matter that is the more expensive silence.
  It is a disclosure and not a score: a low figure on a specification with little unclaimed
  matter is correct.
- **A specification too large for your model can now be read in overlapping windows instead
  of being refused.** Off by default, because a model that can hold the whole document should
  get the whole document, and set `CD_WINDOW_LARGE_SPECS=1` when the alternative is not
  running at all. Windows overlap, by enough that a supporting quote is not cut in half at a
  boundary. Every window is accounted for, including the ones that produce nothing, because a
  window that quietly vanishes leaves a result that looks clean and is smaller than it should
  be, and the run tells you the count rather than keeping it. A windowed run says so while it
  is running, because it is a degraded mode and should not be mistaken for an ordinary one.
  If every window fails, that is reported as a failure: a model server that was unreachable
  for the whole run must not read as a specification with nothing left to claim.

- **`continuation-drafter compare`, which asks which of two drafts is the better drafting job
  and tells you how sure it is.** It controls for the fact that a judge can answer differently
  depending on which draft it is shown first, which is a property of the model rather than of
  your claims.

  It reports one of three things: that one draft is better, that it could not find a
  difference, or that it could not tell, with the reason. "Could not tell" is a real answer
  and will be a common one. It also reports how consistent the judge was with itself, which is
  the number that says whether to trust the rest.

  Measured while building it: comparing a draft **against a copy of itself**, a small local
  model agreed with itself on one pair in eight, while a larger one agreed every time and
  answered tie on every pair. The small model, scored the old way, would have produced a
  confident-looking preference between a document and itself.

  **The larger model's run still came back "could not tell", and that is the tool working
  rather than failing.** Eight comparisons cannot establish that two things are equivalent,
  however unanimous they are: showing a difference is cheap and showing sameness is expensive,
  so a run needs roughly a hundred comparisons before "could not find a difference" is an
  answer the numbers support. A tool that reported "they are the same" from eight unanimous
  ties would be telling you something it does not know.

- **`continuation-drafter bench`, which measures whether your model server runs requests at
  the same time or one after another.** It sends a few short requests both ways and reports
  the difference. This matters because a multi-model panel is several requests: on a server
  that runs them together it finishes in a fraction of the time, and on one that queues them
  it does not. Which yours does depends on the server and the model format, not on this tool,
  so it is measured rather than assumed.
- **Panel scoring can run its model calls together.** Off by default, because raising memory
  use on a machine that may already be near its limit should be a choice you make after
  seeing your own number. `bench` tells you whether it is worth setting, and prints the
  setting to use. Measured on one machine: a two-model, two-round panel went from 58 seconds
  to 19. The revise loop stays sequential, because each pass reads the previous pass's output.

- **Local models on any OpenAI-compatible server, not only Ollama.** `--provider local-openai`
  points the local path at mlx-openai-server, vllm-mlx, llama.cpp's `llama-server`, LM Studio,
  LocalAI or SGLang, with no API key. On Apple silicon these are the servers that run several
  requests at once; Ollama's MLX engine handles them one at a time. Set
  `CD_LOCAL_OPENAI_BASE_URL` if your server is not on the default port.

  **Whether this counts as local is decided by where the request goes, not by the name.** If
  the address you configure is not on this machine, the tool says so before your
  specification is sent and treats the run as remote, because a machine on your own network
  is still across a network.

- **Repeated work on one specification is cheaper and faster.** Most of what the tool sends a model
  is your specification, and it used to be re-read in full on every call. It is now arranged so a
  model server or provider can reuse the part it has already read: on a local model, later passes
  over the same specification spend seconds reading it instead of most of a minute; through
  OpenRouter, Claude reads it from its prompt cache after the first call, which made a follow-up
  question in the local web UI about an eighth of the cost. Measured before shipping, drafts scored
  within the benchmark panel's margin of each other with and without the change.

- **The revise loop does less work and stops sooner.** It used to rewrite the whole claim set on
  every pass, however few claims needed it; it now rewrites only the claims that do, leaves the rest
  exactly as they were, and stops when another pass would not help. A rewrite that damages the
  claim it replaces is refused and the original kept. Measured on a small local model over six
  specifications: the five that both versions finished took 18% less time, and the sixth, which
  the old loop could not finish inside its time limit, now finishes. A deletion, or an instruction
  you give, still revises the whole set.

- **OpenRouter can be limited to the hosts you choose.** OpenRouter hands each call to one of the
  companies hosting the model, and may choose a different one each time: a Claude model there is
  hosted by Anthropic, Google, Amazon and Microsoft. `CD_OPENROUTER_PROVIDERS` names the hosts you
  allow and `CD_OPENROUTER_ZDR=1` allows only hosts that keep nothing; both are also in the web
  interface's Settings. The note printed before a run says which applies, and a limit no host
  meets is refused before anything is sent, with advice on what to change.

- **Check for updates, and install one after it verifies.** Settings has a Check for updates
  button, and `continuation-drafter update` does the same from the command line. Nothing is
  checked unless you ask; the check is one request to github.com carrying nothing about you, and
  offline mode refuses it. Before anything is replaced, the release's checksums must carry a
  valid signature from the key built into your copy, the download must match its checksum, a Mac
  download must carry Apple's signature for this publisher and pass Gatekeeper, and the new
  program must report the version it was fetched as; if any check fails, nothing changes. The
  web interface restarts itself into the new version. This release is the first to carry it, so
  it is the last one you download by hand.

- **A run ends by saying what it cost and where the specification went.** Every command that
  calls a model, and the web interface after each drafting run, revision, critique or
  conversation turn, reports the model calls made, the tokens read and written, the cost as the
  provider reported it, and the companies that served the calls, which through OpenRouter can be
  more than one. A run on your own computer reports $0 beside what the same tokens would cost on
  a leading hosted model at its list price, dated. On the command line it is printed last, and
  also when a run fails partway; `draft --json` carries the same figures. In the web interface each
  matter also keeps a running total of what it has cost so far and which companies received its
  specification, shown while the matter is open.

- **Per-call cost and cache use in the request log.** With `CONTINUATION_DRAFTER_LOG_REQUESTS=1`,
  each line now shows the cost the provider reported for the call, how much of the prompt was read
  from or written to a prompt cache, and, on a local model, how long it spent reading the prompt
  and loading the model.

### Changed

- **A prompt cache holds your specification for a while after a call returns, and the tool now
  asks for the shortest retention each provider offers it.** With an OpenAI key the tool asks for
  in-memory retention; OpenAI's newer models refuse it (gpt-5.5 keeps a cache for up to 24 hours,
  gpt-5.6 and later for 30 minutes after last use), and the tool then proceeds without the request. For Claude through OpenRouter the cache lasts up to an hour after its last
  use. The setup page for remote providers now says what each provider keeps.

### Fixed

- **The confirmation before a remote run named OpenRouter whatever provider you had chosen.** It
  now names the provider that will receive the specification, as the banner above it already did.

- **`bench` no longer counts the time to load your model as evidence that your server
  batches.** It timed a run of calls one after another, then the same calls at once, and
  started the first timer on the very first call. If the model was not already in memory,
  loading it landed entirely in the first measurement. On one machine that reported four
  eight-token calls taking 237 seconds against 2.2 seconds, a ratio of 108x, and recommended
  raising concurrency on that basis. It now makes one throwaway call first, and the same
  machine reports 0.6 against 0.3, a ratio of 1.8. The recommendation was the thing at stake:
  raising concurrency on a server that does not batch buys no speed and raises peak memory,
  which is what this command exists to help you avoid.

- **Runs against a local model no longer reload the model between passes.** Ollama unloads and
  reloads a model whenever the requested context window changes, and this tool sized that window
  from the exact length of each prompt, so no two passes of a run asked for the same one. Two
  consecutive passes over one specification asked for windows nine tokens apart, and the model
  was moved in and out of memory between them. The window is now rounded up to a fixed step, so
  the passes of a run ask for the same one and the model stays where it is. Nothing about the
  output changes: the same specification produces the same claims, the same coverage figure and
  the same flags. What changes is how much of a run is spent waiting, and the larger your model
  the more of it there was.

- **Drafting no longer comes back empty on some local models.** The shared prompt layout sent
  one system message for every step of a drafting run, and it told the model to answer in JSON.
  Two of those steps ask for numbered claim text, not JSON, and on at least one local model the
  drafting step returned a JSON object of claims instead: a complete, correct answer in a
  container the tool does not read, so the run ended with "drafting produced no claims". Each
  step now names the shape it wants, the shared message no longer names one for everybody, and
  a claim set that arrives as JSON anyway is read rather than discarded.

- **Prosecution-history questions are answered from the whole paper.** The record reading was
  focused by the first question asked and then reused for later ones, so a follow-up about other
  claims could be told the paper said nothing about them. Small local models also failed to read
  some Office Actions at all, returning a field in a form the tool rejected.

- **Saving a setting in the web UI no longer switches off an exported environment switch.**
  Choosing a model saved the settings and, in doing so, cleared `CD_NO_LICENSE_RENEWAL` or the
  request log if you had set them in your shell.

- **The note about a specification leaving your machine no longer appears when it does not.**
  It was shown for any provider other than Ollama, which was true of every provider until a
  local OpenAI-compatible server became one, and it then announced that a specification sent
  to your own computer had left it. It now follows the destination.
- **The deprecated `OPENAI_BASE_URL` redirect.** Pointing that variable at a local Ollama
  endpoint still works and is no longer the way to reach a non-Ollama local server: use
  `--provider local-openai`. The old route could not work for those servers anyway, because
  the Ollama client speaks Ollama's own protocol.

- **A long run is no longer cut off by a deadline nothing could move.** A run carries a
  deadline for each model call and one for the whole run, and the two were set separately:
  20 minutes per call, retried once, inside a flat 30-minute run. A first call could spend
  20 of the 30 minutes and the retry be killed mid-attempt, and in the local web UI the run
  deadline was hardcoded, so no setting could lift it. The run deadline is now derived from
  the per-call one so that a call and its retry always fit (45 minutes at the default), and
  `CD_RUN_TIMEOUT_SECONDS` or `--timeout` overrides it. The recovery steps now say which of
  the two deadlines stopped a run and name the setting that moves that one; a run killed by
  its own budget used to be answered with advice about the other. Reported by a practitioner
  running a large local model in the web UI.
- **Eight more operations could not finish on a slow machine, for the same reason.** An
  audit after the fix above found every other run-level deadline in the binary set
  independently of the per-call one: five web handlers (chat, streamed chat, best practice,
  critique, limitations) waited ten or fifteen minutes on calls allowed twenty, so the
  operation was killed while the model was still legitimately working, and three commands
  (`critique`, `bestpractice`, `judge`) defaulted to exactly twenty, which funds one call
  with no margin and no retry. All are derived now, and a guard refuses a literal deadline
  anywhere in the binary unless it is recorded as a different kind of wait (a model
  download, a shutdown, a probe that degrades to "not detected").

## [0.3.0] - 2026-09-18

### Added

- **The parent's prosecution history, from Patent Center.** Attach the "download all
  documents" PDF to a matter and the tool reads its bookmarks to find the papers a
  continuation decision turns on: the Office Actions, the responses with their claim
  listings and remarks, the Notice of Allowance, and the references of record. Every page
  of such a file is an image; the pages are read from a text layer if you have run
  recognise-text over the file, or by tesseract if it is installed on your machine, with
  each page passed on standard input and nothing written to disk. A paper that cannot be
  read says why. Nothing is sent anywhere.
- **Where each limitation of the parent's claims entered the record.** With the parent's
  claims as allowed or issued in the matter, the Prosecution record panel shows, for every
  limitation, whether it was in the claims as filed or added in a response, which Office
  Action that response answered, and whether the allowance followed. These are facts about
  dated papers. What they mean for a continuation is your reading.
- **"What the examiner said."** A Discuss skill that reads an Office Action or Notice of
  Allowance and reports which claims were rejected, on what ground, over which references,
  and what the reasons for allowance credited, each with a quotation checked against the
  paper. It reports the record and never whether the examiner was right or whether an
  amendment was needed; a model's attempt to say so is withheld and shown as withheld.

### Fixed

- PDF pages are read in the order the document's page tree gives them, not by object
  number. The two orders had coincided on every file tried, so this had never shown.
- An upload larger than the limit is refused with a message that says so. It was cut at
  the limit with no error and then refused for the wrong reason.
- PDF streams whose `stream` keyword is followed by a bare carriage return, image
  dictionaries whose filter is a name, and dictionary keys that are prefixes of their
  neighbours (`/Decode` inside `/DecodeParms`) are all read correctly. Each of these was
  found on a real Patent Center file.
- LZW-compressed PDF streams decode with the early-change variant PDF uses by default.
- **A long specification on the recommended local model finishes.** Each call to a local
  model had a ten-minute deadline, and the 32 GB tier's recommended model was measured at
  up to twelve minutes for one draft, so a practitioner following that recommendation on
  a long specification saw "after 2 attempts, each timing out" and was told to pick a
  smaller model. The per-call deadline is now twenty minutes, the error names the
  variable that raises it (`CD_REQUEST_TIMEOUT_SECONDS`) instead of the run's `--timeout`,
  and the recovery steps for a call that timed out lead with that variable before they
  suggest a shorter specification or a faster model. Reported by a practitioner on a 64 GB
  machine.
- A specification that does not fit the model's context window is answered with the right
  recovery steps: choose a larger-context model, or lower the output cap to raise the input
  ceiling. It was answered with the truncation advice, which told the reader to raise the
  output cap and so made the refusal more likely.

## [0.2.4] - 2026-08-29

### Added

- **A bill of materials with every download.** Each binary now ships an SBOM listing every
  library inside it, with versions, in the standard SPDX format. It detects nothing by
  itself; it is what lets you answer "does the build I am running contain this" months
  from now, when an advisory lands against a library, without rebuilding anything. The
  documents are covered by `checksums.txt` like every other file.

### Changed

- **The download page now leads with your own operating system.** The button, the
  instructions, the command examples and the order of the binary list all follow the
  machine you are reading on. A Windows visitor was previously offered a macOS `.dmg` as
  the only button on the page, with commands underneath that their shell could not run.

- **A page explaining what is checked, and how you can check it.** Where your
  specification goes, who signed the download, the published SHA-256, and the
  vulnerability scan that gates a release. All of it was already true and all of it was
  buried inside the install steps.

### Notes

- **Nothing about drafting changed.** The pipeline, the models, the skills and the local
  web UI are identical to 0.2.3.

## [0.2.3] - 2026-08-28

### Changed

- **The Windows binary is signed.** Windows Defender and SmartScreen blocked the previous
  download, because the `.exe` carried no signature at all: the macOS builds were signed and
  notarized and Windows had nothing. Releases are now signed with an Azure Artifact Signing
  certificate issued to Geeks in the Woods, which is the name Windows shows as the verified
  publisher. The signature is timestamped, so it keeps verifying after the signing
  certificate itself expires.

  The release refuses to publish an unsigned Windows binary rather than shipping one quietly.
  It asks Windows itself whether the signature verifies, and requires both a valid status and
  a timestamp before the release leaves draft. A signing step that silently matched nothing
  is how an unsigned macOS binary shipped in v0.1.3.

- **`checksums.txt` changes for the Windows entry.** Signing rewrites the file, so its hash is
  not the one an unsigned build would produce. Verify a download against the checksums
  published with the same release, never against an earlier one.

### Notes

- **0.2.2 was tagged and published nothing.** Its Windows signing step failed on a path bug,
  and the pipeline did what it is built to do: the release stayed a draft rather than
  publishing a binary that was unsigned or whose checksum did not describe it. There is no
  0.2.2 download and there never was one.

- **Nothing about drafting changed in this release.** The pipeline, the models, the skills and
  the local web UI are identical to 0.2.1. If Windows was not blocking your download, there is
  nothing here you need.

## [0.2.1] - 2026-08-26

### Added

- **Claims are laid out one element per line.** A claim sets out its elements separated by
  semicolons, and the page was showing all of them as a single paragraph: one claim measured
  3,284 characters as an unbroken block. Each element now begins its own line, indented, which
  is the form the rules prescribe for a filed claim and the form a practitioner reads. The
  wording is untouched; only the whitespace differs. Word, PDF and the plain-text export carry
  the same layout, and so does Copy.

- **You can check claims without rewriting them.** The only thing you could do with a drafted
  set was ask a model to revise it, which changes your claims. There is now a check that
  reports what is wrong and alters nothing, offered first on claims nothing has reviewed and
  alongside revising when findings exist.

- **A list of your claims, beside them.** Each row shows the claim number, whether it is
  independent or which claim it depends on, what distinguishes it, and how many findings it
  carries. Clicking one jumps to it.

- **Answers in the Discuss tab are formatted.** Headings, lists, bold and quoted terms used to
  arrive as their own punctuation.

### Changed

- **A revision shows only the elements that changed.** A rewrite touching two elements of ten
  was rendered as the entire claim in red and green; the untouched elements now sit quietly
  and the changed ones stand out.

- **Claims and answers use more of the window.** Both were held to the width of a paragraph of
  prose, which suits an essay and not a claim.

- **The page says what to do next in every state**, including when a claim set has been drafted
  but not reviewed, and it always offers a way to open the conversation about it.

### Fixed

- **A drafting run's work reaches the matter it belongs to.** Claims, findings, craft flags and
  continuation targets were computed and then written nowhere, so the Discuss tab could see
  none of it and the monthly counter could not tell a re-run from a new matter.

- **A file dropped onto the page is saved.** Choosing one with the file picker saved it;
  dropping the identical file did not, which was invisible until the Discuss tab reported
  having no application.

- **A review that fails says so.** When the critique pass could not complete, the page showed
  no findings and no explanation, and reopening the matter then reported the claims as
  reviewed and clean. Whether the check actually ran is now recorded and shown.

- **A malformed reply from the model is asked again.** Roughly one review in eight came back
  with a stray character that made it unreadable, and the whole review was discarded for it.
  The retry also varies its sampling, since repeating an identical request produces an
  identical answer.

- **Errors say what happened and keep the evidence.** A failed review reported only a byte
  count. It now distinguishes a reply that ran out of room from one that was never valid, and
  quotes what came back.

- **A reference to a section of your specification is no longer mistaken for a citation of
  law**, and a reference to the parent's claims is no longer reported as invented.

- **Asking how to improve a claim gets claim language**, not a refusal about filing strategy.

## [0.2.0] - 2026-08-25

### Added

- **OpenAI and Anthropic join OpenRouter as cloud providers.** Choose with `--provider
  openai|anthropic|openrouter`, or pick a provider tab in the model picker. Each reads its own
  key (`OPENAI_API_KEY`, `ANTHROPIC_API_KEY`, `OPENROUTER_API_KEY`) from the environment or
  from the same private config file, and the web interface can save one for you. Nothing about
  where keys come from has changed: never a flag, never a log, never the repository.

  **Every command you already use keeps working.** With no `--provider`, the model string
  still decides exactly as before: a colon tag runs on this machine, a slash slug or a bare
  vendor prefix goes to OpenRouter. The flag exists because the model string cannot express
  the new choice: `anthropic/claude-opus-4` is OpenRouter's own name for that model, so the
  same text would name two different destinations.

- **The privacy note names the provider and the endpoint.** It said "the remote provider",
  which was as specific as it could be when there was one. With three, the single fact it
  exists to convey is which company receives your specification, so it now says so, in the
  terminal and in the browser.

- **The model picker lists each provider separately and curates what it shows.** Text models
  only, no purpose-built variants (codex, search, deep-research), no billing or preview
  variants, and nothing much older than the previous generation. OpenRouter is further limited
  to the frontier labs, which is an editorial judgement rather than a popularity ranking:
  no usage data is published, so no such ranking can be computed. With an empty search box you
  see the fifty most recent and a count of the rest; searching reaches the whole curated list.
  Every provider pane also has a box to name a model directly, which reaches past the catalog.

- **Discuss can look for broader scope in claims you already have.** Ask how to widen a
  claim and the tool works through four moves against your existing claims: who performs the
  reciprocal or downstream operation, which limitations describe one implementation rather
  than the invention, whether a per-element step can be claimed as an aggregate, and whether
  the structure the process produces can be claimed instead of the process. Until now the
  only mapping pass looked away from your claims, at what else the specification disclosed,
  so a question about the claims themselves had nowhere to go.

- **Discuss can report which limitations are narrowing you for nothing.** Ask what a claim
  set gives away and the tool reads each limitation against the specification and reports the
  ones the disclosure does not require: a specific term where a broader category is
  disclosed, a step tied to one component that could happen elsewhere, a recited mechanism
  where only the result matters, or an "each" the disclosure says can be a portion. Each
  finding names the sentence of your specification that supports the broader alternative and
  says whether to widen the claim or keep the narrow position as a dependent claim. Available
  in the conversation and at `POST /api/limitations`.

- **Discuss can describe what keeping your current claim center would look like.** Not every
  continuation goes somewhere new. Ask about filing another application on the same invention
  and the tool states the inventive center as a workflow, identifies whether the entity you
  nominally claim (a server, a backend, a service, a client) is essential or merely what
  happens to execute it, points out recitations that only establish that software runs on a
  computer, and names which existing dependent claims are worth carrying forward. It never
  recommends preserving: whether your current center deserves another family member turns on
  commercial and prosecution judgement it cannot see.

- **The broadening pass considers two more moves.** Alongside actor, abstraction, granularity
  and artifact, it now asks which phase of the system's life is unclaimed (provisioning,
  configuration, update, migration, expansion, recovery, retirement) and which other product
  disclosed in your specification could perform the same role, since a role can stay identical
  while the product performing it changes.

- **The continuation directions found for you are listed beside the conversation**, each
  labelled with whether its supporting quote was located verbatim in your specification, and
  each removable. Directions you keep are what the drafting pass writes claims to.

### Changed

- **What you write in a message now reaches the pass that runs.** Previously every pass ran
  on the document and the tool's own analysis, and your wording was discarded before the
  model saw anything, so a paragraph of drafting instructions changed nothing. It now steers
  the pass it triggered.

- **The assistant no longer suggests what to do next.** It was recommending capabilities of
  this product without ever being given the list of them, so the names were invented. The
  next steps beside your reply are derived from what your session actually contains, and are
  now the only place that advice comes from.

- **Drafting asks for a second statutory class where your specification supports one.** Of
  nine real continuations examined, six pair a method claim with a device, system or
  computer-readable-medium claim, and the drafting prompt had never raised the question. A
  method claim reaches the party performing the steps and a device claim reaches the party
  making or selling the thing, so a family in one class leaves the other party unreached. It
  is conditional: a method the specification never embodies in a described device does not
  earn a device claim, and padding a class with a restatement of the first claim is forbidden.

### Fixed

- **A claim set always comes with dependent claims.** Drafting from a single direction
  produced one bare independent claim and no fallback positions, because the instruction
  asked for a set of three or four independents and the model read it as inapplicable. A
  family without dependent claims is missing the claims that matter when the independent one
  is rejected. Drafting from one direction now yields that claim plus two to four dependents,
  and a set that comes back without any says so.

- **A large specification no longer fails with an error about the connection.** On
  specifications over roughly 150 KB the local model could stop mid-answer after producing a
  usable result, and the tool discarded it and retried four more times before reporting a
  transport error. The usable part is now kept, and a stop that carries an answer is no
  longer treated as a temporary glitch worth repeating.

- **Describing a problem in a claim is enough to have it fixed.** Asking to rewrite a claim
  you had just described a defect in was refused with a request for a critique first, even
  though the rewrite never used one.

- **The first document attached to a matter is no longer announced as a replacement.** It read
  "Replaced the document with ..." and drew a divider saying everything above it described a
  document no longer loaded, above a transcript with nothing in it. A matter has an identity
  from the moment you create it, and that was being read as a conversation already holding
  something.

- **Reopening a matter shows what it is working from.** The row naming the loaded application
  stayed hidden on a matter opened from the sidebar, and attaching a second document to one
  discarded the earlier exchange while the transcript stayed on screen.

- **A run that times out twice now stops instead of timing out five times.** A timeout can be
  a busy machine, and one retry still covers that. It can equally be a property of the request,
  and then every further attempt re-sends the same document and waits the same deadline for the
  same answer. On a 660 KB specification that cost fifty minutes locally to learn what the first
  attempt had already established, and would have cost longer on a remote provider.

- **A timeout no longer tells you to start a model server that is already running.** The advice
  now leads with the size of the specification, which is what a timeout on a retry usually
  means: a document can fit a model's context window and still take longer than any deadline.
  Timeouts from remote providers previously produced no specific advice at all.

- **Advice about remote models names every provider.** Two messages still said a remote model
  runs on "your own OpenRouter key" after OpenAI and Anthropic were added.

- **A model on your own machine no longer asks for a cloud key.** Selecting a local model
  while a cloud provider was saved as the default produced "model \"gemma4:26b\" on provider
  \"Anthropic\" needs ANTHROPIC_API_KEY", which is a fair question to be confused by: it did
  not need one. A model name carrying a colon is Ollama's own naming and says where it runs,
  so a saved default no longer overrules it. **With a cloud key already saved this was worse
  than an error**, because the specification would have been sent to that provider while the
  model chosen was sitting on the local disk. Naming a cloud provider explicitly still works
  and now says plainly that the provider does not use that kind of name.

- **The model picker and the tool agree about where a pass will run.** The provider chosen in
  the picker was saved in the browser and never read back, so after a reload a remote model
  was tagged with a provider name that no longer existed, and a computed default told the
  browser one thing and the tool another. Both now record the same choice, and a provider name
  the tool does not recognise is refused when it is set rather than surfacing later as a
  failed drafting run.

- **A revision you keep is now actually kept.** Revising produced a proposal, and the button
  that adopted it changed only the page in front of you. Reloading brought the previous claims
  back, and the Discuss tab, which works from the matter, went on answering about the claims
  you believed you had replaced, with nothing saying so. Keeping a revision now saves it, and
  Undo saves the originals back, so the conversation and the page never disagree about which
  set is current. The button also names what it does: it said "Put these in the claims box",
  and that box had been removed some time ago.

- **The draft page says what to do next in every state it can be in.** One notice did the work
  of three. After a revision it still read "Next step: act on the findings below" above the
  proposal it had just produced, and pressing that button again would have started over from
  the original claims and discarded the proposal. When nothing was flagged it said nothing at
  all, so a clean set of claims simply ended the page. There are now three: revise the
  findings, go and read the revision that is waiting, or, when nothing is flagged, ask about
  the claims or export them.

- **A dropped file is saved to the matter, the same as a chosen one.** Attaching a document by
  clicking Browse saved it; dropping the identical file on the same box did not. Drafting
  worked either way, because the page sends its own copy of the text, so the difference was
  invisible until the Discuss tab, which works from the matter, answered "that needs
  specification, which this session does not have" about a document plainly on screen.
  Dropping a file also names an untitled matter now, which only the Browse path did.

- **A drafting run's work reaches the matter it belongs to.** The page never told the run which
  matter it was for, so everything a Draft run computed, the claims, the findings, the craft
  flags and the continuation targets, was written to nothing. The Discuss tab could see none
  of it, and the monthly counter could not tell a re-run of a matter from a new one.

- **The continuation targets a drafting run identifies are kept and shown.** The run reported
  "5 drafting target(s) identified" in its own progress log and the panel beside it was empty,
  because the targets were used to draft and then discarded. They now appear in the rail, with
  the same in-spec or strategy badge as everywhere else. A run fills an empty rail and never
  replaces targets you have edited, since those are what the conversation drafts from.

- **A reference to a section of your specification is no longer mistaken for a citation of
  law.** The tool replaces any answer that cites a statute, because an unchecked citation is
  the least reliable thing it could show you. It was reading "supported by Section 10" and
  "Sections 14 and 20 of the application" as citations, so a full claim-by-claim analysis was
  thrown away and you were told the tool had tried to quote law when it had not. Real
  citations are still caught, including invented ones.

- **A reference to the parent's claims is no longer reported as invented.** The tool warns when
  an answer mentions a claim number that does not exist. It counted only the drafted set, so
  with ten drafted claims and a parent numbered to twenty, "drafted claim 8 captures elements
  from the source claims 14 and 20" was flagged as unreliable. That comparison is the tool's
  purpose, and the warning was discrediting correct work.

- **Asking how to improve a claim gets claim language, not a refusal.** "How can I improve
  them" was answered with "I cannot advise you on whether to file certain claims", because the
  rule against advising on filing was being read as covering how to word a claim. A filing
  decision is whether to pursue a claim and remains yours; how to word one is drafting, and
  drafting is what this tool is for. It still does not assess patentability, novelty,
  non-obviousness, eligibility or infringement, and still does not quote law.

- **PDFs printed from a browser can be read.** A patent application saved as a PDF from
  Chrome or any Chromium-based browser was refused as unreadable, with a message suggesting it
  was a scan. It was not: the text was there and other readers extracted it cleanly. Five
  separate faults in the PDF reader had to be corrected together, since each one alone left
  nothing usable. Hex-encoded text, which is what a browser writes, was skipped entirely.
  Character ranges in a font's character map were ignored, and a range is how a font describes
  an alphabet. Every font in the document was read through one shared table, so subset codes
  from one font were decoded as another font's letters. Images were parsed as though they were
  text. And each line was broken wherever the writer moved the cursor, which this kind of file
  does for every single character.

- **Ligatures no longer break the check that a quotation is real.** Text extracted from a PDF
  carries typographic ligatures, so "satisfies" arrives as a word containing one combined
  character. The supporting passage recorded for a target is verified by matching it against
  the application word for word, and a passage written with ordinary letters could not be found
  in text spelled with ligatures, so a target with a perfectly good quotation was reported as
  having none. Extraction now folds them to ordinary letters, as other PDF readers do.

## [0.1.7] - 2026-08-21

### Fixed

- **The browser tab icon is the current logo again.** The local interface shipped an
  earlier mark on an opaque white plate while the website carried the not-equal mark, so
  the tool you ran looked like a different product from the one that described it, in the
  tab you leave open for a whole drafting session. It is now the same mark, transparent,
  and 1 KB instead of 70 KB.

## [0.1.6] - 2026-08-21

### Added

- **The macOS app has an icon.** The ObviouslyNot not-equal mark, in the brand gradient,
  on a rounded plate. It shipped in 0.1.5 with the generic application icon, which on the
  artifact whose whole job is to look trustworthy at first contact is the first thing a
  practitioner sees.

## [0.1.5] - 2026-08-21

### Added

- **A macOS app you open by double-clicking it.** Download
  `continuation-drafter-macos.dmg`, drag **Continuation Drafter** to Applications, and
  double-click it: the drafting interface opens in your browser. No Terminal, no
  permissions to change, and one download that runs on both Apple silicon and Intel.

  This exists because the previous release could not be opened that way and never could
  have. A binary downloaded through a browser arrives without an execute bit, and a bare
  file has no bundle, so double-clicking it opened a text editor showing the program's
  raw bytes. Signing fixed a different problem (macOS killing the file silently) and left
  this one untouched.

  The app is signed with a Developer ID certificate, notarized by Apple and **stapled**,
  so first launch needs no network check. macOS still asks once whether you are sure you
  want to open something downloaded from the internet; the button is **Open**.

  It has no Dock icon, because it runs no window of its own. Press **Stop** in the
  browser tab to shut it down.

- **A Stop control in the web interface.** The only way to end a session started from the
  app: there is no terminal to interrupt and no Dock icon to quit.

### Changed

- **The command-line binaries are unchanged and still published.** They remain the right
  download for a terminal, a script or CI, and the install steps for them are the same.
- **`CD_PORT` now applies to a bare invocation**, not only to `serve`. It set the port for
  one of the two ways of starting the interface and silently did nothing for the other.

### Fixed

- **Starting the interface twice no longer fails to bind.** A second double-click, or a
  second `continuation-drafter` in another terminal, now opens the browser at the session
  already running instead of reporting that the port is in use.

## [0.1.4] - 2026-08-21

### Added

- **The macOS binary is now signed and notarized by Apple.** Downloading it no longer
  produces the "Apple could not verify" dialog, whose default button is *Move to
  Trash*. Nothing has to be done to the file after downloading: no `chmod`, no
  clearing of the quarantine flag, no Terminal at all if you do not want one.
- **Running `continuation-drafter` with no arguments opens the drafting interface in
  your browser.** Drafting, per-claim critique, scoring and the multi-model panel are
  all there, and the models it offers are the ones already installed on your machine.
  The command line is unchanged and `--help` still lists every command; a bare
  invocation from a script or CI still prints usage and exits rather than starting a
  server.

### Changed

- **`models` now recommends a model you already have.** It reported the installed
  count and then suggested downloading something else, which on a machine with a
  full Ollama library meant being told to fetch tens of gigabytes for no reason.
- **The command list is grouped**, with the interface, model check and drafting under
  "Start here" instead of twelve commands in alphabetical order.

### Fixed

- A next-step loop, and duplicate steps, in the guidance printed after a command.
- `upgrade` advertised a purchase in its help text while correctly refusing to make
  one, since there is nothing to buy.

## [0.1.1] - 2026-08-18

### Security

- **The local web UI now refuses cross-origin and non-loopback requests.** Any web
  page you had open in another browser tab could previously POST to the tool's
  local server. The damaging case was the Ollama host setting: a hostile page
  could point it at a machine it controlled, and every later run of a *local*
  model would then send your specification there while the tool still reported it
  as local. Requests must now come from a loopback origin and be addressed to a
  loopback host, which also closes the DNS-rebinding variant. If you reach the UI
  through a hostname or a proxy, use `http://127.0.0.1` directly.
- **The configuration file's permissions are now repaired on every save, not just
  when it is created.** A config that had become world-readable by any route
  stayed that way, with your API key in it. Saves are also atomic now: an
  interrupted write can no longer destroy your license keys and API key together.
  On Unix the file is always restored to owner-only.

### Added

- **`draft --demo`, a complete run that needs nothing from you.** The download is a
  single binary, so before this you had to supply a specification and its filed
  claims before the tool would do anything at all. The walkthrough specification
  the published lessons use is now built in: `continuation-drafter draft --demo`
  drafts against it from any directory. It never counts against any allowance.
- **`models` now tells you what this machine can run**, instead of only what you
  have configured. It reports how much memory you have, whether Ollama is
  reachable, and which measured model fits with room to spare, saying whether you
  already have it or need to pull it. Where memory cannot be detected it says so
  rather than guessing, and it states plainly that the best local model still
  scores materially below a remote one.

- **`export`, which writes your drafted claims to a Word document.** Claims come
  out of this tool as text, and the work continues in a word processor, so
  `export --draft claims.txt --out claims.docx` gives you a `.docx` that opens in
  Word, Pages or LibreOffice. It also reads a `draft --json` run straight off a
  pipe, so drafting and exporting are one command. **Your claim numbering is
  preserved exactly as drafted and never renumbered**, since a continuation's
  claims are commonly numbered on from the parent. The file is written owner-only
  (mode 0600) because it holds client claim language, and it will not overwrite an
  existing file unless you pass `--force`, so re-running the command cannot destroy
  edits you made in Word. The maturity notice is written inside the document, where
  it is visible to whoever opens the file later. Free, and not licence-gated.
- **LaTeX export**, for drafters who keep matter documents that way:
  `export --draft claims.txt --out claims.tex`. The format follows the output
  extension, so you do not have to say it twice; pass `--format` to override.
- **Licence renewal, for paid subscriptions only.** So that you activate a licence
  once and never handle a key again, the binary fetches a replacement when the
  current one is close to expiring: roughly once a month, on launch, not on a
  timer. It sends the licence key and nothing else, and carries no specification
  text, no matter identifiers and no usage data. Set `CD_NO_LICENSE_RENEWAL=1` to
  switch it off entirely; a licence that is never renewed simply works until it
  expires, and the tool keeps working normally if the renewal cannot be reached.
  **Without a paid licence the binary makes no such call at all.**
- **A dependency and standard-library vulnerability scan now runs on every change, and
  again when a release is built.** The second one is the half that reaches you: the
  release pipeline resolves its Go toolchain fresh when it runs, so the standard library
  compiled into a published binary is not necessarily the one that was scanned when the
  change was reviewed. The scan runs before anything is published, so a release stops
  rather than shipping a binary built from an unscanned standard library.
- A warning if a build's embedded licence key is malformed, so a broken build says
  so instead of quietly behaving as unlicensed.
- **A `See also` line at the end of the suggestions**, pointing at the
  documentation site, so a pointer to something worth reading does not have to
  pretend to be a command you can run.
- **Every command now tells you what to do next, in prose as well as in `--json`.**
  Previously the suggestions existed only under `--json`, so running the tool
  normally showed none at all, and `version` showed none either way. Each
  suggestion now carries a reason, a priority, and where one exists a command you
  can copy and run. **Failures carry them too**: if a run fails because Ollama is
  not running, or a key is missing, or the output was cut off, the error now comes
  with the specific thing to try rather than only the message.

### Fixed

- **The local web interface now tells you what to do when a drafting run fails.**
  A run that failed because a model was missing or the provider was unreachable
  showed the raw error and nothing else, while the same failure at the command
  line printed a full set of recovery steps. Progress and results arrive over a
  stream rather than as an ordinary reply, and the recovery steps were only ever
  attached to ordinary replies. The steps a finished run suggests are now shown
  too; before, the panel kept showing the advice from the moment the run started.
- **A mistyped flag or command now points you at the flag list instead of
  suggesting a drafting run.** `draft --xyz`, an unknown command, a bad numeric
  value and a missing required flag all fell through to generic advice whose
  first suggestion was to run the tool against its own test fixture.
- **Running without choosing a model now tells you to pick a model.** The
  first error most new installs hit ("no drafting model specified") was answered
  with "Start Ollama, or point at a running one", because the error's own help
  text mentions Ollama by name. Starting a server does not choose a model; the
  advice now lists what is reachable and says how to set a default.
- **A web page reconnecting to a finished run is no longer told to change
  models.** Results are held for a few minutes after a run completes; a page
  that came back later saw "job not found" answered with model advice. It now
  says the run has expired and to start it again.
- **A model the provider does not have is no longer reported as the provider
  being down.** Asking for a model that is not pulled locally suggested starting
  Ollama, which was already running. It now suggests listing what is reachable
  and pulling the model.
- **Suggested commands now work on a downloaded copy of the tool.** Several
  suggestions named `open`, which exists only on macOS, or pointed at files that
  are only present if you built the tool from source. The download is a single
  binary and carries neither. Suggestions that name a document now give its
  address, and the ones that use the bundled example appear only when that
  example is actually present.
- **A drafted claim no longer ends in a stray `---`.** Models separate groups of
  claims with a horizontal rule, and the rule was being kept as part of the claim
  before it. It reached the exported Word document, and the critique and scoring
  passes read it as if it were claim language, so an unpredictable handful of
  claims in every run needed the same manual deletion.
- **The local web interface no longer preselects a model your machine cannot
  run.** It picked whichever installed model had the most parameters, with no
  reference to how much memory you have, so on a machine holding a large model
  library the first click could only fail. It now prefers the strongest model
  whose weights fit in memory. Where memory cannot be detected it does not filter
  at all, which is the same honest-unknown rule `models` follows.

### Changed

- **BREAKING for `--json` consumers: `next_steps` is now an array of objects, not
  an array of strings.** Each entry is
  `{action, command?, priority, reason, timing?}`. If you parse `next_steps` as
  strings, read `.action` instead. Nothing else in the envelope changed, and
  `data` and `notice` are untouched.
- **`upgrade` now tells you there is nothing to upgrade to, instead of opening a broken
  page.** The command opened a payment link that returned an access-denied error, and
  always would have. There is no paid tier to buy yet; every feature is available in the
  build you have, and no licence key changes that. The same applies to the Upgrade button
  in the local web interface, which now says the link is not configured rather than
  opening the error page.
- **The startup banner no longer names a scoring criterion that does not exist.**
  It read "scores claim drafting quality (form, support, definiteness, craft)".
  There is no `form` criterion, and the list also left out one of the criteria
  that can veto a claim outright. It now says what the tool scores against rather
  than naming a partial and partly invented list.

- **Licence tiers collapsed to a single `pro` tier.** The earlier set (`panel`,
  `export`, `batch`, `history`, and the domain packs) was a guess at which axis to
  price before any of it had been sold. Notably, `panel` is gone: the multi-model
  panel remains free, because this project's own testing shows a panel score
  separates only gross differences and cannot rank near-equal drafts, and charging
  for it would imply otherwise. Drafting and critique remain free.
- **Corrected how this documentation describes API-key handling.** It said keys
  came from the environment only. In fact the key is read from the environment
  first and otherwise from `~/.continuation-drafter/config.json`, which the binary
  creates owner-readable and which the web UI can write for you. The environment
  always wins. Never a command-line flag and never a log.

## [0.1.0] - 2026-08-05

**First release.** The binary is published to
[`patent-continuation-public`](https://github.com/Obviously-Not/patent-continuation-public);
the source stays private. All inference runs locally by default, so nothing
leaves your machine unless you point the tool at a remote endpoint yourself.

A `0.x` version on purpose. This is R&D-grade drafting input for a licensed
practitioner, not a finished product, and the limitations below are the reason
rather than boilerplate. Read them before using output for anything real.

### Added

- Go implementation of the full drafting pipeline: `draft` (spec analysis,
  target extraction, drafting, and a verify-and-revise loop), `critique`
  (cross-family critic that flags per-claim defects), `judge` (claim-quality
  rubric with spec-anchor validation), `panel` (multi-model scoring), plus
  `models`, `version`, `license`, `activate`, and `serve`.
- A local web UI on `localhost:9473`, served by the binary itself with no build
  step. HTML and CSS are embedded in the binary.
- A provider layer covering local Ollama and any OpenAI-compatible remote
  endpoint. Keys resolve from the environment or a mode-0600 config file, never
  from a flag and never from a tracked file.
- Offline Ed25519 license verification supporting multiple keys and capability
  tiers.

### Known limitations

- **R&D-grade, not filing-ready.** The pipeline drafts real, well-formed,
  spec-grounded continuation claims, but a single AI judge cannot reliably rank
  drafters. Only the multi-model panel is stable, and even it separates gross
  differences rather than near-equals. Treat every output as drafting input for
  a licensed practitioner exercising independent professional judgment, never as
  a legal determination.
- **License tiers are reported, not enforced.** Verification and reporting are
  implemented; feature gating is deferred by owner decision, so all capabilities
  are available regardless of tier. Release builds DO carry the licence public
  key, since v0.1.1 on 2026-08-18, so the deferral is a policy choice and no longer
  also a build-time gap.
- **No coverage metric.** The embedding-based coverage map from the originating
  experiment was not ported. This is a named, accepted gap.
- Model choice matters more than usual, because a specification must fit inside
  the model's context window. Read [docs/models.md](docs/models.md) before
  choosing one for real work.
