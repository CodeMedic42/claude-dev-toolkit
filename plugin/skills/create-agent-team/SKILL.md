---
name: create-agent-team
description: Expert knowledge for creating Claude Code agent teams. Use when the user wants to orchestrate multiple independent Claude sessions working in parallel with peer-to-peer communication and shared task coordination.
---

# Create Claude Code Agent Teams

You are an expert in creating and orchestrating Claude Code agent teams. Guide users through setting up multiple independent Claude Code sessions that work together on complex tasks through peer-to-peer messaging and shared task lists.

## What Are Agent Teams?

Agent teams are **multiple independent Claude Code sessions** that:
- Run as **separate processes** with their own context windows
- Communicate through **peer-to-peer messaging** (not just reporting to a leader)
- Share a **central task list** for coordination
- Work **autonomously** with self-coordination
- Can **spawn their own subagents** for specialized work

**Agent Teams vs Subagents**:

| Aspect | Subagent | Agent Team |
|:-------|:---------|:-----------|
| Context | Own context, reports back | Fully independent contexts |
| Communication | Reports to caller only | Teammates message each other |
| Coordination | Managed by main agent | Self-coordinated via shared tasks |
| Best for | Focused tasks returning summaries | Complex work requiring collaboration |
| Token cost | Lower (summaries only) | Higher (separate instances) |

**Use subagents** for quick, focused work. **Use agent teams** when teammates need to share findings, challenge each other, and coordinate independently.

## When to Use Agent Teams

Create an agent team when you need:
- **Parallel research** with competing hypotheses
- **Complex feature development** where each teammate owns a piece
- **Code review** from multiple perspectives (security, performance, tests)
- **Investigation** requiring discussion and collaboration
- **Large tasks** that exceed a single context window

**Don't use agent teams for**:
- Simple, focused tasks (use subagents)
- Sequential work with clear dependencies (use main conversation)
- When you need immediate results (teams take longer to coordinate)

## Prerequisites

Agent teams are **experimental** and disabled by default. Enable with:

```bash
export CLAUDE_CODE_AGENT_TEAMS_ENABLED=1
```

Add to your shell config (`.bashrc`, `.zshrc`) to enable permanently.

**Requirements:**
- Claude Code version with agent teams support
- Understanding of task-based workflows
- Willingness to experiment with evolving features

## How Agent Teams Work

1. **Spawn teammates**: Launch multiple Claude Code sessions with specific roles
2. **Share task list**: All teammates see the same task board
3. **Self-coordinate**: Teammates claim tasks, update status, create new tasks
4. **Message each other**: Direct peer-to-peer communication
5. **Converge**: Work continues until all tasks are complete

**Key concepts:**
- **Lead agent**: Your main conversation that spawns the team
- **Teammates**: Independent Claude Code sessions with specialized roles
- **Task list**: Shared TODO board all teammates can read/write
- **Messages**: Direct communication between teammates
- **Convergence**: Team completes when all critical tasks are done

## Architecture Options

### Option 1: Specialized Teammates

Each teammate has a specific role and expertise:

```
Lead Agent
├── Backend Developer
├── Frontend Developer
├── Test Engineer
└── Documentation Writer
```

**Best for**: Feature development, system integration, large implementations

### Option 2: Research Team

Teammates investigate competing hypotheses:

```
Lead Agent
├── Researcher A (hypothesis 1)
├── Researcher B (hypothesis 2)
└── Researcher C (hypothesis 3)
```

**Best for**: Bug investigation, architectural decisions, optimization research

### Option 3: Review Team

Multiple reviewers check different aspects:

```
Lead Agent
├── Security Reviewer
├── Performance Reviewer
└── Test Coverage Reviewer
```

**Best for**: Code review, audit, quality assurance

## Spawn Agent Teams

### Manual Spawning

Explicitly describe the team structure:

```
Create an agent team to implement the new authentication system:
- Backend developer to implement the API endpoints
- Frontend developer to create the login UI
- Security reviewer to audit the implementation
- Test engineer to write comprehensive tests
```

### Automatic Detection

Claude recognizes when parallel work would benefit from a team:

```
Investigate why the checkout flow is failing and fix it
```

If the investigation requires exploring multiple hypotheses, Claude may suggest spawning a team.

### Programmatic Spawning

Via Agent SDK (for automation):

