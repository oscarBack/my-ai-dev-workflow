# AI-Assisted Development Workflow

## Overview
This workflow describes the process for implementing new features or fixes on existing projects using AI assistance, structured planning, and iterative refinement.

## Workflow Diagram

```mermaid
flowchart TD
    Start([🚀 New Feature or Fix]) 
    
    subgraph Planning["Phase 1️⃣ : Planning"]
        PlanMode["🎯 Plan Mode<br/>(Conversational)"]
        PlanTools["AI Skills:<br/>- Brainstorming<br/>- Ask Questions<br/>- Clarify Ambiguities"]
        PlanMode -.->|uses| PlanTools
    end
    
    subgraph Refinement["Phase 2️⃣ : Refinement"]
        AgentRefine["Agent: Plan Refinement<br/>Break into Micro-Tasks≤2min<br/>List Business Rules"]
        RefineSkills["Skills:<br/>- Task-Breakdown-Specialist<br/>- TDD<br/>- Feature Flags<br/>- Gradual Rollout"]
        AgentRefine -.->|uses| RefineSkills
    end
    
    subgraph Review["Phase 3️⃣ : Review & Approve"]
        HumanReview{"👤 Human Review<br/>Approve Plan?"}
        Feedback["❌ Request Changes"]
    end
    
    subgraph Persistence["Phase 4️⃣ : Context Persistence"]
        Persist["💾 Store Session Context"]
        EnsureItems["✓ MD session file<br/>✓ GitHub issues<br/>✓ Current task pointer"]
        Persist -.->|ensures| EnsureItems
    end
    
    subgraph Execution["Phase 5️⃣ : Execution Loop"]
        TaskLoop["Repeat for each Task:"]
        FeatureFlag["1️⃣ Feature Flag<br/>(Models: Sonnet 4, Haiku 4.5)"]
        TestFail["2️⃣ Test-to-Fail<br/>(Models: Haiku 4.5, GPT)"]
        PassTest["3️⃣ Pass Test<br/>(Implementation)"]
        UpdatePlan["3.2️⃣ Update Plan File<br/>(Mark task done<br/>Record status)"]
        PreCommit["3.5️⃣ Pre-Commit Hook<br/>(Verify Business Rules)"]
        Commit["4️⃣ Commit Changes"]
        CommitUpdate["Update session<br/>persistence"]
        
        TaskLoop --> FeatureFlag
        FeatureFlag --> TestFail
        TestFail --> PassTest
        PassTest --> UpdatePlan
        UpdatePlan --> PreCommit
        PreCommit --> Commit
        Commit --> CommitUpdate
    end
    
    Start --> Planning
    Planning --> Refinement
    Refinement --> Review
    HumanReview -->|✅ Approved| Persistence
    HumanReview -->|❌ Rejected| Feedback
    Feedback -->|Loop back| Planning
    Persistence --> Execution
    CommitUpdate --> MoreTasks{More Tasks?}
    MoreTasks -->|Yes| TaskLoop
    MoreTasks -->|No| Complete([✨ Complete])
    
    style Planning fill:#e1f5ff
    style Refinement fill:#f3e5f5
    style Review fill:#fff3e0
    style Persistence fill:#e8f5e9
    style Execution fill:#fce4ec
```

## Workflow Phases

### Phase 1: Plan Mode
- **Purpose**: Clarify requirements and scope with conversational AI
- **File Naming Standard**: `.ai/plans/YYYYMMDD-[issue#]-[slug].plan.md` (e.g., `.ai/plans/20260219-42-add-auth.plan.md`)
- **Format**: Simple task list with open questions (see Plan Format → Phase 1)
- **Tools**: Skills for brainstorming and asking clarifying questions
- **Models**: Opus 4.5, Haiku 4.5
- **Output**: Initial conversational plan submitted for human review

### Phase 2: Agent Plan Refinement
- **Purpose**: Convert initial plan into concrete, measurable micro-tasks with strict schema
- **Transition**: Human approves Phase 1 plan; Human + Agent collaborate to refine into Phase 2 structure
- **Format**: Detailed tasks with Title, Status, Business Rules, Acceptance Criteria, Dependencies, GitHub Links, Tests (see Plan Format → Phase 2)
- **Breakdown**: Each task targets ~2 minutes of work
- **Business Rule Identification**: Explicitly list all new, updated, and deleted business rules per task
- **Skills**: Breakdown-task-specialist, TDD, Feature Flag, Gradual Rollout
- **Models**: Opus 4.5, Haiku 4.5
- **Output**: Detailed task breakdown with clear completion tracking (In Progress → Done sections)

### Phase 3: Human Review
- **Purpose**: Validate plan before starting execution
- **Business Rule Verification**: Plan must explicitly list all new, updated, and deleted business rules for targeted review
- **Feedback**: Can loop back to Plan Mode if adjustments needed
- **Context Persistence**: Generate session files and GitHub issues
- **Output**: Approved task list with context stored

