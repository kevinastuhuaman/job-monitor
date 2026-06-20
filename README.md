# job-monitor

<p align="center">
  <img src="assets/hero.png" alt="job-monitor hero" width="1100">
</p>

<p align="center">
  <img src="https://img.shields.io/badge/TypeScript-3178C6?style=for-the-badge&logo=typescript&logoColor=white" alt="TypeScript">
  <img src="https://img.shields.io/badge/Node.js-339933?style=for-the-badge&logo=nodedotjs&logoColor=white" alt="Node.js">
  <img src="https://img.shields.io/badge/PostgreSQL-4169E1?style=for-the-badge&logo=postgresql&logoColor=white" alt="PostgreSQL">
  <img src="https://img.shields.io/badge/AWS-232F3E?style=for-the-badge" alt="AWS">
  <img src="https://img.shields.io/badge/Puppeteer-40B5A4?style=for-the-badge" alt="Puppeteer">
</p>

Trackly monitors direct company career pages so job seekers can see roles before they spread across job boards. The public Trackly site currently shows 1,900+ companies and 128,975+ roles, with coverage across 40+ ATS and custom career-page patterns.

> **This is a closed-source project.** The README documents the architecture and learnings.
>
> Public proof: [usetrackly.app](https://usetrackly.app) and [Kevin's proof ledger](https://portfolio.kevinastuhuaman.com/proof/).

## What it does

- Monitors direct company career pages on a scheduled polling loop
- Covers 1,900+ companies and 40+ ATS/custom career-page patterns
- Extracts real posting dates (not "just posted" approximations)
- Classifies jobs by role, level, and location using LLM chain
- Sends alerts for new roles matching saved criteria
- Anti-bot v2 with circuit breaker and feature-flag rollout

## How it works

```
Cron (*/15) ──► Worker ──► Company List ──► ATS Dispatcher
                                                │
                    ┌───────────────────────────┘
                    ▼
              ATS Scraper          ┌─────────────────┐
         (40+ patterns covered)    │   Anti-Bot v2    │
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

- **Anti-bot is an arms race:** circuit breakers with auto-rollback saved us from silent failures. Without them, scrapers would return zero jobs and we'd never know.
- **Composite database indexes turned a 229-second query into 0.3ms:** always profile the database before optimizing application code.
- **Career pages do not behave like one product:** the scraper abstraction layer was the hardest part to get right. Each ATS has its own pagination, date format, and API quirks.
