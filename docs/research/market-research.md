# Interview every role, then sell the automation

**Verdict: build it, but as a vertical, done-for-you service first and as software second.** AI tools that interview employees to capture knowledge exist, but every one we found serves **enterprises** (Ontora, Klarity, Tacit) or **one-expert offboarding** (Sensay, Nuggetz). We found **no product for small businesses** where the owner lists the roles, an AI interviews each person in each role, and the result is a company-wide operations guide plus a list of what to automate. The need is real but stays dormant until something forces it. **95% of owners say transition planning matters, but 65% have no written plan** ([EPI](https://blog.exit-planning-institute.org/2022-colorado-state-of-owner-readiness-report-is-released)). In HVAC, M&A advisors tie documented systems to valuation: **about 2.3x on average versus a 5x target** ([ACHR News](https://achrnews.com/articles/143874-how-to-achieve-a-5x-valuation-multiple-in-an-hvac-business)). Owners already pay for help getting systematized: Trainual runs **$3K–$5K a year**, trades coaching groups **$500–$2,500 a month**, and EOS implementers **$20K–$45K a year**. So a **$3,500–$7,500 done-for-you diagnostic**, credited toward automation work, fits budgets that already exist. The main risks are a crowded field moving down-market from enterprise, Florida's all-party recording-consent law, and employees fearing they are training their replacement. **Direct demand ("I want AI to interview my staff") is unverified, because Reddit was blocked from the research environment.** Close that gap by hand before you build (searches listed in the last section).

## Ontora and Klarity do this for enterprises, and nobody does it for a 15-person HVAC shop

The category took shape in 2025–2026, and it has split into three groups. None of them targets SMB trades. Ontora (YC, Spring 2026) is the closest match to the concept: "an AI agent that interviews every employee in your company to map how work actually gets done." It delivers process maps and an automation roadmap, aimed at enterprises ([YC](https://www.ycombinator.com/launches/PyU-ontora-read-your-company-like-a-book)). Klarity's AI Interviewer goes from interview to SOP to **automation opportunities**, but it is contact-sales only and backed by a $70M Series B ([Klarity](https://support.klarity.ai/hc/en-us/articles/23660996892444-Process-Performers-Current-state-discovery-observations); [SiliconANGLE](https://siliconangle.com/2024/06/24/klarity-intelligence-raises-70m-automate-document-review-process/)). The SOP tools SMBs actually buy (Trainual, Scribe, Whale) assume the owner already knows the process and will write it down or record a screen. Screen capture cannot see phone, truck or job-site work, and that is most of the work in a trades business.

| Product | What it does | Target | Pricing | Gap vs. MacTutor concept |
|---|---|---|---|---|
| **Ontora** (YC S26) | AI interviews every employee, then process map and automation roadmap | Enterprise | Not public | Closest concept; not SMB/trades; likely to move down-market |
| **Klarity** | AI interviewer plus screen observation, then SOP and automation opportunities | Enterprise, PE operating partners | Contact sales | Enterprise price and setup |
| **Tacit** (getAbstract) | 30-min AI interviews before retirements, fed into Copilot/Teams | Enterprise M365 | Demo only | Offboarding focus, one expert at a time |
| **Sensay** | AI voice offboarding interviews, then knowledge base and chatbot | Mid/enterprise | ~$500/yr (low-confidence source) | Triggered by departures; not role-mapping |
| **Nuggetz** | AI voice/text interviews; cross-checks several SMEs and flags conflicts | Mid-market | Not public | Organized by topic, not role; no automation output |
| **KS-Agents "Torch"** | HR lifecycle interviews plus SOP generation | EU SMB/HR | Free ≤10 users; €49/mo unlimited | HR framing; no systems inventory |
| **Docsie** | Interview templates plus recording, then SOPs | Manufacturing, legal, construction | 14-day trial | Human runs the interview; no AI interviewer |
| **Audity** | White-label AI-readiness questionnaires plus *generated* stakeholder interview questions, then scored report | AI consultants/agencies | $99 / $397/mo (vendor's own figures) | Assessment engine, not an ops manual; unclear whether AI conducts the interviews |
| **Trainual / Whale / Waybook** | Host and author SOPs; AI writes from prompts | SMB | $99–$499/mo; Trainual +$1K setup | Owner must supply the content; no interviews |
| **Scribe / Tango / Guidde** | Click-capture SOPs | SMB–enterprise | $13–$25/creator/mo | Screen work only |
| **Celonis / UiPath mining** | Event-log process mining | Fortune 500 | $15K–$200K+/yr | Needs ERP logs; irrelevant at SMB scale |

Sources: [Ontora](https://www.ycombinator.com/launches/PyU-ontora-read-your-company-like-a-book), [Klarity](https://neuronfeed.com/startups/klarity), [Tacit](https://www.getabstract.com/en/productivity/tacit), [Sensay PR](https://www.accessnewswire.com/newsroom/en/education/sensay-launches-world%E2%80%99s-first-ai-offboarding-platform-for-knowledge-transfer-1090553), [Nuggetz](https://nuggetz.ai/), [KS-Agents](https://ks-agents.com/blog/ai-employee-interviews-when-to-use/), [Docsie](https://www.docsie.io/solutions/tacit-knowledge-capture/), [Audity](https://auditynow.com/), [CostBench Trainual](https://www.costbench.com/software/learning-management/trainual/), [Whale](https://usewhale.io/pricing/), [Scribe](https://scribe.com/pricing), [Bardeen](https://www.bardeen.ai/best/celonis-alternatives).

Trades vendors leave the same gap open. Sera tells HVAC contractors to write their own operations manual ([Sera](https://sera.tech/blog/creating-a-hvac-business-operations-manual?hsLang=en)). ServiceTitan's Contractor Playbook is advice content ([ServiceTitan](https://help.servicetitan.com/docs/servicetitans-contractor-playbook.md)). Nexstar gives members a blank template. **Every coach and vendor tells the owner to "write the manual," and none of them writes it.** That step is exactly what an AI interviewer can do.

**What no competitor combines** (this is MacTutor's opening):

| Unmet need | Nearest analog | Why it matters for MacTutor |
|---|---|---|
| Owner-defined roles, then per-role interviews, then **handoff reconciliation** (CSR to dispatch to tech to billing) | Nuggetz conflict detection (by topic) | Turns separate SOPs into a map of how the company actually runs |
| **Systems inventory** by role (ServiceTitan, QuickBooks, Excel, texts) | None | This is the raw material for selling integrations and automation |
| Mobile **voice** interviews for field and phone staff | Sensay (offboarding) | Captures work that screen-capture tools cannot see |
| **Ranked automation backlog** as an output | Klarity, Scribe Optimize (enterprise) | Feeds directly into the consulting upsell |
| Trades and legal **interview templates** (HVAC dispatcher, intake paralegal) | Static PDFs (Nexstar, Aptora) | Vertical depth is the moat against horizontal tools |
| SMB price point and **white-label interviewer** for agencies | Audity (questionnaires only) | A second revenue line after local proof |

## The pain is real but stays dormant until a crisis

The strongest evidence comes from surveys and from people who sell to owners. First-person owner complaints are thin in this research because Reddit was inaccessible. The pattern that holds across sources is **an intent–action gap**: owners agree documentation matters and don't do it, and 32% say it's because they are busy growing the company ([EPI](https://blog.exit-planning-institute.org/2022-colorado-state-of-owner-readiness-report-is-released)). The pitch therefore has to promise **near-zero owner time**, and the buying moment is usually a trigger: a key person leaves, the owner gets sick, a sale comes up, or PE comes knocking.

| Signal | Strength | Evidence |
|---|---|---|
| Owners lack documentation and exit readiness | **Strong** (survey, n=400+) | 65% have no written plan; 78% have no advisory team ([EPI](https://blog.exit-planning-institute.org/2022-colorado-state-of-owner-readiness-report-is-released)) |
| Owners already pay to get systematized | **Strong** | Trainual $2,988–$4,788/yr plus $1K setup ([CostBench](https://www.costbench.com/software/learning-management/trainual/)); EOS $20K–$45K/yr ([Strety](https://strety.com/blog/eos-implementer-cost/)) |
| Enterprise VC is funding this exact concept | **Strong** (supply-side) | Ontora (YC S26), KNOA ([Feedbagel](https://feedbagel.com/post/knoa-ai-platform-for-capturing-team-knowledge-through-structured-interviews)) |
| PE buyers require documented dispatch and CSR processes | **Moderate–strong** | Named as a diligence priority ([ServiceTitan PE guide](https://www.servicetitan.com/blog/hvac-private-equity)) |
| Documentation raises trades valuation | **Moderate** (expert opinion) | 2.3x average vs. 5x target ([ACHR News](https://achrnews.com/articles/143874-how-to-achieve-a-5x-valuation-multiple-in-an-hvac-business)); [OffDeal](https://offdeal.io/blog/developing-standard-operating-procedures-when-preparing-to-sell-your-hvac) |
| Existing SOP tools fail on effort, adoption and staleness | **Moderate** (reviews) | "Took several weeks of content building" ([G2](https://www.g2.com/products/trainual/reviews?page=9)); "extremely expensive for small businesses" ([Capterra](https://www.capterra.com/p/175749/Trainual/reviews/?page=2)) |
| SOPs written by AI without staff input fail | **Moderate** (one case study) | 2 of 19 AI SOPs used; net 40% time *loss* ([DEV](https://dev.to/fieldwork/i-automated-my-sop-writing-with-ai-and-it-was-a-disaster-heres-what-i-learned-1jbh)). Interviewing the actual role-holder is the fix |
| FSM complexity creates one-person dependence | **Weak–moderate** | Reviewer predicts a "ServiceTitan Administrator" job title ([Capterra](https://capterra.com/p/150053/ServiceTitan/reviews/?page=7)) |
| Trades office-staff churn | **Weak** (proxy) | Call-center turnover 30–45%/yr ([Nextiva](https://www.nextiva.com/blog/call-center-turnover-rates)); no trades-specific figure found |
| **Direct "AI should interview my staff" demand** | **Unverified** | Reddit blocked; no first-person requests found |
| Law-firm demand | **Not researched** | No evidence gathered |

The role ranking for trades follows the money. The **CSR/phone and dispatch seats** hold the most tribal knowledge and the clearest revenue impact. ServiceTitan data shows an average **42% call booking rate against an attainable 90%**, with shops under 5 techs at 24%. Five points of booking rate is worth **about $100K/yr** to a 5–14 tech shop ([ServiceTitan](https://www.servicetitan.com/blog/data-call-booking-rates)). Investors are funding automation of exactly these seats: Probook raised $40M for AI dispatch, and **a Florida operator cut dispatchers from 22 to 10** with it ([Fortune](https://www.fortune.com/2026/06/23/exclusive-this-startup-wants-to-be-the-ai-brain-for-home-services-and-it-just-raised-40-million-from-sequoia-and-a16z/)). Housecall Pro's 2026 survey shows 48% of contractors use AI, mostly for follow-up (52%) and quoting (51%). **52% of non-users don't know which trade AI tools exist** ([Housecall Pro](https://www.housecallpro.com/wp-content/uploads/2026/06/062026-HCP-The-AI-Advantage-report.pdf)). A vendor-neutral opportunity map that names Avoca, Probook or the FSM's own AI answers that confusion directly.

## A $3,500–$7,500 diagnostic fits budgets owners already spend

The commodity floor is low and the ceiling is high, so packaging decides the price. Sold as "SOPs," the work competes with **$50–$300 per SOP on Upwork** and **$500–$1,500 role-based manuals** ([Upwork](https://www.upwork.com/services/product/admin-customer-support-a-custom-detailed-operations-manuals-to-streamline-your-business-1841261846803085201)). Sold as a diagnostic that leads to automation, it sits in the established SMB AI-audit band. Audits for 10–60 employees cost **$1,500–$2,500** ([BetOnAI](https://betonai.net/ai-automation-audit-paid-service-2026-solo-operators-1500-5000-engagement-implementation-retainers/)). Main & Machine charges **$3,500–$8,500**, credited toward implementation sprints of $18K–$60K ([Main & Machine](https://www.mainandmachine.com/pricing/)). UK audits for 10–50 staff that include staff interviews run **£2,500–£8,000** ([Lilach Bullock](https://www.lilachbullock.com/ai-audit-small-business-cost/)). Follow-on implementation typically costs **$5K–$25K** plus **$500–$5,000/mo** retainers ([Enterprise DNA](https://enterprisedna.co/resources/ai-pulse/ai-pulse-2026-08-25-the-ai-implementation-agency-retainer-model-has-settled-into)).

**Benchmarks**

| Anchor | Price | Source |
|---|---|---|
| Upwork SOP writer | $50–$300 per SOP | [Upwork](https://www.upwork.com/services/product/consulting-hr-process-documentation-standardization-small-business-operations-2051263516372938352) |
| Upwork "AI automation audit" (race to the bottom) | $50 | [Upwork](https://www.upwork.com/services/product/development-it-ai-automation-audit-find-what-to-automate-and-how-2050966996584825962) |
| SweetProcess / Waybook | $99–$198/mo | [Waybook](https://www.waybook.com/pricing) |
| Trainual | $249–$399/mo + $1K setup | [CostBench](https://www.costbench.com/software/learning-management/trainual/) |
| Whale | $249–$499/mo | [Whale](https://usewhale.io/pricing/) |
| Audity (white-label for consultants) | $99–$397/mo | [Audity blog](https://auditynow.com/blog/best-ai-readiness-assessment-tools) |
| SMB AI audit (10–60 employees) | $1,500–$2,500 | [BetOnAI](https://betonai.net/ai-automation-audit-paid-service-2026-solo-operators-1500-5000-engagement-implementation-retainers/); [Justin Harris](https://justinharris.ai/blog/las-vegas-ai-consultant-pricing/) |
| Premium AI readiness audit | $3,500–$8,500 | [Main & Machine](https://www.mainandmachine.com/pricing/) |
| Trades coaching groups (Nexstar etc.) | $500–$2,500/mo | [Lightning Path](https://lightningpathpartners.com/blog/hvac-coaching) (undated, single source) |
| EOS implementer | $20K–$45K/yr | [Strety](https://strety.com/blog/eos-implementer-cost/) |

**Recommended pricing** (hypotheses derived from the benchmarks, to be tested)

| Model | Who buys | Price | Includes | Notes |
|---|---|---|---|---|
| **Free lead magnet** | Any owner | $0 | 10-min owner self-assessment: roles, systems, "where does it break when X is out" score | Value Builder/Audity-style top of funnel ([Built to Sell](https://builttosell.com/?p=1663)) |
| **Done-for-you diagnostic** (lead offer) | Trades/SMB 5–50 staff | **$3,500** (≤15 staff) / **$5,500** (16–30) / **$7,500** (31–50) | AI interviews of every role, reviewed by a consultant; ops guide; systems map; handoff map; ranked automation backlog with ROI | Credit **50–100% toward implementation signed within 60 days** |
| **Hybrid / living manual** | Post-diagnostic clients | $1,500 setup + **$299–$499/mo** | Quarterly re-interviews, onboarding interviews for new hires, offboarding capture, guide kept current | Addresses the staleness complaint; sits inside coaching-group budgets |
| **SaaS self-serve** | DIY owners outside Tampa | **$199 / $349 / $499/mo** flat by staff band (5–15 / 16–30 / 31–50) or $1,500–$3,000 one-time capture + $99–$199/mo | Self-run interviews and guide; export to Trainual/Whale | Flat pricing avoids per-seat friction (Trainual's 10-seat minimum was called "wasteful") |
| **White-label for agencies** | AI automation agencies, MSPs, EOS implementers, CEPAs | **$299–$999/mo** platform fee + **$150–$400 per completed company assessment** | Branded interviewer, trades/legal templates, report and automation backlog | Priced above Audity ($99–$397) because it conducts the interviews; launch after 5–10 local case studies |
| **Downstream implementation** | Converted diagnostic clients | $7.5K–$20K builds; $1K–$2.5K/mo retainers | Automations from the backlog (CSR, dispatch, follow-up, AR) | Plan on **30–50% conversion**; the 50–70% figures in circulation are self-reported |

The unit economics justify pricing the diagnostic close to cost. If **one converted client brings $5K–$25K in implementation plus $500–$2,500/mo in retainer**, then at a 30–50% conversion rate the diagnostic pays off as a lead engine even at break-even.

## Florida consent law and staff fear are the risks most likely to stop deals

| Risk | Severity | Evidence | Mitigation |
|---|---|---|---|
| **Florida all-party recording consent** (§ 934.03) | High | Recording without every party's consent is generally a **third-degree felony**, with civil damages under § 934.10 ([RCFP](https://www.rcfp.org/reporters-recording-guide/florida/)) | Spoken *and* written consent at the start of every session; employee privacy notice; have counsel confirm the statute text (wording not verified in this research) |
| **Employee fear of replacement** | High | 52% of workers worried about AI vs. 36% hopeful ([Pew](https://www.pewresearch.org/social-trends/2025/02/25/u-s-workers-are-more-worried-than-hopeful-about-future-ai-use-in-the-workplace/markdown)); a Florida operator cut dispatchers 22 to 10 ([Fortune](https://www.fortune.com/2026/06/23/exclusive-this-startup-wants-to-be-the-ai-brain-for-home-services-and-it-just-raised-40-million-from-sequoia-and-a16z/)) | Two outputs: a **staff-facing playbook** ("your coverage when you're out") and an owner-only automation backlog; no attribution of individual answers in reports |
| Interviews capture work as described, not work as actually done | Medium | Workarounds go unreported ([Koji](https://www.koji.so/docs/work-as-imagined-vs-work-as-done), vendor); [BISE 2025](https://uni-goettingen.de/en/701431.html) | Cross-check against ServiceTitan/HCP/QuickBooks reports and screen walkthroughs; MacTutor's DDR and database background fits this check |
| Enterprise players move down-market | Medium | Ontora and Klarity have funding and the same output | Vertical templates, local delivery and consultant review are hard for horizontal SaaS to copy |
| Owners don't act on the report | Medium | Intent–action gap ([EPI](https://blog.exit-planning-institute.org/2022-colorado-state-of-owner-readiness-report-is-released)) | Fee credit, a 30/60/90 implementation offer, and a booking-rate dollar hook |
| Commoditization ($50 audits) | Medium | [Upwork](https://www.upwork.com/services/product/development-it-ai-automation-audit-find-what-to-automate-and-how-2050966996584825962) | Sell multi-role interviews plus vertical benchmarks, never a single owner call |
| Law firms: privilege and confidentiality | Unknown | Not researched | Exclude client-matter content from interviews; get bar-ethics review before entering legal |

Positioning should change by segment and avoid leading with "AI." Owners over 65 use AI at about half the rate of owners under 35 ([Contractor Magazine](https://www.contractormag.com/technology/article/55294441/more-than-70-of-home-service-pros-use-ai-to-cut-admin-work-but-not-field-jobs)). For owners aged 55+, sell **"owner-independence / exit-readiness playbook."** For growth shops of 10–50 people, sell **"find the revenue leaking from your phones and dispatch board."** For high-churn shops, sell **"never retrain from scratch."**

## Next steps: validate before building

**The Reddit gap is open.** Reddit (WebFetch, JSON API and `site:` search) was blocked from the research environment, so this report cites **no Reddit threads**. Run these searches by hand:

| Subreddit | Searches |
|---|---|
| r/smallbusiness, r/Entrepreneur | "tribal knowledge", "only one who knows", "office manager quit", "document my business", "operations manual", "SOP" + "too long" |
| r/sweatystartup | "SOP", "step away", "vacation business runs without me", "hire office manager" |
| r/HVAC, r/HVACadvice, r/Plumbing, r/electricians | "dispatcher quit", "CSR", "office manager left", "ServiceTitan" + "only person" |
| r/FieldService, r/ServiceTitan | "ServiceTitan admin", "setup only one person understands", "onboarding office staff" |
| r/EOS, r/Entrepreneur | "process documentation", "Trainual", "core processes" |
| r/LawFirm, r/Lawyertalk, r/paralegal | "intake process", "paralegal left", "firm procedures manual" |
| r/AI_Agents, r/automation, r/msp | "discovery call", "AI audit", "interview client staff", "process discovery" |
| Any of the above | "tool that interviews employees", "AI write SOP from talking", "record how employees do their job" |

**Validation plan (about 6 weeks)**

| # | Test | Pass threshold | Owner/cost |
|---|---|---|---|
| 1 | Run the Reddit sweep above; log every first-person pain post | ≥15 relevant posts from trades/SMB owners in the last 2 years | 3–4 hrs |
| 2 | 10 owner problem interviews (Tampa HVAC/plumbing, 10–50 staff) via RACCA/MACCA ([ACPro](https://acprosite.com/fracca-announces-2025-officers-and-board-of-directors/)) and the Tampa Bay Chamber | ≥5 name a specific seat that would break if its person left | 2 weeks |
| 3 | Build a manual concierge MVP: Claude voice/chat interview script for CSR, dispatcher and office manager; consent script; report template | Usable report from one 30-min interview per role | 1 week |
| 4 | Sell **3 paid pilots** at $2,500 (founder discount from $3,500), credited toward implementation | 3 signed within 30 days of offering | Direct outreach |
| 5 | Measure pilot outcomes: interview completion rate, staff sentiment, automation items found, conversion to build | ≥1 of 3 converts to implementation; staff completion ≥80% | Pilot period |
| 6 | Price test in pilot proposals: "exit-readiness" vs "phone/dispatch revenue" vs "onboarding manual" | One framing wins a clear majority of replies | Ongoing |
| 7 | Channel test: pitch a "map your workflows, then automate" workshop to SBDC Hillsborough ([SBA](https://www.sba.gov/event/83442)) and one RACCA/MACCA meeting | 1 booked slot; ≥3 leads per event | 4 weeks |
| 8 | Florida counsel review of the consent script and employee notice; separate bar-ethics check for law firms | Approved script before any recorded interview | One-time legal fee |
| 9 | After 5+ case studies: offer the white-label version to 3 AI agencies or EOS implementers | 1 paying agency at ≥$299/mo | Month 3+ |

## Conclusion

The opening exists because current tools split the problem in two. Enterprise tools interview staff but cost too much, and SMB tools are cheap but make the owner write everything. Ontora shows the interview-to-automation loop has investor backing, and Probook shows the dispatch seat this would document is already being automated in Florida. The work could stay a commodity of documentation, or become a funnel for automation. That depends on two things MacTutor can own: **trades-specific role templates connected to the actual FSM stack**, and **consultant review that turns the interview output into a priced build backlog**. Lead with the done-for-you diagnostic, earn local proof, then turn the method into a white-label product for agencies. Until the Reddit sweep and pilot sales confirm direct demand, treat the SaaS tier as a later option, not the launch product.
