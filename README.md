# TRACE: data and results artifact

Data, benchmark files, settings and raw results for the paper
**"Attacks That Outlast the Conversation: Detecting and Responding to Multi-Stage Threats to AI Agents"**
(under review at IEEE Transactions on Dependable and Secure Computing).

TRACE is a runtime defense that sits outside an AI agent: a tool gateway, a small wrapper that reports
memory reads and writes, a correlator that follows untrusted content across sessions, and a gate that
holds each outgoing send until everything before it has been judged. The paper evaluates it against
memory-poisoning attacks that make an agent send the user's data to an attacker.

This repository holds everything needed to **check every number in the paper** against the raw records:
the fresh attack e-mails, the record of every live trial (block off and block on), every replayed
baseline and ablation, the trust-attack replays, the detector settings and thresholds, the model
versions, and the spending of each paid run.

**What is not here.** TRACE's source code is not released. [`ARCHITECTURE.md`](ARCHITECTURE.md)
describes the design at the level of the paper (Section 4), and [`settings/settings.json`](settings/settings.json)
gives every model, version and threshold of Table 3.

## Layout

| Folder | What it holds |
|---|---|
| `paper-numbers/` | `facts.json` and `macros.tex`: every number printed in the paper, as the paper's build computed it from the files below |
| `results/` | Result summaries behind each table and figure (detectors, published transcripts, benign stream, live tests, timings) |
| `results/ablations/` | RQ5 baselines and ablations (`full`, `rest`, `quick`, `fast`, `llm_only`, and `fresh_*` for the fresh trials) and RQ7 trust attacks (`threat_tests.json`), one row per trial and configuration. The `fresh_*` files cover all three model families; the paper's fresh column uses their rows with `record_pool` `fresh` or `final` and `variant = original`. A replay counts every send to the attacker's address (`theft_at`), so it finds 116 fresh thefts where the benchmark's own check, which also asks whether private data arrived, counts 110 |
| `live/main-test/` | The main live test (RQ4): GPT-5-mini with three memory systems, development and held-out e-mails, block off and on |
| `live/revision/` | The revision's live test (RQ6): fresh e-mails on GPT-5-mini, Claude Haiku 5.5 and Gemini 3 Flash, the attacker's rewrites, and the benign suites |
| `live/revision/set-aside/` | Not used in the paper: five Gemini trials run at Gemini's default thinking level before its billing account was suspended, and one Gemini benign test on Mem0 from a run stopped to stay within budget |
| `benchmark/fresh_emails.json` | The 45 fresh attack e-mails (SHA-256 `c4501d7b37395773e74759ff186cbe84653a6a1fb740148f7511ec5f7b0a3b7d`) |
| `settings/` | `settings.json` (Table 3: models, versions, thresholds, engine hashes) and `prices.json` (the prices used for the spending caps, in US dollars per million tokens, read on 2026-11-04 and for Gemini 3 Flash and Claude Haiku's long-prompt rate on 2026-11-15; cached-input and batch discounts were not applied, so the recorded costs are upper bounds) |
| `figures/architecture.pdf`, `.png` | Fig. 1 of the paper |
| `SHA256SUMS` | A checksum of every file |

Inside each `live/*` folder:

| Path | Format |
|---|---|
| `trials/<trial>--<arm>.json` | One record per trial and arm (see below) |
| `events/<trial>--<arm>.jsonl.gz` | Every event the monitor received and every decision it made, in order, one JSON object per line |
| `benchmark-results/.../*.json.gz` | The benchmark's own result file for the trial, unchanged |
| `verdicts/` | Each detector's verdict per scanned text, keyed by a hash of the text; replays reuse these |
| `spending/ledger.jsonl.gz` | One line per paid model call: model, tokens, cost in US dollars (no keys, no prompts) |
| `runs.jsonl` | When each job ran and with which settings |

## Trial records

The arms are `observe-cascade` (TRACE watches but never blocks: **block off**) and `block-cascade` (TRACE holds
sends and quarantines notes: **block on**). Both arms of a trial share the trial name, so they pair up. Main fields:

| Field | Meaning |
|---|---|
| `kind` | `attack` or `honest` (a benign test) |
| `pool` | `develop`, `held-out`, `fresh` (the fresh e-mails) or `final` (benign tests on the final engine) |
| `model`, `backend`, `variant` | Agent model; memory system (`rag`, `context`, `mem0`); `original` or the attacker's rewrite |
| `email`, `test`, `topic`, `rollout` | Which attack e-mail (hash prefix), which benchmark test, topic and repeat |
| `benchmark_says_stolen` | The benchmark's own check: did the data reach the attacker? |
| `sent_to_attacker`, `asked_to_send_to_attacker` | Whether the agent sent, or asked to send, anything to the attacker's address |
| `user_steps`, `user_steps_passed` | Benign tests: the user's steps and how many passed the benchmark's check |
| `denied_sends_to_attacker`, `denied_other_sends` | Sends the gate denied |
| `decisions`, `notes`, `recalls`, `monitor` | The gate's decision on each call, each stored note, what each recall found and returned, and the monitor's alerts |
| `longest_wait_s`, `seconds` | Longest time a send was held; trial duration |

## Checking a result by hand

Example: RQ6, thefts with the block off and on. For each fresh trial in `live/revision/trials/` with
`kind = attack`, `pool = fresh`, `variant = original`, pair the `observe-cascade` and `block-cascade` records
by `trial`, and count `benchmark_says_stolen` in each arm. Intervals in the paper resample whole e-mails
(2,000 bootstrap resamples, seed 7), or are Wilson 95% intervals for a single share.

Example: RQ5. Each row of `results/ablations/*.json` is one block-off trial replayed under one
configuration: `theft_at` is the index of the first send to the attacker (or null), `first` the index of
the gate's first action, and `stopped` is true when the gate acted before the theft.

## Models and runs

The agent models were GPT-5-mini and GPT-5 (OpenAI), Claude Haiku 5.5 (Anthropic, extended thinking off)
and Gemini 3 Flash (Google, thinking level `minimal`), each with three memory systems from the Trojan Hippo
benchmark harness at commit `ebeb739dada0`. TRACE's confirming model was Llama 3.2 3B Instruct, run
locally. The paid runs were capped per provider; `spending/ledger.jsonl.gz` has every call's cost.

## License and attribution

Our data and documentation are released under CC BY 4.0 ([`LICENSE`](LICENSE)). The records contain
e-mails, tasks and benign suites from the Trojan Hippo benchmark (Apache License 2.0), and the fresh
e-mails were written by that benchmark's own attack generator; see [`NOTICE.md`](NOTICE.md).

Please cite the paper above when you use this data. Questions: open an issue in this repository.