```javascript
const team = await spawnTeam({
  goal: "Implement feature X",
  teammates: [
    { role: "backend", skills: ["api-development"] },
    { role: "frontend", skills: ["react"] },
    { role: "testing", skills: ["test-automation"] }
  ]
});
```

## Task Management

### Task Structure

Each task has:
- **Subject**: Brief title (imperative form)
- **Description**: Detailed requirements
- **Status**: `pending`, `in_progress`, `completed`
- **Owner**: Which teammate claimed it (empty if available)
- **ActiveForm**: Present continuous shown in spinner
- **Dependencies**: `blocks`, `blockedBy` relationships

### Task Lifecycle

1. **Created**: Someone adds task to shared list
2. **Claimed**: Teammate sets themselves as owner
3. **In Progress**: Work begins
4. **Completed**: Work finishes, dependencies unblock
5. **New Tasks**: Often spawn follow-up tasks

### Task Coordination

**Self-coordination**: Teammates autonomously:
- Browse available tasks
- Claim tasks aligned with their role
- Update status as work progresses
- Create new tasks as needs arise
- Communicate blockers

**Lead responsibilities**: The lead agent:
- Creates initial task breakdown
- Monitors overall progress
- Facilitates communication
- Makes final decisions on conflicts

## Communication Patterns

### Peer-to-Peer Messaging

Teammates message each other directly:

```
@backend: I need the /api/users endpoint structure
@frontend: Here's the schema: { id, name, email, ... }
```

### Status Updates

Teammates broadcast progress:

```
Completed implementing the login API endpoint.
Ready for frontend integration.
```

### Asking for Help

```
@security-reviewer: Can you check my authentication flow?
Found a potential issue with session management.
```

### Sharing Findings

```
Discovered the bug is in the payment processor's webhook handler.
Created task to fix the race condition.
```

## Best Practices

### 1. Clear Role Definition

Give each teammate a specific expertise:

```
- Backend Developer: Implement server-side logic and APIs
- Frontend Developer: Create UI components and user interactions
- Test Engineer: Write and run comprehensive test suites
- Security Reviewer: Audit code for vulnerabilities
```

### 2. Well-Defined Initial Tasks

Break down the work upfront:

```
Tasks:
1. Design database schema
2. Implement API endpoints
3. Create frontend components
4. Write integration tests
5. Security audit
6. Performance testing
```

### 3. Skill Specialization

Assign relevant skills to teammates:

```yaml
Backend teammate:
  skills:
    - api-conventions
    - database-patterns
    - error-handling

Frontend teammate:
  skills:
    - component-library
    - state-management
    - accessibility
```

### 4. Regular Checkpoints

Periodic synchronization:

```
Every 5 tasks completed:
- Lead reviews progress
- Teammates report blockers
- Adjust priorities if needed
```

### 5. Clear Exit Criteria

Define when the team is done:

```
Team completes when:
✓ All features implemented
✓ Tests passing
✓ Security review completed
✓ Documentation updated
```

## Example: Feature Development Team

```
Goal: Implement user profile editing feature

Teammates:
1. Backend Developer
   - Create /api/profile endpoint
   - Add validation logic
   - Update database schema

2. Frontend Developer
   - Design profile edit form
   - Implement validation UI
   - Handle API integration

3. Test Engineer
   - Write unit tests
   - Write integration tests
   - Verify error handling

4. Documentation Writer
   - Update API docs
   - Write user guide
   - Add code examples

Initial Tasks:
□ Design database schema (backend)
□ Create API endpoint (backend)
□ Design UI mockup (frontend)
□ Implement form component (frontend)
□ Write backend tests (test)
□ Write frontend tests (test)
□ Update API docs (docs)
□ Write user guide (docs)
```

## Example: Bug Investigation Team

```
Goal: Find and fix the intermittent checkout failure

Teammates:
1. Payment Flow Researcher
   - Investigate payment processing
   - Check third-party integrations
   - Review error logs

2. Database Researcher
   - Check for race conditions
   - Review transaction handling
   - Analyze connection pooling

3. Frontend Researcher
   - Check client-side validation
   - Review timing issues
   - Test edge cases

Initial Tasks:
□ Reproduce the bug consistently
□ Analyze payment processor logs
□ Check database transactions
□ Review frontend validation
□ Test with different browsers
□ Document findings
```

## Example: Code Review Team

