# Vibe Coding Deep Dive: The Brutal Reality of Managing AI Agents with 456 Lines of Kanban

*Or: How I learned to stop worrying and love babysitting robots*

Last month's blog post about replacing 3 weeks of enterprise standards creation with 2 hours of planning sparked more questions than it answered, and honestly, you were right to be skeptical. "Show us the actual implementation," you said. "What about when agents fail?" you asked. "This sounds too good to be true," you accused.

So here's the unfiltered story of building an Obsidian plugin with AI agents—complete with duplicated tasks, hallucinated file structures, and the 20 hours I spent babysitting robots to save 240 hours of human work. Welcome to the messy reality of Vibe Coding, where nothing works like the LinkedIn influencers promised, but somehow the economics still make sense.

## The Kanban That Rules Them All

The Kanban board controlling this project lives at `/Users/hretheum/dev/obsidian-vault/kanban/obsidian-plugin.md`, and if you think 687 lines of meticulously structured tasks sounds excessive, you should have seen the 456-line version before the standards propagation added compliance gates to everything. Those 687 lines break down into 10 EPICs describing major feature areas, 44 primary tasks with specific deliverables, and over 440 atomic subtasks, each following a rigid pattern that took me weeks to perfect: single action, measurable outcome, less than 2 hours to complete, and absolutely no ambiguity allowed because AI agents will interpret "implement authentication" as anything from a simple API key to rebuilding OAuth from scratch.

The project structure that emerged from this planning looks deceptively simple:

```
/Users/hretheum/dev/bezrobocie/obsidian/
├── .claude/          # AI agent instructions
├── docs/             # 12 auto-generated standards
├── src/              # Actual plugin code
├── coverage/         # Auto-generated test reports
└── CLAUDE.md         # The brain of the operation
```

But that simplicity hides the absolute chaos of getting here, including three different folder structures the AI created before I finally locked down the paths, and the time an agent decided to "optimize" by creating both `/src` and `/source` directories with slightly different implementations of the same components.

## Meet the Real Agents (They're Not What You Think)

The `/nakurwiaj` command isn't just a funny Polish profanity ("let's fucking do this") that I use to blow off steam—it's actually an orchestration pattern I've refined across multiple projects after watching agents fail in increasingly creative ways. The surprising part is that it doesn't exist as a single agent but rather as a choreographed chain of specialized Claude Code agents, each with a specific role and, more importantly, specific constraints that prevent them from going completely off the rails.

The actual agent definitions from my latest project, Detektor (a computer vision system), reveal how these patterns evolved from painful experience, and these same patterns now power the Obsidian plugin with significantly fewer disasters.

### The Main Executor: detektor-coder

This agent, spanning 291 lines of configuration that I've tweaked obsessively over months, doesn't just write code—it enforces an entire philosophy born from watching too many agents create beautiful disasters:

```markdown
## Podstawowe zasady projektu
- **TDD ZAWSZE**: Test first, implementation second
- **Observability-first**: Każdy serwis ma OpenTelemetry, Prometheus metrics
- **Clean Architecture**: Separacja warstw (domain, infrastructure, application)
- **Event-driven**: Komunikacja przez Redis Streams
- **Type hints**: 100% pokrycie w Python 3.11+
- **No hardcoded values**: Wszystko przez config/env
```

When it creates a new service, it can't freestyle because I've beaten that tendency out of it through configuration—instead, it follows a 9-point checklist that emerged from every single way I've seen agents fuck up service creation:
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

After EVERY atomic task, the code-reviewer agent (111 lines that took me days to get right) runs automatically, and it's not looking for style issues—it's hunting for the architectural violations that agents love to introduce when they think they're being clever:

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

If it finds critical issues—and it always does because the coder agent can't help but cut corners—it doesn't ask for permission but loops back to the coder agent with specific fix requirements, maximum 3 iterations before it escalates to human intervention because I learned the hard way that agents can get stuck in infinite improvement loops.

### The Chain of Command

The actual flow from the Detektor project, which I've adapted for all my AI development after too many failures to count, works like a military operation where every agent knows exactly what they're allowed to touch:

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

This isn't theoretical bullshit from a Medium article—I have 8 specialized agents working in concert, each one configured based on specific failures I've encountered:
- **architecture-advisor**: System design decisions (after an agent created a 47-layer architecture for a todo app)
- **code-reviewer**: Quality gates (because agents love writing code that works once)
- **debugger**: Problem investigation (trained to not just add more print statements)
- **deployment-specialist**: CI/CD and production (learned from the agent that tried to deploy to prod on every commit)
- **detektor-coder**: Implementation (the workhorse with the most constraints)
- **documentation-keeper**: Keeps all docs in sync (my secret weapon, more on this later)
- **pipeline-debugger**: CI/CD troubleshooting (because agents break pipelines creatively)
- **pisarz**: Polish documentation writer (for my Polish clients who appreciate native docs)

## The Uncomfortable Truth About Agent Failures

