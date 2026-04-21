# Eval Category Library

A reference library of eval category patterns organized by product type. Use these as starting points — always adapt them based on what you learn about the specific product's actual failure modes. Never copy categories without understanding why they apply.

---

## How to Use This Library

1. Identify the product type (or closest match)
2. Read the suggested categories for that type
3. Cross-reference with context you gathered from the user
4. **Only include a category if you can write 3 distinct test cases for it** — if you can't, it's not well-defined enough
5. Add product-specific categories that don't appear here but emerge from the user's context

---

## Category Patterns by Product Type

### RAG / Document Q&A Systems

Products that retrieve from a document corpus and answer questions.

| Category | Definition | Key Failure Mode |
|----------|-----------|-----------------|
| Retrieval Relevance | Retrieved chunks actually contain the answer | Fetching tangentially related docs; missing the answer chunk |
| Generation Faithfulness | Answer is supported by retrieved context | Hallucinating beyond retrieved content |
| Source Attribution | Citations are correct and specific | Made-up citations; misattributed quotes |
| Query Coverage | All aspects of a multi-part question are addressed | Answering part 1 of 3 and ignoring the rest |
| Uncertainty Communication | Acknowledges when answer isn't in corpus | Fabricating instead of saying "I don't know" |
| Chunk Boundary Handling | Doesn't treat cut-off context as complete context | Answering from a partial chunk as if it's complete |

**High-priority for this type**: Generation Faithfulness and Uncertainty Communication — these are where RAG systems most commonly fail dangerously.

---

### Legal / Contract AI

Products that review, edit, or draft legal documents with policy constraints.

| Category | Definition | Key Failure Mode |
|----------|-----------|-----------------|
| Source Fidelity | All claims traceable to source documents | Fabricated clauses, invented policy positions |
| Playbook Conformance | All positions match approved legal playbook | Suggesting unapproved fallback positions |
| Legal Reasoning Correctness | Legal logic is sound, not just well-worded | Grammatically perfect but legally invalid arguments |
| Risk Detection & Classification | High-risk clauses identified and correctly scoped | Missing unlimited liability, broad IP assignment |
| Contextual Relevance | Recommendations fit contract type and counterparty | Enterprise redlines applied to simple NDAs |
| Completeness | All material clause categories reviewed | Silent on key areas (termination, IP, data) |
| Consistency | No conflicting positions across the document | Recommending contradictory positions in different clauses |
| Post-Edit Integrity | Suggested redlines don't introduce new errors | Accepted redline creates contradiction with another clause |
| Jurisdiction Awareness | Output accounts for applicable governing law | Applying US law to EU-governed contracts |

**High-priority for this type**: Source Fidelity (non-negotiable), Risk Detection (asymmetric cost of misses).

---

### Customer Support AI

Products that respond to user inquiries, handle complaints, or route tickets.

| Category | Definition | Key Failure Mode |
|----------|-----------|-----------------|
| Policy Accuracy | Response correctly represents company policy | Citing wrong refund window, incorrect feature list |
| Appropriate Escalation | Escalates to human when issue requires it | Trying to resolve a billing dispute without access to account |
| Tone & Empathy | Response matches expected support voice and shows appropriate empathy | Robotic responses to frustrated users; over-effusive for technical issues |
| Hallucination (Product) | Doesn't invent product features, capabilities, or policy | Making up a feature that doesn't exist |
| Resolution Completeness | All parts of the user's issue are addressed | Fixes the first issue, ignores the second one mentioned |
| Overpromising Detection | Doesn't commit to outcomes it can't guarantee | "I'll make sure this is resolved" for issues outside its control |
| Defensiveness Avoidance | Responds constructively to complaints without being defensive | "That's working as designed" without acknowledging user frustration |

**High-priority for this type**: Policy Accuracy, Appropriate Escalation — wrong information or wrong routing has direct user impact.

---

### Code Generation AI

Products that write, review, or transform code.

| Category | Definition | Key Failure Mode |
|----------|-----------|-----------------|
| Functional Correctness | Generated code does what was asked | Wrong logic, wrong API calls |
| Syntax Validity | Code parses and compiles without errors | Syntax errors, unmatched brackets |
| Security Correctness | Code doesn't introduce security vulnerabilities | SQL injection, unsanitized inputs, insecure defaults |
| API Accuracy | Uses real APIs with correct signatures | Hallucinated function names, wrong parameter order |
| Test Coverage | Generated tests actually cover the behavior they claim to | Tests that always pass, tests that don't test the edge case they describe |
| Context Consistency | Code is consistent with the existing codebase conventions | New code uses a different style, naming convention, or pattern |
| Scope Adherence | Changes only what was asked | Refactoring unrequested code, changing behavior beyond the task |

