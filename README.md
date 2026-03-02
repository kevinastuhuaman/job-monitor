# job-monitor 🕷️ — Scrapes 775 companies across 22 ATS types, every 15 minutes.

<p align="center">
  <img src="assets/hero.png" alt="job-monitor hero" width="1100">
</p>

<p align="center">
  <img src="https://img.shields.io/badge/TypeScript-3178C6?style=for-the-badge&logo=typescript&logoColor=white" alt="TypeScript">
  <img src="https://img.shields.io/badge/Node.js-339933?style=for-the-badge&logo=nodedotjs&logoColor=white" alt="Node.js">
  <img src="https://img.shields.io/badge/PostgreSQL-4169E1?style=for-the-badge&logo=postgresql&logoColor=white" alt="PostgreSQL">
  <img src="https://img.shields.io/badge/AWS-232F3E?style=for-the-badge&logo=amazonwebservices&logoColor=white" alt="AWS">
  <img src="https://img.shields.io/badge/Puppeteer-40B5A4?style=for-the-badge&logo=puppeteer&logoColor=white" alt="Puppeteer">
</p>

A production job monitoring system that scrapes 775 companies across 22 applicant tracking systems (Greenhouse, Ashby, Lever, SmartRecruiters, Workday, and 17 others) on a 15-minute cron. Tracks ~94,000 job postings with real posting date extraction, deduplication, and classification.

> **This is a closed-source project.** The README documents the architecture and learnings.

## What it does

- Monitors 775 companies every 15 minutes via cron
- Supports 22 ATS types with custom scrapers for each
- Extracts real posting dates (not "just posted" approximations)
- Classifies jobs by role, level, and location using LLM chain
- Sends Slack alerts for new PM roles matching criteria
- Anti-bot v2 with circuit breaker and feature-flag rollout

## How it works

```
Cron (*/15) ──► Worker ──► Company List ──► ATS Dispatcher
                                                │
                    ┌───────────────────────────┘
                    ▼
              ATS Scraper          ┌─────────────────┐
           (22 types supported)    │   Anti-Bot v2    │
                    │              │  - Rotating IPs  │
                    │◄────────────►│  - Fingerprints  │
                    │              │  - Circuit Break  │
                    ▼              └─────────────────┘
              Normalize Data
                    │
                    ▼
          Deduplicate (PostgreSQL)
                    │
                    ▼
        ┌──────────────────────┐
        │   6-Tier LLM Chain   │
        │ Azure ► Agent SDK    │
        │ ► Bedrock ► Foundry  │
        │ ► Vertex ► Anthropic │
        └──────────────────────┘
                    │
                    ▼
            Slack Alerts (#trackly-critical)
```

Anti-bot system uses rotating proxies, browser fingerprint randomization, and circuit breaker pattern (auto-rollback if zero-job rate exceeds threshold).

## Tech stack

- **Runtime:** Node.js on AWS Lightsail (xlarge)
- **Database:** PostgreSQL (RDS), composite indexes for sub-millisecond queries
- **AI:** 6-tier LLM chain (Azure > Agent SDK > Bedrock > Foundry > Vertex > Anthropic)
- **Infra:** GitHub Actions CI/CD, PM2 process manager, Healthchecks.io
- **Monitoring:** Slack (#trackly-critical, #trackly-info), Uptime Kuma

## What I learned

- **Anti-bot is an arms race** — circuit breakers with auto-rollback saved us from silent failures. Without them, scrapers would return zero jobs and we'd never know.
- **Composite database indexes turned a 229-second query into 0.3ms** — always profile the database before optimizing application code.
- **22 ATS types means 22 different HTML structures** — the scraper abstraction layer was the hardest part to get right. Each ATS has its own pagination, date format, and API quirks.
