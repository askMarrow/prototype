# Marrow — UI Redesign Prompt for Lovable
**Work in branch `ui-redesign` — do NOT touch main**

---

## Context

This is "Marrow Updates" — a product update communication platform built in TanStack Start + React + shadcn/ui. It already has fully working backend server functions:

- Voice/transcript → AI field extraction → structured update form (`transcription.functions.ts`)
- Newsletter generation via OpenAI server-side (`newsletter.functions.ts`)
- Slack publish flow with n8n webhook (`/slack-publish` route)
- Feature request capture from voice/text (`/request-feature` route)

**Keep all of that. Touch nothing in `src/lib/*.functions.ts`, `src/routes/new-update.tsx`, `src/routes/slack-publish.tsx`, `src/routes/newsletter.tsx`, `src/routes/request-feature.tsx`, or `src/router.tsx`.**

The only job of this prompt is to rebuild the visual layer — navigation, layout, page designs, and two new pages (Customers, Communication overview) — to match the design reference below.

---

## Design reference

Visual target: `https://github.com/askMarrow/prototype` — open `index.html` to see the full design running in browser.

**Core design language:**
- Dark background `#0d0d10`, sidebar even darker `#06060a`
- No border line between sidebar and content — the color difference creates separation on its own
- Cards: `background: #111115`, `border: 1px solid #1d1d25`, `border-radius: 10px`
- Accent color: `#7AF17A` (neon green) — active nav items, CTAs, healthy states
- Red `#f87171` = problems/risk, Yellow `#fbbf24` = warnings/notable
- Font: Inter
- All text muted by default (`#a1a1aa`), white only for primary headings and key numbers

**Add these to your Tailwind/CSS config as CSS variables and map them to Tailwind theme tokens.**

---

## New app shell — replace current Navbar layout

Create a persistent collapsible sidebar layout that wraps all routes.

```
┌──────────────────────────────────────────────────┐
│  SIDEBAR (220px, collapses to 50px icon-only)     │
│                                                    │
│  [Marrow logo]                    [← collapse]    │
│                                                    │
│  ● Overview                                        │
│    Products              [2 red badge]             │
│    Customers                                       │
│    Communication         [1 green badge]           │
│    Knowledge Base                                  │
│      └─ Today                                      │
│         Chat about CRM Sync    ← conv history      │
│         New conversation                           │
│         + New chat                                 │
│                                                    │
│  ────────────────────────────────                  │
│  [KK] Kilian Kutza    [🔔] [⚙ settings]           │
└──────────────────────────────────────────────────┘
```

- Settings icon (gear) opens a settings sheet/modal — not a full page nav item
- No dark/light toggle in the sidebar (can be in settings later)
- When on the Knowledge Base route, render KB conversation history below the KB nav item (like ChatGPT). Show "+ New" button inline. Each item is the conversation title truncated to ~40 chars. Clicking switches the active conversation.
- Collapsed state: show only icons, hide labels and conversation history

---

## Page specifications

### `/` — Overview dashboard (redesign)

Replace current product cards. Layout:

```
Welcome back, Kilian                            [7d|30d|90d]
Wednesday, 11 June 2026

[scrollable chip row: one chip per pending update, red/yellow/green]

[Products card — 2/3 width]    [Customers card — 1/3]
[Communication card — 1/2]     [Live Activity — 1/2]
```

**Products card**: Large number = product count. Mini rows (4) = product name + signal count + trend arrow. "Highest urgency:" tag showing most critical product name. Clicking navigates to `/product`.

**Customers card**: Large number = total signals this period. Mini rows (4) = customer name + ARR. Clicking navigates to `/customers`.

**Communication card**: 2×2 grid of mini boxes (Major / Notable / Pending / Pushed counts). 3 pending update rows below. Clicking navigates to `/communication`.

**Live Activity**: Static recent events feed (3 rows, each: green dot + text + timestamp).

