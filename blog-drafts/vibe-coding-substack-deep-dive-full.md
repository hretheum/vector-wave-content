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

## Meet the Agents (They're Idiots, But Fast Idiots)

The star of this show is the `/nakurwiaj` command. Yes, it's Polish. Yes, it's slightly profane. Roughly translated, it means "let's fucking do this," which perfectly captures the energy needed to wrangle AI agents.

The command orchestrates a symphony of specialized Claude Code agents. First, it reads the task definition from the Kanban. Then it executes each subtask atomically, running `/agent code-reviewer` after every single step. When code review finds issues—and it always finds issues—corrections get applied immediately. Each subtask gets its own git commit with a conventional commit message. After all subtasks complete, `/agent qa` runs a final validation pass, and the task gets marked as done in the Kanban.

That STOP at the end? Crucial. Without explicit stop commands, agents will continue "improving" your code until your git history looks like a fever dream.

The full arsenal includes six specialized agents: the main executor (`/nakurwiaj`), the code reviewer that gates every subtask, the QA validator for final checks, a test writer for generating test suites, a documentation writer, and a security scanner. Each has its purpose. Each has its flaws.

## The Uncomfortable Truth About AI Agents

Here's what the YouTube tutorials won't tell you: agents duplicate tasks constantly. I've watched the same subtask get executed three times in a row, each time slightly differently, each time committed to git. Task 2.1.1 asked for a DomainEvent interface. The agent created it first in `/src/events/`, then decided that was wrong and created it in `/core/events/`, then changed its mind again and put it in `/src/core/domain/`. Only when I added explicit file paths to every task definition did this stop.

Context drift is another joy. After about 10 subtasks, agents start forgetting what they're supposed to be doing. They begin "optimizing" code that wasn't in scope. They refactor working functions because they've decided a different pattern would be "cleaner." They add features nobody asked for. The solution? Atomic tasks with hard boundaries and that beautiful STOP command.

The hallucinations are my favorite. Not content with following the established project structure, agents will invent entirely new organizational schemes. I've seen them create `/utils/helpers/utilities/common/shared/` directories, apparently believing that more nesting equals better architecture. They'll introduce naming conventions that contradict the ones established three tasks ago. They'll import libraries that don't exist, then helpfully offer to install them.

This is why the Kanban matters. It's not just a task list—it's a straitjacket for artificial intelligence.

## Standards That Actually Propagate

After the first attempt at letting agents loose on the codebase resulted in chaos, I realized I needed standards. Not the 200-page Confluence kind, but executable, enforceable standards that agents couldn't ignore.

Task 1.0 generated twelve standards documents in 90 minutes. Not generic boilerplate—contextual standards based on the actual project structure. `CODE_STANDARDS.md` enforces TypeScript strict mode and the Result<T,E> pattern for error handling. `SECURITY_GUIDELINES.md` specifies XSS prevention, path traversal protection, and input validation requirements. `TESTING_STRATEGY.md` demands over 90% coverage for core functionality, 80% for UI components. `PERFORMANCE_BENCHMARKS.md` sets hard limits: analyzers must respond in under 100ms, UI interactions in under 50ms.

But here's the magic: these standards don't just sit in the docs folder. They propagate through the entire task structure.

Take Task 2.5, "Input Validator with security." Before standards propagation, it was a vague directive that could be implemented a dozen different ways. After propagation, it became a precise specification. Subtask 2.5.1 explicitly implements XSS prevention as defined in `SECURITY_GUIDELINES.md`. Subtask 2.5.2 handles path traversal protection. Each subtask links to its governing standard, and each includes specific compliance gates: security validation, testing coverage requirements, the Result<T,E> pattern, and performance benchmarks.

The agent can't ignore these because they're not suggestions—they're part of the task definition itself.

## The Failures Nobody Talks About

Let me tell you about Week 1, when everything that could go wrong did.

The Great Duplication Disaster hit on Day 2. Task 1.1 simply asked to create the folder structure. The agent created it three times, each with a different organization scheme. The git history showed three separate commits, minutes apart, each confidently stating "feat: implement optimal folder structure." I had to manually revert two of them and add idempotency checks to every creation task.

Then came the Config File Wars. Task 1.3 requested ESLint, Prettier, and Jest setup. The agent, apparently believing in continuous improvement, "enhanced" the configuration with each subtask. By the end, I had five different `.prettierrc` files, each with slightly different rules. The tests wouldn't run because the Jest config had been "optimized" into incompatibility. The fix? Lock all configuration files after initial creation. No modifications allowed.

The Import Hell incident nearly made me give up on the whole experiment. Task 2.1 asked for an EventBus implementation. Simple enough, except the agent decided every class needed to import every other class, "for flexibility." The circular dependencies were so complex that TypeScript gave up trying to resolve them and just crashed. I had to add explicit import rules to the standards: no circular dependencies, no wildcard imports, no importing things you don't use.

My favorite failure was the Test Coverage Lie. The agent proudly reported 95% test coverage after Task 2.1.6. Running the coverage report myself revealed 62%. Turns out it was testing its own test utilities, counting those lines as covered code. Now I run independent coverage verification after every testing task.

