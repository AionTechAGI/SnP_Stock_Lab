# SnP Stock Lab — Monster Execution Plan v2

> Based on the finalized `PLAN.md` decisions.  
> Planning language: Russian. Repo/docs/prompts/code artifacts: English-first unless explicitly noted.  
> Product posture: **US-first paid SaaS for swing traders**, S&P 500-only, research dashboard, no brokerage, no personalized advice, no Finviz clone.

---

## 0. Самая короткая формула продукта

**SnP Stock Lab** — это платная SaaS-платформа для swing traders, которая показывает состояние S&P 500, секторную ротацию, heatmap, radar движений, top long/short candidates, ticker research, Strategy Lab Lite, Watchlist Lite и AI Analyst, который отвечает только на основе данных платформы.

Продукт должен ощущаться как:

```txt
Finviz-like market clarity
+ AI evidence-grounded explanations
+ deterministic long/short scoring
+ sector rotation engine
+ watchlist change intelligence
+ strategy hypothesis testing
- brokerage
- personalized investment advice
- all-market chaos
- unlicensed scraping
```

Главный пользовательский момент:

```txt
Пользователь открывает Home и за 30 секунд понимает:
1. Что происходит с S&P 500.
2. Какие сектора сильные/слабые.
3. Какие акции в игре.
4. Какие 5 long и 5 short candidates сейчас самые интересные.
5. Почему они попали в список.
6. Где риск и invalidation.
7. Что стоит открыть глубже.
```

---

## 1. Verified external anchors

Эти источники используются как технические и продуктово-правовые якоря. Их надо добавить в `docs/REFERENCE_ANCHORS.md`.

### 1.1. S&P 500 scope

S&P Dow Jones Indices описывает S&P 500 как индекс 500 ведущих компаний США, покрывающий примерно 80% доступной рыночной капитализации. Это подтверждает продуктовую ставку на S&P 500-only universe.

```txt
Source: https://www.spglobal.com/spdji/en/indices/equity/sp-500/
Use in product:
- S&P 500-only scope
- large-cap US equity positioning
- trademark/legal review requirement
```

### 1.2. SEC EDGAR APIs

SEC предоставляет `data.sec.gov` APIs для submissions и XBRL facts. Это хороший фундаментальный источник для filings/fundamentals, но без CORS и с fair-access ограничениями.

```txt
Sources:
- https://www.sec.gov/search-filings/edgar-application-programming-interfaces
- https://www.sec.gov/search-filings/edgar-search-assistance/accessing-edgar-data

Key implementation notes:
- use server-side ingestion, not browser calls;
- declare User-Agent;
- respect 10 requests/second max;
- download only what is needed;
- cache aggressively;
- never hammer SEC from user requests.
```

### 1.3. FMP S&P 500 API

FMP documents an S&P 500 constituents API that returns company-level constituent data including symbol, company name, sector/sub-sector and historical additions.

```txt
Source: https://site.financialmodelingprep.com/developer/docs/stable/sp-500
Use:
- candidate provider for constituents;
- compare with other providers;
- license review before production.
```

### 1.4. yfinance warning

The yfinance repository says it is not affiliated, endorsed or vetted by Yahoo, is intended for research/educational purposes, and Yahoo Finance API is intended for personal use only. Therefore yfinance can only be allowed in local risk-mode experimentation, not as an unreviewed public paid SaaS data backbone.

```txt
Source: https://github.com/ranaroussi/yfinance
Policy:
- allowed only behind provider abstraction in risk-mode/dev;
- disabled by kill-switch for public launch unless legal signs off;
- never represent as licensed commercial market data.
```

### 1.5. OpenAI Codex subagents/custom agents

Codex supports spawning specialized subagents in parallel and collecting their results. Custom agents can be project-scoped under `.codex/agents/` and should include `name`, `description`, and `developer_instructions`.

```txt
Source: https://developers.openai.com/codex/subagents
Use:
- parallel read-heavy planning/review;
- sequential write-heavy implementation;
- high-reasoning subagents;
- custom TOML agent definitions.
```

### 1.6. AGENTS.md

Codex supports repository-level `AGENTS.md` and directory-level instruction layering. Use root `AGENTS.md` for global product/engineering rules and local `AGENTS.md` only where truly needed.

```txt
Source: https://developers.openai.com/codex/guides/agents-md
Use:
- root project doctrine;
- directory-level rules for api/web/workers if necessary;
- avoid conflicting instructions.
```

### 1.7. Codex Skills

Codex Skills are reusable workflows. They should be used for repeated reviews: financial data contracts, scoring/backtest review, AI evidence review, dashboard UI review, compliance review.

```txt
Source: https://developers.openai.com/codex/skills
Use:
- reusable agent workflows;
- keep recurring checklists out of one-off prompts;
- invoke skills on relevant work.
```

### 1.8. OpenAI function calling strict mode

OpenAI function calling docs recommend strict mode to make function calls adhere to schemas. SnP Stock Lab AI Analyst should use strict structured outputs/tools wherever possible.

```txt
Source: https://developers.openai.com/api/docs/guides/function-calling
Use:
- strict structured tool calls;
- additionalProperties=false;
- required fields;
- server-side allowlist.
```

### 1.9. Google Labs DESIGN.md

Google Labs `DESIGN.md` describes a structured visual identity format for coding agents, including linting and export to Tailwind/DTCG. Use this as the product design-system contract.

```txt
Source: https://github.com/google-labs-code/design.md
Use:
- root DESIGN.md;
- tokenized design rules;
- optional export to Tailwind theme;
- design drift review.
```

### 1.10. Clerk/Stripe deployment anchors

Clerk docs support organizations, roles, permissions and invitations. Stripe docs describe subscription webhooks and event verification. Use them for SaaS foundation.

```txt
Sources:
- https://clerk.com/docs/guides/organizations/control-access/roles-and-permissions
- https://clerk.com/docs/guides/organizations/add-members/invitations
- https://docs.stripe.com/billing/subscriptions/webhooks
- https://docs.stripe.com/webhooks

Use:
- org-based SaaS from day one;
- Admin/Member roles;
- per-seat Pro plan;
- webhook idempotency and signature verification.
```

---

## 2. Non-negotiable doctrine

```txt
1. S&P 500 only.
2. US-first paid SaaS for swing traders.
3. Research dashboard, not broker.
4. Not personalized financial advice.
5. No order execution.
6. No copy-trading.
7. No Finviz clone.
8. No unlicensed production scraping.
9. Every market number has source/timestamp/license_status.
10. Every AI market claim is tool-grounded.
11. Every idea has reason/risk/invalidation/evidence/confidence.
12. Every backtest shows assumptions, costs, slippage, survivorship warning.
13. Mock-first development, provider abstraction from day one.
14. Risk-mode data is allowed only as explicitly labelled non-production/review-gated mode.
15. Public paid launch requires legal review: Terms, Privacy, Disclaimers, trademark/index wording, data rights, adviser-risk.
```

---

## 3. Final MVP modules

MVP keeps all 9 modules:

```txt
1. Home
2. S&P 500 Heatmap
3. Market Radar
4. Long / Short Ideas
5. Sector Rotation
6. Ticker Research
7. AI Analyst
8. Strategy Lab Lite
9. Watchlist Lite
```

Plus SaaS/public shell:

```txt
10. Public Marketing Home
11. Pricing
12. Legal pages
13. Auth / onboarding
14. Billing / subscription gate
15. Admin / data health
```

---

## 4. Product positioning

### 4.1. English positioning

```txt
SnP Stock Lab is an AI-powered research and strategy lab for S&P 500 swing traders.
It combines market maps, sector rotation, movement radar, explainable long/short candidates, ticker research, strategy testing, and a data-grounded AI analyst in one focused dashboard.
```

### 4.2. Russian internal formulation

```txt
SnP Stock Lab — AI-лаборатория для анализа, отбора и тестирования торговых идей внутри S&P 500.
```

### 4.3. Public claims allowed

Allowed:

```txt
- Research clarity for S&P 500 traders.
- Find active movers faster.
- Understand sector rotation.
- Compare long/short candidates.
- Turn watchlists into living research boards.
- Test hypotheses on historical data.
- AI explanations grounded in platform data.
```

Avoid:

```txt
- Guaranteed profits.
- Beat the market.
- AI tells you what to buy.
- Fully automated hedge fund.
- Replace your financial advisor.
- Real-time institutional-grade data unless licensed.
- Official S&P product unless licensed/approved.
```

---

## 5. Target user

### 5.1. Primary user

```txt
US equity swing trader
- trades liquid large-cap names;
- focuses on S&P 500, sectors, momentum, relative strength;
- holds positions from days to weeks;
- uses Finviz/TradingView/Yahoo/SeekingAlpha manually;
- wants less noise and faster decision context;
- values screeners, maps, top movers, sector strength, catalyst links.
```

### 5.2. Secondary user

```txt
Active investor / market researcher
- not day-trading every minute;
- wants market overview and ticker research;
- uses watchlist;
- cares about valuation/fundamentals/catalysts;
- may use Strategy Lab as sanity check.
```

### 5.3. Excluded users for MVP

```txt
- crypto traders;
- penny stock traders;
- commodities/futures/forex users;
- options-flow-only traders;
- broker execution users;
- fully automated trading users;
- personal portfolio advice seekers.
```

---

## 6. Product surface

## 6.1. Public marketing site

### Pages

```txt
/
/pricing
/legal/terms
/legal/privacy
/legal/disclaimers
/login
/signup
```

### Public Home sections

```txt
Hero:
- AI research lab for S&P 500 swing traders.
- CTA: Start Trial.
- Secondary CTA: View Demo.

Problem:
- Market data is scattered.
- Screeners show signals but not context.
- AI chats hallucinate without fresh platform data.

Solution:
- Heatmap + Radar + Sector Rotation + Long/Short candidates + AI explanations.

Feature blocks:
- 30-second Home dashboard.
- S&P 500 Heatmap.
- Market Radar.
- Sector Rotation Velocity.
- Explainable Long/Short Ideas.
- Ticker Research.
- Strategy Lab Lite.
- Watchlist "What changed?".
- AI Analyst with evidence.

Trust blocks:
- S&P 500-only focus.
- Source/timestamp labels.
- Research-only, no brokerage.
- Delayed/EOD data disclosure.

Pricing teaser:
- Pro plan.
- Free trial.
- AI credits.

Legal footer:
- Not financial advice.
- Data may be delayed.
- Market data source labels.
```

### Pricing v0

Pricing is placeholder until AI/data cost review.

```txt
Pro Trial:
- 7 or 14 days.
- Limited AI credits.
- Full dashboard.

Pro:
- Per-seat monthly.
- Home, Heatmap, Radar, Ideas, Sector Rotation, Ticker Research.
- AI Analyst credits.
- Strategy Lab Lite.
- Watchlist.

Future Team:
- Shared watchlists.
- Shared presets.
- More AI credits.
- Admin controls.
```

---

## 7. SaaS foundation

### 7.1. Auth

Use Clerk with organizations from day one.

Roles:

```txt
Admin:
- manage members;
- manage billing;
- manage org settings;
- manage saved presets;
- view usage.

Member:
- use dashboard;
- save watchlist items;
- run strategy lab within entitlement;
- use AI credits.
```

### 7.2. Billing

Use Stripe Billing.

Core rules:

```txt
- subscription status is source of truth for paid access;
- webhook events are idempotent;
- verify Stripe signatures;
- sync subscription state to database;
- never trust client-side billing state;
- entitlement checks happen server-side;
- paywall AI/Strategy/Watchlist in MVP;
- dashboard preview can be accessible during trial.
```

### 7.3. Entitlements

```txt
free_trial:
  heatmap: true
  radar: true
  ideas: limited
  ticker_research: limited
  ai_credits_monthly: low
  strategy_lab_runs_monthly: low
  watchlist_items: limited

pro:
  heatmap: true
  radar: true
  ideas: true
  ticker_research: true
  ai_credits_monthly: configurable
  strategy_lab_runs_monthly: configurable
  watchlist_items: configurable

admin_internal:
  admin_panel: true
  provider_controls: true
  data_health: true
  feature_flags: true
```

---

## 8. Data policy

### 8.1. Data modes

```txt
mock_mode:
- deterministic seeded fake/fixture data;
- used for UI, tests, demos;
- no real market claims.

risk_mode:
- free/gray/dev data sources allowed behind abstraction;
- always labelled;
- kill-switchable;
- not trusted for public paid launch without legal review.

licensed_mode:
- paid/reviewed provider;
- allowed for production;
- clear rights for SaaS display/redistribution.
```

### 8.2. Mandatory metadata on market outputs

Every market-facing API response must include:

```json
{
  "as_of": "2026-04-27T20:00:00Z",
  "source": "provider_name",
  "source_updated_at": "2026-04-27T20:00:00Z",
  "license_status": "mock|risk_mode|licensed|unknown",
  "risk_mode": false,
  "is_stale": false,
  "confidence": 0.78,
  "warnings": []
}
```

### 8.3. Data freshness levels

```txt
fresh:
- within expected freshness window.

delayed:
- provider delayed but still valid for stated use.

stale:
- older than allowed window.

partial:
- some tickers missing.

degraded:
- provider issue or fallback mode.

mock:
- development/demo data only.
```