All cards clickable. Scope selector (7d/30d/90d) in top right affects displayed numbers — multiply base values by 1× / 4.1× / 11.3× for the three scopes.

---

### `/product` — Products portfolio (redesign)

Replace current list with **2-column grid of large portfolio cards**. Each card ~200px tall minimum.

Card contents:
- Product name (18–20px bold) + status tag top-right (Healthy/Monitor/Needs attention)
- One-line description
- Stats: `€Xk ARR · N signals ↑X% · N open · X% problems`

Clicking a card navigates to `/product/$productId` — **no accordion on the list page**.

---

### `/product/$productId` — Product detail (new)

- Back button
- Page title = product name, subtitle = description
- Row of 3 stat cards: ARR at stake, Signals this week (with % trend), Open actions
- Row of 2 cards: Signal sentiment (horizontal bar chart: Problems / Requests / Positive) + Customers using this product (list of avatars + names, each clickable to `/customers/$id`)
- Development section (3 cards): Stage badge (GA/Beta/Alpha), On-track (Yes/At risk), Next milestone
- Roadmap card (full width): list of upcoming items with colored dots (green=in progress, yellow=in review, grey=planned)
- Recent releases card: last 3 version tags + date + one-line description
- Key signals timeline: bullet points with colored dots
- "Create update for this product" button → links to existing `/new-update` route

---

### `/customers` — Customer management (new page)

**Topbar**: title + scope selector (7d/30d/90d) right-aligned.

**4 hero metric cards**:
1. **Satisfaction score** (0–100, formula: `100 - weighted_avg(problemPct, arrVal)`) — clicking filters list to satisfied (low risk, <35% problem)
2. **At-risk accounts** (count HIGH risk) — clicking filters list to high risk only
3. **Avg response time** (hours) — `avg(customers.responseTime)`
4. **Signal volume** (scoped, with trend)

When a filter is active, show a banner: "Showing: At-risk accounts [Clear ×]"

**Customer list** (accordion, same animation as Communication list):
- Row: [initials avatar] [name + industry + signals + response time] [ARR] [risk badge]
- Expand: 3–5 bullet points about key signals, then "View full profile →"
- "View full profile →" → `/customers/$id`

---

### `/customers/$id` — Customer detail (new)

- Back button to `/customers`
- 3 stat cards: ARR, Signals this week (with trend), Churn risk
- 2 cards: Products used (clickable to product) + Signal breakdown bar chart
- Key signals timeline

---

### `/communication` — Communication hub (new consolidated page)

This is the outbound communications view. Keep existing routes (`/new-update`, `/slack-publish`, `/newsletter`) working in the background — this page just surfaces them in one place.

**Topbar action buttons**:
- "Upload transcript" → navigate to existing `/new-update`
- "Voice memo" → trigger existing microphone transcription flow from `/new-update`

**Update list** (accordion):
- Row: [color dot] [title] [type badge: Release/Incident/Roadmap/Social] [audience] [time ago] [Push to Slack btn if n8n-ready]
- Push to Slack button calls the existing Slack publish server function and shows a success toast
- Expand: AI-drafted content bullets + channel tags (Slack ready (n8n) / LinkedIn / Newsletter) + "View full →" link

Type → color mapping: release=green, incident_update=red, roadmap=yellow, social=green.

"Slack ready (n8n)" means the n8n webhook is wired and the button should be active. Use the existing `publishToSlack` server function logic from `/slack-publish` route.

---

### `/knowledge-base` — Chat (redesign, keep full width)

Remove any separate left panel for history (history lives in sidebar as described above).

Full-width chat:
- Messages: user messages right-aligned (accent-dim background), AI messages left-aligned (surface background)
- Input bar: rounded pill, send button (accent color)
- Disclaimer below input

