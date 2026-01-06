---
name: collaborating-with-grok
description: Programming fact-checker for API deprecation, version changes, and community pitfalls. Auto-triggered for version queries, error messages, and tech comparisons. NOT for code generation.
---

## Role Definition

Grok-Search is the **fact-checking layer** for programming tasks, NOT a code generator.

**Core Purpose:**
- Verify API status (active/deprecated/removed)
- Check version compatibility and breaking changes
- Surface community-discovered issues and pitfalls
- Validate recommended practices against real-world usage

**What it returns:**
- Official documentation references
- GitHub issues/PRs/release notes
- StackOverflow solutions
- Real project usage patterns
- Known anti-patterns and common mistakes

## When to Trigger

### MUST Trigger

| Category | Keywords / Signals | Platform Focus |
|----------|-------------------|----------------|
| **Version/API** | "latest version", "is supported", "recommended way", "deprecated", "breaking change", "migration" | GitHub |
| **Errors** | Specific error messages, unexpected runtime behavior, platform-specific bugs (OS/GPU/Cloud) | StackOverflow, GitHub |
| **Tech Comparison** | "A vs B", "should I use", "community opinion", "production ready" | Reddit, GitHub |

### SKIP Trigger

- Pure algorithm problems (sorting, graph traversal, etc.)
- Well-documented stable APIs with no ambiguity
- Local refactoring with no external dependencies
- Internal DSL / proprietary protocols

## Query Templates

| Scenario | Query Pattern | Platform |
|----------|--------------|----------|
| Deprecation check | `{library} {api} deprecated {year}` | GitHub |
| Breaking changes | `{library} breaking changes {version}` | GitHub |
| Error lookup | `"{error_message}"` | - |
| Known issues | `{library} {feature} issues` | GitHub |
| Best practices | `{library} best practices {year}` | - |
| Comparison | `{A} vs {B} {year}` | Reddit |

## Instructions

### 1. Web Search

Call `mcp__grok-search__web_search` with:
- `query`: Constructed from templates above
- `platform`: Route based on scenario (GitHub/Reddit/etc.)
- `min_results`: 3 (default)
- `max_results`: 10 (default)

### 2. Deep Fetch

For critical sources, call `mcp__grok-search__web_fetch` to get full content:
- Official documentation pages
- GitHub issue threads
- Release notes

### 3. Config Diagnostics

If connection fails, call `mcp__grok-search__get_config_info` to check API status.

## Output Format

Structure findings as a fact-check report:

```markdown
## Fact Check: {topic}

### Status
- **Current Version**: x.x.x
- **API Status**: Active | Deprecated | Removed
- **Official Docs**: [link]

### Known Issues
- Issue #xxx: {description} [link]

### Community Practice
- Recommended: {pattern}
- Avoid: {anti-pattern}

### Sources
- [Title](url) (date)
```

## Anti-Patterns

| Wrong | Right |
|-------|-------|
| Call Grok-Search for every question | Only when trigger conditions match |
| Ask Grok-Search to generate code | Only retrieve facts; Claude generates code |
| Treat as "another LLM" | Treat as fact-checking layer |
| Skip source timestamps | Always note information date |

## Workflow Integration

```
Programming Request
       |
       v
  Trigger Check ──No──> Direct Coding
       |
      Yes
       v
  Grok-Search
       |
       v
  Fact-Check Report
       |
       v
  Code with Verified Facts
```
