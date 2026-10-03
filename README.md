# dokument.ai

An AI interviewer that makes every position in a business **transferable, delegable and automatable**.

1. The owner defines the positions (or picks an industry template).
2. Each person dictates or types how they do their job. A Claude interviewer keeps probing (why? anything else? are you sure? how often?) until the process is fully captured, then reads it back for confirmation.
3. People in the same role are compared step by step and linked to results, to find the best way of doing the job.
4. Every step is sorted into **Delete / Code / Agent / Delegate / Human**, producing playbooks, training gaps and an automation backlog.

## Status
Pre-build. Pilot target: MacTutor's own roles, under $50.

## Docs
| File | What it is |
|---|---|
| [docs/position-capture-spec.md](docs/position-capture-spec.md) | Product spec v0.1: data model, interviewer design, step schema, variance and outcome logic, MVP scope |
| [docs/kickoff-prompt.md](docs/kickoff-prompt.md) | First prompt for the Claude Code + Superpowers brainstorming session |
| [docs/research/voice-interview-agent-tooling.md](docs/research/voice-interview-agent-tooling.md) | Voice/dictation stack, interview methods, prompt libraries, build vs buy, costs |
| [docs/research/market-research.md](docs/research/market-research.md) | Competitors, demand evidence, pricing benchmarks |
| [docs/research/prompt-libraries.md](docs/research/prompt-libraries.md) | prompts.chat, interview skills to adapt, Anthropic Interviewer test dataset |

## Principles
- **Claude-only reasoning.** Sonnet for interviewing, Haiku for cheap tasks. Third-party services only where unavoidable (speech-to-text, text-to-speech), under zero-retention terms.
- **v0 input is text or bring-your-own dictation** (Wispr Flow, OS dictation). No audio pipeline yet.
- **Stack:** React + Supabase (Postgres, RLS, Auth, Edge Functions) + Vercel.
- **Consent first:** Florida requires all-party consent for recordings.
