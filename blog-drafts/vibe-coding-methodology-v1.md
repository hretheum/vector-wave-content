# Vibe Coding: Why Engineers Shouldn't Write Bureaucracy

## Or: How I Replaced 3 Weeks of Standards Creation with 2 Hours of Structured Thinking

True story: I once watched a team of 5 architects spend 3 weeks creating development standards on Confluence. Multiple meetings. Endless debates about naming conventions. Passionate arguments about folder structures. The result? 200 pages of documentation that became immediately outdated and was ceremoniously ignored by every developer who was supposed to follow it.

I shit you not.

Last week, I created better standards in 2 hours. Then I used those standards to build a complete Obsidian plugin in 3.5 hours. The entire project—from zero to functional plugin—took less time than a single "alignment meeting" from that Confluence disaster.

This is Vibe Coding: the radical idea that engineers should engineer, not write bureaucratic boilerplate.

## The $30,000 Documentation Nobody Reads

Let's do the math on that Confluence project:
- 5 architects × 3 weeks × 40 hours × $200/hour = $120,000
- Plus the meetings with stakeholders = $30,000
- Plus the review cycles = $20,000
- Total cost: ~$170,000

What did we get? Standards that were:
- Outdated before they were published
- Too generic to be useful
- Too specific to be flexible
- Stored in a place developers never visited
- Written in corporate-speak that obscured actual requirements

The real kicker? Every new project copy-pasted these standards, changed 20%, and created their own 200-page variant. The multiplication of waste was exponential.

## The Uncomfortable Truth About Development Standards

Here's what nobody wants to admit: development standards are necessary but boring. They're like the safety instructions on airplanes—crucial for consistency and quality, but nobody wants to write them, and barely anyone wants to read them.

What goes into these standards?
- Code quality guidelines
- Testing requirements
- Documentation templates
- Security protocols
- Performance benchmarks
- Architecture patterns
- Git workflows
- CI/CD configurations

Is any of this rocket science? No.
Is it necessary? Absolutely.
Should senior engineers spend weeks writing it? Hell no.

## Enter Vibe Coding: Structure Without Bureaucracy

Vibe Coding isn't about eliminating standards—it's about eliminating the waste in creating them. Here's the methodology:

### Phase 1: Human Defines Intent (30 minutes)
I start with clarity about what I'm building. Not how—what. For the Obsidian plugin:
- "Build a plugin that generates presentations from prompts"
- "Must integrate with existing Obsidian plugins"
- "Zero configuration for users"
- "Production-ready, not a prototype"

### Phase 2: Structured Decomposition (90 minutes)
This is where 20 years of design thinking pays off. I create a Kanban with:
- **Epics**: Major feature areas (10 total)
- **Tasks**: Specific deliverables (44 total)
- **Subtasks**: Atomic actions <2 hours each (440+ total)

Each subtask follows a pattern:
```
Task 2.1.1: Define DomainEvent interface
- Action: Create interface file
- Success: TypeScript compiles, events are type-safe
- Time: <30 minutes
```

### Phase 3: Standards Generation (30 minutes)
Here's where it gets interesting. Instead of writing standards, I define Task 1.0:

```markdown
Task 1.0: Project Standards & Guidelines Definition
- 1.0.1: Define development methodology
- 1.0.2: Create code quality standards
- 1.0.3: Establish testing strategy
- 1.0.4: Define documentation templates
- 1.0.5: Create security guidelines
- 1.0.6: Establish performance benchmarks
```

Then I let AI generate the actual standards based on the project context. The AI reads my decomposition, understands the project scope, and creates appropriate standards. Not generic "best practices"—specific, contextual, actionable standards.

### Phase 4: Execution Automation (3.5 hours)
The `/nakurwiaj` command (Polish engineering at its finest):
```bash
/nakurwiaj Task 1.1
```

What happens:
1. AI reads the task definition
2. Executes each subtask sequentially
3. Runs code review after each step
4. Commits with conventional commit messages
5. Stops at task boundary

No context switching. No "continue" prompts. No babysitting.

## The Real Metrics That Matter

