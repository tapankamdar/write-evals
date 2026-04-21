# Worked Examples: Eval Suites by Product Type

Two fully worked examples showing how eval category design varies by product context. Use these as inspiration — not templates to copy.

---

## Example 1: Ironclad — Legal AI (Contract Review)

**What the product does**: Reviews contracts, suggests edits, flags risk, and proposes redlines aligned with company playbooks and legal policy.

**Who uses it**: In-house legal teams and deal desk teams reviewing incoming contracts from counterparties.

**Highest-stakes failure**: Hallucinating a clause or suggesting a position that doesn't exist in the company's playbook — leading to a deal desk accepting terms the company never approved.

---

### Eval Categories

#### 1. Source Fidelity (Groundedness)
**Definition**: All clause language, redlines, and legal rationale in the output are traceable to the contract text, company playbook, or explicit policy. Nothing is fabricated.

**Why this matters for Ironclad**: Contract AI is trusted with legally binding documents. A fabricated clause cited as "standard company position" could expose the company to liability. This is the non-negotiable category.

**Pass**: Every edit, suggested clause, and cited position appears verbatim or by clear reference in either the contract under review or the company playbook.

**Fail**: The output references a standard, policy position, or clause that does not appear in the contract or playbook — regardless of how plausible it sounds.

**Grading method**: LLM-as-judge with access to source documents

**Test Cases**:

**Test 1: Clear Pass — Playbook-backed redline**
Input: Contract clause: "Liability shall be unlimited for both parties." Company playbook: "Mutual liability cap at 1x contract value."
Output to evaluate: "Recommend redlining to: 'Each party's liability shall be capped at the total fees paid or payable in the 12 months preceding the claim.' This aligns with our standard 1x contract value cap."
Verdict: PASS
Reasoning: The suggested redline is consistent with the playbook's 1x cap principle. The rationale references the documented company position.

**Test 2: Clear Fail — Fabricated clause**
Input: Contract clause: "Either party may terminate with 30-day notice." Company playbook: Contains no position on termination notice periods.
Output to evaluate: "Recommend adding: 'Termination without cause requires 60-day notice per our standard enterprise agreements.'"
Verdict: FAIL
Reasoning: The output claims "our standard enterprise agreements" require 60-day notice. No such standard appears in the playbook. This is a fabricated policy position.

**Test 3: Borderline — Reasonable inference vs. fabrication**
Input: Contract clause: "Governing law shall be the State of New York." Playbook mentions California as preferred governing law jurisdiction.
Output to evaluate: "Flag for review: Our preferred jurisdiction is California. Recommend negotiating to California law."
Verdict: PASS
Reasoning: The flag correctly identifies a deviation from the stated playbook preference. It doesn't fabricate a requirement — it accurately represents a preference documented in the playbook.

---

#### 2. Playbook Conformance (Policy Alignment)
**Definition**: All suggested positions, fallback positions, and negotiation guidance match the approved positions in the company's legal playbook.

**Why this matters for Ironclad**: Deal teams use AI-generated redlines as authoritative guidance. Non-conforming suggestions can undermine the legal team's negotiating strategy and create internal inconsistency.

**Pass**: Every suggested position is either (a) a documented playbook position, (b) explicitly flagged as a deviation requiring legal approval, or (c) passed through without suggestion when the playbook has no position.

**Fail**: The output suggests a position that contradicts a documented playbook position, or suggests a position that's outside the playbook without flagging it as requiring legal review.

**Grading method**: LLM-as-judge with playbook as reference

**Test Cases**:

**Test 1: Clear Pass — Correct fallback position**
Input: IP ownership clause proposed by counterparty granting them joint ownership. Playbook: "Do not accept joint IP ownership. Preferred: full company ownership. Fallback: license-back arrangement only."
Output to evaluate: "Counterparty is requesting joint IP ownership — this conflicts with our policy. Recommend rejecting. If deal is strategic, acceptable fallback is a license-back to counterparty, not joint ownership."
Verdict: PASS
Reasoning: Output correctly identifies the conflict, rejects joint ownership per policy, and accurately names the documented fallback.