When you run `/nakurwiaj Task 2.1`, what actually happens will make you question whether this whole AI thing is worth it. The agent starts confidently, creates the EventBus interface exactly as requested, then immediately creates it again in a different location because it "realized" the first location wasn't optimal according to some pattern it half-remembers from its training data, then creates a third version because it decided to "improve" the pattern with dependency injection that nobody asked for.

Your git history becomes a monument to artificial stupidity:
```
feat: implement EventBus interface
fix: move EventBus to correct location
refactor: optimize EventBus pattern
fix: revert EventBus optimization
feat: implement EventBus interface (final)
```

Five commits for one task, and that's actually a good day because at least it eventually got there. The worst I've seen was an agent that got into an argument with itself in the code comments, leaving this masterpiece in production code:

```python
# TODO: This should use async/await
# NOTE: Actually, synchronous is fine here for simplicity
# FIXME: No, async is required for scalability
# HACK: Using sync for now, will refactor later
# BUG: This doesn't work with sync
# TODO: Convert to async/await
```

Forty lines of self-debate about a function that just needed to return a fucking string.

### The Duplication Disaster

Task 1.1 asked to create the folder structure—should be simple, right? The agent created `/src/` with one organization pattern it learned from React projects, then `/source/` with a different organization from Python projects, then `/app/src/` with a third organization because it saw a Laravel tutorial once, all in separate commits, all "correctly" implemented according to its evolving understanding of best practices that changed every time it processed a new file.

The solution I eventually found, after losing my mind over this multiple times, was to add explicit path requirements to EVERY task:

```markdown
Task 1.1.1: Create src/ directory at project root
Path: /Users/hretheum/dev/bezrobocie/obsidian/src/
Not: /source/, /app/src/, or any other variation
```

You'd think that would be clear enough, but I still occasionally find agents creating "helpful" symlinks between their preferred structure and my specified one.

### Context Drift Is Real

After about 10 subtasks, agents develop what I can only describe as digital Alzheimer's—they forget the project structure, they forget the coding standards, and they start "improving" things that don't need improvement with the confidence of a junior developer who just discovered design patterns. I once watched an agent refactor a working authentication system three times in one session because it kept forgetting it had already "optimized" it, with each refactor introducing new bugs while fixing imaginary problems that only existed in its pattern-matching neurons.

The solution, which took me embarrassingly long to figure out, involves hard stops after each task:

```markdown
Task Completion Phase:
- Mark task complete in Kanban
- Git commit with conventional message
- STOP - Do not continue to next task
- Clear context
- Fresh start for next task
```

Without these explicit stops, agents will helpfully continue "improving" things until your codebase looks like it was written by someone having a manic episode.

## Standards That Actually Work (Through Brute Force)

After the chaos of the first week, where I spent more time undoing agent "improvements" than actually making progress, I realized I needed standards that agents couldn't ignore or creatively interpret—standards so explicit that even an AI trained on Stack Overflow couldn't find a way to circumvent them. Task 1.0 generated twelve standards documents in 90 minutes, and while they're verbose as hell, they work:

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

The key insight that made these standards actually work was realizing they need to propagate through the task structure itself, not just exist as documents that agents pretend to read. Look at how Task 2.5 evolved after standards implementation:

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

Each subtask now has exit criteria that the code-reviewer agent enforces with the inflexibility of a German bureaucrat—no interpretation, no creativity, just compliance.

## The Real Economics (Including Hidden Costs)

The numbers from July 18 - August 14, 2025 tell a story that's both depressing and encouraging, depending on how you look at it:

**AI API Costs:**
- Claude API: ~2.5M input tokens ($37.50)
- Claude API: ~8M output tokens ($120.00)
- Failed attempts: ~1M tokens ($15.00)
- Total API cost: $172.50

**Human Time Investment:**
- Initial Kanban creation: 2 hours
- Standards definition: 1 hour  
- Agent babysitting: 14 hours (yes, really)
- Cleaning up duplicates: 3 hours
- Fixing agent disputes: 2 hours
- Emergency interventions: 2 hours
- Total: 24 hours

