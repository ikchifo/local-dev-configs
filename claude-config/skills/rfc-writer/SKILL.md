---
name: rfc-writer
description: Use when writing, reviewing, or improving RFCs (Request for Comments). Helps create compelling, well-structured technical proposals with proper formatting, diagrams, and styling that convince engineers and PMs.
---

# RFC Writer Skill

This skill helps you write compelling, well-structured RFCs (Request for Comments) that convince engineers and PMs. It generates professional DOCX documents with proper formatting, diagrams, and styling.

## Usage

Invoke with: `/rfc-writer` or `/rfc`

### Arguments
- `<topic>` - The topic or problem you want to write an RFC about
- `--output <path>` - Optional output path for the DOCX file (default: `docs/RFC_<topic>.docx`)

### Examples
```
/rfc-writer Add delta-based refinement evaluation
/rfc-writer Migrate from REST to GraphQL --output proposals/graphql_migration.docx
/rfc Implement caching layer for API responses
```

## RFC Structure

When writing an RFC, follow this proven structure:

### 1. Title & Metadata
```markdown
# RFC: [Clear, Descriptive Title]

Author: [Your Name/Team]
Status: Draft
Created: [Date]
```

### 2. Executive Summary (1-2 paragraphs)
- What problem are you solving?
- What's your proposed solution?
- Why should readers care?

### 3. Problem Statement
#### Current State
- Describe how things work today
- Include concrete examples and data if available

#### The Gap / Pain Points
- What's wrong with the current approach?
- Use bullet points for clarity
- Quantify impact when possible (latency, errors, developer time)

### 4. Proposed Solution

#### Key Insight
Highlight the core insight that makes your solution work. This is often the "aha moment" that convinces skeptical readers.

#### Architecture / Design
Include diagrams! Use ASCII diagrams in code blocks - they will be converted to proper images:

```
+------------------+     +------------------+
|   Component A    |---->|   Component B    |
+------------------+     +------------------+
         |                        |
         v                        v
+------------------+     +------------------+
|   Component C    |<----|   Component D    |
+------------------+     +------------------+
```

### 5. Alternative Solutions Considered

Always include at least 2-3 alternatives you considered and why you rejected them:

| Approach | Pros | Cons | Why Not |
|----------|------|------|---------|
| Option A | Fast to implement | Doesn't scale | Short-term fix |
| Option B | Clean architecture | 3 month timeline | Too slow |
| Option C (proposed) | Balanced | Medium effort | Best trade-off |

### 6. Implementation Details

Break down the implementation into phases:

#### Phase 1: Foundation
- Specific, actionable steps
- Clear deliverables

#### Phase 2: Core Features
- Dependencies on Phase 1
- Measurable milestones

### 7. Migration / Rollout Plan

| Phase | Changes | Risk | Rollback Plan |
|-------|---------|------|---------------|
| Phase 1 | Feature flag | Low | Disable flag |
| Phase 2 | Gradual rollout | Medium | Percentage dial |

### 8. Open Questions

List unresolved questions for discussion:
1. Should we support backward compatibility?
2. What's the SLA requirement?
3. Who owns long-term maintenance?

### 9. Appendix (Optional)

Include reference material:
- API specifications
- Data models
- Benchmark results

## Writing Tips for Convincing RFCs

### For Engineers
- Include technical depth and edge cases
- Show you've considered failure modes
- Reference prior art and industry standards
- Provide clear complexity analysis (O(n), storage requirements)

### For PMs
- Lead with business impact
- Quantify benefits (time saved, errors reduced)
- Include timeline/phases
- Address risks upfront

### General Best Practices
- **Be specific**: Replace "improve performance" with "reduce p99 latency from 500ms to 100ms"
- **Use diagrams**: A picture is worth 1000 words
- **Anticipate objections**: Address them before they're raised
- **Show alternatives**: Demonstrates thoroughness
- **Keep it focused**: One RFC = one proposal

## Diagram Types to Include

### Architecture Diagrams
```
+------------------------------------------------------------------+
|                         System Name                               |
+------------------------------------------------------------------+
|  +--------------+    +--------------+    +--------------+        |
|  |  Service A   |--->|  Service B   |--->|  Service C   |        |
|  +--------------+    +--------------+    +--------------+        |
+------------------------------------------------------------------+
```

### Flow Diagrams
```
  [Start]
     |
     v
  <Decision?> --No--> [Action A]
     |                    |
    Yes                   |
     |                    |
     v                    v
  [Action B] -------> [End]
```

### Data Flow
```
Input --> [Process 1] --> [Process 2] --> Output
              |               ^
              v               |
          [Storage] ----------+
```

## Output

The skill generates a professional DOCX document with:
- **Proxima Nova font** throughout
- **Black text** for readability
- **Proper heading hierarchy**
- **Formatted tables**
- **ASCII diagrams converted to images**
- **Code blocks with syntax highlighting**
- **Consistent styling**

## Implementation

The skill uses `rfc_to_docx.py` to convert markdown-like content to DOCX format. The converter:
1. Parses markdown headings, lists, code blocks, and tables
2. Detects ASCII diagrams and converts them to PNG images
3. Applies consistent Proxima Nova styling
4. Outputs a professional DOCX document

---

## Quick Start Template

When asked to write an RFC, start with this template:

```markdown
# RFC: [Title]

Author: [Author]
Status: Draft
Created: [Date]

## Executive Summary

[2-3 sentences describing the problem and proposed solution]

## Problem Statement

### Current State

[How things work today]

### The Gap

- [Pain point 1]
- [Pain point 2]
- [Pain point 3]

## Proposed Solution

### Key Insight

[The core insight that makes this work]

### Architecture

[Include diagram here]

### Implementation Details

[Break into phases]

## Alternatives Considered

| Approach | Pros | Cons | Decision |
|----------|------|------|----------|
| ... | ... | ... | ... |

## Migration Path

| Phase | Changes | Risk |
|-------|---------|------|
| ... | ... | ... |

## Open Questions

1. [Question 1]
2. [Question 2]
```
