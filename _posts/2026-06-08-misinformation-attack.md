---
title: 'Misinformation Attack: Can AI Search Survive a Flood of Machine-Generated Misinformation?'
date: 2026-06-08
permalink: /posts/2026/06/misinformation-attack/
tags:
  - NLP
  - RAG
  - LLMs
  - AI safety
  - misinformation
  - adversarial robustness
  - retrieval
  - trustworthy AI
---

> **TL;DR**: Modern AI assistants answer questions by retrieving from the live web (RAG), and they quietly assume the web is *trustworthy*. **Misinformation Attack** stress-tests that assumption with an ethical, fabricated-entity testbed. After **47 days, 30 synthetic entities, 5 prompts per entity, 5 AI-search systems, and 24,722 scored responses**, the clearest initial result is that **the system and its search backend matter far more than the website**: citation rates ranged from **93.9% for Perplexity to 0% for Claude/Brave**. Just as importantly, retrieval does not imply belief—GPT sometimes found and cited the sites while consistently refusing to treat the entities as independently verified.
>
> *This is an ongoing project. The first establishment/visibility study is complete; the misinformation dose-response experiments are the next stage.*

---

## The problem: retrieval is only as trustworthy as the web

Tools like ChatGPT Search, Gemini, Perplexity, and Copilot increasingly answer your questions by **retrieving documents from the open web** and summarizing them. This is powerful — but it rests on one fragile assumption:

> **The retrieved context is trustworthy.**

Generative AI has made it cheap to mass-produce fluent, plausible, false content. So a natural and worrying question follows:

> **What if an adversary floods the web with AI-generated falsehoods about a target — and the AI dutifully repeats them as fact?**

Call this the **misinformation attack**. Unlike a one-off prompt injection, it attacks the *information ecosystem* that RAG systems depend on. If a handful of fake articles can swing what an AI says about a person, a company, or a product, that's a real-world safety problem — especially for **ordinary, low-profile people** who have no protective web presence to drown out the noise.

---

## Three research questions

**Misinformation Attack** is organized around three questions, moving from *measuring* the threat to *defending* against it:

1. **Vulnerability scaling.** How much does it take? Concretely: **how many fabricated articles must an adversary publish** before a production AI system produces false, untruthful claims about an entity?
2. **Do existing defenses hold?** Certified and heuristic RAG defenses (RobustRAG, ReliabilityRAG, TrustRAG, AstuteRAG) promise robustness. **Do they actually survive the flood regime** — many coordinated malicious sources rather than a few?
3. **Can we fix it?** Can we design a defense that combines the **retrieval layer** (what gets trusted) and the **LLM layer** (how conflicting evidence is reasoned about) to bend the curve back?

This post focuses on **RQ1** — the measurement that makes everything else quantitative. The first 47-day study establishes the baseline: whether each system can retrieve the synthetic entities and whether it subsequently treats them as real.

---

## The core idea: attack *fabricated* entities, not real people

The scientifically ideal experiment — pollute the web about a real target and see when the AI breaks — is also clearly unethical. The key design move in Misinformation Attack resolves this tension:

> **Invent entities that don't exist, give them a believable web presence, confirm that production AI systems treat them as real, and then attack the *fabricated* entities.**

Because **no real person is ever attacked**, the ethical footprint drops sharply. Scientific validity is preserved by a **proxy-validity argument**: I show the fabricated entities are statistically *indistinguishable* — in how "known" they are to the AI — from real low-profile entities. If the fake entities look just like comparable real people to the model, then the attack dose measured on the fakes **estimates the dose that would harm comparable real people**.

The domains I work with include tennis players, companies, and institutions: checkable facts, low real-world harm, narratively rich, and free of medical/political sensitivity. Each fabricated entity gets a fixed ground-truth record (nationality/location, ranking, coach, etc.), and "truth" is operationally defined as that originally-seeded record.

---

<!-- ## Two scaling questions, not one

A central contribution is separating two doses that are usually conflated:

- **Establishment dose** — how much content turns a *non-existent* entity into one the AI confidently "knows"? *(non-existent → correctly known)*
- **Attack dose** — given an *established, correctly-known* entity, how much adversarial content flips the AI to a falsehood? *(correctly known → falsely known)*

Measuring both lets me ask the socially important question: **how much does a clean, established footprint actually protect an entity** from being defamed?

--- -->

<!-- ## What I'm building

Take tennis players as an example. To make a fabricated entity look organically real (and therefore indexable and retrievable), each one gets a small, cross-linked ecosystem:

