# Logs: Sparks of Binary-Space Programming

The logs of the experiments in *Sparks of Binary-Space Programming: LLMs can read and write executables byte by byte*. 
https://sgiannopoulos.github.io/blog/machine-code-programming/

There is one run per binary in each experiment, the latest one. For each run you get:

- everything we sent the model;
- everything it sent back, including its reasoning summaries;
- the evaluation behind each score;
- the binaries it read or wrote.

Each experiment folder has its own README. It says which run is which, and which file each number comes from.

| Folder | Experiment | Runs | Tools |
|---|---|---|---|
| [`exp1_understand/`](exp1_understand) | Experiment 1: what does this binary do? | `prog1` to `prog5` | none |
| [`exp2_patch/`](exp2_patch) | Experiment 2: change a binary by editing its bytes | `prog1` to `prog3` | none |
| [`exp3_construct/`](exp3_construct) | Experiment 3: build a program from its description | `prog1` to `prog3` | none |

That makes 11 runs. Earlier attempts and trial runs of earlier prompt versions or other models are left out.

## How the model was run

The model is OpenAI's `gpt-6.1-sol`, through Codex CLI 0.166.0.

**One turn.** Every run is one call of the Codex CLI, `codex exec`, in a fresh conversation. The prompt goes in on
stdin. The reasoning effort is `high` for every run and is recorded in each `run.json`.

**No tools.** Every built-in Codex capability that gives the model a tool is switched off. The configuration is in
[`harness/codex_config.toml`](harness/codex_config.toml), and the system instructions are one neutral line,
[`harness/instructions.md`](harness/instructions.md). Configuration alone can't empty the tool list, so a local relay
between Codex and the API also removed all tools from every request. The relay is the `tool_stripper.py` named in the
configuration's comments and the `bx_relay` provider in each `run.json`. Its code isn't in this repository; each run's
`audit.wire` records what it saw and removed.

**Every turn was audited.** We checked each turn's event stream. A turn counts as tool-free when the stream holds only
the model's messages, its reasoning summaries, and Codex's notice that code mode is disabled. The result is the
`audit.tool_free` field of the turn's `run.json`. `audit.lockdown_ok` also requires that the prompt on disk matches what
the relay delivered. All 11 runs in this release pass both.

## Files

Not every file appears in every run.

| File | What it is |
|---|---|
| `prompt.txt` | What we sent. |
| `instructions.md` | The Codex system instructions file. |
| `events.jsonl` | The raw Codex event stream of the turn: the model's messages and reasoning summaries, plus token usage. |
| `answer.md` | The model's reply. |
| `run.json` | The turn's record: model, effort, tokens, wall time, and the audit. |
| `stderr.log` | Codex's stderr. |
| `eval/` | The scoring: each test's expected and actual output. |
| `binaries/` | The binaries the model read or wrote. |

**Placeholders.** In `run.json`, `<repo>` stands for our experiment repository.

Codex thread ids are replaced by `id-` and 8 hex digits, in `events.jsonl` and `run.json`. Everything else in those files
is as logged.

## Not included

- **The `prog4` executable.** It appears to be real malware. Its hex dump is in its prompt.
- **`prog1`'s two earlier attempts in Experiment 1.**
- **The harness code and the scoring code,** including the relay and the test suite's source.
- **Scores for Experiment 1.** Its answers are not formally scored.

## Licenses

See [`LICENSE`](LICENSE).

| Content | License |
|---|---|
| Logs and data | CC BY 4.0 |
| `prog5` binary and hex dump | The licenses of `ministat` and musl |
| `prog4` hex dump | No license granted |
