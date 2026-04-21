# Eval Principles

Eight principles that separate rigorous eval suites from checkbox exercises. Reference these when designing categories and explaining tradeoffs to the user.

---

## Principle 1: Error Analysis Before Automation

**The rule**: Don't write automated evaluators for failure modes you imagined. Discover real failures first, then automate detection of those.

**Why**: LLMs have near-infinite failure surfaces. Pre-emptive evals almost always miss the actual ways a product fails and instead measure things that look rigorous but aren't discriminating. The most common eval mistake is writing 20 criteria before shipping and then discovering your real failures are completely different.

**What this means in practice**:
- Before writing any eval, review at least 20-50 real outputs (even if the product isn't live yet, use the model to generate outputs for realistic inputs)
- Take notes on what looks wrong — this is called "open coding"
- Group those notes into 5-8 failure categories — this is "axial coding"
- Write evals for those categories, not for abstract quality dimensions

**When you can skip this**: Very early-stage products with no outputs yet. In this case, write a lightweight first draft of evals based on the product's highest-risk failure mode, and plan to revise after seeing real outputs.

---

## Principle 2: Binary Verdicts Only

**The rule**: Every eval criterion produces PASS or FAIL. Never 1-5 scores.

**Why**: Likert scales introduce three problems:
1. **Inconsistency** — one annotator's "3" is another's "4." You need huge sample sizes to get statistical significance.
2. **Ambiguity** — "what does 3.5 mean I should fix?" PASS/FAIL tells you what to do; 3.5 doesn't.
3. **False precision** — a 3.8 vs. 4.0 difference looks meaningful but usually isn't.

Binary verdicts force clear thinking: you must define exactly what crosses the pass/fail line. That definitional work is where the real quality thinking happens.

**One exception**: Allow a "Tie" option in pairwise comparisons (A vs. B) when two outputs are genuinely equivalent. This reduces noise without introducing multi-point scales.

---

## Principle 3: One Criterion Per Evaluator Call

**The rule**: Each LLM-as-judge call evaluates exactly one criterion. Never ask a single call to assess multiple dimensions.

**Why**: Multi-criterion evaluation causes "halo effects" — one dimension bleeds into another. A polished, well-written response gets overrated on accuracy because it sounds confident. A terse response gets underrated on relevance because it seems incomplete.

**In practice**: Build separate evaluator prompts for each category. Combine results via simple logic (e.g., FAIL if any criterion fails, or define which criteria are blocking vs. advisory).

**What to watch for**: If your category definition has the word "and" in it, it probably needs to be split. "Accurate and complete" = two categories.

---

## Principle 4: Custom Criteria Always

**The rule**: Never use off-the-shelf metrics (BLEU, ROUGE, BERTScore, "helpfulness score") as your primary eval criteria.

**Why**: Generic metrics don't map to what actually fails in your product. BLEU measures n-gram overlap with a reference answer — completely useless for most AI products where multiple valid outputs exist. "Helpfulness" is undefined until you specify what helpfulness means for your specific feature.

**What works instead**: Derive criteria from your specific failure modes. "Policy Reference Accuracy" for Ironclad (does the output cite real clauses?) is infinitely more useful than "Hallucination Rate" — because it's specific enough to catch the actual failure and ignore valid variation.

**Standard metrics as supplementary signals**: Latency, cost-per-query, follow-up query rate, and regression rate are useful operational metrics but aren't quality evals. Track them separately.

---

## Principle 5: Ground Truth Before Automation

**The rule**: Before deploying an LLM-as-judge, validate it against human-labeled data. Target Cohen's Kappa ≥ 0.4.

**Why**: An uncalibrated judge can confidently grade every case wrong and you'd never know. You're trusting automation with quality — that trust must be earned.

**Calibration process**:
1. Label 50-100 examples by hand (or have a domain expert do it)
2. Run your judge on the same examples
3. Measure agreement: Cohen's Kappa or simple agreement rate
4. If Kappa < 0.4: your criteria are too vague or your judge prompt is miscalibrated. Revise.
5. Hold out 25% of your labels as a test set — don't use them for prompt iteration

**Target thresholds**:
- Kappa 0.4-0.6: Substantial agreement (acceptable for most use cases)
- Kappa 0.7+: Excellent agreement (target for high-stakes use cases)

---

## Principle 6: Capability Evals + Regression Evals

**The rule**: Maintain two eval suites serving different purposes.

**Capability evals** (development):
- Start at low pass rates (50-70%)
- Measure what you're improving
- These are the challenge problems
- When they saturate near 100%, graduate them to regression

**Regression evals** (production):
- Maintained near 100% pass rate
- Detect backsliding after model updates, prompt changes, or new features
- Should run on every deploy
- These are the "do not break" tests

**Why this matters**: If all your evals are regression tests, you can't tell if you're improving. If all your evals are capability tests, you don't know if you broke something. You need both.

**Warning sign**: If you're at 100% pass rate on all evals, your evals aren't hard enough. A meaningful eval suite should have some stress-test cases that are genuinely difficult.

---

## Principle 7: Deterministic First, LLM-as-Judge Second

**The rule**: Whenever you can check something with code, check it with code. Reserve LLM-as-judge for genuinely subjective qualities.

**Why**: Code-based checks are fast, cheap, perfectly consistent, and debuggable. LLM judges are slow, expensive, non-deterministic, and require calibration. Don't use a judge for what an assertion can verify.

**Use code/assertions for**:
- Format compliance (is the output valid JSON? does it have required fields?)
- Length constraints (is it under 200 words?)
- Forbidden content (does it contain any of these prohibited terms?)
- Reference validation (does it only cite from the provided source list?)
- Schema compliance (does the structured output match the expected schema?)

**Use LLM-as-judge for**:
- Tone and brand voice alignment
- Logical coherence and reasoning quality
- Relevance and contextual fit
- Nuanced completeness ("did it address the user's real concern?")

**Use human review for**:
- High-stakes edge cases
- Calibrating your LLM judge
- Catching evaluator drift over time
- Novel failures that automated systems haven't seen before

---

## Principle 8: Eval the Whole Pipeline, Not Just the Model

**The rule**: For systems with retrieval, tool use, or multi-step flows, evaluate each component separately in addition to end-to-end.

**Why**: End-to-end failures often originate upstream. If your retrieval is fetching the wrong chunks, your generator can't produce a correct answer — but an end-to-end eval just shows "wrong answer" without pointing to the root cause.

**For RAG systems**: Evaluate retrieval (Recall@k, Precision@k — did we fetch the right chunks?) separately from generation (Faithfulness — did the answer stay within what was retrieved?).

**For agentic systems**: Use a transition failure matrix — track what was the last successful step vs. where the first failure occurred. Hotspots reveal where to invest eval effort.

**For multi-turn systems**: Log complete sessions, not just individual turns. Annotate the first upstream failure — downstream failures often cascade from a single earlier mistake.

---

## Reference: Common Failure Mode Categories by Product Type

| Product Type | Common Failure Modes |
|---|---|
| RAG / Document Q&A | Hallucinated citations, out-of-context answers, missed relevant chunks |
| Legal / Contract AI | Fabricated clauses, policy non-conformance, missed high-risk provisions |
| Customer Support | Incorrect policy references, failure to escalate, unhelpful deflection |
| Code Generation | Syntax errors, logic bugs, security vulnerabilities, wrong API usage |
| Content Generation | Brand voice drift, factual errors, inappropriate tone, structural inconsistency |
| Browser/Navigation AI | Misread user intent, unnecessary detours, cognitive load increase |
| Medical / High-Stakes | Dangerous recommendations, missing caveats, wrong dosing/protocol |
| Data Extraction | Missing fields, wrong field mapping, format non-compliance |

Use these as starting points — always adapt to the specific product context.
