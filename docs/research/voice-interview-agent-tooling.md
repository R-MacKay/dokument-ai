# Build one Claude brain, two voice doors

**Verdict: build it yourself on a cascaded pipeline with Claude as the only brain. Keep all interview state in Supabase. Use off-the-shelf parts for audio only.** Anthropic has **no realtime or audio-input API**. Claude voice mode exists only in the consumer apps ([Claude pricing](https://platform.claude.com/docs/en/about-claude/pricing); [Claude docs index](https://platform.claude.com/llms.txt)). So "Claude everywhere" means speech-to-text → Claude with tools → text-to-speech, not a speech-to-speech model. That trade costs little here. A cascade adds **600–1,100 ms of latency versus about 300 ms** for OpenAI Realtime ([Forasoft](https://www.forasoft.com/article/openai-realtime-api-pricing), third-party), and interview turns are slow and thoughtful. The recommended stack:

- **Live mode:** LiveKit Agents (Node, Apache-2.0) with Deepgram streaming STT and Claude Sonnet 5.5.
- **Async dictation mode:** an Expo or PWA recorder uploads to Supabase Storage, Deepgram Nova-3 or AssemblyAI batch STT transcribes it with keyterms, and the **same Claude agent loop and tool contract** processes it.

Estimated cost of goods (COGS) is **about $1.00–1.30 per 30-minute live interview** and **about $0.40–0.70 per 30-minute async round**. That works out to **under $5 per employee** across 2–3 sessions. It is a rounding error against a $3.5K–7.5K diagnostic. The hard part is not voice. **Follow-up quality is the documented weakness of every AI interviewer studied** ([Wuttke et al.](https://arxiv.org/html/2410.01824v1); [Lang & Eskenazi](https://arxiv.org/abs/2502.20140)). Put the engineering effort into a coverage-driven probe loop, explicit read-back and cross-person contradiction checks, not into lower latency. Before writing code, run a 2-week v0 pilot with 5–8 staff in one role.

---

## The recommended stack fits React/Node/Supabase with one new dependency

| Layer | Pick | Fallback | Why |
|---|---|---|---|
| Brain (both modes) | **Claude Sonnet 5.5** ($2 in / $10 out / $0.20 cache read per MTok) | Haiku 4.5 for quick probes; Opus 5.5 for post-session synthesis | One model, one prompt, one tool contract across modes. Batch is 50% off for after-the-fact synthesis ([Claude pricing](https://platform.claude.com/docs/en/about-claude/pricing)) |
| Live voice transport | **LiveKit Agents (Node/AgentsJS)** + LiveKit Cloud | Pipecat (Python, BSD-2), self-hosted SmallWebRTC | Supports Anthropic natively, MCP, semantic turn detection, and RN/iOS/Android/web SDKs ([livekit/agents](https://github.com/livekit/agents)). Costs $0.01/agent-min, with 1,000 min/mo free ([LiveKit pricing](https://livekit.com/pricing)) |
| Live STT | **Deepgram Flux** (turn-aware, $0.0065/min promo) or Nova-3 streaming ($0.0048/min + $0.0013 keyterms) | AssemblyAI Universal-Streaming ($0.15/hr) | Keyterm boosting for "ServiceTitan", "PEX", "backflow" ([Deepgram](https://deepgram.com/pricing)) |
| Async STT | **Deepgram Nova-3 batch** ($0.0043/min) or **AssemblyAI Universal-3.5 Pro** ($0.21/hr, keyterms included) | Gemini 3.5 Transcribe (~$0.003/min) | Deterministic vocabulary boost. LLM-based transcription can hallucinate on silence ([AssemblyAI](https://www.assemblyai.com/pricing); [Gemini](https://ai.google.dev/gemini-api/docs/pricing)) |
| TTS | LiveKit Inference TTS ($0.009–0.03/min) or ElevenLabs Flash ($0.04/1K chars) | Deepgram Aura, Cartesia (not priced in research) | Commodity layer; stream it sentence by sentence ([LiveKit](https://livekit.com/pricing); [ElevenLabs](https://elevenlabs.io/pricing/api)) |
| Mobile capture | **Expo `expo-audio`** with `enableBackgroundRecording`, `directory: 'document'` | PWA MediaRecorder (unreliable when the iOS screen is locked; not verified) | Records in the background from a truck, stores files durably offline, and shows a persistent Android notification ([Expo audio](https://docs.expo.dev/versions/latest/sdk/audio/)) |
| Storage/queue | Supabase Storage (resumable TUS uploads) → Edge Function → STT webhook → agent worker | — | Already in the stack |
| Interview scaffolding | **Learn from OpenInterviewer** (MIT, Next.js/TS, Claude support, synthesis with provenance) | — | Closest open-source match to the stack, but text-only ([GitHub](https://github.com/linxule/openinterviewer)) |

**Rejected as the core:**

| Option | Reason |
|---|---|
| OpenAI Realtime / Gemini Live as the brain | Replaces Claude, so live and async would run on different brains. Sessions cap at 60 min (OpenAI) and 15 min audio / 10 min WebSocket (Gemini) ([OpenAI guide](https://developers.openai.com/api/docs/guides/realtime-conversations); [Gemini session docs](https://ai.google.dev/gemini-api/docs/live-session)) |
| Vapi / Retell / Bland | Phone-centric, with platform fees of $0.05–0.14/min. Bland limits web chat and custom LLM to its Enterprise tier ([Vapi](https://vapi.ai/pricing); [Retell](https://www.retellai.com/pricing); [Bland](https://www.bland.ai/pricing)) |
| ElevenLabs Agents | Its custom LLM must be **OpenAI-compatible**, so Claude needs a proxy. Costs $0.08/min plus the LLM ([ElevenLabs custom LLM](https://elevenlabs.io/docs/agents-platform/customization/llm/custom-llm)) |

---

## Live and async are two I/O adapters around one interview service

```
                 ┌──────────────── Interview Service (Node worker) ────────────────┐
 LIVE            │                                                                  │
 RN/web ─WebRTC─▶│ LiveKit Agent ─ STT stream ─▶ ┐                                  │
                 │                                │   Claude agent loop             │
 ASYNC           │                                ├─▶ (same system prompt + tools)  │──▶ Supabase
 Expo rec ─TUS──▶│ Storage → Edge Fn → batch STT ▶┘        │                        │   sessions, turns,
                 │                                          ▼                        │   steps, probes,
                 │                         tool calls (non-blocking DB writes)       │   coverage, consent
                 │                                          │                        │
                 │  LIVE: stream text→TTS per sentence   ASYNC: 1–3 next questions   │
                 │                                       as text + optional TTS clip │
                 └──────────────────────────────────────────────────────────────────┘
```

**Design rules:**

1. **Supabase is the memory, the vendor session is not.** On every new connection (resume, a dropped call, the next async round), rebuild context from Supabase: the position brief, a recap of the procedures captured so far, the coverage gaps, and the top open probes. This survives vendor caps and lets a person switch modes mid-interview, for example by starting live and finishing with voice notes from the truck.
2. **Use one tool contract.** It unifies the spec's §14 names with the research. Schemas map 1:1 to §6/§7.4:

| Tool | Writes to | Purpose |
|---|---|---|
| `upsert_procedure{name, trigger, frequency, est_minutes}` | `procedures` | Phase 2 rhythm inventory |
| `upsert_step{procedure_id, seq, action, trigger, tool, input, output, handoff_to, decision_rule, est_minutes, wait_time, judgment_level, why, source: policy\|practice, confidence, evidence}` | `steps` | §7.4 schema plus `source` (new) |
| `flag_exception{step_id, condition, path, frequency_pct, loop_to_step_id}` | `steps` (exception_of / loop_to) | Branches and rework |
| `record_policy{rule, scope, written: bool, override_by}` | `policies` | Phase 5 |
| `record_heuristic{when, because, do, unless, novice_error}` | **new** `heuristics` | CDM-format tacit rules ([cdm-interviewer](https://skills.cat/skills/curiositech/port-daddy/cdm-interviewer)) |
| `record_tool{name, category, used_for}` | `tools` | Systems inventory |
| `mark_gap{entity_id, slot, reason: unknown\|declined\|deferred}` | **new** `open_probes` | Queues a slot for later instead of probing past fatigue |
| `get_coverage{procedure_id?}` | read | Returns empty slots per step; **the agent picks its next question from the result** |
| `readback_ready{procedure_id}` → `confirm_readback{status: yes\|corrected}` | `procedures.validated_at` | Explicit-yes gate |
| `end_segment{reason}` | `interview_sessions` | Stop or pause |

3. **Live-mode latency:** writes to Supabase fire and forget. Speak the next question right away. Use Haiku for "anything else?" probes and Sonnet for synthesis turns. Use push-to-talk ("hold to speak", as Ontora does ([DEV.to reconstruction](https://dev.to/sanoojcools/how-ontoras-interview-to-map-architecture-works-a-technical-reconstruction-3c8m))) or a long endpoint threshold. People pause mid-procedure, and aggressive voice-activity detection (VAD) cuts them off. OpenAI's guide documents turning auto-response off for app-controlled turns ([OpenAI guide](https://developers.openai.com/api/docs/guides/realtime-conversations)), and LiveKit ships semantic turn detection.
4. **Per-position keyterm list** (owner-supplied tools, jargon and customer names): send it to STT keyterms *and* the Claude system prompt. Claude then normalizes jargon against the glossary.
5. **Raw audio is retained** for audit and consent evidence, with a retention policy for each org.

---

## Voice vendors cost $0.13–$4.00 per 30 minutes; the cascade lands near $1

**Cost per 30-minute interview** (estimates from list prices; about 60 Claude turns; context grows to about 20k tokens and is cached; about 300 output tokens per turn; the AI speaks about 7–8k characters):

| Option | Brain | Est. $/30 min | Notes |
|---|---|---|---|
| **A. LiveKit + Deepgram Flux + Claude Sonnet 5.5 + TTS** | Claude | **$1.00–1.30** | STT ~$0.20 · Claude ~$0.40–0.50 · TTS ~$0.10–0.30 · LiveKit $0.30 ($0 within the free tier or self-hosted) |
| A′. Same, with Haiku 4.5 | Claude | $0.80–1.00 | Weaker probing; use it for simple turns only |
| **B. Async dictation (30 min audio, 3–5 follow-up rounds)** | Claude | **$0.40–0.70** | Nova-3 batch ~$0.13–0.17 · Claude ~$0.25–0.40 · optional TTS for questions ~$0.05–0.10 |
| C. OpenAI gpt-realtime-2.1-mini S2S | GPT | $0.60–1.50 | $0.02–0.05/min (third-party estimate) ([Forasoft](https://www.forasoft.com/article/openai-realtime-api-pricing)) |
| C′. OpenAI gpt-realtime-2.1 S2S | GPT | $1.80–3.30 | $0.06–0.11/min cached (third-party estimate) |
| D. Gemini 3.8 Live S2S | Gemini | $0.70–2.00+ | ~$0.023/min nominal, plus full-context replay ([Gemini pricing](https://ai.google.dev/gemini-api/docs/pricing)) |
| E. ElevenLabs Agents + Claude via proxy | Claude | ~$2.90 | $0.08/min + LLM ([ElevenLabs](https://elevenlabs.io/pricing/agents)) |
| F. Retell (voice engine + Claude 5 Sonnet + TTS) | Claude | ~$4.00 | ~$0.134/min ([Retell](https://www.retellai.com/pricing)) |
| G. Vapi + BYOK Deepgram/Claude/ElevenLabs | Claude | $2.50–3.00 | $0.05/min platform fee ([Vapi](https://vapi.ai/pricing)) |
| H. Hume EVI | ? | $1.20–2.10+ | $0.04–0.07/min. It's unclear whether external LLMs support Claude ([Hume](https://www.hume.ai/pricing)) |
| All-in Deepgram / AssemblyAI voice agent APIs | vendor | $1.50–4.90 / $2.25 | $0.050–0.163/min / $0.075/min ([Deepgram](https://deepgram.com/pricing); [AssemblyAI](https://www.assemblyai.com/pricing)) |

**Batch STT for async (per 30 min of audio):**

| Vendor/model | $/30 min | Keyterms | Diarization | Free credit |
|---|---|---|---|---|
| Deepgram Nova-3 | $0.13 (+$0.04 keyterms) | Yes | Included (batch) | $200 |
| AssemblyAI Universal-3.5 Pro | $0.105 | Included | +$0.01 | 185 hrs pre-recorded |
| ElevenLabs Scribe v2 | $0.11 (+$0.025) | Yes | Not verified | — |
| OpenAI gpt-transcribe | $0.135 | — | — | — |
| Gemini 3.5 Transcribe | ~$0.09 | Prompt-based only | — | — |

Sources: [Deepgram](https://deepgram.com/pricing), [AssemblyAI](https://www.assemblyai.com/pricing), [ElevenLabs API](https://elevenlabs.io/pricing/api), [OpenAI](https://developers.openai.com/api/docs/pricing), [Gemini](https://ai.google.dev/gemini-api/docs/pricing). Several Deepgram streaming rates are marked "promotional", so treat them as volatile. **No independent 2026 word error rate (WER) benchmark exists for noisy truck or job-site audio.** Pick the vendor through a bake-off on 10–20 real recordings, not from datasheets.

**Implication:** the voice vendor choice moves COGS by cents. A typical person needs 2–3 sessions, roughly 60–90 minutes in total. All-in COGS per employee runs **$2–5 on the recommended stack and $8–12 on Retell**. Pick on control and IP ownership, not price.

---

## Six elicitation methods map onto the spec's eight phases

The research groups the best-documented methods for tacit work knowledge into three layers. *Scope the job* with DACUM/SIPOC and the ACTA task diagram. *Walk each task* with episodic "last time" recall and the CDM timeline. *Probe the expertise* with ACTA knowledge-audit probes, CDM deepening and counterfactuals, and light laddering. The Mom Test rules keep every answer grounded in concrete past behavior ([Commoncog on ACTA](https://commoncog.com/an-easier-method-for-extracting-tacit-knowledge/); [AHRQ CDM](https://digital.ahrq.gov/health-it-tools-and-resources/evaluation-resources/workflow-assessment-health-it-toolkit/all-workflow-tools/critical-decision-method); [EKU DACUM](https://www.eku.edu/in/guides/dacum-occupational-analysis/); [mtlynch Mom Test](https://mtlynch.io/book-reports/the-mom-test/)).

| §7.2 phase | Method | Sample question stems | Fills |
|---|---|---|---|
| 1. Role overview | **DACUM** duties → tasks; ACTA "big picture" | "Your owner listed these 6 responsibilities. What's missing?" · "What would break if you were out a week?" · "What's the big picture of this job, the thing a new hire wouldn't get at first?" | `responsibilities` |
| 2. Rhythm | DACUM task list + frequency/criticality | "Walk me through yesterday from the moment you started." · "What happens only weekly? Monthly? Only in summer?" · "Which of these would hurt most if done wrong?" | `procedures` (name, trigger, frequency) |
| 3a. Task diagram *(new)* | **ACTA task diagram** | "Break [procedure] into 3–6 big steps." · "Which step takes the most judgment?" | step skeleton, `judgment_level` |
| 3b. Episodic walk *(new)* | **CDM timeline + Mom Test** | "Tell me about the *last* time you did this. When was it?" · "What was the first thing you did?" · "Then what?" · "How was that time different from normal?" | `steps` (action, trigger, tool, input, output, handoff) |
| 3c. Deepen per step | §7.3 ladder + **ACTA knowledge audit** | "What tells you it's time?" · "Which screen, which field?" · "Who gets it next, and how do they know?" · "Any shortcut a new hire wouldn't know?" | remaining §7.4 fields, `heuristics` |
| 4. Exceptions & loops | **CDM counterfactual + novice error** | "What's the most common thing that goes wrong?" · "Out of 10 times, how many come back to you?" · "**Where would a new person get this wrong?**" (CDM's highest-yield probe ([cdm-interviewer](https://skills.cat/skills/curiositech/port-daddy/cdm-interviewer))) · "What if [system] were down?" | `exceptions[]`, `loop_frequency_pct`, `novice_error` |
| 5. Policies & decisions | **Laddering** (sparingly) + policy-vs-practice probe | "How do you decide? What would make you choose differently?" · "Is that written down or just how it's done?" · "What happens if you skip it? Who would notice?" | `decision_rule`, `policies`, `source` |
| 6. Tools | SIPOC suppliers/inputs | "Every login, spreadsheet, sticky note, group text you touch for this?" | `tools` |
| 7. Friction | Lean waste prompts | "What do you re-type? What do you wait on? What would you never miss if it vanished?" | classification inputs (§8) |
| 8. Read-back | **Contextual inquiry** interpretation check | "Here's what I heard, 6 steps. What did I get wrong or leave out?" Requires an explicit yes ([NN/g](https://nngroup.com/articles/contextual-inquiry)) | `validated_at` |

**Guardrails to encode in the prompt:**

| Rule | Source |
|---|---|
| Treat "I usually / I will / I might" as fluff. Redirect to one concrete past instance | [Mom Test](https://mtlynch.io/book-reports/the-mom-test/) |
| Ask open "walk me through" first. A guessed answer ("I'm guessing X — right?") is for **confirmation only**, never first discovery, because it anchors people to the official process | Synthesis of [interview-me](https://raw.githubusercontent.com/addyosmani/agent-skills/main/skills/interview-me/SKILL.md) + Mom Test |
| Vary "why" phrasing and come back to stuck probes later. Repeated "why" tires people out | [UXmatters laddering](https://www.uxmatters.com/mt/archives/2009/07/laddering-a-research-interview-technique-for-uncovering-core-values.php) |
| Flag "supposed to / should" language and probe the policy-vs-practice gap | Synthesis |
| Ask everyone in a role the **same core questions**, with LLM-generated probes underneath, so the variance engine compares like with like | [AInterviewer, ACL 2026](https://preview.aclanthology.org/ingest-acl/2026.acl-demo.12/) |
| Use a neutral, non-affirming persona. A *Science* study reportedly found AI interviewers affirmed people ~49% more than humans (**secondhand, unverified**) | [UserCall blog](https://usercall.co/post/outset-alternatives-2026) |

---

## The loop is a coverage state machine, not "probe deeper"

Engineering effort goes here. Wuttke et al. found **88% of AI interviewer guideline violations were follow-up failures**, while humans failed mostly at active listening ([arXiv 2410.01824](https://arxiv.org/html/2410.01824v1)). Aftercare's real-time completeness scoring is the closest commercial analog ([Versive](https://getversive.com/help/ai-moderated-interview-platforms)).

### Coverage checklist (per step; drives `get_coverage`)

| Slot | Required for "complete" | Probe if empty |
|---|---|---|
| action (verb + object) | Yes | "What exactly do you do first?" |
| trigger | Yes | "What tells you it's time?" |
| tool | Yes | "Where do you do that?" |
| input / output | Yes | "What do you need in hand? What comes out?" |
| handoff_to | Yes (or "none") | "Who gets it next?" |
| decision_rule | If the step contains "depends" or "if" | "What's the threshold?" |
| frequency + est_minutes | Yes | "How many last week? How long each?" |
| exceptions[] | ≥1, or explicit "never goes wrong" | "Most common thing that goes wrong?" |
| why | Yes | "What happens if you skip it?" |
| source (policy/practice) | Yes | "Written down anywhere?" |
| novice_error / heuristic | Per procedure, ≥1 | "Where would a new person slip?" |

### Stop rules (resolves the §7.3 open question)

| Level | Stop when | Basis |
|---|---|---|
| Slot | Filled · or the person says "don't know" (`confidence=low`) · or **2 probes fail**, then `mark_gap(deferred)` and return later | Fatigue research ([UXmatters](https://www.uxmatters.com/mt/archives/2009/07/laddering-a-research-interview-technique-for-uncovering-core-values.php)) |
| Procedure | All required slots filled **and** either 3 consecutive probes add no new step, exception or decision, **or** the agent passes the "can I predict the next three answers?" test. Then run the read-back | [interview-me](https://raw.githubusercontent.com/addyosmani/agent-skills/main/skills/interview-me/SKILL.md) |
| Session | 20–25 min hard cap, or a fatigue signal (shorter answers, "I don't know" streak). Close with: "Anything a replacement would need that I didn't ask?" | Vendor-cited fatigue rise after ~25 min ([Perspective AI](https://getperspective.ai/blog/ai-moderated-interviews-how-they-work-when-to-use-them-and-what-they-replace/markdown)); Anthropic ran 10–15 min ([Anthropic](https://www.anthropic.com/news/anthropic-interviewer)) |
| Person | All seeded and discovered procedures have validated read-backs. Expect 2–3 sessions | Sensay's 2–3 sessions over 1–2 weeks ([AccessNewswire](https://www.accessnewswire.com/newsroom/en/education/sensay-launches-world%E2%80%99s-first-ai-offboarding-platform-for-knowledge-transfer-1090553)) |
| Role (how many people to interview) | A run of 2–3 interviews adds ≤5% new steps. Expect ~6 for a homogeneous role, 8–9 for a mixed one | [Guest, Namey & Chen 2020](https://journals.plos.org/plosone/article?id=10.1371/journal.pone.0232076) |

### Read-back protocol

The 5–8 line restate includes an **"Out of scope / not covered"** line. Only an **explicit yes** counts; "sounds good", silence and "whatever you think" do not ([interview-me](https://raw.githubusercontent.com/addyosmani/agent-skills/main/skills/interview-me/SKILL.md)). Read back numbers and thresholds digit by digit in voice ([OpenAI Realtime prompting guide](https://developers.openai.com/cookbook/examples/realtime_prompting_guide)). Corrections write back via `upsert_step` and then re-run the read-back. Validation then layers DACUM-style: self → peer → manager/owner ([EKU](https://www.eku.edu/in/guides/dacum-occupational-analysis/)).

### Async rounds

| Step | Detail |
|---|---|
| 1. Kickoff | The person records a free "walk me through your week" dictation (5–15 min) |
| 2. Process | STT → Claude extracts procedures and steps via the same tools → `get_coverage` |
| 3. Next round | Send **1–3 questions**, picked from the open-probe queue by priority (missing required slot > exception > why). Deliver by push/SMS as text plus a TTS clip |
| 4. Reply | A voice note per question; repeat. The ACTA customization study found async iterative rounds improved understanding ([arXiv 2108.05622](https://arxiv.org/pdf/2108.05622)) |
| 5. Close | A read-back doc (draft SOP) with an approve/correct button |

### Cross-person contradiction checks (resolves §7.5 and §16 Q6)

Use Tacit's triage ([getAbstract Tacit](https://www.getabstract.com/en/productivity/tacit)) **after** the person's free description of each procedure, never before. That avoids anchoring the answer.

| Claim from person N vs. persons 1..N-1 | Action |
|---|---|
| Consistent | Auto-confirm; no question |
| New (nobody else said it) | Keep; mark for a quick yes/no confirmation from one peer later |
| Missing (others do X, N didn't mention it) | "Some people in this role also do X. Do you?" (unattributed) |
| Contradiction (different rule or threshold) | "I've heard this handled two ways: A and B. Which do you do, or do both happen?" (never name the colleague) |

Also check for contradictions within one person: compare the frequency and steps in the episodic walk against the generic description, and flag any gap between policy language and practice.

---

## Reusable prompts and skills compose into one system prompt

| Asset | Take from it | Link |
|---|---|---|
| **obra/superpowers `brainstorming`** | One question per message, multiple choice preferred, present the result in sections with approval after each, depth scaled to complexity | [SKILL.md](https://raw.githubusercontent.com/obra/superpowers/main/skills/brainstorming/SKILL.md) |
| **addyosmani `interview-me`** (MIT) | Hypothesis plus 0–100% confidence, guess-attached questions, "predict next three" stop test, explicit-yes restate with Out of scope | [SKILL.md](https://raw.githubusercontent.com/addyosmani/agent-skills/main/skills/interview-me/SKILL.md) · [repo](https://github.com/addyosmani/agent-skills) |
| **mattpocock `grill-me`** | "Interview me relentlessly until shared understanding", walk each branch of the tree, look it up instead of asking when the answer is knowable (for PositionMap: check the tool inventory and owner seeds) | [skills.sh](https://www.skills.sh/vinvcn/mattpocock-skills/grill-me) |
| **curiositech `cdm-interviewer`** | Four timed CDM sweeps; heuristic format "When [cue], because [reason], do [action], unless [exception]"; rejects composite or hypothetical incidents | [skills.cat](https://skills.cat/skills/curiositech/port-daddy/cdm-interviewer) |
| **Anthropic Interviewer** | Plan → interview → analyze architecture; the interview guide is drafted by Claude and approved by a human; Claude classifiers tag transcripts afterward; 97.6% satisfaction (5+/7) across 1,250 interviews. System prompt not public | [announcement](https://www.anthropic.com/news/anthropic-interviewer) · [81k study](https://www.anthropic.com/81k-interviews) |
| **Anthropic Interviewer dataset** (CC-BY) | Real Claude-run interviews of professionals about their work; use as a style reference and **evaluation baseline** | [Hugging Face](https://huggingface.co/datasets/Anthropic/AnthropicInterviewer) |
| **OpenAI Realtime prompting guide** | Voice prompt skeleton: Role & Objective, Personality & Tone, Context, Pronunciations, Tools, Rules, Conversation Flow, Safety & Escalation. Phases with exit criteria as a JSON state machine; variety rules; ask again on unclear audio; swap in each phase's rules and tools mid-session | [Cookbook](https://developers.openai.com/cookbook/examples/realtime_prompting_guide) |
| **Thariq "interview me" pattern** | "Interview me in detail…until complete", then write the spec to a file. 20–30 questions is typical; implement in a fresh session | [VelvetShark](https://velvetshark.com/stop-prompting-claude-code-let-it-interview-you) |
| **Geiecke & Jaravel prompt** | Non-directive questioning, "palpable evidence" (concrete examples), cognitive-empathy follow-ups. **Noncommercial license: learn from it, don't copy it** | [GitHub](https://github.com/friedrichgeiecke/interviews) · [CEPR DP19705](https://cepr.org/publications/DP19705) |
| ACTA knowledge-audit probes | Past & Future, Big Picture, Noticing, Job Smarts, Improvising, Self-Monitoring, Anomalies, Equipment Difficulties | [DTIC ADA335225](https://apps.dtic.mil/sti/pdfs/ADA335225.pdf) |

**System prompt skeleton (OpenAI guide structure plus the pieces above):**

```
# Role & Objective   Capture how {person} actually does {position} so someone else could do it Monday.
# Personality/Tone  Neutral, curious, brief. Never praise answers. "Help capture your expertise."
# Context           Position brief, owner-seeded responsibilities, tool inventory, glossary,
                    recap of captured procedures, coverage gaps, open probes (top 3).
# Pronunciations    Tenant glossary (ServiceTitan, PEX, RPZ...).
# Tools             upsert_procedure, upsert_step, flag_exception, record_policy, record_heuristic,
                    record_tool, mark_gap, get_coverage, readback_ready/confirm_readback, end_segment.
# Rules             One question per turn. Past, concrete instance before generalities.
                    Guess-attached questions only to confirm. ≤2 probes per slot, then mark_gap.
                    Call get_coverage after every procedure-changing answer; ask from its gaps.
                    Treat "should/supposed to" as a policy-vs-practice probe. No colleague names.
                    If audio unclear: ask to repeat; never guess numbers.
# Conversation Flow Phase state machine 0–8 with exit criteria (see loop design).
# Safety            Stop on distress or HR/legal complaints → end_segment(escalate). Remind about
                    bystanders. Honor "skip" and "off the record" (do not store).
```

---

## Buy a $1–3 pilot, learn from Ontora and Tacit, build on OpenInterviewer

| Option | Verdict | Why | Cost |
|---|---|---|---|
| **Koji** | **Buy for v0** (also the cheapest scripted option) | Voice and text, Mom Test/JTBD presets, REST API, **Claude MCP**, embeddable widget | ~€1 text / €3 voice per interview ([Koji](https://www.koji.so/docs/best-ai-interview-software-2026), vendor) |
| UserCall | Buy-alt for v0 | Webhooks, branding, quotes traceable to source; interviews up to 25 min on Core | $199/mo ([UserCall](https://www.usercall.co/pricing)) |
| Perspective AI | Buy-alt | Conditional probe rules in the protocol; can sample from HR data | $5–30/interview ([Perspective AI](https://getperspective.ai/blog/ai-moderated-interviews-how-they-work-when-to-use-them-and-what-they-replace/markdown)) |
| **Ontora** (YC S26) | **Benchmark; get a demo** | Same thesis: chat with hold-to-speak voice, 20–30 min resumable sessions, campaign templates, questions seeded from the tool inventory, swimlane maps. Enterprise focus, no published pricing | [YC](https://www.ycombinator.com/launches/PyU-ontora-read-your-company-like-a-book) |
| **Tacit** (getAbstract) | Learn from | Consistent/new/contradiction claim triage; 5-min micro-captures | [Tacit](https://www.getabstract.com/en/productivity/tacit) |
| Sensay | Learn from | Multi-session offboarding; "chat with the departed employee" output (a future PositionMap add-on) | [AccessNewswire](https://www.accessnewswire.com/newsroom/en/education/sensay-launches-world%E2%80%99s-first-ai-offboarding-platform-for-knowledge-transfer-1090553) |
| **OpenInterviewer** (MIT) | **Fork or learn from** | Next.js/TS, Claude per study, "Concrete Incidents" manner preset, synthesis with provenance; text-only | [GitHub](https://github.com/linxule/openinterviewer) |
| AInterviewer (ACL 2026) | Learn from | Fixed core questions plus LLM probes, which keeps people in a role comparable | [ACL](https://preview.aclanthology.org/ingest-acl/2026.acl-demo.12/) |
| LiveKit Agents / Pipecat | **Build voice layer** | Mature, open source, Claude-native. Interview-specific templates are thin job-screening demos ([Maxim example](https://www.getmaxim.ai/articles/how-to-build-a-real-time-ai-interview-voice-agent-with-livekit-and-maxim-a-technical-guide/)), so write your own | ~$0.01/min |
| Voice Agent Readiness (nexaintel) | Learn from | Simulated test personas against a live voice interview agent, a pattern for QA regression | [GitHub topics](https://github.com/topics/ai-interviews) |
| Outset / Listen Labs / Strella / Conveo | Skip | $10K–25K+, built around panels and consumers | [Versive](https://getversive.com/help/ai-moderated-interview-platforms) |

**Why build after the pilot:** no vendor offers the step schema, cross-person alignment, outcome linkage or 5-bucket classification. Every research platform probes for *richness*, not *checklist coverage*. None markets internal employee process capture, and their terms for HR-adjacent use are unverified.

---

## v0 pilot: two weeks, one role, eight people, under $100

| Day | Action | Output |
|---|---|---|
| 1–2 | Pick one role at a friendly trades client (CSR or service tech) with 5–8 people. Get written FL consent forms signed | Consent on file |
| 1–2 | Write the interview guide using the phase table above. Load it into **Koji** (voice). *In parallel*, build a **Claude Project** with the system prompt skeleton plus the §7.4 schema as the output format, as the text/dictation arm | Two interviewer arms |
| 3 | Record 10–20 real field voice memos. Run them through Deepgram Nova-3 (keyterms on) and AssemblyAI U-3.5 Pro. Hand-score jargon accuracy | STT vendor pick |
| 3–8 | Interview each person: 1 live session (Koji or Claude Project), then 1–2 async voice-note rounds (person records on phone → Nova-3 → Claude → 1–3 questions back by text) | 5–8 transcripts plus step JSON |
| 9 | Read-back: each person approves or corrects their procedure | Validated rate |
| 10 | Manually align steps across people in a spreadsheet; check the ≤5% new-steps saturation point | Variance grid prototype |
| 10 | Score: % of required slots filled · steps per procedure vs owner's estimate · # exceptions · read-back correction rate · candor survey (1–5) · minutes per person | Go/no-go for the LiveKit build |

**Pass bar (suggested):** ≥80% of required slots filled after the read-back; **at least 1.5x as many steps as the owner's documented version** (the spec's Deloitte 7→20 pattern); ≥4/5 comfort rating; voice-note STT jargon accuracy good enough that Claude's normalization fixes the rest.

---

## Risks: consent, field audio and candor will bite before cost does

| Risk | Evidence | Mitigation |
|---|---|---|
| **Florida all-party consent** | §934.03(2)(d) requires prior consent of all parties; a violation is a **third-degree felony** (§934.03(4)(a)) ([Fla. Stat. 934.03](https://www.flsenate.gov/Laws/Statutes/2025/934.03)) | Spoken plus checkbox consent at the start of **every** session (live and each async note), stored as `consent_events` rows with the audio offset. A bystander warning ("don't record near customers"), with optional auto-pause. Chat-only mode is always available. Get counsel to review the script |
| **Noisy field audio** | Streaming transcription errors run ~10.9% in AI voice interviews ([Tirumala et al.](https://arxiv.org/html/2509.01814v1)). Retell sells denoising as an add-on, a sign the problem is known ([Retell](https://www.retellai.com/pricing)). No independent field-audio WER data | Prefer **async batch** for field staff (more accurate than streaming). Use keyterms, Claude normalization against the glossary, and a "say it again" rule for numbers. Read-back catches the rest. Bake off on real recordings |
| **Employee candor / fear of replacement** | Employees hold back tacit knowledge without confidentiality and appreciation ([Benderoth et al.](https://arxiv.org/abs/2508.19942v1)). There is no data on candor when the employer deploys the interviewer | "Capture your expertise" framing. Staff see only their own playbook plus the approved standard. Contradiction probes are unattributed. Credit contributors in the standard procedure. Offer "off the record" and deletion on request ([Expert Mind](https://arxiv.org/abs/2603.14541)) |
| **Shallow follow-ups** | 88% of AI violations were follow-up failures ([Wuttke](https://arxiv.org/html/2410.01824v1)) | Coverage-driven `get_coverage` loop; evaluation suite on the Anthropic dataset plus synthetic personas |
| **Sycophancy / anchoring to official process** | Unverified 49%-more-affirmation claim ([UserCall](https://usercall.co/post/outset-alternatives-2026)) | Neutral persona, no praise; guesses only for confirmation; cross-person probes only after free description |
| **Vendor session caps / pricing churn** | OpenAI 60 min, Gemini 15 min audio; Deepgram "promotional" rates | State in Supabase; STT behind an adapter interface |
| **AI "data without meaning" critique** | Penn State IST ([link](https://ist.psu.edu/news/ist-researchers-discuss-ai-interviewers-on-the-conversation)) | Human review layers (peer → manager → consultant) before anything becomes a standard |

---

## Suggested spec additions (drop-in edits to position-capture-spec.md)

| § | Change |
|---|---|
| §6 `interview_sessions` | Change `mode` to `live_voice \| async_voice \| chat`; add `round_no`, `parent_session_id`, `stt_vendor`, `audio_ref`, `duration_s`, `cost_usd` |
| §6 new `interview_turns` | session_id, seq, speaker, text, audio_offset_ms, stt_confidence. This makes `evidence` refs resolvable |
| §6 new `consent_events` | session_id, person_id, type (recording/data_use), method (verbal/checkbox), audio_offset_ms, at |
| §6 new `open_probes` | person_id, entity_type/id, slot, priority, reason (unknown/declined/deferred/contradiction/missing_vs_peers), status, source_claim_id. Feeds the async question picker and the session recap |
| §6 new `heuristics` | step_id/procedure_id, when, because, do, unless, novice_error, evidence (CDM format) |
| §6 new `glossary_terms` | org_id, position_id, term, aliases[], category. Feeds STT keyterms and the prompt |
| §6 `steps` | Add `source` (policy/practice), `novice_error`, `tricks`, `validated_at`, `claim_status` (consistent/new/contradicted, Tacit triage) |
| §7.2 | Split phase 3 into **3a task diagram (ACTA) → 3b episodic "last time" walk (CDM/Mom Test) → 3c deepen**. Add the counterfactual and novice-error probes to phase 4. Add the bystander warning to phase 0 |
| §7.3 stop rule | Replace the ❓ with: ≤2 probes per slot then `mark_gap(deferred)`; procedure done = required slots plus (3 null probes or predict-next-3); session cap 20–25 min |
| §7.5 | Resolve: cross-person probes only **after** free description; use the consistent/new/missing/contradiction table; never attribute |
| §7 new 7.6 | **Modes:** live and async share one agent, prompt and tool contract; async rounds ask 1–3 questions; context is rebuilt from Supabase each round |
| §7 new 7.7 | **Core-question set per position template** (fixed wording, as in AInterviewer) for comparability |
| §13 | Consent on every session and every async note; "off the record" honored; retention policy for raw audio |
| §14 MVP | Move **async voice notes into v1** (batch STT is cheap and robust for techs in trucks); keep **live voice as v1.1** on LiveKit. Answers §16 Q2: "chat plus voice-note first, live voice second" |
| §14 stack | Tool names → `upsert_procedure, upsert_step, flag_exception, record_policy, record_heuristic, record_tool, mark_gap, get_coverage, readback_ready, confirm_readback, end_segment` |
| §16 new Qs | STT vendor bake-off result; Haiku/Sonnet routing threshold; per-person session budget (2 vs 3); evaluation harness (Anthropic dataset plus synthetic personas) |

---

## Conclusion

The research reframes PositionMap's build risk. Voice is now a solved, cheap commodity at **about $1 per live half hour**. Owning the stack costs no more than renting it, which supports MacTutor's build-over-adopt approach. What is not solved anywhere is the **coverage discipline**: knowing which slot of which step is still empty and asking for exactly that. That should be the product's core IP. It means a `get_coverage`-driven state machine, CDM/ACTA probes as the question bank, explicit-yes read-backs, and Tacit-style cross-person triage. The research tools optimize for rich quotes. PositionMap needs complete, comparable step graphs, which is a different problem and a defensible one.

The second insight is that **async voice notes are the better v1 for trades, not a fallback**. Batch transcription is more accurate than streaming in noisy settings. It costs about $0.13 per 30 minutes, fits a tech's day between jobs, and turns the AI's main weakness (follow-up quality under live pressure) into a strength, because Claude can plan each round's 1–3 questions against the whole coverage map. For the brainstorming session, three decisions are open: (1) per-person versus canonical procedure storage (§6 ❓); (2) how many probes per slot before deferring; (3) whether the v0 pilot's step counts beat the owner's documented process by enough to justify the LiveKit build.
