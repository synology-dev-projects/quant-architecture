---
id: FEAT-003
type: Feature
title: SEC Filings 10-K/10-Q Material Risk Factor Ingestion
priority: P4
status: Active
components: [sec-filings-pipeline, quant-pwa]
impact: Low-Med
effort: Med
target_release: v1.3.0
assignee: QuantFleetCommander
created: 2026-10-21
---

# [FEAT-003] SEC Filings 10-K/10-Q Material Risk Factor Ingestion

## 🎯 Objective
Extract, clean, and structure itemized material risk factors (Item 1A) and management discussion (MD&A) from company SEC EDGAR 10-K and 10-Q filings for ticker drilldowns in Quant PWA.

## 📋 User Story
As an equity derivatives analyst, I want to view top itemized company risks and recent material disclosures directly within a ticker's Cockpit card to understand idiosyncratic tail risks.

## 🏗️ Technical Architecture & Design
1. **SEC Ingestion (`sec-filings-pipeline`)**:
   - Queries SEC EDGAR submissions API with compliant User-Agent headers and 10 req/sec rate limits.
   - Parses Item 1A (Risk Factors) HTML, strips formatting boilerplate, and parses section paragraphs.
2. **Database Storage (`sec_filing_disclosures`)**:
   - Stores parsed text blocks with metadata (CIK, ticker, form type, filing date, fiscal period).
3. **PWA UI Card**:
   - Compact collapsible Bloomberg-style card displaying top 3 concise risk factors for the selected ticker.

## ✅ Acceptance Criteria
- [ ] Pipeline tests in `sec-filings-pipeline/tests/` passing 100%.
- [ ] Strict compliance with SEC EDGAR automated access guidelines (User-Agent formatting, rate limiting).
- [ ] Fast retrieval (< 50ms) via indexed lookup on `(ticker, filing_date DESC)`.
