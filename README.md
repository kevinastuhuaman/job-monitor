# Trackly Job Discovery Pipeline

A public architecture case study for operating a multi-source job discovery system without confusing missing data with an empty market.

**Public evidence, July 12, 2026:** 1,969 monitored company career sites, 40 ATS and source types, and 128,975 job records.

[Open Trackly](https://usetrackly.app) | [Inspect the proof ledger](https://portfolio.kevinastuhuaman.com/proof/) | [See the AI product stack](https://kevinastuhuaman.github.io/ai-product-builder-stack/) | [Use the public CLI + MCP](https://github.com/trackly-app/trackly-cli)

This repository documents the product architecture, operating controls, and infrastructure evolution. It does not publish proprietary source adapters, customer data, credentials, private topology, or bypass techniques.

## The product problem

Company career sites do not behave like one product. They expose different APIs, pagination rules, date semantics, identifiers, locations, and failure modes. A source that returns zero jobs might represent a real empty result, a changed ATS slug, a blocked request, a parser regression, a deactivated company, or a temporary upstream incident.

The system therefore treats source reliability and evidence quality as product behavior, not background infrastructure.

## Reference pipeline

```text
Monitored company catalog
        |
        v
Scheduled source workers
        |
        v
ATS and career-site adapter layer
        |
        v
Normalize identifiers, dates, locations, and descriptions
        |
        v
Deduplicate and preserve source provenance in PostgreSQL
        |
        v
Freshness, false-zero, and lifecycle guards
        |
        v
LLM classification and search enrichment
        |
        v
Matching, alerts, web, native apps, CLI, MCP, chat, and voice
```

## Reliability contracts

| Failure mode | Product control |
| --- | --- |
| Source suddenly returns zero | Compare with known history; hold destructive lifecycle changes until the result is verified |
| One adapter throws | Isolate the company failure so the rest of the polling batch completes |
| Posting disappears temporarily | Separate source availability from job lifecycle state |
| ATS identifier changes | Detect stale or invalid source configuration and route it for rediscovery |
| Description is missing | Treat incomplete content as a data-quality failure before classification |
| Model or prompt changes | Run golden-set and regression evals before promotion |
| Fleet coverage drifts | Watch stale pollers, never-polled sources, error spikes, and dropped-company accounting |

The core operating principle is fail loud, preserve known-good data, and retry the smallest uncertain boundary.

## Current production architecture

Trackly production moved from AWS to Azure on June 30, 2026.

| Layer | Current role |
| --- | --- |
| Azure App Service | Production API, source workers, and scoring services |
| Azure PostgreSQL | Blue production database behind private networking |
| Azure AI Search | Hybrid retrieval for job search and product experiences |
| Azure OpenAI and model fallbacks | Classification, enrichment, and user-facing AI workflows |
| Application Insights and Log Analytics | Runtime observability and incident evidence |
| GitHub Actions | Tested deployments and operational verification |
| Terraform | Audit-first inventory and read-only drift detection |

Legacy AWS Lightsail and RDS resources are rollback or reference infrastructure after the cutover, not the live production source of truth. Langfuse and a small number of separate dependencies remain migration-specific exceptions rather than evidence that the primary product still runs on AWS.

## Infrastructure evolution

| Earlier system | Current system | Product reason |
| --- | --- | --- |
| AWS Lightsail API and workers | Azure App Service workloads | Standardize deployment and post-cutover operations |
| AWS RDS PostgreSQL | Azure blue PostgreSQL | Keep the live database close to the Azure application path |
| Ad hoc infrastructure knowledge | Audit-first Terraform and deferred-resource checks | Make ownership, drift, and rollback boundaries inspectable |
| Basic uptime checks | Fleet watchdogs plus Application Insights evidence | Detect silent data-quality regressions, not only server downtime |

The migration was not presented as a clean-room rewrite. Rollback resources, staging, public hostnames, search, runners, and observability moved through explicit readiness and verification gates.

## Product decisions

### Provenance survives normalization

Every normalized job retains enough source context to explain where it came from and whether the evidence is current. Search quality is not useful if a user cannot trust the underlying posting.

### Empty is an outcome, not an assumption

The product distinguishes a verified empty source from a failed read. That prevents a transient upstream problem from silently deleting inventory or hiding a company from users.

### Models do not own lifecycle truth

LLMs classify and enrich records; they do not decide whether a job exists. Source identity, polling evidence, and deterministic lifecycle guards remain separate from model output.

### Operations are part of product quality

Fleet coverage, stale sources, false-zero rates, and dropped work affect the user promise directly. They belong in release criteria and product review, not only an engineering dashboard.

## What this demonstrates

- Product judgment for reliability, uncertainty, and evidence provenance.
- Technical fluency across source adapters, PostgreSQL, hybrid search, LLM evals, cloud migration, CI/CD, and observability.
- Experience operating one product across web, iOS, macOS, CLI, MCP, chat, and voice surfaces.
- The ability to translate infrastructure work into user-visible quality and clear product controls.

## Public boundary

This is a public-safe architecture artifact. It contains no proprietary adapters, private endpoints, credentials, customer or applicant data, source-bypass techniques, cloud identifiers, or production topology details. See [`IP-NOTICE.md`](IP-NOTICE.md), [`llms.txt`](llms.txt), and [`project.json`](project.json).
