# AI-Powered Paid Search Automation Systems

**Proprietary AI-driven systems for search engine marketing campaign management.**

Developed by [Dominic Mac-Ennin, CDMP, PCM®](https://www.linkedin.com/in/dominicmacennin) — Lead PPC Specialist with 6+ years managing Google Ads, Microsoft Ads, and Meta campaigns across e-commerce, B2B SaaS, home services, manufacturing, and automotive verticals.

---

## Overview

This repository documents two proprietary systems that automate core search engine marketing workflows using the Claude AI API. Both systems are designed to reduce manual effort, improve consistency, and surface decisions that are difficult to make reliably at scale through unaided human review.

Neither system applies changes to a live advertising account automatically. All outputs are structured for human review before any campaign action is taken.

---

## System 1 — AI-Powered Keyword Research & Evaluation System

### What it does

Evaluates keyword idea exports from keyword research tools and produces a structured, prioritised review sheet of keyword candidates — categorised by intent, filtered by relevance, and formatted with match type syntax ready for campaign build.

### The problem it solves

Manual keyword evaluation requires a practitioner to assess hundreds or thousands of keyword candidates simultaneously across multiple criteria: search intent, relevance to the specific business offering, trend direction, cost, and match type suitability. Applied inconsistently, this process produces campaign keyword sets that are either too narrow (missed volume) or too broad (wasted spend).

### How it works

The system operates in five stages:

**Stage 0 — Business Model Classification**
Reads the business profile provided at intake and assigns one of five evaluation modes: Lead Gen B2B/SaaS, Lead Gen Local Services, E-Commerce, Foot Traffic, or Awareness. The assigned mode governs relevance thresholds, intent tolerance, tiebreaker behaviour, and match type assignment for the entire run.

**Stage 1 — Hard Filters**
Rule-based exclusions applied before AI evaluation: zero-volume terms, negative trend terms, competitor brand references, job/employment intent, informational-only queries, and DIY/product-only intent.

**Stage 2 — Trend Logic**
Evaluates 3-month and year-over-year trend data. Positive 3-month trend overrides flat or negative YoY — recency takes precedence.

**Stage 3 — Relevance & Intent Evaluation**
Claude API evaluates each keyword against the business profile. Each keyword receives a relevance assessment, an intent label (Commercial, Transactional, Informational, Navigational, Ambiguous), and a confidence score (High, Medium, Low).

**Stage 4 — Tiebreaker Logic**
For Low confidence keywords, top-of-page bid range is used as a cost-informed tiebreaker. Keywords without CPC data route to a dedicated Needs Review output.

### Market Maturity Diagnostic Layer

For B2B/SaaS and high-ticket service campaigns, the system includes a post-evaluation diagnostic that analyses the intent distribution of the returned keyword set to detect whether search demand is sufficiently mature to support a viable campaign.

Four signal types are produced:

| Signal | Meaning |
|---|---|
| `STRONG_IMMATURITY_SIGNAL` | Market demand is in discovery/education phase — Search is premature |
| `POSSIBLE_IMMATURITY_SIGNAL` | Possible immaturity — seed keyword methodology should be reviewed first |
| `NICHE_MARKET` | Low volume but real buyers present — small TAM, not immature |
| `DEVELOPING_MARKET` | Low volume with positive trend — market is growing into viability |

On a confirmed `STRONG_IMMATURITY_SIGNAL`, the system automatically adjusts match type posture, surfaces business-context-specific keyword expansion directions, and provides a channel recommendation — all derived from the business profile provided at intake.

### Output structure

Keywords are routed to one of four tabs:

| Tab | Contents |
|---|---|
| Included Keywords | High and Medium confidence inclusions, sorted by confidence → intent → trend → volume |
| Needs Review | Low confidence keywords with tiebreaker lean and cost context for human decision |
| Excluded Keywords | All filtered terms with exclusion reason |
| Volume Overflow | Keywords not processed due to token budget constraints — ready for next pass |

Match type syntax is applied in the output: `[exact match]` and `"phrase match"`. Broad match is never assigned by the system under any condition.

---

## System 2 — AI-Powered Search Term Analysis & Optimization Pipeline

### What it does

Automates the identification of negative keyword candidates from live search term data in Google Ads and Microsoft Ads accounts, on a configurable biweekly cadence.

### The problem it solves

Negative keyword hygiene — the exclusion of irrelevant search queries from triggering ads — directly affects wasted spend and cost-per-conversion. Manual search term review is time-intensive, inconsistently applied, and poorly suited to the volume of data generated by active campaigns. Accounts without regular negative keyword review accumulate wasted spend silently.

### How it works

**Data pull**
On Google Ads: a GAQL query pulls search terms from configured campaigns via Google Ads Scripts, filtered by status (NONE — not yet actioned), minimum impressions threshold, and a configurable date window.

On Microsoft Ads: an iterator-based traversal of the account hierarchy (campaigns → ad groups → keywords → search terms) replaces the GAQL layer. All downstream logic is identical across platforms.

**Priority sort**
Search terms are sorted before the AI call using a three-tier priority:
1. Cost — descending (highest wasted spend first)
2. Conversions — ascending (zero-conversion terms bubble up within cost tier)
3. Impressions — descending (highest exposure as tiebreaker)

This ensures that if token budget constraints require truncation, the lowest-priority terms are dropped — not arbitrary ones.

**Business profile evaluation**
The Claude API evaluates each term against a structured business profile — including services offered, services not offered, customer type, competitor brands, location, and irrelevant verticals. The system derives a negative aggression posture from the business model classifier and applies posture-specific match type rules to all recommendations.

**Posture levels**

| Posture | Triggered by | Match type behaviour |
|---|---|---|
| Very Conservative | One-time / Over $20K AOV | Exact and Phrase only. Broad never recommended. |
| Conservative | One-time / $500–$20K AOV | Phrase as default. Broad only for single-word disqualifiers. |
| Moderate | Recurring revenue model | Phrase default. Broad acceptable for clear category mismatches. |
| Per-Campaign Override | Hybrid revenue model | Conservative on one-time campaigns. Moderate on recurring. |

**API resilience (v1.1)**
The Claude API call is wrapped with retry logic: 3 attempts with exponential backoff (10s → 20s → 40s), HTTP status code validation, and email notification on run failure via MailApp. The cadence gate is intentionally not updated on failure — the next scheduled weekly trigger retries automatically.

### Output structure

| Tab | Contents |
|---|---|
| Suggested Negatives | Claude's categorised output with match type, category, reason, apply-at level, and spend context |
| Raw Log | Append-only priority-sorted search term data — full audit trail, never cleared |
| Run Log | One row per execution: timestamp, term counts, top term by cost, status |
| Config | All client parameters — the only manually-edited surface |

---

## Unified Business Profile Schema

Both systems are configured from a single business profile — one intake, two systems activated. The schema comprises seven field groups:

1. Business Model Classifier (BUSINESS_MODEL, REVENUE_MODEL, TRANSACTION_VALUE_TIER)
2. Business Identity (BUSINESS_NAME, BRAND_TERMS, WEBSITE_URL, LOCATIONS)
3. Offering Definition (BUSINESS_CATEGORY, SERVICES_OFFERED, SERVICES_NOT_OFFERED, CUSTOMER_TYPE)
4. Competitive Landscape (COMPETITOR_BRANDS, COMPETITOR_STRATEGY)
5. Intent Signal Parameters (INFORMATIONAL_SIGNALS, SEED_KEYWORDS, IRRELEVANT_VERTICALS)
6. Campaign Overrides (for hybrid revenue models — per-campaign posture mapping)
7. Pipeline Operating Parameters (API key, campaign names, date window, impression floor, cadence)

---

## Design Principles

- **Human review is required before any changes are applied in-platform.** Both systems produce structured recommendations, not automated actions.
- **The Config tab is the sole manual-edit surface.** All logic parameters are externalised — no code changes needed to reconfigure for a new client.
- **Priority sort runs before the AI call.** Token budget constraints cut from the lowest-priority tail, not arbitrarily.
- **Broad match is never assigned by the keyword research system under any condition.**
- **The unified business profile activates both systems from a single intake.** No duplicate data entry across platforms.

---

## Technical Stack

| Component | Role |
|---|---|
| Google Ads Scripts | Scheduler, GAQL query, sort, Sheets write, Claude API call |
| Microsoft Ads Scripts | Scheduler, iterator chain, sort, Sheets write, Claude API call |
| Claude API (Anthropic) | Analysis and evaluation engine |
| Google Sheets | Config, output, and audit trail |
| Google Apps Script / UrlFetchApp | Orchestration and API call layer |

---

## Intellectual Property

These systems are proprietary works. All rights reserved.

© 2026 Dominic Andoh Mac-Ennin. Registered with the U.S. Copyright Office (Case #1-15169698851, effective May 23, 2026).

This repository is published for professional documentation and portfolio purposes. The system logic, prompt architecture, classification frameworks, and specification documents are not open-source and are not licensed for use, reproduction, or adaptation without a separate written agreement.

---

## Licensing & Enquiries

Commercial licensing, white-label arrangements, and professional enquiries welcome.

**hello@dominicandohofficial.com**

---

*Dominic Mac-Ennin, CDMP, PCM® — Lead PPC Specialist*
*Google Ads · Microsoft Ads · Meta Ads · AI-Powered SEM Automation*
