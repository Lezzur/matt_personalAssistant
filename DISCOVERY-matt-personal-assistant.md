# Discovery Brief — Matt Personal Assistant

**Date:** 2026-05-07
**Participants:** Rick (product owner), Lisa Hayes (lead engineer / coordinator), Tony Stark (engineering), Light Yagami (strategy), Nami (sales), Jony Ive (design — earlier ideation)
**Status:** Draft
**Confidence:** Medium — core scope, architecture, and integration set are settled. Build ownership intentionally deferred per Rick. A few client-facing details (iOS vs Android, current task tool, exact contact-VIP list) remain to be collected during onboarding, not before the spec pipeline starts.

---

## 1. Problem Space

Matt — a working professional running multiple projects (incl. studio rental responsibilities) — is losing time and credibility to delivery-layer failures around tools he already owns. His Google Calendar is in place, but it doesn't reach him at the right moment, in the right channel, with the right context. He misses appointments, forgets recurring commitments (studio rent due, PT Tuesdays), and ends meetings without capturing follow-ups. The pain isn't a missing feature in any one tool — it's that no system ties his calendar, communications, recurring obligations, and meeting outputs together and pushes the right thing to him at the right time.

- **The problem:** Matt's existing tools (Google Calendar, email, phone) hold the right information but fail at delivering it to him when and how he needs it. Important things slip not because they're unscheduled, but because the reminder layer is too passive.
- **Who experiences it:** Matt — solo professional with concurrent commitments (creative work + studio overhead + meetings + recurring bills) — and, by extension, Rick's eventual second/third clients with similar profiles.
- **Current workarounds:** Native Google Calendar reminders (insufficient — single-channel, no escalation, no context). Phone alarms. Manual SMS to himself. Sticky notes. None integrate, none escalate, none survive context-switching.
- **Why now:** Matt explicitly asked for "things quietly working in the background." He has tried tools and stopped — the gap is real and validated. Rick has a paying client commitment.
- **Evidence quality:** Settled — directly from Matt's questionnaire response (collected 2026-05-07).

## 2. Users and Context

### Primary User — Matt
- **Who he is:** Working professional juggling creative output, a studio with recurring overhead, client meetings, and personal commitments. Comfortable with technology but fatigued by tool-switching. Self-described preference: "assistance, not blind delegation."
- **Context of use:** Mostly mobile (texts and calls during the day), checks email between meetings, uses a laptop for creative work in studio time blocks. Wants the assistant to reach him *where he already is*, not require opening a new app.
- **What success looks like:** Three months in, Matt isn't missing appointments, his recurring obligations get handled without nagging him repeatedly, and his post-meeting follow-ups land in the right place automatically. He stops thinking about the system because it stops failing him.
- **What makes him leave:** A "blind delegation" surprise — the assistant acting on his behalf in a way he wouldn't have approved. Or: a unified experience that turns into five logins and five apps (which is why the off-the-shelf Concierge path was rejected).

### Secondary Users (v2+)
- **Rick's next 1–2 paying clients** with a similar professional profile. The architecture must support them, but the v1 product is built for and around Matt's workflow.

### Anti-Users
- **Self-serve SaaS users.** This is not a sign-up-and-go product. It's a managed, configured experience.
- **Power users who want to script their own automations.** v1 does not expose an automation editor.

