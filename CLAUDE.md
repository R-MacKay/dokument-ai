# CLAUDE.md — dokument.ai

Read `docs/position-capture-spec.md` before designing or building anything. Research context is in `docs/research/`.

## Non-negotiables
- All reasoning uses Claude models (Sonnet for interviews and analysis, Haiku for cheap high-volume tasks). Do not add other LLM providers.
- Third-party services only where Claude cannot do the job (speech-to-text, text-to-speech, optionally embeddings — prefer `gte-small` in Supabase over an external embedding API). Flag any new third party for approval.
- v0: text or bring-your-own dictation only. No audio capture or transcription pipeline.
- The interviewer writes structured data through tool calls (`record_step`, `flag_exception`, `record_policy`, `get_coverage`, `mark_complete`) into Supabase. Free text alone is not acceptable output.
- Every interview ends with an explicit read-back that the person confirms with a clear yes.
- Multi-tenant on `org_id` with RLS on every table.
- Staff never see other people's names in comparisons, gaps or the automation backlog.
- Never store recordings or transcripts without a logged consent event.

## Stack
React, Supabase (Postgres, RLS, Auth, Edge Functions, Storage), Vercel. Node for any server-side agent loop.
