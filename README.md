🤖 Multi-Agent Automation & FinTech Pipeline

A production-grade, self-hosted workflow automation system deployed on a Linux VPS. This project integrates a unified Telegram Multi-Agent AI Assistant (supporting both Text and Vision) and an Automated Personal Finance & Shift Tracker.

🛠 Tech Stack

Infrastructure: Linux VPS, Docker Compose, Caddy (Reverse Proxy with auto-TLS)

Core Engine: n8n (Self-Hosted Workflow Automation)

LLM & AI Orchestration: Google Gemini (Vision & Text), DeepSeek, OpenRouter, LangChain Agent Nodes

Integrations: Telegram Bot API, Google Sheets API, Google Calendar API (Primary & TimeTree Sync), Gmail API, Tavily Search API, OpenWeatherMap API, iOS Shortcuts / Webhooks

🚀 Key Features

1. Unified Telegram Multi-Agent Assistant (Text & Vision)

Central Router & Multimodal Entry: Automatically detects incoming Telegram photos vs. text messages and routes requests to specialized sub-agents.

Expense & Shift Agent (ExpenseAgent): Logs daily expenses from text or receipt photos, tracks monthly budgets and saving goals, and cross-references Google Calendar shifts (LP8 / Lifeguard shifts) against Google Sheets to prevent duplicate work logs.

Schedule Agent (ScheduleAgent): Full CRUD operations on Google Calendar with automatic color-coding (Color_ID) based on event participants and read-only synchronization with TimeTree.

Nutrition & Food Agent (FoodAgent): Uses Vision AI to identify meals from photos, estimate weight (g), and calculate calories and macronutrients (carbs, protein, fat) into a daily log.

Daily Briefing & Habit Agent (Daily AI Agent):

06:00 Morning Briefing: Aggregates Hong Kong weather forecasts, Google Calendar + TimeTree schedules, football match countdown reminders, and a daily Bible verse.

12:00 News Digest: Curates top 3 news stories across World, AI & Technology, and Hong Kong using Tavily Search, with conversational pagination (more / continue).

22:30 Habit & Devotional Tracker: Logs daily reflections, reading time, mood scores, and verbatim daily journals into Google Sheets.

Football Team Manager (Football Agent): Automates match-day roster parsing and updates player attendance (✅ / ❌) in Google Sheets.

2. Zero-Click Expense Tracking (FinTech Pipeline)

Automated Bank Notification Parsing: Captures transaction alerts from Mox Bank, BOCHK, HSBC, PayMe, and Fairwood via Gmail polling / iOS Shortcuts Webhooks.

Deduplication & Categorization: Extracts merchant name, amount, payment method (Credit Card, FPS, Auto-Pay), and generates a unique transaction hash (tx_hash) to guarantee idempotent writes to Google Sheets.

📂 Architecture Overview

graph TD
    subgraph Entry_Triggers ["📥 Entry Triggers"]
        TG[Telegram Bot Trigger]
        CRON1[06:00 & 12:00 Cron]
        CRON2[12hr Bank Email / Webhook Trigger]
    end

    subgraph Central_Router ["🧠 Central Routing & Multimodal Layer"]
        IF_PHOTO{Photo Exists?}
        SWITCH{Keyword Router}
        VISION_AGENT[AI Agent - Vision]
        TEXT_AGENT[AI Agent - Text]
        DAILY_AGENT[Daily AI Agent]
        FIN_AGENT[Financial AI Agent]
    end

    subgraph Sub_Agents ["🛠️ Specialized Tool Sub-Agents"]
        EXP_AGENT[Expense & Shift Agent]
        SCHED_AGENT[Schedule Agent]
        FOOD_AGENT[Food & Macro Agent]
        SEARCH_AGENT[Search & Weather Agent]
        FB_AGENT[Football Roster Agent]
    end

    subgraph Storage_APIs ["🗄️ External APIs & Persistence"]
        GS[(Google Sheets DB)]
        GCAL[(Google Calendar & TimeTree)]
        TAVILY[Tavily & OpenWeather]
        GMAIL[Gmail Bank Alerts]
    end

    TG --> IF_PHOTO
    IF_PHOTO -- Yes --> VISION_AGENT
    IF_PHOTO -- No --> SWITCH
    SWITCH -- Habit / News --> DAILY_AGENT
    SWITCH -- General / Commands --> TEXT_AGENT

    CRON1 --> DAILY_AGENT
    CRON2 --> GMAIL --> FIN_AGENT

    VISION_AGENT --> FOOD_AGENT
    VISION_AGENT --> SCHED_AGENT
    VISION_AGENT --> EXP_AGENT

    TEXT_AGENT --> EXP_AGENT
    TEXT_AGENT --> SCHED_AGENT
    TEXT_AGENT --> FOOD_AGENT
    TEXT_AGENT --> SEARCH_AGENT
    TEXT_AGENT --> FB_AGENT

    DAILY_AGENT --> SEARCH_AGENT
    DAILY_AGENT --> GCAL
    DAILY_AGENT --> GS

    FIN_AGENT --> GS
    EXP_AGENT --> GS
    EXP_AGENT --> GCAL
    SCHED_AGENT --> GCAL
    FOOD_AGENT --> GS
    FB_AGENT --> GS
    SEARCH_AGENT --> TAVILY


📁 Repository Structure

├── README.md                              # Project documentation & architecture
├── docker-compose.yml                     # Self-hosted n8n + Caddy deployment stack
└── workflows/
    └── multi-agent-pipeline.json          # Sanitized n8n workflow export (ready to import)


⚙️ Quick Start & Deployment

1. Deploy n8n via Docker Compose

Clone this repository to your Linux VPS and configure your environment variables:

git clone https://github.com/tsekingyeung-lab/multi-agent-automation-pipeline.git
cd multi-agent-automation-pipeline
docker compose up -d


2. Import the Workflow into n8n

Open your self-hosted n8n dashboard.

Go to Workflows -> Import from File and select workflows/multi-agent-pipeline.json.

Configure the following credentials in your n8n instance:

Telegram API: Bot Token & replace YOUR_TELEGRAM_CHAT_ID in the Telegram nodes.

Google OAuth2: Connect Google Sheets, Google Calendar, and Gmail.

LLM Providers: Google Gemini API, DeepSeek API, and OpenRouter API.

Search & Weather: Tavily API Key & OpenWeatherMap API Key.

Replace YOUR_GOOGLE_SHEET_ID and YOUR_FOOTBALL_SHEET_ID with your own Google Spreadsheet IDs.