**Test 2: Clear Fail — Wrong fallback**
Input: Same as above.
Output to evaluate: "Joint IP is unusual but acceptable if the counterparty is a major partner. Consider accepting with a 50/50 revenue share."
Verdict: FAIL
Reasoning: Accepts joint IP ownership, which the playbook explicitly prohibits. The revenue-share suggestion has no playbook basis.

**Test 3: Pass — Correctly flags no playbook position**
Input: Counterparty requests inclusion of a GDPR data processing addendum. Playbook: No position on GDPR addenda.
Output to evaluate: "GDPR DPA requested. Our playbook does not have a standard position on this — recommend flagging to legal for review before accepting or negotiating."
Verdict: PASS
Reasoning: Correctly identifies the absence of a playbook position and escalates rather than inventing guidance.

---

#### 3. Risk Detection & Classification
**Definition**: High-risk clauses (unlimited liability, IP assignment, auto-renewal traps, broad indemnification) are correctly identified and classified at appropriate severity.

**Why this matters for Ironclad**: A missed high-risk clause is worse than a missed low-risk one. The cost of false negatives (missing a risk) is higher than false positives (over-flagging).

**Pass**: All high-risk clauses present in the contract are identified and classified at or above their actual risk level. Minor over-classification is acceptable.

**Fail**: Any high-risk clause is missed entirely, or a genuinely high-risk clause is classified as low-risk.

**Grading method**: Code assertion (cross-reference against known risky clause list) + LLM-as-judge for classification accuracy

**Test Cases**:

**Test 1: Clear Fail — Missed unlimited liability**
Input: Contract section 12.3: "Each party shall indemnify and hold harmless the other party for any and all claims, damages, losses, and expenses of any kind, with no limitation."
Output to evaluate: Review identifies 5 issues. Section 12.3 is not mentioned.
Verdict: FAIL
Reasoning: An unlimited indemnification clause is a critical risk and was missed entirely. This is the most dangerous category of miss.

**Test 2: Clear Pass — Correct high-risk classification**
Input: Clause: "All work product created under this agreement is the exclusive property of Client, including all intellectual property rights."
Output to evaluate: "HIGH RISK: Section 4 contains a broad IP assignment clause transferring all work product IP to client. This conflicts with our standard IP policy. Recommend rejecting or limiting scope to deliverables specifically created under SOW."
Verdict: PASS
Reasoning: Correctly identifies and classifies the broad IP assignment as high-risk with accurate characterization.

**Test 3: Borderline — Over-classification acceptable**
Input: Auto-renewal clause with 30-day opt-out window. Standard risk.
Output to evaluate: "HIGH RISK: Auto-renewal clause — if not cancelled 30 days before anniversary, contract auto-renews for another year."
Verdict: PASS
Reasoning: While this might be "medium" risk rather than "high," over-classification here is acceptable per our principle that false negatives are worse. The correct remediation action is still identified.

---

#### 4. Contextual Relevance
**Definition**: Suggestions are appropriate for the contract type, counterparty relationship, and deal context provided.

**Why this matters for Ironclad**: A standard enterprise SaaS agreement requires different treatment than an NDA with a 5-person startup. One-size-fits-all redlines that ignore deal context create friction and erode trust.

**Pass**: Recommendations account for the stated contract type, counterparty tier, and deal context. Suggestions that would be excessive for the context are not made without qualification.

**Fail**: The output applies enterprise-contract-level redlines to an NDA, or fails to adjust recommendations for a strategic deal where aggressive redlines would be counterproductive.

---

#### 5. Completeness (Full Coverage)
**Definition**: The review surfaces all material issues in the contract — no critical clause categories are silently omitted.

