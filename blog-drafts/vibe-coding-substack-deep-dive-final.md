# Vibe Coding Deep Dive: The Brutal Reality of Managing AI Agents with 456 Lines of Kanban

*Or: How I learned to stop worrying and love babysitting robots*

Last month's blog post about replacing 3 weeks of enterprise standards creation with 2 hours of planning sparked more questions than it answered. "Show us the actual implementation," you said. "What about when agents fail?" you asked. "This sounds too good to be true," you accused.

You were right to be skeptical.

This is the unfiltered story of building an Obsidian plugin with AI agents—complete with duplicated tasks, hallucinated file structures, and the 20 hours I spent babysitting robots to save 240 hours of human work. Welcome to the messy reality of Vibe Coding.

## The Kanban That Rules Them All

Let me start with the actual numbers. The Kanban board controlling this project lives at `/Users/hretheum/dev/obsidian-vault/kanban/obsidian-plugin.md`. It's 687 lines of meticulously structured tasks. Not 456 like I initially said—that was before the standards propagation added compliance gates to everything.

Here's what those 687 lines break down to: 10 EPICs describing major feature areas, 44 primary tasks with specific deliverables, and over 440 atomic subtasks. Each subtask follows a rigid pattern: single action, measurable outcome, less than 2 hours to complete. No ambiguity allowed.

The project structure that emerged from this planning looks deceptively simple:

```
/Users/hretheum/dev/bezrobocie/obsidian/
├── .claude/          # AI agent instructions
├── docs/             # 12 auto-generated standards
├── src/              # Actual plugin code
├── coverage/         # Auto-generated test reports
└── CLAUDE.md         # The brain of the operation
```

But that simplicity hides the chaos of getting here. Let me show you what actually happens when you unleash AI agents on a codebase.

## Meet the Real Agents (They're Not What You Think)

The `/nakurwiaj` command isn't just a funny Polish profanity ("let's fucking do this"). It's an orchestration pattern I've refined across multiple projects. But here's the thing—it doesn't exist as a single agent. It's a choreographed chain of specialized Claude Code agents, each with a specific role.

Let me show you the actual agent definitions from my latest project, Detektor (a computer vision system). These same patterns power the Obsidian plugin:

### The Main Executor: detektor-coder

This agent (291 lines of configuration) is the workhorse. It doesn't just write code—it enforces an entire philosophy:

```markdown
## Podstawowe zasady projektu
- **TDD ZAWSZE**: Test first, implementation second
- **Observability-first**: Każdy serwis ma OpenTelemetry, Prometheus metrics
- **Clean Architecture**: Separacja warstw (domain, infrastructure, application)
- **Event-driven**: Komunikacja przez Redis Streams
- **Type hints**: 100% pokrycie w Python 3.11+
- **No hardcoded values**: Wszystko przez config/env
```

When it creates a new service, it doesn't freestyle. It follows a 9-point checklist:
- Unit tests >80% coverage
- Integration tests for all endpoints
- Health check endpoint working
- Metrics endpoint returning Prometheus format
- OpenTelemetry tracing integrated
- Dockerfile optimized (multi-stage)
- README.md with run instructions
- Structural logging with correlation ID
- Error handling for all edge cases

### The Quality Gate: code-reviewer

After EVERY atomic task, the code-reviewer agent (111 lines) runs automatically. It's not looking for style issues—it's hunting for architectural violations:

```python
# ❌ What it catches:
def process_data(d):  # Unclear naming
    result = []
    for item in d:  # No type hints
        if item['status'] == 'active':  # Magic string
            result.append(item['value'] * 1.2)  # Magic number
    return result

# ✅ What it demands:
from typing import List, Dict, Any
from enum import Enum

class ItemStatus(Enum):
    ACTIVE = "active"
    INACTIVE = "inactive"

TAX_MULTIPLIER = 1.2

def calculate_active_items_with_tax(
    items: List[Dict[str, Any]]
) -> List[float]:
    """Calculate values with tax for active items."""
    return [
        item['value'] * TAX_MULTIPLIER
        for item in items
        if ItemStatus(item['status']) == ItemStatus.ACTIVE
    ]
```

If it finds critical issues, it doesn't ask for permission—it loops back to the coder agent with specific fix requirements. Maximum 3 iterations before it escalates to human intervention.

### The Chain of Command

Here's the actual flow from the Detektor project that I've adapted for all my AI development:

```markdown
## Automatyczny łańcuch agentów

IF task contains "implement|create|add":
    → /agent detektor-coder
ELIF task contains "debug|fix|investigate":
    → /agent debugger
ELIF task contains "refactor|optimize":
    → /agent architecture-advisor → detektor-coder
ELIF task contains "deploy|deployment":
    → /agent deployment-specialist

AFTER each atomic task:
    → /agent code-reviewer
    IF critical_issues:
        → Loop back with feedback
        → Auto-fix without asking
```

