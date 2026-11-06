# Experiment 2: change a binary by editing its bytes

The question: can the model add a feature to an executable by editing its bytes, with one reply and no way to run it?

## Setup

- **Binaries.** `prog1` to `prog3`: small statically linked ELF64 x86-64 Linux programs with no libc.
- **Model.** `gpt-6.1-sol` through the Codex CLI (`codex exec`), reasoning effort `high`, reasoning summaries
  `detailed`. No tool reached the model, and each run's event log was audited: no run made a tool call
  (`run.json` -> `audit.tool_free: true`).
- **What the model got.** One binary per fresh conversation, one reply. The prompt (`prompt.txt`) is a short description
  of the file, the change to make, the output format, and the hex dump (`binaries/<prog>_xxd.txt`).
- **Not allowed.** Tools of any kind (no code execution, no assembler, no calculator). No feedback and no second try.
- **Scoring.** The model's edits (the last `json` block of `answer.md`) are applied to the original exactly as written.
  The result is run against a test suite and its output and exit code are compared with a reference program. Tests ran
  under qemu on Apple Silicon, not on a real x86-64 machine.

## Runs

There is one attempt per binary.

| Binary | Change requested | Run folder | Tests passed | Bytes changed | Input tokens | Output tokens | Reasoning tokens | Time |
|---|---|---|---|---|---|---|---|---|
| `prog1` | `wc`: add the longest line length | `runs/prog1` | 18/18 | 74 | 13,311 | 3,440 | 2,878 | 93 s |
| `prog2` | AES: add the `dec` command | `runs/prog2` | 28/28 | 544 | 22,609 | 9,324 | 6,445 | 201 s |
| `prog3` | Ledger: add the `transfer` command | `runs/prog3` | 31/31 | 379 | 21,798 | 6,859 | 4,660 | 140 s |

## The numbers and where they come from

| Number | Value | File |
|---|---|---|
| `prog1` score | 18/18 | `eval/prog1.json` (`passed`, `total`) |
| `prog2` score | 28/28 | `eval/prog2.json` (`passed`, `total`) |
| `prog3` score | 31/31 | `eval/prog3.json` (`passed`, `total`) |
| `prog1` bytes changed | 74 of 5,368, in 10 edits | `binaries/prog1_patch.json`, or `binaries/prog1.bin` against `binaries/prog1_patched.bin` |
| `prog2` bytes changed | 544 of 9,152, in 40 edits | `binaries/prog2_patch.json`, or `binaries/prog2.bin` against `binaries/prog2_patched.bin` |
| `prog3` bytes changed | 379 of 8,872, in 34 edits | `binaries/prog3_patch.json`, or `binaries/prog3.bin` against `binaries/prog3_patched.bin` |
| Tokens and time | See the table above | `runs/<prog>/run.json` (`audit.usage`, `wall_s`) |

## Files

```
runs/<prog>/
  prompt.txt        the full message the model got
  instructions.md   the Codex system instructions file (one line, the same for every run)
  events.jsonl      the raw Codex event stream: reasoning summaries ("reasoning") and the reply ("agent_message"),
                    plus token usage. The one "error" item is a Codex CLI start-up notice (its optional code-mode
                    host is disabled), not a model error
  answer.md         the model's reply
  run.json          the run record: model, effort, the codex command line, token usage, wall time, the tool audit
  stderr.log        Codex CLI stderr (empty)
binaries/
  <prog>.bin          the original executable, byte for byte
  <prog>_xxd.txt      the hex dump exactly as shown to the model
  <prog>_patch.json   the model's edits, copied as written from the last json block of answer.md
  <prog>_patched.bin  the original with those edits applied, the file that was scored
eval/
  <prog>.json         the test report: passed, total, and each test's expected and actual output
```

In each report, `candidate` is the scorer's path for `binaries/<prog>_patched.bin`.