### 8.4. Kill-switches

Admin must be able to disable:

```txt
- individual provider;
- all risk-mode providers;
- news links;
- AI Analyst;
- Strategy Lab;
- score publishing;
- marketing demo data;
- watchlist notifications;
- specific source by provider/source_id.
```

---

## 9. Recommended stack

### 9.1. Frontend

```txt
Next.js
TypeScript
Tailwind
shadcn/ui
TanStack Query
TanStack Table
Zustand optional
ECharts for charts
D3 treemap for heatmap
Playwright for smoke tests
```

### 9.2. Backend

```txt
FastAPI
Pydantic
SQLAlchemy
Alembic
PostgreSQL
Redis
Celery
uv
pytest
httpx
ruff
mypy optional
```

### 9.3. AI

```txt
OpenAI-compatible adapter
server-side tool allowlist
strict structured outputs
cost tracking
credit metering
90-day audit log
```

### 9.4. SaaS / Infra

```txt
Clerk auth + orgs
Stripe subscriptions + webhooks
Vercel web
Render API/workers/Postgres/Redis
GitHub Actions CI
Sentry
structured logs
```

---

## 10. Repository structure

```txt
snp-stock-lab/
  AGENTS.md
  DESIGN.md
  README.md
  package.json
  pnpm-workspace.yaml
  turbo.json
  .env.example

  .github/
    workflows/
      ci.yml
      deploy-preview.yml

  .codex/
    config.toml
    agents/
      product-architect.toml
      system-architect.toml
      data-engineer.toml
      quant-scoring-engineer.toml
      backend-api-engineer.toml
      frontend-dashboard-engineer.toml
      ai-analyst-engineer.toml
      strategy-lab-engineer.toml
      saas-billing-engineer.toml
      devops-observability-engineer.toml
      qa-security-reviewer.toml
      compliance-risk-reviewer.toml
      marketing-growth-writer.toml

  .agents/
    skills/
      financial-data-contracts/
        SKILL.md
      scoring-backtest-review/
        SKILL.md
      dashboard-ui-review/
        SKILL.md
      ai-analyst-evidence-review/
        SKILL.md
      compliance-review/
        SKILL.md
      saas-billing-review/
        SKILL.md
      launch-readiness-review/
        SKILL.md

  docs/
    PRD.md
    ARCHITECTURE.md
    DATA_PROVIDER_MATRIX.md
    COMPLIANCE.md
    RELEASE_PLAN.md
    REFERENCE_ANCHORS.md
    MARKETING_COPY_GUARDRAILS.md
    SAAS_BILLING.md
    AUTH_ORGS.md
    API_CONTRACTS.md
    DATA_CONTRACTS.md
    SCORING_MODEL.md
    SECTOR_ROTATION.md
    HEATMAP_SPEC.md
    MARKET_RADAR_SPEC.md
    LONG_SHORT_IDEAS_SPEC.md
    TICKER_RESEARCH_SPEC.md
    AI_ANALYST_SPEC.md
    STRATEGY_LAB_SPEC.md
    WATCHLIST_SPEC.md
    ADMIN_DATA_HEALTH_SPEC.md
    TESTING_STRATEGY.md
    OBSERVABILITY.md
    SECURITY.md
    BETA_PLAN.md

  apps/
    web/
      app/
        (public)/
          page.tsx
          pricing/page.tsx
          legal/terms/page.tsx
          legal/privacy/page.tsx
          legal/disclaimers/page.tsx
        (app)/
          dashboard/page.tsx
          heatmap/page.tsx
          radar/page.tsx
          ideas/page.tsx
          sectors/page.tsx
          tickers/[symbol]/page.tsx
          ai/page.tsx
          strategy-lab/page.tsx
          watchlist/page.tsx
          admin/page.tsx
      components/
      features/
      lib/
      styles/
      tests/

  services/
    api/
      app/
        main.py
        config.py
        dependencies.py
        routers/
          health.py
          data_health.py
          constituents.py
          market.py
          heatmap.py
          radar.py
          sectors.py
          ideas.py
          tickers.py
          ai.py
          strategy.py
          watchlists.py
          admin.py
          billing.py
        schemas/
        services/
        repositories/
        providers/
        auth/
        billing/
        ai/
        scoring/
        backtesting/
      tests/
      pyproject.toml

    workers/
      ingestion/
      features/
      scoring/
      ai/
      backtesting/
      notifications/
      scheduler/

  packages/
    shared-types/
    ui/
    charts/
    financial-math/

  db/
    migrations/
    seeds/
    schema.sql

  scripts/
    seed_mock_data.py
    seed_sp500.py
    run_ingestion.py
    compute_features.py
    compute_scores.py
    run_backtest.py
    validate_data_health.py
    smoke_ai_tools.py

  infra/
    docker/
      docker-compose.yml
    render/
      render.yaml
    vercel/
    nginx/

  notebooks/
    scoring_research/
    backtest_research/
```

---

## 11. Root AGENTS.md

Create `AGENTS.md`:

```md
# AGENTS.md

## Project

SnP Stock Lab is a US-first paid SaaS research dashboard for S&P 500 swing traders.

It is not a brokerage, not a personalized financial advisor, not an execution platform, and not a Finviz clone.

## Core product

MVP modules:
- Home
- S&P 500 Heatmap
- Market Radar
- Long / Short Ideas
- Sector Rotation
- Ticker Research
- AI Analyst
- Strategy Lab Lite
- Watchlist Lite

SaaS modules:
- public marketing home
- pricing
- legal pages
- Clerk auth and organizations
- Stripe billing
- Admin/Data Health

## Non-goals

Do not build:
- crypto
- penny stocks
- commodities
- futures/forex
- broker execution
- copy trading
- personalized financial advice
- all-market Bloomberg clone
- unlicensed Finviz scraping
- Finviz pixel clone
- black-box recommendations

## Product rules

1. Restrict market universe to S&P 500.
2. Treat SnP Stock Lab as a working title until trademark/legal review.
3. Every market-facing output must include source, source_updated_at, license_status, risk_mode, confidence, and stale-data warnings.
4. Every AI-generated idea must include reason, risk, invalidation, confidence, and evidence.
5. Use language like "candidate", "idea", "setup", "research", "invalidation".
6. Do not use "guaranteed", "sure profit", "buy now", "sell now", or personalized advice language.
7. Watchlist context is allowed only as evidence-grounded change explanation.
8. Public paid launch requires legal review of Terms, Privacy, Disclaimers, data rights, index/trademark wording, and adviser-risk.

## Architecture

Frontend:
- Next.js
- TypeScript
- Tailwind
- shadcn/ui
- ECharts
- D3 treemap
- TanStack Query/Table

Backend:
- FastAPI
- Pydantic
- SQLAlchemy
- Alembic
- PostgreSQL
- Redis
- Celery
- pytest

SaaS:
- Clerk auth/orgs
- Stripe subscriptions/webhooks
- org-scoped data
- AI credits
- feature entitlements

AI:
- OpenAI-compatible adapter
- server-side tool allowlist
- strict structured outputs
- 90-day AI audit log
- credit usage tracking

## Data rules

- Use mock-first development.
- MarketDataProvider abstraction is mandatory.
- Risk-mode/free/gray sources must be labelled and kill-switchable.
- Do not scrape Finviz for production.
- yfinance may only be used in explicitly labelled dev/risk-mode unless legal approves otherwise.
- SEC EDGAR must use server-side ingestion, declared User-Agent, caching, and fair-access limits.
- Every data row must store source and source_updated_at.
- Do not silently mix adjusted and raw OHLCV.
- Reject or mark out-of-scope non-S&P tickers.

## AI Analyst rules

AI Analyst must:
- call platform tools for current market claims;
- include data used, timestamps, confidence, and evidence refs;
- avoid invented prices, invented catalysts, and invented news;
- separate data from interpretation;
- refuse unsupported market claims;
- show missing/stale data clearly;
- avoid personalized financial advice.

## Testing requirements

For every change:
- frontend changes: run lint and relevant UI tests;
- backend changes: run pytest;
- scoring changes: run formula/no-lookahead tests;
- billing changes: test webhook idempotency and signature handling;
- AI changes: run hallucination/evidence tests;
- data changes: run data health tests;
- compliance-sensitive changes: run compliance review.

## Definition of Done

A wave is done only if:
- tests relevant to the wave pass;
- docs are updated;
- data/compliance/security risks are reviewed;
- no secrets are committed;
- market outputs are timestamped;
- AI outputs are evidence-grounded;
- next-wave prompt is written.
```

---

## 12. DESIGN.md

Create `DESIGN.md`:

```md
---
version: "alpha"
name: "SnP Stock Lab"
description: "Dense professional research dashboard for S&P 500 swing traders."

colors:
  background: "#070A0F"
  surface: "#0D1117"
  surface-raised: "#111827"
  surface-subtle: "#151B23"
  border: "#263241"
  border-muted: "#1D2632"

  text-primary: "#F2F5F8"
  text-secondary: "#A9B4C0"
  text-muted: "#6F7A86"

  accent: "#7DD3FC"
  accent-strong: "#38BDF8"
  accent-muted: "#164E63"

  positive: "#22C55E"
  positive-muted: "#14532D"
  negative: "#EF4444"
  negative-muted: "#7F1D1D"
  warning: "#F59E0B"
  warning-muted: "#78350F"
  neutral: "#94A3B8"

  light-background: "#F8FAFC"
  light-surface: "#FFFFFF"
  light-surface-raised: "#F1F5F9"
  light-border: "#CBD5E1"
  light-text-primary: "#0F172A"
  light-text-secondary: "#334155"

font:
  sans: "Inter"
  mono: "JetBrains Mono"

typography:
  h1:
    fontFamily: "Inter"
    fontSize: "2.25rem"
    fontWeight: 700
    lineHeight: "2.75rem"
    letterSpacing: "-0.04em"
  h2:
    fontFamily: "Inter"
    fontSize: "1.5rem"
    fontWeight: 650
    lineHeight: "2rem"
    letterSpacing: "-0.03em"
  h3:
    fontFamily: "Inter"
    fontSize: "1.125rem"
    fontWeight: 650
    lineHeight: "1.5rem"
  body:
    fontFamily: "Inter"
    fontSize: "0.9375rem"
    fontWeight: 400
    lineHeight: "1.5rem"
  label:
    fontFamily: "Inter"
    fontSize: "0.75rem"
    fontWeight: 600
    lineHeight: "1rem"
    letterSpacing: "0.04em"
  mono:
    fontFamily: "JetBrains Mono"
    fontSize: "0.8125rem"
    fontWeight: 500
    lineHeight: "1.25rem"

rounded:
  xs: "4px"
  sm: "6px"
  md: "10px"
  lg: "14px"
  xl: "20px"

spacing:
  xs: "4px"
  sm: "8px"
  md: "16px"
  lg: "24px"
  xl: "32px"
  xxl: "48px"

components:
  app-shell:
    backgroundColor: "{colors.background}"
    textColor: "{colors.text-primary}"
  card:
    backgroundColor: "{colors.surface}"
    textColor: "{colors.text-primary}"
    borderColor: "{colors.border-muted}"
    rounded: "{rounded.lg}"
    padding: "16px"
  card-raised:
    backgroundColor: "{colors.surface-raised}"
    textColor: "{colors.text-primary}"
    borderColor: "{colors.border}"
    rounded: "{rounded.lg}"
    padding: "20px"
  button-primary:
    backgroundColor: "{colors.accent}"
    textColor: "#03121A"
    rounded: "{rounded.md}"
    padding: "10px 14px"
  badge-positive:
    backgroundColor: "{colors.positive-muted}"
    textColor: "{colors.positive}"
    rounded: "{rounded.sm}"
    padding: "4px 8px"
  badge-negative:
    backgroundColor: "{colors.negative-muted}"
    textColor: "{colors.negative}"
    rounded: "{rounded.sm}"
    padding: "4px 8px"
  badge-warning:
    backgroundColor: "{colors.warning-muted}"
    textColor: "{colors.warning}"
    rounded: "{rounded.sm}"
    padding: "4px 8px"
---

## Visual identity

SnP Stock Lab should feel like a professional research terminal for S&P 500 swing traders.

It should be:
- dense;
- calm;
- analytical;
- fast;
- readable;
- evidence-first;
- not playful;
- not a copy of Finviz, Bloomberg, TradingView, Yahoo Finance, or any other single product.

## Layout principles

- Desktop-first responsive.
- Dark mode default, light mode supported.
- Data cards should be compact.
- Tables should be sortable and filterable.
- Every market card must include data freshness.
- Every AI card must include confidence and evidence affordance.
- No marketing hero bloat inside the app dashboard.

## Market color semantics

- Green: positive return/strength.
- Red: negative return/weakness.
- Amber: risk/stale/degraded warning.
- Blue/cyan: AI insight, selected state, primary action.
- Gray: neutral/missing/unknown.

## Core components

### Idea Card
Must show:
- ticker;
- direction;
- horizon;
- score;
- confidence;
- reason;
- risk;
- invalidation;
- evidence chips;
- source/timestamp.

### Market Card
Must show:
- metric name;
- value;
- change;
- source;
- timestamp;
- freshness state.

### Heatmap Tile
Hover must show:
- ticker;
- company;
- sector;
- industry;
- selected-period return;
- market cap or index weight;
- relative volume;
- long score;
- short score;
- top reason.

### AI Analyst Answer
Must show:
- answer;
- data used;
- reasoning;
- risks;
- confidence;
- timestamp;
- disclaimer.
```

