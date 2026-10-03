# Kickoff prompt (Claude Code + Superpowers)

Install Superpowers first (per its README):

```
/plugin marketplace add obra/superpowers-marketplace
/plugin install superpowers@superpowers-marketplace
```

Then paste:

```
Use the superpowers brainstorming skill. Read docs/position-capture-spec.md and
docs/research/voice-interview-agent-tooling.md. Goal: a v0 of dokument.ai I can
pilot on my own company (MacTutor) for under $50.

Constraints:
- Claude-only for all reasoning (Sonnet for interviewing, Haiku for cheap tasks).
  No third-party AI except where unavoidable.
- v0 input = typed text or bring-your-own dictation (Wispr Flow / OS dictation)
  into a text box. No audio pipeline yet.
- Stack: React + Supabase (Postgres, RLS, Auth, Edge Functions), Vercel.
- Interviewer must use tool calls to fill the step schema (spec §7.4), follow the
  probing ladder (§7.3), and do an explicit read-back with yes/no confirmation.
- Must support: org → positions → people, resumable sessions, 2+ people in the
  same role compared side by side, Markdown playbook export.

Resolve the ❓ open questions with me one at a time, then write the design and an
implementation plan.
```

## Pilot (MacTutor, ≤ $50)
| Step | What | Cost |
|---|---|---|
| Day 1 | Interviewer prompt in a Claude Project; 2–3 people dictate about one role each | $0 |
| Week 1–2 | Build v0 app | Supabase/Vercel free tiers, Claude API ~$10–25 |
| Week 2 | Two people in the same role; test comparison view | ~$5 |
| Buffer | Wispr Flow Pro, 1 month, if needed | $12 |

**Pass bar:** playbook has more steps than the owner would have written; a new hire could use it; the automation backlog contains at least one thing worth building.
