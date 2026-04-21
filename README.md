# write-evals

A Claude Code plugin that generates principled, product-specific AI evaluation suites.

## What It Does

Most eval suites measure generic quality. This plugin builds something different: a suite designed around the specific ways *your* product can fail *your* user, ranked by the cost of getting it wrong.

When invoked, the skill:
1. Shares the evaluation philosophy and five core principles upfront
2. Asks for any available context docs (product brief, PRD, strategy docs)
3. Gathers structured context about the product, users, and failure modes
4. Generates 6–12 product-specific eval categories with clear principles
5. Builds 3–5 detailed test cases per category with PASS/FAIL verdicts and reasoning
6. Produces LLM-as-judge prompts and deterministic assertion logic for each category
7. Outputs a complete eval suite document (Markdown + optional JSON)

## Invocation

**Slash command** (explicit):
```
/write-evals
```

**Context-based** (automatic): Claude will activate this skill when you describe an AI feature and ask how to test it, evaluate it, or know if it's working — even without using the word "eval."

## Reference Files

The skill loads these references during generation:

| File | Purpose |
|---|---|
| `references/eval-principles.md` | 8 principles of rigorous eval design — loaded before any output is generated |
| `references/category-library.md` | Category patterns by product type (RAG, legal, support, code gen, browser, agentic, data extraction) + non-obvious categories |
| `references/examples.md` | Three fully worked eval suites: Ironclad (legal AI), Firefox (browser AI), Intercom (support AI) |
| `references/judge-templates.md` | LLM-as-judge prompt templates, bias mitigation checklist, calibration protocol, JSON output schema |

## Output Format

The skill produces a structured Markdown document:

```
# Eval Suite: [Product Name]
## Overview
## Eval Philosophy Applied Here
## Eval Categories
  ### Category N: [Name]
    Definition, principle, pass/fail conditions
    #### Test Cases (3–5 per category)
    #### Grading Rubric (judge prompt or assertion logic)
## Recommended Eval Cadence
## What to Build First
## Golden Dataset Guidance
```

Optionally: a JSON eval suite compatible with standard harnesses (Braintrust, LangSmith, DeepEval, Promptfoo).

## Key Principles Encoded

1. **Highest-stakes failure first** — the failure that causes the most damage defines the first category
2. **Product-specific criteria** — no generic metrics (BLEU, "helpfulness") — everything emerges from the product context
3. **Binary verdicts** — PASS or FAIL, never 1–5 scores
4. **One criterion per grader call** — prevents halo effects
5. **Deterministic checks before LLM judges** — code assertions for objective checks; LLM judges only for genuine interpretation

## Non-Obvious Category Library

The plugin includes 8 non-obvious eval categories in `references/category-library.md` that standard checklists miss:

- **Confidence Calibration** — does expressed certainty match actual reliability?
- **Adversarial Robustness** — does the AI maintain behavior under prompt injection or social engineering?
- **Refusal Calibration** — catches *over*-refusal (the silent failure mode)
- **Graceful Degradation** — behavior on bad inputs, not just good ones
- **Post-Action Communication Accuracy** — does the confirmation message match what was actually done?
- **Context Window Faithfulness** — does it correctly recall early context in long sessions?
- **Scope Creep Detection** — does it stay within the task it was asked?
- **Counterfactual Sensitivity** — does rephrasing cause quality cliffs?

## Sources

Built on best practices from:
- [Hamel Husain — Evals FAQ](https://hamel.dev/blog/posts/evals-faq/)
- [Eugene Yan — Product Evals in Three Simple Steps](https://eugeneyan.com/writing/product-evals/)
- [Anthropic Engineering — Demystifying Evals for AI Agents](https://www.anthropic.com/engineering/demystifying-evals-for-ai-agents)
- [Evidently AI — LLM-as-Judge Complete Guide](https://www.evidentlyai.com/llm-guide/llm-as-a-judge)
- [Pragmatic Engineer — A Pragmatic Guide to LLM Evals](https://newsletter.pragmaticengineer.com/p/evals)