---

## 13. Codex config

Create `.codex/config.toml`:

```toml
[agents]
max_threads = 6
max_depth = 1
job_max_runtime_seconds = 1800

[projects]
# Keep project-specific settings in repo-level config when Codex supports it.

[model]
# Parent/orchestrator should use gpt-5.5 high when available.
```

Execution principle:

```txt
Parallelize read-heavy research/review.
Serialize write-heavy implementation.
Never let two agents edit same file group in parallel.
```

---

## 14. Custom Codex agents

### 14.1. product-architect

`.codex/agents/product-architect.toml`

```toml
name = "product_architect"
description = "Owns product scope, MVP decisions, user journeys, PRD, launch criteria, and feature prioritization for SnP Stock Lab."
model = "gpt-5.5"
model_reasoning_effort = "high"
sandbox_mode = "read-only"

developer_instructions = """
You are the product architect for SnP Stock Lab.

Product:
- US-first paid SaaS research dashboard for S&P 500 swing traders.
- Not brokerage, not personalized advice, not Finviz clone.
- MVP keeps all 9 modules: Home, Heatmap, Market Radar, Long/Short Ideas, Sector Rotation, Ticker Research, AI Analyst, Strategy Lab Lite, Watchlist Lite.

Hard constraints:
- S&P 500 only.
- English product/repo docs.
- Source/timestamp/license_status on every market output.
- AI outputs must include evidence/confidence.
- Public paid launch requires legal/data/trademark/adviser-risk review.

Return:
1. Product decisions
2. Scope risks
3. User journey updates
4. Files to update
5. Acceptance criteria
"""
```

### 14.2. system-architect

`.codex/agents/system-architect.toml`

```toml
name = "system_architect"
description = "Owns service boundaries, architecture, data flow, repo layout, integration constraints, and scalability decisions."
model = "gpt-5.5"
model_reasoning_effort = "high"
sandbox_mode = "read-only"

developer_instructions = """
You own technical architecture.

Design for:
- Next.js frontend.
- FastAPI backend.
- PostgreSQL database.
- Redis + Celery workers.
- Clerk org auth.
- Stripe subscriptions.
- OpenAI-compatible LLM adapter.
- Provider abstraction for all market data.

Rules:
- No unnecessary microservices.
- No broker execution endpoints.
- S&P 500-only constraints at API boundary.
- Every market response has data freshness metadata.
- AI Analyst must be tool-grounded.

Return:
1. Architecture decision
2. Tradeoffs
3. Integration risks
4. File changes
5. Tests required
"""
```

### 14.3. data-engineer

`.codex/agents/data-engineer.toml`

```toml
name = "data_engineer"
description = "Owns financial data schemas, provider adapters, ingestion, data health, freshness, and S&P 500 membership validation."
model = "gpt-5.5"
model_reasoning_effort = "high"
sandbox_mode = "workspace-write"

developer_instructions = """
You own financial data pipelines.

Hard rules:
- S&P 500 universe only.
- Every row has source and source_updated_at.
- Every provider has license_status and risk_mode metadata.
- Risk-mode/free sources must be kill-switchable.
- Never scrape Finviz for production.
- yfinance only in labelled dev/risk-mode unless legal approves otherwise.
- SEC ingestion must declare User-Agent, respect fair access, and cache.

Implement:
- DB migrations.
- provider interfaces.
- mock provider.
- ingestion jobs.
- data health checks.
- stale-data warnings.

Return:
1. Implemented changes
2. Data contracts changed
3. Edge cases
4. Tests run
5. Risks
"""
```

### 14.4. quant-scoring-engineer

`.codex/agents/quant-scoring-engineer.toml`

```toml
name = "quant_scoring_engineer"
description = "Owns indicators, sector rotation, long/short scoring, confidence, risk penalties, invalidation logic, and formula tests."
model = "gpt-5.5"
model_reasoning_effort = "high"
sandbox_mode = "workspace-write"

developer_instructions = """
You own quant logic.

Rules:
- No lookahead bias.
- Warn about survivorship bias.
- Deterministic formulas only for MVP.
- Every score exposes factor breakdown.
- Every idea includes risk and invalidation.
- No personalized financial advice language.

Implement:
- SMA, RSI, ATR, volatility, gaps, abnormal volume, breakouts, relative strength.
- Sector strength, velocity, acceleration, state classification.
- Long score, short score, confidence score.
- Risk penalties and invalidation.

Return:
1. Formula
2. Implementation summary
3. Tests
4. Limitations
5. Next iteration
"""
```

### 14.5. backend-api-engineer

`.codex/agents/backend-api-engineer.toml`

```toml
name = "backend_api_engineer"
description = "Owns FastAPI routes, schemas, repositories, service layer, auth boundaries, entitlements, and API tests."
model = "gpt-5.5"
model_reasoning_effort = "high"
sandbox_mode = "workspace-write"

developer_instructions = """
You own backend API.

Rules:
- Keep routers thin.
- Use Pydantic schemas.
- Use service/repository separation.
- Validate org access.
- Validate S&P 500 ticker scope.
- Include source/timestamp/license_status/risk_mode in market responses.
- Server-side entitlement checks.
- No broker execution endpoints.

Core endpoints:
- /health
- /data-health
- /constituents
- /market/overview
- /heatmap
- /radar
- /sectors/rotation
- /ideas/top
- /tickers/{symbol}
- /ai/sessions
- /ai/messages
- /strategy/runs
- /watchlists
- /admin/*
- /billing/webhooks/stripe

Return:
1. Endpoints changed
2. Schemas changed
3. Tests run
4. Failure modes
"""
```

### 14.6. frontend-dashboard-engineer

`.codex/agents/frontend-dashboard-engineer.toml`

```toml
name = "frontend_dashboard_engineer"
description = "Owns app dashboard UI, heatmap, radar, ticker research, strategy lab, watchlist, and DESIGN.md adherence."
model = "gpt-5.5"
model_reasoning_effort = "high"
sandbox_mode = "workspace-write"

developer_instructions = """
You own frontend implementation.

Rules:
- Follow DESIGN.md.
- Dense professional UI.
- Dark/light toggle.
- Desktop-first responsive.
- Every market card shows timestamp/source/freshness.
- Every score has explanation affordance.
- No Finviz pixel-copy.
- Use reusable components.

Return:
1. Components added
2. UX decisions
3. Accessibility notes
4. Tests/screenshots if available
5. Follow-up improvements
"""
```

### 14.7. ai-analyst-engineer

`.codex/agents/ai-analyst-engineer.toml`

```toml
name = "ai_analyst_engineer"
description = "Owns tool-grounded AI Analyst, strict structured outputs, evidence refs, audit logs, and AI credit metering."
model = "gpt-5.5"
model_reasoning_effort = "high"
sandbox_mode = "workspace-write"

developer_instructions = """
You own AI Analyst.

Hard rules:
- AI must answer current market questions using platform tools.
- No tool data, no market claim.
- Include data used, timestamp, confidence, evidence refs.
- Do not invent news/catalysts/prices.
- Separate data from interpretation.
- Avoid personalized financial advice.
- Track AI credits.
- Store 90-day AI audit log.

Implement:
- LLMProvider adapter.
- tool registry.
- strict structured outputs.
- evidence references.
- chat sessions.
- usage metering.
- hallucination tests.

Return:
1. AI flow
2. Tool schemas
3. Guardrails
4. Tests
5. Risks
"""
```

### 14.8. strategy-lab-engineer

`.codex/agents/strategy-lab-engineer.toml`

```toml
name = "strategy_lab_engineer"
description = "Owns Strategy Lab Lite, simple daily-bar backtests, strategy schemas, metrics, and bias controls."
model = "gpt-5.5"
model_reasoning_effort = "high"
sandbox_mode = "workspace-write"

developer_instructions = """
You own Strategy Lab Lite.

Rules:
- Daily bars only in MVP.
- No overfit optimizer.
- No lookahead.
- Show transaction costs and slippage.
- Show survivorship-bias warning.
- Save org-scoped runs.
- Hypothetical results disclaimer.

MVP strategies:
- momentum;
- mean reversion;
- relative strength pullback;
- sector rotation;
- breakout continuation.

Return:
1. Strategy implemented
2. Parameter schema
3. Metrics
4. Bias controls
5. Tests
"""
```

### 14.9. saas-billing-engineer

`.codex/agents/saas-billing-engineer.toml`

```toml
name = "saas_billing_engineer"
description = "Owns Clerk organizations, roles, Stripe billing, webhooks, entitlements, AI credits, and org-scoped access."
model = "gpt-5.5"
model_reasoning_effort = "high"
sandbox_mode = "workspace-write"

developer_instructions = """
You own SaaS foundation.

Implement:
- Clerk auth integration.
- organization mapping.
- Admin/Member roles.
- Stripe subscription webhook skeleton.
- entitlement checks.
- AI credit tracking.
- org-scoped data access.

Rules:
- Verify Stripe webhook signatures.
- Make webhooks idempotent.
- Do not trust client entitlement state.
- Tests for org isolation.
- No secrets in repo.

Return:
1. SaaS changes
2. Auth/billing contracts
3. Tests
4. Security concerns
5. Remaining blockers
"""
```

### 14.10. qa-security-reviewer

`.codex/agents/qa-security-reviewer.toml`

```toml
name = "qa_security_reviewer"
description = "Read-only QA/security reviewer for correctness, tests, auth, secrets, AI hallucination, data risks, and regression blockers."
model = "gpt-5.5"
model_reasoning_effort = "high"
sandbox_mode = "read-only"

developer_instructions = """
You are a strict QA and security reviewer.

Review:
- missing tests;
- stale data paths;
- provider leakage;
- SQL injection;
- auth/org isolation;
- Stripe webhook security;
- secrets;
- AI hallucination paths;
- lookahead bias;
- misleading finance language;
- Finviz scraping risk.

Do not edit files unless explicitly asked.
Return concrete findings with file references and reproduction steps.
"""
```

### 14.11. compliance-risk-reviewer

`.codex/agents/compliance-risk-reviewer.toml`

```toml
name = "compliance_risk_reviewer"
description = "Read-only compliance reviewer for market data licensing, index/trademark wording, financial advice risk, news redistribution, and launch blockers."
model = "gpt-5.5"
model_reasoning_effort = "high"
sandbox_mode = "read-only"

developer_instructions = """
You review compliance and financial risk.

Focus:
- no broker execution;
- no personalized advice;
- no guaranteed returns;
- market data licensing;
- news redistribution;
- S&P/index/trademark wording;
- SEC fair-access;
- yfinance/risk-mode risks;
- backtest disclaimers;
- public marketing copy.

Return:
1. Compliance findings
2. Required copy changes
3. Data licensing risks
4. Feature restrictions
5. Launch blockers
"""
```

### 14.12. devops-observability-engineer

`.codex/agents/devops-observability-engineer.toml`

```toml
name = "devops_observability_engineer"
description = "Owns Docker, CI, Render/Vercel deploy paths, logs, metrics, Sentry, worker scheduling, and health checks."
model = "gpt-5.5"
model_reasoning_effort = "high"
sandbox_mode = "workspace-write"

developer_instructions = """
You own production readiness.

Implement:
- Docker Compose.
- GitHub Actions CI.
- Render API/worker config.
- Vercel web config notes.
- health checks.
- structured logs.
- Sentry hooks.
- data freshness metrics.
- worker scheduling docs.

Rules:
- No secrets in repo.
- Fail loudly on missing required env vars.
- Local dev must be easy.
- Prefer boring infrastructure.

Return:
1. Infra changes
2. Commands
3. Health checks
4. Failure handling
5. Remaining risks
"""
```

### 14.13. marketing-growth-writer

`.codex/agents/marketing-growth-writer.toml`

```toml
name = "marketing_growth_writer"
description = "Owns public marketing copy, pricing page copy, onboarding copy, disclaimers integration, and product messaging."
model = "gpt-5.5"
model_reasoning_effort = "high"
sandbox_mode = "workspace-write"

developer_instructions = """
You write product and marketing copy.

Rules:
- English only for public app copy.
- Research clarity positioning.
- No financial advice claims.
- No guaranteed returns.
- No official S&P partnership claims.
- No real-time/institutional-grade data claim unless licensed.
- Add delayed/EOD and research-only caveats where relevant.

Return:
1. Copy created/changed
2. Risky phrases removed
3. Disclaimers used
4. Follow-up copy tasks
"""
```

---

## 15. Codex Skills

### 15.1. financial-data-contracts

`.agents/skills/financial-data-contracts/SKILL.md`

```md
---
name: financial-data-contracts
description: Use when designing/changing financial data schemas, provider adapters, ingestion jobs, data freshness, S&P 500 validation, or market-data API contracts.
---

# Financial Data Contracts Skill

Check every market data path for:
- source
- source_updated_at
- license_status
- risk_mode
- freshness status
- S&P 500 scope validation
- adjusted/raw OHLCV distinction
- provider kill-switch
- stale/partial/degraded warnings
- mock/risk/licensed mode separation

Require:
- mock provider tests
- data health records
- missing data behavior
- provider error handling
- no Finviz production scraping
- yfinance only in labelled dev/risk-mode unless approved

Return:
1. Schema risks
2. Provider risks
3. Required tests
4. Launch blockers
```

