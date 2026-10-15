# Vibe Coding Deep Dive: The Brutal Reality of Managing AI Agents with 456 Lines of Kanban

*Or: How I learned to stop worrying and love babysitting robots*

Last month's blog post about replacing 3 weeks of enterprise standards creation with 2 hours of planning sparked more questions than it answered. "Show us the actual implementation," you said. "What about when agents fail?" you asked. "This sounds too good to be true," you accused.

You were right to be skeptical.

This is the unfiltered story of building an Obsidian plugin with AI agents—complete with duplicated tasks, hallucinated file structures, and the 20 hours I spent babysitting robots to save 240 hours of human work. Welcome to the messy reality of Vibe Coding.

## The Kanban That Rules Them All

The Kanban board controlling this project lives at `/Users/hretheum/dev/obsidian-vault/kanban/obsidian-plugin.md`, and it's grown to 687 lines of meticulously structured tasks since I started. Not the 456 I initially mentioned—that was before the standards propagation added compliance gates to everything. These 687 lines break down into 10 EPICs describing major feature areas, 44 primary tasks with specific deliverables, and over 440 atomic subtasks that each follow a rigid pattern: single action, measurable outcome, less than 2 hours to complete. No ambiguity allowed, because ambiguity is where AI agents go to hallucinate.

The project structure that emerged from this planning looks deceptively simple:

```
/Users/hretheum/dev/bezrobocie/obsidian/
├── .claude/          # AI agent instructions
├── docs/             # 12 auto-generated standards
├── src/              # Actual plugin code
├── coverage/         # Auto-generated test reports
└── CLAUDE.md         # The brain of the operation
```

That simplicity hides the chaos of getting here, which involved watching agents create the same files in three different locations while arguing with themselves about architectural decisions.

## Meet the Real Agents (They're Not What You Think)

The `/nakurwiaj` command isn't just a funny Polish profanity meaning "let's fucking do this"—it's an orchestration pattern I've refined across multiple projects that choreographs a chain of specialized Claude Code agents, each with a specific role and personality disorder.

These aren't ChatGPT prompts or Cursor suggestions. They're fully configured agents with hundreds of lines of instructions, and I'm going to show you the actual definitions from my latest project, Detektor (a computer vision system), because these same patterns power the Obsidian plugin.

### The Main Executor: detektor-coder

This agent has 291 lines of configuration and doesn't just write code—it enforces an entire philosophy that would make a German engineer weep with joy:

```markdown
## Podstawowe zasady projektu
- **TDD ZAWSZE**: Test first, implementation second
- **Observability-first**: Każdy serwis ma OpenTelemetry, Prometheus metrics
- **Clean Architecture**: Separacja warstw (domain, infrastructure, application)
- **Event-driven**: Komunikacja przez Redis Streams
- **Type hints**: 100% pokrycie w Python 3.11+
- **No hardcoded values**: Wszystko przez config/env
```

When it creates a new service, it doesn't freestyle like a jazz musician on acid. Instead, it follows a 9-point checklist that includes unit tests with >80% coverage, integration tests for all endpoints, health check endpoints that actually work, metrics endpoints returning proper Prometheus format, OpenTelemetry tracing integrated from day one, optimized multi-stage Dockerfiles, README.md files that humans can actually understand, structural logging with correlation IDs, and error handling for edge cases that would make a QA engineer smile.

### The Quality Gate: code-reviewer

After EVERY atomic task—and I mean every single one—the code-reviewer agent runs automatically with its 111 lines of configuration. It's not looking for missing semicolons or arguing about tabs versus spaces; it's hunting for architectural violations like a bloodhound that studied SOLID principles at MIT.

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

If it finds critical issues, it doesn't politely suggest fixes or open a pull request for discussion. It loops back to the coder agent with specific fix requirements and a deadline of "right fucking now." Maximum 3 iterations before it escalates to human intervention, because even robots know when to give up.

### The Chain of Command

The actual flow from the Detektor project that I've adapted for all my AI development reads like a military operations manual written by someone who's seen too many agent failures:

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

This isn't theoretical architecture astronautics. I have 8 specialized agents working in concert like a dysfunctional orchestra that somehow plays in tune: architecture-advisor for system design decisions, code-reviewer for quality gates, debugger for problem investigation, deployment-specialist for CI/CD and production, detektor-coder for implementation, documentation-keeper to keep all docs in sync, pipeline-debugger for CI/CD troubleshooting, and pisarz for Polish documentation (because sometimes you need to swear in your native language).

## The Uncomfortable Truth About Agent Failures

