# Prompt libraries, skills & datasets for the interviewer

Researched 2026-10-03. Our interviewer prompt is core IP: adapt patterns from these, don't copy wholesale.

## Libraries

| Source | Link | License | Use for dokument.ai |
|---|---|---|---|
| **prompts.chat** (formerly Awesome ChatGPT Prompts, ~172K stars) | https://github.com/f/prompts.chat | Prompts CC0, code MIT | General prompt library; MCP server, CLI (`npx prompts.chat`), Claude Code plugin; self-hostable (`npx prompts.chat new my-prompt-library`). Interviewer prompts are generic — low value for the core product. |
| prompts.chat dataset | https://huggingface.co/datasets/fka/prompts.chat | CC0 | Mine role / interviewer prompts as starting material |

## Interview-style skills (high value — adapt these)

| Skill | Pattern to take |
|---|---|
| obra/superpowers — brainstorming | One question per message; multiple choice where possible; approve section by section |
| addyosmani — interview-me | Stop when the interviewer can predict the next 3 answers; 5–8 line read-back incl. out-of-scope; explicit "yes" required |
| mattpocock — grill-me | Walk every branch of the decision tree; offer a recommended answer |
| curiositech/port-daddy — cdm-interviewer | Critical Decision Method; record rules as "When [cue], because [reason], do [action], unless [exception]" → maps to `steps.decision_rule` |
| OpenAI Realtime prompting guide | Labeled prompt sections; phases with exit criteria as a state machine; varied phrasing (useful for v1.1 live voice) |

## Test data

| Dataset | Link | License | Use |
|---|---|---|---|
| Anthropic Interviewer transcripts (1,250 professionals) | https://huggingface.co/datasets/Anthropic/AnthropicInterviewer | CC-BY | Benchmark follow-up quality: replay opening answers through our interviewer and compare probe depth |

## Test plan (pilot)
1. Pick 20 transcripts with process-heavy answers.
2. Feed each participant's first answer to our interviewer; capture its next 3 probes.
3. Score probes: specific vs generic, schema field targeted, leading vs neutral.
4. Target: ≥80% of probes target an empty step-schema field (spec §7.4) and none are leading.
