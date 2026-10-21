---
id: ENH-002
type: Enhancement
title: Quant Level Email Attachment PDF Fallback Parser
priority: P3
status: Active
components: [quant-level-pipeline]
impact: Low-Med
effort: Low
target_release: v1.2.0
assignee: QuantFleetCommander
created: 2026-10-21
---

# [ENH-002] Quant Level Email Attachment PDF Fallback Parser

## 🎯 Objective
Equip the Quant Level Ingestion Pipeline with an automated PDF fallback parser (`pdfplumber` / `pypdf`) to extract structured price support and resistance tables when email reports are sent exclusively as PDF attachments without HTML table bodies.

## 📋 User Story
As a data engineer, I want the Quant Level pipeline to parse attached PDF morning reports automatically when Mighty Networks sends daily levels in PDF format, eliminating manual data entry.

## 🏗️ Technical Architecture & Design
1. **Fallback Extractor (`quant-level-pipeline/src/pdf_parser.py`)**:
   - Detects when email body text contains no extractable price tables but has `.pdf` attachments.
   - Extracts tabular data using `pdfplumber.extract_tables()`.
   - Normalizes column names (Ticker, Support, Resistance, Major Levels, Gamma Neutral).
2. **Safe Transformation**:
   - Casts numeric price strings to standard floats with 2-digit to 6-digit regex validation.
   - Reuses existing `transform_levels()` logic and loads into `quant_levels` table.

## ✅ Acceptance Criteria
- [ ] Offline unit test with mock sample morning report PDF passing in `quant-level-pipeline/tests/test_pdf_parser.py`.
- [ ] Memory limit maintained: stream PDF bytes directly without writing unmanaged temporary files to disk.
- [ ] Zero crash fallback: if PDF parsing fails, log structured warning and exit cleanly with code 0.