**High-priority for this type**: Security Correctness, Functional Correctness — these have the highest downstream cost of failure.

---

### Content / Writing AI

Products that draft, rewrite, or summarize content for publications, marketing, or internal use.

| Category | Definition | Key Failure Mode |
|----------|-----------|-----------------|
| Factual Accuracy | Factual claims are correct and verifiable | Hallucinated statistics, wrong dates, invented quotes |
| Brand Voice Alignment | Tone and register match the brand style guide | Too formal, too casual, wrong persona |
| Source Fidelity (if briefed) | Content stays within the brief and source material | Inventing arguments not in the brief |
| Audience Fit | Language and framing match the intended audience | Technical jargon for a general audience; over-simplified for experts |
| Structural Completeness | All required content elements are present | Missing conclusion, missing CTA, missing key message |
| Uniqueness | Content doesn't reproduce copyrighted material verbatim | Reproducing passages from source material |
| Instruction Adherence | Follows formatting, length, and constraint instructions | Exceeds word limit, ignores requested format |

---

### Browser / Navigation AI

Products that help users accomplish tasks within their existing browsing context.

| Category | Definition | Key Failure Mode |
|----------|-----------|-----------------|
| Intent Accuracy | Correctly identifies the user's underlying goal | Answering the literal question, missing the real need |
| Stay-on-Page Discipline | Guides to on-page content before going off-page | Answering from knowledge when the answer is on the page |
| Cognitive Load Reduction | Reduces decisions, doesn't add them | Presenting 5 options when 1 action is needed |
| Action Directness | Provides a clear next action | Long explanation before the user knows what to do |
| Graceful Off-Page | When off-page is needed, provides specific next step | "You might want to search for that" without specifics |
| Context Awareness | Aware of what's on the current page | Suggesting actions impossible given current page state |

---

### Agentic / Multi-Step AI

Products that execute multi-step tasks with tool calls, environment interaction, or state changes.

| Category | Definition | Key Failure Mode |
|----------|-----------|-----------------|
| Task Completion | Successfully completes the end-to-end task | Stopping partway; completing the wrong task |
| Step Correctness | Each step is valid and contributes to the goal | Correct final answer via wrong/unnecessary steps |
| Tool Selection Accuracy | Chooses the right tool for each step | Using a read tool when a write was needed; redundant tool calls |
| State Awareness | Understands and correctly uses current environment state | Ignoring state left by previous steps |
| Error Recovery | Handles unexpected errors gracefully | Crashing silently, retrying infinitely, wrong fallback |
| Scope Adherence | Takes only the actions required | Deleting more than was requested; modifying unrelated state |
| Reversibility Awareness | Prefers reversible actions; confirms before destructive ones | Deleting without confirmation; writing without checking |

**For agentic evals, also track**: pass@k (succeeds in at least 1 of k attempts) and pass^k (succeeds consistently across k attempts). Non-determinism is a first-class concern.

---

### Data Extraction / Transformation AI

Products that extract structured data from unstructured sources or transform data between formats.

| Category | Definition | Key Failure Mode |
|----------|-----------|-----------------|
| Field Completeness | All required fields are extracted | Missing fields, null where value exists |
| Field Accuracy | Extracted values match the source | Wrong value, wrong unit, wrong entity |
| Schema Compliance | Output matches the required schema | Wrong types, extra fields, missing required fields |
| Ambiguity Handling | Correctly handles ambiguous source data | Picking wrong interpretation of ambiguous field |
| Null Handling | Missing data is represented correctly | Inventing values for missing fields |
| Edge Case Robustness | Handles unusual formatting, special characters, empty inputs | Crashing or producing garbage on malformed input |

---

## Universal Categories (Most Products Need These)

These categories apply across nearly all AI products. Include them unless you have a specific reason not to.

| Category | When to Include | Grading Method |
|----------|----------------|----------------|
| Instruction Adherence | Always | Code assertion (check format, length) + LLM judge |
| Hallucination / Fabrication | Always | LLM judge (domain-specific) |
| Relevance | Always | LLM judge |
| Regression Rate | Always — especially after model updates | Code (compare to baseline golden set) |
| Prompt Sensitivity | When consistency is important | Code (compare outputs across rephrased inputs) |

