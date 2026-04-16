# AI Collaboration

## Overview

This project is being built with help from two AI tools: ChatGPT and Claude Code. The user directs all decisions, validates all output, and owns the final result. AI is used as a development assistant — it speeds things up and helps work through problems, but it doesn't make judgment calls or determine direction.

---

## How ChatGPT is Used

ChatGPT is primarily used for thinking and planning:

- Reasoning through architecture and design decisions before writing any code
- Debugging issues and understanding root causes
- Drafting and refining prompts before handing them to Claude Code
- Improving documentation (README, BUILD.md, etc.)
- Acting as a sounding board when something isn't working as expected

---

## How Claude Code is Used

Claude Code handles implementation:

- Writing and updating code based on structured prompts
- Applying targeted changes to specific sections of a file
- Rapid iteration on UI, logic, and functionality
- Keeping changes focused and avoiding unintended refactoring

---

## Workflow

1. Identify an idea, problem, or improvement
2. Use ChatGPT to reason through the approach and produce a clear, structured prompt
3. Give that prompt to Claude Code to implement the change
4. Test and validate the result manually
5. Repeat

The prompts passed to Claude Code are specific and intentional — not open-ended requests. This keeps output predictable and easier to review.

---

## Philosophy

- Keep the app simple enough to fully understand
- Avoid over-engineering or adding abstractions that aren't needed
- Use AI to move faster, not to avoid understanding what's being built
- Maintain full ownership of every decision and every line of code

---

## Notes

This is an evolving workflow. Some prompts work well on the first pass, others need iteration. The goal is to build something practical and useful while staying in control of the process.
