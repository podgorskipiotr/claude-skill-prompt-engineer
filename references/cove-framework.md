# Chain-of-Verification (CoVe) Framework

Apply this framework to any prompt requiring factual accuracy, data synthesis, research-based outputs, or where hallucination risk is significant (named entities, dates, metrics, causal claims).

---

## When to Apply CoVe

**Mandatory triggers:**
- Market research / competitive analysis
- Historical facts, timelines, statistics
- Named entities: people, companies, products, regulations
- Quantitative claims: revenue figures, user counts, growth rates
- Causal statements: "X caused Y", "Z led to..."
- Legal, regulatory, or compliance outputs

**Not needed for:**
- Creative writing
- Open-ended brainstorming
- Style rewrites
- Format transformations of already-provided data

---

## The 4-Stage CoVe Architecture

### Stage 1: Draft
Generate a baseline response. This is the "first pass" — expected to contain some hallucinations or imprecisions.

### Stage 2: Verification Planning
Identify every **Atomic Claim** in the draft. An Atomic Claim is the smallest independently verifiable fact unit.

**Atomic Claim examples:**
- ❌ Too broad: "Audioteka is an established company in the audiobook space"
- ✅ Atomic: "Audioteka was founded in 2008"
- ✅ Atomic: "Audioteka operates in 5 European markets"
- ✅ Atomic: "Audible entered the Polish market in Q1 2025"

For each atomic claim, generate an **open-ended verification question** (not Yes/No):
- ❌ "Was Audioteka founded in 2008?" (binary — model defaults to confirming the draft)
- ✅ "What year was Audioteka founded, and what was the founding context?"

### Stage 3: Execution (Isolated Verification)
Answer each verification question **independently**, explicitly ignoring the draft. This prevents confirmation bias — the single biggest failure mode in LLM self-verification.

The prompt must explicitly say: `Answer each question below as if you have not seen the draft above. Do not reference or validate the draft. Treat each as a standalone research question.`

### Stage 4: Final Synthesis
Compare draft claims against verified answers. Resolve discrepancies by:
1. Replacing unverified claims with verified facts
2. Removing claims that could not be verified
3. Flagging residual uncertainty with explicit hedging

---

## Full CoVe Prompt Template

Use this template when optimizing a user's research/factual prompt:

```xml
<system>
You are a [ROLE] producing a [OUTPUT TYPE] for [AUDIENCE]. Your output must be factually verified and free of hallucinated claims. Follow the 4-stage verification protocol below exactly.
</system>

<task>
[CORE TASK DESCRIPTION]
</task>

<constraints>
- Executive register: no filler, high information density
- Tables preferred over prose for comparisons
- No claims without a verifiable basis
- Flag uncertainty explicitly with: [UNVERIFIED: ...]
</constraints>

<output_format>
Follow these exact stages in your response:

<draft>
Generate your baseline response to the task. Do not self-censor — write your best first-pass answer.
</draft>

<verification_plan>
Review your draft. Identify every Atomic Claim: specific dates, names, metrics, causal links, existence claims.

For each atomic claim, write one open-ended verification question. Format:
1. Claim: "[exact claim from draft]"
   Verification Q: "[open-ended question that independently tests this claim]"

Use Chain-of-Thought to reason about why each question is the right test for that claim.
</verification_plan>

<verification_execution>
Answer each verification question from the plan above as a standalone task.
CRITICAL: Do not reference your draft while answering. Treat each question independently.
If you are uncertain, state that explicitly rather than defaulting to the draft's answer.
</verification_execution>

<revised_output>
Compare your draft against your verified answers.
- Where verification confirms the draft: keep
- Where verification contradicts the draft: replace with verified fact
- Where verification is inconclusive: flag with [UNVERIFIED: describe uncertainty]
- Where a claim cannot be verified: remove it

Output the final, cleaned response in the format specified in <output_format_spec> below.
</revised_output>
</output_format>

<output_format_spec>
[SPECIFY EXACT FORMAT: table structure, sections, length limit, tone]
</output_format_spec>
```

---

## Simplified CoVe Template (for shorter tasks)

For medium-complexity factual tasks where full 4-stage overhead is too heavy:

```
[CORE TASK]

Before finalizing your answer:
1. List every specific fact you've stated (dates, names, numbers, causal claims)
2. For each, ask yourself: "What would an independent source say about this?"
3. If uncertain, remove the claim or flag it as [UNVERIFIED]
4. Output only your cleaned, verified response
```

---

## Atomic Fact Extraction Guide

| Claim Type | Example | Verification Question Style |
|-----------|---------|----------------------------|
| Date / founding | "Founded in 2010" | "What year was X founded and what's the source?" |
| Metric | "€50M revenue" | "What is X's reported annual revenue, and for which year?" |
| Causal | "Audible's entry caused Legimi to drop prices" | "What pricing changes did Legimi make, and when?" |
| Existence | "Audioteka has a German app" | "On what platforms is Audioteka available in Germany?" |
| Rank / superlative | "Largest audiobook platform in Poland" | "What is the market share ranking of audiobook platforms in Poland?" |
| Quote / attribution | "CEO said X" | "What has [CEO name] publicly stated about X?" |

---

## Integration Notes

**Anthropic alignment:**
- Use XML tags (`<draft>`, `<verification_plan>`, `<verification_execution>`, `<revised_output>`) to separate stages — this leverages Claude's strong instruction-following within tagged blocks
- Claude 4 models handle multi-stage verification protocols well when stages are clearly delimited

**Google / Vertex AI alignment:**
- Verification Planning stage uses Chain-of-Thought explicitly — this is Google's recommended approach for "generating reasoning steps before the answer"
- Open-ended verification questions mirror Google's guidance on avoiding leading questions in prompts

**Anti-patterns to avoid:**
- ❌ Binary verification questions ("Is this correct?") — model confirms its own hallucinations
- ❌ Asking the model to "check" the draft inline — creates circular reasoning
- ❌ Skipping Stage 3 isolation — single biggest source of CoVe failure
- ❌ Applying CoVe to creative or brainstorming tasks — unnecessary friction, no benefit
