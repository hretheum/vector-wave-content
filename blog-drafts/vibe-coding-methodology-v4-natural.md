# Vibe Coding: Why Engineers Shouldn't Write Boilerplate

## The 2-Hour Sprint That Replaced a 3-Week Committee

Today, I built development standards for an Obsidian plugin in 2 hours. At one of the previous enterprise tech companies I know, the same task took 3 weeks and involved a lot of people at different seniority levels.

I shit you not.

The difference? I wrote 456 lines of executable Kanban that AI could follow. They wrote gazillion nodes of Confluence documentation that collected digital dust (hope I'm wrong here).

## The Enterprise Standards Theatre

Many times I watched the standards creation process unfold. First week was all folder structure debates. Second week they argued about naming conventions. By week three, they were writing documentation about how to write documentation. Five senior developers burning $50-150 per hour. 

Final output: 200+ pages on Confluence that every developer ceremoniously ignored and copy-paste-modified for their own projects.

Conservative math puts this at 600 person-hours × $200 average = $120,000. For boilerplate that GitHub literally gives away as templates.

The standards weren't wrong. They just took 598 hours too long to create.

## My 2-Hour Alternative

On July 18, 2025, I tried something different. Instead of writing the standards themselves, I wrote the structure that would generate them.

First hour went to defining the shape—not the content. I created 10 epics describing feature areas, broke those into 44 specific tasks, then decomposed everything into 440+ atomic subtasks. Each subtask had to be a single action, take less than 2 hours, and have a measurable outcome.

Second hour was defining standards as tasks, not documents:

```markdown
Task 1.0: Project Standards & Guidelines Definition
- 1.0.1: Define development methodology
- 1.0.2: Create code quality standards
- 1.0.3: Establish testing strategy (>90% coverage)
- 1.0.4: Define documentation templates
- 1.0.5: Create security guidelines
- 1.0.6: Establish performance benchmarks
```

Then I let AI execute with my `/nakurwiaj` command (Polish engineering humor—roughly "let's fucking do this"):

```bash
/nakurwiaj Task 1.0
```

Three and a half hours later, AI had generated all standards based on the project context. Not generic templates—actual, contextual, usable standards.

## When Standards Become Code

After generating standards, I ran Task 1.0.11 (Standards Implementation Audit) to apply them across the entire project. This is where theory met reality.

The standards propagated to all 10 epics, updated 44 tasks with compliance gates, and pushed requirements down to 440+ subtasks. Every task handling user input got security validation. Testing requirements hit minimum 90% coverage for core, 80% for UI. Performance budgets locked in at <100ms for analyzers, <50ms for UI response. TypeScript strict mode meant zero `any` types. The Result<T,E> pattern replaced exception throwing entirely.

We prevented issues before they existed. InputValidator got added to 12 tasks that would've had XSS vulnerabilities. Eight long-running tasks got cleanup requirements to prevent memory leaks. Every async operation got an AbortController to prevent race conditions. All UI tasks got WCAG 2.1 AA compliance baked in.

The standards weren't sitting in a wiki—they were executable requirements that AI enforced during implementation. No human had to remember them. No reviewer had to check for them. They just happened.

## The Time Math That Hurts

I've watched this movie three times across different companies from 2019 to 2024. Traditional enterprise approach runs something like 80-120 hours for standards creation, another 40-60 hours applying them to projects, then 60-80 hours fixing the gaps you find later. Call it 180-260 hours total.

My August 2025 approach? Two hours of structure creation, hour and a half of AI standards generation, two hours of AI implementation, half hour of human validation. Six hours total.

That's a 30-43x efficiency gain. And the AI-generated standards had 100% adoption because they were baked into the task structure, not rotting in Confluence.

## The /nakurwiaj Philosophy

The command reads the task and all subtasks, executes each atomically, runs quality checks after each step, commits with proper messages, then stops at task boundaries. No context switching. No "continue" prompts. No drift from requirements.

Watch it work:
```
Task 2.1: EventBus implementation
→ Subtask 2.1.1: Define DomainEvent interface ✓
→ Subtask 2.1.2: Implement EventBus class ✓
→ Subtask 2.1.3: Add type-safe publishing ✓
[7 more subtasks...]
→ Complete: 18 minutes, 0 errors, 95% test coverage
```

## Design Thinking Applied

Twenty years of designing UX for large systems taught me that ambiguity multiplies downstream. Give a vague component spec, get 10 different implementations. Give a vague task to AI, get a hallucination festival.

Constraints enable creativity, design tokens prevent color chaos, atomic tasks prevent scope creep.

Systems beat heroics every time. You don't freestyle every button. You shouldn't freestyle every coding task either.

The Obsidian plugin has 440+ subtasks. Each one explicit, measurable, and executable by AI without human intervention. That's not overengineering—that's engineering.

## The Uncomfortable Math for Leaders

Your $200/hour architect spending even 10% of their time on boilerplate burns $40,000 per year per person on solved problems. 

AI should write test scaffolding, CRUD endpoints, standard patterns, documentation templates, and CI/CD pipelines. Humans should define architecture, make trade-off decisions, review AI output, handle edge cases, and solve novel problems.

This isn't philosophy. It's math.

## Implementation Without the Bullshit

Pick your most boring, repetitive task tomorrow morning. The one everyone avoids. Spend 90 minutes decomposing it into tasks under 2 hours each, with clear success criteria. Write the task to create the standard, not the standard itself. Then point your AI at it and measure what happens.

Track time saved, quality metrics, and issues prevented. Share the data.

## The Actual Point

Engineers should engineer, not write bureaucracy. Every hour spent on boilerplate is an hour not spent understanding user needs, solving complex problems, building innovative features, mentoring juniors, or improving architecture.

The tools exist. The methodology works. You can keep paying engineers to do robot work, or you can let robots do robot work while engineers do engineering.

Write structure, not standards. Manage outcomes, not prompts. Direct AI instead of fighting it.

That's Vibe Coding. Not because it sounds cool, but because it actually fucking works.

---

*Eryk Orłowski is a Design Leader with 20+ years of experience applying design systems thinking to AI development. Currently building [Vector Wave](https://vectorwave.io). This post documents the actual development of the Smart Slides Obsidian plugin, with all metrics verifiable in the project repository.*

*Think this is bullshit? Check the receipts: [GitHub repo with all the standards](https://github.com/hretheum/obsidian-smart-slides)*

*Want to argue? Find me on [X @eofek](https://twitter.com/eofek) or [LinkedIn](https://linkedin.com/in/eofek). Bring data, I am always willing to learn from someone smarter or more experienced!*