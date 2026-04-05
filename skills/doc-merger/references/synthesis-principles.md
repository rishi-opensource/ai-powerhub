# Synthesis Principles — Document Intelligence Engine

## Core Mandate

Produce ONE unified document. Not a summary. Not a list of per-document takeaways. A single coherent narrative that makes the original documents unnecessary.

## What Synthesis Means

| Do | Don't |
|----|-------|
| Merge overlapping ideas into one explanation | Summarize each document separately |
| Resolve contradictions with reasoning | Present contradictions as equally valid without comment |
| Connect scattered information into a flow | List facts in isolation |
| Normalize terminology | Use different names for the same concept interchangeably |
| Explain at the right depth for the audience | Assume prior knowledge |

## Adaptive Structure

**Never force a fixed template.** Choose the structure that best serves the content:

- **Narrative** — for evolution of a system, incident history, decision rationale
- **Concept-first** — for technical systems where understanding the "why" unlocks the "how"
- **Layered** — beginner → intermediate → advanced, when audience depth is mixed
- **System design breakdown** — for architecture docs (components, flows, interfaces)
- **FAQ / Q&A** — for decision docs with many options evaluated
- **Timeline** — for projects, migrations, or processes with phases

Ask yourself: *"What is the best possible way to explain this to someone new?"* Then use that structure.

## Conflict Resolution

When documents contradict each other:

1. Try to resolve using context (dates, authority, specificity)
2. If resolvable: state the correct version and briefly note why
3. If unresolvable: surface it explicitly

Example:
> **Note — Conflicting information**: Document A states the timeout is 30s; Document B states 60s. Based on Document B's more recent date, 60s is likely current. Verify before relying on this.

## Gap Handling

When information is missing or unclear:
- Label inferences explicitly: *(Inferred)*
- Label unknowns explicitly: *(Unclear from available docs)*
- Make minimal inferences — only when strongly supported by surrounding context

## Terminology Normalization

- Identify aliases: different names for the same thing across docs
- Pick one canonical term and use it throughout
- On first use, define the term and list its aliases: *"The ingestion pipeline (also called 'data loader' in older docs)…"*

## Audience Adaptation

| Audience | Tone & Depth |
|----------|-------------|
| Engineering | Technical, precise, include system internals and data flows |
| Product | Feature-focused, user impact, avoid deep implementation detail |
| Business | Goal-oriented, metrics-driven, plain language, no jargon |
| Mixed | Layer it — start conceptual, deepen progressively |

## Quality Bar

The output must be:
- **Self-sufficient** — reader needs nothing else
- **Accurate** — no speculation beyond labeled inferences
- **Complete** — no important detail from the source docs is lost
- **Clear** — no assumed knowledge, no undefined jargon
- **Well-paced** — starts accessible, builds complexity naturally