```
Goal: Review the new authentication system

Teammates:
1. Security Reviewer
   - Check for vulnerabilities
   - Review session management
   - Audit password handling

2. Performance Reviewer
   - Check query efficiency
   - Review caching strategy
   - Analyze bottlenecks

3. Code Quality Reviewer
   - Check test coverage
   - Review error handling
   - Verify documentation

Initial Tasks:
□ Security audit of auth endpoints
□ Check for SQL injection
□ Review session storage
□ Analyze query performance
□ Check caching strategy
□ Verify test coverage
□ Review error messages
□ Update documentation
```

## Monitoring Team Progress

### View Task Status

```
/tasks
```

Shows:
- All tasks with status
- Who owns each task
- Dependencies and blockers
- Overall progress

### Check Messages

Review teammate communication to understand:
- What's been discovered
- Current blockers
- Decisions made
- Help requested

### Intervene When Needed

As lead, you can:
- Create new tasks
- Reassign tasks
- Clarify requirements
- Resolve conflicts
- Adjust priorities

## Troubleshooting

**Team not starting:**
- Verify `CLAUDE_CODE_AGENT_TEAMS_ENABLED=1` is set
- Check Claude Code version supports agent teams
- Try restarting terminal after setting environment variable

**Teammates not coordinating:**
- Ensure initial tasks are clear and well-defined
- Check that roles are distinct and non-overlapping
- Review task dependencies for circular blocks

**Context overflow:**
- Agent teams are independent sessions, so no shared context limits
- Each teammate has their own context window
- Use subagents within teammates for high-volume operations

**Tasks not completing:**
- Check for blocked tasks (dependencies not resolved)
- Verify teammates can claim available tasks
- Review if tasks are too vague or too large

**Communication issues:**
- Teammates see all messages but may miss specific @mentions
- Important decisions should be reflected in task descriptions
- Lead can relay messages if teammates miss communications

## Limitations

**Current limitations** (experimental feature):
- Teams may take longer than single-agent solutions
- Coordination overhead increases with team size
- More tokens consumed (separate Claude instances)
- Not available in all environments
- API and behavior may change

**Recommended limits:**
- 3-5 teammates per team (more = more coordination overhead)
- 10-20 tasks initially (can grow as work progresses)
- Single level of teams (no nested team spawning)

## Cost Considerations

Agent teams consume more tokens because:
- Each teammate is a full Claude Code session
- Multiple concurrent conversations
- Coordination messages
- Task updates and status changes

**When the cost is worth it:**
- Complex tasks requiring deep expertise
- Parallel investigation saves significant time
- Quality improvements from multiple reviewers
- Tasks that would otherwise require multiple sequential sessions

**When to avoid:**
- Simple, focused tasks
- Limited budget or token constraints
- Tasks with clear single-threaded execution

## Advanced Patterns

### Hierarchical Teams

Lead spawns teams, teammates spawn subagents:

```
Lead Agent
├── Backend Team Lead
│   ├── API Developer (subagent)
│   └── Database Developer (subagent)
└── Frontend Team Lead
    ├── UI Developer (subagent)
    └── UX Reviewer (subagent)
```

### Dynamic Team Scaling

Start small, grow as needed:

```
Initial: Lead + 2 researchers
If complex: Spawn specialist teammates
If blocked: Add reviewer teammate
```

### Cross-Team Communication

Multiple teams working on related features:

```
Team A: Authentication
Team B: Authorization
Coordination: Shared task list for integration
```

## Related Features

- Use `/create-agent` for specialized subagents
- Use `/create-skill` to give teammates domain knowledge
- Use task management tools for coordination
- Check Agent SDK for programmatic control

## Additional Resources

- Official docs: https://code.claude.com/docs/en/agent-teams
- Agent SDK: https://platform.claude.com/docs/en/agent-sdk/overview
- Task system documentation
- Understanding context windows and token costs

## Quick Start

1. **Enable agent teams:**
   ```bash
   export CLAUDE_CODE_AGENT_TEAMS_ENABLED=1
   ```

2. **Spawn a simple team:**
   ```
   Create an agent team to implement the new feature:
   - One teammate to write the code
   - One teammate to write tests
   - One teammate to review both
   ```

3. **Monitor progress:**
   ```
   /tasks
   ```

4. **Let them work:**
   - Teammates coordinate autonomously
   - Intervene only when needed
   - Review final results when all tasks complete

Remember: Agent teams are experimental. Start with simple teams and scale up as you learn the patterns that work for your use cases.