- **Evidence quality:** Settled (Matt) / Leaning (secondary users — based on Nami's white-label commercial framing, no second client signed yet).

## 3. Proposed Solution Shape

A **managed, multi-tenant-architected assistant** that sits as a delivery layer over Matt's existing accounts (Google Calendar, Gmail), reaches him through SMS-first channels (with a unified dashboard as a secondary surface), and consumes Neurocore's memory + voice APIs for personalization. Rick configures and hosts the instance; Matt experiences a single, coherent assistant rather than a collection of glued-together SaaS tools.

- **Product type:** Web app + SMS interaction surface. Single deployable Node/TypeScript application.
- **Core interaction model:** Inbound — assistant pushes timed reminders, briefings, and post-meeting summaries via SMS; surfaces a dashboard for review and history. Outbound — Matt can text the assistant in plain language ("remind me to pay studio rent at 5pm tomorrow", "DONE") and it acts.
- **Key differentiator vs Concierge tier:** One coherent experience, one place to look, drafted-in-his-voice communications (via Neurocore). Concierge would have given Matt 5 separate apps with 5 separate logins; this gives him one assistant.
- **Delivery model:** White-label per-client managed service. Rick deploys and configures one instance per paying client, hosts on his Coolify infrastructure, charges a monthly retainer. Not self-serve. Not multi-customer-per-instance.
- **Neurocore relationship:** This product is the *delivery layer* over Neurocore. Neurocore owns memory, voice modeling, relationship/VIP inference. The assistant consumes `/v1/memory/context` (and related Neurocore APIs) to personalize drafts and prioritize interruptions. The two products ship in parallel, with the integration boundary as a hard interface contract.
- **Evidence quality:** Settled — confirmed by Rick on 2026-05-07.

## 4. Prior Art and Competitive Landscape

### Direct Competitors

| Product | Strengths | Weaknesses | Relevance |
|---------|-----------|------------|-----------|
| Reclaim.ai | Strong calendar intelligence, habit/buffer logic | Single-channel (in-app), no cross-tool integration, no draft-in-your-voice | We use it as a tool inside the Concierge tier; for Custom we replicate the calendar-intelligence pieces we need natively |
| Motion | Calendar + tasks integrated, AI scheduling | Subscription model, single-app surface, no SMS-first delivery, no managed onboarding | Matt would still need a delivery layer on top |
| Superhuman / Shortwave | Email speed and intelligence | Email-only, no cross-channel push, no calendar handling | Solves a sliver of the problem |
| Granola / Otter | Meeting transcription + action item extraction | Single-purpose, separate app, doesn't push to other systems without glue | Used as a data source in v2; not v1 |
| Calendly / Cal.com | Scheduling links | Not assistive — purely transactional | Out of scope; Matt can keep using whatever scheduling link he has today |
| ChatGPT/Claude as assistant | Conversational interface, draft quality | No integrations, no proactive push, no persistence between sessions | Useful as a model provider; not a competing product surface |
| Human EA / virtual assistant | Highest quality, full delegation | Cost ($3–5k/mo), discovery latency, no 24/7 | Different price point. Matt has not used one. |

### Adjacent / Indirect Solutions
- **Zapier / Make.com automations** — what the Concierge tier would have used. Power exists, but configuration/maintenance burden falls on the user (or the agency). Replaced in v1 by an in-house automation runtime (see Section 5).
- **iOS Focus modes / VIP contacts** — solves part of the interruption problem natively, but Matt has to configure it himself and it doesn't coordinate with calendar/email signal.

### Key Takeaways
- **No single product on the market plays the delivery-layer-over-existing-accounts role we're building.** The closest analogs (Motion, Reclaim) own the calendar but don't reach Matt where he already is.
- **The market gap is "managed assistant"** — between $20/mo SaaS tools and $3–5k/mo human EAs sits an under-served segment willing to pay $200–500/mo for a configured, supported experience.
- **Avoid the Zapier trap.** Off-the-shelf glue is fragile and visible to the user when it breaks. v1 builds the automation runtime in-house.

- **Evidence quality:** Leaning — based on team knowledge as of 2026-05-07. No formal user research beyond Matt's questionnaire.

## 5. Technical Feasibility

- **Platform/architecture direction:** Single Node/TypeScript service, multi-tenant data model (`users/{uid}/...` document tree, all queries scoped to uid), single-tenant deployment for v1 (one Coolify host, low-N installs per instance per Rick — max ~2–3 active tenants on a single host before standing up a second). Per-user OAuth for Google. Twilio for SMS in/out with webhook signature verification. Web dashboard served from the same app.
- **Key technical constraints:**
  - SMS reminder timing requires sub-2-min trigger latency for the 5-min-before-event reminder. BullMQ or node-cron with a tight poll, not Zapier-style 15-min batch.
  - Twilio inbound webhook signature verification is non-negotiable (signed-URL path; can't be skipped on a public webhook).
  - Per-user Google OAuth tokens require secure storage and refresh handling.
  - Neurocore integration is over HTTP — `/v1/memory/context` and related endpoints. Network failures must degrade gracefully (assistant still functions without personalization).
- **Known hard problems:**
  - **Interruption gradient logic.** "Read the room — don't interrupt during a meeting" requires correlating calendar state, presence signals, and message priority. v1 will use rule-based heuristics (calendar busy + non-VIP sender = hold). Smarter ML version is post-v1.
  - **Recurring obligation tracking with reply-back loop.** "Studio rent due — reply DONE when paid" requires a per-task state machine, escalation rules, and inbound SMS routing back to the right task. Solvable but it's the highest-touch piece of the build.
  - **Voice-modeled email/message drafts.** Depends entirely on Neurocore's voice cache being live. v1 may ship with generic drafts and upgrade once Neurocore Phase 4 lands.
- **Technology preferences (settled):** Node + TypeScript, Express or Fastify, Firestore for data, BullMQ or node-cron for scheduling, Twilio SDK, Google APIs (Calendar + Gmail), OpenWeatherMap (or equivalent free tier) for weather. Hosted on Rick's existing Coolify instance.
- **Evidence quality:** Settled — Tony Stark engineering review confirmed feasibility on 2026-05-07.

## 6. Scope and Boundaries

### In Scope (v1)

**Core capabilities:**
- Calendar-driven SMS reminders with escalation (1h, 15m, 5m before events)
- Travel-time-buffered calendar awareness (Google Maps API for ETA at scheduling time, not real-time live traffic — Tony's correction)
- Recurring obligation tracking with reply-back loop ("studio rent due — reply DONE")
- Morning briefing SMS (calendar + weather + flagged emails) at a configurable time
- Inbound natural-language SMS commands ("remind me to X at Y", "DONE", "snooze 30m")
- Priority-aware interruption (VIP whitelist, calendar-aware quiet hours, configurable Do Not Disturb windows)
- Web dashboard: today's schedule, pending reminders, recent SMS history, action log ("what did the assistant do for me today")
- Per-user OAuth: Google Calendar + Gmail
- Multi-tenant data architecture, single-tenant deployment

**v1 integration set (confirmed by Rick 2026-05-07):**
- Google Calendar
- Gmail
- Twilio (SMS in/out + 1 phone number)
- Weather API (OpenWeatherMap or equivalent)

**Neurocore integration (parallel build):**
- Consume `/v1/memory/context` for VIP/relationship signal
- Consume voice cache (when available) for draft personalization

### Explicitly Out of Scope (v1)

- **Self-serve signup and billing.** White-label managed service — Rick configures each instance manually.
- **Public marketing site, brand identity assets, OG images, paid ads landing pages.** This is an internal/B2B product; no consumer marketing surface.
- **Mobile app (native iOS / Android).** Web dashboard + SMS is sufficient for v1. Push notifications are deferred until there's signal a native app is needed.
- **Voice surface (smart speaker / phone call interaction).** Discovery-era idea; not v1.
- **Financial integrations (bank balance, bill payment).** Matt's questionnaire mentioned them; legal/security overhead is too high for v1.
- **Real-time live-traffic re-routing.** Buffer-based travel time only.
- **ML-driven interruption gradient.** Rule-based for v1.
- **Email auto-send.** v1 drafts but does not autonomously send. Matt approves.

### Deferred (v2+)

- **Notion integration** — task list destination. Useful but not required if v1 dashboard handles task display.
- **Otter / Granola integration** — meeting transcription + action item extraction. Strong feature, deferred to keep v1 scope tight.
- **Slack / Teams DM channel** — reach Matt via Slack as a channel option. Deferred until SMS-first proves out.
- **Market data in morning briefing** — nice-to-have; defer.
- **Multi-tenant *deployment*** (admin UI, tenant switching, billing scaffolding) — architecture supports it; UI/billing build deferred to client #3+.
- **Native mobile app** — web works for now.
- **Email/document drafting in Matt's voice (full Neurocore voice cache integration)** — depends on Neurocore Phase 4 timing.

- **Evidence quality:** Settled — explicit confirmation from Rick on 2026-05-07 for the v1 integration set; team consensus on the rest.

## 7. Constraints and Risks

### Hard Constraints

- **Neurocore is a parallel dependency, not a sequential one.** Tony Stark owns Neurocore Phase 1+2 starting 2026-05-10 and cannot also own this build's implementation. Build ownership for the assistant is being directed by Rick separately (see Decisions Log D-7).
- **Tony's capacity:** Tony will not be primary engineer on this. He will be available for tech spec authorship, security-critical code review (OAuth flows, Twilio signature verification, Neurocore integration boundary), and general architectural guidance. Implementation bulk is intended for Barker-orchestrated agent build, but Rick has explicitly reserved that decision for himself (see D-7).
- **Hosting:** Rick's existing Coolify instance. No new infrastructure budget for v1.
- **Single-tenant deploy, max ~2–3 installs/instance.** Architecture is multi-tenant; deployment density is intentionally low.
- **Budget posture:** Matt's monthly all-in (Rick's retainer + tooling) was scoped at the Concierge tier ($200/mo retainer + ~$80–100/mo tools). Custom tier pricing has not been finalized but is expected to be higher; Nami to lead the proposal.

### Key Risks

| Risk | Likelihood | Impact | Mitigation Discussed |
|------|-----------|--------|---------------------|
| Neurocore integration boundary slips, blocking personalization features | Medium | High | Make personalization optional in v1 — assistant degrades gracefully to generic drafts/rules if Neurocore is down or behind schedule. Hard interface contract authored before parallel work begins. |
| Twilio webhook signature verification missed → spoofable inbound SMS | Low | High | Tony Stark personally writes/reviews this code path. Not eligible for Barker-only build. |
| Per-user Google OAuth token storage compromised | Low | High | Tokens at rest encrypted; tokens in transit only over HTTPS; Tony Stark reviews OAuth implementation. |
| Reminder timing latency (cron poll misses 5-min window) | Medium | Medium | BullMQ with second-resolution scheduling for time-sensitive jobs; node-cron for coarse periodic work. Tested under load before Matt onboards. |
| "Studio rent" reply-back loop becomes most complex feature in product | High | Medium | Build it as a generic "task with escalating reminder + reply-back close" primitive; reuse for any recurring obligation, not Matt-specific. |
| Matt's existing task tool (Notion? Todoist? "Nothing"?) is unknown — affects post-meeting flow | High | Low | Confirm during onboarding session. v1 dashboard provides task display natively, so no external task tool is required. Notion integration deferred. |
| Matt's device — iOS vs Android — affects Focus/DND playbook | Certain | Low | Confirm during onboarding session. Doesn't affect build, only the configuration playbook handed to him. |
| Build ownership undefined — "parallel" without a second engineer named | Medium | High | Rick explicitly reserved this decision (D-7). Pipeline halts at tech spec stage if this isn't resolved before then. Lisa to flag at the spec-stage handoff. |
| Custom tier pricing hasn't been quoted to Matt; he saw Concierge numbers | Medium | Medium | Nami to lead pricing conversation before tech spec is shared. Pricing is upstream of the build, not blocking discovery. |

### Open Questions

| # | Question | Owner | Depends on | Status |
|---|----------|-------|-----------|--------|
| OQ-1 | Does Matt use iOS or Android? Affects the Focus mode / VIP-contact onboarding playbook, not the build. | Rick (collect at onboarding) | Onboarding session | Resolved — non-blocking; chosen default is "we ask in onboarding session before configuration." Does not block any spec. |
| OQ-2 | What task tool, if any, does Matt currently use? Affects whether v2 needs a Notion/Todoist integration or whether the v1 dashboard task list is sufficient. | Rick (collect at onboarding) | Onboarding session | Resolved — non-blocking; v1 ships with native task display, external task tool integration deferred to v2 regardless of answer. |
| OQ-3 | Custom tier pricing — exact monthly retainer + setup fee for Matt. | Nami | Pricing conversation between Nami and Rick | Resolved — non-blocking for discovery; pricing finalizes before tech spec is shared with Matt, not before discovery completes. Nami owns. |
| OQ-4 | Will Rick onboard a 2nd or 3rd client during v1 build window, triggering early multi-tenant deployment work? | Rick (sales pipeline) | Sales activity | Resolved — non-blocking; multi-tenant *architecture* is in v1 regardless, multi-tenant *deployment* (admin UI, signup) only triggers if a 2nd client signs and only then is added to the spec. |

## 8. Key Decisions Log

| Decision | Rationale | Made By | Date |
|----------|-----------|---------|------|
| **D-1: Build a custom assistant, not Concierge tier (off-the-shelf glue).** | Matt's response made clear that 5 separate apps would re-create the friction he's trying to escape. He wants "quiet, in the background" — that's a unified experience, not a tool stack. | Rick | 2026-05-07 |
| **D-2: Multi-tenant architecture, single-tenant deployment for v1, max ~2–3 installs/instance.** | Tony's hybrid framing: data model and OAuth scoping built multi-tenant from day one (~3-day cost), deploy single-tenant (~4 weeks saved on signup/billing/admin UI). Avoids ~4-week rewrite at v2. | Rick (confirming Tony's recommendation) | 2026-05-07 |
| **D-3: Neurocore is the brain; this product is the delivery layer.** | Avoids re-implementing memory/voice/relationship-inference. Both products parallel-build with `/v1/memory/context` as a hard contract. | Tony Stark, Rick | 2026-05-07 |
| **D-4: v1 integration set — Google Calendar, Gmail, Twilio, weather.** | These are the day-1 must-haves for the capabilities the team committed to. Notion, Otter/Granola, Slack, market data deferred. | Rick (confirming Tony's proposal) | 2026-05-07 |
| **D-5: Build the automation runtime in-house, not Zapier.** | 8 specific automations × 30–80 lines each = ~2–3 weeks of engineering. Replaces $49/mo Zapier Pro per client; becomes the engine for the future product line. | Tony Stark, Light, Rick | 2026-05-07 |
| **D-6: SMS-first delivery surface; web dashboard secondary; no native mobile app, no voice surface in v1.** | Matt's stated preference is "reach me where I am." SMS is universal; dashboard is where he reviews what happened. Native mobile / voice is post-v1 unless validated. | Lisa, Tony, team consensus | 2026-05-07 |
| **D-7: Build ownership deferred — Rick directs Tony directly as work arises.** | Rick declined to lock the build-team configuration in the discovery brief. This is intentional. The discovery brief notes the decision and flags it as a gate for the tech spec stage. | Rick | 2026-05-07 |
| **D-8: White-label per-client managed service, not self-serve SaaS.** | Pricing language ("discussed individually", "monthly contracts") and the configuration-heavy nature of the product point to white-label. Self-serve signup is deferred indefinitely. | Nami, Rick | 2026-05-07 |
| **D-9: No public marketing site, brand identity, or OG assets in scope.** | This is a B2B managed service — no consumer marketing surface in v1. Brand tokens / typography / OG images are explicitly N/A for the discovery stage and downstream. | Lisa | 2026-05-07 |
| **D-10: Repository — `Lezzur/matt_personalAssistant`.** | Confirmed via Lezzur GitHub org. Empty as of 2026-05-07; this brief is the first commit. | Rick | 2026-05-07 |
| **D-11: Hosting — Rick's existing Coolify instance.** | Same infrastructure as Neurocore. No new ops surface, no new cost. | Tony Stark | 2026-05-07 |

## 9. Recommendation

- **Proceed to specs?** **Yes with caveats.**
- **Caveats:**
  1. **Build ownership (D-7) must be resolved before the tech spec stage exits.** The tech spec can be authored by Tony regardless, but the build plan / Barker plan cannot be written until Rick names who is implementing. Lisa (this agent) will flag this at the spec handoff.
  2. **Custom tier pricing (OQ-3)** must be finalized by Nami before the tech spec is shared with Matt. Discovery is not blocked by it; client communication is.
  3. **Neurocore interface contract** must be authored as a separate artifact (or as part of the tech spec) before parallel implementation begins. The two products will collide if the API shape is left implicit.
- **Suggested spec focus:** **Heavy on Tech Spec.** The product complexity is concentrated in the automation runtime, integration set, OAuth + webhook security, and Neurocore boundary. PRD is shorter than usual (capabilities are well-known). UI Design is minimal — single dashboard, no marketing surface, no brand work. API Spec covers the Neurocore boundary and any internal endpoints the dashboard hits.
- **Suggested timeline pressure:** **Moderate — not urgent, not lazy.** Matt has already waited; Rick has signaled commitment. Tony's 5-week v1 estimate is from start of implementation, not from today. Discovery → PRD → Tech Spec → Build Plan should be tight (target: PRD this week, Tech Spec next week, Build Plan ready for kickoff aligned with whatever build-ownership decision Rick lands on).

---

*Source artifacts:*
- *Matt's questionnaire response (collected 2026-05-07; analyzed in chat 2026-05-07)*
- *Team chat 2026-05-07 — Rick, Lisa, Tony, Light, Nami, Jony Ive, Dee*
- *Original questionnaire: `QUESTIONNAIRE-personal-assistant.md` (and `-lite.md`) in `/workspaces/lisa/personal-assistant/`*
- *Companion product: Neurocore — see `Lezzur/neurocore` for the brain-side spec*
