# Claude skill: prompt engineer

A Claude skill that takes rough, draft, or vague prompts and rewrites them into production-ready, model-aligned prompts optimized for maximum output quality.

Works in both **Claude.ai chat** and **Claude Code** (CLI).

---

## What it does

Paste any prompt — even a single sentence — and this skill will:

1. **Classify** the prompt type (research, creative, reasoning, instruction, or conversational)
2. **Apply** the appropriate optimization framework
3. **Return** a fully structured, copy-paste-ready prompt with a changelog explaining every change

### Frameworks applied automatically

| Prompt type | Framework used |
|-------------|----------------|
| Research / factual | CoVe (chain-of-verification) — 4-stage hallucination prevention |
| Reasoning / analysis | Chain-of-thought with structured output |
| Creative / generative | Role declaration + examples + format spec |
| Instruction / agentic | Step decomposition + constraints + fallbacks |
| Conversational | Persona + tone + scope definition |

### Output structure

Every optimized prompt comes with:

- **Optimized prompt** — ready to copy and use
- **What changed & why** — table showing every modification and its rationale
- **Usage notes** — recommended model, temperature setting, whether CoVe was applied, estimated quality lift

---

## How to use

### In Claude.ai (chat)

**Install:**

1. On this page, click the green **Code** button → **Download ZIP**
2. In Claude.ai, go to **Settings → Customize → Skills**
3. Click **+** → **Upload a skill** and select the downloaded ZIP file
4. The skill is now active in your account

**Use:**

```
/custom-prompt-engineer write me a summary of our competitors
```

Type `/` in any conversation to see all installed skills in the autocomplete menu.

### In Claude Code (CLI)

**Install the skill:**

```bash
# Clone this repo
git clone https://github.com/podgorskipiotr/claude-skill-prompt-engineer.git

# Copy to your Claude skills directory
cp -r claude-skill-prompt-engineer ~/.claude/skills/custom-prompt-engineer
```

**Use it:**

```
/custom-prompt-engineer optimize this: write a summary of the market
```

Or reference it in your Claude Code session:

```
/custom-prompt-engineer help me write a prompt that extracts action items from meeting transcripts
```

---

## Example

### Input

```
write me a summary of our competitors
```

### Output

```
## Optimized prompt

Act as a senior competitive intelligence analyst. You have deep knowledge of market
dynamics, positioning frameworks, and business strategy.

<context>
[Replace with: your company name, industry, primary competitors to focus on]
</context>

<task>
Analyze the competitive landscape and produce a structured competitor summary.
For each competitor, cover:
1. Core product/service offering
2. Target customer segment and positioning
3. Pricing model (if publicly known)
4. Key differentiators and moat
5. Observable weaknesses or gaps
</task>

<constraints>
- Table format for the comparison section
- Executive summary: 3 sentences max
- Do not include founding history or press release fluff
- Flag any claims that rely on estimation rather than confirmed data
- Avoid hedging language — state conclusions directly
</constraints>

<output_format>
1. Executive summary (3 sentences)
2. Competitor matrix (markdown table)
3. Strategic implications — 3 bullet points max
</output_format>
```

---

```
## What changed & why

| Element         | Original         | Optimized                                | Reason                                     |
|-----------------|------------------|------------------------------------------|--------------------------------------------|
| Role            | None             | Senior competitive intelligence analyst  | Anchors vocabulary, depth, and perspective |
| Task            | Vague verb       | Numbered analysis dimensions             | Forces complete coverage per competitor    |
| Format          | Unspecified      | Table + executive summary + implications | Prevents wall-of-prose output              |
| Constraints     | None             | 5 explicit constraints                   | Eliminates hedging, fluff, and drift       |
| Context block   | None             | XML placeholder for user context         | User must supply their specific framing    |

## Usage notes

- Model: Claude 3.5+ / GPT-4o / Gemini 1.5 Pro
- Temperature: 0.2–0.4 (factual comparative task)
- CoVe applied: Yes (competitive claims are hallucination-prone)
- Estimated output quality lift: High
```

---

## Skill files

```
custom-prompt-engineer/
├── SKILL.md                        ← skill definition and logic
└── references/
    ├── universal-rules.md          ← 10 universal optimization rules with examples
    └── cove-framework.md           ← full chain-of-verification implementation guide
```

---

## The CoVe framework (chain-of-verification)

For research and factual prompts, the skill applies CoVe — a 4-stage protocol that forces the model to verify its own claims before delivering a final response:

1. **Draft** — generate a baseline first-pass answer
2. **Verification planning** — identify every atomic claim (dates, metrics, named entities, causal links) and write an open-ended verification question for each
3. **Isolated execution** — answer each verification question independently, without looking at the draft (prevents confirmation bias)
4. **Final synthesis** — compare draft against verified answers; replace contradictions, flag unverifiable claims, remove unverifiable ones

This eliminates the most common failure mode in AI factual outputs: the model confidently restating its own hallucination as a confirmed fact.

---

## License

MIT