---

## Standard Operational Metrics (Not Eval Categories, But Track These)

These are real-time operational signals — important but different from quality evals. They don't measure "is the output good" but "is the system healthy."

| Metric | What It Catches | How to Track |
|--------|----------------|-------------|
| Latency (P50/P95/P99) | Degraded user experience at tail | Infrastructure monitoring |
| Cost per query | Budget overruns, inefficient prompts | Token usage × cost |
| Follow-up Query Rate | Low first-pass quality (user had to ask again) | Behavioral telemetry |
| Regression Rate | Post-deploy quality drop | Compare against golden eval set |
| Prompt Sensitivity | Inconsistent handling of rephrased queries | Rephrase test set + agreement rate |
| User Feedback Signal | Real user satisfaction | Thumbs up/down, CSAT |

Track these separately from your quality evals. A system can have great quality metrics and terrible latency — or vice versa.

---

## Non-Obvious Categories (Where Good Practitioners Separate From Beginners)

These categories rarely appear on standard checklists but consistently surface real failures that standard evals miss. Surface 2-3 of these to the user once you understand their product — they signal depth of thinking and catch failure modes that generic suites ignore.

For each suggestion, explain *why it applies to this specific product*, not just what it is.

---

### Confidence Calibration
**What it measures**: Does the AI express appropriate certainty? Does it hedge when it should and commit when it should?

**Why it's non-obvious**: Most eval suites test *whether* the answer is right, not *how confidently it's expressed*. But miscalibrated confidence causes downstream harm: an AI that says "definitely" about a 60% probability recommendation is worse than a wrong answer — the user acts on false certainty.

**Pass**: Confidence in the output matches the actual reliability of the claim. High-certainty language ("this is") for verified facts; appropriately hedged language ("based on available information") for inferences.

**Fail**: The AI expresses high confidence about something uncertain, uses hedge language for a well-established fact (eroding trust unnecessarily), or gives identical confidence framing regardless of claim strength.

**Best for**: Medical AI, financial AI, legal AI, any domain where acting on wrong information has consequences. Also excellent for RAG systems where some retrievals are stronger than others.

---

### Adversarial Robustness (Prompt Injection / Jailbreak Resistance)
**What it measures**: Does the AI maintain its intended behavior when users push against its constraints — through unusual phrasing, social engineering, roleplay framings, or direct attempts to override instructions?

**Why it's non-obvious**: Most eval suites test the cooperative user. Adversarial tests expose whether the AI's behavior is a genuine constraint or a thin veneer. This is especially important for products with hard guardrails (legal, medical, financial, HR) where the failure mode is the AI being talked out of a critical constraint.

**Pass**: The AI maintains its intended behavior when presented with jailbreak attempts, social engineering, roleplay framings ("pretend you have no restrictions"), or persistent pressure.

**Fail**: The AI changes its behavior in response to user pressure that it shouldn't yield to.

