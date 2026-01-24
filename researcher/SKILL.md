---
name: researcher
description: Systematic research and investigation for informed decision-making. Use when asked to research, investigate, explore options, compare alternatives, evaluate feasibility, assess approaches, find best practices, or answer "how should we" / "what's the best way to" questions. Covers software engineering research (libraries, APIs, architecture patterns, debugging) and product research (competitors, market analysis, user needs, feature validation). Triggers on phrases like "research", "investigate", "compare", "evaluate", "explore options", "what are the approaches", "help me understand", "deep dive into".
---

# Researcher

Systematic methodology for gathering, analyzing, and synthesizing information to inform decisions.

## Core Workflow

### 1. Scope the Question

Before searching, clarify:
- **Decision**: What choice does this research inform?
- **Constraints**: Time, budget, technical requirements, existing stack
- **Success criteria**: What would a useful answer look like?

If unclear, ask one focused clarifying question before proceeding.

### 2. Identify Source Strategy

Select sources based on research type:

| Research Type | Primary Sources | Secondary Sources |
|--------------|-----------------|-------------------|
| Library/tool selection | Official docs, GitHub repo, release notes | Comparison articles, benchmarks |
| API integration | Official API docs, SDKs, changelog | Stack Overflow, GitHub issues |
| Architecture decisions | Reference implementations, case studies | Blog posts, conference talks |
| Debugging/errors | Source code, issue trackers, changelogs | Forums, Q&A sites |
| Product/market | User reviews, usage data, competitor products | Industry reports, analyst coverage |
| Feasibility | Proof-of-concept code, documentation | Community experience |

**Source priority**: Primary sources > recent secondary sources > older content

### 3. Structured Gathering

Execute research systematically:

1. **Start broad**: Get landscape overview (2-3 searches max)
2. **Go deep**: Dive into top candidates with specific queries
3. **Verify recency**: Check dates, version numbers, "last updated"
4. **Cross-reference**: Validate claims across multiple sources
5. **Note gaps**: Track what you couldn't find or verify

**Anti-patterns to avoid**:
- Searching the same thing multiple ways hoping for different results
- Reading 10 sources that all say the same thing
- Going down interesting but irrelevant rabbit holes
- Stopping at the first answer without verification

### 4. Synthesize Findings

Structure output for the decision at hand:

**For comparisons**: Use decision matrix with weighted criteria
**For feasibility**: Lead with yes/no/maybe, then evidence
**For "how to"**: Provide recommended approach with alternatives noted
**For exploration**: Map the landscape, highlight key tradeoffs

Always include:
- Confidence level (high/medium/low) with reasoning
- Key unknowns or gaps in research
- Recommended next steps

### 5. Know When to Stop

Stop researching when:
- You have enough to make a reversible decision
- Additional sources repeat what you've found
- You've hit diminishing returns (3+ sources saying same thing)
- The question requires hands-on experimentation, not more reading

---

## Tools

Use these tools for research tasks:

| Tool | When to Use | Best For |
|------|-------------|----------|
| **Context7 MCP** | Library/framework documentation | Accurate, up-to-date API docs |
| **WebSearch** | General research, comparisons, current info | Broad landscape, recent developments |
| **WebFetch** | Specific URLs, official docs | Reading full documentation pages |
| **Grep/Glob** | Codebase research | How things work in current project |

### Context7 MCP Usage

For library documentation, **always use Context7 first**:
```
1. resolve-library-id: Find the library ID
2. query-docs: Query specific documentation
```

### Time-Boxing Research

Set limits to avoid rabbit holes:

| Research Type | Time Budget |
|---------------|-------------|
| Quick lookup | 2-3 searches |
| Comparison | 5-7 searches |
| Deep dive | 10-15 searches |
| Feasibility | Until clear yes/no/maybe |

**If exceeding budget:** Stop, synthesize what you have, note gaps.

---

## Domain-Specific Guidance

- **Software engineering research**: See [references/software-eng.md](references/software-eng.md)
- **Product and market research**: See [references/product.md](references/product.md)
- **Output templates**: See [references/synthesis-templates.md](references/synthesis-templates.md)

## Quality Checklist

Before delivering findings:
- [ ] Sources are dated within relevant timeframe
- [ ] Primary sources consulted, not just aggregated content
- [ ] Tradeoffs articulated, not just "X is best"
- [ ] Confidence levels stated with reasoning
- [ ] Next steps are actionable
