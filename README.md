# Soch Ops Score

A lightweight, self-contained ops readiness assessment tool for early-stage B2B startups. Built by [Soch](https://withsoch.com) — Operations-as-a-Service for pre-seed through Series A teams.

Live at: **[withsoch.com/score](https://withsoch.com/score)**

---

## What it does

Walks a founder or operator through 12 questions across three pillars:

- **Ops Readiness** — process documentation, role clarity, performance tracking
- **Automation Maturity** — workflow automation, tool integration, AI adoption
- **Team Efficiency** — prioritisation, handoffs, onboarding, leverage

Scores their business out of 100 and assigns one of four tiers: Pre-ops, Early-stage, Building, or Scale-ready.

The score and pillar breakdown are shown immediately. A full personalised report — key findings, root cause analysis, action plan, and Soch service recommendations — is sent to their email via an n8n automation triggered by a webhook.

---

## Files

```
soch-ops-score.html          — The full quiz, results page, and email gate. Self-contained.
soch-ops-score-n8n-workflow.json  — n8n workflow: webhook → Anthropic API → email to lead + internal alert
README.md                    — This file
```

---

## How to deploy

### 1. Host the HTML

The quiz is a single HTML file with no build step and no external dependencies beyond Google Fonts.

**Webflow**
1. Create a new blank page at `/score`
2. Add an Embed element to the page body
3. Paste everything between `<body>` and `</body>` into the embed
4. Alternatively: Page Settings → Custom Code → paste full file content into the Footer Code field
5. Publish

**GitHub Pages**
1. Push `soch-ops-score.html` to your repo
2. Rename it `index.html` if you want it at the root, or keep it as-is and access it at `/soch-ops-score.html`
3. Enable GitHub Pages in repo Settings → Pages → deploy from `main` branch

**Any static host (Netlify, Vercel, Cloudflare Pages)**
Drop the HTML file into your repo and deploy. No configuration needed.

---

### 2. Set up the n8n workflow

1. In n8n, go to **Workflows → Import** and upload `soch-ops-score-n8n-workflow.json`
2. Open the **Webhook** node and copy the generated webhook URL
3. In `soch-ops-score.html`, find this line and replace the placeholder:

```js
const WEBHOOK_URL = 'https://YOUR_N8N_WEBHOOK_URL';
```

4. Set up your environment variable:
   - Add `ANTHROPIC_API_KEY` to your n8n environment (Settings → Variables or via your hosting provider)

5. Configure the **Email Send** nodes with your SMTP credentials for `info@withsoch.com`

6. Activate the workflow

7. Enable CORS on the Webhook node:
   - Response Mode: `Immediately`
   - Add response header: `Access-Control-Allow-Origin: *`

---

### 3. Test end-to-end

1. Open the HTML file in a browser
2. Complete all 12 questions
3. On the results page, enter a test name and email
4. Confirm the n8n workflow executes
5. Check the test inbox for the report email
6. Check `info@withsoch.com` for the internal alert

---

## Webhook payload

When a lead submits their email, the widget sends a POST request with this shape:

```json
{
  "name": "Sarah",
  "email": "sarah@company.com",
  "overall": 42,
  "tier": "Building",
  "headline": "Good bones, but visible gaps",
  "summary": "You have the right instincts but...",
  "pillar_ops": 58,
  "pillar_automation": 33,
  "pillar_team": 50,
  "answers": [
    { "q": "How are your core business processes documented?", "a": "Partially documented in docs or wikis" },
    ...
  ]
}
```

---

## Scoring logic

Each question is worth 0–3 points (4 options per question, 4 questions per pillar).

| Pillar | Questions | Max raw score |
|---|---|---|
| Ops Readiness | Q1–Q4 | 12 |
| Automation Maturity | Q5–Q8 | 12 |
| Team Efficiency | Q9–Q12 | 12 |

Pillar scores are expressed as percentages. Overall score is the aggregate across all 36 points, expressed out of 100.

| Tier | Score range |
|---|---|
| Pre-ops | 0–24 |
| Early-stage | 25–49 |
| Building | 50–74 |
| Scale-ready | 75–100 |

---

## Customisation

**Questions** — edit the `questions` array in the `<script>` block. Each question needs a `pillar` (0, 1, or 2), `text`, and four `options` with `value` 0–3.

**Report prompt** — the Anthropic API prompt lives in the n8n workflow's HTTP Request node (`Generate Report (Anthropic API)`). Edit the `content` field to change tone, structure, or Soch-specific messaging.

**Branding** — colours and fonts are set via CSS variables at the top of the `<style>` block. Swap `--font` and colour values to match your brand.

**Tier thresholds** — adjust score cutoffs in the `buildResults()` function.

---

## Stack

- Vanilla HTML / CSS / JS — no framework, no build step
- [DM Sans](https://fonts.google.com/specimen/DM+Sans) via Google Fonts
- [n8n](https://n8n.io) for workflow automation
- [Anthropic API](https://docs.anthropic.com) (`claude-sonnet-4-20250514`) for report generation
- SMTP via n8n Email Send node

---

## About Soch

[Soch](https://withsoch.com) is an Operations-as-a-Service consultancy for pre-seed through Series A B2B startups. We help founders build the operational systems that make AI actually work — process documentation, automation workflows, and fractional ops and PM support.

**Get in touch:** [riz@sochconsulting.com](mailto:riz@sochconsulting.com)

---

## License

MIT. Free to use, adapt, and build on. Attribution appreciated but not required.
