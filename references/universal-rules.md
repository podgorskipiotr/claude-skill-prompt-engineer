# Universal Prompt Engineering Rules

Sources: Anthropic Claude 4 Best Practices, Google Vertex AI Prompt Design Strategies

---

## 1. The Direct Imperative Principle

**Rule:** Open every prompt with a clear, direct verb. Never use passive or indirect phrasing.

| Weak | Strong |
|------|--------|
| "Can you help me understand..." | "Explain..." |
| "I was wondering if you could..." | "Analyze..." |
| "It would be great if..." | "Generate..." |
| "Please try to..." | "Write..." |

---

## 2. Role / Persona Declaration

**Rule:** Declare the model's role in the first sentence. This anchors output style, vocabulary, and depth.

**Template:** `Act as a [specific role] with [relevant expertise/context].`

**Examples:**
- `Act as a senior M&A analyst specializing in European digital media.`
- `Act as a principal software engineer reviewing production-grade Python.`
- `Act as a CMO writing for a B2B SaaS audience at the VP level.`

**Claude 4 note:** Claude responds better to role declarations that specify *expertise depth* and *audience*, not just job titles.

---

## 3. Context Block

**Rule:** Always provide a context block before the instruction. Use XML tags for Claude.

```xml
<context>
Company: Audioteka — subscription audiobook platform, 5 markets (PL, DE, CZ, LT, SK)
Audience: C-level executives
Purpose: Strategic offsite preparation
Tone: Executive briefing — no fluff, high density
</context>

<task>
Analyze the competitive positioning of Audioteka vs Audible entering Poland.
</task>
```

**Google alignment:** "Provide clear context before instructions" — Vertex AI prompt design strategy #1.

---

## 4. Output Format Specification

**Rule:** Explicitly specify format. Never leave it to the model's discretion.

Specify all of:
- **Structure:** prose / bullets / table / JSON / markdown
- **Length:** word count range, or "no more than X sentences"
- **Sections:** exact headers you want
- **Tone:** formal / executive / technical / conversational

**Example:**
```
Output format:
- Executive summary: 3 sentences max
- Competitive matrix: markdown table with columns [Player, Pricing, Catalog, Moat]
- Strategic recommendation: numbered list, 3 items max
- Do NOT include background history or founding dates
```

---

## 5. Negative Constraints

**Rule:** Always add what NOT to do. Models without constraints drift toward generic outputs.

**Template additions:**
- `Do not include disclaimers or caveats.`
- `Do not use filler phrases like "It's worth noting that..."`
- `Do not repeat the question back to me.`
- `Avoid hedging language. State conclusions directly.`
- `Do not suggest consulting a professional — assume the reader is one.`

---

## 6. Few-Shot Examples

**Rule:** For non-obvious output formats, include 1–3 examples. Especially effective for:
- Custom table structures
- Specific writing register
- Multi-step reasoning outputs
- Edge case handling

**Template:**
```xml
<example>
Input: [sample input]
Output: [exact desired output]
</example>
```

**Note:** Even one high-quality example outperforms 3 paragraphs of description.

---

## 7. Chain-of-Thought (CoT) for Reasoning Tasks

**Rule:** For analysis, evaluation, or multi-step reasoning, explicitly instruct step-by-step thinking.

**Templates:**
- `Think step by step before writing your final answer.`
- `First, identify the key variables. Then, reason through each. Finally, state your conclusion.`
- `Before recommending, list the tradeoffs for each option in a table.`

**Google alignment:** CoT is Google Vertex AI's recommended strategy for complex reasoning tasks.

**Claude 4 note:** Claude 4 (Sonnet/Opus) has strong native CoT capability — activate it explicitly for best results. Use `<thinking>` tags to request scratchpad reasoning.

---

## 8. XML Structure for Claude

**Rule:** Use XML tags to organize multi-part prompts for Claude. This reduces ambiguity and improves instruction-following on complex tasks.

**Common tags:**
```xml
<context> </context>       — background information
<task> </task>             — the core instruction
<constraints> </constraints> — rules and limits
<output_format> </output_format> — structure spec
<example> </example>       — demonstrations
<data> </data>             — structured input data
```

**Do not** use XML tags for simple, single-instruction prompts — it adds noise without benefit.

---

## 9. Specificity Over Length

**Rule:** Longer prompts ≠ better prompts. Precision beats volume.

| Vague (long) | Precise (short) |
|-------------|-----------------|
| "Please provide a comprehensive and detailed analysis covering all aspects of the market situation..." | "Analyze: market size, top 3 competitors, entry barriers. Table format. 300 words max." |

**Google alignment:** "Use clear and specific language" — Vertex AI prompt design strategy #2.

---

## 10. Grounding for Factual Tasks

**Rule:** For factual prompts, anchor the model to provided data rather than relying on parametric memory.

```xml
<data>
[paste your data here]
</data>

<task>
Based solely on the data above, identify the top 3 trends. Do not use external knowledge.
</task>
```

This is the precursor to the full CoVe framework for tasks where hallucination risk is high.
