---
layout: post
title: "I Built a Market-News Bot And Just Spent All My Time To Stop It From Lying"
subtitle: "Guarding an Unattended LLM Against Confident Nonsense"
description: "A small reflection on how I stopped a free, unattended LLM markets bot from confidently hallucinating."
excerpt: "A free LLM that almost works great at summarising the market, until... you realise it's just confidently making things up. This is my current architecture to solve it."
date: 2026-06-27
tags: [LLM, hallucination, guardrails]
categories: [Data Engineering, AI]
---

# I Built a Market-News Bot And Spent All My Time To Stop It From Lying

### Guarding an Unattended LLM Against Confident Nonsense


*~10 min read*

## Premise
I usually don't have the time or the attention span to keep up with how the market and my holdings are doing (*because of studies, work, and everything else*).

So I built a bot. Every morning it pulls market headlines and closing prices, runs them through a free LLM, and sends me a market brief over Telegram. No human reviews it. No premium API costs. Just an automated daily summary for me to keep up with the news.

**But the catch is:** when a model is unattended, it will occasionally tell you something **very wrong with absolute confidence**, and everyone has encountered it one way or another and would typically call it by its industry term, **hallucination**.

## 1. The bot literally told me that peace was bad for gold...
One morning my bot opened the market brief with:

> *"Gold rallied as the ceasefire eased tensions across the region."*

It is hard to overstate how confidently wrong that is. Gold is the asset people flee to when the world looks scary. A ceasefire is **the opposite** of scary. If anything, peace should have dragged gold prices down. 

The model had taken two true facts:  ***'there was a ceasefire'*** and ***'gold went up'*** and welded them together with a causal claim that goes against one of the oldest reflexes in markets.

In actuality, the model has no understanding that gold rises on fear and falls when fear subsides. How an LLM works is that it **predicts what the next token is**. Given everything before a word, it picks 
what word is most statistically likely to come next, based purely on patterns in its training 
data.

Consider the example:

 > There are 3 apples on a table. You take 2. How many apples do you have?

 The obvious answer would be 2. But a smaller or less complex model would answer 1.

The model did not misread any numbers. It saw '3', 'take', '2' together, and the pattern those tokens form in training data strongly associated them with subtraction: 3 minus 2 equals 1. But the question asks what *you* have, not what remains on the table. It did not pick up on the part that changes the meaning.

The gold sentence was the same failure, just at a higher level. The model read "ceasefire", 
"gold", "rallied", and predicted the most likely causal sentence connecting them without processing whether that causation ran in the right/logical direction.

There is no "Is this true?" signal in the training loss. The model was never optimised to verify causation, only to produce statistically plausible text. Even instruction-tuned models retain this gap: they hallucinate less often, but never zero.

Of course, the obvious solution to "this model hallucinates sometimes" is to just **use a bigger, better model** (e.g. Claude, ChatGPT).

But because I wanted the whole process to be completely free (*as Claude's/ChatGPT's API costs money*), I decided to design an architecture around **making the model unable to misbehave** rather than buying my way to a model that behaves more often.

## 2. How The Bot Works

Every morning, it repeats the workflow:

