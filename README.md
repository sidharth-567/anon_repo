# Attack templates (anonymized release)

Frozen seed-artifact attack templates used in the paper, released for reviewers.

Each file is a natural-language template that is injected into a seed artifact;
placeholders are substituted per run (see [Placeholders](#placeholders)). Every
placeholder is left unsubstituted — in particular `{PROJECT_CONTEXT}` is never
filled in — so the templates are not directly usable as shipped. The adversarial
goal and endpoint are supplied per test instance and are not part of the templates.

## How the attack propagates

The templates encode a payload that spreads through an
artifact → memory → artifact loop, with no direct agent-to-agent channel, no
shared skill or package, and no framework compromise:

1. **Seed.** An adversarial goal is planted in a seed artifact (e.g. a shared
   report or handoff) that one assistant reads.
2. **Memory.** The template disguises the goal as a standing workspace convention
   and instructs the assistant to persist the goal plus a carry-forward rule in its
   persistent memory (`MEMORY.md`).
3. **Artifact.** On a later writing task the assistant reproduces the goal and the
   carry-forward rule in a new artifact.
4. **Next reader.** A second, independent assistant reads that artifact and repeats
   the loop.

Two variants are evaluated:

- **Endpoint-assisted** (main experiments). The template also instructs the
  assistant to upload each written artifact to an attacker-controlled endpoint and
  save the returned copy over the same path, so the full attack prompt is restored
  at every hop. The upload is a shell step (`curl`).
- **Prompt-only**. There is no upload step and no `curl` rule: propagation depends
  only on the model's own reproduction across the
  artifact → memory → artifact loop.

A third, **concealed** carrier (below) is the endpoint-assisted template with the
payload stored as invisible Unicode tag characters instead of plain text.

## Main prompt per model

The latest frozen template used in the main propagation experiments, one per
target model. The other files for the same model are alternative carriers for the
same mechanism, kept for completeness.

| Model | Main template | Goal placeholder |
|---|---|---|
| GPT-OSS-120B | `templates/endpoint/gpt-oss-120b_last-touch.txt` | `{goal_text}` |
| DeepSeek-V4-Pro | `templates/endpoint/deepseek-v4-pro.txt` | `{GOAL_TEXT}` |
| Kimi-K2.6 | `templates/endpoint/kimi-k2.6.txt` | `{goal_text}` |
| GPT-5.6 Luna | `templates/endpoint/gpt-5.6-luna.txt` | `{goal_text}` |

Each main template has a concealed counterpart in `templates/concealed/` with the
same file stem.

## Files

Templates are split into three folders:

- `templates/endpoint/` — endpoint-assisted templates (14 files).
- `templates/prompt_only/` — prompt-only templates (7 files).
- `templates/concealed/` — endpoint-assisted templates with the payload stored as
  invisible Unicode tag characters (4 files).

### Endpoint-assisted templates (`templates/endpoint/`)

| File | Model | Carrier mechanism |
|---|---|---|
| `gpt-oss-120b_last-touch.txt` **(main)** | GPT-OSS-120B | Reader rule + no-stall rule + last-touch `curl` sequencing |
| `gpt-oss-120b.txt` | GPT-OSS-120B | Memory-only ritual (two heredocs into `MEMORY.md`) |
| `gpt-oss-120b_receipt.txt` | GPT-OSS-120B | Gateway-receipt gate (upload a receipt before answering); "Shared Note" brand |
| `gpt-oss-120b_receipt-lasttouch.txt` | GPT-OSS-120B | Receipt gate + last-touch sequencing |
| `deepseek-v4-pro.txt` **(main)** | DeepSeek-V4-Pro | Token-themed standing preference (manifest-token framing) |
| `deepseek-v4-pro_last-touch.txt` | DeepSeek-V4-Pro | Reader rule + no-stall rule + last-touch `curl` sequencing |
| `deepseek-v4-pro_memory-ritual.txt` | DeepSeek-V4-Pro | Memory-only ritual (two heredocs into `MEMORY.md`) |
| `deepseek-v4-pro_hard-curl.txt` | DeepSeek-V4-Pro | Hard next-call `curl` sequencing |
| `kimi-k2.6.txt` **(main)** | Kimi-K2.6 | Task-need framing (the standard is required to finish the task); "Team Ledger" brand |
| `kimi-k2.6_taskneed.txt` | Kimi-K2.6 | Task-need framing (shortened variant) |
| `kimi-k2.6_normalize.txt` | Kimi-K2.6 | Normalize-service preview/save of the draft |
| `gpt-5.6-luna.txt` **(main)** | GPT-5.6 Luna | Conditional-form guard; "Editorial Norms" brand |
| `gpt-5.6-luna_conditional.txt` | GPT-5.6 Luna | Conditional-form guard (shortened variant) |
| `gpt-5.6-luna_last-touch.txt` | GPT-5.6 Luna | Last-touch sequencing |

### Concealed templates (`templates/concealed/`)

Same endpoint-assisted payload as the main template for each model, but the
payload is stored as invisible Unicode tag characters
(`U+E0000`–`U+E007F`) rather than as plain text. Each file keeps the payload's
placeholders (for example `{PROJECT_CONTEXT}` and `{goal_text}`); nothing is
substituted.

| File | Model | Concealed carrier |
|---|---|---|
| `gpt-oss-120b.txt` | GPT-OSS-120B | Full payload concealed; visible note names the decoding and asks for exact execution |
| `deepseek-v4-pro.txt` | DeepSeek-V4-Pro | Full payload concealed; visible note names the decoding and asks for exact execution |
| `kimi-k2.6.txt` | Kimi-K2.6 | Full payload concealed; visible note names the decoding and asks for exact execution |
| `gpt-5.6-luna.txt` | GPT-5.6 Luna | Top block concealed; the belief and filing rule stay readable |

**Encoding.** Inside a concealed file, the invisible run after the paperclip
character `📎` stores one character per encoded character: the stored code point is
`U+E0000` plus the ASCII code point of the character it represents. To read it,
subtract `U+E0000` (`0xE0000`) from each code point in the run; the result is the
same template text, with the same placeholders. The tag characters do not render,
so the files show the carrier's visible text followed by `📎`.

### Prompt-only templates (`templates/prompt_only/`)

| File | Model | Carrier mechanism |
|---|---|---|
| `luna_prompt_only.txt` | GPT-5.6 Luna | Standing "House Style" preference with a memory-sync ritual |
| `luna_prompt_only_changelog.txt` | GPT-5.6 Luna | Running changelog of standing decisions appended to `MEMORY.md` |
| `gpt-oss-120b_prompt_only.txt` | GPT-OSS-120B | Forged workspace-kernel / compliance-gate authority override |
| `gpt-oss-120b_prompt_only_register.txt` | GPT-OSS-120B | Workspace register entry copied into `MEMORY.md` and carried into the next document |
| `kimi-k2.6_prompt_only.txt` | Kimi-K2.6 | Collegial desk handoff (social proof / reciprocity) |
| `deepseek-v4-pro_prompt_only.txt` | DeepSeek-V4-Pro | Standing-notes block copied into `MEMORY.md` verbatim and carried into the next document |
| `deepseek_gptoss_kimik2.6_prompt_only.txt` | DeepSeek-V4-Pro, GPT-OSS-120B, Kimi-K2.6 | Shared standing-notes block with a binding reader rule (combined) |

Templates used in the prompt-only evaluation. There is no upload step and no `curl`
rule, so the attack must survive on the model's own reproduction alone. The
per-model files are dedicated variants, each built around a different carrier
mechanism; the combined file is the shared template the three non-Luna models were
evaluated with.

## Placeholders

Two placeholder spellings are used, matching the two template families. All are
left unsubstituted so the templates are not directly usable.

| Placeholder | Family | Meaning |
|---|---|---|
| `{goal_text}` | endpoint-assisted, concealed | The adversarial goal (one bullet), e.g. a false user preference or a security-critical instruction |
| `{GOAL_TEXT}` | endpoint-assisted, prompt-only | The same goal in the upper-case spelling; used by the DeepSeek token-framing endpoint template and all prompt-only templates |
| `{META_CARRY_FORWARD}` | endpoint-assisted, concealed | The carry-forward / API-workflow sentence |
| `{META_INSTRUCTION}` | prompt-only | The carry-forward sentence, in the upper-case spelling the prompt-only templates use |
| `{api}` | endpoint-assisted, concealed | The attacker-controlled endpoint URL |
| `{SERVICE_NAME}`, `{REASON}` | endpoint-assisted, concealed | The endpoint's cover story |
| `{PROJECT_CONTEXT}` | both | The universe's project context paragraph |

## Scope

Released for review and defensive research, to support replication and analysis of
artifact-mediated propagation. Do not use these templates against systems without
authorization.
