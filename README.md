<div align="center">

# 🤖 Multi-Agent Automation & FinTech Pipeline

[![n8n](https://img.shields.io/badge/n8n-Workflow_Automation-EA4B71?style=for-the-badge&logo=n8n&logoColor=white)](https://n8n.io/)
[![Docker](https://img.shields.io/badge/Docker-Self--Hosted-2496ED?style=for-the-badge&logo=docker&logoColor=white)](https://www.docker.com/)
[![Caddy](https://img.shields.io/badge/Caddy-Reverse_Proxy-1F88C0?style=for-the-badge&logo=caddy&logoColor=white)](https://caddyserver.com/)
[![Telegram](https://img.shields.io/badge/Telegram-Bot_API-26A5E4?style=for-the-badge&logo=telegram&logoColor=white)](https://core.telegram.org/bots/api)
[![Google Gemini](https://img.shields.io/badge/Google_Gemini-Vision_&_Text-8E75B2?style=for-the-badge&logo=googlegemini&logoColor=white)](https://deepmind.google/technologies/gemini/)

<br>

<!-- LANGUAGE SWITCHER BUTTONS -->
[![English](https://img.shields.io/badge/🇬🇧_Language-English_(Active)-2ea44f?style=for-the-badge)](./README.md)
[![Traditional Chinese](https://img.shields.io/badge/🇭🇰_切換語言-繁體中文版-007ec6?style=for-the-badge)](./README.zh-TW.md)

<p align="center">
  <a href="#-overview">Overview</a> •
  <a href="#️-system-architecture-full-overview">Architecture</a> •
  <a href="#-part-by-part-breakdown-8-core-modules">8 Core Modules</a> •
  <a href="#️-google-sheets-database-schema">Database Schema</a> •
  <a href="#-deployment--setup">Deployment</a> •
  <a href="#-security--privacy-audit">Privacy Audit</a>
</p>

</div>

---

## 📖 Overview
A production-grade, self-hosted workflow automation system deployed on a Linux VPS. This project integrates a unified **Telegram Multimodal AI Assistant** (supporting both Text and Vision) and a **Zero-Click Personal Finance & Shift Tracker**.

## 🛠 Tech Stack
| Layer | Technologies |
| :--- | :--- |
| **Infrastructure** | Linux VPS, Docker Compose, Caddy (Auto-TLS Reverse Proxy) |
| **Core Engine** | n8n (Self-Hosted Workflow Automation) |
| **AI & LLMs** | Google Gemini 2.5 Flash (Vision & Text), DeepSeek Chat, OpenRouter, LangChain Agents |
| **Integrations** | Telegram Bot API, Google Sheets API, Google Calendar API (Primary & Shared Sync), Gmail API, Tavily Search, OpenWeatherMap |

---

## 🗺️ System Architecture (Full Overview)

```mermaid
graph TD
    subgraph Triggers ["📥 1. Entry Triggers"]
        TG[Telegram Bot Trigger]
        CRON_DAILY[06:00 / 12:00 / 22:30 Cron]
        CRON_FIN[12hr Bank Alert Trigger]
    end

    subgraph Router ["🧠 2. Central Routing & Multimodal Layer"]
        PHOTO_CHECK{Photo Exists?}
        SWITCH_ROUTER{Keyword Switch}
        VISION_AGENT[AI Agent - Vision]
        TEXT_AGENT[AI Agent - Text]
        DAILY_AGENT[Daily Briefing & Habit Agent]
        FIN_AGENT[Financial Bookkeeping Agent]
    end

    subgraph SubAgents ["🛠️ 3. Specialized Tool Sub-Agents"]
        EXP_AGENT[Expense & Shift Agent]
        SCHED_AGENT[Schedule CRUD Agent]
        FOOD_AGENT[Food & Macro Agent]
        SEARCH_AGENT[Web Search & Weather Agent]
        ROSTER_AGENT[Team Roster Agent]
    end

    subgraph Storage ["🗄️ 4. External APIs & Persistence"]
        GS[(Google Sheets DB)]
        GCAL[(Google Calendar)]
        WEB[Tavily & OpenWeather]
        GMAIL[Gmail Bank Alerts]
    end

    TG --> PHOTO_CHECK
    PHOTO_CHECK -- Yes --> VISION_AGENT
    PHOTO_CHECK -- No --> SWITCH_ROUTER
    SWITCH_ROUTER -- Habit / News --> DAILY_AGENT
    SWITCH_ROUTER -- General / Commands --> TEXT_AGENT

    CRON_DAILY --> DAILY_AGENT
    CRON_FIN --> GMAIL --> FIN_AGENT

    VISION_AGENT --> FOOD_AGENT & SCHED_AGENT & EXP_AGENT
    TEXT_AGENT --> EXP_AGENT & SCHED_AGENT & FOOD_AGENT & SEARCH_AGENT & ROSTER_AGENT
    DAILY_AGENT --> SEARCH_AGENT & GCAL & GS
    FIN_AGENT --> GS
```

---

## 🧩 Part-by-Part Breakdown (8 Core Modules)

Click on any module below to expand its architecture diagram and technical logic.

<details open>
<summary><b>🔹 Module 1: Central Multimodal Router (<code>01-central-router-and-agents.json</code>)</b></summary>
<br>

**What it does:**
Acts as the single Telegram entry point. It inspects incoming webhook payloads for binary photos vs. text, routes image payloads to `AI Agent (Vision)` and text to a regex `Switch` router, and enforces strict Telegram HTML formatting (`<b>`, `<i>`, `<blockquote>`) on all outgoing messages.

```mermaid
graph LR
    TG[Telegram Trigger] --> IF{Has Photo?}
    IF -- Yes --> GET[Get Binary File] --> VA[AI Agent - Vision]
    IF -- No --> SW{Regex Switch}
    SW -- Habit / News --> DA[Daily AI Agent]
    SW -- Commands / Chat --> TA[AI Agent - Text]
    VA & TA & DA --> CLEAN[JS Clean HTML] --> OUT[Telegram Reply]
```
</details>

<details>
<summary><b>🔹 Module 2: Expense, Budget & Work Shift Sync Sub-Agent (<code>ExpenseAgent</code>)</b></summary>
<br>

**What it does:**
Manages personal finances, monthly saving goals, and part-time/shift work tracking:
* **Receipt & Text Bookkeeping:** Extracts merchant, amount, category, and payment method from receipt photos or text messages, appending them to Google Sheets.
* **Budget & Saving Goal Management:** Updates monthly budget and saving targets (`config_row`) and calculates how many work shifts are required to hit the target.
* **Anti-Duplicate Work Shift Sync:** When triggered with *"Update work records"*, it queries Google Calendar for shift events, cross-references existing rows in the `Work_Records` sheet, and appends only newly completed shifts.

```mermaid
graph LR
    IN[User / Vision Input] --> EA[ExpenseAgent]
    EA --> SAVE[SaveExpense Tool]
    EA --> GOAL[UpdateBudgetGoal Tool]
    EA --> CAL[GetCalendarEvents] --> DIFF{Cross-Check Existing} --> LOG[LogWorkShift]
    SAVE & GOAL & LOG --> READ[ReadExpense Summary]
```
</details>

<details>
<summary><b>🔹 Module 3: Dual-Calendar CRUD & Color-Coding Sub-Agent (<code>ScheduleAgent</code>)</b></summary>
<br>

**What it does:**
Manages personal and shared schedules across two Google Calendars:
* **Access Control:** Full CRUD (`Create`, `Query`, `Update`, `Delete`) on the primary personal calendar, and strict read-only `Query` enforcement on the imported shared calendar (TimeTree sync).
* **Automatic Color-Coding (`Color_ID`):** Assigns Google Calendar color IDs automatically based on participants (`2` Green for solo, `5` Yellow for partner, `9` Blue for joint events, `4` Flamingo for anniversaries) and supports `RRULE` recurring events.

```mermaid
graph LR
    IN[Schedule Request / Flyer Photo] --> SA[ScheduleAgent]
    SA -- Personal Calendar --> CRUD[Create / Query / Update / Delete]
    SA -- Shared TimeTree Calendar --> Q_ONLY[Read-Only Query]
```
</details>

<details>
<summary><b>🔹 Module 4: Vision Food & Macronutrient Sub-Agent (<code>FoodAgent</code>)</b></summary>
<br>

**What it does:**
Triggered when a meal photo is sent to Telegram:
* Identifies up to 3 primary food items (sorted by calorie density) and incorporates user captions.
* Estimates weight (g), total calories (kcal), carbohydrates (g), protein (g), and fat (g).
* Calls `SaveFood` to log the meal, then immediately calls `CalFood` to aggregate and return the user's cumulative daily nutrition totals.

```mermaid
graph LR
    PHOTO[Meal Photo + Caption] --> VA[Vision Agent] --> FA[FoodAgent]
    FA --> SAVE[SaveFood: Append Row] --> CAL[CalFood: Sum Today's Macros] --> REP[Nutrition Report]
```
</details>

<details>
<summary><b>🔹 Module 5: Web Research, Categorized News & Weather Sub-Agent (<code>Search Agent</code>)</b></summary>
<br>

**What it does:**
Provides real-time external context using **Tavily Search** and **OpenWeatherMap**:
* Fetches current weather conditions, temperature ranges, and rain probabilities for morning outfit/umbrella recommendations.
* Searches and curates news across three categories: **World**, **AI & Technology**, and **Local (Hong Kong)**.
</details>

<details>
<summary><b>🔹 Module 6: Sports Team Roster & Attendance Sub-Agent (<code>RosterAgent</code>)</b></summary>
<br>

**What it does:**
Automates sports team management:
* Reads the master player roster from Google Sheets (`Read` tool).
* Parses unstructured match-day lineup messages from Telegram and updates every player's attendance column with `✅` or `❌` (`Attendance` tool), returning a headcount summary.
</details>

<details>
<summary><b>🔹 Module 7: Daily Briefing, News Pagination & Habit Tracker (<code>02-daily-briefing-and-habit-tracker.json</code>)</b></summary>
<br>

**What it does:**
Runs on scheduled Cron triggers and interactive replies:
* **06:00 Morning Briefing:** Combines weather, primary + shared calendar events, automated match administrative countdowns (14-day venue draw, 13-day poll, 1-day lineup, -1-day referee fee), and a random daily verse.
* **12:00 News Digest & Memory Pagination:** Delivers 9 curated news items (3 per category). Replying `more` or `continue` checks `Simple Memory` (20-message window) and fetches the next 3 unseen articles.
* **22:30 Habit & Verbatim Journal Tracker:** Prompts for evening reflections, reading minutes, mood score (1–10), and logs the user's daily journal **100% verbatim without summarization** into `Habit_Tracker`.

```mermaid
graph LR
    C1[06:00 Cron] --> JS[Countdown & Verse JS]
    C2[12:00 Cron] --> NEWS[3x3 News Digest]
    C3[22:30 Cron] --> HABIT[Habit Prompt]
    JS & NEWS & HABIT --> DA[Daily AI Agent] <--> MEM[(20-Msg Window Memory)]
    DA --> GS[(Habit_Tracker Sheet)]
```
</details>

<details>
<summary><b>🔹 Module 8: Zero-Click FinTech Auto-Bookkeeping (<code>03-fintech-auto-bookkeeping.json</code>)</b></summary>
<br>

**What it does:**
Automates expense and income tracking from banking alerts (Mox Bank, BOCHK, HSBC, PayMe, FPS):
* Polls unprocessed banking emails every 12 hours (or receives payloads via iOS Shortcuts Webhook).
* Extracts 7 standardized fields and generates a deterministic `tx_hash` (Authorization reference or `YYYYMMDD_AMOUNT_MERCHANT`).
* Queries `check_existing_transactions` first—only appending new transactions and applying the `Processed` Gmail label afterward.

```mermaid
graph LR
    CRON[12hr Trigger] --> GM[Fetch Unlabeled Bank Emails]
    GM --> FA[Financial AI Agent]
    FA <--> CHK[(Check Existing Hash)]
    FA --> ADD[(Append New Row)]
    FA --> LBL[Apply 'Processed' Gmail Label]
```
</details>

---

## 🗄️ Google Sheets Database Schema

To run this pipeline, create a Google Spreadsheet with the following **5 tabs**:

| Sheet Tab Name | Purpose | Required Columns |
| :--- | :--- | :--- |
| **`自動記帳資料庫`**<br>*(Auto Bookkeeping DB)* | Stores all expenses & FPS transfers | `日期時間`, `商戶/交易對象`, `金額`, `交易類型`, `支付方式`, `備註/原始通知`, `交易ID/Hash` |
| **`每月財務摘要`**<br>*(Monthly Financial Summary)* | Stores global budget config & formulas | `ID` (`config_row`), `每月薪金預算 ($)`, `目標儲蓄 ($)`, `每次返工日薪/時薪 ($)`, `總累計收入`, `總累計開支`, `目前已返幾多工` |
| **`返工紀錄`**<br>*(Work Shift Records)* | Synced work shifts from Google Calendar | `日期`, `工作名稱/地點`, `返工日數/時數`, `預計人工 ($)`, `備註` |
| **`Habit_Tracker`**<br>*(Daily Habit & Journal)* | Evening check-ins & verbatim journals | `Date`, `與主對話`, `讀經／書 (min)`, `心情 (1-10)`, `今日發生咩事 (完整紀錄)` |
| **`個人飲食日誌`**<br>*(Personal Food Log)* | Meal macro & calorie logs | `date`, `name`, `calories`, `carbohydrate`, `protein`, `fat` |

---

## 🚀 Deployment & Setup

### Repository Structure
```text
📦 multi-agent-automation-pipeline
 ┣ 📂 workflows
 ┃ ┣ 📜 00-full-multi-agent-pipeline.json          # Complete All-in-One Workflow
 ┃ ┣ 📜 01-central-router-and-agents.json          # Part 1: Telegram Router + 5 Sub-Agents
 ┃ ┣ 📜 02-daily-briefing-and-habit-tracker.json   # Part 2: Morning Briefing, News & Habit Tracker
 ┃ ┗ 📜 03-fintech-auto-bookkeeping.json           # Part 3: Automated Bank Alert Bookkeeping
 ┣ 📜 docker-compose.yml                           # Self-Hosted n8n + Caddy Stack
 ┣ 📜 Caddyfile                                    # Reverse Proxy Auto-HTTPS Config
 ┣ 📜 .env.example                                 # Environment Variables Template
 ┣ 📜 README.md                                    # English Documentation (Default)
 ┗ 📜 README.zh-TW.md                              # Traditional Chinese Documentation
```

### Quick Start
1. **Clone & Configure Environment:**
   ```bash
   git clone https://github.com/tsekingyeung-lab/multi-agent-automation-pipeline.git
   cd multi-agent-automation-pipeline
   cp .env.example .env
   nano .env
   ```
2. **Launch Docker Stack:**
   ```bash
   docker compose up -d
   ```
3. **Import Workflows into n8n:**
   Import either `workflows/00-full-multi-agent-pipeline.json` (complete system) or the modular files (`01`–`03`) into your n8n UI, and bind your credentials.

---

## 🔒 Security & Privacy Audit

All files in this repository have been thoroughly sanitized before publication:
- **Personal Emails & Calendar IDs:** Replaced with `YOUR_PRIMARY_EMAIL@gmail.com` and `YOUR_SHARED_CALENDAR_ID@import.calendar.google.com`.
- **Telegram Identifiers:** Chat IDs and Webhook UUIDs replaced with `YOUR_TELEGRAM_CHAT_ID` and placeholder strings.
- **Google Spreadsheet IDs:** Document IDs replaced with `YOUR_GOOGLE_SHEET_ID` and `YOUR_ROSTER_SHEET_ID`.
- **API Credentials:** Internal n8n credential IDs replaced with `YOUR_*_CREDENTIAL_ID` placeholders.
- **Personal/Workplace References:** Specific shift codes, team names, and opponent names generalized to standard placeholders.