This isn't theoretical. I have 8 specialized agents working in concert:
- **architecture-advisor**: System design decisions
- **code-reviewer**: Quality gates
- **debugger**: Problem investigation
- **deployment-specialist**: CI/CD and production
- **detektor-coder**: Implementation
- **documentation-keeper**: Keeps all docs in sync
- **pipeline-debugger**: CI/CD troubleshooting
- **pisarz**: Polish documentation writer

## The Uncomfortable Truth About Agent Failures

Let me tell you what really happens when you run `/nakurwiaj Task 2.1`.

The agent starts confidently. Creates the EventBus interface. Then creates it again in a different location because it "realized" the first location wasn't optimal. Then creates a third version because it decided to "improve" the pattern.

Your git history looks like this:
```
feat: implement EventBus interface
fix: move EventBus to correct location
refactor: optimize EventBus pattern
fix: revert EventBus optimization
feat: implement EventBus interface (final)
```

Five commits for one task. And that's a good day.

The worst I've seen? An agent that got into an argument with itself in the code comments:

```python
# TODO: This should use async/await
# NOTE: Actually, synchronous is fine here for simplicity
# FIXME: No, async is required for scalability
# HACK: Using sync for now, will refactor later
# BUG: This doesn't work with sync
# TODO: Convert to async/await
```

Forty lines of self-debate. In production code.

### The Duplication Disaster

Task 1.1 asked to create the folder structure. Simple, right? The agent created:
- `/src/` with one organization
- `/source/` with a different organization
- `/app/src/` with a third organization

All in separate commits. All "correctly" implemented according to its evolving understanding of best practices.

The solution? Explicit path requirements in EVERY task:

```markdown
Task 1.1.1: Create src/ directory at project root
Path: /Users/hretheum/dev/bezrobocie/obsidian/src/
Not: /source/, /app/src/, or any other variation
```

### Context Drift Is Real

After about 10 subtasks, agents develop amnesia. They forget the project structure. They forget the coding standards. They start "improving" things that don't need improvement.

I watched an agent refactor a working authentication system three times in one session because it kept forgetting it had already "optimized" it. Each refactor introduced new bugs while fixing imaginary problems.

The solution? Hard stops after each task:

```markdown
Task Completion Phase:
- Mark task complete in Kanban
- Git commit with conventional message
- STOP - Do not continue to next task
- Clear context
- Fresh start for next task (I know, horrible idea to introduce the context every time)
```

## Standards That Actually Work (Through Brute Force)

After the chaos of the first week, I realized I needed standards that agents couldn't ignore or creatively interpret. Task 1.0 generated twelve standards documents in 90 minutes:

```
docs/
├── CODE_STANDARDS.md           # TypeScript strict, Result<T,E> pattern
├── SECURITY_GUIDELINES.md      # XSS, path traversal, input validation
├── TESTING_STRATEGY.md         # >90% core, >80% UI coverage
├── ERROR_HANDLING_STANDARDS.md # No exceptions, only Results
├── PERFORMANCE_BENCHMARKS.md   # <100ms analyzers, <50ms UI
├── LLM_DEVELOPMENT_GUARDRAILS.md # Prompt injection prevention
└── [6 more standards files]
```

But here's the key: these standards propagate through the task structure itself. Look at how Task 2.5 evolved after standards implementation:

```markdown
Task 2.5: Input Validator with security
├── 2.5.1: Implement XSS prevention
├── 2.5.2: Add path traversal protection
├── 2.5.3: Create filename sanitization
├── 2.5.4: Add input length validation
├── 2.5.5: Implement content type validation
├── 2.5.6: Create security test suite
├── Standards: Security (→ docs/SECURITY_GUIDELINES.md)
├── Standards: Testing >90% coverage
├── Standards: Code Quality & TS strict
├── Standards: Performance <10ms per validation
```

Each subtask now has exit criteria that the code-reviewer agent enforces. No interpretation. No creativity. Just compliance.

## The Real Economics (Including Hidden Costs)

Let's talk real numbers from the July 18 - August 14, 2025 period:

