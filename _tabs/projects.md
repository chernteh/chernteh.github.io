---
layout: page
title: Projects
icon: fas fa-code-branch
order: 2
---

<div style="display: flex; flex-direction: column; gap: 1.5rem;">

  <a href="/posts/i-built-a-market-news-bot-and-just-spent-all-my-time-to-stop-it-from-lying/" style="text-decoration: none; color: inherit;">
    <div style="border: 1px solid var(--border-color, #444); border-radius: 8px; padding: 1.25rem 1.5rem; transition: border-color 0.2s;">
      <h3 style="margin: 0 0 0.5rem;">📰 Financial News Bot</h3>
      <p style="margin: 0 0 0.75rem; opacity: 0.8;">A fully automated daily market brief delivered over Telegram. Two-model LLM pipeline (Llama 3.3 70B + Llama 4 Scout) with Python guardrails for hallucination prevention and 4-source price arbitration.</p>
      <small style="opacity: 0.6;">Python · Groq API · Finnhub · Telegram · moomoo OpenD</small>
    </div>
  </a>

  <a href="https://github.com/chernteh" target="_blank" style="text-decoration: none; color: inherit;">
    <div style="border: 1px solid var(--border-color, #444); border-radius: 8px; padding: 1.25rem 1.5rem; transition: border-color 0.2s;">
      <h3 style="margin: 0 0 0.5rem;">📈 Systematic Quant Trading Pipeline</h3>
      <p style="margin: 0 0 0.75rem; opacity: 0.8;">Role-separated research pipeline for developing and validating systematic long-only US equity strategies. Three-gate framework: in-sample backtest, OOS walk-forward + Deflated Sharpe validation, and paper-trade execution on moomoo.</p>
      <small style="opacity: 0.6;">Python · moomoo OpenD API · pandas · scipy</small>
    </div>
  </a>

  <a href="https://github.com/chernteh" target="_blank" style="text-decoration: none; color: inherit;">
    <div style="border: 1px solid var(--border-color, #444); border-radius: 8px; padding: 1.25rem 1.5rem; transition: border-color 0.2s;">
      <h3 style="margin: 0 0 0.5rem;">🤖 Claude Agent Ecosystem</h3>
      <p style="margin: 0 0 0.75rem; opacity: 0.8;">Personal agent ecosystem covering blog research, SRM exam prep, portfolio advisory, and a 4-agent QA panel (Correctness, Assumption, Devil's Advocate, Synthesis) for rigorous output validation.</p>
      <small style="opacity: 0.6;">Claude Code · Python · MCP</small>
    </div>
  </a>

</div>
