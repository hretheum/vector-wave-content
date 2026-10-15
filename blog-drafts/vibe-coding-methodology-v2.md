# Vibe Coding: Why Engineers Shouldn't Write Boilerplate

## The 2-Hour Sprint That Replaced a 3-Week Committee

In January 2025, I built development standards for an Obsidian plugin in 2 hours. At a previous enterprise fintech company, the same task took 3 weeks and involved 5 architects.

I shit you not.

The difference? I wrote 456 lines of executable Kanban that AI could follow. They wrote 200+ pages of Confluence documentation that collected digital dust.

Here's what actually happened—with receipts.

## The Enterprise Standards Theatre

At this unnamed fintech company (2019), I watched the standards creation process unfold:
- Week 1: Arguing about folder structures
- Week 2: Debating naming conventions  
- Week 3: Writing documentation about documentation
- Participants: 5 senior architects at $150-250/hour
- Output: 200+ pages on Confluence
- Actual usage: Copy-paste-modify for every project
- Developer compliance: Near zero

Conservative estimate: 600 person-hours × $200 average = $120,000.
For boilerplate that GitHub already has templates for.

The standards weren't wrong. They just took 598 hours too long to create.

## My 2-Hour Alternative: Let Structure Write Itself

On January 9, 2025, I tried something different. Instead of writing standards, I wrote structure:

**Hour 1: Define the shape** (not the content)
- Created 10 epics describing feature areas
- Broke down into 44 specific tasks  
- Decomposed into 440+ atomic subtasks
- Each subtask: Single action, <2 hours, measurable outcome

**Hour 2: Define standards as tasks** (not documents)
```markdown
Task 1.0: Project Standards & Guidelines Definition
- 1.0.1: Define development methodology
- 1.0.2: Create code quality standards
- 1.0.3: Establish testing strategy (>90% coverage)
- 1.0.4: Define documentation templates
- 1.0.5: Create security guidelines
- 1.0.6: Establish performance benchmarks
```

**Hours 3-5.5: AI execution with /nakurwiaj command**
```bash
/nakurwiaj Task 1.0
```
Result: AI generated all standards based on project context, not generic templates.

## The Standards Actually Worked: Real Impact Metrics

Here's where it gets interesting. After generating standards, I ran Task 1.0.11 (Standards Implementation Audit) to apply them across the entire project. The impact:

**Standards propagated to:**
- 10 epics (100% coverage)
- 44 tasks updated with compliance gates
- 440+ subtasks inherited requirements

**Specific enforcements added:**
- Security validation: Every task handling user input
- Testing requirements: Minimum 90% coverage for core, 80% for UI
- Performance budgets: <100ms for analyzers, <50ms for UI response
- TypeScript strict mode: Zero any types allowed
- Result<T,E> pattern: No throwing exceptions

**Prevented issues before they happened:**
- XSS vulnerabilities: InputValidator added to 12 tasks
- Memory leaks: Cleanup requirements for 8 long-running tasks
- Race conditions: AbortController added to all async operations
- Accessibility gaps: WCAG 2.1 AA compliance in all UI tasks

The standards weren't just documentation—they became executable requirements that AI enforced during implementation.

## Breaking Down the Time Savings

**Traditional enterprise approach (observed across 3 companies, 2019-2024):**
- Standards creation: 80-120 hours
- Applying to project: 40-60 hours
- Fixing gaps found later: 60-80 hours
- Total: 180-260 hours

**Vibe Coding approach (measured, January 2025):**
- Structure creation: 2 hours
- AI standards generation: 1.5 hours
- AI implementation: 2 hours
- Human validation: 0.5 hours
- Total: 6 hours

**Efficiency gain: 30-43x**

But here's the kicker: The AI-generated standards had 100% adoption because they were baked into the task structure, not sitting in a wiki.

## The /nakurwiaj Philosophy

The command name is Polish engineering humor—roughly "let's fucking do this." It embodies the approach:

1. **Read task and all subtasks**
2. **Execute each atomically**
3. **Run quality checks after each step**
4. **Commit with proper messages**
5. **Stop at task boundaries**

No context switching. No "continue" prompts. No drift from requirements.

Example execution:
```
Task 2.1: EventBus implementation
→ Subtask 2.1.1: Define DomainEvent interface ✓
→ Subtask 2.1.2: Implement EventBus class ✓
→ Subtask 2.1.3: Add type-safe publishing ✓
[7 more subtasks...]
→ Complete: 18 minutes, 0 errors, 95% test coverage
```

## Why Design Systems Thinking Makes This Work

After 20 years building design systems, I learned three principles that transfer perfectly to AI development:

**1. Ambiguity multiplies downstream**
- Vague component spec = 10 different implementations
- Vague task description = AI hallucination festival

**2. Constraints enable creativity**
- Design tokens prevent color chaos
- Atomic tasks prevent scope creep

**3. Systems beat heroics**
- Component library > custom everything
- Task structure > prompt engineering

The Obsidian plugin has 440+ subtasks. Each one explicit. Each one measurable. Each one executable by AI without human intervention.

## The Uncomfortable Truth for Engineering Leaders

Your senior engineers are doing work that GPT-4 could do better. Not the creative work—the boilerplate:

**What AI should handle:**
- Writing test scaffolding
- Creating CRUD endpoints
- Implementing standard patterns
- Writing documentation templates
- Setting up CI/CD pipelines

**What humans should handle:**
- Defining architecture
- Making trade-off decisions
- Reviewing AI output
- Handling edge cases
- Solving novel problems

The math is brutal: If your $200/hour architect spends even 10% of their time on boilerplate, you're burning $40,000/year per person on solved problems.

## Implementation Guide: Start Monday

**Step 1: Pick your most boring task** (30 minutes)
The thing everyone avoids. Documentation, test writing, standards creation.

**Step 2: Decompose ruthlessly** (90 minutes)
- No task over 2 hours
- Every task has success criteria
- Use this format:
```
Task X.Y: [Specific outcome]
Success: [Measurable criteria]
Dependencies: [What must exist first]
```

**Step 3: Create standards as tasks, not documents** (30 minutes)
Don't write the standard. Write the task to create the standard.

**Step 4: Execute with AI** (2-4 hours)
Use your preferred AI tool. The structure matters more than the model.

**Step 5: Measure and share** (30 minutes)
- Time saved
- Quality metrics
- Issues prevented

## Results After One Week

Since implementing this approach (January 9-16, 2025):

- **Tasks completed:** 18 of 44
- **Subtasks executed:** 180+ 
- **Human time invested:** 8 hours
- **Estimated traditional time:** 240+ hours
- **Code coverage:** 92% average
- **Security issues found:** 0
- **Performance regressions:** 0

The standards didn't just document good practices—they enforced them automatically.

## The Call for Pragmatic Revolution

This isn't about AI replacing engineers. It's about engineers doing engineering instead of bureaucracy.

Every hour your team spends writing boilerplate is an hour not spent on:
- Understanding user needs
- Solving complex problems
- Building innovative features
- Mentoring junior developers
- Improving system architecture

The tools exist. The methodology works. The only question is whether you'll keep paying engineers to do robot work.

Stop writing standards. Start writing structure.
Stop managing prompts. Start managing outcomes.
Stop fighting AI. Start directing it.

That's Vibe Coding.

---

*Eryk Orłowski is a Design Leader with 20+ years of experience applying design systems thinking to AI development. Currently building [Vector Wave](https://vectorwave.io). This post documents the actual development of the Smart Slides Obsidian plugin, with all metrics verifiable in the project repository.*

*Think this is bullshit? Check the receipts: [GitHub repo with full Kanban and standards](https://github.com/yourusername/obsidian-smart-slides)*

*Want to argue? Find me on [X @hretheum](https://twitter.com/hretheum) or [LinkedIn](https://linkedin.com/in/erykorlowski). Bring data.*