**Traditional Approach:**
- Standards creation: 3 weeks (120 hours)
- Plugin architecture: 1 week (40 hours)
- Implementation: 2 weeks (80 hours)
- Total: 240 hours

**Vibe Coding Approach:**
- Planning & decomposition: 2 hours
- AI execution: 3.5 hours
- Human validation: 0.5 hours
- Total: 6 hours

**Efficiency gain: 40x**

But here's the honest part: Yes, these standards should ideally be created once for all team projects. But even that "once" shouldn't take 3 weeks and 5 architects. And in reality, every team creates their own variant anyway.

## Why Design Leaders Make Better AI Engineers

After 20 years in design leadership, I've learned that design systems and code systems follow the same principles:
- Components need clear contracts
- Ambiguity multiplies downstream
- Standards prevent chaos
- Documentation should live in the code, not in wikis

The difference? Designers accepted long ago that systems need structure. We don't freestyle every button. We don't reinvent navigation patterns. We use systems.

Software engineering, paradoxically, often resists this systematization when it comes to process and standards. Every team wants their special snowflake approach.

## The Controversial Part: Engineers Shouldn't Write Boilerplate

This is where I lose some people: I don't think senior engineers should write development standards, documentation templates, or testing frameworks. These are solved problems with known patterns.

What SHOULD senior engineers do?
- Solve novel technical challenges
- Design system architectures
- Mentor junior developers
- Make critical technical decisions
- Review and validate AI output

Having a $300/hour architect write Git workflow documentation is organizational malpractice. It's like having a surgeon fill out insurance forms.

## Practical Implementation: How to Start

1. **Accept the Reality**: Your standards documentation is probably outdated and ignored. That's okay. Everyone's is.

2. **Start Small**: Pick one project. Give yourself 2 hours to decompose it into atomic tasks.

3. **Use the Right Tools**: You need:
   - A Kanban system (Obsidian, Notion, whatever)
   - AI with good context windows (Claude, GPT-4)
   - Version control (obviously)
   - Automated testing (non-negotiable)

4. **Define Success Criteria**: Every task needs measurable completion criteria. "Implement authentication" is not a task. "Create JWT validation middleware that returns 401 on invalid tokens" is a task.

5. **Let AI Do the Boring Parts**: Standards, boilerplate, documentation templates, test scaffolding—all perfect for AI.

## The Objections I Always Hear

**"But AI makes mistakes!"**
Yes. So do humans. The difference? AI mistakes are consistent and fixable. Human bureaucracy multiplies and mutates.

**"This is overengineering!"**
Writing 200 pages on Confluence is overengineering. Creating 456 lines of executable Kanban is engineering.

**"Our standards are too complex for AI!"**
If your standards are too complex for AI to understand, they're too complex for humans to follow.

**"We need human insight!"**
Absolutely. That's why humans define WHAT and WHY. AI handles HOW and WHERE.

## The Future of Engineering Work

Vibe Coding isn't about replacing engineers—it's about letting engineers do engineering. The future isn't AI vs. humans. It's humans doing human work (creative problem solving, architecture, design) while AI does robot work (boilerplate, documentation, standards).

The teams that understand this will ship 40x faster. Not because they work harder, but because they stop wasting time on solved problems.

## The Call to Action

Your engineering team is probably wasting 60+ hours per project on bureaucracy. That's not an exaggeration—that's what I measured across multiple teams.

Here's what you can do this week:
1. Pick your most boring, bureaucratic task
2. Spend 2 hours decomposing it into atomic subtasks
3. Let AI execute it while you review
4. Measure the time saved
5. Share the results

Stop making expensive humans do robot work. Let robots do robot work. Let humans do human work.

That's Vibe Coding.

---

*Eryk Orłowski is a Design Leader with 20+ years of experience who now builds AI-powered tools. He's currently working on [Vector Wave](https://vectorwave.io), where he applies design thinking to AI development. This post is based on real metrics from building the Smart Slides Obsidian plugin.*

*Want to argue about this? Find me on [Twitter/X @hretheum](https://twitter.com/hretheum) or [LinkedIn](https://linkedin.com/in/erykorlowski). I have data.*