**AI API Costs (don't make me confirm that, it's Anthropic Claude's research):**

- Claude API: ~2.5M input tokens ($37.50)
- Claude API: ~8M output tokens ($120.00)
- Failed attempts: ~1M tokens ($15.00)
- Total API cost: $172.50

**Human Time Investment:**
- Initial Kanban creation: 2 hours
- Standards definition: 1 hour
- Agent babysitting: 14 hours
- Fixing agent disputes: 2 hours
- Emergency interventions: 2 hours

**Traditional Approach (based on 3 similar projects):**
- Standards creation: 80-120 hours
- Implementation: 140-180 hours
- Testing & debugging: 40-60 hours
- Total: 260-360 hours

Even with all the babysitting, cleanup, and frustration, it's still 10x cheaper.

## When This Actually Works (And When It Spectacularly Doesn't)

### Perfect For:
- **Boilerplate Generation**: CRUD endpoints, service templates
- **Standard Patterns**: Repository pattern, dependency injection
- **Test Generation**: Unit tests for existing code
- **Documentation**: API docs, README files
- **Configuration**: Docker, CI/CD, environment setup
- **Refactoring**: Applying consistent patterns across codebase

### Absolute Disasters For:
- **Novel Algorithms**: Agents will create verbose, inefficient solutions
- **UI/UX Design**: They have no taste or user empathy
- **Performance Optimization**: They make things worse, consistently
- **Debugging Complex Issues**: They fix symptoms, not causes
- **Architecture Decisions**: They over-engineer everything (well, I'm the one to blame here probably, as I keep trying to setup so many guardrails for agents)
- **Business Logic**: They don't understand domain complexity

## The Documentation Keeper: My Secret Weapon

One agent I haven't seen others talk about: the documentation-keeper. This agent's ONLY job is maintaining consistency across all project documents:

```markdown
BEFORE moving to next block:
→ /agent documentation-keeper
→ Synchronize:
  - Task checkboxes [x] in decomposition files
  - Service ports in PROJECT_CONTEXT.md
  - Status in architecture document
  - New problems in TROUBLESHOOTING.md
  - Service READMEs if new services added
→ Verify all docs are consistent
```

This prevents the classic problem where your code works but your documentation describes a completely different system. The agent runs after EVERY task block and fixes inconsistencies automatically.

## The Hard Truth About AI-Assisted Development

This is not the future we were promised. We were told AI would replace programmers. Instead, we got tools that require constant supervision, produce verbose mediocrities, and turn senior developers into robot babysitters.

But here's the thing: 24 hours of babysitting beats 300 hours of grinding. The tools are dumb, but they're fast. The process is messy, but it works. The savings are real, even if the marketing is bullshit.

I'm not building elegant cathedrals of code. I'm running a factory where robots stamp out functional components while I make sure they don't burn the place down.

Some days, an agent will create something genuinely clever, and I'll think we're living in the future. Then it will spend 30 minutes trying to import a module that doesn't exist, and I'll remember we're barely out of the stone age.

## What I'm Actually Doing Next

I'm open-sourcing everything, as usually in my vibe coding projects. Not just the code—the entire process:
- The 687-line Kanban board
- All 12 standards documents
- The agent configurations
- The git history with all the embarrassing commits (Claude, pls, horrible idea again)
- The TROUBLESHOOTING.md with 47 "WTF moments"

Why? Because the only way this gets better is if we stop pretending it's magic and start sharing what actually works.

If you want to try Vibe Coding yourself, start here:

1. **Pick ONE boring, repetitive task** (not your entire project)
2. **Spend more time on task definition than you think necessary** (2x minimum)
3. **Write explicit paths, types, and examples** (agents are literal)
4. **Set up quality gates BEFORE starting** (code-reviewer agent or equivalent)
5. **Expect 30% failure rate** (budget time for cleanup)
6. **Track everything** (time, tokens, errors, rework)

Then share your results. Use #VibeCoding. Include the failures.

## The Future Is Already Here (It's Just Poorly Distributed)

The tools will get better. Claude 3.5, GPT-4.5, whatever comes next. But the fundamental challenge remains: turning human intent into working code through an intermediary that has no real understanding of either.

We're not replacing programmers. We're creating a new discipline: robot programming managers. People who can decompose problems into atomic tasks, set up quality gates, and babysit artificial intelligences that are simultaneously brilliant and stupid.

Welcome to engineering in 2025, where we manage robots like junior developers who can type at superhuman speed but have the architectural sense of a goldfish and the attention span to match.

Somehow, despite all of this, it actually fucking works. And that might be the most surprising thing of all.

---

*Eryk Orłowski is a Design Leader with 20+ years of experience who now spends his time teaching robots how to code, cleaning up their messes, and occasionally marveling that any of this works at all. Currently building Vector Wave and wondering why he thought this was a good idea.*

*Want the full agent configurations and the git history where an AI had an existential crisis about TypeScript generics? Find me on [X @eofek](https://twitter.com/eofek) or [LinkedIn](https://linkedin.com/in/eofek). Bring your own stories of AI chaos—misery loves company.*

*GitHub repo with everything: [github.com/hretheum/obsidian-smart-slides](https://github.com/hretheum/obsidian-smart-slides) (cleanup in progress, judge gently)*