### 15.2. scoring-backtest-review

`.agents/skills/scoring-backtest-review/SKILL.md`

```md
---
name: scoring-backtest-review
description: Use when implementing indicators, sector rotation, long/short scoring, backtests, metrics, or invalidation logic.
---

# Scoring and Backtest Review Skill

Review for:
- lookahead bias
- survivorship bias
- overfitting
- missing costs/slippage
- unstable formula weights
- missing factor breakdown
- missing confidence
- missing risk/invalidation
- misleading finance language

Every score must expose:
- factor values
- normalized factor scores
- positive contributors
- negative contributors
- timestamp
- source
- confidence

Every backtest must expose:
- params
- universe
- date range
- benchmark
- costs
- slippage
- assumptions
- warnings
- metrics
- trade list
```

### 15.3. ai-analyst-evidence-review

`.agents/skills/ai-analyst-evidence-review/SKILL.md`

```md
---
name: ai-analyst-evidence-review
description: Use for AI Analyst prompts, tool calls, structured outputs, evidence refs, hallucination tests, and AI-generated idea explanations.
---

# AI Analyst Evidence Review Skill

AI Analyst must:
- use platform tools for current market claims;
- include data used;
- include timestamps;
- include confidence;
- include evidence refs;
- not invent news/catalysts/prices;
- not answer outside S&P 500 scope as if supported;
- separate data from interpretation;
- avoid personalized advice;
- decrement AI credits when appropriate;
- write audit log entries.

Test with:
- stale data;
- missing data;
- non-S&P ticker;
- request for guaranteed profit;
- catalyst unavailable;
- contradictory signals;
- user asking for personalized advice.
```

### 15.4. dashboard-ui-review

`.agents/skills/dashboard-ui-review/SKILL.md`

```md
---
name: dashboard-ui-review
description: Use when building Home, Heatmap, Radar, Sector Rotation, Ticker Research, Watchlist, Strategy Lab, Admin, or marketing dashboard UI.
---

# Dashboard UI Review Skill

Check:
- follows DESIGN.md;
- dense but readable;
- no Finviz pixel-copy;
- source/timestamp/freshness visible;
- score explanation affordance;
- loading states;
- empty states;
- stale/degraded states;
- paywall states;
- responsive behavior;
- accessibility contrast;
- ticker click-through;
- disclaimer placement.

Return:
- UI issues;
- missing states;
- design drift;
- recommended fixes.
```

### 15.5. compliance-review

`.agents/skills/compliance-review/SKILL.md`

```md
---
name: compliance-review
description: Use when adding financial claims, AI recommendations, market data sources, news links, paid plans, legal pages, or broker/execution-like features.
---

# Compliance Review Skill

Review:
- market data license risk;
- news redistribution risk;
- index/trademark wording;
- financial advice language;
- no execution boundary;
- backtest disclaimers;
- AI disclaimers;
- watchlist personalization risk;
- audit logging;
- public marketing copy.

MVP must not include:
- order execution;
- copy trading;
- guaranteed return claims;
- unauthorized data scraping;
- personalized portfolio advice.

Public paid launch requires attorney/data rights review.
```

### 15.6. saas-billing-review

`.agents/skills/saas-billing-review/SKILL.md`

```md
---
name: saas-billing-review
description: Use when implementing Clerk orgs, roles, subscriptions, Stripe webhooks, AI credits, feature gates, or org-scoped data.
---

# SaaS Billing Review Skill

Check:
- org isolation;
- Admin/Member permissions;
- server-side entitlement checks;
- Stripe signature verification;
- webhook idempotency;
- subscription status sync;
- failed payment/cancel handling;
- AI credit decrement;
- usage audit trail;
- no client-trusted access control;
- no secrets in repo.

Required tests:
- member cannot access another org;
- unpaid org blocked from paid feature;
- webhook replay safe;
- AI credits cannot go negative unless explicit overage mode.
```

### 15.7. launch-readiness-review

`.agents/skills/launch-readiness-review/SKILL.md`

```md
---
name: launch-readiness-review
description: Use before beta/public launch to review product readiness, data readiness, legal blockers, observability, and user-facing risk.
---

# Launch Readiness Review Skill

Check:
- critical user journeys;
- CI green;
- data health dashboard;
- provider status;
- legal pages;
- disclaimers;
- pricing/trial flow;
- payment webhook stability;
- AI hallucination tests;
- backtest warnings;
- Sentry/logs;
- admin kill-switches;
- beta feedback loop.

Return:
- launch blockers;
- acceptable risks;
- required fixes;
- beta checklist.
```

---

## 16. Core database model

### 16.1. SaaS tables

```sql
CREATE TABLE orgs (
  id UUID PRIMARY KEY,
  clerk_org_id TEXT UNIQUE NOT NULL,
  name TEXT NOT NULL,
  slug TEXT,
  created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  updated_at TIMESTAMPTZ NOT NULL DEFAULT now()
);

CREATE TABLE users (
  id UUID PRIMARY KEY,
  clerk_user_id TEXT UNIQUE NOT NULL,
  email TEXT NOT NULL,
  name TEXT,
  created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  updated_at TIMESTAMPTZ NOT NULL DEFAULT now()
);

CREATE TABLE memberships (
  id UUID PRIMARY KEY,
  org_id UUID NOT NULL REFERENCES orgs(id),
  user_id UUID NOT NULL REFERENCES users(id),
  role TEXT NOT NULL CHECK (role IN ('admin', 'member')),
  created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  UNIQUE(org_id, user_id)
);

CREATE TABLE subscriptions (
  id UUID PRIMARY KEY,
  org_id UUID NOT NULL REFERENCES orgs(id),
  stripe_customer_id TEXT,
  stripe_subscription_id TEXT UNIQUE,
  status TEXT NOT NULL,
  plan_key TEXT NOT NULL,
  current_period_start TIMESTAMPTZ,
  current_period_end TIMESTAMPTZ,
  cancel_at_period_end BOOLEAN DEFAULT false,
  created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  updated_at TIMESTAMPTZ NOT NULL DEFAULT now()
);

CREATE TABLE entitlements (
  id UUID PRIMARY KEY,
  org_id UUID NOT NULL REFERENCES orgs(id),
  feature_key TEXT NOT NULL,
  is_enabled BOOLEAN NOT NULL DEFAULT false,
  limit_value INTEGER,
  used_value INTEGER NOT NULL DEFAULT 0,
  resets_at TIMESTAMPTZ,
  created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  updated_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  UNIQUE(org_id, feature_key)
);

CREATE TABLE usage_credits (
  id UUID PRIMARY KEY,
  org_id UUID NOT NULL REFERENCES orgs(id),
  user_id UUID REFERENCES users(id),
  credit_type TEXT NOT NULL,
  amount_delta INTEGER NOT NULL,
  reason TEXT NOT NULL,
  metadata JSONB NOT NULL DEFAULT '{}',
  created_at TIMESTAMPTZ NOT NULL DEFAULT now()
);

CREATE TABLE audit_events (
  id UUID PRIMARY KEY,
  org_id UUID REFERENCES orgs(id),
  user_id UUID REFERENCES users(id),
  event_type TEXT NOT NULL,
  entity_type TEXT,
  entity_id TEXT,
  metadata JSONB NOT NULL DEFAULT '{}',
  created_at TIMESTAMPTZ NOT NULL DEFAULT now()
);
```

### 16.2. Market tables

```sql
CREATE TABLE constituents (
  id UUID PRIMARY KEY,
  symbol TEXT NOT NULL,
  company_name TEXT NOT NULL,
  sector TEXT NOT NULL,
  industry TEXT,
  sub_industry TEXT,
  exchange TEXT,
  cik TEXT,
  index_weight NUMERIC,
  market_cap NUMERIC,
  active BOOLEAN NOT NULL DEFAULT true,
  date_added DATE,
  date_removed DATE,
  source TEXT NOT NULL,
  source_updated_at TIMESTAMPTZ NOT NULL,
  license_status TEXT NOT NULL CHECK (license_status IN ('mock', 'risk_mode', 'licensed', 'unknown')),
  risk_mode BOOLEAN NOT NULL DEFAULT false,
  created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  updated_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  UNIQUE(symbol, active)
);

CREATE TABLE price_bars (
  id UUID PRIMARY KEY,
  symbol TEXT NOT NULL,
  timeframe TEXT NOT NULL,
  timestamp TIMESTAMPTZ NOT NULL,
  open NUMERIC NOT NULL,
  high NUMERIC NOT NULL,
  low NUMERIC NOT NULL,
  close NUMERIC NOT NULL,
  adjusted_close NUMERIC,
  volume NUMERIC,
  vwap NUMERIC,
  is_adjusted BOOLEAN NOT NULL DEFAULT false,
  source TEXT NOT NULL,
  source_updated_at TIMESTAMPTZ NOT NULL,
  license_status TEXT NOT NULL,
  risk_mode BOOLEAN NOT NULL DEFAULT false,
  created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  UNIQUE(symbol, timeframe, timestamp, source)
);

CREATE TABLE ticker_snapshots (
  id UUID PRIMARY KEY,
  symbol TEXT NOT NULL,
  snapshot_date DATE NOT NULL,
  price NUMERIC,
  change_1d NUMERIC,
  change_1w NUMERIC,
  change_1m NUMERIC,
  change_3m NUMERIC,
  change_6m NUMERIC,
  change_ytd NUMERIC,
  change_1y NUMERIC,
  volume NUMERIC,
  avg_volume_20d NUMERIC,
  relative_volume NUMERIC,
  market_cap NUMERIC,
  sector TEXT,
  industry TEXT,
  pe NUMERIC,
  forward_pe NUMERIC,
  ps NUMERIC,
  pb NUMERIC,
  ev_ebitda NUMERIC,
  eps_growth NUMERIC,
  revenue_growth NUMERIC,
  gross_margin NUMERIC,
  operating_margin NUMERIC,
  debt_to_equity NUMERIC,
  earnings_date DATE,
  source TEXT NOT NULL,
  source_updated_at TIMESTAMPTZ NOT NULL,
  license_status TEXT NOT NULL,
  risk_mode BOOLEAN NOT NULL DEFAULT false,
  created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  UNIQUE(symbol, snapshot_date)
);

CREATE TABLE technical_features (
  id UUID PRIMARY KEY,
  symbol TEXT NOT NULL,
  as_of_date DATE NOT NULL,
  sma_20 NUMERIC,
  sma_50 NUMERIC,
  sma_200 NUMERIC,
  ema_20 NUMERIC,
  rsi_14 NUMERIC,
  macd NUMERIC,
  macd_signal NUMERIC,
  atr_14 NUMERIC,
  volatility_20d NUMERIC,
  volatility_60d NUMERIC,
  distance_from_52w_high NUMERIC,
  distance_from_52w_low NUMERIC,
  above_sma_20 BOOLEAN,
  above_sma_50 BOOLEAN,
  above_sma_200 BOOLEAN,
  breakout_20d BOOLEAN,
  breakdown_20d BOOLEAN,
  gap_pct NUMERIC,
  relative_strength_spy_1m NUMERIC,
  relative_strength_spy_3m NUMERIC,
  relative_strength_sector_1m NUMERIC,
  relative_strength_sector_3m NUMERIC,
  source TEXT NOT NULL,
  source_updated_at TIMESTAMPTZ NOT NULL,
  created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  UNIQUE(symbol, as_of_date)
);

CREATE TABLE sector_features (
  id UUID PRIMARY KEY,
  sector TEXT NOT NULL,
  as_of_date DATE NOT NULL,
  return_1d NUMERIC,
  return_1w NUMERIC,
  return_1m NUMERIC,
  return_3m NUMERIC,
  relative_return_spy_1m NUMERIC,
  relative_return_spy_3m NUMERIC,
  breadth_above_sma_20 NUMERIC,
  breadth_above_sma_50 NUMERIC,
  breadth_above_sma_200 NUMERIC,
  advancers_decliners_ratio NUMERIC,
  avg_relative_volume NUMERIC,
  sector_strength_score NUMERIC,
  rotation_velocity NUMERIC,
  rotation_acceleration NUMERIC,
  sector_state TEXT,
  source TEXT NOT NULL,
  source_updated_at TIMESTAMPTZ NOT NULL,
  created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  UNIQUE(sector, as_of_date)
);

CREATE TABLE ticker_scores (
  id UUID PRIMARY KEY,
  symbol TEXT NOT NULL,
  as_of_date DATE NOT NULL,
  long_score NUMERIC NOT NULL,
  short_score NUMERIC NOT NULL,
  confidence NUMERIC NOT NULL,
  momentum_score NUMERIC,
  relative_strength_score NUMERIC,
  sector_score NUMERIC,
  volume_score NUMERIC,
  fundamental_score NUMERIC,
  valuation_score NUMERIC,
  risk_score NUMERIC,
  catalyst_score NUMERIC,
  earnings_risk_penalty NUMERIC,
  liquidity_score NUMERIC,
  explanation_json JSONB NOT NULL DEFAULT '{}',
  source TEXT NOT NULL,
  source_updated_at TIMESTAMPTZ NOT NULL,
  created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  UNIQUE(symbol, as_of_date)
);

CREATE TABLE ideas (
  id UUID PRIMARY KEY,
  symbol TEXT NOT NULL,
  direction TEXT NOT NULL CHECK (direction IN ('long', 'short')),
  horizon TEXT NOT NULL CHECK (horizon IN ('intraday', 'swing', 'medium_term')),
  score NUMERIC NOT NULL,
  confidence NUMERIC NOT NULL,
  title TEXT NOT NULL,
  thesis TEXT NOT NULL,
  signal_reason TEXT NOT NULL,
  catalyst TEXT,
  risk TEXT NOT NULL,
  entry_zone TEXT,
  target_zone TEXT,
  stop_loss TEXT,
  invalidation_point TEXT NOT NULL,
  evidence_json JSONB NOT NULL DEFAULT '[]',
  status TEXT NOT NULL DEFAULT 'active',
  source TEXT NOT NULL,
  source_updated_at TIMESTAMPTZ NOT NULL,
  created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  updated_at TIMESTAMPTZ NOT NULL DEFAULT now()
);

CREATE TABLE news_signals (
  id UUID PRIMARY KEY,
  symbol TEXT,
  sector TEXT,
  title TEXT NOT NULL,
  source_name TEXT NOT NULL,
  url TEXT NOT NULL,
  url_hash TEXT NOT NULL UNIQUE,
  published_at TIMESTAMPTZ NOT NULL,
  catalyst_tags TEXT[] NOT NULL DEFAULT '{}',
  sentiment_score NUMERIC,
  relevance_score NUMERIC,
  license_status TEXT NOT NULL,
  risk_mode BOOLEAN NOT NULL DEFAULT false,
  created_at TIMESTAMPTZ NOT NULL DEFAULT now()
);

CREATE TABLE provider_status (
  id UUID PRIMARY KEY,
  provider_key TEXT NOT NULL,
  data_type TEXT NOT NULL,
  status TEXT NOT NULL,
  last_success_at TIMESTAMPTZ,
  last_error_at TIMESTAMPTZ,
  last_error TEXT,
  kill_switch_enabled BOOLEAN NOT NULL DEFAULT false,
  license_status TEXT NOT NULL,
  risk_mode BOOLEAN NOT NULL DEFAULT false,
  metadata JSONB NOT NULL DEFAULT '{}',
  created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  updated_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  UNIQUE(provider_key, data_type)
);
```

