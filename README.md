# Multi-Agent Automation & FinTech Pipeline

A production-grade, self-hosted workflow automation system deployed on a Linux VPS. This project integrates a unified Telegram AI Assistant and an automated personal finance tracker.

## 🛠 Tech Stack
- **Infrastructure:** Linux VPS, Docker Compose, Caddy (Reverse Proxy)
- **Core Engine:** n8n (Workflow Automation)
- **Integrations:** Telegram Bot API, Google Sheets API, iOS Shortcuts, Webhooks

## 🚀 Features
1. **Telegram Multi-Agent Assistant:** Orchestrates sub-agents for schedule queries, morning news briefings, and ad-hoc web searches.
2. **Zero-Click Expense Tracking:** Captures mobile banking push notifications (HSBC, PayMe, Mox Bank) via iOS Shortcuts and forwards payload via Webhooks to n8n for automated Google Sheets categorization.

## 📂 Architecture Overview
(建議在這裡放一張你自己畫的流程圖，或者系統運作的截圖/GIF)
