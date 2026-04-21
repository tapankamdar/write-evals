---
description: Generate a principled, product-specific AI eval suite. Opens with evaluation philosophy, gathers context about the product and feature, then produces eval categories with test cases, pass/fail criteria, and LLM-as-judge prompts. Use when evaluating any AI feature or product.
argument-hint: "[optional: product name or feature description]"
allowed-tools: Read, Write
---

Load the `write-evals` skill from `skills/write-evals/SKILL.md` and follow it exactly.

If the user provided an argument, treat it as the initial product/feature description and begin the session with that context already known — skip asking what the product does and proceed to the docs question.

If no argument was provided, begin from the opening philosophy as instructed in the skill.