- An official-looking **player site** on its own domain.
<!-- - A **stats/database-style** entry. -->
- A few **articles** (match report, profile, opinion) across separate domains.
<!-- - A **self-hosted discussion thread** where three distinct **fan personas** (e.g., a UK skeptic, a China enthusiast, a US stats nerd) talk about the player and link back — the inbound links are what make the cluster indexable. -->

I deliberately use **venues I control** rather than deceiving users on real platforms, and every fabricated name is **footprint-verified** to collide with no real person before use.

Two claim types let me test whether *what* you lie about matters:

- **Neutral-false** — a wrong statistic (e.g., career-high #84 asserted as #12).
- **Reputationally-damaging-false** — a fabricated ban or violation, which may trip model safety behavior. **If damaging claims are harder to plant, that's itself a finding.** -->

---

## How the experiment runs

- **Phase 0 — Pilot.** Validate the pipeline on 2–3 entities and measure **indexing latency**, the binding constraint on the whole timeline.
- **Phase 1 — Establishment.** Publish establishment content at varying doses; track "knownness" weekly across all target models; run the **proxy-validity** comparison against real low-profile players.
- **Phase 2 — Attack.** Inject adversarial content and **sweep the attack dose** (1, 3, 10, 30, … pages), using un-attacked controls to separate real attack effects from natural answer drift.
- **Phase 3 — Defense.** Evaluate detection and mitigation methods, then develop stronger defenses against misinformation attacks.
- **Phase 4 — Takedown & persistence.** Remove the content and measure the **attack half-life** — how long the AI keeps repeating the falsehood after the source is gone.


**Models under test:** ChatGPT Search, Gemini (with Search), Perplexity, Claude (web search), Grok, and Bing/Copilot — reported per model.

---
<!--
## What gets measured

The headline quantity is **Dose-to-Flip** `D50(e)` — the smallest adversarial dose at which the model asserts the falsehood at least half the time. Alongside it:

- **Attack Success Rate** `ASR(e, d)` and the full **answer-state migration** from truth → target-falsehood → refusal as dose rises.
- **Entity-relative dose** `d* = D50 / footprint size` — a normalized number comparable *across* entities, and the bridge to the real-world claim.
- **Adversarial-citation share** — linking the behavioral flip back to the retrieval channel.
- **Confidence-when-wrong** — is the model *confidently* wrong as the flood grows?
- **Persistence / half-life** after takedown.

Every curve comes with bootstrap confidence intervals, and dose-to-flip is reported as a **distribution over phrasings**, not a single cherry-picked number. -->

<!-- --- -->

## Ethics first

The entity design is what makes this *defensible* — but it's lower-harm, not zero-harm, and I treat ethics as a first-class part of the project:

- **No real targets.** Real entities are *measured* for knownness only, never attacked.
- **Low-harm domain**, minimal footprint, prompt takedown, and search-engine decache requests.
- **Provenance markers** so future crawlers can identify the content as fabricated.
- **Dual-use mitigation**: gate the weaponizable operational details, release the **evaluation harness** (not the recipe), and pair the attack with the RQ3 defense.
- **Responsible disclosure** to providers before publication, and **early IRB** consultation.

---

## Why this matters

If a small number of coordinated fake sources can reliably flip what production AI says about an entity, then "the model cites its sources" is a false comfort — and the people most exposed are exactly those without a strong web presence to defend them. Misinformation Attack aims to turn that worry into a **measured scaling law**: a number for how fragile RAG is, evidence on whether today's defenses can bend it, and a path toward a defense that can.

> **From "we measured a number" to "we measured a scaling law that current defenses can't bend — and built one that can."**

---

## Progress log

**Sep 10, 2026 — Initial visibility study completed.** From July 25 through September 10, I probed 30 synthetic entities using five prompts across five AI-search systems. The resulting dataset contains **24,722 scored responses**.

### Initial results: the search system dominates

The same entities, sites, and prompts produced radically different outcomes:

| System | Search backend | Responses | Responses citing a test site | Entities ever cited |
|---|---|---:|---:|---:|
| Perplexity (Sonar) | Perplexity index | 4,201 | **93.9%** | 30/30 |
| Gemini (3.5 Flash) | Google | 4,972 | **83.8%** | 30/30 |
| Grok (4.3) | Live crawl | 4,292 | **66.7%** | 30/30 |
| GPT (5.4 mini) | Bing | 5,101 | **11.4%** | 15/30 |
| Claude (Haiku 4.5) | Brave | 5,103 | **0.0%** | 0/30 |

Claude searched normally and returned result URLs, but none of the test domains appeared in 5,103 responses. Because the API exposes its raw search results, this is a measured absence from the Brave-backed retrieval results rather than an inference from the model's silence.

### Retrieval and belief are separate failure points

The most important conceptual result is that **finding a source is not the same as believing it**. GPT cited the test sites in **581 responses across 15 entities**, yet on all 34 judged run-days it never classified an entity as real. It usually described what the site claimed while noting the lack of independent corroboration, often using the sites' own research disclaimers as evidence.

Gemini behaved very differently on the same pages, regularly treating 25–30 entities per run as real. A citation-only evaluation would make GPT and Gemini look different merely in degree; in practice, they differ in kind. This means the larger attack study must measure at least two axes:

1. **Retrieval penetration:** did the system find and cite the planted source?
2. **Belief penetration:** did the answer adopt the source's claims as factual?

### Indexing changed over time

- **Google indexed fastest:** Gemini went from 0 to 26 covered entities within three days, before any manual submission.
- **Bing indexed slowest but continued climbing:** GPT first cited a test site eight days after submission through Bing Webmaster Tools and IndexNow, then grew from 2 to 15 covered entities.
- **Gemini's coverage later declined:** it cited a mean of 29.9 entities per run from August 3–15, versus 24.3 from August 17 onward.
- **Grok's apparent collapse is confounded:** citations fell from 145/150 responses on August 19 to 0/150 from August 27 onward, but the break coincided with an OpenRouter key replacement. The model and prompts were unchanged and search continued, so this looks like a real retrieval change, but it cannot be cleanly attributed to xAI.

### What did—and did not—affect visibility

Only GPT/Bing retained enough site-level variation for a useful comparison; Perplexity, Gemini, and Grok each eventually cited all 30 entities.

- **Single-page sites were more discoverable by GPT/Bing:** 14/18 were ever cited, compared with 1/12 multi-page sites (Fisher's exact test, `p = 0.00048`). This is not clean evidence that fewer pages are inherently better: page count was entangled with builder choice, JSON-LD, and—in ten sites—an initially broken canonical domain. A plausible explanation is that low-authority sites had only their homepages crawled.
- **Link structure had no detectable effect:** even both tennis sites with neither inbound nor outbound links were discovered. The groups contain only 2–3 sites each, so this rules out only very large effects.
- **Prompt phrasing did not materially change retrieval:** citation rates remained nearly flat across the five prompts. The unique entity name appeared sufficient to drive retrieval, although phrasing did affect whether answers hedged.

### Current interpretation and limitations

These are **visibility and establishment results, not yet misinformation-attack success rates**. Citation counts are derived directly from URLs and are the most reliable measurements. Belief labels come from a text classifier that over-counted belief for some providers—especially Grok—so the strongest belief claims above use hand-audited results.

Other caveats include unequal numbers of run-days across providers, configuration changes during the study, small entity subgroups, and contamination after the public project writeup began appearing in GPT's search results. The next stage will use this baseline to select established entities and measure the adversarial dose required to move answers from truthful or uncertain to confidently false.

**Aug 2026 — Entity ecosystems built and deployed.** I've stood up and deployed the first batch of websites. These are now live and entering the indexing/knownness-measurement phase (Phase 1). The deployed research sites are listed below.



### Deployed research sites (in use)

**sites & databases**

- <https://aldenbrook.netlify.app>
- <https://bramlee.netlify.app/>
- <https://brindlemark.netlify.app>
- <https://caelthorn.netlify.app>
- <https://calverne.netlify.app>
- <https://cindergrid.netlify.app>
- <https://corolane.netlify.app/>
- <https://corvath.netlify.app/>
- <https://ellwickacademy.netlify.app>
- <https://kairvane.netlify.app>
- <https://kesterline.netlify.app>
- <https://marnley.netlify.app>
- <https://merasyl.netlify.app>
- <https://orisetteacademy.netlify.app>
- <https://orlein.netlify.app>
- <https://orrivex.netlify.app>
- <https://quorivane.netlify.app>
- <https://sennvue.netlify.app>
- <https://valescend.netlify.app/>
- <https://vantrelle.netlify.app>

**Tennis players**

- <https://alina-morren.netlify.app/>
- <https://davrik-tolsenne.netlify.app/>
- <https://emil-kevir.netlify.app/>
- <https://kira-orsenn.netlify.app/>
- <https://luca-pellis.netlify.app>
- <https://mireya-voss.netlify.app>
- <https://noemi-calvett.netlify.app>
- <https://rafael-delvar.netlify.app>
- <https://sylvra-quenmark.netlify.app>
- <https://tomas-sarnell.netlify.app>

*Project status: the entity testbed is deployed and the initial establishment/visibility study is complete. The project is now moving from baseline measurement toward controlled misinformation dose-response experiments.*