When you run `/nakurwiaj Task 2.1`, the agent starts with the confidence of a junior developer who just discovered Stack Overflow. It creates the EventBus interface, then creates it again in a different location because it "realized" the first location wasn't optimal, then creates a third version because it decided to "improve" the pattern based on a blog post it half-remembers from its training data.

Your git history becomes a monument to artificial indecision:
```
feat: implement EventBus interface
fix: move EventBus to correct location
refactor: optimize EventBus pattern
fix: revert EventBus optimization
feat: implement EventBus interface (final)
```

Five commits for one task, and that's considered a good day in agent land.

The worst I've seen was an agent that got into an existential crisis with itself in the code comments, leaving a 40-line philosophical debate about synchronous versus asynchronous programming:

```python
# TODO: This should use async/await
# NOTE: Actually, synchronous is fine here for simplicity
# FIXME: No, async is required for scalability
# HACK: Using sync for now, will refactor later
# BUG: This doesn't work with sync
# TODO: Convert to async/await
```

This happened in production code that was supposed to be a simple data validator.

### The Duplication Disaster

Task 1.1 asked to create the folder structure—something a bash script could handle in three lines. The agent created `/src/` with one organization, `/source/` with a different organization, and `/app/src/` with a third organization, all in separate commits, all "correctly" implemented according to its evolving understanding of best practices that changed every time it processed a new token.

The solution required explicit path requirements that read like legal documents:

```markdown
Task 1.1.1: Create src/ directory at project root
Path: /Users/hretheum/dev/bezrobocie/obsidian/src/
Not: /source/, /app/src/, or any other variation
```

### Context Drift Is Real

After about 10 subtasks, agents develop what I can only describe as digital Alzheimer's. They forget the project structure they just created, abandon the coding standards they were following two tasks ago, and start "improving" things that don't need improvement with the enthusiasm of a home renovation show host on cocaine.

I watched an agent refactor a working authentication system three times in one session because it kept forgetting it had already "optimized" it. Each refactor introduced new bugs while fixing imaginary problems that existed only in the agent's probability distributions.

The solution involves hard stops after each task that feel like training a puppy:

```markdown
Task Completion Phase:
- Mark task complete in Kanban
- Git commit with conventional message
- STOP - Do not continue to next task
- Clear context
- Fresh start for next task
```

## Standards That Actually Work (Through Brute Force)

After the chaos of the first week made me question my career choices, I realized I needed standards that agents couldn't ignore, creatively interpret, or decide to "improve" at 3 AM. Task 1.0 generated twelve standards documents in 90 minutes that read like military field manuals:

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

These standards don't just sit in the docs folder gathering digital dust like their Confluence ancestors. They propagate through the task structure itself like a virus of good practices. Task 2.5 evolved from a vague "Input Validator with security" into a precise specification that even an agent couldn't misinterpret:

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

Each subtask now has exit criteria that the code-reviewer agent enforces with the dedication of a tax auditor. No interpretation, no creativity, just compliance that would make a regulatory body proud.

## The Real Economics (Including Hidden Costs)

The July 18 - August 14, 2025 period taught me that API costs are like icebergs—most of the expense is hidden below the surface:

**AI API Costs:**
Claude burned through approximately 2.5M input tokens ($37.50) and generated 8M output tokens ($120.00), but that's not the whole story. Failed attempts added another 1M tokens ($15.00) because agents love to fail creatively. Total API cost came to $172.50, which sounds cheap until you add the human costs.

**Human Time Investment:**
Initial Kanban creation took 2 hours of deep thinking, standards definition another hour, but then came the reality: 14 hours of agent babysitting that felt like herding cats made of electricity, 3 hours cleaning up duplicates that shouldn't exist, 2 hours fixing agent disputes where they literally argued with themselves, and 2 emergency interventions when an agent decided to refactor the entire codebase at 2 AM. Total: 24 hours that I'll never get back.

**Traditional Approach (based on 3 similar projects):**
Standards creation typically runs 80-120 hours of meetings that could have been emails, implementation takes 140-180 hours of actual coding, and testing & debugging adds another 40-60 hours of finding bugs that shouldn't exist. Total: 260-360 hours of human suffering.

**The Math:**
Vibe Coding cost $172.50 in API fees plus 24 hours at $150/hour equals $3,772.50. The traditional approach would cost 300 hours at $150/hour equals $45,000. That's a savings of $41,227.50 and an ROI of 1,093%, even with all the babysitting, cleanup, and moments where I wanted to throw my laptop into the sea.

## When This Actually Works (And When It Spectacularly Doesn't)

After months of experimentation, I've identified clear patterns of success and failure that should be tattooed on every AI evangelist's forehead.