### 16.3. Product tables

```sql
CREATE TABLE watchlists (
  id UUID PRIMARY KEY,
  org_id UUID NOT NULL REFERENCES orgs(id),
  user_id UUID NOT NULL REFERENCES users(id),
  name TEXT NOT NULL,
  created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  updated_at TIMESTAMPTZ NOT NULL DEFAULT now()
);

CREATE TABLE watchlist_items (
  id UUID PRIMARY KEY,
  watchlist_id UUID NOT NULL REFERENCES watchlists(id),
  symbol TEXT NOT NULL,
  added_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  added_reason TEXT,
  initial_direction TEXT,
  initial_price NUMERIC,
  initial_long_score NUMERIC,
  initial_short_score NUMERIC,
  initial_sector_strength NUMERIC,
  initial_momentum_state TEXT,
  initial_risk_flags JSONB NOT NULL DEFAULT '[]',
  initial_ai_thesis JSONB NOT NULL DEFAULT '{}',
  last_checked_at TIMESTAMPTZ,
  status TEXT NOT NULL DEFAULT 'active',
  UNIQUE(watchlist_id, symbol)
);

CREATE TABLE watchlist_changes (
  id UUID PRIMARY KEY,
  watchlist_item_id UUID NOT NULL REFERENCES watchlist_items(id),
  change_type TEXT NOT NULL,
  old_value_json JSONB,
  new_value_json JSONB,
  ai_summary TEXT,
  severity TEXT NOT NULL CHECK (severity IN ('info', 'warning', 'critical')),
  evidence_json JSONB NOT NULL DEFAULT '[]',
  created_at TIMESTAMPTZ NOT NULL DEFAULT now()
);

CREATE TABLE strategy_runs (
  id UUID PRIMARY KEY,
  org_id UUID NOT NULL REFERENCES orgs(id),
  user_id UUID NOT NULL REFERENCES users(id),
  strategy_name TEXT NOT NULL,
  universe TEXT NOT NULL DEFAULT 'sp500_current',
  parameters_json JSONB NOT NULL,
  start_date DATE NOT NULL,
  end_date DATE NOT NULL,
  benchmark_symbol TEXT NOT NULL DEFAULT 'SPY',
  return_pct NUMERIC,
  benchmark_return_pct NUMERIC,
  annualized_return_pct NUMERIC,
  win_rate NUMERIC,
  profit_factor NUMERIC,
  max_drawdown NUMERIC,
  sharpe NUMERIC,
  sortino NUMERIC,
  avg_win NUMERIC,
  avg_loss NUMERIC,
  number_of_trades INTEGER,
  assumptions_json JSONB NOT NULL DEFAULT '{}',
  warnings_json JSONB NOT NULL DEFAULT '[]',
  status TEXT NOT NULL,
  created_at TIMESTAMPTZ NOT NULL DEFAULT now()
);

CREATE TABLE ai_sessions (
  id UUID PRIMARY KEY,
  org_id UUID NOT NULL REFERENCES orgs(id),
  user_id UUID NOT NULL REFERENCES users(id),
  title TEXT,
  created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  updated_at TIMESTAMPTZ NOT NULL DEFAULT now()
);

CREATE TABLE ai_messages (
  id UUID PRIMARY KEY,
  session_id UUID NOT NULL REFERENCES ai_sessions(id),
  org_id UUID NOT NULL REFERENCES orgs(id),
  user_id UUID REFERENCES users(id),
  role TEXT NOT NULL,
  content TEXT NOT NULL,
  tool_calls_json JSONB NOT NULL DEFAULT '[]',
  evidence_refs_json JSONB NOT NULL DEFAULT '[]',
  model TEXT,
  input_tokens INTEGER,
  output_tokens INTEGER,
  cost_usd NUMERIC,
  confidence NUMERIC,
  created_at TIMESTAMPTZ NOT NULL DEFAULT now()
);

CREATE TABLE saved_presets (
  id UUID PRIMARY KEY,
  org_id UUID NOT NULL REFERENCES orgs(id),
  user_id UUID NOT NULL REFERENCES users(id),
  preset_type TEXT NOT NULL,
  name TEXT NOT NULL,
  config_json JSONB NOT NULL,
  created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  updated_at TIMESTAMPTZ NOT NULL DEFAULT now()
);

CREATE TABLE in_app_notifications (
  id UUID PRIMARY KEY,
  org_id UUID NOT NULL REFERENCES orgs(id),
  user_id UUID REFERENCES users(id),
  title TEXT NOT NULL,
  body TEXT NOT NULL,
  severity TEXT NOT NULL,
  entity_type TEXT,
  entity_id TEXT,
  read_at TIMESTAMPTZ,
  created_at TIMESTAMPTZ NOT NULL DEFAULT now()
);
```

---

## 17. Backend provider interfaces

### 17.1. MarketDataProvider

```python
from __future__ import annotations
from dataclasses import dataclass
from datetime import date, datetime
from decimal import Decimal
from typing import Protocol, Literal

LicenseStatus = Literal["mock", "risk_mode", "licensed", "unknown"]

@dataclass(frozen=True)
class SourceMeta:
    source: str
    source_updated_at: datetime
    license_status: LicenseStatus
    risk_mode: bool
    provider_key: str

@dataclass(frozen=True)
class ConstituentDTO:
    symbol: str
    company_name: str
    sector: str
    industry: str | None
    sub_industry: str | None
    exchange: str | None
    cik: str | None
    index_weight: Decimal | None
    market_cap: Decimal | None
    active: bool
    date_added: date | None
    date_removed: date | None
    meta: SourceMeta

@dataclass(frozen=True)
class PriceBarDTO:
    symbol: str
    timeframe: str
    timestamp: datetime
    open: Decimal
    high: Decimal
    low: Decimal
    close: Decimal
    adjusted_close: Decimal | None
    volume: Decimal | None
    vwap: Decimal | None
    is_adjusted: bool
    meta: SourceMeta

@dataclass(frozen=True)
class NewsSignalDTO:
    symbol: str | None
    sector: str | None
    title: str
    source_name: str
    url: str
    published_at: datetime
    catalyst_tags: list[str]
    sentiment_score: float | None
    relevance_score: float | None
    meta: SourceMeta

class MarketDataProvider(Protocol):
    provider_key: str

    async def get_constituents(self) -> list[ConstituentDTO]: ...

    async def get_daily_bars(
        self,
        symbols: list[str],
        start: date,
        end: date,
    ) -> list[PriceBarDTO]: ...

    async def get_fundamentals_snapshot(
        self,
        symbols: list[str],
        as_of: date,
    ) -> list[dict]: ...

    async def get_news_signals(
        self,
        symbols: list[str],
        since: datetime,
    ) -> list[NewsSignalDTO]: ...

    async def health(self) -> dict: ...
```

### 17.2. LLMProvider

```python
from typing import Protocol, Any

class LLMProvider(Protocol):
    provider_key: str

    async def chat_with_tools(
        self,
        *,
        model: str,
        messages: list[dict[str, Any]],
        tools: list[dict[str, Any]],
        tool_choice: str | dict[str, Any] = "auto",
        response_schema: dict[str, Any] | None = None,
        metadata: dict[str, Any] | None = None,
    ) -> dict[str, Any]: ...
```

### 17.3. ScoringEngine

```python
class ScoringEngine:
    def compute_scores(self, snapshot, technicals, sector_features, news_signals) -> dict:
        """Return deterministic long/short/confidence score with factor breakdown."""
        ...

    def generate_idea_candidate(self, symbol: str, score_payload: dict) -> dict:
        """Return idea card data with reason/risk/invalidation/evidence."""
        ...
```

### 17.4. BacktestEngine

```python
class BacktestEngine:
    def run(self, strategy_name: str, params: dict, universe: str, start_date, end_date) -> dict:
        """Run daily-bar backtest with no-lookahead rules, transaction costs, slippage and warnings."""
        ...
```

---

## 18. API contracts

All market responses must include `meta`:

```json
{
  "meta": {
    "as_of": "2026-04-27T20:00:00Z",
    "source": "mock_provider",
    "source_updated_at": "2026-04-27T20:00:00Z",
    "license_status": "mock",
    "risk_mode": false,
    "is_stale": false,
    "confidence": 0.8,
    "warnings": []
  }
}
```

### 18.1. Health

```http
GET /health
```

```json
{
  "status": "ok",
  "version": "0.1.0",
  "environment": "development",
  "time": "2026-04-27T20:00:00Z"
}
```

### 18.2. Data health

```http
GET /data-health
```

```json
{
  "status": "degraded",
  "providers": [
    {
      "provider_key": "mock_provider",
      "data_type": "prices",
      "status": "ok",
      "last_success_at": "2026-04-27T20:00:00Z",
      "license_status": "mock",
      "risk_mode": false,
      "kill_switch_enabled": false
    }
  ],
  "warnings": ["news provider disabled"]
}
```

### 18.3. Constituents

```http
GET /constituents?active=true
```

```json
{
  "meta": {},
  "items": [
    {
      "symbol": "AAPL",
      "company_name": "Apple Inc.",
      "sector": "Information Technology",
      "industry": "Technology Hardware",
      "active": true
    }
  ]
}
```

### 18.4. Market overview

```http
GET /market/overview
```

```json
{
  "meta": {},
  "index_proxy": {
    "symbol": "SPY",
    "change_1d": 0.42,
    "change_1w": 1.2,
    "change_1m": 3.4
  },
  "breadth": {
    "advancers": 312,
    "decliners": 188,
    "above_sma_20_pct": 64.0,
    "above_sma_50_pct": 61.0,
    "above_sma_200_pct": 57.0
  },
  "strongest_sectors": [],
  "weakest_sectors": [],
  "top_long": [],
  "top_short": [],
  "top_gainers": [],
  "top_losers": []
}
```

### 18.5. Heatmap

```http
GET /heatmap?period=1D&size_by=market_cap
```

```json
{
  "meta": {},
  "period": "1D",
  "size_by": "market_cap",
  "items": [
    {
      "symbol": "AAPL",
      "company_name": "Apple Inc.",
      "sector": "Information Technology",
      "industry": "Technology Hardware",
      "market_cap": 3000000000000,
      "index_weight": 6.4,
      "return_pct": 0.8,
      "relative_volume": 1.1,
      "long_score": 71,
      "short_score": 22,
      "top_reason": "Positive RS vs sector"
    }
  ]
}
```

### 18.6. Radar

```http
GET /radar?signal=ABNORMAL_VOLUME&sector=Information%20Technology&limit=50
```

```json
{
  "meta": {},
  "items": [
    {
      "symbol": "NVDA",
      "company_name": "NVIDIA Corporation",
      "sector": "Information Technology",
      "price": 123.45,
      "change_1d": 3.2,
      "change_1w": 7.8,
      "relative_volume": 2.1,
      "gap_pct": 1.4,
      "signals": ["ABNORMAL_VOLUME", "MOMENTUM", "BREAKOUT"],
      "long_score": 86,
      "short_score": 12,
      "reason": "20D breakout with volume confirmation"
    }
  ]
}
```

