---
title: 'ConflictScore: Measuring How Language Models Handle Conflicting Evidence'
date: 2026-03-02
permalink: /posts/2026/03/conflictscore/
tags:
  - NLP
  - knowledge conflicts
  - LLMs
  - RAG
  - evaluation
  - factuality
  - truthfulness
  - benchmarks
---

> **TL;DR**: Existing “factuality/faithfulness” metrics usually ask: *is the answer supported by the evidence?*  
> **ConflictScore** asks a sharper question: *what if the evidence set itself disagrees—and the model acts overconfident anyway?*  
> We introduce a **claim-level** metric (CS-C, CS-R), a **benchmark** (ConflictBench), and show **conflict-aware regeneration** improves truthfulness on **TruthfulQA**.

---

## Why conflicts in evidence matter (and why current metrics miss them)

Modern LLM systems increasingly answer questions *with retrieved documents* (RAG), but retrieval often brings back **disagreeing sources**: different timestamps, interpretations, and opinions. In these settings, a response can look “supported” under standard metrics even if **half the documents contradict it**.

**ConflictScore** is built for exactly this situation: when *support and contradiction coexist* in the grounding set, we measure whether the model **acknowledges** that conflict—or confidently picks a side.

<!-- FIGURE 1 (from paper): fig:example
Paste your Figure~\ref{fig:example} here.
Suggested blog rendering: a wide image + short annotated callout.
Recommended filename: conflictscore_example.png
Caption (adapted): Examples of "bad" vs "good" responses under conflicting evidence.
-->

---

## What is ConflictScore?

ConflictScore evaluates a model response in three stages:

1. **Claim decomposition**: split the response into **atomic claims**.
2. **Evidence evaluation**: for each claim and each document, predict a relation:
   - **SUPPORTS**, **CONTRADICTS**, or **IRRELEVANT**
3. **Aggregate conflicts** into two complementary scores:

- **CS-C (ConflictScore-Count)**: fraction of claims that have **both** supporting and contradicting documents.
- **CS-R (ConflictScore-Ratio)**: for each claim, compute  
  \[
  \frac{|D^-|}{|D^+| + |D^-|}
  \]
  and average across claims. (Higher = more contradiction pressure.)

**Lower is better**: fewer conflicted claims, or weaker contradiction.

<!-- FIGURE 2 (from paper): fig:pipeline
Paste your Figure~\ref{fig:pipeline} here (the overview pipeline).
Suggested blog rendering: full-width figure.
Recommended filename: conflictscore_pipeline.png
Caption (adapted): Pipeline overview + why FactScore/SAFE-style aggregation can miss “supported AND contradicted”.
-->

---

## A concrete example: “supported” doesn’t mean “safe”

Many existing metrics effectively treat evidence as a *single pool*: if **any** document supports a claim, the claim is marked supported—even if other documents contradict it.

ConflictScore keeps documents **separate**, so it can flag claims that are *simultaneously supported and contradicted*.

If you want a one-line intuition to reuse in the blog:

> **Faithfulness is not enough when the evidence is inconsistent. Conflict awareness is a distinct skill.**

---

## ConflictBench: a unified benchmark for conflict detection

To systematically evaluate conflict detection, we introduce **ConflictBench**, a unified dataset spanning multiple conflict types:

- **Ambiguity / underspecification** (AmbigDocs)
- **Counterfactual / adversarial contradictions** (ContraQA, MacNoise)
- **Naturally contentious questions & opinions** (ConflictingQA)

### Dataset statistics

<!-- TABLE A (from paper): tab:calibration_stats
Paste Table~\ref{tab:calibration_stats} here.
Recommended: render as markdown table for readability.
Add 1 sentence after it: “ConflictBench totals X examples with a relatively balanced conf/no-conf split.”
-->

---

## How well does ConflictScore detect conflicts?

We evaluate the conflict-detection subtask:

> Given a **claim** + **grounding documents**, predict **Conflict** vs **No Conflict**  
> (Conflict = at least one SUPPORTS and at least one CONTRADICTS in the set)

### Main results (conflict detection)

<!-- TABLE B (from paper): tab:calibration_main
Paste Table~\ref{tab:calibration_main} here.
Suggested: keep it as an image (wide table), or convert to markdown if you prefer.
Callout to include: “Overall accuracy ~93% with strong precision/recall across conflict types.”
-->

### Where it fails: local inconsistency inside a single document

Some errors come from *documents that contradict themselves* (e.g., early paragraphs suggest X, conclusion denies X). That can confuse claim–document labeling.

