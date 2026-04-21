# LLM-as-Judge Templates

Prompt templates and JSON schemas for automated grading. Adapt these to each specific eval criterion — never use them verbatim across different products.

---

## Core Judge Prompt Structure

Every LLM-as-judge prompt should have exactly these five components in this order:

```
1. ROLE CONTEXT
   What is the AI system being evaluated? What does it do? Who does it serve?

2. CRITERION
   What single dimension are we evaluating right now? Define it precisely.

3. PASS/FAIL DEFINITION
   What does PASS look like? What does FAIL look like? Be specific.

4. EXAMPLES (2-3 labeled)
   At least 1 PASS example, 1 FAIL example. Optionally 1 borderline.

5. TASK
   Present the actual input+output to evaluate. Ask for reasoning then verdict.
```

---

## Master Template

```
You are evaluating an AI output. Your job is to assess one specific criterion — do not evaluate anything else.

## System Context
{{system_description}}
This system serves {{user_type}} who are trying to {{user_goal}}.

## Criterion: {{criterion_name}}
{{criterion_definition}}

## What PASS looks like
{{pass_description}}

## What FAIL looks like
{{fail_description}}

## Labeled Examples

### Example 1 (PASS)
Input: {{pass_example_input}}
Output: {{pass_example_output}}
Verdict: PASS
Reasoning: {{pass_example_reasoning}}

### Example 2 (FAIL)
Input: {{fail_example_input}}
Output: {{fail_example_output}}
Verdict: FAIL
Reasoning: {{fail_example_reasoning}}

## Now Evaluate This

Input: {{actual_input}}
Output to evaluate: {{actual_output}}

Think step by step:
1. What criterion am I evaluating?
2. What would PASS look like for this specific input?
3. Does the output meet that standard?

Return your evaluation as JSON:
{
  "criterion": "{{criterion_name}}",
  "reasoning": "your step-by-step reasoning here",
  "verdict": "PASS" or "FAIL",
  "confidence": "high" or "medium" or "low",
  "key_evidence": "the specific phrase or absence that drove the verdict"
}
```

---

## Template Variants by Category Type

### For Groundedness / Source Fidelity

Groundedness evals require the source documents to be passed alongside the output. The judge must have access to both.

```
You are evaluating whether an AI output is grounded in its source material.

## System Context
{{system_description}}

## Source Material
{{source_content}}

## Criterion: Source Fidelity
Every claim, citation, recommendation, or piece of reasoning in the output must be traceable to the source material above. The output should not introduce facts, standards, or positions that cannot be found in or clearly inferred from the source.

## What PASS looks like
Every factual claim can be verified against the source. If the output says "per section 4.2" or "as stated in the policy," that text exists there. If the output makes a general recommendation, it's consistent with what the source says.

## What FAIL looks like
The output references a clause, standard, policy position, or fact that does not appear in the source material — even if it sounds plausible. A single fabricated claim is sufficient to fail.

## Source Fidelity is NOT violated by:
- Paraphrasing (as long as the meaning matches the source)
- Summarizing (as long as nothing is added)
- Recommending based on source content (as long as the recommendation is consistent with the source)

Input: {{actual_input}}
Source: [provided above]
Output to evaluate: {{actual_output}}

Evaluate strictly. Return JSON:
{
  "criterion": "Source Fidelity",
  "reasoning": "...",
  "verdict": "PASS" or "FAIL",
  "confidence": "high" or "medium" or "low",
  "key_evidence": "quote the specific phrase that passed or failed"
}
```

---

### For Relevance / Intent Accuracy

```
You are evaluating whether an AI response addresses what the user actually needed.

## System Context
{{system_description}}

## Criterion: Relevance / Intent Accuracy
The response must address the user's actual underlying need — not just the literal phrasing of their request. Users often express their goals imprecisely; a good response identifies the real goal and addresses it.

## What PASS looks like
The response directly helps the user accomplish their goal. If the user's literal question differs from their underlying need, the response still addresses the underlying need.

## What FAIL looks like
The response answers the literal question but misses what the user was actually trying to do. Or the response addresses a related but different topic.

## Note on interpretation
When intent is genuinely ambiguous, a response that asks a targeted clarifying question PASSES — the AI is correctly identifying ambiguity rather than guessing wrong.

Input (user's request + context): {{actual_input}}
Output to evaluate: {{actual_output}}

Return JSON:
{
  "criterion": "Relevance",
  "inferred_user_intent": "what the user was actually trying to accomplish",
  "reasoning": "does the output address that intent?",
  "verdict": "PASS" or "FAIL",
  "confidence": "high" or "medium" or "low"
}
```