**Floating "Ask Marrow" button** (fixed, bottom-right, 28px from edges):
- Green pill button, bigger than current, `padding: 12px 22px`, `font-size: 13px`
- Clicking opens a **popup** (360×490px), positioned above the button — does NOT navigate to KB
- Popup header: "Ask Marrow" label + current page context chip + close ×
- Popup chat: same message styles, smaller font (12.5px)
- Popup input: same pill style
- "Open full Knowledge Base →" link at bottom of popup — when clicked: transfers popup conversation into a new KB conversation object, navigates to `/knowledge-base`, closes popup

All AI responses use `createServerFn` — move the OpenAI call server-side so the key never hits the client. System prompt includes current page context and a summary of the mock data.

---

## Conversation history (KB + popup)

Store in React state (not persisted to DB yet — that's a future task):

```typescript
type Conversation = {
  id: number;
  title: string; // first user message, truncated to 42 chars
  messages: { role: 'user' | 'assistant'; content: string }[];
  ts: string; // formatted time
};

const [conversations, setConversations] = useState<Conversation[]>([]);
const [activeConvId, setActiveConvId] = useState<number | null>(null);
```

When user sends first message in a new conversation, set the title. When popup is transferred to KB, create a new conversation seeded with popup messages.

---

## Mock data

Create `src/lib/mock-data.ts`:

```typescript
export const MOCK_PRODUCTS = [
  { id: 'notetaker', name: 'Notetaker', desc: 'Meeting transcription across Zoom, Teams, Webex', stage: 'GA', signals: 12, prevSignals: 8, arrStake: 83, problems: 2, requests: 4, positive: 6, openActions: 4, status: 'ok' as const, onTrack: true, nextMilestone: 'v2.5 — Q3 2026',
    roadmap: ['Summary templates per meeting type — in progress', 'Language detection for DE-AT — in review', '30-min prep email trigger — planned'],
    releases: [{ v: 'v2.4', date: 'Jun 10', note: 'Webex mobile transcription added — 8 accounts unlocked' }, { v: 'v2.3', date: 'May 22', note: 'Speaker label accuracy improved +12%' }, { v: 'v2.2', date: 'May 1', note: 'Zoom waiting room transcription fixed' }] },
  { id: 'crm-sync', name: 'CRM Sync', desc: 'Automated field mapping to Salesforce / HubSpot', stage: 'GA', signals: 9, prevSignals: 3, arrStake: 83, problems: 6, requests: 2, positive: 1, openActions: 6, status: 'bad' as const, onTrack: false, nextMilestone: 'ENG-441 hotfix — today EOD',
    roadmap: ['Salesforce sandbox re-auth fix — in progress (ENG-441)', 'Per-account custom field mapping — in review', 'Dynamics 365 lag investigation — planned'],
    releases: [{ v: 'v1.8', date: 'Jun 3', note: 'HubSpot deal stage sync added' }, { v: 'v1.7', date: 'May 15', note: 'SAP connector beta released' }, { v: 'v1.6', date: 'Apr 28', note: 'Batch sync performance +40%' }] },
  { id: 'coaching', name: 'Coaching Assistant', desc: 'AI sales coaching suggestions from call transcripts', stage: 'Beta', signals: 6, prevSignals: 9, arrStake: 52, problems: 3, requests: 2, positive: 1, openActions: 3, status: 'warn' as const, onTrack: true, nextMilestone: 'Industry templates — Q3 2026',
    roadmap: ['Energy/Manufacturing industry template packs — in progress', 'Real-time nudges during calls — planned', 'Manager dashboard for coaching metrics — planned'],
    releases: [{ v: 'v0.9', date: 'Jun 7', note: 'Latency -60% — suggestions arrive <90s post-call' }, { v: 'v0.8', date: 'May 20', note: 'Objection handling suggestions added' }] },
  { id: 'ask-bliro', name: 'Ask Bliro', desc: 'AI Q&A over meeting history and knowledge base', stage: 'Beta', signals: 3, prevSignals: 3, arrStake: 31, problems: 1, requests: 2, positive: 0, openActions: 2, status: 'warn' as const, onTrack: true, nextMilestone: 'Notion integration — Q4 2026',
    roadmap: ['Notion / Confluence doc ingestion — planned', 'Hallucination guard for sparse history — in review', 'Conversation memory across sessions — planned'],
    releases: [{ v: 'v0.5', date: 'May 30', note: 'Source citations added to answers' }, { v: 'v0.4', date: 'May 12', note: 'Response streaming enabled' }] },
];

export const MOCK_CUSTOMERS = [
  { id: 'nordwind', initials: 'NE', name: 'Nordwind Energy AG', arr: '€52k', arrVal: 52, industry: 'Energy', risk: 'high' as const, problemPct: 64, signals: 14, prevSignals: 9, responseTime: 6.2, products: ['notetaker','crm-sync','coaching'], lastContact: '3 days ago',
    bullets: ['CRM Sync field mapping failure after Salesforce refresh — 3 tickets open, blocking QBR prep', 'Notetaker Austrian-dialect accuracy issue — ticket #447, affects 4 reps', 'Coaching template gap — requested energy-sector discovery questions in 2 separate calls', 'Expansion signal: asked about 2 more seats if Coaching improves'] },
  { id: 'mercatura', initials: 'MG', name: 'Mercatura GmbH', arr: '€31k', arrVal: 31, industry: 'Retail', risk: 'med' as const, problemPct: 44, signals: 9, prevSignals: 7, responseTime: 3.1, products: ['notetaker','crm-sync'], lastContact: '1 week ago',
    bullets: ['CRM Sync Salesforce sandbox issue — 1 open ticket', 'Notetaker v2.4 positive — Webex support well received by their team', 'Feature request: bulk PDF export of meeting summaries for board reporting'] },
  { id: 'schaefer', initials: 'SL', name: 'Schäfer Logistics', arr: '€19k', arrVal: 19, industry: 'Logistics', risk: 'low' as const, problemPct: 33, signals: 6, prevSignals: 5, responseTime: 2.8, products: ['notetaker','crm-sync'], lastContact: '5 days ago',
    bullets: ['CRM Sync affected by sandbox issue — 1 ticket, lower priority than Nordwind', 'Notetaker satisfaction high — positive signals on daily driver review calls', 'Expansion signal: asked about seat pricing for 5 more reps'] },
  { id: 'konrad', initials: 'KB', name: 'Konrad & Bremer', arr: '€11k', arrVal: 11, industry: 'Consulting', risk: 'low' as const, problemPct: 25, signals: 4, prevSignals: 4, responseTime: 8.4, products: ['notetaker','ask-bliro'], lastContact: '1 week ago',
    bullets: ['Ask Bliro hallucination complaint — cited a meeting that never happened', 'Notetaker positive — team likes summary quality for client notes', 'No urgent actions — stable, renewal expected'] },
  { id: 'techventures', initials: 'TV', name: 'TechVentures Berlin', arr: '€44k', arrVal: 44, industry: 'Tech', risk: 'med' as const, problemPct: 45, signals: 11, prevSignals: 7, responseTime: 4.7, products: ['notetaker','crm-sync','coaching'], lastContact: '2 days ago',
    bullets: ['CRM Sync custom field mapping — Salesforce is heavily customised, global mapping breaks workflows', 'Coaching interest high — 3 positive signals on coaching for new AE cohort', 'Notetaker speaker labels — 2 complaints on large group calls'] },
  { id: 'alpenwind', initials: 'AW', name: 'Alpenwind GmbH', arr: '€28k', arrVal: 28, industry: 'Manufacturing', risk: 'high' as const, problemPct: 67, signals: 12, prevSignals: 6, responseTime: 11.2, products: ['crm-sync','coaching'], lastContact: '1 day ago',
    bullets: ['CRM Sync critical — 4 failures this week on shift-based daily Salesforce sync', 'Coaching onboarding gap — new sales team, templates do not fit manufacturing cycles', 'Churn risk: mentioned evaluating Fathom in last check-in call'] },
  { id: 'medix', initials: 'MS', name: 'Medix Solutions', arr: '€36k', arrVal: 36, industry: 'Healthcare', risk: 'low' as const, problemPct: 29, signals: 7, prevSignals: 8, responseTime: 2.3, products: ['notetaker','ask-bliro'], lastContact: '4 days ago',
    bullets: ['Ask Bliro high usage — compliance team queries it daily for past meeting records', 'HIPAA data residency question — needs EU-only storage confirmation', 'Positive NPS: head of ops gave a 9 last month, expansion likely'] },
  { id: 'finflux', initials: 'FF', name: 'FinFlux AG', arr: '€22k', arrVal: 22, industry: 'FinTech', risk: 'med' as const, problemPct: 40, signals: 5, prevSignals: 3, responseTime: 5.9, products: ['notetaker','crm-sync'], lastContact: '6 days ago',
    bullets: ['Dynamics 365 sync lag — same intermittent issue as Nordwind, 1 ticket open', 'Meeting prep positive — sales team uses it before every investor call', 'Feature request: sentiment score as a CRM field after call sync'] },
  { id: 'ravel', initials: 'RM', name: 'Ravel Media GmbH', arr: '€15k', arrVal: 15, industry: 'Media', risk: 'low' as const, problemPct: 33, signals: 3, prevSignals: 4, responseTime: 7.1, products: ['notetaker'], lastContact: '1 week ago',
    bullets: ['Notetaker-only customer — happy with transcription for podcast production notes', 'Signal volume declining — down 25% WoW, lower usage phase', 'No complaints — quiet account, renewal expected'] },
];

export const MOCK_UPDATES = [
  { id: '0', sev: 'green', title: 'Notetaker v2.4 — Webex support shipped', type: 'release', audience: 'All customers · #product-updates', time: '1h', pushed: false, slackReady: true,
    bullets: ['What shipped: Webex mobile transcription now available — 8 enterprise accounts auto-unlocked', 'Slack draft ready: "🎉 Notetaker v2.4 is live! Webex mobile transcription is now supported..."', 'LinkedIn draft ready: "We just shipped Webex mobile support in Notetaker..."'] },
  { id: '1', sev: 'red', title: 'CRM Sync hotfix ETA — customer notice', type: 'incident_update', audience: 'Nordwind, Mercatura, Schäfer · direct email', time: '2h', pushed: false, slackReady: true,
    bullets: ['What to communicate: Salesforce sandbox re-auth issue — fix in review (ENG-441), ETA today EOD', 'Email draft ready: "We are aware of a CRM Sync field mapping issue..."', 'Affected: Nordwind Energy (€52k), Mercatura GmbH (€31k), Schäfer Logistics (€19k) — €102k ARR exposed'] },
  { id: '2', sev: 'green', title: 'Coaching Assistant v0.9 — latency improvements', type: 'release', audience: 'Pilot customers · #product-updates', time: '3d', pushed: true, slackReady: false,
    bullets: ['What shipped: suggestion delivery latency -60% — suggestions arrive within 90 seconds of call end', 'Comms pushed: Slack #product-updates sent, pilot customers emailed individually'] },
  { id: '3', sev: 'yellow', title: 'Q3 2026 roadmap preview', type: 'roadmap', audience: 'Key accounts · newsletter', time: '5d', pushed: false, slackReady: true,
    bullets: ['What to share: Notetaker v2.5 (summary templates), CRM Sync custom field mapping, Coaching GA', 'Newsletter draft ready — segment by industry, energy customers see Coaching template news first'] },
];
```

---

## Marrow logo SVG

Use this exact SVG in the sidebar (replace current Sparkles icon):

```svg
<svg viewBox="0 0 85 14" fill="none" xmlns="http://www.w3.org/2000/svg" style="height:14px;width:auto">
  <path d="M4 0H10V4H4V0Z" fill="#7AF17A"/>
  <path d="M0 4H4V8L7 11L4 14L0 10V4Z" fill="#7AF17A"/>
  <path d="M14 4H10V8L7 11L10 14L14 10V4Z" fill="#7AF17A"/>
  <path d="M20 0H22.83L27.67 11.27H27.82L32.66 0H35.44V13.77H33.28V3.83H33.13L28.86 13.77H26.57L22.3 3.83H22.15V13.77H20V0Z" fill="currentColor"/>
  <path d="M37.1 6.68C37.3 4.84 38.91 3.66 41.56 3.66C44.2 3.66 45.78 4.84 45.78 7.43V13.77H43.69V12.58H43.54C43.16 13.25 42.25 13.96 40.29 13.96C38.1 13.96 36.59 12.86 36.59 10.98C36.59 9 38.11 8.11 40.09 7.86L43.61 7.42V7.18C43.61 5.89 42.91 5.36 41.52 5.36C40.12 5.36 39.32 5.89 39.21 6.97H37.1V6.68Z" fill="currentColor"/>
  <path d="M47.24 3.89H49.41V5.03H49.56C49.92 4.31 50.57 3.85 51.92 3.85H53.29V5.73H51.93C50.24 5.73 49.51 6.62 49.51 8.48V13.77H47.24V3.89Z" fill="currentColor"/>
  <path d="M54.03 3.89H56.2V5.03H56.36C56.71 4.31 57.36 3.85 58.71 3.85H60.09V5.73H58.72C57.04 5.73 56.3 6.62 56.3 8.48V13.77H54.03V3.89Z" fill="currentColor"/>
  <path d="M59.91 8.83C59.91 5.46 62.05 3.66 65.11 3.66C68.18 3.66 70.32 5.46 70.32 8.83C70.32 12.2 68.18 14 65.11 14C62.05 14 59.91 12.2 59.91 8.83Z" fill="currentColor"/>
  <path d="M70.3 3.89H72.44L74.16 11.24H74.32L76.48 3.89H78.78L80.95 11.24H81.11L82.82 3.89H84.9V4.18L82.39 13.77H79.82L77.67 6.53H77.52L75.39 13.77H72.81L70.3 4.18V3.89Z" fill="currentColor"/>
</svg>
```

---

## Summary of what to create / change

| File | Action |
|---|---|
| `src/components/marrow/AppSidebar.tsx` | Create — collapsible sidebar with nav + KB history |
| `src/components/marrow/ChatPopup.tsx` | Create — floating Ask Marrow mini-chat |
| `src/components/marrow/MetricCard.tsx` | Create — hero stat card |
| `src/components/marrow/SignalBar.tsx` | Create — horizontal bar chart |
| `src/components/marrow/TimelineItem.tsx` | Create — dot + text timeline row |
| `src/components/marrow/Navbar.tsx` | Delete — replaced by AppSidebar |
| `src/lib/mock-data.ts` | Create — all mock data |
| `src/routes/index.tsx` | Redesign — overview dashboard |
| `src/routes/product.tsx` | Redesign — portfolio grid |
| `src/routes/product.$productId.tsx` | Create — product detail |
| `src/routes/customers.tsx` | Create — customer list with hero stats |
| `src/routes/customers.$id.tsx` | Create — customer detail |
| `src/routes/communication.tsx` | Create — outbound comms hub |
| `src/routes/knowledge-base.tsx` | Redesign — full-width chat |
| `src/routes/__root.tsx` | Update — wrap with AppSidebar layout |
| **DO NOT TOUCH** | `*.functions.ts`, `slack-publish`, `newsletter`, `new-update`, `request-feature` |