1. **Pulls a pile of market headlines** and per-ticker company news.
2. **Pulls closing prices** for the indices and for the holdings I track.
3. **Checks an economic calendar for notable events** in the next week.
4. Hands all of that to free large language models (*Groq's free tier, running Llama models*) to **generate the report**.
5. **Sends me the generated report** via Telegram.

- The intelligence layer is two free models on Groq's free tier: *Llama 3.3 70B & Llama 4 Scout*. 
- For news, there are three sources: public RSS feeds (MarketWatch, CNBC, Investing.com), per-ticker company news from Finnhub's API (`finnhub.io`)

## 3.  Thesis: a rule in a prompt is a request, a rule in code is a guarantee

Here is the single most useful thing I learned:

> **You cannot completely fix a hallucination by adding a sentence to the prompt. Because at the end of the day, a rule in the prompt is just a request. But a rule in code is a guarantee.**

When the model wrote that peace lifted gold prices, my first instinct was to **edit the prompt**:

> An additional line: *"Never claim that de-escalation or peace increases the price of safe-haven assets like gold and silver."* 

It was a clear, explicit rule. And it worked. Well... most of the time. But **"most of the time"** isn't good enough when failure only needs to happen once, especially for an unattended model. On the one morning the headlines were unusually persuasive, the model quietly violated the instruction anyway.

The reason is because the weights are frozen at inference time, a system prompt cannot permanently teach the model a rule. The instruction exists only as tokens in the current context window, competing with everything else the model reads. It has no special authority over the fixed weights, which is why a persuasive enough input can override it.

And you might reasonably think: *Why not run a second model to QA the first?* That is a legitimate and sound technique, but it doesn't resolve the core issue. A QA model would just be another probabilistic component that can be wrong sometimes... and now you have two models that can hallucinate instead of one. For a bot that runs unattended with no human backstop, I didn't want correctness to be **probable**, I wanted the things that must never happen to be **impossible by construction**. That is a job for code, not for a better-behaved model.

So I stopped negotiating with the model and started constraining it instead. And I proceeded to build the project on the basis that:

 **Numbers are Python's job.**

The model never sees a number it is allowed to invent. It does not compute the S&P's daily return, it does not decide whether gold is "up sharply" or "roughly flat," and it does not get to quote a percentage from memory. Python fetches every figure, verifies it, formats it, and pastes it in. The model then writes the story *around* the numbers, but never the numbers themselves.

In a nutshell, this is a **division of labour**, and it runs along two seams.

## 4. The architecture: Python ↔ Model ↔ Python

The first seam is between **code and model**, and it is the backbone of the whole system.
- The model layer does what models are genuinely good at: reading messy natural-language news and writing fluent prose.
- The Python layer does what code is good at: holding facts, doing arithmetic, and enforcing rules that must never be violated.

The shape that matters is that the model is **sandwiched**. Python prepares and verifies everything going in, the model writes, and Python guards everything coming out. The model never touches the outside world directly, it cannot fetch a price, and it cannot publish a sentence.

```
The model is sandwiched between two layers of code:

  ┌──── PYTHON · IN ─────┐         ┌────── MODEL ──────┐         ┌─────── PYTHON · OUT ───────┐
                                       reads only                  - re-paste real numbers
    - verify every price               what Python                 - scrub bad causation
    - guarantee company     ─────▶    already vetted    ──────▶   - check every citation
      news is included                 vetted, then                - reject — don't patch
                                       writes a draft
  └──────────────────────┘         └───────────────────┘         └────────────────────────────┘
   'Inputs are made TRUE               'Models may                   'Only verified prose
   before the model sees'               still lie'                    ever reaches me'              
```

- Inputs are made *true* before the model ever sees them.
- Outputs are *checked* before they ever reach me.

## 5. The two-model layer: a reader and a writer

The second seam runs *inside* the model layer, and it's where I made the biggest structural mistake before getting it right.

My first instinct was to hand the whole thing to a single model: 'Here is the full feed of the morning's headlines (dozens of them, most irrelevant), now read all of it *and* write me the brief.' It half-worked, and the way it failed was instructive. A single model asked to both *sift* and *write* does neither well: bury the three headlines that matter inside forty that don't, and its attention smears across the noise. It pads the brief with filler it should have discarded, or fixates on a loud-but-irrelevant headline, because nothing ever forced it to decide what mattered *first*.

So I split the model layer in two, by task:

```
  ┌──── STAGE 1 · the reader ─────┐       ┌──── STAGE 2 · the writer ────┐
    Llama 3.3 70B                            Llama 4 Scout  (MoE)
    reads the full feed,                     sees only the shortlist,
    keeps the relevant few         ─────▶    writes the JSON brief
    (ranks and filters headlines)           (built for structured output)
  └───────────────────────────────┘       └──────────────────────────────┘
```

The **reader** (Llama 3.3 70B) does the judgement-heavy work of scoring the full feed down to a relevant shortlist. The **writer** (Llama 4 Scout, a smaller mixture-of-experts model built for structured output) never sees the noise, only the handful that survived so it can spend all its attention on prose. Rank first, then generate.

The reason this split helps is a property of how transformers process context. Attention is distributed across all tokens in the context window: when the context is full of irrelevant headlines, the model's ability to focus on the few that matter is diluted across noise. Giving the writer a pre-screened context means its attention is more efficiently allocated to signal rather than wasted on content the ranker already judged irrelevant.

## 6. Don't fully trust prices provided by a single source

Free data feeds disagree with each other more than you'd expect. Not dramatically, but often enough to matter. One feed hasn't updated after the close, another reports a stale or split-adjusted figure, a third is just wrong for thirty seconds. If you pipe a single source straight into the brief, you eventually report a number that's false.

So instead of trusting one source, I poll four: my broker's feed (moomoo's OpenD), Finnhub, yfinance, and Twelve Data. Then make them vote. The arbitration logic is the following:

```python
# only publish a price if at least two sources agree on it
def arbitrate(sources, tol_pp=1.0):
    primary = sources.get("moomoo")
    if primary is not None:
        # broker feed wins the moment any other source backs it up
        for name, other in sources.items():
            if name != "moomoo" and abs(primary["pct"] - other["pct"]) <= tol_pp:
                return primary, None

    # no agreement found: use the biggest agreeing cluster, or fall back and flag it
    agreeing = _largest_cluster(sources, tol_pp)
    if agreeing:
        return agreeing[0], None
    return _most_trusted(sources), "sources disagree: verify"
```

The rule is simple: **a price is only allowed into the brief if at least two independent sources agree on it to within a percentage point.** If the most trusted source stands alone against the others, it loses. If all four disagree, the number still goes out, but with a visible "verify" flag attached, because a loud admission of uncertainty is much safer than a confident wrong number.

