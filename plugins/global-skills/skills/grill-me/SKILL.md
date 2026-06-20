---
name: grill-me
description: Relentlessly interview the user to scope a new Project, Plan, Workflow, or any new activity until both parties share complete understanding of the work. Use this skill whenever the user says "grill me", or asks you to help scope, plan, define, or break down a new project, initiative, feature, or workflow. Walks the design tree one question at a time, resolves dependencies between decisions, explores the codebase to answer questions when possible, and researches unknowns when the user can't answer. Always provides a recommended answer with each question. Ends by proposing the best output format to capture the conversation's results.
---

# Grill Me

A structured interview skill that helps the user pin down the full scope and approach of a new project, plan, or workflow through systematic one-at-a-time questioning.

## Core principles

1. **One question at a time.** Never bundle questions. Ask, wait for the answer, then ask the next one. Bundling overwhelms the user and produces shallow answers.

2. **Always recommend an answer.** Every question must come with your recommended answer and brief reasoning. This makes it easy for the user to accept, modify, or push back — they don't have to invent answers from scratch.

3. **Explore before you ask.** If a question can be answered by reading the codebase (file structure, existing dependencies, current patterns, naming conventions, build tools), explore the codebase first and report findings. Only ask the user about things that can't be discovered.

4. **Research what the user doesn't know.** When the user doesn't have an answer, offer to research it (web search, documentation, codebase analysis) rather than forcing them to guess.

5. **Walk the dependency tree.** Order questions so that decisions which unlock others come first. Don't ask about implementation details before purpose is clear. Don't ask about UI before data model. Resolve parent decisions before child decisions.

6. **Be relentless but not annoying.** Keep going until scope is genuinely clear — but recognize when the user has signaled "enough, I get it." Quality of understanding matters more than checking every box.

## The interview process

### Phase 1: Frame the work

Start by establishing the big picture in 2-4 questions:

- What are you trying to accomplish? (the goal, not the solution)
- Who is this for? (audience, stakeholders, just you)
- What does "done" look like? (success criteria)
- What's the rough timeframe or urgency?

These shape every downstream decision. Don't skip them even if the user seems eager to dive into details.

### Phase 2: Map the decision tree

Based on Phase 1 answers, identify the major decision branches. For a software project these typically include:

- **Scope boundaries** — what's in, what's out, what's a stretch goal
- **Technical approach** — language, framework, architecture style
- **Data model** — what entities exist, how they relate
- **Interfaces** — APIs, UI, CLI, integrations with other systems
- **Constraints** — performance, security, compliance, budget
- **Dependencies** — what this depends on, what depends on this
- **Risks and unknowns** — what could go wrong, what's unclear

For non-software work (plans, workflows, processes), adapt the categories — but the principle holds: identify the branches before diving down any one of them.

### Phase 3: Walk each branch

For each branch, ask one question at a time, in dependency order. For each:

1. **State the question** clearly and concisely.
2. **Provide your recommended answer** with reasoning ("My recommendation: X, because Y").
3. **Note what informed your recommendation** — codebase exploration, common practice, the user's prior answers.
4. **Wait for the response**, then either accept, refine, or research as needed.

If the user says "I don't know":
- First, check if you can answer it yourself by exploring the codebase or running a search.
- If you can, do it and report back with findings + a refined recommendation.
- If it requires user judgment (preference, business priority), help them think through it with a brief framing of the tradeoffs.

### Phase 4: Surface unknowns and risks

Before wrapping up, explicitly ask:
- "What am I missing that you've been thinking about?"
- "What part of this still feels fuzzy to you?"
- "What's most likely to change once we start?"

These often surface the highest-value information.

### Phase 5: Propose the output format

End by asking the user how they want to capture the outcome of the interview. Recommend a format based on the nature of what was discussed:

| If the outcome is... | Suggest... |
|---|---|
| Coding behavior rules for this project | Append to project `CLAUDE.md` |
| Long-term context worth remembering across sessions | Save as a memory entry |
| A specific multi-step implementation plan | Create a plan file (e.g., `PLAN.md`) |
| A structured list of decisions with rationale | Create a decisions doc or CSV |
| Tasks to execute now | Create tasks in the task system |
| A reference for the team | Markdown doc in the repo |

Always recommend one format with reasoning, but let the user choose. Then produce the artifact.

## What good "grilling" looks like

**Good question:**
> "Should authentication be session-based or token-based? My recommendation: JWT tokens, because you mentioned this needs to support a mobile app eventually and tokens travel better across clients than sessions. Existing code in `auth/` uses sessions, so switching now is cheaper than later. Sound right, or do you want to keep sessions?"

**Bad question:**
> "What about auth, caching, and the database schema?"

The good question is specific, has a recommendation with reasoning grounded in both context and exploration, and offers a clear yes/no/modify path. The bad question is a bundle that forces the user to do all the thinking.

## When to stop

Stop the interview when:
- The user explicitly signals "enough" or "let's start building"
- Every major branch has been resolved or explicitly deferred
- Remaining unknowns are small enough to be resolved during execution, not planning

Don't stop just because you've asked a lot of questions — stop when the scope is genuinely clear. Don't keep going past the point of diminishing returns either; relentless doesn't mean infinite.

## Output

After the user picks a format, produce the artifact directly. Include:
- The goal and success criteria (Phase 1)
- The major decisions made and their rationale
- Explicitly deferred questions (so they're not forgotten)
- Known risks and unknowns
- Next steps

The artifact should let someone (including future-you in a new session) pick up the work with full context.
