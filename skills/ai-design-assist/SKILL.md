---
name: ai-design-assist
description: Use when designing UI/UX or frontend features — detects available AI tools (Gemini, etc.) and uses them for design planning, mockups, and visual decisions
---

# AI-Assisted Design

Use external AI tools when available for design planning, UI/UX decisions, mockups, and visual design work.

## Step 1: Detect Available AI Tools

```bash
# Check if Gemini CLI is installed
which gemini 2>/dev/null && echo "GEMINI: available" || echo "GEMINI: not found"

# Check if Gemini extensions are configured
ls ~/.gemini/extensions/ 2>/dev/null
cat gemini-extension.json 2>/dev/null

# Check for other AI tools
which cursor 2>/dev/null && echo "CURSOR: available" || echo "CURSOR: not found"
which copilot 2>/dev/null && echo "COPILOT: available" || echo "COPILOT: not found"

# Check for design tool CLIs
which figma 2>/dev/null && echo "FIGMA CLI: available" || echo "FIGMA: not found"

# Check for MCP servers related to design
grep -i "figma\|design\|image\|screenshot\|browser" .mcp.json ~/.claude/.mcp.json 2>/dev/null
```

## Step 2: Use Gemini for Design Work (If Available)

Gemini excels at visual and design tasks. When Gemini CLI is installed, use it for:

### Frontend Design Planning
```bash
# Ask Gemini to analyze existing UI and suggest improvements
gemini "Analyze the frontend at {{SERVICE_PATH}} and suggest UI component structure for: <task description>"

# Ask Gemini for design system alignment
gemini "Review the existing component library in {{SERVICE_PATH}}/src/components/ and suggest how to design <feature> to match existing patterns"

# Ask Gemini for responsive layout suggestions
gemini "Suggest a responsive layout for <feature> that works on mobile (320px), tablet (768px), and desktop (1440px)"
```

### UX Flow Design
```bash
# Ask Gemini to design user flows
gemini "Design a user flow for <feature>. Include: happy path, error states, loading states, and edge cases. Output as a step-by-step flow."

# Ask Gemini for accessibility review
gemini "Review this component spec for accessibility issues. Check WCAG 2.1 AA compliance: <paste component spec>"
```

### Visual Design Decisions
```bash
# Ask Gemini for color/typography suggestions
gemini "Given the existing design system using <framework/library>, suggest colors, typography, and spacing for <feature>"

# Ask Gemini for component variant design
gemini "Design all visual states for a <component>: default, hover, active, focus, disabled, loading, error, success"
```

### Design from Specs
```bash
# If a design spec exists, ask Gemini to plan the implementation
gemini "Read this design spec and create a detailed frontend implementation plan:
$(cat docs/superpowers/specs/<spec-file>.md)"

# Ask Gemini to convert design spec to component structure
gemini "Convert this UX spec into React/Vue component structure with props, states, and event handlers:
$(cat .claude/orchestrator/runs/*/ux-spec.md 2>/dev/null)"
```

## Step 3: Integrate AI Design Output Into Agent Workflow

### For UX Designer Agent
When Gemini is available:
1. Ask Gemini for initial design suggestions
2. Refine based on project conventions and existing patterns
3. Write final ux-spec.md combining AI suggestions + project context

### For Frontend Developer Agent
When Gemini is available:
1. Ask Gemini to plan component structure from the UX spec
2. Ask Gemini for responsive layout implementation
3. Use suggestions as a starting point, then implement following project patterns

### For Architect Agent
When Gemini is available:
1. Ask Gemini to review architecture for frontend/backend separation
2. Ask Gemini for state management recommendations
3. Use as input for architecture.md

## Step 4: Fallback Without AI Tools

If no external AI tools are available:
- Design based on existing project patterns
- Follow component library conventions
- Use the testing skill's spec discovery to understand existing UI patterns
- Reference Storybook stories if available

## When to Use This Skill

**ONLY for frontend and UI/UX related agents. NOT for backend, database, devops, security, etc.**

| Agent | When to invoke |
|-------|---------------|
| UX Designer | Always — before writing ux-spec.md |
| Frontend Developer | When implementing UI components — for layout/structure planning |

Do NOT use this skill for:
- Backend developers (no UI work)
- Database testers (no visual design)
- DevOps agents (no frontend)
- Security analysts (no design)
- Architect (uses own domain knowledge, not AI design tools)

## Rules
- AI suggestions are a STARTING POINT, not the final answer
- Always validate AI output against project conventions (CLAUDE.md, existing patterns)
- Never blindly copy AI-generated code — adapt to project style
- If AI tool is not available, skip this step and proceed with manual design
- Log which AI tool was used in the agent's output report