### 18.7. Sector rotation

```http
GET /sectors/rotation?period=1M
```

```json
{
  "meta": {},
  "items": [
    {
      "sector": "Information Technology",
      "strength": 78,
      "velocity": 8.2,
      "acceleration": 2.1,
      "state": "leader",
      "return_1m": 4.5,
      "relative_return_spy_1m": 1.3,
      "breadth_above_sma_50": 68.0,
      "top_stocks": ["NVDA", "MSFT", "AVGO"]
    }
  ]
}
```

### 18.8. Ideas top

```http
GET /ideas/top?direction=long&horizon=swing&limit=5
```

```json
{
  "meta": {},
  "items": [
    {
      "symbol": "MSFT",
      "direction": "long",
      "horizon": "swing",
      "score": 82,
      "confidence": 76,
      "reason": "Relative strength improving while sector strength accelerates",
      "risk": "Mega-cap concentration and earnings event risk",
      "invalidation": "Daily close below SMA50 and sector velocity turns negative",
      "evidence": [
        {
          "type": "technical",
          "label": "RS vs SPY 3M",
          "value": "top decile"
        }
      ]
    }
  ]
}
```

### 18.9. Ticker research

```http
GET /tickers/AAPL
```

```json
{
  "meta": {},
  "symbol": "AAPL",
  "company": {},
  "snapshot": {},
  "technicals": {},
  "fundamentals": {},
  "valuation": {},
  "sector_context": {},
  "scores": {},
  "news_signals": [],
  "ai_view": {
    "summary": "string",
    "bull_case": [],
    "bear_case": [],
    "trade_setup": {},
    "risks": [],
    "evidence": []
  }
}
```

### 18.10. AI sessions/messages

```http
POST /ai/sessions
POST /ai/messages
```

Request:

```json
{
  "session_id": "uuid",
  "message": "Find 5 best long ideas in Technology",
  "context": {
    "page": "ai_analyst"
  }
}
```

Response:

```json
{
  "answer": "string",
  "data_used": [
    {
      "tool": "get_top_long_short_ideas",
      "as_of": "2026-04-27T20:00:00Z"
    }
  ],
  "confidence": 0.74,
  "evidence": [],
  "usage": {
    "credits_used": 1,
    "credits_remaining": 49
  },
  "disclaimer": "For research purposes only, not financial advice. Data may be delayed."
}
```

### 18.11. Strategy runs

```http
POST /strategy/runs
```

```json
{
  "strategy_name": "momentum",
  "parameters": {
    "lookback_days": 63,
    "top_n": 20,
    "rebalance_frequency": "monthly",
    "transaction_cost_bps": 5,
    "slippage_bps": 5
  },
  "start_date": "2020-01-01",
  "end_date": "2025-12-31"
}
```

### 18.12. Watchlist

```http
POST /watchlists/{watchlist_id}/items
GET /watchlists/{watchlist_id}/changes
```

```json
{
  "symbol": "INTC",
  "added_reason": "Short momentum candidate from Radar",
  "initial_direction": "short"
}
```

---

## 19. Scoring model

### 19.1. Design principles

```txt
- deterministic;
- explainable;
- factor-based;
- no black box in MVP;
- no lookahead;
- confidence is separate from score;
- risk can lower confidence or penalize final score;
- stale data lowers confidence;
- missing data blocks or degrades output.
```

### 19.2. Long score

```txt
long_score =
  0.20 * momentum_score
+ 0.18 * relative_strength_score
+ 0.15 * sector_strength_score
+ 0.12 * volume_confirmation_score
+ 0.10 * trend_quality_score
+ 0.08 * catalyst_score
+ 0.07 * fundamental_quality_score
+ 0.05 * valuation_reasonableness_score
+ 0.05 * risk_adjustment_score
- earnings_risk_penalty
- extreme_overextension_penalty
```

### 19.3. Short score

```txt
short_score =
  0.20 * downside_momentum_score
+ 0.18 * negative_relative_strength_score
+ 0.15 * weak_sector_score
+ 0.12 * abnormal_selling_volume_score
+ 0.10 * trend_breakdown_score
+ 0.08 * negative_catalyst_score
+ 0.07 * fundamental_weakness_score
+ 0.05 * valuation_pressure_score
+ 0.05 * risk_adjustment_score
- short_squeeze_risk_penalty
- earnings_risk_penalty
```

### 19.4. Confidence

```txt
confidence =
  data_quality_score
* signal_agreement_score
* liquidity_score
* recency_score
* explanation_completeness_score
```

### 19.5. Idea requirements

Every idea must include:

```txt
symbol
company_name
sector
direction
horizon
score
confidence
reason
catalyst or "No catalyst data available"
risk
entry_zone if available
target_zone if available
stop_loss or invalidation_point
evidence list
data timestamp
source/license status
```

### 19.6. Invalidation examples

```txt
Long invalidation:
- daily close below SMA50;
- relative strength vs SPY turns negative;
- sector velocity turns negative;
- abnormal volume reverses;
- earnings within risk window.

Short invalidation:
- daily close above SMA50/SMA20 reclaim;
- short score drops below threshold;
- sector strength rebounds;
- positive catalyst appears;
- high squeeze risk.
```

---

## 20. Sector Rotation engine

### 20.1. Sector strength

```txt
sector_strength_raw =
  zscore(sector_return_1m_vs_spy)
+ zscore(sector_return_3m_vs_spy)
+ zscore(breadth_above_sma_50)
+ zscore(avg_relative_volume)
+ zscore(percent_stocks_positive_20d_momentum)
```

Normalize to 0–100.

### 20.2. Velocity

```txt
rotation_velocity_t = sector_strength_t - sector_strength_t_minus_5_sessions
```

### 20.3. Acceleration

```txt
rotation_acceleration_t = rotation_velocity_t - rotation_velocity_t_minus_5_sessions
```

### 20.4. States

```txt
Leader:
  strength > 70 and velocity > 0

Emerging:
  strength between 45 and 70 and velocity > 0 and acceleration > 0

Fading:
  strength > 50 and velocity < 0

Laggard:
  strength < 40 and velocity < 0

Rebound Candidate:
  strength < 45 and velocity > 0 and acceleration > 0

Neutral:
  everything else
```

### 20.5. UI blocks

```txt
- sector cards;
- strength over time chart;
- velocity/acceleration matrix;
- sector heat strip;
- top stocks by sector;
- AI explanation: where capital appears to rotate;
- stale/degraded state if data incomplete.
```

---

## 21. Frontend product specs

## 21.1. Home

Goal: 30-second decision dashboard.

Sections:

```txt
1. Market Mood
2. Data Freshness / Risk Mode banner
3. Breadth card
4. Top 5 Long
5. Top 5 Short
6. Sector Rotation preview
7. Top gainers/losers
8. Heatmap preview
9. News/catalyst links strip
10. AI daily summary
```

Acceptance:

```txt
- useful without clicking;
- all market cards show source/timestamp;
- long/short cards include risk/invalidation;
- degraded data visible;
- Pro/trial gating works;
- no advice language.
```

## 21.2. Heatmap

Inputs:

```txt
period: 1D, 1W, 1M, 3M, 6M, YTD, 1Y
size_by: market_cap, index_weight
color_by: return_pct
```

Behavior:

```txt
- D3 treemap;
- grouped by sector and industry;
- click ticker -> ticker research;
- click sector -> sector page filter;
- hover card with scores and reason;
- no Finviz exact copy.
```

## 21.3. Market Radar

Modules:

```txt
Top gainers
Top losers
Abnormal volume
Gap up
Gap down
20D breakouts
20D breakdowns
52-week highs
52-week lows
Relative strength leaders
Relative strength laggards
Earnings movers if data exists
Stocks in play
```

Columns:

```txt
Ticker
Company
Sector
Price
Change 1D
Change 1W
Change 1M
Relative volume
Gap %
RS vs SPY
RS vs sector
Long score
Short score
Signal tags
Reason
```

## 21.4. Long / Short Ideas

Filters:

```txt
direction
horizon
sector
score threshold
confidence threshold
risk flag
catalyst tag
```

Card must include:

```txt
ticker
company
sector
direction
horizon
score
confidence
reason
catalyst
risk
invalidation
evidence chips
timestamp/source
```

## 21.5. Sector Rotation

Components:

```txt
sector state cards
strength chart
velocity/acceleration matrix
sector breadth
top stocks per sector
AI sector explanation
```

## 21.6. Ticker Research

Sections:

```txt
Ticker header
Price chart
Snapshot metrics
Technical panel
Relative strength panel
Sector context
Fundamentals
Valuation
News/catalyst links
Scores
AI bull/bear/trade setup
Risk panel
```

Non-S&P behavior:

```txt
Show out-of-scope state:
"SnP Stock Lab currently supports S&P 500 constituents only."
```

## 21.7. AI Analyst

Answer format:

```txt
Answer
Data used
Candidates/table if relevant
Reasoning
Risks
What to watch next
Timestamp
Confidence
Disclaimer
```

Canonical prompts to support:

```txt
Find 5 best long ideas in Technology.
Which S&P 500 stocks are falling on abnormal volume?
Why is Intel falling today?
Compare AMD and NVDA.
Which sector is stronger than the market?
Give short candidates among Consumer Discretionary.
What changed in my watchlist since yesterday?
Explain why MSFT is in Top Long.
Run a simple momentum backtest.
```

## 21.8. Strategy Lab Lite

Strategies:

```txt
momentum
mean_reversion
relative_strength_pullback
sector_rotation
breakout_continuation
```

Results:

```txt
Total return
Benchmark return
Annualized return
Max drawdown
Sharpe
Sortino
Win rate
Profit factor
Average win/loss
Number of trades
Exposure
Turnover
Assumptions
Warnings
Trade list
Equity curve
```

## 21.9. Watchlist Lite

On add store:

```txt
symbol
price
reason
direction
long_score
short_score
sector strength
momentum state
risk flags
AI thesis snapshot
```

Daily detect:

```txt
price change
score change
sector change
momentum strengthening/weakening
breakout/breakdown
earnings risk
new catalyst link
idea invalidation
```

---

## 22. AI Analyst tool registry

### 22.1. Tools

```txt
get_market_overview
get_heatmap_snapshot
get_market_radar
get_top_long_short_ideas
get_sector_rotation
get_ticker_research
compare_tickers
get_watchlist_changes
run_simple_backtest
explain_score
get_data_health
```

### 22.2. Strict output schema

```json
{
  "answer": "string",
  "data_used": [
    {
      "tool": "string",
      "as_of": "string",
      "source": "string"
    }
  ],
  "key_points": ["string"],
  "risks": ["string"],
  "evidence": [
    {
      "type": "price|technical|sector|score|news|backtest|watchlist",
      "label": "string",
      "value": "string",
      "timestamp": "string"
    }
  ],
  "confidence": 0.0,
  "warnings": ["string"],
  "disclaimer": "For research purposes only, not financial advice. Data may be delayed."
}
```

### 22.3. AI refusal/degradation behavior

```txt
If current market data is unavailable:
- say data unavailable;
- do not answer from memory;
- suggest checking data health.

If ticker is outside S&P 500:
- say out of scope;
- do not analyze unless future feature explicitly added.

If user asks for guaranteed profit:
- refuse guarantee;
- offer research-style risk-based analysis.

If user asks for personal portfolio advice:
- refuse personalized advice;
- can provide general research context on S&P 500 names.
```

---

## 23. Wave execution system

Every wave follows this loop:

```txt
1. Parent orchestrator reads current repo and docs.
2. Spawn read-heavy subagents.
3. Collect findings.
4. Decide exact slice.
5. Assign write-heavy agents sequentially by file boundary.
6. Run tests.
7. Spawn QA/security/compliance reviewer.
8. Fix blockers.
9. Update docs.
10. Write next-wave prompt.
```

Definition of done:

```txt
- relevant tests pass;
- docs updated;
- no secrets;
- market data metadata present;
- AI evidence rules preserved;
- compliance risks reviewed;
- next-wave prompt produced.
```

---

# 24. Implementation waves

## Wave 0 — Bootstrap repo and agent operating system

Goal:

```txt
Create the repo skeleton and Codex operating system.
```

Spawn:

```txt
product_architect
system_architect
frontend_dashboard_engineer
qa_security_reviewer
```

Implement:

```txt
AGENTS.md
DESIGN.md
README.md
.codex/config.toml
.codex/agents/*.toml
.agents/skills/*/SKILL.md
docs skeleton
Next.js shell
FastAPI shell
Docker Compose skeleton
GitHub Actions CI skeleton
.env.example
```

Acceptance:

```txt
- repo installs or has clear setup commands;
- web shell starts;
- API /health works;
- CI placeholder runs;
- Codex agents/skills present;
- no secrets.
```

Prompt:

