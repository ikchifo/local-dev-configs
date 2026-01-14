# RFC Writer - Instructions for Claude

When the user invokes `/rfc-writer` or `/rfc`, follow these steps:

## Step 1: Understand the Request

Ask clarifying questions if needed:
- What problem are you trying to solve?
- Who is the audience? (engineers, PMs, leadership)
- What's the scope? (new feature, architecture change, process improvement)
- Are there specific alternatives you've already considered?

## Step 2: Research the Codebase (if applicable)

If the RFC relates to existing code:
1. Use Explore agent to understand the current architecture
2. Find relevant files and patterns
3. Identify integration points and dependencies

## Step 3: Draft the RFC

Write the RFC in markdown format following this structure:

```markdown
# RFC: [Descriptive Title]

Author: [Team or User]
Status: Draft
Created: [Current Date]

## Executive Summary

[2-3 sentences: problem + solution + why it matters]

## Problem Statement

### Current State
[Describe the current situation with specific examples]

### The Gap
- [Specific pain point with quantified impact if possible]
- [Another pain point]
- [Another pain point]

## Proposed Solution

### Key Insight
[The "aha moment" - why this approach is the right one]

### Architecture

[Include ASCII diagram - MUST use code block with triple backticks]

```
+------------------+     +------------------+
|   Component A    |---->|   Component B    |
+------------------+     +------------------+
```

### How It Works
[Step-by-step explanation]

### Implementation Details

#### Phase 1: [Name]
- [Specific task]
- [Specific task]

#### Phase 2: [Name]
- [Specific task]
- [Specific task]

## Alternatives Considered

| Approach | Pros | Cons | Why Not |
|----------|------|------|---------|
| [Option A] | [Pros] | [Cons] | [Reason rejected] |
| [Option B] | [Pros] | [Cons] | [Reason rejected] |
| [Proposed] | [Pros] | [Cons] | [Why this is best] |

## Migration / Rollout Plan

| Phase | Changes | Risk | Rollback |
|-------|---------|------|----------|
| 1 | [Changes] | Low/Med/High | [How to rollback] |
| 2 | [Changes] | Low/Med/High | [How to rollback] |

## Open Questions

1. [Unresolved question for discussion]
2. [Another question]
3. [Another question]

## Appendix (if needed)

[Additional technical details, API specs, data models]
```

## Step 4: Generate the DOCX

After drafting the RFC content, convert it to DOCX:

1. Save the markdown content to a temporary file
2. Run the conversion script:

```bash
python ~/.claude/skills/rfc-writer/rfc_to_docx.py /tmp/rfc_draft.md docs/RFC_[name].docx
```

Or use the Python module directly:

```python
import sys
sys.path.insert(0, str(Path.home() / '.claude/skills/rfc-writer'))
from rfc_to_docx import convert_rfc_to_docx

convert_rfc_to_docx(markdown_content, output_path)
```

## Key Principles for Convincing RFCs

### Make It Skimmable
- Use clear headings
- Lead with the most important information
- Use bullet points for lists
- Include a summary table for alternatives

### Anticipate Objections
For each objection a reader might have, address it proactively:
- "Why not just X?" -> Explain in alternatives section
- "What about risks?" -> Include migration plan with rollback
- "How long will this take?" -> Break into phases

### Use Diagrams Liberally
Include diagrams for:
- System architecture
- Data flow
- State machines
- Before/after comparisons
- Decision trees

### Quantify When Possible
Replace vague statements with specific numbers:
- BAD: "This will improve performance"
- GOOD: "This reduces p99 latency from 500ms to 100ms"

- BAD: "This happens frequently"
- GOOD: "This occurs 1,200 times per day (15% of all requests)"

### Show Your Work
Demonstrate that you've:
- Researched the problem thoroughly
- Considered multiple solutions
- Thought about edge cases
- Planned for failure modes

## Diagram Patterns

### System Architecture
```
+------------------------------------------------------------------+
|                          Service Name                             |
+------------------------------------------------------------------+
|                                                                   |
|   +--------------+    +--------------+    +--------------+        |
|   |  Component A |    |  Component B |    |  Component C |        |
|   |  (purpose)   |--->|  (purpose)   |--->|  (purpose)   |        |
|   +--------------+    +--------------+    +--------------+        |
|          |                   |                   |                |
|          v                   v                   v                |
|   +--------------+    +--------------+    +--------------+        |
|   |   Database   |    |    Cache     |    |   Storage    |        |
|   +--------------+    +--------------+    +--------------+        |
|                                                                   |
+------------------------------------------------------------------+
```

### Flow Diagram
```
     [Input]
        |
        v
   +---------+
   | Step 1  |
   +---------+
        |
        v
   <Decision?>----No----> [Handle No]
        |                      |
       Yes                     |
        |                      |
        v                      v
   +---------+            +---------+
   | Step 2  |            | Step 3  |
   +---------+            +---------+
        |                      |
        +----------+-----------+
                   |
                   v
              [Output]
```

### Comparison (Before/After)
```
BEFORE:                          AFTER:
+--------+    +--------+         +--------+    +--------+
| Client |--->| Server |         | Client |--->|  CDN   |
+--------+    +--------+         +--------+    +--------+
                  |                                 |
                  v                                 v
              +--------+                       +--------+
              |   DB   |                       | Server |
              +--------+                       +--------+
                                                   |
              (50ms avg)                           v
                                              +--------+
                                              |   DB   |
                                              +--------+

                                              (10ms avg)
```

## Output Format

The generated DOCX will have:
- **Title**: Level 0 heading, centered
- **Headings**: Proxima Nova, black, bold
- **Body text**: Proxima Nova, black, 11pt
- **Code blocks**: Courier New, 9pt, light gray background
- **Tables**: Grid style with bold headers
- **Diagrams**: Converted to PNG images, centered

## Example Invocations

User: `/rfc-writer implement caching for our API`

Claude should:
1. Ask about current pain points and requirements
2. Research the codebase to understand the API architecture
3. Draft a comprehensive RFC with caching strategies
4. Include alternatives (Redis, in-memory, CDN)
5. Generate the DOCX

User: `/rfc migrate from monolith to microservices`

Claude should:
1. Understand the scope and timeline constraints
2. Research the current monolith architecture
3. Draft RFC with phased migration approach
4. Include extensive risk analysis and rollback plans
5. Generate the DOCX