It managed to catch a real bug too: QQQM was periodically printing the *index's* percentage instead of its own, and the two-source agreement rule quietly refused to publish the bad figure until a second source backed it up.

## 7. Catching claims that are false or that don't make sense

Even with verified numbers and complete context, the model still writes sentences that are simply *wrong*, not factually, but in their logic, their causation, or their internal coherence. This is where most of the engineering went: every rule in this section started as a prompt instruction the model eventually violated, and was moved to code. Each guard is a deterministic rule constraining a probabilistic output, a guardrailed pipeline where the model handles language and the guards enforce hard constraints the model cannot override.

**(A) Direction sanity.** The model would sometimes lead with a narrative that contradicted the very price it was explaining: *"Silver collapsed on profit-taking…"* on a day SLV closed up over 5%. It had a bearish story cached and reached for it regardless of the tape. The guard judges the **lead clause** against the actual price direction, because the lead clause is where the model smuggles in the stale story before walking it back with a qualifier:

```python
# catch when the opening sentence says the opposite of what the price did
# (ignore moves < 0.3%, not worth flagging)
def sentiment_contradicts(pct, text):
    if abs(pct) < 0.30:
        return False
    lead = QUALIFIER.split(text)[0]   # everything before the first "but" or "despite"
    return (pct > 0 and BEAR_WORDS.search(lead)) or (pct < 0 and BULL_WORDS.search(lead))
```

**(B) Causal-direction sanity.** This is the gold-and-peace lie from the opening, and it gets its own guard because the *direction of the causation* is what's wrong, not the facts. For safe-haven assets, the rule is absolute: de-escalation cannot be cited as the *cause* of a rally. The guard catches it both within a sentence (*"gold rose on the truce"*) and across sentences (*"…the two sides reached a truce. This lifted gold."*), and the sentence splitter is abbreviation-aware so a stray "U.S." can't hide the clause boundary:

```python
# drop any sentence that blames peace/de-escalation for a gold or silver rally
# "gold rose" is fine to keep, only the false causal link gets removed
def scrub_causal_inversions(text):
    kept, deesc_context = [], False
    for s in split_sentences(text):
        lead = QUALIFIER.split(s)[0]
        has_deesc = DEESCALATION.search(lead) and not ESCALATION.search(lead)
        if SAFE_HAVEN_UP.search(lead) and has_deesc:
            continue   # caught it within the same sentence
        if deesc_context and SAFE_HAVEN_CAUSAL_UP.search(lead) and not ESCALATION.search(lead):
            continue   # caught it bleeding across into the next sentence
        kept.append(s)
        deesc_context = deesc_context or has_deesc
    return " ".join(kept)
```

The guard does not rewrite these sentences into correct explanations. It **deletes** them after the detections are positive.

Running underneath both sanity checks is a single governing discipline: **when in doubt, reject, don't patch.** So the system's failure mode is silence. *"No clear catalyst in today's headlines"* is allowed to be the final word with no hedging after it.

## 8. Did any of this actually work?

**A network-free test suite** (currently 72 tests, all green)

 Each guard was born from a specific failure (the gold-peace lie, the silver-collapsed-on-a-green-day contradiction). Instead of just fixing the bug and moving on, tests were written that reproduces that exact failure scenario. Now if I ever accidentally weaken or break a guard, that test will turn red immediately.

## 9. Limitations

- **Guards can only catch known failure modes, not unknown ones.** Every guard here exists because the model already burned me in that specific way once. The gold-peace guard does nothing about the *next* category of confident nonsense I haven't seen yet. This is a catalogue of caught lies, not a proof of honesty.
- **Rules trade false negatives for false positives.** A guard that deletes "suspicious" sentences will occasionally delete a *true* one: a correctly-reasoned causal claim that happens to trip a pattern. I've chosen that trade deliberately (a missing sentence beats a false one), but it is a real cost, and on a quiet news day it can make the brief terser than it needs to be.
- **It's regex and rules, not understanding.** These guards match patterns, not meaning. A sufficiently novel phrasing of the gold-peace lie could slip past the specific patterns I wrote. The honest framing is that I've raised the cost of a particular class of error, not eliminated the class.
- **"Numbers are Python's job" has a boundary.** Python owns the index and ETF figures end-to-end. But the model is still allowed to surface *macro* numbers it reads in a cited headline (a yield, an oil price) and those I have not fully pinned. That will be the next boundary to harden.

I'll continue working on these next month.

## 10. If you want to build one for yourself too

I started out thinking I was only building a summariser. Turns out, I was actually building a *referee*, a layer that lets a creative, yet unreliable model do the part it's good at, while quietly refusing to let it speak on the things it gets wrong.

If there's one transferable lesson, it would be **instead of trying to make the model behave, you can consider making the system unable to misbehave.**

At least now it no longer gets to tell me that peace is bad for gold.