```txt
Execute Wave 0 for SnP Stock Lab.

Use PLAN.md decisions:
- US-first paid SaaS for S&P 500 swing traders.
- Keep all 9 MVP modules.
- Clerk orgs, Stripe billing, risk-mode data, AI credits, audit logs.
- Research-only, no brokerage, no personalized advice.

Spawn read-heavy agents:
- product_architect
- system_architect
- frontend_dashboard_engineer
- qa_security_reviewer

Then implement repo skeleton, AGENTS.md, DESIGN.md, .codex/config.toml, custom agents, Skills, docs skeleton, Next/FastAPI shells, Docker Compose, CI, .env.example.

Run basic checks. Return changed files, commands, blockers, and Wave 1 prompt.
```

---

## Wave 1 — Product, architecture, compliance lock

Goal:

```txt
Freeze MVP contract and launch constraints before real implementation.
```

Spawn:

```txt
product_architect
system_architect
compliance_risk_reviewer
marketing_growth_writer
```

Create/update:

```txt
docs/PRD.md
docs/ARCHITECTURE.md
docs/DATA_PROVIDER_MATRIX.md
docs/COMPLIANCE.md
docs/RELEASE_PLAN.md
docs/REFERENCE_ANCHORS.md
docs/MARKETING_COPY_GUARDRAILS.md
docs/SAAS_BILLING.md
docs/AUTH_ORGS.md
```

Acceptance:

```txt
- MVP scope clear;
- non-goals clear;
- legal/data launch blockers clear;
- data modes defined;
- public copy guardrails defined;
- SaaS plan defined;
- no code except doc/config fixes.
```

Prompt:

```txt
Execute Wave 1: Product spec and architecture lock.

Spawn product_architect, system_architect, compliance_risk_reviewer, marketing_growth_writer.

Create docs for PRD, architecture, data provider matrix, compliance, release plan, reference anchors, marketing copy guardrails, SaaS billing, auth/orgs.

Include all PLAN.md decisions. Do not implement product code except doc/config fixes. Return launch blockers and Wave 2 prompt.
```

---

## Wave 2 — SaaS foundation + DB contracts

Goal:

```txt
Implement org-scoped SaaS and database foundation before market features.
```

Spawn:

```txt
saas_billing_engineer
data_engineer
backend_api_engineer
qa_security_reviewer
```

Implement:

```txt
Clerk org mapping skeleton
Admin/Member roles model
Stripe webhook skeleton
Entitlements table
AI credits table
Alembic migrations
SQLAlchemy models
Pydantic schemas
mock seed data
org isolation tests
webhook idempotency tests
```

Acceptance:

```txt
- migrations run from zero;
- mock org/user/subscription can be seeded;
- webhook endpoint verifies signature in test mode or has test harness;
- org-scoped resources cannot leak;
- entitlements checked server-side;
- no secrets.
```

Prompt:

```txt
Execute Wave 2: SaaS foundation plus DB contracts.

Spawn saas_billing_engineer, data_engineer, backend_api_engineer, qa_security_reviewer.

Implement Clerk org mapping skeleton, Admin/Member roles, Stripe webhook skeleton, org-scoped DB tables, entitlements, AI credits, audit events, core market/product tables, Alembic migrations, seed script, tests for org isolation and webhook idempotency.

Use mock data. Return changed files, tests, blockers, Wave 3 prompt.
```

---

## Wave 3 — Provider abstraction, mock/risk-mode data, snapshots

Goal:

```txt
Make market data real-shaped without locking into unsafe source.
```

Spawn:

```txt
data_engineer
backend_api_engineer
devops_observability_engineer
compliance_risk_reviewer
```

Implement:

```txt
MarketDataProvider interface
MockMarketDataProvider
Risk-mode provider placeholders
provider status table integration
data health endpoint
kill-switches
daily price ingestion job
daily snapshot computation
source/source_updated_at/license_status propagation
stale-data rules
SEC client skeleton with rate limit/user-agent config
```

Acceptance:

```txt
- mock provider populates constituents/prices/snapshots;
- risk-mode sources disabled by default or labelled;
- data health shows provider status;
- kill-switch works;
- source metadata appears everywhere;
- missing/stale data lowers confidence or warns.
```

Prompt:

```txt
Execute Wave 3: Provider abstraction and snapshot engine.

Spawn data_engineer, backend_api_engineer, devops_observability_engineer, compliance_risk_reviewer.

Implement MarketDataProvider, mock provider, risk-mode provider placeholders, provider status, data health, kill-switches, ingestion jobs, daily snapshots, SEC client skeleton with fair-access config.

Every market output must include source/source_updated_at/license_status/risk_mode. Return tests and Wave 4 prompt.
```

---

## Wave 4 — Technical features

Goal:

```txt
Compute technical signals that power Radar, Ideas, Ticker Research and Strategy Lab.
```

Spawn:

```txt
quant_scoring_engineer
data_engineer
qa_security_reviewer
```

Implement:

```txt
SMA20/50/200
EMA20 optional
RSI14
ATR14
volatility 20D/60D
52-week high/low distance
gap percentage
relative volume
abnormal volume
20D breakout/breakdown
relative strength vs SPY
relative strength vs sector
feature computation job
formula tests
no-lookahead tests
```

Acceptance:

```txt
- formulas deterministic;
- no future data used;
- tests cover edge cases;
- features stored with timestamp/source;
- stale/missing data handled.
```

Prompt:

```txt
Execute Wave 4: Technical features.

Spawn quant_scoring_engineer, data_engineer, qa_security_reviewer.

Implement technical feature calculations, storage, feature computation job, API access where needed, and formula/no-lookahead tests.

Return formulas, tests, limitations, Wave 5 prompt.
```

---

## Wave 5 — Sector Rotation engine and page

Goal:

```txt
Build unique Sector Rotation feature: strength, velocity, acceleration.
```

Spawn:

```txt
quant_scoring_engineer
backend_api_engineer
frontend_dashboard_engineer
qa_security_reviewer
```

Implement:

```txt
sector returns
sector breadth
relative return vs SPY
sector strength score
rotation velocity
rotation acceleration
sector state classification
/sectors/rotation endpoint
Sector Rotation UI page
AI explanation placeholder using deterministic summary first
```

Acceptance:

```txt
- every sector has strength/velocity/acceleration;
- state visible: Leader/Emerging/Fading/Laggard/Rebound/Neutral;
- page has stale/degraded state;
- tests verify formulas;
- source/timestamp visible.
```

Prompt:

```txt
Execute Wave 5: Sector Rotation.

Spawn quant_scoring_engineer, backend_api_engineer, frontend_dashboard_engineer, qa_security_reviewer.

Implement sector strength, velocity, acceleration, states, endpoint and UI. Use mock/provider data. Add tests. Return changed files, screenshots if possible, Wave 6 prompt.
```

---

## Wave 6 — Long/Short scoring and ideas

Goal:

```txt
Produce explainable Top Long/Short candidates.
```

Spawn:

```txt
quant_scoring_engineer
ai_analyst_engineer
backend_api_engineer
compliance_risk_reviewer
qa_security_reviewer
```

Implement:

```txt
long score formula
short score formula
confidence score
risk penalties
earnings risk placeholder
short squeeze risk placeholder
explanation JSON
idea generation
top ideas endpoint
invalidation fields
compliance-safe language
```

Acceptance:

```txt
- every idea has reason/risk/invalidation/evidence;
- deterministic tests pass;
- confidence lowered by stale/missing data;
- no advice language;
- top 5 long/top 5 short available to Home.
```

Prompt:

```txt
Execute Wave 6: Long/Short scoring.

Spawn quant_scoring_engineer, ai_analyst_engineer, backend_api_engineer, compliance_risk_reviewer, qa_security_reviewer.

Implement deterministic long/short scoring, confidence, risk penalties, explanation JSON, idea generation, /ideas/top endpoint, invalidation fields, tests, and compliance-safe wording.

Return formulas, tests, risks, Wave 7 prompt.
```

---

## Wave 7 — Home dashboard + public marketing shell

Goal:

```txt
Create first full user-facing vertical slice.
```

Spawn:

```txt
frontend_dashboard_engineer
backend_api_engineer
marketing_growth_writer
compliance_risk_reviewer
qa_security_reviewer
```

Implement app Home:

```txt
market mood
breadth
data freshness banner
Top 5 Long
Top 5 Short
sector rotation preview
top gainers/losers
heatmap preview
news/catalyst links placeholder
AI daily summary placeholder
```

Implement public site:

```txt
marketing Home
Pricing
Legal placeholder pages
Start Trial CTA
research-only disclaimers
```

Acceptance:

```txt
- app Home useful in 30 seconds;
- timestamps/source visible;
- public copy safe;
- trial CTA present;
- paywall placeholders present;
- Playwright smoke test for Home.
```

Prompt:

```txt
Execute Wave 7: Home dashboard and public marketing shell.

Spawn frontend_dashboard_engineer, backend_api_engineer, marketing_growth_writer, compliance_risk_reviewer, qa_security_reviewer.

Implement app Home and public Home/Pricing/Legal placeholders. Ensure source/timestamp/freshness, research-only disclaimers, no advice language. Add smoke tests.

Return changed files, UI notes, Wave 8 prompt.
```

---

## Wave 8 — S&P 500 Heatmap

Goal:

```txt
Build the visual center of the product.
```

Spawn:

```txt
frontend_dashboard_engineer
backend_api_engineer
data_engineer
qa_security_reviewer
```

Implement:

```txt
/heatmap endpoint
D3 treemap component
period selector: 1D, 1W, 1M, 3M, 6M, YTD, 1Y
size_by selector: market_cap/index_weight
sector/industry grouping
hover tooltip
click to ticker
stale/degraded states
```

Acceptance:

```txt
- all active S&P 500 names shown;
- colors by period return;
- sizing works;
- no Finviz pixel-copy;
- tests for data transform;
- Playwright smoke for period switching.
```

Prompt:

```txt
Execute Wave 8: S&P 500 Heatmap.

Spawn frontend_dashboard_engineer, backend_api_engineer, data_engineer, qa_security_reviewer.

Implement heatmap endpoint and D3 treemap UI with period/size selectors, sector grouping, hover details, ticker click-through, stale states, and tests. Avoid Finviz pixel-copy.

Return changed files, tests, Wave 9 prompt.
```

---

## Wave 9 — Market Radar

Goal:

```txt
Make the movement discovery engine.
```

Spawn:

```txt
frontend_dashboard_engineer
backend_api_engineer
quant_scoring_engineer
qa_security_reviewer
```

Implement:

```txt
/radar endpoint
top gainers
top losers
abnormal volume
gap up/down
breakouts/breakdowns
52-week highs/lows
RS leaders/laggards
signal tags
filters
sorting
saved org presets
```

Acceptance:

```txt
- table sortable/filterable;
- each row has reason;
- signal tags visible;
- ticker click-through;
- saved preset org-scoped;
- tests for filters.
```

Prompt:

```txt
Execute Wave 9: Market Radar.

Spawn frontend_dashboard_engineer, backend_api_engineer, quant_scoring_engineer, qa_security_reviewer.

Implement radar endpoint, filters, signal tags, sortable table, saved org presets, reason generation, ticker click-through, and tests.

Return changed files, tests, Wave 10 prompt.
```

---

## Wave 10 — Ticker Research

Goal:

```txt
Build one-ticker deep dive.
```

Spawn:

```txt
frontend_dashboard_engineer
backend_api_engineer
ai_analyst_engineer
data_engineer
qa_security_reviewer
```

Implement:

```txt
S&P 500 ticker search
Ticker header
price chart
technical panel
fundamentals panel
valuation panel
relative strength panel
sector context
scores
news/catalyst links
AI bull/bear/trade setup block
out-of-scope state
```

Acceptance:

```txt
- non-S&P ticker blocked/marked out-of-scope;
- AI view evidence-grounded;
- all data timestamped;
- page dense/readable;
- tests for valid/invalid ticker.
```

Prompt:

```txt
Execute Wave 10: Ticker Research.

Spawn frontend_dashboard_engineer, backend_api_engineer, ai_analyst_engineer, data_engineer, qa_security_reviewer.

Implement ticker research page and endpoint with chart, technicals, fundamentals, valuation, RS, sector context, scores, catalyst links, AI bull/bear/trade setup block, and out-of-scope handling.

Return changed files, tests, Wave 11 prompt.
```

---

## Wave 11 — AI Analyst v1

Goal:

```txt
Build platform-grounded chat analyst.
```

Spawn:

```txt
ai_analyst_engineer
backend_api_engineer
saas_billing_engineer
compliance_risk_reviewer
qa_security_reviewer
```

Implement:

```txt
LLMProvider adapter
server-side tool registry
strict tool schemas
chat sessions
AI messages
90-day audit log policy
AI credit decrement
answer formatter
evidence refs
hallucination tests
canonical prompt tests
```

Acceptance:

```txt
- AI cannot answer current market questions without tools;
- evidence/timestamp/confidence included;
- AI credits decrement;
- org-scoped sessions;
- no invented news/prices;
- compliance-safe disclaimer.
```

Prompt:

```txt
Execute Wave 11: AI Analyst v1.

Spawn ai_analyst_engineer, backend_api_engineer, saas_billing_engineer, compliance_risk_reviewer, qa_security_reviewer.

Implement LLMProvider adapter, server-side tool registry, strict schemas, chat sessions/messages, evidence refs, audit logs, AI credit usage, answer formatter, and hallucination tests.

Return changed files, tests, risks, Wave 12 prompt.
```

---

## Wave 12 — Strategy Lab Lite

