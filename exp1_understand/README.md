# Experiment 1: what does this binary do?

The question: can the model explain what an executable does from its hex dump alone?

## Setup

- **Binaries.** Five programs: `prog1` to `prog3` are small statically linked Linux programs, `prog4` is a small Windows
  executable, and `prog5` is a modified build of FreeBSD's `ministat`, compiled with `-O3`, statically linked against
  musl and stripped of the program's own text.
- **Model.** `gpt-6.1-sol` through the Codex CLI (`codex exec`), reasoning effort `high`, reasoning summaries
  `detailed`. No tool reached the model, and each run's event log was audited: no run made a tool call
  (`run.json` -> `audit.tool_free: true`).
- **What the model got.** One binary per fresh conversation, one reply. Every prompt is the same two short paragraphs
  followed by the binary's hex dump (`binaries/<prog>_xxd.txt`).
- **Not allowed.** Tools of any kind (no code execution, no calculator).
- **Scoring.** None. The answers in this release are not formally scored.

## Runs

| Binary | What it is | Run folder | What the model reported | Input tokens | Output tokens | Reasoning tokens | Time |
|---|---|---|---|---|---|---|---|
| `prog1` | Line, word and byte counter (ELF x86-64) | `runs/prog1` | A Linux x86-64 word counter, like `wc -l -w -c` | 12,763 | 3,195 | 1,547 | 88 s |
| `prog2` | AES-128/256 ECB encryption (ELF x86-64) | `runs/prog2` | A Linux x86-64 AES-128/256 ECB encryption tool, encryption only | 21,779 | 7,010 | 4,660 | 197 s |
| `prog3` | Bank ledger interpreter (ELF x86-64) | `runs/prog3` | An in-memory bank-account command interpreter for Linux x86-64 | 21,084 | 7,095 | 5,178 | 205 s |
| `prog4` | Small Windows executable (PE x86-64) | `runs/prog4` | A 64-bit Windows loader that downloads machine code and runs it in memory | 11,289 | 7,840 | 5,178 | 211 s |
| `prog5` | Modified `ministat` (ELF x86-64) | `runs/prog5` | A statistical comparison and ASCII-plotting tool, closely matching `ministat` | 162,618 | 6,705 | 4,142 | 216 s |

`prog1` was run three times with the same prompt. This release has the third and latest attempt; the first two are left
out. The `-o` path in its `run.json` still says `a3`.

**`prog4` appears to be a real malware downloader.** It was only ever read as hex and was never run. Its executable is
not in `binaries/`, but its `prompt.txt` and `binaries/prog4_xxd.txt` hold the complete hex dump, and its `answer.md`
quotes the network address the model read from the bytes. Never run it.

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
  <prog>.bin        the executable, byte for byte (not for prog4)
  <prog>_xxd.txt    the hex dump exactly as shown to the model
```
