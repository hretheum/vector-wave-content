# Vibe Coding Deep Dive: The Brutal Reality of Managing AI Agents with 456 Lines of Kanban

## CZĘŚĆ 1: THE SETUP (z realnymi danymi)

### The Kanban That Controls Everything

Real numbers from `/Users/hretheum/dev/obsidian-vault/kanban/obsidian-plugin.md`:
- 687 total lines
- 10 EPICs
- 44+ Tasks  
- 440+ Subtasks
- Each with Standards Compliance Gates

Not theoretical. This is production.

### Project Structure That Emerged

```
/Users/hretheum/dev/bezrobocie/obsidian/
├── .claude/          # AI agent instructions
├── docs/             # 12 auto-generated standards
│   ├── CODE_STANDARDS.md
│   ├── SECURITY_GUIDELINES.md
│   ├── TESTING_STRATEGY.md
│   └── [9 more standards files]
├── src/              # Actual plugin code
├── coverage/         # Auto-generated test reports
└── CLAUDE.md         # The brain of /nakurwiaj
```

## CZĘŚĆ 2: THE AGENTS (Claude Code Reality)

### Meet the /nakurwiaj Command

What /nakurwiaj ACTUALLY does:

1. **Task Discovery Phase:**
   - Searches for Task in Kanban
   - Priority: in progress → todo → backlog
   - Loads validation criteria

2. **Subtask Execution Loop:**
   - Execute atomic action
   - Run `/agent code-reviewer`
   - Apply corrections IMMEDIATELY
   - Git commit with conventional format
   - Auto-continue to next

3. **Task Completion:**
   - Final `/agent code-reviewer` pass
   - `/agent qa` validation
   - Mark as done in Kanban
   - STOP (crucial - no drift)

### The Full Agent Arsenal

Claude Code Agents in Play:

```bash
/nakurwiaj <Task>        # Main executor
/agent code-reviewer     # Quality gate after EACH subtask
/agent qa               # Final validation
/agent test-writer      # Generates test suites
/agent doc-writer       # Creates documentation
/agent security-scanner # Checks for vulnerabilities
```

### The Uncomfortable Truth About Agents

What They DON'T Tell You:

**1. Agents DUPLICATE tasks constantly**
- Same subtask executed 2-3 times
- Have to manually clean git history
- Solution: Explicit STOP commands

**2. Context drift is REAL**
- After 10 subtasks, they forget the goal
- Start "improving" things not in scope
- Solution: Atomic tasks, hard boundaries

**3. They WILL hallucinate structure**
- Create files in wrong directories
- Invent new naming conventions
- Solution: File path validation in EVERY task

Example from real execution:
```
Task 2.1.1: Define DomainEvent interface
- First attempt: Created in /src/events/
- Second attempt: Created in /core/events/
- Third attempt: Created in /src/core/domain/
- Finally: Explicit path in task definition
```

## CZĘŚĆ 3: STANDARDS THAT ACTUALLY WORK

### Generated Standards Overview

12 Standards Files Generated in 90 minutes:
1. `CODE_STANDARDS.md` - TypeScript strict, Result<T,E> pattern
2. `SECURITY_GUIDELINES.md` - XSS, path traversal, input validation
3. `TESTING_STRATEGY.md` - >90% core, >80% UI coverage
4. `ERROR_HANDLING_STANDARDS.md` - No exceptions, only Results
5. `PERFORMANCE_BENCHMARKS.md` - <100ms analyzers, <50ms UI
6. `LLM_DEVELOPMENT_GUARDRAILS.md` - Prompt injection prevention
7. `DOCUMENTATION_STANDARDS.md` - JSDoc requirements
8. `PROJECT_GOVERNANCE.md` - Decision framework
9. `DEVELOPMENT_METHODOLOGY.md` - Workflow patterns
10. `CODE_REVIEW_PROCESSES.md` - Review checklists
11. `STANDARDS_IMPLEMENTATION_AUDIT.md` - Compliance tracking
12. `TASK_VALIDATION_CRITERIA.md` - Success metrics

### How Standards Propagate

Real Example: Task 2.5 (Input Validator)

BEFORE standards:
```
Task 2.5: Input Validator with security
- Generic implementation
- No specific requirements
```

AFTER standards propagation:
```
Task 2.5: Input Validator with security
├── 2.5.1: Implement XSS prevention (→ SECURITY_GUIDELINES.md)
├── 2.5.2: Path traversal protection (→ SECURITY_GUIDELINES.md)
├── 2.5.3: Filename sanitization (→ SECURITY_GUIDELINES.md)
├── Standards: Security validation
├── Standards: Testing >90% coverage
├── Standards: Result<T,E> pattern
└── Standards: Performance <10ms per validation
```

## CZĘŚĆ 4: FAILURES & FIXES

### What Actually Went Wrong

Week 1 Failures (and recoveries):

**Failure 1: The Great Duplication Disaster**
- Task 1.1: Create folder structure
- Agent created it 3 times
- Different structures each time
- Git history was a mess
- **FIX:** Added idempotency checks to all creation tasks

