# 📱 Deep-Dive Code Review: Bot & Frontend Applications

**Target Directory:** `C:\Coding\VSCode\Quant System`  
**Applications Reviewed:**
1. **`discord-quant-bot`** (Discord alerting bot with Gemini and GEX/DEX chart integration)
2. **`quant-pwa`** (Mobile-first Quant AI PWA with FastAPI Gateway, Canvas Charting, and MCP support)

---

## 1. Executive Summary

Both user-facing applications demonstrate institutional-grade quantitative design, modular architecture, and modern APIs.

| Project | Key Vulnerabilities | UX & Performance | Status |
| :--- | :--- | :--- | :---: |
| **`discord-quant-bot`** | Discord 2000-char message overflow; mention regex bug (`<@!...>`); missing HTTP timeouts | Good Embeds; Zero-trust AllowList | 🟡 **Beta** |
| **`quant-pwa`** | DOM ID mismatch breaking Lightbox; unauthenticated chart/MCP routes; simulated token streaming latency | High (PWA, OLED Dark Theme, 0-dep Canvas) | 🟢 **Near-Prod** |

---

## 2. Findings & Deep-Dive Analysis

---

### 2.1 `quant-pwa`

#### [HIGH] BUG-01: Lightbox & Fullscreen DOM ID Mismatch
- **File:** `quant-pwa/frontend/src/components/lightbox.js` (Lines 3–4) vs `frontend/index.html` (Lines 132–134)
- **Problem:**
  In `index.html`:
  ```html
  <div id="lightboxOverlay" class="lightbox-overlay">
    <div class="lightbox-content">
      <img id="lightboxImg" src="" alt="Chart Full View" />
  ```
  In `lightbox.js`:
  ```javascript
  this.overlay = document.getElementById('lightboxModal'); // null!
  this.image = document.getElementById('lightboxImage');   // null!
  ```
  `this.overlay` is `null`, causing chart zoom and `QuantChart.toggleFullscreen` to fail with unhandled TypeError.
- **Fix:**
  ```javascript
  this.overlay = document.getElementById('lightboxOverlay');
  this.image = document.getElementById('lightboxImg');
  ```

---

#### [HIGH] SEC-05: Unauthenticated Gateway Routes
- **Files:**
  - `quant-pwa/gateway/app/main.py` (Line 218)
  - `quant-pwa/gateway/app/mcp/server.py` (Lines 332, 376)
- **Problem:**
  1. `GET /api/v1/gexdex/chart.png` lacks `dependencies=[Depends(verify_app_passcode)]`.
  2. `/mcp/sse` and `/mcp/messages` endpoints are completely unprotected, exposing tool execution (`get_gexdex`, `get_strike_distribution`) to the open network.
- **Fix:**
  Attach passcode validation dependency to both routes:
  ```python
  @app.get("/api/v1/gexdex/chart.png", dependencies=[Depends(verify_app_passcode)])
  ```

---

#### [MEDIUM] PERF-02: Simulated vs. Real Gemini Token Streaming
- **File:** `quant-pwa/gateway/app/core/agent.py` (Lines 113, 165)
- **Problem:**
  `response = await asyncio.to_thread(chat.send_message, active_prompt)` blocks until the entire Gemini response is generated, and then artificially yields tokens with `asyncio.sleep(0.003)`. This causes a high Time-To-First-Token (TTFT ~2–4 seconds).
- **Fix:**
  Use native Gemini streaming:
  ```python
  response_stream = chat.send_message_stream(active_prompt)
  for chunk in response_stream:
      if chunk.text:
          yield f"data: {json.dumps({'type': 'token', 'content': chunk.text})}\n\n"
  ```

---

#### [MEDIUM] UI-01: Continuous Level Line vs. Discrete Strike Row Drift
- **File:** `quant-pwa/frontend/src/components/quant_chart.js` (Line 520)
- **Problem:**
  Strike rows are spaced uniformly by index (`paddingTop + idx * rowStep`), but level lines (Spot, Call Wall, Put Wall) are calculated via continuous linear normalization `(strike - min) / (max - min)`. On non-linear option chains (e.g. $1 steps ATM, $5 steps OTM), level lines drift away from their matching strike bars.
- **Fix:**
  Compute reference line Y coordinates by interpolating between the discrete Y coordinates of the two bounding strike bar rows.

---

### 2.2 `discord-quant-bot`

#### [MEDIUM] BOT-01: Discord 2000-Character Message Crash & Error Leak
- **File:** `discord-quant-bot/src/main.py` (Lines 62, 68)
- **Problem:**
  1. If Gemini generates analysis exceeding 2,000 characters, `await message.reply(response_text)` throws `discord.errors.HTTPException: 400 Bad Request`.
  2. Line 68 echoes raw python error tracebacks into the Discord channel, exposing internal IPs (`192.168.1.68:8090`).
- **Fix:**
  ```python
  chunks = [response_text[i:i+1900] for i in range(0, len(response_text), 1900)] or [""]
  if image_path and os.path.exists(image_path):
      file = discord.File(image_path, filename=os.path.basename(image_path))
      await message.reply(chunks[0], file=file)
      for extra in chunks[1:]:
          await message.channel.send(extra)
  else:
      for chunk in chunks:
          await message.reply(chunk) if chunk == chunks[0] else await message.channel.send(chunk)
  ```

---

#### [MEDIUM] BOT-02: Nickname Mention Parsing Bug
- **File:** `discord-quant-bot/src/main.py` (Line 47)
- **Problem:**
  `message.content.replace(f'<@{bot.user.id}>', '')` fails when users mention the bot using nickname syntax (`<@!{bot.user.id}>`), leaving corrupted tags in prompts.
- **Fix:**
  ```python
  import re
  query = re.sub(rf'<@!?{bot.user.id}>', '', message.content).strip()
  ```

---

#### [MEDIUM] BOT-03: Missing HTTP Timeouts & Static Temp Files
- **File:** `discord-quant-bot/src/api.py` (Lines 23, 28)
- **Problem:**
  1. `requests.get()` lacks a `timeout` argument and can hang the event loop indefinitely.
  2. `f"{symbol}_gexdex.png"` static filename creates collisions during concurrent queries.
- **Fix:**
  ```python
  response = requests.get(chart_endpoint, headers=headers, timeout=15.0)
  ```
  Use `tempfile.NamedTemporaryFile(suffix=".png", delete=False)` with cleanup in `finally: os.unlink()`.
