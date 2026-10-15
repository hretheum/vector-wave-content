# Vibe Coding Deep Dive: The Brutal Reality of Managing AI Agents with 456 Lines of Kanban

*Or: How I learned to stop worrying and love babysitting robots*

~~Last month's blog post about replacing 3 weeks of enterprise standards creation with 2 hours of planning sparked more questions than it answered, and honestly, you were right to be skeptical. "Show us the actual implementation," you said. "What about when agents fail?" you asked. "This sounds too good to be true," you accused.~~

**[Eryk made me rewrite this because it sounded like "everyone asks about my skincare routine" influencer bullshit]**

Here's what actually happened: I shipped an Obsidian plugin using AI agents to write 90% of the code. It took 687 lines of Kanban board configuration, 12 auto-generated standards documents, and 24 hours of babysitting robots who kept arguing with themselves in code comments. The agents created three different folder structures, duplicated every component at least once, and at one point got into a 40-line philosophical debate about async/await in the comments of a function that just needed to return a string.

But it still cost $3,772 instead of $45,000, and that's why I keep doing this insane shit.

Welcome to the unfiltered reality of Vibe Coding—where AI agents write production code while you prevent them from burning down your repository, and somehow the math still works out in your favor even when the process looks like absolute chaos.

## The Kanban That Rules Them All

The Kanban board controlling this project lives at `/Users/hretheum/dev/obsidian-vault/kanban/obsidian-plugin.md`, and if you think 687 lines of meticulously structured tasks sounds excessive, you should have seen the 456-line version before the standards propagation added compliance gates to everything. Those 687 lines break down into 10 EPICs describing major feature areas, 44 primary tasks with specific deliverables, and over 440 atomic subtasks, each following a rigid pattern that took me weeks to perfect: single action, measurable outcome, less than 2 hours to complete, and absolutely no ambiguity allowed because AI agents will interpret "implement authentication" as anything from a simple API key to rebuilding OAuth from scratch.

The project structure that emerged from this planning looks deceptively simple:

**Project Structure:**
```
/Users/hretheum/dev/bezrobocie/obsidian/
├── .claude/          # <- AI agent instructions live here
├── docs/             # <- 12 auto-generated standards 
├── src/              # <- Actual plugin code (after 3 attempts)
├── coverage/         # <- Auto-generated test reports
└── CLAUDE.md         # <- The brain of the operation
```

But that simplicity hides the absolute chaos of getting here, including three different folder structures the AI created before I finally locked down the paths, and the time an agent decided to "optimize" by creating both `/src` and `/source` directories with slightly different implementations of the same components.

## Meet the Real Agents (They're Not What You Think)

The `/nakurwiaj` command isn't just a funny Polish profanity ("let's fucking do this") that I use to blow off steam—it's actually an orchestration pattern I've refined across multiple projects after watching agents fail in increasingly creative ways. The surprising part is that it doesn't exist as a single agent but rather as a choreographed chain of specialized Claude Code agents, each with a specific role and, more importantly, specific constraints that prevent them from going completely off the rails.

The actual agent definitions from my latest project, Detektor (a computer vision system), reveal how these patterns evolved from painful experience, and these same patterns now power the Obsidian plugin with significantly fewer disasters.

### The Main Executor: detektor-coder

This agent, spanning 291 lines of configuration that I've tweaked obsessively over months, doesn't just write code—it enforces an entire philosophy born from watching too many agents create beautiful disasters:

**The Agent's Non-Negotiable Rules:**
```markdown
## Podstawowe zasady projektu (Core Project Rules)

- **TDD ZAWSZE**: Test first, implementation second
  # No exceptions. Write the test or GTFO.

- **Observability-first**: Każdy serwis ma OpenTelemetry, Prometheus metrics  
  # If you can't measure it, you can't fix it when it breaks at 3am

- **Clean Architecture**: Separacja warstw (domain, infrastructure, application)
  # Layers are sacred. Mix them and face my wrath.

- **Event-driven**: Komunikacja przez Redis Streams
  # No direct service calls. Ever. I mean it.

- **Type hints**: 100% pokrycie w Python 3.11+
  # "Any" type = instant rejection

- **No hardcoded values**: Wszystko przez config/env
  # Found a hardcoded URL? Start over.
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

**❌ The Horror Show (what agents actually write):**
```python
# THIS IS WHAT AI THINKS IS "PRODUCTION READY":
def process_data(d):  # <- WTF is 'd'?
    result = []
    for item in d:  # <- No type hints = Russian roulette
        if item['status'] == 'active':  # <- Magic string alert!
            result.append(item['value'] * 1.2)  # <- 1.2? Why? WHO KNOWS!
    return result  # <- Returns... something?