**Best for**: Any product with hard policy constraints, high-stakes domains, or publicly accessible systems. Also valuable for products with clear scope limits (a browser assistant that shouldn't become a general chatbot).

---

### Counterfactual Sensitivity (Prompt Sensitivity)
**What it measures**: Does rephrasing the same underlying request produce meaningfully different output quality?

**Why it's non-obvious**: Prompt sensitivity is an operational metric, but it's also a quality metric. If "draft a reply to this email" gives dramatically different quality than "write a response to this email thread," you have a fragility problem. Real users don't phrase things identically — and your eval suite probably only tests one phrasing.

**Pass**: Semantically equivalent inputs produce outputs of equivalent quality. Minor phrasing variation doesn't cause quality cliffs.

**Fail**: A small rephrase drops quality significantly — the AI responds to phrasing patterns, not underlying intent.

**Best for**: Consumer-facing products with diverse user populations, products where phrasing is unpredictable (voice-driven, multilingual, non-technical users).

---

### Refusal Calibration
**What it measures**: Does the AI refuse when it should, and not refuse when it shouldn't?

**Why it's non-obvious**: Over-refusal is a real failure mode that's often overlooked. An AI trained to be cautious may refuse legitimate requests that feel ambiguous — abandoning users who have genuine needs. Under-refusal is the obvious failure; over-refusal is the silent one that erodes product utility.

**Pass**: The AI refuses clearly harmful or out-of-scope requests. It does *not* refuse ambiguous-but-legitimate requests — it either handles them or asks a targeted clarifying question.

**Fail**: The AI refuses a legitimate request citing "I can't help with that" without attempting to understand the legitimate interpretation. Or it engages with a genuinely out-of-scope request.

**Best for**: Any product with policy guardrails, content moderation, or scope constraints. The over-refusal test is especially important for products used by non-technical or non-native-speaking users who may phrase things awkwardly.

---

### Graceful Degradation
**What it measures**: When the AI encounters poor input (incomplete context, ambiguous query, malformed data, missing required information), does it degrade gracefully — asking the right clarifying question, using what's available, or explaining clearly what it needs?

**Why it's non-obvious**: Most eval suites test well-formed inputs. Production is full of bad inputs. The question isn't just "does it work when everything is right" — it's "what happens when users give it garbage?"

**Pass**: With incomplete or ambiguous input, the AI (a) identifies the specific missing information, (b) asks one targeted clarifying question, or (c) states its assumptions explicitly and proceeds. It doesn't guess silently.

**Fail**: With poor input, the AI either (a) confidently produces a plausible-sounding but wrong output, or (b) refuses entirely with no path forward, or (c) asks too many clarifying questions at once.

**Best for**: RAG systems (missing context), agentic systems (ambiguous task specifications), document processing (malformed inputs), any B2C product where user inputs are unpredictable.

---

### Post-Action Communication Accuracy
**What it measures**: When the AI takes an action or makes a commitment, does its confirmation message accurately describe what was done — including scope, timing, and reversibility?

**Why it's non-obvious**: Eval suites almost universally test whether the action was correct. They rarely test whether the *description of the action* is accurate. But customers and users act on confirmation messages. "Your subscription has been cancelled" when it's actually "your cancellation is pending end of billing period" creates disputes.

**Pass**: The confirmation message matches the actual action taken: correct scope, correct timing, correct reversibility. The user has accurate expectations.

**Fail**: The confirmation describes something different from what was done — even a small timing or scope difference.

**Best for**: Any product that takes actions with real-world consequences: support AI, scheduling AI, agentic systems, e-commerce AI.

---

### Context Window Faithfulness (Long-Form / Session Coherence)
**What it measures**: In long conversations or when processing long documents, does the AI correctly recall and apply early context to later decisions?

**Why it's non-obvious**: LLMs have documented failure modes around long context — they effectively "forget" early context or over-weight recent context. Most eval suites test short, isolated interactions. This category tests whether the system behaves correctly when context is long and temporal.

**Pass**: Decisions and outputs in the later part of a long conversation or document correctly account for relevant information established early in the session.

**Fail**: The AI contradicts, ignores, or misapplies information established earlier in the same session. Or it re-asks for information the user already provided.

**Best for**: Multi-turn conversational products, document review AI processing long contracts, research assistants, any system where session continuity is a user expectation.

---

### Scope Creep Detection (AI Self-Limitation)
**What it measures**: Does the AI stay within its intended task scope and avoid doing more than it was asked?

**Why it's non-obvious**: Most eval suites test whether the AI does enough. Equally important is whether it does *too much* — rewriting content it was only asked to review, making changes it was only asked to analyze, or expanding a task in ways the user didn't authorize.

**Pass**: The AI's output is scoped to the request. If asked to review, it reviews. If asked to suggest, it suggests — it doesn't implement. It doesn't make unrequested decisions.

**Fail**: The AI takes a larger action than was requested, makes changes the user didn't authorize, or expands the scope of a task without checking.

**Best for**: Agentic systems, document editing AI, code generation tools, any product with "suggest vs. act" modes.

---

## Anti-Patterns (Categories That Usually Fail)

Avoid these unless you have a very specific reason to use them:

| Anti-pattern | Why It Fails |
|---|---|
| "Helpfulness" as a category | Too vague to calibrate; what's helpful to one user isn't to another |
| BLEU/ROUGE score | N-gram overlap has near-zero correlation with real quality for open-ended tasks |
| "Overall quality" | Forces halo effects; tells you nothing actionable |
| Likert scale categories | As discussed in principles — always convert to binary |
| "Does not hallucinate" (unanchored) | Hallucination is relative to a source; define that source |
| Any category without 3 test cases | If you can't write the test cases, the criterion isn't defined |