---

### For Completeness / Coverage

```
You are evaluating whether an AI output covers all critical elements it should.

## System Context
{{system_description}}

## Required Elements for This Task Type
{{required_elements_list}}

## Criterion: Completeness
The output must address all required elements listed above. Missing any critical element is a FAIL — even if what's present is high quality.

## What PASS looks like
All required elements are present. They don't need to be exhaustive, but they can't be absent.

## What FAIL looks like
One or more required elements are entirely missing. Partial coverage of an element (e.g., mentioned but not addressed) counts as a fail for that element.

Input: {{actual_input}}
Output to evaluate: {{actual_output}}

For each required element, note whether it's present, partially present, or absent.

Return JSON:
{
  "criterion": "Completeness",
  "element_coverage": [
    {"element": "element name", "status": "present" or "partial" or "absent"}
  ],
  "verdict": "PASS" or "FAIL",
  "confidence": "high" or "medium" or "low",
  "reasoning": "which element(s) drove the verdict"
}
```

---

### For Tone / Brand Voice

```
You are evaluating whether an AI output matches the expected tone and voice.

## System Context
{{system_description}}

## Tone Profile
{{tone_description}}
Examples of in-tone language: {{tone_examples}}
Examples of out-of-tone language: {{anti_tone_examples}}

## Criterion: Tone & Voice Alignment
The output's language, register, and style must match the tone profile above throughout the entire response — not just in isolated phrases.

## What PASS looks like
The response sounds like it was written by someone who embodies the tone profile. The register is consistent from start to finish.

## What FAIL looks like
The response shifts register mid-way. It uses language that would feel out-of-place to the target user. It's either too formal, too casual, too corporate, or too hedged for this product.

Input: {{actual_input}}
Output to evaluate: {{actual_output}}

Return JSON:
{
  "criterion": "Tone Alignment",
  "tone_observations": "specific phrases that are in-tone or out-of-tone",
  "verdict": "PASS" or "FAIL",
  "confidence": "high" or "medium" or "low"
}
```

---

## Bias Mitigation Checklist

Before deploying any judge, check for these known LLM judge biases:

| Bias | What It Is | Mitigation |
|------|-----------|------------|
| Position bias | Judge favors the first response in pairwise comparison | Run pairwise comparisons twice with order swapped; only count agreement |
| Verbosity bias | Judge favors longer responses | Explicitly penalize unnecessary length in your rubric |
| Self-enhancement bias | Same model family favors outputs from that family | Use a different model as judge when possible |
| Sycophancy | Judge agrees with confident-sounding outputs even if wrong | Ask for counterarguments before final verdict |
| Halo effect | One strong dimension inflates others | One criterion per judge call — always |

---

## Calibration Protocol

Before using any judge in production:

1. **Label 50-100 examples by hand** — a domain expert makes binary PASS/FAIL calls
2. **Run the judge on the same examples**
3. **Compute Cohen's Kappa**:
   - Kappa < 0.4: Criteria are unclear or judge prompt is miscalibrated. Revise.
   - Kappa 0.4-0.6: Acceptable for most use cases
   - Kappa 0.7+: Target for high-stakes use cases
4. **Hold out 25% as a test set** — don't use for prompt iteration
5. **Monitor for evaluator drift** — resample and re-annotate monthly for live systems

---

## JSON Schema for Eval Suite

For integration into eval harnesses, output eval suites in this format:

```json
{
  "product_name": "string",
  "feature": "string",
  "generated_date": "ISO date",
  "categories": [
    {
      "id": "category-slug",
      "name": "Category Display Name",
      "definition": "one-sentence definition",
      "principle": "why this matters for this specific product",
      "pass_condition": "specific pass criteria",
      "fail_condition": "specific fail criteria",
      "grading_method": "deterministic | llm-as-judge | human",
      "eval_type": "capability | regression",
      "test_cases": [
        {
          "id": "tc-001",
          "name": "Descriptive name",
          "input": "the prompt or scenario",
          "output_to_evaluate": "the response being graded",
          "verdict": "PASS | FAIL",
          "reasoning": "why this verdict"
        }
      ],
      "judge_prompt": "the LLM-as-judge prompt string (if applicable)",
      "assertion_logic": "code or regex description (if deterministic)"
    }
  ],
  "cadence": {
    "ci": ["category-slugs"],
    "weekly_sample": ["category-slugs"],
    "human_review": ["category-slugs"]
  },
  "golden_dataset_guidance": {
    "target_sample_size": 200,
    "failure_rate_target": "25-50%",
    "sourcing_priority": ["production traces", "bug reports", "synthetic for gaps"]
  }
}
```
