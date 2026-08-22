# 🟡 Priority 3 Implementation Plan: Quant Level PDF Morning Report Parser Fallback

## 1. Objective
In quant-level-pipeline, if the HTML email body from TradingEdge contains an empty or truncated table, automatically download the attached morning report PDF (.pdf) and extract levels using pdfplumber.

---

## 2. Implementation Specs
1. Add pdfplumber to quant-level-pipeline/requirements.txt.
2. In src/scraper.py, add extract_levels_from_pdf_attachment(attachment_bytes: bytes) -> List[Dict[str, Any]].
3. Add regex table parser extracting Ticker, Support, Resistance, Zero GEX, Trend.