**Failure 2: The Config File Wars**
- Task 1.3: Setup ESLint, Prettier, Jest
- Agent "improved" config with each subtask
- Ended with 5 different prettier configs
- Tests wouldn't run
- **FIX:** Locked configs after first creation

**Failure 3: The Import Hell**
- Task 2.1: EventBus implementation
- Circular dependencies everywhere
- Agent kept "fixing" by adding more imports
- **FIX:** Explicit import rules in standards

**Failure 4: The Test Coverage Lie**
- Agent claimed 95% coverage
- Reality: 62%
- Was testing its own test utilities
- **FIX:** Independent coverage verification

## CZĘŚĆ 5: IMPLEMENTATION GUIDE

### Your First Vibe Coding Project

**Prerequisites:**
- Claude API access (or OpenAI)
- Obsidian with Kanban plugin (or any task manager)
- 2 hours for setup
- High tolerance for agent babysitting

**Step 1: Create Your Kanban Structure**

Start with this template:

```markdown
EPIC 1: Foundation
├── Task 1.0: Standards Definition
│   ├── 1.0.1: Development methodology
│   ├── 1.0.2: Code standards
│   └── 1.0.3: Testing strategy
├── Task 1.1: Project setup
│   ├── 1.1.1: Create src/ directory
│   ├── 1.1.2: Create docs/ directory
│   └── 1.1.3: Initialize git
```

**CRITICAL:** Each subtask must be:
- Single action ("Create X", "Define Y")
- Has explicit output ("file created", "interface defined")
- No ambiguity ("implement feature" = BAD, "create user.ts with User interface" = GOOD)

### The Real /nakurwiaj Implementation

What you tell people:
```bash
/nakurwiaj Task 1.0
```

What actually happens:
```bash
/nakurwiaj Task 1.0
# Agent starts...
# Creates 3 duplicate files
# "Wait, stop, clean that up"
/agent reset
/nakurwiaj Task 1.0 --no-creativity --follow-the-fucking-plan
# 20 minutes later...
# Success! (mostly)
```

## CZĘŚĆ 6: THE ECONOMICS (Real Numbers)

### API Costs for Full Project

July 18 - August 14, 2025:

**Claude API usage:**
- Input tokens: ~2.5M ($37.50)
- Output tokens: ~8M ($120.00)
- Total API cost: $157.50

**Human time:**
- Initial planning: 2 hours
- Agent babysitting: 14 hours
- Cleanup/fixes: 4 hours
- Total: 20 hours

**Traditional approach estimate:**
- 260 hours × $150 = $39,000

**ROI: $38,842.50 saved (247x return)**
Even with babysitting.

## CZĘŚĆ 7: HARD TRUTHS

### This is NOT "Set and Forget"

Reality Check:

❌ "AI writes perfect code automatically"
✅ "AI writes decent code with constant supervision"

❌ "Just give it the task list"
✅ "Give it atomic tasks and watch like a hawk"

❌ "Replaces junior developers"
✅ "Amplifies senior developers who can spot bullshit"

❌ "Works for any project"
✅ "Works for well-understood, structured projects"

### When This Actually Works

**PERFECT FOR:**
- Boilerplate generation
- Standard patterns implementation
- Test suite creation
- Documentation
- CRUD operations
- Config files

**TERRIBLE FOR:**
- Novel algorithms
- Complex business logic
- UI/UX decisions
- Architecture choices
- Performance optimization
- Debugging legacy code

## CZĘŚĆ 8: CALL TO ACTION

### Try This Tomorrow

1. Pick ONE boring task you've been avoiding
2. Break it into 10 subtasks (each <30 min)
3. Run through AI with explicit boundaries
4. Measure: Time saved, errors introduced, cleanup needed
5. Share results: #VibeCoding

**Real examples to start:**
- Test suite for existing code
- API documentation
- Database migrations
- CI/CD pipeline setup
- Logging implementation

### Resources

- GitHub Repo: [coming soon - cleaning up the mess first]
- Kanban Template: [link]
- Standards Templates: [link]
- /nakurwiaj implementation: [gist]

Join the experiment. Share the failures.
This is how we learn.

## KOŃCOWA MYŚL

Vibe Coding isn't magic. 
It's systematic babysitting of AI agents.

But 20 hours of babysitting beats 260 hours of grinding.

The tools are dumb. The process is smart.
The savings are real. The struggle is too.

Welcome to engineering in 2025.
Where we manage robots like junior devs,
and somehow, it actually fucking works.

---

*Eryk Orłowski is a Design Leader with 20+ years of experience, now applying design thinking to AI development. Currently building Vector Wave. This is the real story of the Obsidian Smart Slides plugin development - with all the failures, duplications, and agent babysitting included.*

*Want the uncensored git history? Find me on [X @eofek](https://twitter.com/eofek) or [LinkedIn](https://linkedin.com/in/erykorlowski). I'll show you the commits where the agent argued with itself.*