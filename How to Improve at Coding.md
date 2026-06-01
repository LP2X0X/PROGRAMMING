---
tags:
  - career
  - learning
  - guideline
---

## 1. Build Things, Don't Just Study

- Notes are great for reference, but the gap between "I understand this concept" and "I can use it under pressure" only closes by **writing real code**.
- Pick small projects that **force you to combine** what you've learned.
	- Example: a full-stack app with React + a .NET API + MariaDB.
- Aim for projects that are slightly beyond your current comfort — that's where growth happens.

```ad-tip
If you can't think of a project, **clone a product you use daily** (a simplified version). You already know how it should work, so you can focus on the *how* instead of the *what*.
```

---

## 2. Read Other People's Code

- Browse **open-source repos** in your stack (C#/.NET, React, SQL).
- Pay attention to:
	- How they **structure projects** (folder layout, separation of concerns)
	- How they **name things** (variables, methods, classes)
	- How they **handle edge cases** and errors
- This builds **pattern recognition** faster than tutorials.

```ad-note
Reading code is a different skill than writing code. Both need practice. Start with small, well-documented repos before diving into large codebases.
```

---

## 3. Solve Problems Deliberately

- Not just LeetCode grinding — pick problems **slightly above your comfort zone**.
- When you solve one, **review your solution** and look for a cleaner approach.
- When you can't solve one, study the solution until you understand the **why**, not just the **what**.
- Focus on understanding the **underlying pattern** (sliding window, two pointers, recursion, etc.) rather than memorizing solutions.

| Phase | What to Do |
|---|---|
| Attempt | Try for 20-30 min before looking at hints |
| Solve | Write the solution, make it work |
| Review | Refactor for clarity and efficiency |
| Reflect | Identify the pattern, write it down |
| Revisit | Re-solve the same problem a week later without notes |

---

## 4. Refactor Your Own Old Code

- Go back to something you wrote **6 months ago**.
- If it embarrasses you, that's **growth**.
- Rewrite it with what you know now — that's where concepts become **instincts**.
- Look for:
	- Unnecessary complexity
	- Poor naming
	- Missing error handling
	- Repeated logic that could be abstracted
	- Performance issues you now understand

```ad-summary
Refactoring old code is one of the fastest ways to internalize new knowledge because you already understand the problem — you're only improving the solution.
```

---

## 5. Get Feedback

- Have others **review your code** (PRs, code review, pair programming).
- You'll learn patterns and pitfalls you can't see yourself.
- When receiving feedback:
	- Don't take it personally — it's about the code, not you.
	- Ask **why** when a suggestion isn't clear.
	- Look for recurring themes in feedback — those are your growth areas.
- When giving feedback:
	- Reviewing others' code also sharpens your own eye.

---

## 6. Go Deep on One Thing at a Time

- **Depth in one area transfers to breadth** — understanding React's reconciliation makes you a better thinker about any UI framework. Understanding GC internals makes you better at memory management in any language.
- Resist the urge to learn everything at once. Instead:
	1. Pick one topic.
	2. Study it until you can **explain it without notes**.
	3. Build something that **uses it**.
	4. Move on.

```ad-important
Shallow knowledge of 10 things is less valuable than deep knowledge of 3. Depth gives you the ability to **reason from first principles** rather than relying on memorized patterns.
```

---

## 7. Debug Intentionally

- When you hit a bug, **resist the urge to shotgun-fix it** (randomly changing things until it works).
- Instead, follow a systematic approach:
	1. **Reproduce** the bug reliably.
	2. **Form a hypothesis** about what's wrong.
	3. **Verify** the hypothesis with evidence (logs, debugger, tests).
	4. **Understand the root cause** before applying a fix.
	5. **Confirm** the fix actually addresses the root cause.
- This builds the **mental model** that separates strong developers from average ones.

```ad-note
The debugging process is often more educational than the coding itself. Every bug is a lesson about how the system actually works versus how you thought it worked.
```

---

## 8. Write Code Every Day (Consistency Over Intensity)

- Even 30 minutes of focused coding daily beats 8-hour weekend marathons.
- Consistency builds **muscle memory** for syntax, patterns, and tooling.
- Keep a streak going — use small tasks on low-energy days:
	- Fix a small bug
	- Write a utility function
	- Refactor one method
	- Solve one coding challenge

---

## 9. Learn Your Tools Deeply

- Master your **IDE** (Visual Studio, VS Code):
	- Keyboard shortcuts
	- Debugger features (conditional breakpoints, watch expressions)
	- Refactoring tools (rename, extract method, inline variable)
- Master **Git** beyond `add`, `commit`, `push`:
	- Interactive rebase, cherry-pick, bisect, stash
	- Understanding the reflog for recovery
- Master your **terminal/CLI**.
- Tool fluency removes friction, letting you focus on the actual problem.

---

## 10. Teach What You Learn

- Explaining a concept to someone else (or writing notes about it) forces you to **fill gaps** in your understanding.
- Methods:
	- Write Obsidian notes (you're already doing this)
	- Explain concepts to a colleague
	- Write a blog post or tutorial
	- Answer questions on Stack Overflow or forums

```ad-summary
If you can't explain it simply, you don't understand it deeply enough.
```

---

## 11. Study Design Patterns and Architecture

- Learn common design patterns (Factory, Observer, Strategy, Repository, etc.) and **recognize when to use them** — not just what they are.
- Study **software architecture** concepts:
	- Separation of concerns
	- SOLID principles
	- Clean architecture / layered architecture
	- When to use abstractions vs. keeping things simple
- The goal isn't to apply patterns everywhere — it's to **recognize the shape of problems** and know which tools fit.

```ad-tip
The best way to learn patterns is to encounter the **pain** of not having them first. Build something messy, feel the friction, then learn the pattern that solves it.
```

---

## 12. Practice Under Constraints

- Set **time limits** on tasks — forces you to prioritize and make decisions faster.
- Try building without your usual tools/libraries — builds understanding of what those tools do for you.
- Pair program — coding while someone watches forces clarity in your thinking.
- Participate in coding challenges, hackathons, or CTFs.

---

## 13. Maintain a "Mistakes Journal"

- When you make a mistake or learn something surprising, **write it down**:
	- What happened
	- Why it happened
	- How to prevent it in the future
- Review it periodically. Patterns will emerge — those are your blind spots.

---

## Summary

| Principle | Key Idea |
|---|---|
| Build things | Concepts become skills only through application |
| Read code | Pattern recognition from exposure |
| Solve deliberately | Struggle is where learning happens |
| Refactor old code | Turns new knowledge into instinct |
| Get feedback | Others see what you can't |
| Go deep | Depth transfers to breadth |
| Debug intentionally | Root-cause thinking builds mental models |
| Be consistent | Daily practice beats sporadic marathons |
| Learn your tools | Fluency removes friction |
| Teach others | Explaining reveals gaps |
| Study patterns | Recognize problem shapes |
| Practice constrained | Pressure sharpens decision-making |
| Track mistakes | Blind spots become visible over time |

```ad-summary
**Build, read, struggle, reflect, repeat.** Studying is the foundation — coding is what turns it into skill.
```
