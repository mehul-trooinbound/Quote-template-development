---
name: quote-estimation
description: Use when the user shares a file (screenshot, PDF, markdown, HTML, Figma export, or written spec) for a HubSpot CPQ quote, proposal, or agreement template and wants a ballpark time and effort estimate before building. Produces a structured estimate with scope, hours per task, duration, assumptions, exclusions, risks, and what is needed from the client. Estimate only, does not write code.
---

# Quote Template Estimation

## Purpose

Give a fast, honest ballpark estimate for building a custom React quote module for HubSpot CPQ (see the `quote-custom-modules-dev` skill for how these modules are built). This skill only estimates. It does not write code and does not start the build.

This skill is self-contained. Do NOT invoke brainstorming, planning, or any other skill. Keep the process short to save time and tokens.

## Process

### Step 1: Analyze the file silently

Read the provided file once (screenshot, PDF, .md, HTML, spec). Note:
- Number of distinct sections and pages
- Dynamic data needed (quote, deal, line items, contacts, company, signers) and whether any of it is not in `quoteTemplateContext` (needs `crm_object()` lookups or custom properties)
- Interactive parts that need Islands (tabs, toggles, optional line items, accordions, sliders, e-sign areas)
- Design complexity: custom fonts, icons, charts, illustrations, gradients, heavy branding
- Content that changes per quote versus static copy
- Responsive requirements (desktop only, or tablet and mobile too)
- Number of modules: one full-page module, or several reusable ones
- Anything missing or contradictory (unreadable text, missing assets, no mobile design)

If several files are given, treat them as one scope and mention each in the scope summary.

### Step 2: Ask only if it changes the estimate (ONE message, max 3 questions)

- If the file is clear enough for a ballpark, skip questions and state assumptions instead. Never ask just to ask.
- Otherwise send one message with at most 3 closed-ended questions, each with your default, for example: `1. Mobile layout needed? (default: yes, responsive)`.
- Good candidates: responsive scope, who supplies assets and copy, number of revision rounds.
- End with: "Reply 'ok' to accept all defaults." Do not ask a second round.
- Do not ask about hourly rate or budget. Give hours, and add cost only if the user provided a rate.

### Step 3: Estimate

Use these baseline hours per item (one experienced developer), then adjust up or down for the file you analyzed. Show the adjusted numbers, not the baseline table.

| Task | Baseline hours |
|---|---|
| Project setup (clone starter, install, auth check) | 1 to 2 |
| Section build, simple (text, header, footer, cover) | 1 to 2 each |
| Section build, medium (line items table, pricing summary, contact or company block) | 3 to 5 each |
| Section build, complex (optional items, toggles, charts, timeline, multi-tier pricing) | 6 to 10 each |
| Data binding and `hublDataTemplate` per distinct data group | 1 to 2 each |
| `crm_object()` or custom property lookup | 1 to 2 each |
| Island (interactive component) | 3 to 6 each |
| Editor fields (`fields` sidebar) | 2 to 4 total |
| `isQuoteBlueprint` placeholder data and editor fallbacks | 1 to 3 |
| Responsive pass (tablet and mobile) | 15 to 25 percent of build hours |
| Local preview and QA against real HubSpot rendering | 10 to 15 percent of build hours |
| Deploy, publish, and test with a real quote | 1 to 3 |
| Client revisions | 2 rounds included at about 10 percent of build hours each |
| Project management and communication | 10 percent of total |

Give every number as a range (low to high) and round to half hours. Use the high end when the file is vague, and say so.

### Step 4: Deliver the estimate in this exact format

```
# Estimate: <Project or module name>

**Source:** <file name(s)>   **Confidence:** Low / Medium / High (<one line why>)

## Summary
| Item | Value |
|---|---|
| Total effort | X to Y hours |
| Calendar duration | X to Y working days (1 developer, 6 productive hours per day) |
| Complexity | Low / Medium / High |
| Modules | N |
| Sections | N |
| Islands | N |

## Scope (what is included)
- <section or feature, one line each>

## Breakdown
| # | Task | Hours (low to high) | Notes |
|---|---|---|---|
| 1 | ... | 2 to 3 | ... |
| | **Total** | **X to Y** | |

## Timeline
| Phase | Duration |
|---|---|
| Setup and data binding | ... |
| Build | ... |
| Responsive and QA | ... |
| Revisions and deploy | ... |

## Assumptions
- <each assumption made in place of a question>

## Exclusions (not included)
- <for example: copywriting, asset or icon creation, Figma design work, changes to existing HubSpot theme, CRM property creation, workflows, e-sign configuration, rework of published quotes>

## Risks and dependencies
- <risk> | Impact: <extra hours or delay>

## Needed from client
- <assets, fonts, logo files, sample quote data, HubSpot portal access, approvals>

## Cost (only if the user gave a rate)
<total hours x rate, as a range>
```

### Step 5: Close

End with one line: "This is a ballpark within roughly plus or minus 25 percent. Reply 'build' to start with the quote-custom-modules-dev skill, or tell me what to adjust."

## Rules

- Ballpark only, never present it as a fixed quote.
- Every hour figure is a range. Never give a single number.
- Always include Assumptions, Exclusions, and Risks, even if short.
- State when something in the file is unsupported by HubSpot quote modules (for example a classic HubL module, or features needing a full CMS page rather than a quote) and estimate the closest supported alternative.
- Keep the whole response under about 70 lines. No long narration before the estimate.
- Do not use em dashes.