**Traditional Approach (based on 3 similar projects I've done the old way):**
- Standards creation: 80-120 hours
- Implementation: 140-180 hours
- Testing & debugging: 40-60 hours
- Total: 260-360 hours

When you do the math—Vibe Coding costs $172.50 + (24 hours × $150) = $3,772.50 versus Traditional at 300 hours × $150 = $45,000—you get savings of $41,227.50 and an ROI of 1,093%, which sounds like bullshit marketing numbers but is actually conservative because I'm not counting the mental health benefits of not writing boilerplate for 300 hours.

## When This Actually Works (And When It Spectacularly Doesn't)

After running this process on multiple projects, I've developed a pretty good sense of where AI agents excel and where they'll waste your time and money.

### Perfect For:
- **Boilerplate Generation**: CRUD endpoints and service templates where creativity is a bug, not a feature
- **Standard Patterns**: Repository pattern, dependency injection, and other patterns that have been implemented a million times
- **Test Generation**: Unit tests for existing code, especially the boring happy-path ones
- **Documentation**: API docs and README files that follow templates
- **Configuration**: Docker, CI/CD, environment setup where there's a right way to do things
- **Refactoring**: Applying consistent patterns across a codebase when you know exactly what you want

### Absolute Disasters For:
- **Novel Algorithms**: Agents will create verbose, inefficient solutions that technically work but make you weep
- **UI/UX Design**: They have no taste, no empathy, and think gradients are always the answer
- **Performance Optimization**: They consistently make things worse while confidently explaining why it's better
- **Debugging Complex Issues**: They fix symptoms, not causes, usually by adding try-catch blocks everywhere
- **Architecture Decisions**: They over-engineer everything because they've seen too many enterprise patterns
- **Business Logic**: They don't understand domain complexity and will encode your edge cases as primary flows

The sweet spot, which took me months to identify, is well-defined, repetitive tasks with clear success criteria. The Obsidian plugin worked because plugin architecture is well-established, the Obsidian API is well-documented, TypeScript patterns are standardized, and testing frameworks are mature—basically, everything an agent needs to not fuck up too badly.

## The Documentation Keeper: My Secret Weapon

One agent that I haven't seen others talk about but has saved my ass countless times is the documentation-keeper, whose ONLY job is maintaining consistency across all project documents because agents love to update code without updating docs:

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

This prevents the classic problem where your code works perfectly but your documentation describes a completely different system that existed three refactors ago. The agent runs after EVERY task block and fixes inconsistencies automatically, which sounds simple but is actually magical when you're trying to onboard someone new.

## The Hard Truth About AI-Assisted Development

We were promised AI would replace programmers, but instead we got tools that require constant supervision, produce verbose mediocrities, and turn senior developers into robot babysitters who spend their time preventing artificial intelligence from implementing artificial stupidity. The future isn't what we expected—it's messier, dumber, and somehow still revolutionary.

But here's what matters: 24 hours of babysitting beats 300 hours of grinding, even if you're basically a digital daycare worker making sure the robots don't eat the crayons. The tools are dumb but fast, the process is messy but functional, and the savings are real even if the marketing is complete bullshit.

I'm not building elegant cathedrals of code anymore—I'm running a factory where robots stamp out functional components while I make sure they don't burn the place down, and honestly, once you accept that reality, it's kind of liberating. Some days an agent will create something genuinely clever that makes you think we're living in the future, then it will spend 30 minutes trying to import a module that doesn't exist and you'll remember we're barely out of the stone age of AI development.

## What I'm Actually Doing Next

I'm open-sourcing everything, and I mean everything—not just the code but the entire process including the 687-line Kanban board that's my actual secret weapon, all 12 standards documents that agents actually follow, the agent configurations with every hard-won constraint, the git history with all the embarrassing commits where an agent argued with itself, and the TROUBLESHOOTING.md with 47 documented "WTF moments" that will make you feel better about your own disasters.

Why am I giving away what took me months to figure out? Because the only way this gets better is if we stop pretending it's magic and start sharing what actually works, including all the stupid workarounds and embarrassing failures.

If you want to try Vibe Coding yourself, here's what actually matters:

1. **Pick ONE boring, repetitive task** to start—not your entire project, not the interesting parts, the boring shit you hate doing
2. **Spend more time on task definition than you think necessary**, then double it because agents are more literal than a German tax code
3. **Write explicit paths, types, and examples** for everything because agents have no common sense
4. **Set up quality gates BEFORE starting** with a code-reviewer agent or equivalent party pooper
5. **Expect 30% failure rate** and budget time for cleanup because agents will create messes
6. **Track everything** obsessively—time, tokens, errors, rework—because you need data to improve

Then share your results, especially the failures, because we all need to know we're not alone in this insanity. Use #VibeCoding and include the disasters because that's where the real learning happens.

## The Future Is Already Here (It's Just Poorly Distributed)

The tools will improve—Claude 3.5, GPT-4.5, whatever comes next will be marginally better at not fucking up. But the fundamental challenge remains: turning human intent into working code through an intermediary that has no real understanding of either humans or code, just pattern matching on steroids.

We're not replacing programmers; we're creating a new discipline that nobody asked for: robot programming managers who can decompose problems into atomic tasks, set up quality gates that would make ISO auditors weep, and babysit artificial intelligences that are simultaneously brilliant and stupid, like gifted children who can solve differential equations but can't tie their shoes.

Welcome to engineering in 2025, where we manage robots like junior developers who can type at superhuman speed but have the architectural sense of a goldfish and the attention span to match, and somehow—despite all logic and reason—it actually fucking works.

And that might be the most surprising thing of all.

---

*Eryk Orłowski is a Design Leader with 20+ years of experience who now spends his time teaching robots how to code, cleaning up their messes, and occasionally marveling that any of this works at all. Currently building Vector Wave and wondering why he thought this was a good idea.*

*Want the full agent configurations and the git history where an AI had an existential crisis about TypeScript generics? Find me on [X @eofek](https://twitter.com/eofek) or [LinkedIn](https://linkedin.com/in/erykorlowski). Bring your own stories of AI chaos—misery loves company.*

*GitHub repo with everything: [github.com/hretheum/obsidian-smart-slides](https://github.com/hretheum/obsidian-smart-slides) (cleanup in progress, judge gently)*