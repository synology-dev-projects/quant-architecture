# Discord Quant Bot: Final Architecture Blueprint

The planning phase is complete. Here is the final FAANG-grade blueprint for the entire system based on your decisions.

## Infrastructure & Hosting
* **Deployment Location:** Standalone Docker container strictly on the Synology NAS.
* **Separation of Concerns:** The bot lives in its own repository (`discord-quant-bot`), completely decoupled from the `gexdex-api`.
* **Execution Model:** The bot acts as an external HTTP client, hitting `gexdex-api-prod:8000`. It does *not* compute quant logic natively.

## Data & LLM Strategy
* **Caching:** None. Every request pulls fresh live data directly from the API.
* **Data Flow:** The bot sends raw, untruncated JSON options data directly into the Gemini API context window for maximum analytical accuracy.
* **Secrets:** Discord tokens and Gemini API keys will be managed via GitHub Actions Secrets and injected securely during deployment.

## User Experience & Security
* **Interaction Model (NLP):** We will use **Conversational AI (NLP)**. Because you want Gemini to run deep analysis right in the chat, NLP is far superior. You can literally ask, *"Can you run an analysis on AAPL and tell me where the put wall is?"* and Gemini will dynamically trigger the tools, read the data, and write a custom analysis right in the channel.
* **Zero-Trust Security:** We will implement a hardcoded **AllowList**. The bot will drop 100% of messages from any Discord User ID that doesn't match yours.

---

> [!IMPORTANT]
> **Plan Locked In!** 
> If you are ready, I will move into Execution Mode. I will scaffold the new repository on your machine, write the Python Discord bot code with the Gemini API integration, and prepare the Docker files!