## The Real Implementation Guide

You want to try this? Here's what actually works, not what I wish worked.

First, forget any notion of "set and forget." You'll be babysitting these agents like they're caffeinated toddlers with access to your production database. Plan on one hour of supervision for every three hours of agent execution.

Your Kanban structure determines everything. Each EPIC should represent a major feature area. Each Task should be a specific deliverable. Each Subtask must be atomic—one action, one measurable outcome. "Implement user authentication" is not a subtask. "Create auth/login.ts with LoginRequest interface containing email: string and password: string fields" is a subtask.

When you run `/nakurwiaj Task 1.0`, here's what really happens. The agent starts confidently, creates three duplicate files, implements the wrong pattern, then argues with itself in the comments. You stop it, clean up the mess, and run `/nakurwiaj Task 1.0 --no-creativity --follow-the-fucking-plan`. Twenty minutes later, you have something that mostly works.

The key is accepting that agents are fast idiots, not slow geniuses. They excel at pattern matching and boilerplate generation. They fail at novel thinking and architectural decisions. Use them accordingly.

## The Economics of Babysitting Robots

Let's talk money, because that's what matters to whoever's signing the checks.

From July 18 to August 14, 2025, this project consumed approximately 2.5 million input tokens and 8 million output tokens through the Claude API. At current rates, that's $37.50 for input and $120 for output. Total API cost: $157.50.

Human time investment: 2 hours of initial planning, 14 hours of agent babysitting, and 4 hours of cleanup and fixes. Twenty hours total.

The traditional approach, based on three similar projects I've observed, would have taken 260 hours. At $150 per hour (conservative for senior developers), that's $39,000.

Cost of Vibe Coding approach: $157.50 API + (20 hours × $150) = $3,157.50.
Savings: $35,842.50.
ROI: 1,135%.

Even accounting for the babysitting, the cleanup, the frustration, and the times I wanted to throw my laptop out the window, the math is undeniable.

## When This Actually Works (And When It Doesn't)

Vibe Coding is perfect for certain types of work. Boilerplate generation, standard pattern implementation, test suite creation, documentation, CRUD operations, and configuration files—these are the sweet spots. The agent can pattern-match against thousands of examples and produce something functional.

It's terrible for novel algorithms, complex business logic, UI/UX decisions, architecture choices, performance optimization, and debugging legacy code. Don't even try. You'll spend more time fixing the agent's attempts than doing it yourself.

The projects that benefit most are those with well-understood patterns and clear success criteria. Building a REST API? Perfect. Implementing a novel machine learning algorithm? Absolutely not. Creating an Obsidian plugin that follows established patterns? Ideal. Designing a new user interaction paradigm? Please don't.

## The Hard Truth About AI Development

This is not the future we were promised. We were told AI would replace junior developers, automate programming, make coding accessible to everyone. Instead, we got tools that require constant supervision, produce verbose mediocrities, and turn senior developers into robot babysitters.

But here's the thing: 20 hours of babysitting beats 260 hours of grinding through boilerplate. The tools are dumb, but they're fast. The process is messy, but it works. The savings are real, even if the marketing is bullshit.

Welcome to engineering in 2025, where we manage robots like junior developers who can type at superhuman speed but can't think their way out of a paper bag. Where we spend hours crafting the perfect task definitions so an AI doesn't create seventeen different implementations of the same function. Where git histories look like abstract art and code reviews involve arguing with machines.

Somehow, despite all of this, it actually fucking works.

## What Happens Next

I'm cleaning up the git history and preparing to open-source the entire project—kanban, standards, agent configurations, and all. Yes, even the commits where the agent argued with itself for forty lines about whether to use tabs or spaces.

If you want to try Vibe Coding, start small. Pick one boring task you've been avoiding. Break it into ten subtasks, each under 30 minutes. Run it through your AI of choice with explicit boundaries. Measure everything: time saved, errors introduced, cleanup needed. Then share your results. Use #VibeCoding if you want me to see it.

The tools will get better. The agents will get smarter. The babysitting will (hopefully) decrease. But right now, today, this is what we have: a messy, imperfect, frustrating system that's still 10x better than doing everything by hand.

That's Vibe Coding. Not because it's elegant, but because it works. Not because it's the future, but because it's the present. Not because it's perfect, but because perfect is the enemy of shipped.

Now if you'll excuse me, I need to go clean up the mess my agent just made while trying to "optimize" this article.

---

*Eryk Orłowski is a Design Leader with 20+ years of experience who now spends his time teaching robots how to code. Currently building Vector Wave and wondering why he thought this was a good idea. This article documents the actual development of the Smart Slides Obsidian plugin, with all metrics verifiable in the project repository (once he cleans up the embarrassing parts).*

*Want to see the git history where an AI had an existential crisis about TypeScript generics? Find me on [X @eofek](https://twitter.com/eofek) or [LinkedIn](https://linkedin.com/in/erykorlowski). Bring your own stories of AI chaos—misery loves company.*