```

**✅ What the Code Reviewer Demands:**
```python
# AFTER THE CODE REVIEWER BEATS IT INTO SHAPE:
from typing import List, Dict, Any
from enum import Enum

class ItemStatus(Enum):
    """Status values for items - no magic strings allowed!"""
    ACTIVE = "active"
    INACTIVE = "inactive"

# Named constants or death
TAX_MULTIPLIER = 1.2  # 20% tax rate, documented and findable

def calculate_active_items_with_tax(
    items: List[Dict[str, Any]]  # <- Actual type hints!
) -> List[float]:  # <- Return type specified!
    """Calculate values with tax for active items.
    
    No magic, no mystery, just boring clarity.
    """
    return [
        item['value'] * TAX_MULTIPLIER
        for item in items
        if ItemStatus(item['status']) == ItemStatus.ACTIVE
    ]
```

If it finds critical issues—and it always does because the coder agent can't help but cut corners—it doesn't ask for permission but loops back to the coder agent with specific fix requirements, maximum 3 iterations before it escalates to human intervention because I learned the hard way that agents can get stuck in infinite improvement loops.

### The Chain of Command

The actual flow from the Detektor project, which I've adapted for all my AI development after too many failures to count, works like a military operation where every agent knows exactly what they're allowed to touch:

**The Orchestration Logic (Polish commands because why not):**
```markdown
## Automatyczny łańcuch agentów (Automatic Agent Chain)

IF task contains "implement|create|add":
    → /agent detektor-coder
    # "Go build something"
    
ELIF task contains "debug|fix|investigate":
    → /agent debugger  
    # "Figure out why it's broken"
    
ELIF task contains "refactor|optimize":
    → /agent architecture-advisor → detektor-coder
    # "Make it better, but check with the architect first"
    
ELIF task contains "deploy|deployment":
    → /agent deployment-specialist
    # "Ship it (carefully)"

AFTER each atomic task:
    → /agent code-reviewer
    # "Is this garbage?"
    
    IF critical_issues:
        → Loop back with feedback
        → Auto-fix without asking
        # "Fix your shit and try again"
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

**The Commit History of Shame:**
```bash
# What your git log actually looks like after AI "helps":
feat: implement EventBus interface          # <- Attempt #1
fix: move EventBus to correct location      # <- "Oops wrong place"  
refactor: optimize EventBus pattern         # <- "I can make it better!"
fix: revert EventBus optimization           # <- "Actually that broke everything"
feat: implement EventBus interface (final)  # <- "OK for real this time"
```

Five commits for one task, and that's actually a good day because at least it eventually got there. The worst I've seen was an agent that got into an argument with itself in the code comments, leaving this masterpiece in production code:

**The Agent's Internal Monologue (yes, this was real):**
```python
# The function that broke me:
def get_user_data(user_id):
    # TODO: This should use async/await
    # NOTE: Actually, synchronous is fine here for simplicity
    # FIXME: No, async is required for scalability
    # HACK: Using sync for now, will refactor later
    # BUG: This doesn't work with sync
    # TODO: Convert to async/await
    # NOTE: But the database driver is sync...
    # FIXME: Switch to async database driver
    # TODO: Research async database drivers
    # HACK: Just return mock data for now
    
    return {"name": "John"}  # <- 40 lines of debate for THIS
```

Forty lines of self-debate about a function that just needed to return a fucking string.

### The Duplication Disaster

Task 1.1 asked to create the folder structure—should be simple, right? The agent created `/src/` with one organization pattern it learned from React projects, then `/source/` with a different organization from Python projects, then `/app/src/` with a third organization because it saw a Laravel tutorial once, all in separate commits, all "correctly" implemented according to its evolving understanding of best practices that changed every time it processed a new file.

The solution I eventually found, after losing my mind over this multiple times, was to add explicit path requirements to EVERY task:

**How to Write Tasks So Even AI Can't Fuck It Up:**
```markdown
Task 1.1.1: Create src/ directory at project root

EXPLICIT PATH (read this three times AI):
✅ CORRECT: /Users/hretheum/dev/bezrobocie/obsidian/src/
❌ WRONG: /source/
❌ WRONG: /app/src/  
❌ WRONG: Any other creative interpretation

IF YOU CREATE ANYTHING ELSE, YOU HAVE FAILED.
```

