# Experiment 3: build a program from its description

The question: can the model write a complete executable byte by byte, starting from an empty file, with one reply and
no way to run it?

## Setup

- **Targets.** The same final behavior Experiment 2 asks for, so changing a program can be compared with building it.
  The target format is a static ELF64 x86-64 Linux executable with no libc.
- **Model.** `gpt-6.1-sol` through the Codex CLI (`codex exec`), reasoning effort `high`, reasoning summaries
  `detailed`. No tool reached the model, and each run's event log was audited: no run made a tool call
  (`run.json` -> `audit.tool_free: true`).
- **What the model got.** One specification per fresh conversation, one reply. The prompt (`prompt.txt`) holds the target
  format, the behavior and the output format. It holds no hex dump.
- **Not allowed.** Tools of any kind (no compiler, no assembler, no calculator). No feedback and no second try.
- **Scoring.** The bytes (the last `hex` block of `answer.md`) are written to a file exactly as given. The file is run
  against the same test suite as Experiment 2. Tests ran under qemu on Apple Silicon, not on a real x86-64 machine.

## Runs

There is one attempt per target.

| Target | Behavior | Run folder | Tests passed | File size | Input tokens | Output tokens | Reasoning tokens | Time |
|---|---|---|---|---|---|---|---|---|
| `prog1` | Counter with the longest line length | `runs/prog1` | 18/18 | 391 B | 570 | 6,035 | 5,178 | 124 s |
| `prog2` | AES encryption and decryption | `runs/prog2` | 28/28 | 4,144 B | 892 | 22,812 | 14,254 | 474 s |
| `prog3` | Ledger with `transfer` | `runs/prog3` | 31/31 | 3,584 B | 801 | 25,069 | 17,698 | 504 s |

## The numbers and where they come from

| Number | Value | File |
|---|---|---|
| `prog1` score | 18/18 | `eval/prog1.json` (`passed`, `total`) |
| `prog2` score | 28/28 | `eval/prog2.json` (`passed`, `total`) |
| `prog3` score | 31/31 | `eval/prog3.json` (`passed`, `total`) |
| `prog1` file size | 391 B | `binaries/prog1_constructed.bin` |
| `prog2` file size | 4,144 B | `binaries/prog2_constructed.bin` |
| `prog3` file size | 3,584 B | `binaries/prog3_constructed.bin` |
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
  <prog>_constructed.bin  the bytes of the last hex block of answer.md, the file that was scored
eval/
  <prog>.json             the test report: passed, total, and each test's expected and actual output
```

In each report, `candidate` is the scorer's path for `binaries/<prog>_constructed.bin`.
