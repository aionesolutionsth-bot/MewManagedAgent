# AIOn3 — Personal AI Agent (Managed) · Product Requirements Document

**Status:** Draft v0.1
**Owner:** AIOn3 Technology
**Last updated:** 2026-06-24

---

## 1. Summary

AIOn3 deploys and operates an **AI growth team** for each client, built on the
open-source **Hermes Agent** (Nous Research). The client gets a **Chief of Staff**
that lives where they already are — primarily **WhatsApp** — and coordinates a
team of specialist agents (marketing/content, CRM/relationships, research,
content production) that work proactively to **find leads and grow the
business**: capturing and qualifying inbound DMs, following up and nurturing
relationships, producing and repurposing content, and running market/competitor
research.

The client never touches a terminal, credits, or model configuration. AIOn3
provisions an **isolated Hermes instance per client**, wires up their
integrations, sets up their recipes, and keeps it running. The **client portal**
is the trust-and-visibility layer: a place to see what the team did, what it
knows, and what it can touch.

**Target client (validated by discovery):** solo founders, agents, and
salespeople (e.g. real estate + insurance) who want more leads and sales but
can't build or run an AI system themselves. See `discovery-call-mew` notes.

**One-line positioning:** *"An always-on AI growth team that finds you leads and
grows your business — deployed, integrated, and run for you by AIOn3, right
inside WhatsApp."*

---

## 2. Why this, why now

- **Hermes does the hard agent work** — multi-channel (WhatsApp/Telegram/Signal/
  email/etc.), persistent cross-channel memory, natural-language scheduling,
  multi-agent subagents, web/browser/vision/image/TTS, sandboxed code execution.
- **But Hermes is a developer tool** — terminal install, sandboxed code, Python
  RPC subagents, BYO model credits, 300+ model picker. No normal person will run
  it.
- **AIOn3's product is the managed abstraction layer** that turns a developer's
  CLI agent into a consumer appliance: provisioning, hosting, integration wiring,
  credit management, security hardening, and a friendly portal.

### Moat / defensibility
The Hermes runtime is a commodity (MIT-licensed, free). AIOn3's durable edge is
the managed layer. Every feature should ladder up to one of these three things
that raw consumer AI apps don't do for a normal person:
1. **It's set up and run for them** (no terminal, no config, no credits).
2. **It's proactive and persistent** across their real accounts and channels.
3. **It has durable, editable memory** of their life.

If a feature doesn't serve one of these three, it's probably out of scope.

---

## 3. Target user

**Primary:** Solo founders, agents, and salespeople — non-technical operators
whose income depends on **leads and sales** (real estate, insurance, coaches,
consultants). They have a personal brand and a warm network, sell via DMs/
WhatsApp, and lack any real CRM or content system. They'll pay to *not* have to
build or run an AI growth system themselves. (First validated client: "Mew" —
real estate + insurance; see `discovery-call-mew`.)