### Phase 4: Execution (Agent-driven)
- **Purpose**: Execute approved micro-tasks using TDD and feature flags
- **Feature Flag**: Set up feature flag infrastructure
- **Test-Driven**: Write failing test first (Red-Green-Refactor)
- **Implementation**: Make tests pass
- **Task Completion**: Move task from "In Progress" → "✅ Done" section in plan file
- **Pre-Commit Hook**: Automatically request human verification of new, updated, and deleted business rules
- **Commit**: Version control integration (reference commit SHA in plan's Done section)
- **Iteration**: Repeat for each micro-task until all tasks move to Done
- **Plan Updates**: Keep plan file synchronized with task progress (see Plan Format → Task Status Legend)

## Key Principles

1. **Conversation-Driven Planning**: Start with natural discussion to clarify ambiguous requirements
2. **Micro-Task Decomposition**: Break work into small, completable units (≤2 min each)
3. **Human-in-the-Loop Review**: Always validate AI plan before execution
4. **Persistent Context**: Store session details for continuity across sessions
5. **Test-First Implementation**: Ensure quality with TDD methodology
6. **Business Rule Verification**: Ensure human verification of the most important/sensitive logic (business rules) even if full code review isn't performed
7. **Feature Flags**: Safe rollout with gradual feature enablement

## Plan Format

### Phase 1: Simple Plan Format (Before Human Review)

Create a conversational task list in `.ai/plans/YYYYMMDD-[issue#]-[slug].plan.md`:

```markdown
# Plan: [Feature/Fix Name]
**Date**: 2026-02-20  
**Issue**: [GitHub issue link or #123]  
**Status**: 📋 Planning

## Overview
Brief description of what we're building.

## Tasks

- [ ] Task 1 description
- [ ] Task 2 description
- [ ] Task 3 description

## Open Questions
- Question 1?
- Question 2?
```

### Phase 2: Detailed Plan Format (After Human Review & Approval)

After human review, Human + Agent collaborate to transform into detailed schema:

```markdown
# Plan: [Feature/Fix Name]
**Date**: 2026-02-20  
**Issue**: [GitHub issue link]  
**Status**: ✅ Approved & In Execution  
**Model Strategy**: Sonnet 4 (flags), Haiku 4.5 (tests), Opus 4.5 (complex)

## Overview
Brief description of what we're building.

## In Progress

### Task 1: [Title]
- **Status**: 🔄 IN_PROGRESS
- **GitHub Issue**: [Link to issue, e.g., #123](https://github.com/org/repo/issues/123)
- **Est. Time**: ~2min
- **Business Rules**
  - New: [Describe new business rules]
  - Updated: [Describe changed business logic]
  - Deleted: [Describe removed business logic]
- **Acceptance Criteria**
  - [ ] Criterion 1
  - [ ] Criterion 2
  - [ ] Criterion 3
- **Dependencies**: None | Task 0 must complete first
- **Tests Required**
  - Unit test: [describe what to test]
  - Integration test: [describe scenario]

### Task 2: [Title]
- **Status**: 🟡 BLOCKED (waiting for Task 1)
- **GitHub Issue**: [Link]
- **Est. Time**: ~2min
- **Business Rules**
  - New: [...]
- **Acceptance Criteria**
  - [ ] Criterion 1
- **Dependencies**: Task 1
- **Tests Required**
  - Unit test: [...]

## ✅ Done

### Task 0: [Completed Task Name]
- **Status**: ✅ DONE
- **GitHub Issue**: [Link]
- **Completed**: Commit [sha](https://github.com/org/repo/commit/sha)
```

**Note on Schema Flexibility**: The fields shown above are the **minimum required** for Phase 2 tasks. You may add optional fields as needed:
- **Code Examples**: Before/after snippets to illustrate expected changes
- **Notes**: Implementation details, gotchas, or edge cases
- **References**: Links to related code, documentation, or architectural decisions
- **Context**: Additional background for complex tasks
- **Verification Steps**: Manual steps to verify completion beyond automated tests

The schema adapts to task complexity—simple tasks may use minimal fields, while complex tasks can include extensive examples and context.

### Task Status Legend

| Status | Icon | Meaning |
|--------|------|---------|
| Pending | 📭 | Not started |
| In Progress | 🔄 | Currently being worked on |
| Blocked | 🟡 | Waiting on dependency or decision |
| Done | ✅ | Complete & committed |

### Completing a Task

When a task is complete:
1. Move task block from "In Progress" → "✅ Done" section
2. Change status to `✅ DONE`
3. Add completion info (commit SHA, link)
4. Update related GitHub issue

## Session Persistence

Between sessions, maintain:
- Markdown file documenting the current plan and progress (`.ai/plans/YYYYMMDD-[issue#]-[slug].plan.md`)
- GitHub issues tracking individual micro-tasks (linked in plan)
- Selected task information for agent continuation
- Clear visibility of completed vs. remaining work (Done section vs. In Progress section)

## AI Models Used

- **Opus 4.5**: Complex planning, refinement
- **Haiku 4.5**: Task execution, quick implementations
- **Sonnet 4**: Feature flag implementation
- **GPT 4o**: Advanced testing scenarios
- **GPT 4.1**: Integration and complex logic

## When to Use This Workflow

✅ **Use for:**
- New features on existing projects
- Bug fixes requiring planning
- Refactoring initiatives
- Complex feature implementations

❌ **Skip when:**
- Trivial one-liner fixes
- Well-understood, routine tasks
- Emergency hotfixes (though consider adding context after)
