---
layout: page
title: Projects
icon: fas fa-code-branch
order: 2
---

## Financial News Bot

A fully automated daily market brief delivered over Telegram. Every morning it pulls market headlines from RSS feeds (MarketWatch, CNBC, Investing.com) and per-ticker company news from Finnhub, retrieves closing prices from four independent sources (moomoo OpenD, Finnhub, yfinance, Twelve Data), and pipes everything through a two-model LLM pipeline on Groq's free tier (Llama 3.3 70B as reader, Llama 4 Scout as writer).

The system is designed around the principle that **a rule in code is a guarantee, a rule in a prompt is just a request**. A Python post-processing layer enforces hard constraints the model cannot override: direction-sanity checks, causal-inversion guards, and a four-source price arbitration system that only publishes a figure when at least two independent sources agree. 72 regression tests, all green.

**Stack:** Python, Groq API, Finnhub API, yfinance, Twelve Data, moomoo OpenD, Telegram Bot API

---

## Systematic Quant Trading Pipeline

A role-separated research pipeline for developing and validating systematic long-only US equity strategies.

- **quant-strategy** — hypothesis author; builds signals and runs in-sample backtests (Gate 1: must dominate SPY on all metrics including CAGR on 2007–2019 training data including the GFC bear market).
- **quant-validation** — independent gatekeeper; runs out-of-sample walk-forward validation and Deflated Sharpe Ratio gating (Gate 2: OOS 2020–present). Issues PASS/FAIL only.
- **quant-execution** — moomoo paper-trade runner and live-vs-backtest reconciliation (Gate 3). Paper by default; real money requires separate per-strategy sign-off.

A Chinese-wall separation between in-sample and out-of-sample data is enforced by design — the same engine is shared but each agent is locked to its designated data partition. Lifetime-scoped Holm correction (FWER) is applied across all out-of-sample looks across all sessions.

**Stack:** Python, moomoo OpenD API, pandas, scipy

---

## Quant Advisor

A read-only portfolio co-pilot on a real brokerage account. Advises on managing holdings and evaluating new trades; all execution is manual. Two-layer advice architecture: validated signals (Layer A) originate directional calls; technical and news context (Layer B) serve as risk gates only. Every recommendation is logged to a ledger and benchmarked against VOO.

**Stack:** Python, moomoo OpenD API, Finnhub API

---

## Claude Agent Ecosystem

A personal agent ecosystem built on Claude Code, covering:
- **Blog/Research agent** — research, drafting, and QA for blog posts
- **SRM Study agent** — structured actuarial exam prep with worked examples
- **Memory hygiene agent** — periodic cleanup of stale context across all agents
- **QA panel** — four-agent review system (Correctness, Assumption, Devil's Advocate, Synthesis) triggered on demand for rigorous output validation