**Explicitly not (for now):** developers (they'd self-host the MIT project),
large enterprises (different portal: SSO, admin roles, compliance), and broad
general-purpose consumers (the wedge is *business growth*, not life-admin — the
discovery call showed the monetizable need is leads/sales, not generic
assistance).

### Anti-goal
The moment marketing targets technical users, the value proposition evaporates.
Keep the surface non-technical: hide terminals, RPC, model pickers, deployment
backends. Likewise, don't dilute into a generic life-assistant — every feature
should ladder to *more leads / more sales / stronger relationships*.

---

## 4. Product principles

1. **WhatsApp is the product; the portal is the control room.** Do not build a
   daily chat UI in the portal that competes with WhatsApp.
2. **Proactivity over responsiveness.** The agent reaches out; the differentiator
   vs. a chatbot is that the client doesn't have to initiate.
3. **Manufacture value fast.** Onboarding ends with live recurring jobs running,
   not just connected accounts. Beat the blank-canvas problem with recipes.
4. **Trust is a feature.** Visible memory, plain-English activity logs, and clear
   "what it can touch" are first-class, not afterthoughts.
5. **Hide the machine.** Subagents, code execution, RPC, model routing, credits —
   all operator concerns. Never leak them into the consumer surface.
6. **Provider-agnostic.** The model provider must be swappable behind an
   abstraction. No hardcoded provider assumptions in the portal.

---

## 5. Architecture overview

```
                ┌─────────────────────────────────────────────┐
   WhatsApp ───▶│  Hermes Agent instance (ISOLATED per client) │
   Telegram ───▶│   • cross-channel memory store               │
   Email    ───▶│   • scheduled jobs (recipes)                 │
                │   • subagents / sandboxed execution          │
                │   • integration tools (OAuth)                │
                └───────────────┬─────────────────────────────┘
                                │ model calls (routed by task tier)
                                ▼
                    OpenRouter (default) ── swappable ──▶ direct provider keys
                                │
                ┌───────────────┴─────────────────────────────┐
                │  AIOn3 control plane                         │
                │   • provisioning / lifecycle (Modal)         │
                │   • credit metering + spend caps             │
                │   • integration / OAuth broker               │
                │   • portal API (read-only in Phase 1)        │
                └───────────────┬─────────────────────────────┘
                                ▼
                          Client Portal (web)
```

### Key decisions (locked)
- **Isolation model:** one isolated Hermes instance per client (Modal backend).
  Rationale: Hermes runs arbitrary sandboxed code per client; the blast radius of
  shared code execution across consumers isn't worth the cost savings, and
  per-client isolation makes the "your agent is yours" privacy story honest.
- **Model provider:** **OpenRouter** as default (single key, 300+ models, hot-swap,
  fallback, ~5% markup). Migrate heavy/stable workloads to direct provider keys
  later. **Route by task tier:** cheap/fast model for constant background
  monitoring & triage; strong model only when a recipe does real work. This
  routing discipline protects flat-subscription margins.
- **Daily surface:** WhatsApp (native to Hermes). Portal is secondary.
- **Desktop app:** Hermes now ships a desktop app — relevant only to a possible
  future "bring your own machine" tier; not part of the managed cloud model.

### Open architecture questions
- Per-tenant credit metering granularity and cap-enforcement mechanism.
- Sandbox hardening standard per tenant (security ownership).
- Contingency if Nous changes credit pricing or the project stalls (MIT means we
  *can* self-host the runtime, but still need model providers — OpenRouter
  cushions this).

---

## 6. Go-to-market phases

### Phase 1 — White-glove (current)
- AIOn3 manually provisions instances, does OAuth wiring, sets up recipes per
  client.
- **Portal is READ-ONLY:** a window for the client to see value and build trust.
  Clients request changes through AIOn3 (human in the loop).
- Goal: learn what clients actually toggle, which recipes stick, where they panic
  — *before* building self-serve controls.

### Phase 2 — Self-serve (future)
- Automated provisioning + lifecycle (Modal, per-tenant isolation).
- One-click channel linking (WhatsApp QR), self-serve recipe gallery, guardrail
  controls, billing/credit metering, pause/cancel/delete + data wipe.
- Portal becomes a control plane. **GUI route** for self-serve configuration.
- Most engineering effort lives here.

---

## 7. Portal scope

### Phase 1 (read-only) — surfaces
All data is read-only; mutations happen via AIOn3 (white-glove).

| Surface | Purpose | Notes |
|---|---|---|
| **Overview / Dashboard** | "Is my agent healthy & what's it been doing?" | Status, last active, activity summary, value-proof stats (tasks handled, time saved). |
| **Activity** | "What did it do while I was away?" | Plain-English log of agent + subagent actions. Translated from Hermes execution logs. The retention/value-proof surface. |
| **Memory** | "What does it know about me?" | View persistent memory (preferences, facts, contacts). Read-only in P1; edit/forget in P2. Delight + #1 trust/privacy anxiety. |
| **Channels** | "Where can I reach it / where does it live?" | Connected channels (WhatsApp, etc.) with status. Linking is one-click in P2. |
| **Recipes** | "What recurring jobs are running for me?" | The agent's scheduled behaviors (morning brief, inbox bodyguard, topic watch). Toggle/customize in P2. |
| **Integrations** | "What can it touch?" | Connected tools, simplified — "Connected ✓ / Not connected", no permission-scope tables. |
| **Usage & Plan** | "What am I on, what have I used?" | Plan, usage abstracted to "tasks"/"$" (never raw tokens), uptime. |

### Deliberately NOT exposed (ever, to consumers)
Subagent terminals · Python RPC · model picker · deployment backend · raw
sandbox · raw token/credit math.

### Phase 2 additions
Recipe gallery (toggle pre-built behaviors) · memory edit/forget · one-click
channel linking · autonomy/guardrail controls (default autonomy LOW; grows with
trust) · approvals queue (drafts → ask → auto, leash loosens over time) · spend
caps · self-serve billing · data export & deletion.

---

## 8. Recipes (the value engine)

Recipes are friendly wrappers over Hermes's natural-language scheduled jobs. They
solve the blank-canvas problem for a general-purpose agent by giving the client
concrete, toggleable behaviors.

Launch recipe ideas (growth-team):
- **Daily growth brief** — morning rundown: new leads, hot DMs, content & market signals.
- **Lead catcher** — watch IG/WhatsApp DMs, qualify inbound, draft first replies (sends nothing without approval in P2).
- **Relationship keeper** — birthdays, referral asks, and re-engaging dormant leads (the CRM engine).
- **Content engine** — ideas, hooks, captions, and waterfall repurposing to TikTok/YT/X.
- **Competitor & market watch** — track competing agents and demand signals in the client's markets.
- **Ad targeting builder** — build lookalike audiences & suggest budget once data is rich.

Each recipe should map to: a schedule (or trigger), a task-tier (cheap monitor vs.
strong execution), required integrations, an owning cast member, and an autonomy
level.

---

## 8.5 Character & persona system (hero + specialist cast)

The agent is **personified as a named character** with an original Pixar-style 3D
avatar and a personality — not "the AI." This is a retention and trust mechanism,
not decoration: consumers form daily habits with *someone*, and a consistent
friendly persona makes handing over inbox/calendar feel safe.

**Model (locked):** **one hero + specialist cast.**
- The client bonds with and only ever talks to **one hero character** (e.g.
  "Aria") — the face in WhatsApp that holds the relationship and the memory.
- For specialist jobs the hero "brings in" a **cast of specialist
  sub-characters** — a CMO (marketing/content), COO (CRM & relationships),
  researcher (market/competitor intel), and content producer. Under the hood
  these are **Hermes subagents** — this turns Hermes's most technical feature
  (RPC subagents) into a delightful, consumer-friendly story instead of a leaked
  terminal. (This is exactly the Chief-of-Staff-plus-CMO/COO team pitched in the
  discovery call.)

**Onboarding hook:** a "meet your agent" moment (name them, pick a personality,
see their face) drives activation — the client bonds *before* the first task.

**In the portal:** the hero anchors the Overview banner (avatar, name,
personality, status); a "{hero}'s team" card introduces the cast; the activity
feed attributes each action to the character who did it ("Vela drafted 3
replies").

**IP guardrail:** create **original** characters in the 3D-stylized look. The
aesthetic is free to use; specific Disney/Pixar characters, names, and branding
are not — never reference the brand. Lock a consistent render/style template
(same lighting, neutral backdrop) so the cast reads as one family.

## 9. Pricing (instinct, to validate)

- Flat monthly subscription with usage guardrails. Prosumer tier ~$20–40/mo.
- Higher **"concierge"** tier: white-glove setup + custom recipes (Phase 1 is
  effectively all concierge).
- **Cost trap:** an always-on, proactive, multi-agent, code-running agent burns
  model credits spikily and unpredictably per user. Flat price sits on variable
  cost → **credit metering + caps are required from day one**, even in white-glove.

---

## 10. Risks

| Risk | Mitigation |
|---|---|
| Arbitrary code execution at consumer scale (security/abuse surface) | Per-tenant isolation (Modal); sandbox hardening standard; curated capability subset. |
| Credit/cost runaway under flat pricing | Task-tier routing; per-tenant metering + hard caps from day one. |
| Dependency on Nous (pricing, roadmap, v0.17 maturity) | OpenRouter for model independence; MIT license allows self-hosting runtime; monitor Nous roadmap. |
| "Why not DIY?" (Hermes is free + MIT) | Target stays non-technical; value is *managed*, not the runtime. |
| Competing with free/cheap first-party consumer AI apps | Wedge = managed + proactive/persistent across real accounts + durable editable memory. |
| Blank-canvas churn (general-purpose agent) | Onboarding installs live recipes; proactivity over responsiveness. |

---

## 11. Success metrics (proposed)

- **Activation:** % of clients with ≥2 live recipes after onboarding.
- **Retention:** D30 / D90 active (agent sent ≥1 proactive message/week).
- **Trust:** % who viewed Memory; % who loosened autonomy over time (P2).
- **Value proof:** self-reported time saved; tasks handled/week.
- **Unit economics:** model cost per active client vs. subscription price (margin).

---

## 12. Phase 1 deliverable (this build)

A polished, **read-only client portal** matching the AIOn3 OS design system,
populated with realistic mock data for a sample prosumer client. Surfaces:
Overview, Activity, Memory, Channels, Recipes, Integrations, Usage & Plan.

Built as a self-contained web app (consistent with the existing `index.html`),
structured so the mock data layer can later be swapped for a real control-plane
API. See `portal.html`.
