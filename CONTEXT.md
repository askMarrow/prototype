# Marrow — Product & Design Handoff
*For continuation in a new chat. Last updated: 2026-06-10*

---

## What Marrow Is

An AI-powered product management platform that replaces/automates the Product Manager role for SaaS scale-ups (15–100 people). It ingests scattered signals — customer calls, support tickets, emails, CRM data, engineering commits — and outputs three things:

1. **What to build next** — AI-prioritized feature backlog
2. **How to tell the org** — role-targeted internal Slack updates
3. **How to tell customers** — external newsletter / LinkedIn post

Tom Müller (revel8, our most important interview): *"AI powered product org for your business. You basically don't need a PM anymore."*

---

## Files on Disk

| File | What it is |
|---|---|
| `~/Downloads/marrow-prototype-v1.html` | Current working prototype (Option A design, bliro mock data) |
| `~/Downloads/marrow-config.js` | API key config — paste OpenAI key here |
| `~/Downloads/marrow-designs.html` | Three original design concepts (A/B/C) for reference |
| `~/Downloads/Marrow.svg` | Official logo — accent color is `#7AF17A` |

---

## Design Direction (DECIDED)

**Target aesthetic: OpenAI Platform console** (platform.openai.com)

Key traits to replicate:
- Very dark background (#0D0D0D), near-black surfaces (#111)
- Narrow left sidebar: project selector top-left, icon+label nav items, user avatar bottom-left
- Large page heading ("Home", "Usage" etc.) in the content area
- Clean card panels with thin borders, lots of whitespace
- Charts/sparklines for data (purple in OpenAI → use `#7AF17A` for Marrow)
- Right-side updates/changelog feed on the home page
- Collapsible sidebar toggle

**Brand colors:**
- Accent: `#7AF17A` (neon green — replaces OpenAI's purple)
- Dark bg: `#0D0D0D`
- Surface: `#111111`
- Border: `#1F1F1F`
- Text: `#FFFFFF` / `#A0A0A0`
- Light mode also required (toggle button)

**Font:** Inter (OpenAI uses their custom OpenAI Sans — Inter is the closest free equivalent)

**Previous prototype (v1) is saved** at `marrow-prototype-v1.html` — keep it as reference, don't overwrite.

---

## Navigation Structure (DECIDED)

Left sidebar items:
```
[bliro ▾]          ← client/project selector
[≡]                ← sidebar collapse

Overview           ← home page
Products
Customers
Communication
Knowledge Base     ← full ChatGPT-style chat page
──────────────
Settings           ← includes Integrations tab

[KK] Kilian Kutza  ← bottom, avatar + name + role
     Product Manager
```

---

## The Two Chat Modes (DECIDED)

1. **Mini popup** (bottom-right bubble, every page except KB):
   - Clicking "Ask Marrow" opens a small ~360×480px popup
   - Context = current page (knows which product/view you're on)
   - Has "Open full Knowledge Base →" escalation link
   - Uses OpenAI API (key in `marrow-config.js`)

2. **Full Knowledge Base page** (sidebar nav item):
   - Full-page ChatGPT-style layout
   - Left: conversation history
   - Right: full chat with source citations
   - Escalation flow: "I don't know → route to [person] → answer saved back to KB"

---

## Demo Mock Data — Bliro (DECIDED)

This is the demo client for the Bliro meeting. All data is swappable via the `CLIENT` config object.

```javascript
CLIENT = {
  company: "bliro",
  products: [
    { name: "Notetaker",          health: 87, status: "ok"   },
    { name: "CRM Sync",           health: 74, status: "warn" },
    { name: "Coaching Assistant", health: 61, status: "bad"  },
    { name: "Ask Bliro",          health: 55, status: "bad"  },
  ],
  customers: [
    { name: "Nordwind Energy AG", arr: "€52k", signals: 14 },
    { name: "Mercatura GmbH",     arr: "€31k", signals:  9 },
    { name: "Schäfer Logistics",  arr: "€19k", signals:  6 },
    { name: "Konrad & Bremer",    arr: "€11k", signals:  4 },
  ],
  // + 5 product updates (2 red, 2 yellow, 1 green)
  // + 3 live activity items
  // + G2 rating: 4.8/5, 12 reviews
}
```

To demo for a different client: swap the CLIENT object — everything re-renders automatically.

---

## ⚠️ THE OPEN QUESTION: What Figures to Show

**This is what the next conversation needs to decide before rebuilding.**

The PM at a 50-person startup needs to make decisions. The dashboard should show exactly the numbers that drive those decisions. We need to agree on:

### Dashboard Home — what 4–6 headline metrics?

Current v1 shows: product count, avg health %, signal count, ARR per customer, comms stats.

Questions to resolve:
- **Product health %** — how is this calculated? From what data? (customer tickets? NPS? uptime? all of the above weighted?)
- **Signals** — what counts as a signal? (email mention + support ticket + call transcript mention = 1 signal per customer?)
- **Customer priority score** — should we show a ranking of which customer needs attention most urgently, weighted by ARR?
- **Feature backlog rank** — should the home page show the #1 thing engineering should build this sprint?
- **Communication status** — "1 pending push" is good, but what triggers "pending"? (new update not yet communicated?)
- **Market signal** — competitor tracking card? How does this get populated?

### Products page — per-product what do we show?
- Health score (composite of what?)
- Signal count (# of customer mentions this week)
- Trend (direction vs last week)
- Open issues / feature requests count
- Last update date

### Customers page — per customer what do we show?
- ARR (or MRR)
- Signal count
- Churn risk score?
- Which products they use
- Last contact date

### Communication page — what triggers an "update"?
- Any transcript uploaded → AI extracts changes → becomes an update
- Manual entry
- Linked to a product version/release

---

## Pipeline (Context for Demos)

| Company | Contact | Status | Value |
|---|---|---|---|
| revel8 | Tom Müller | LOI offered — Q1 2027 paying customer | High |
| **bliro** | **Maurice Schweitzer** | **Meeting TODAY (Jun 11)** | **Priority demo** |
| AnyBill | Lea Frank | Warm — pending CTO validation | Medium |
| Netz16 | Thomas | Not now — revisit 2027 | Low |

**For the bliro demo:** Maurice already knows us (he was interview #13 in our first round). He likes the concept. Demo goal: close a €500/month, 3-month development partnership.

---

## What's Already Built (v1 prototype)

- [x] All 5 pages rendering (Overview, Products, Customers, Communication, Knowledge Base)
- [x] Dark/light mode toggle
- [x] Bliro mock data wired into CLIENT config
- [x] Settings → Integrations page (9 integrations, Slack + n8n pre-connected)
- [x] Mini chat bubble (needs popup upgrade)
- [x] KB page with pre-loaded Q&A + escalation card
- [x] + Add product button
- [x] Logo sized correctly

## What's NOT Done Yet

- [ ] Redesign to OpenAI console aesthetic (next task)
- [ ] Mini chat popup (not navigate-to-KB, but floating popup)
- [ ] Real OpenAI API wired into chat (config file exists, integration not complete)
- [ ] Transcript upload → AI extraction flow (working concept, not wired)
- [ ] Slack push simulation
- [ ] Figures/metrics defined (the conversation to have in next chat)

---

## Key Decisions Already Made

| Decision | Choice |
|---|---|
| Product name | **Marrow** |
| Build approach | Single HTML prototype for demos → Next.js for real product |
| Stack (real product) | Next.js 14 + shadcn/ui + Supabase + Anthropic Claude API |
| Design reference | OpenAI Platform console |
| Accent color | `#7AF17A` |
| Font | Inter |
| Demo client | bliro (swappable via CLIENT config) |
| Internal comms | n8n workflow already built (Slack + cartoon) |
| External comms | LinkedIn post + Newsletter (two output options) |
| Pricing model | €500/month × 3-month dev partnership |
| KB approach | Invisible backend + agent chat (no visual wiki page) |

---

## For the Next Chat — Suggested Opening Prompt

```
I'm building "Marrow" — an AI product management platform for SaaS scale-ups.
We have a working HTML prototype at ~/Downloads/marrow-prototype-v1.html
and a handoff doc at ~/Downloads/marrow-handoff.md that has full context.

I need to:
1. Decide what specific metrics/figures to show on each dashboard page
   (thinking from the POV of a PM at a 50-person startup)
2. Then rebuild the prototype in the OpenAI console visual style
   (dark, minimal, #7AF17A accent, Inter font) with those metrics

Let's start with step 1: what numbers does a PM actually need to see?
```
