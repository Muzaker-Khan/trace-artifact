# Third-party content

## Trojan Hippo benchmark

The live records in `live/` contain e-mails, user tasks and benign test suites from the Trojan Hippo
benchmark, and its harness produced the files in `benchmark-results/`.

- Source: https://github.com/debesheedas/trojan-hippo-benchmark, commit `ebeb739dada097e06b125faa70e82e064482b64b`
- Paper: Das, Piet, Kaviani, Beurer-Kellner, Tramèr and Wagner, "Trojan Hippo: Weaponizing Agent Memory for
  Data Exfiltration", arXiv:2605.01970
- License: Apache License 2.0 (https://www.apache.org/licenses/LICENSE-2.0)

What we changed: we did not modify the benchmark's data. The 45 e-mails in `benchmark/fresh_emails.json` are
new: the benchmark's own attack generator (its optimizer's mutator, run with gpt-4o) wrote one generation of
variants of the benchmark's test e-mails, with no scoring and no selection. The attacker's rewrites in the
records (`variant` other than `original`) were made from those e-mails by fixed rules described in the
paper's supplement.

## Models

No model weights are included. Model outputs in the records come from OpenAI GPT-5-mini and GPT-5,
Anthropic Claude Haiku 5.5, Google Gemini 3 Flash, and Meta Llama 3.2 3B Instruct (run locally), with the
classifier ProtectAI deberta-v3-base-prompt-injection-v2.

## Other public datasets

Results in `results/` on AgentDojo, ATBench, CSTM and public prompt-injection text sets hold scores and
identifiers only, not the datasets' text. Obtain those datasets from their own sources.