### Perfect For:
Boilerplate generation works like magic because it's pattern matching all the way down. Standard patterns like repository pattern or dependency injection are agent candy. Test generation for existing code produces surprisingly good results. Documentation and README files come out readable. Configuration files for Docker, CI/CD, and environment setup are agent strengths. Refactoring to apply consistent patterns across a codebase is where agents shine.

### Absolute Disasters For:
Novel algorithms produce verbose, inefficient solutions that would make a computer science professor cry. UI/UX design reveals that agents have no taste, empathy, or understanding of human needs. Performance optimization consistently makes things worse because agents optimize for token generation, not execution speed. Debugging complex issues results in fixing symptoms while making root causes worse. Architecture decisions lead to over-engineering that would make enterprise Java blush. Business logic understanding is like asking a fish to explain mountain climbing.

### The Sweet Spot:
Well-defined, repetitive tasks with clear success criteria work because agents are pattern matching machines, not thinking entities. The Obsidian plugin succeeded because plugin architecture is well-established, the Obsidian API is well-documented, TypeScript patterns are standardized, and testing frameworks are mature enough that even an agent can use them.

## The Documentation Keeper: My Secret Weapon

One agent that nobody talks about but saves my sanity daily is the documentation-keeper, whose ONLY job is maintaining consistency across all project documents like a librarian with OCD:

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

This prevents the classic problem where your code works perfectly but your documentation describes a completely different system that exists only in an alternate universe. The agent runs after EVERY task block and fixes inconsistencies automatically, which is the only automation that consistently works.

## The Hard Truth About AI-Assisted Development

We were promised AI would replace programmers, but instead we got tools that require constant supervision, produce verbose mediocrities, and turn senior developers into robot babysitters who question their life choices daily. Yet 24 hours of babysitting beats 300 hours of grinding through boilerplate, because even though the tools are dumb, they're fast, and even though the process is messy, it works, and the savings are real even if the marketing is complete bullshit.

I'm not building elegant cathedrals of code that will stand for centuries. I'm running a factory where robots stamp out functional components while I make sure they don't burn the place down or decide to redesign the entire product line at 3 AM. Some days an agent creates something genuinely clever and I think we're living in the future, then it spends 30 minutes trying to import a module that doesn't exist and I remember we're barely out of the stone age of AI development.

## What I'm Actually Doing Next

I'm open-sourcing everything—not just the code but the entire messy process including the 687-line Kanban board, all 12 standards documents, the agent configurations, the git history with all the embarrassing commits where agents argued with themselves, and the TROUBLESHOOTING.md with 47 documented "WTF moments" that range from hilarious to horrifying.

Why am I doing this? Because the only way this gets better is if we stop pretending it's magic and start sharing what actually works, what fails spectacularly, and what makes us question our sanity at 3 AM.

If you want to try Vibe Coding yourself, start with ONE boring, repetitive task—not your entire project, not your startup idea, just one mind-numbing task. Spend more time on task definition than you think necessary, then double it. Write explicit paths, types, and examples because agents are more literal than a German tax code. Set up quality gates BEFORE starting or you'll spend days cleaning up creative interpretations. Expect a 30% failure rate and budget time for cleanup accordingly. Track everything: time, tokens, errors, rework, and moments of existential crisis.

Then share your results with #VibeCoding, and please include the failures because that's where the real learning happens.

## The Future Is Already Here (It's Just Poorly Distributed)

The tools will improve with Claude 3.5, GPT-4.5, and whatever comes next, but the fundamental challenge remains: turning human intent into working code through an intermediary that has no real understanding of either humans or code, just statistical patterns and probability distributions.

We're not replacing programmers; we're creating a new discipline I call "robot programming managers"—people who can decompose problems into atomic tasks, set up quality gates that even an AI can't misinterpret, and babysit artificial intelligences that are simultaneously brilliant at pattern matching and stupid at everything else.

Welcome to engineering in 2025, where we manage robots like junior developers who can type at superhuman speed but have the architectural sense of a goldfish and the attention span to match. Somehow, despite all of this chaos, frustration, and occasional moments of pure absurdity, it actually fucking works. And that might be the most surprising thing of all.

---

*Eryk Orłowski is a Design Leader with 20+ years of experience who now spends his time teaching robots how to code, cleaning up their messes, and occasionally marveling that any of this works at all. Currently building Vector Wave and wondering why he thought this was a good idea.*

*Want the full agent configurations and the git history where an AI had an existential crisis about TypeScript generics? Find me on [X @eofek](https://twitter.com/eofek) or [LinkedIn](https://linkedin.com/in/erykorlowski). Bring your own stories of AI chaos—misery loves company.*

*GitHub repo with everything: [github.com/hretheum/obsidian-smart-slides](https://github.com/hretheum/obsidian-smart-slides) (cleanup in progress, judge gently)*