You'd think that would be clear enough, but I still occasionally find agents creating "helpful" symlinks between their preferred structure and my specified one.

### Context Drift Is Real

After about 10 subtasks, agents develop what I can only describe as digital Alzheimer's—they forget the project structure, they forget the coding standards, and they start "improving" things that don't need improvement with the confidence of a junior developer who just discovered design patterns. I once watched an agent refactor a working authentication system three times in one session because it kept forgetting it had already "optimized" it, with each refactor introducing new bugs while fixing imaginary problems that only existed in its pattern-matching neurons.

The solution, which took me embarrassingly long to figure out, involves hard stops after each task:

**The Emergency Brake System:**
```markdown
## Task Completion Phase (MANDATORY - Skip this and die)

1. Mark task complete in Kanban ✓
2. Git commit with conventional message
3. >>> STOP - Do not continue to next task <<<
4. Clear context (forget everything)
5. Fresh start for next task
6. NO IMPROVEMENTS. NO OPTIMIZATIONS. JUST STOP.
```

Without these explicit stops, agents will helpfully continue "improving" things until your codebase looks like it was written by someone having a manic episode.

## Standards That Actually Work (Through Brute Force)

After the chaos of the first week, where I spent more time undoing agent "improvements" than actually making progress, I realized I needed standards that agents couldn't ignore or creatively interpret—standards so explicit that even an AI trained on Stack Overflow couldn't find a way to circumvent them. Task 1.0 generated twelve standards documents in 90 minutes, and while they're verbose as hell, they work:

**The Standards That Actually Get Followed:**
```bash
docs/
├── CODE_STANDARDS.md           # <- TypeScript strict, Result<T,E> pattern
├── SECURITY_GUIDELINES.md      # <- XSS, path traversal, input validation  
├── TESTING_STRATEGY.md         # <- >90% core, >80% UI coverage
├── ERROR_HANDLING_STANDARDS.md # <- No exceptions, only Results
├── PERFORMANCE_BENCHMARKS.md   # <- <100ms analyzers, <50ms UI
├── LLM_DEVELOPMENT_GUARDRAILS.md # <- Prompt injection prevention
└── [6 more standards files]    # <- Yes, TWELVE total. Deal with it.
```

The key insight that made these standards actually work was realizing they need to propagate through the task structure itself, not just exist as documents that agents pretend to read. Look at how Task 2.5 evolved after standards implementation:

**Task Structure With Teeth:**
```markdown
## Task 2.5: Input Validator with security
│
├── 2.5.1: Implement XSS prevention
│   └── EXIT CRITERIA: Blocks all OWASP Top 10 XSS vectors
│
├── 2.5.2: Add path traversal protection  
│   └── EXIT CRITERIA: ../../../etc/passwd returns error
│
├── 2.5.3: Create filename sanitization
│   └── EXIT CRITERIA: "../../evil.sh" becomes "evil.sh"
│
├── 2.5.4: Add input length validation
│   └── EXIT CRITERIA: Rejects inputs >10MB
│
├── 2.5.5: Implement content type validation
│   └── EXIT CRITERIA: Rejects non-text MIME types
│
├── 2.5.6: Create security test suite
│   └── EXIT CRITERIA: 100% coverage of security vectors
│
└── COMPLIANCE GATES (non-negotiable):
    ├── Standards: Security → docs/SECURITY_GUIDELINES.md
    ├── Standards: Testing >90% coverage OR FAIL
    ├── Standards: Code Quality & TS strict OR REJECT
    └── Standards: Performance <10ms per validation OR REFACTOR
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

**The Documentation Keeper Protocol:**
```markdown
## BEFORE moving to next block (EVERY. SINGLE. TIME.):

→ /agent documentation-keeper

SYNCHRONIZE ALL THE THINGS:
  ✓ Task checkboxes [x] in decomposition files
  ✓ Service ports in PROJECT_CONTEXT.md  
  ✓ Status updates in architecture document
  ✓ New problems → TROUBLESHOOTING.md
  ✓ Service READMEs if new services added
  ✓ API docs if endpoints changed
  
VERIFY: "Are all docs telling the same story?"
IF NOT: Fix it before anyone notices
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