# N8N Automation Workflows

A personal repository of N8N automation workflows designed to monitor, extract, and analyse financial and investment content using Claude AI. Each workflow monitors a specific content source, passes the content to Claude for structured analysis, and delivers a formatted HTML email summary to the owner's inbox.

---

## Repository Structure

```
n8n-workflows/
├── a16z-crypto-analyzer/
│   ├── workflow.json
│   ├── README.md
│   ├── prompts/
│   │   └── analysis-prompt-v2.md
│   └── code/
│       ├── extract-email-body.js
│       └── markdown-to-html.js
├── daily-dirtnap-analyzer/
│   ├── workflow.json
│   ├── README.md
│   ├── prompts/
│   │   └── trading-prompt-v1.md
│   └── code/
│       └── markdown-to-html.js
└── article-analyzer/
    ├── workflow.json
    ├── README.md
    ├── prompts/
    │   └── generic-analysis-prompt-v1.md
    └── code/
        ├── append-url.js
        └── markdown-to-html.js
```

---

## Workflows

### 1. a16z Crypto Newsletter Analyzer
**Trigger:** Hourly Gmail polling — emails from `a16zcrypto@substack.com`

Monitors the inbox for new editions of the a16z Crypto newsletter published by Andreessen Horowitz, one of the most influential venture capital firms in the crypto space. When a new edition arrives, the workflow extracts the plain text body and sends it to Claude for a five-section critical analysis calibrated for a reader with solid Bitcoin knowledge and moderate broader crypto familiarity. The analysis includes a core narrative assessment, non-obvious insights, argument vulnerabilities, a portfolio agenda check flagging where a16z's VC interests may be shaping the narrative, and a single actionable takeaway.

**Key Nodes:** Gmail Trigger → Gmail Get Message → Code (extract body) → Anthropic (Claude) → Code (Markdown → HTML) → Gmail Send

**Prompt Version:** v2 — Adversarial / Critical Framing

---

### 2. Daily Dirtnap Newsletter Analyzer
**Trigger:** Hourly Gmail polling — emails from `dillian@dailydirtnap.com` with PDF attachment

Monitors the inbox for new editions of The Daily Dirtnap, a daily financial markets newsletter written by hedge fund manager Jared Dillian. When a new edition arrives with a PDF attachment, the workflow fetches the full PDF binary, sends it directly to Claude as a base64-encoded document, and returns a trading-focused analysis covering non-obvious insights, internal tensions in the argument, the single highest-conviction market view, and a structured trade inventory table listing all actionable ideas with ticker, direction, instrument type, entry timing, and implied holding period.

**Key Nodes:** Gmail Trigger → Gmail Get Message (Download Attachments ON) → HTTP Request (Anthropic API with PDF as base64) → Code (Markdown → HTML) → Gmail Send

**Prompt Version:** v1 — Signal / Precision Focused

---

### 3. Article Analyzer
**Trigger:** Manual — web form submission

A manually triggered workflow for on-demand analysis of any publicly accessible article. The user submits an article URL via a bookmarked N8N web form, optionally adding additional context to guide Claude's focus. The workflow fetches clean article text via the Jina AI Reader API and passes it to Claude for a five-section critical analysis covering the core thesis, non-obvious insights, argument vulnerabilities, source and agenda assessment, and a single key takeaway. The analysis is delivered as a formatted HTML email with the article URL and user-defined subject line.

**Key Nodes:** Form Trigger → HTTP Request (Jina AI) → Anthropic (Claude) → Code (Append URL) → Code (Markdown → HTML) → Gmail Send

**Prompt Version:** v1 — Generic / Self-Calibrating

---

## Shared Design Patterns

All three workflows share the following architectural patterns:

**Styled HTML Email Output**
Each workflow converts Claude's markdown response to a fully inline-styled HTML email using a custom JavaScript converter. Styling includes a birch (`#DFD7C8`) header banner with newsletter name and date, Arial font family throughout, section headers with bottom borders, alternating-row tables for trade inventories, and a consistent footer with generation timestamp.

**Markdown Table Handling**
The HTML converter includes a dedicated table parser that renders Claude's markdown tables as properly styled HTML tables — used primarily in the Daily Dirtnap trade inventory section.

**Token Configuration**
All Claude API calls are configured with a maximum of 3000 output tokens to ensure complete five-section responses without truncation.

**Retry Logic**
All nodes calling the Anthropic API are configured with Retry on Fail enabled — 3 retries with a 2-minute wait between attempts — to handle API timeouts gracefully without workflow failure.

**Date Formatting**
All workflows use N8N's Luxon DateTime object (`$today.toFormat('MMMM d, yyyy')`) for consistent date formatting across banner headers and footers.

---

## External Dependencies

| Service | Purpose | Authentication |
|---|---|---|
| Gmail API (OAuth2) | Email trigger, message fetch, send | OAuth2 credential in N8N |
| Anthropic API | Claude AI analysis | API key credential in N8N |
| Jina AI Reader API | Article text extraction (Article Analyzer only) | None required for personal use |

---

## Prerequisites

- N8N instance (self-hosted or cloud)
- Gmail OAuth2 credential configured in N8N
- Anthropic API key configured in N8N
- Active workflows toggled ON in N8N for scheduled triggers

---

## Change Log

| Date | Workflow | Change |
|---|---|---|
| Mar 2026 | a16z Crypto | Upgraded to prompt v2 — adversarial framing with Portfolio Agenda Check section |
| Mar 2026 | a16z Crypto | Increased max tokens from 1500 to 3000 |
| Mar 2026 | Daily Dirtnap | Added markdown table parser to HTML converter for Trade Inventory section |
| Mar 2026 | Article Analyzer | Initial build — Form Trigger + Jina AI + Claude generic prompt |
| Mar 2026 | All | Standardised HTML styling — birch banner, Arial font, consistent footer |
| Mar 2026 | All | Added Retry on Fail to all Anthropic API nodes — 3 retries, 2 min wait |

---

## Notes

- The Daily Dirtnap workflow is designed specifically for newsletters delivered as PDF attachments. If Jared Dillian changes the delivery format, Node 2's Download Attachments configuration and Node 3's base64 PDF handling will need to be reviewed.
- The Article Analyzer will not work on paywalled content, JavaScript-rendered pages, or sites that block the Jina AI reader. For those cases, manually copying and pasting article text into a modified version of the workflow is the recommended fallback.
- Claude's Portfolio Agenda Check section in the a16z workflow relies on Claude's training knowledge of a16z's known investments. It may not reflect the most recent fund positions and should be treated as directional rather than definitive.

---