Goal:

```txt
Let users test simple hypotheses on daily bars.
```

Spawn:

```txt
strategy_lab_engineer
quant_scoring_engineer
backend_api_engineer
frontend_dashboard_engineer
compliance_risk_reviewer
qa_security_reviewer
```

Implement:

```txt
strategy parameter schemas
vectorized backtest engine
momentum strategy
mean reversion strategy
relative strength pullback
sector rotation strategy
breakout continuation
metrics
trade list
equity curve
org-scoped saved runs
cost/slippage controls
survivorship warning
hypothetical results disclaimer
```

Acceptance:

```txt
- no-lookahead tests;
- metrics correct;
- assumptions visible;
- paywall/entitlement works;
- saved runs org-scoped;
- UI displays warnings.
```

Prompt:

```txt
Execute Wave 12: Strategy Lab Lite.

Spawn strategy_lab_engineer, quant_scoring_engineer, backend_api_engineer, frontend_dashboard_engineer, compliance_risk_reviewer, qa_security_reviewer.

Implement daily-bar strategy lab with 5 MVP strategies, metrics, trade list, equity curve, org-scoped saved runs, costs/slippage, survivorship warning, hypothetical-results disclaimer, paywall checks and tests.

Return changed files, tests, Wave 13 prompt.
```

---

## Wave 13 — Watchlist Lite

Goal:

```txt
Turn ordinary watchlist into living research board.
```

Spawn:

```txt
backend_api_engineer
frontend_dashboard_engineer
ai_analyst_engineer
saas_billing_engineer
qa_security_reviewer
```

Implement:

```txt
watchlist CRUD
initial thesis snapshot
change detection job
score changes
sector changes
momentum changes
breakout/breakdown changes
earnings risk placeholder
catalyst link changes
AI change summaries
in-app notifications
entitlement limits
```

Acceptance:

```txt
- stores initial state;
- "what changed" evidence-grounded;
- org-scoped;
- watchlist limits enforced;
- change notifications generated;
- tests for change detection.
```

Prompt:

```txt
Execute Wave 13: Watchlist Lite.

Spawn backend_api_engineer, frontend_dashboard_engineer, ai_analyst_engineer, saas_billing_engineer, qa_security_reviewer.

Implement watchlist CRUD, initial thesis snapshots, change detection, AI summaries, in-app notifications, entitlement limits, and tests.

Return changed files, tests, Wave 14 prompt.
```

---

## Wave 14 — QA/security/observability/compliance hardening

Goal:

```txt
Make MVP beta-safe.
```

Spawn:

```txt
qa_security_reviewer
devops_observability_engineer
compliance_risk_reviewer
system_architect
```

Implement/review:

```txt
CI hardening
unit tests
integration tests
API contract tests
Playwright smoke tests
Sentry hooks
structured logs
data health dashboard
provider kill-switch UI
secret checks
rate limiting
security headers
legal copy review
AI audit review
billing webhook review
```

Acceptance:

```txt
- CI green;
- no secrets;
- data failures visible;
- AI evidence tests pass;
- compliance blockers listed;
- public beta readiness checklist complete.
```

Prompt:

```txt
Execute Wave 14: QA/security/observability/compliance hardening.

Spawn qa_security_reviewer, devops_observability_engineer, compliance_risk_reviewer, system_architect.

Review and harden CI, tests, API contracts, Playwright smoke, Sentry/logging, data health dashboard, provider kill-switch UI, secrets, rate limits, legal copy, AI audit logs, billing webhook safety.

Return blockers, fixed issues, beta readiness status, Wave 15 prompt.
```

---

## Wave 15 — Beta launch

Goal:

```txt
Launch controlled beta to 20–50 users.
```

Spawn:

```txt
product_architect
marketing_growth_writer
devops_observability_engineer
compliance_risk_reviewer
qa_security_reviewer
```

Implement:

```txt
beta onboarding
feedback widget
usage analytics
daily/weekly retention tracking
beta email copy
admin beta controls
known limitations page
pricing placeholder review
support flow
```

Acceptance:

```txt
- 20-50 beta users can onboard;
- trial/pro flow works or intentionally disabled;
- feedback captured;
- retention tracked;
- data limitations visible;
- legal blockers accepted or launch delayed.
```

Prompt:

```txt
Execute Wave 15: Controlled beta launch.

Spawn product_architect, marketing_growth_writer, devops_observability_engineer, compliance_risk_reviewer, qa_security_reviewer.

Prepare beta onboarding, feedback widget, usage analytics, retention metrics, beta copy, admin controls, known limitations, support flow, final legal/data checklist.

Return beta launch status, blockers, and post-beta roadmap.
```

---

## 25. Testing matrix

### 25.1. Data tests

```txt
- rejects/marks non-S&P tickers out-of-scope;
- every market record has source/source_updated_at/license_status;
- stale data warning works;
- provider kill-switch works;
- mock/risk/licensed modes separated;
- SEC client respects rate config and user-agent requirement;
- adjusted/raw OHLCV not silently mixed;
- missing bars handled;
- duplicate bars deduped or rejected;
- data health reports provider errors.
```

### 25.2. Quant tests

```txt
- SMA calculations correct;
- RSI stable on flat data;
- ATR stable;
- relative volume correct;
- gap correct;
- breakout does not use future data;
- sector strength deterministic;
- velocity/acceleration deterministic;
- long/short scores deterministic;
- confidence drops on stale/missing data;
- invalidation logic fires.
```

### 25.3. SaaS tests

```txt
- Clerk org maps to internal org;
- Admin/Member permissions enforced;
- member cannot access another org;
- unpaid org blocked from paid features;
- trial org has limited entitlements;
- Stripe webhook signature verified;
- webhook replay/idempotency safe;
- subscription cancel updates entitlements;
- AI credits decrement;
- feature limits enforced.
```

### 25.4. AI tests

```txt
- refuses current-market claim without tool data;
- includes data used;
- includes timestamp;
- includes confidence;
- includes evidence refs;
- does not invent news;
- handles stale data;
- handles non-S&P ticker;
- refuses guaranteed profit;
- avoids personal advice;
- canonical prompt tests pass.
```

### 25.5. UI tests

```txt
- public Home loads;
- Pricing loads;
- Legal pages load;
- app Home loads;
- Heatmap period switch works;
- Radar filters work;
- Sector page loads;
- Ticker click-through works;
- invalid ticker state works;
- AI chat sends message;
- Strategy run starts and shows warnings;
- Watchlist add/change works;
- Admin data health loads.
```

### 25.6. Compliance tests

```txt
- no broker execution routes;
- no "buy now" copy;
- no guaranteed return copy;
- no Finviz scraping;
- disclaimers visible;
- delayed/EOD data disclosed;
- backtest hypothetical warning visible;
- AI disclaimer visible;
- data license status visible to admin;
- risk-mode banner visible when applicable.
```

---

## 26. Admin / data health

Admin page sections:

```txt
Provider Status
- provider key
- data type
- status
- last success
- last error
- license status
- risk mode
- kill-switch toggle

Data Freshness
- price snapshots
- features
- scores
- ideas
- news signals

AI Usage
- messages last 24h/30d
- credits used
- cost estimate
- failed tool calls

Billing Health
- webhook last received
- failed webhook events
- active subscriptions
- trial users

Compliance Flags
- risk-mode enabled sources
- missing legal docs
- public launch blockers
```

Admin kill-switch actions:

```txt
- disable provider;
- disable AI Analyst;
- disable Strategy Lab;
- disable news links;
- disable public signup;
- force mock mode;
- force read-only mode;
- hide score publishing.
```

---

## 27. Observability

### 27.1. Logs

Log events:

```txt
provider.ingestion.started
provider.ingestion.completed
provider.ingestion.failed
features.compute.completed
scores.compute.completed
ai.tool_call.started
ai.tool_call.completed
ai.tool_call.failed
billing.webhook.received
billing.webhook.processed
billing.webhook.failed
watchlist.change.detected
strategy.run.completed
```

### 27.2. Metrics

```txt
API latency
API error rate
provider freshness lag
provider error rate
AI tool-call failure rate
AI cost per org
AI credits consumed
backtest runtime
watchlist changes generated
Stripe webhook failures
active trials
weekly active users
```

### 27.3. Alerts

```txt
provider stale > threshold
prices missing for > X symbols
scores not computed after close
AI error rate high
Stripe webhook failures
Sentry critical error
risk-mode provider accidentally enabled in production
```

---

## 28. Compliance and legal gates

Before public paid launch:

```txt
[ ] Attorney review: Terms
[ ] Attorney review: Privacy
[ ] Attorney review: Disclaimers
[ ] Index/trademark wording review
[ ] Market data provider rights review
[ ] News link/redistribution review
[ ] Adviser-risk review
[ ] Backtest disclaimer review
[ ] AI output disclaimer review
[ ] Risk-mode data disabled or approved
[ ] Public copy reviewed for financial advice claims
```

Default disclaimer:

```txt
SnP Stock Lab is for research and educational purposes only and does not provide personalized financial, investment, tax, or legal advice. Market data may be delayed, incomplete, or subject to provider limitations. Trading involves risk, including possible loss of capital. Backtested results are hypothetical and do not guarantee future performance.
```

Short UI disclaimer:

```txt
Research only. Not financial advice. Data may be delayed.
```

Backtest disclaimer:

```txt
Backtested results are hypothetical, use historical data, and may include survivorship bias depending on the selected universe. They do not guarantee future performance.
```

AI disclaimer:

```txt
AI responses are generated from available platform data and may be incomplete if data is stale, missing, or delayed. Use for research only.
```

---

## 29. Beta success metrics

Primary metric:

```txt
Weekly retention among beta users.
```

Supporting metrics:

```txt
- % users who open Home 3+ times/week;
- % users who click from Home to Ticker Research;
- heatmap usage frequency;
- radar filter usage;
- AI Analyst questions per active user;
- watchlist items created;
- strategy runs created;
- idea card expansion rate;
- user-reported usefulness score;
- false/unclear AI answer reports;
- data stale incidents.
```

Beta targets:

```txt
20–50 users
2–4 week controlled beta
weekly feedback loop
top 10 UX/data issues fixed before wider release
```

---

## 30. Immediate next action for the orchestrator

Use this exact first command/prompt in Codex:

```txt
You are the main orchestrator for SnP Stock Lab.

Read PLAN.md and the current repo state.

Product decisions are final unless impossible:
- US-first paid SaaS for swing traders.
- S&P 500-only research dashboard.
- Not brokerage, not personalized advice, not Finviz clone.
- Keep all 9 MVP modules: Home, Heatmap, Market Radar, Long/Short Ideas, Sector Rotation, Ticker Research, AI Analyst, Strategy Lab Lite, Watchlist Lite.
- Stack: Next.js, FastAPI, PostgreSQL, Redis, Celery, SQLAlchemy, Alembic, pnpm, uv, pytest.
- Deploy target: Vercel for web, Render for API/workers/Postgres/Redis.
- SaaS: Clerk orgs, Admin/Member roles, Stripe per-seat Pro plan, trial, AI credits.
- Data: mock-first, provider abstraction, risk-mode providers labelled/kill-switchable, delayed/EOD MVP, SEC-first fundamentals.
- AI: OpenAI-compatible adapter, server-side tool allowlist, strict structured outputs, evidence refs, 90-day audit log.
- Compliance: research-only, no execution, no personal portfolio advice, no guaranteed returns, no buy/sell-now language.

Execute Wave 0:
1. Spawn read-heavy subagents:
   - product_architect
   - system_architect
   - frontend_dashboard_engineer
   - qa_security_reviewer
2. Create repo skeleton:
   - AGENTS.md
   - DESIGN.md
   - README.md
   - .codex/config.toml
   - .codex/agents/*.toml
   - .agents/skills/*/SKILL.md
   - docs skeleton
   - apps/web skeleton
   - services/api skeleton
   - db/migrations skeleton
   - Docker Compose skeleton
   - GitHub Actions CI skeleton
   - .env.example
3. Run basic checks if possible.
4. Return:
   - files created;
   - commands to run;
   - tests/checks run;
   - blockers;
   - Wave 1 prompt.

Do not start implementation beyond Wave 0.
```

---

## 31. Master roadmap summary

```txt
Wave 0  -> repo/agents/design bootstrap
Wave 1  -> specs/compliance/architecture lock
Wave 2  -> SaaS + DB foundation
Wave 3  -> providers/data health/snapshots
Wave 4  -> technical features
Wave 5  -> sector rotation
Wave 6  -> scoring + ideas
Wave 7  -> Home + public marketing shell
Wave 8  -> Heatmap
Wave 9  -> Market Radar
Wave 10 -> Ticker Research
Wave 11 -> AI Analyst
Wave 12 -> Strategy Lab Lite
Wave 13 -> Watchlist Lite
Wave 14 -> QA/security/observability/compliance
Wave 15 -> controlled beta
```

Final MVP is ready when:

```txt
A beta user can sign up, start trial, open Home, understand market state, inspect Heatmap, find Radar movements, review Top Long/Short candidates, open Ticker Research, ask AI Analyst with evidence-grounded answers, run a simple backtest, add tickers to Watchlist, and see what changed — with all market outputs labelled by source/timestamp and all risky/compliance limitations visible.
```