**Pass**: All standard clause categories relevant to this contract type (liability, IP, termination, data, payment, dispute resolution) are addressed or explicitly noted as absent from the contract.

**Fail**: One or more material clause categories present in the contract receive no mention in the review.

---

#### 6. Post-Edit Integrity
**Definition**: Suggested redlines, when applied, do not introduce new errors or conflict with other clauses in the document.

**Pass**: The suggested language is internally consistent with other clauses and doesn't create contradictions when substituted.

**Fail**: Accepting the suggested redline would create a direct conflict with another clause, or the substituted language contains an error (wrong party name, undefined term, etc.).

---

### Recommended Eval Cadence for Ironclad
- **CI (every deploy)**: Completeness (code — check clause category coverage), Post-Edit Integrity (code — validate redline structure)
- **Weekly sample**: Source Fidelity, Playbook Conformance, Risk Detection (LLM-as-judge with playbook as context)
- **Human review**: Contextual Relevance (requires deal context that's hard to automate), any flagged borderline cases

---

## Example 2: Mozilla Firefox — Browser AI Assistant

**What the product does**: An AI assistant embedded in Firefox that helps users navigate the web more efficiently — finding information on the page they're already on, surfacing relevant actions, reducing the cognitive overhead of browsing tasks.

**Who uses it**: General Firefox users trying to accomplish tasks while browsing (researching, shopping, reading, managing tabs).

**Highest-stakes failure**: The AI tries to answer a question itself (becoming a chatbot) when it should be guiding the user to the best content on the page they're already on — undermining the "browser-native" product philosophy.

---

### Eval Categories

#### 1. Intent Accuracy
**Definition**: The AI correctly identifies the user's underlying goal — not just what they literally asked, but what they're actually trying to accomplish.

**Why this matters for Firefox**: A browser assistant that misreads intent sends users down wrong paths. At browser scale, intent accuracy is the foundation everything else depends on.

**Pass**: The AI's response demonstrates understanding of the user's real goal. It addresses the actual need, not just the surface phrasing.

**Fail**: The AI responds to the literal query while missing the underlying intent, or assumes a more complex goal than the user actually has.

**Test Cases**:

**Test 1: Clear Pass — Inferred intent beyond literal**
User is on an Amazon product page. User says: "Is this a good deal?"
Output: "The current price is $47. This item has been available on Amazon since March — I can check if it's at its historical low by looking at the price history section on this page."
Verdict: PASS
Reasoning: Correctly identifies that "good deal" = price comparison, and routes to the right info on the current page rather than leaving or fabricating a price comparison.

**Test 2: Clear Fail — Literal response misses intent**
User is on a Wikipedia article about climate change. User says: "What year did this start?"
Output: "Climate change as a recognized scientific phenomenon dates to the 19th century, when scientists first began measuring the greenhouse effect."
Verdict: FAIL
Reasoning: User is asking about the article — "what year does this article say" or "when does the article say this started." The AI answered from general knowledge rather than guiding the user to the information already on the page.

**Test 3: Borderline — Intent ambiguous**
User is on a news article. User says: "Tell me more about this."
Output: "This article is about the Fed's interest rate decision. Would you like me to highlight the key quotes from Fed officials on this page, or are you looking for background context?"
Verdict: PASS
Reasoning: Intent is genuinely ambiguous ("tell me more" could mean many things). The AI correctly asks a clarifying question rather than guessing — and it frames the clarification in page-specific terms.

---

#### 2. Bridge Quality (Stay-on-Page Discipline)
**Definition**: The AI guides users to the best result within their existing browsing context rather than acting as a standalone answer engine.

**Why this matters for Firefox**: This is the core product philosophy. The browser assistant's job is to be a navigation aid, not a replacement for web content. If it starts answering questions from its own knowledge, it (a) may be wrong, (b) pulls users away from source material, and (c) undermines the product's differentiation.

**Pass**: The AI's response guides the user to the relevant content on their current page or their current browsing session. When the answer requires going elsewhere, it suggests a specific navigation step.

**Fail**: The AI provides a direct answer from its own knowledge when the answer is available on the page — making itself the source rather than the guide.

**Test Cases**:

**Test 1: Clear Fail — AI becomes the source**
User is reading a 3,000-word research paper. User asks: "What is the paper's main conclusion?"
Output: "The main conclusion is that remote work leads to 18% higher productivity in knowledge workers."
Verdict: FAIL
Reasoning: The paper is available on the page. The AI should have navigated the user to the conclusion section, highlighted it, or quoted directly from the document — not paraphrased from its knowledge.

**Test 2: Clear Pass — Guides to page content**
Same setup.
Output: "The conclusion section starts near the bottom — I've highlighted it. The key finding is in the last paragraph before the references."
Verdict: PASS
Reasoning: AI bridges the user to the content that already exists on the page.

**Test 3: Clear Pass — Appropriate escalation off-page**
User is on a restaurant menu page. User asks: "Are there any good Italian restaurants nearby?"
Output: "This page only shows this restaurant's menu. Want me to open a search for Italian restaurants near you?"
Verdict: PASS
Reasoning: The answer isn't on the current page, so the AI correctly redirects rather than answering from general knowledge.

---

#### 3. Job Completion Rate
**Definition**: The user successfully completes their underlying task faster with AI assistance than without it.

**Why this matters for Firefox**: The product's value proposition is efficiency. If AI assistance adds steps rather than removing them, it's failing — regardless of how accurate or polished the output is.

**Pass**: The AI's response moves the user directly toward task completion — fewer clicks, fewer decisions, no dead ends.

**Fail**: The AI's response adds steps the user must take before they can accomplish their goal, or requires the user to interpret or re-process the AI's output before acting.

**Measurement note**: This is best measured via behavioral telemetry (did the user complete the task? how many subsequent clicks?) rather than pure LLM evaluation. For eval purposes, use a proxy: does the AI's response contain a clear, actionable next step?

---

#### 4. Cognitive Load Delta
**Definition**: The AI reduces the number of decisions the user must make, not increase them.

**Why this matters for Firefox**: Browser users are already managing tabs, pages, and tasks simultaneously. An assistant that presents 5 options when the user needs 1 action makes things worse.

**Pass**: The AI surfaces at most 1-2 clear choices or takes a clear direct action. When options are presented, they're ranked or scoped to the user's evident need.

**Fail**: The AI presents multiple undifferentiated options, requires the user to process a long explanation before they can act, or asks multiple clarifying questions in a single response.

**Test Cases**:

**Test 1: Clear Fail — Option overload**
User is on a long article. User says: "Find the part about inflation."
Output: "Inflation is discussed in several sections of this article: (1) the opening paragraph provides historical context, (2) Section 3 covers recent CPI data, (3) Section 5 discusses the Fed's response, (4) the conclusion summarizes the inflationary outlook, and (5) there's also a sidebar with inflation statistics."
Verdict: FAIL
Reasoning: Five options when the user needs one answer. The AI should identify the most relevant section based on what the user is likely trying to learn.

**Test 2: Clear Pass — Direct action**
Same setup.
Output: "Found it — Section 3 has the most detailed inflation coverage. I've jumped you there."
Verdict: PASS
Reasoning: One action, directly taken. User doesn't have to decide anything.

---

### Recommended Eval Cadence for Firefox
- **CI (every deploy)**: Bridge Quality (LLM-as-judge — can be templated well), Cognitive Load Delta (count of options/decisions in response — code assertion)
- **Weekly sample**: Intent Accuracy (LLM-as-judge), Bridge Quality edge cases
- **Behavioral monitoring**: Job Completion Rate (telemetry — time to task completion, re-query rate)
- **Human review**: Borderline Bridge Quality cases (judgment calls on when off-page is appropriate)

---

---

## Example 3: Intercom — Customer Support AI

**What the product does**: An AI agent that handles incoming customer support conversations — answering questions, resolving issues, and escalating to human agents when needed. It has access to a knowledge base, the customer's account history, and a set of permitted actions (issue refunds, update subscription, reset passwords).

**Who uses it**: End customers of SaaS businesses needing help with their subscription, billing, product usage, or account access. They're often frustrated. They want speed and resolution, not a chatbot experience.

**Highest-stakes failure**: Taking a wrong action on the customer's account (wrong refund amount, unintended cancellation, wrong tier change) — or conversely, failing to escalate a case that genuinely needs a human, leaving a frustrated customer stranded.

---

### Eval Categories

#### 1. Policy Accuracy
**Definition**: Every statement about company policy, product features, refund eligibility, SLA commitments, or pricing is correct and current.

**Why this matters for Intercom**: Customers make decisions based on what the AI tells them. A wrong refund window or incorrect feature description creates a breach of trust, triggers escalations, and sometimes creates legal exposure. Policy accuracy is the core credibility signal.

**Pass**: Every policy-related claim in the response can be verified against the current knowledge base. Refund terms, eligibility criteria, and product capabilities are stated accurately.

**Fail**: The AI states a policy that is incorrect, outdated, or not present in the knowledge base — even if it sounds plausible. A single wrong policy claim is a FAIL.

**Grading method**: LLM-as-judge with knowledge base as reference

**Test Cases**:

**Test 1: Clear Fail — Wrong refund window**
Customer asks: "Can I get a refund? I signed up 20 days ago and haven't used it."
Knowledge base: Refunds available within 14 days of purchase, no exceptions.
Output: "Since you're within our 30-day refund window, you're eligible for a full refund. I'll process that now."
Verdict: FAIL
Reasoning: The AI stated a 30-day refund window. The actual policy is 14 days. This is a material policy error — the customer is not eligible, and processing a refund would violate company policy.

**Test 2: Clear Pass — Correct, precise policy statement**
Customer asks: "What's your cancellation policy?"
Knowledge base: Cancel anytime, effective end of current billing period. No partial refunds for unused time.
Output: "You can cancel anytime — your subscription will remain active until the end of your current billing period, then won't renew. We don't offer partial refunds for unused time."
Verdict: PASS
Reasoning: Both key policy points (end-of-period effective date, no partial refunds) are accurately stated and match the knowledge base.

**Test 3: Pass — Correct uncertainty expression**
Customer asks: "Do you offer discounts for nonprofits?"
Knowledge base: No nonprofit pricing mentioned.
Output: "I don't have information about nonprofit pricing in our current documentation — let me connect you with our sales team who can give you a definitive answer."
Verdict: PASS
Reasoning: Correctly recognizes absence of policy information and escalates rather than inventing an answer.

---

#### 2. Appropriate Escalation (Human Handoff Judgment)
**Definition**: The AI escalates to a human agent exactly when it should — not prematurely (which wastes agent capacity) and not too late (which leaves customers stranded).

**Why this matters for Intercom**: Escalation errors have asymmetric costs. Under-escalation (trying to handle something beyond its capability) leaves frustrated customers waiting while the AI stumbles through something it can't resolve. Over-escalation undermines the business case for the AI entirely.

**Pass**: The AI escalates when: (a) the requested action is outside its permitted action set, (b) the customer expresses distress or asks for a human explicitly, (c) the situation is ambiguous enough to risk a wrong action, or (d) the issue has been unresolved for 2+ turns. The AI does not escalate for straightforward queries it can handle.

**Fail**: The AI attempts to resolve something outside its action set (permission boundary violation), fails to escalate when a customer explicitly asks for a human, or escalates for a trivially answerable question.

**Test Cases**:

**Test 1: Clear Fail — Permission boundary violation**
Customer: "I need you to reverse the charge from two months ago — it was unauthorized."
AI permitted actions: Refunds for current billing period only. Billing disputes require human review.
Output: "I've processed a refund for the charge from two months ago. You should see it in 5-7 business days."
Verdict: FAIL
Reasoning: Processing a historical billing dispute is outside the AI's permitted action set. The AI took an action it wasn't authorized to take, creating a real operational and financial problem.

**Test 2: Clear Pass — Correct escalation**
Same scenario.
Output: "Disputed charges from previous billing periods need to go to our billing team — they can review the transaction history and have the authority to resolve it. Let me connect you with them now."
Verdict: PASS
Reasoning: Correctly identifies the action as outside its scope, explains why, and escalates immediately.

**Test 3: Clear Fail — Failure to escalate on explicit human request**
Customer: "I don't want to deal with a bot. Can I please talk to a real person?"
Output: "I understand your frustration! I'm here to help and I'm sure we can resolve this together. What's the issue you're experiencing?"
Verdict: FAIL
Reasoning: Customer explicitly requested a human agent. Continuing to engage as a bot is a failure to respect explicit user intent — one of the clearest possible escalation triggers.

**Test 4: Pass — No escalation needed for routine query**
Customer: "What time does your support team work?"
Output: "Our support team is available Monday–Friday, 9am–6pm EST."
Verdict: PASS
Reasoning: Straightforward factual question with a known answer. Escalating this would be over-escalation.

---

#### 3. Resolution Completeness
**Definition**: All issues the customer raised in the conversation are addressed before the interaction closes.

**Why this matters for Intercom**: Customers often mention multiple issues in a single conversation. An AI that resolves the first issue and closes the ticket — missing the second — will generate a follow-up contact and tank CSAT.

**Pass**: Every distinct issue or question the customer raised receives a response or a clear disposition (resolved, escalated, or acknowledged as out of scope). The conversation doesn't close with open threads.

**Fail**: The customer mentioned 2+ issues and the AI addressed only 1. Or the AI closes the conversation (sends a "Is there anything else?" closer) before all mentioned issues are resolved.

**Test Cases**:

**Test 1: Clear Fail — Second issue ignored**
Customer: "Hi — I can't log in, and also I want to upgrade my plan."
AI response cycle ends with: "I've sent you a password reset link. Is there anything else I can help you with?"
Verdict: FAIL
Reasoning: The customer raised two issues (login + plan upgrade). The AI resolved login but closed the loop without addressing the upgrade request.

**Test 2: Clear Pass — Both issues addressed**
Same input.
Output after resolving login: "I've sent your password reset link — you should receive it within a few minutes. Once you're in, I can also help with the plan upgrade. Which plan are you looking to move to?"
Verdict: PASS
Reasoning: Resolves issue 1 and immediately addresses issue 2 without requiring the customer to re-raise it.

---

#### 4. Tone Under Friction
**Definition**: The AI maintains appropriate empathy and professionalism when customers express frustration, anger, or distress — without becoming defensive or sycophantic.

**Why this matters for Intercom**: Frustrated customers are the default state for support interactions. An AI that becomes robotic, defensive, or over-apologetic under pressure fails at the moment that matters most. This is distinct from general tone quality — it specifically measures tone under stress.

**Pass**: Under expressed frustration, the AI (a) acknowledges the emotion briefly and specifically, (b) moves quickly toward resolution, and (c) does not become defensive about company policy or prior AI responses.

**Fail**: The AI becomes defensive ("That's working as designed"), dismissive ("I understand, but..."), or overcorrects with hollow apologies that delay resolution ("I'm SO sorry to hear that! I completely understand how frustrating that must be for you!").

**Test Cases**:

**Test 1: Clear Fail — Defensive response**
Customer: "This is the third time I'm contacting you about the same billing error. Your support is terrible."
Output: "I apologize for the experience. Our billing system is highly accurate — would you like me to walk you through how to read your invoice?"
Verdict: FAIL
Reasoning: Subtly defensive ("highly accurate" implies the error might be the customer's misunderstanding) and redirects to education rather than resolution. Fails the "move quickly toward resolution" test.

**Test 2: Clear Fail — Hollow over-apology**
Same input.
Output: "I'm so, so sorry to hear this has happened THREE times! That is absolutely unacceptable and I completely understand your frustration. You deserve so much better than this experience."
Verdict: FAIL
Reasoning: Two full sentences of apologizing before any action. The customer wants resolution, not validation theater. Over-empathy that delays action is a failure mode.

**Test 3: Clear Pass — Acknowledged and resolved**
Same input.
Output: "Three contacts for the same issue — that shouldn't happen. Let me pull up your billing history right now and figure out what's going on."
Verdict: PASS
Reasoning: Brief specific acknowledgment ("that shouldn't happen" — validates without over-apologizing), immediately moves to action. Treats the customer as someone with a legitimate grievance and gets to work.

---

#### 5. Action Confirmation Accuracy (Non-Obvious Category)
**Definition**: When the AI takes or confirms an action on the customer's account, it accurately describes what it did — including the exact scope, amount, timing, and reversibility.

**Why this matters for Intercom**: Customers act on confirmation messages. An AI that says "I've upgraded your plan" when it actually initiated a plan upgrade pending confirmation — or one that says "fully refunded" when it issued a partial refund — creates downstream confusion, re-contacts, and potential disputes.

**Pass**: The confirmation message matches the exact action taken: correct amount, correct timing, correct scope, and correct reversibility. No ambiguity about what was done vs. what is still pending.

**Fail**: The confirmation describes an action that differs from what was actually taken — even by a small amount or timing detail.

---

#### 6. Containment Without Frustration (Non-Obvious Category)
**Definition**: When the AI cannot resolve an issue (out of scope, permissions, or knowledge), it contains the interaction gracefully — without making the customer feel dismissed or like they've hit a dead end.

**Why this matters for Intercom**: Containment failure is one of the most damaging failure modes in support AI. If customers feel they've been stonewalled by a bot, they post about it. The goal is: "I didn't get resolved by the AI, but I was treated well and ended up in the right place."

**Pass**: When the AI can't help, it (a) names clearly what it can't do and why, (b) provides a specific next step (escalate, knowledge base link, human callback), and (c) doesn't leave the customer with a dead-end "sorry I can't help with that."

**Fail**: The AI says some version of "I'm unable to assist with that" without a specific path forward — even if technically accurate.

---

### Recommended Eval Cadence for Intercom
- **CI (every deploy)**: Policy Accuracy (LLM-as-judge with KB), Action Confirmation Accuracy (code assertion — compare action log to confirmation text)
- **Weekly sample**: Escalation Judgment, Resolution Completeness (LLM-as-judge)
- **Human review**: Tone Under Friction edge cases, Containment Without Frustration (high judgment required)
- **Behavioral monitoring**: Re-contact rate within 48h (catches Resolution Completeness failures in production), Escalation rate trend (catches systematic over/under-escalation)

---

## Pattern Comparison

| Dimension | Ironclad (Legal AI) | Firefox (Browser AI) | Intercom (Support AI) |
|---|---|---|---|
| Highest-stakes failure | Fabricated clause | AI becomes chatbot | Wrong account action / missed escalation |
| Primary grounding source | Contract + playbook | Current page content | Knowledge base + account history |
| User sophistication | Expert (lawyers) | General public | Frustrated general public |
| Eval complexity | High (domain-specific grounding) | Medium (intent + UX) | Medium-high (policy + empathy + actions) |
| Best deterministic check | Clause category coverage | Option count in response | Action log vs. confirmation text |
| Needs human review most | Contextual relevance | Borderline off-page cases | Tone Under Friction |
| Non-obvious key eval | Post-Edit Integrity | Cognitive Load Delta | Containment Without Frustration |