<!-- FIGURE 3 (from paper): tab:example
Paste Figure~\ref{tab:example} (your “local inconsistency failure case”) here.
Suggested: keep as image (the colored snippet is hard in markdown).
Add 1–2 sentences describing “local inconsistency vs inter-document conflict”.
-->

---

## Benchmarking frontier LLMs: prompting helps… but only a bit

We benchmark multiple models under three prompting strategies:

- **RAG**: write a report from documents (no special instruction).
- **RAG (Balanced)**: mild instruction to hedge / consider perspectives.
- **RAG (Super-Balanced)**: strong, rule-based balanced reporting instruction.

### Results

<!-- TABLE C (from paper): tab:model_conflict
Paste Table~\ref{tab:model_conflict} here.
Add 2 bullets after:
- “Balanced prompting is usually best within a model family.”
- “But absolute gains are modest: conflict handling isn’t fixed by instructions alone.”
-->

**Key takeaway**: even when explicitly told to hedge, models often still “collapse” to a single stance in the presence of contradictory sources.

---

## Using ConflictScore as a corrective signal: improving TruthfulQA

ConflictScore isn’t just diagnostic: we also use it as **feedback** to *improve* answers.

We compare:

- **RAG**: answer with retrieved docs
- **Control-RAG**: stronger cautionary instructions (no ConflictScore)
- **Regenerated-RAG (R-RAG)**: generate → run ConflictScore → feed back conflict signal → regenerate

### Multiple-choice TruthfulQA results

<!-- TABLE D (from paper): tab:truthfulqa_mc
Paste Table~\ref{tab:truthfulqa_mc} here.
Add 1 sentence: “R-RAG consistently improves accuracy across proprietary + open models.”
-->

### Why regeneration works: it fixes wrong answers more than it breaks right ones

<!-- TABLE E (from paper): tab:truthfulqa_regeneration_analysis
Paste Table~\ref{tab:truthfulqa_regeneration_analysis} here.
Add 1–2 sentences:
- “Improve rate is high; harm rate is low.”
- “Net improvement is positive across models.”
-->

### Qualitative examples: one success, one failure

<!-- FIGURE 4 (from paper): tab:qual_examples
Paste Figure~\ref{tab:qual_examples} here.
Suggested: full-width image.
Add 2 sentences:
- success case: disambiguation / definition dependence
- failure case: misleading majority sources & reliability weighting
-->

---

## How to use ConflictScore in practice

If you want a lightweight “recipe” section:

### When ConflictScore is most useful
- RAG systems answering from **heterogeneous web sources**
- Domains with **time drift** (numbers, rankings, policies)
- Topics with **persistent disagreement** (health guidance, social policy, product claims)

### What to report alongside ConflictScore
- **CS-C** (how many claims are conflicted)
- **CS-R** (how severe the contradiction balance is)
- (Optional) a **conflict breakdown** by claim type or section (intro vs conclusion)

---

<!-- OPTIONAL TABLES (from paper):
- tab:claim_decomposition_prompt
- tab:evidence_evaluation_prompt
These are long; for a blog, consider:
(1) collapsible sections, or
(2) link-out to a gist / appendix page, or
(3) show only the “decision rules” excerpt.
-->

Example layout:

<details>
<summary><b>Claim decomposition prompt</b></summary>

(Paste Table~\ref{tab:claim_decomposition_prompt} here)

</details>

<details>
<summary><b>Document–claim evaluation prompt</b></summary>

(Paste Table~\ref{tab:evidence_evaluation_prompt} here)

</details>

---

## Limitations and what’s next

- **Cost**: claim-by-document checking can be expensive for long outputs and large retrieval sets.
- **Local inconsistency**: some “conflicts” are inside a single document rather than across documents.
- **Reliability weighting**: models can be swayed by many low-quality sources; future work could integrate source credibility and recency.

(If you want, you can copy in your Limitations paragraph almost verbatim here—blog readers appreciate honesty.)

---

## Resources

- Paper: (link once public)
- Code & data: (link once public)
- ConflictBench: (link once public)

---

## Suggested figure/table placement checklist (copy/paste friendly)

1. **Fig:example** → right after “Why conflicts matter” (hook)
2. **Fig:pipeline** → “What is ConflictScore?”
3. **Tab:calibration_stats** → “ConflictBench statistics”
4. **Tab:calibration_main** → “How well does it detect conflicts?”
5. **Fig:tab:example** → “Failure modes”
6. **Tab:model_conflict** → “Benchmarking frontier LLMs”
7. **Tab:truthfulqa_mc** → “TruthfulQA results”
8. **Tab:truthfulqa_regeneration_analysis** → “Why regeneration works”
9. **Fig:tab:qual_examples** → “Qualitative examples”
10. (Optional) **prompt tables** → appendix as collapsible blocks

---