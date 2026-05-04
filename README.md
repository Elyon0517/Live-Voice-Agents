# Live-Voice-Agents

Learning project for building AI news agents and live voice/podcast agents with Google ADK (Agent Development Kit).

## Project Structure

```
app_01/
├── agent.py          # Single agent: AI news researcher with callback system
├── multi-agents.py   # Multi-agent: AI news researcher + podcast audio generator
├── __init__.py       # Exposes root_agent for ADK runner
├── .env              # API keys (not committed)
└── .env.example      # Environment variable template
```

## Agents

### `agent.py` — AI News Research Agent

A single `root_agent` (`ai_news_research_coordinator`) that:

1. Searches for recent AI news using **DuckDuckGo** (`search_web`)
2. Extracts NASDAQ-listed company stock tickers from results
3. Fetches real-time stock prices and daily change via **yfinance** (`get_financial_context`)
4. Formats findings into a structured Markdown report and saves it as `ai_research_report.md`

**Callback system:**
- `filter_news_sources_callback` (before tool) — blocks queries targeting low-quality domains (Wikipedia, Reddit, YouTube, etc.)
- `inject_process_log_after_search` (after tool) — extracts source domains and injects a `process_log` into the tool response for transparency

---

### `multi-agents.py` — AI News Podcast Producer (Multi-Agent)

An orchestrated multi-agent pipeline with two agents:

#### `root_agent` (`ai_news_researcher`)
Coordinates the full end-to-end workflow:
1. Searches for recent AI news via **Google Search** (whitelisted domains only)
2. Enriches results with real-time stock data via **yfinance**
3. Structures findings into an `AINewsReport` (Pydantic schema)
4. Saves a Markdown report to `ai_research_report.md`
5. Writes a two-host conversational podcast script (Joe & Jane)
6. Delegates audio generation to `podcaster_agent`

**Callback system:**
- `filter_news_sources_callback` — rewrites queries to only search whitelisted news domains (TechCrunch, VentureBeat, The Verge, MIT Technology Review, Ars Technica)
- `enforce_data_freshness_callback` — appends `tbs=qdr:w` to limit results to the past week
- `inject_process_log_after_search` — records source domains into session state for audit logging

#### `podcaster_agent`
A specialist sub-agent that receives a podcast script and calls `generate_podcast_audio` to produce a two-speaker WAV file using **Gemini TTS** (`gemini-2.5-flash-preview-tts`), with Joe (voice: Kore) and Jane (voice: Puck).

**Output files:**
- `ai_research_report.md` — structured news report
- `ai_today_podcast.wav` — generated podcast audio

## Setup

### 1. Clone and create virtual environment

```bash
git clone git@github.com:Elyon0517/Live-Voice-Agents.git
cd Live-Voice-Agents
python -m venv .venv
source .venv/bin/activate
```

### 2. Install dependencies

```bash
pip install google-adk yfinance ddgs pydantic
```

### 3. Configure environment variables

```bash
cp .env.example .env
```

Edit `.env` and fill in your API key:

```
GOOGLE_GENAI_USE_VERTEXAI=0
GOOGLE_API_KEY=your_google_api_key_here
```

### 4. Run with ADK

```bash
# Launch the web UI
adk web

# Or run in terminal
adk run app_01
```

## Official Documentation

- [ADK Python GitHub](https://github.com/google/adk-python)
- [ADK Java GitHub](https://github.com/google/adk-java)
- [ADK Documentation](https://google.github.io/adk-docs)
- [Gemini API Documentation](https://ai.google.dev/gemini-api)
- [Model Information](https://ai.google.dev/gemini-api/docs/models)
- [Sample Agents](https://github.com/google/adk-samples)
