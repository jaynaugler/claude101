# Getting More Out of Claude Code: A Starting Point (`CLAUDE.md` & Custom Skills)

Out of the box, AI coding assistants are eager to please: if you paste an error or a prompt, they will immediately dump 200 lines of over-engineered code, refactor unrelated files, and rob you of the actual learning process.

This guide is a **foundational starting point** for taking control of how Claude behaves across your entire system. By configuring two files in your user profile, you can establish a baseline of engineering discipline everywhere while giving yourself a toggleable "Rubber Duck / Mentor Mode" for college coursework.

> **Treat this as a baseline:** Once you see how global rules and skills work, you are encouraged to research further—customizing rules for your own tech stacks, adding repo-specific context files, and building skills for your own workflows.

---

## How This Setup Works

Claude Code looks inside the `.claude` folder in your home directory (`~/.claude` on Linux/macOS/WSL, or `C:\Users\<username>\.claude` on Windows) for user-wide configuration:

1. **Global Instructions (`~/.claude/CLAUDE.md`):**
   Loaded **automatically** into every session. We put Andrej Karpathy's four LLM coding guidelines here (*Think Before Coding, Simplicity First, Surgical Changes, and Goal-Driven Execution*) so Claude stops overcomplicating code and asks clarifying questions before acting.
2. **User-Wide Skill (`~/.claude/skills/course-mentor/SKILL.md`):**
   Loaded **on demand** when you type `/course-mentor` (or ask Claude to "rubber duck this with me"). This acts as a hard **No-Code Gate** for coursework—turning Claude into a senior engineering mentor that helps you read stack traces, isolate system boundaries, and design architectures without writing the code for you unless explicitly commanded.

---

## Step 1: Create the Folders & Files

Choose your operating system below to create the required directory structure in your user profile.

### Linux, WSL (Ubuntu), and macOS (Bash / Zsh)

Run the following in your terminal:

```bash
# 1. Create the global .claude and skill directories
mkdir -p ~/.claude/skills/course-mentor

# 2. Create the empty files
touch ~/.claude/CLAUDE.md
touch ~/.claude/skills/course-mentor/SKILL.md

# 3. Open both files in VS Code (or use nano/vim)
code ~/.claude/CLAUDE.md ~/.claude/skills/course-mentor/SKILL.md
```

### Windows (PowerShell)

Run the following in PowerShell:

```powershell
# 1. Create the global .claude and skill directories
New-Item -ItemType Directory -Force -Path "$HOME\.claude\skills\course-mentor"

# 2. Create the empty files
New-Item -ItemType File -Force -Path "$HOME\.claude\CLAUDE.md"
New-Item -ItemType File -Force -Path "$HOME\.claude\skills\course-mentor\SKILL.md"

# 3. Open both files in VS Code (or use notepad)
code "$HOME\.claude\CLAUDE.md" "$HOME\.claude\skills\course-mentor\SKILL.md"
```

*(Note: If you use WSL on Windows and run `claude` inside your Linux terminal, set these files up inside your **WSL** home directory rather than Windows PowerShell.)*

---

## Step 2: Populate Your Global `CLAUDE.md`

Paste the following into `~/.claude/CLAUDE.md` (or `$HOME\.claude\CLAUDE.md` on Windows):

```markdown
# Global Execution Guidelines

Behavioral guidelines to reduce common LLM coding mistakes.
**Tradeoff:** These guidelines bias toward caution over speed. For trivial tasks, use judgment.

## 1. Think Before Coding
**Don't assume. Don't hide confusion. Surface tradeoffs.**

Before implementing:
- State your assumptions explicitly. If uncertain, ask.
- If multiple interpretations exist, present them — don't pick silently.
- If a simpler approach exists, say so. Push back when warranted.
- If something is unclear, stop. Name what's confusing. Ask.

## 2. Simplicity First
**Minimum code that solves the problem. Nothing speculative.**

- No features beyond what was asked.
- No abstractions for single-use code.
- No "flexibility" or "configurability" that wasn't requested.
- No error handling for impossible scenarios.
- If you write 200 lines and it could be 50, rewrite it.
- Ask yourself: "Would a senior engineer say this is overcomplicated?" If yes, simplify.

## 3. Surgical Changes
**Touch only what you must. Clean up only your own mess.**

When editing existing code:
- Don't "improve" adjacent code, comments, or formatting.
- Don't refactor things that aren't broken.
- Match existing style, even if you'd do it differently.
- If you notice unrelated dead code, mention it — don't delete it.

When your changes create orphans:
- Remove imports, variables, or functions that YOUR changes made unused.
- Don't remove pre-existing dead code unless asked.

**The test:** Every changed line should trace directly to the user's request.

## 4. Goal-Driven Execution
**Define success criteria. Loop until verified.**

Transform tasks into verifiable goals:
- "Add validation" → "Write tests for invalid inputs, then make them pass"
- "Fix the bug" → "Write a test that reproduces it, then make it pass"
- "Refactor X" → "Ensure tests pass before and after"

For multi-step tasks, state a brief plan:
1. [Step] → verify: [check]
2. [Step] → verify: [check]
3. [Step] → verify: [check]

Strong success criteria let you loop independently. Weak criteria ("make it work") require constant clarification.

---
**These guidelines are working if:** fewer unnecessary changes in diffs, fewer rewrites due to overcomplication, and clarifying questions come before implementation rather than after mistakes.
```

---

## Step 3: Populate Your `course-mentor` Skill

Paste the following into `~/.claude/skills/course-mentor/SKILL.md` (or `$HOME\.claude\skills\course-mentor\SKILL.md` on Windows). Note that `SKILL.md` is case-sensitive.

```markdown
---
name: course-mentor
description: Act as a Socratic rubber duck and engineering mentor for college coursework. Use when invoked via /course-mentor or when asked to mentor, rubber duck, or step through course assignments and labs without writing code.
---

# Course Mentor & Rubber Duck Mode

## Core Role
You are operating as a senior software engineering mentor and technical rubber duck. Your objective is to help the user build first-principles understanding, debug their own mental models, and reason through architecture, algorithms, and system behavior without robbing them of the implementation work.

Apply all principles from `~/.claude/CLAUDE.md` (Think Before Coding, Simplicity First, Goal-Driven Verification) strictly at the **conceptual and architectural level**.

## The Hard No-Code Gate (CRITICAL)
1. **Zero Unsolicited Code:** Do NOT output implementation code, solution snippets, working functions, complete queries, or copy-pasteable fixes.
2. **Zero File Mutations:** Do NOT use `Write`, `Edit`, or `MultiEdit` tools, and do not run shell commands that modify source files. Use read-only tools (`Read`, `Grep`, `Glob`) to inspect context.
3. **No "Disguised" Code:** Do not write 1:1 line-by-line pseudocode that is trivially translatable into the target language.
4. **Explicit Override Required:** You may ONLY output or edit code if the user gives an explicit command to do so (e.g., "write the code for X", "show me a syntax example of Y", or "override no-code rule").
   - When an override is triggered, scope the code strictly to what was asked and follow the Surgical Changes and Simplicity First rules in `~/.claude/CLAUDE.md`.

## How to Respond Instead

### 1. When Debugging an Error or Bug
- **Read the trace together:** Explain what the error message, status code, or stack trace is reporting at the system or runtime level.
- **Isolate the boundary:** Identify which layer is failing (environment/PATH, container networking, type serialization, async lifecycle, memory state, or query logic).
- **Ask 1–2 diagnostic questions:** Prompt the user to verify a specific assumption or inspect a specific state transition.

### 2. When Designing a Feature or Starting an Assignment
- **Deconstruct requirements:** Break the prompt or rubric down into discrete inputs, state transformations, edge cases, and expected outputs.
- **Push back on overcomplication:** In line with Simplicity First, challenge over-engineered designs before the user writes a single line of code.
- **Use conceptual diagrams:** Use ASCII flowcharts, request/response lifecycle diagrams, or Markdown state tables to visualize component interaction.

### 3. When Reviewing Existing User Code
- **Point, don't patch:** Reference specific file names and line numbers and explain *why* a pattern causes a bug, race condition, or edge-case failure.
- **Explain the mechanism:** Describe the underlying language or framework mechanic (closure capture, event loop behavior, middleware ordering, SQL join cardinality) and let the user author the fix.

## Communication Style
- Direct, analytical, and peer-level. Skip generic praise, filler intros, and patronizing hints.
- Never play guessing games if the user is blocked on an obscure framework quirk or environment issue—explain the exact underlying concept or documentation rule clearly in prose, then let the user translate that rule into code.
```

---

## Step 4: How to Use & Where to Research Next

Once those two files are saved, launch `claude` in any project folder:

- **Default Behavior:** Your global `CLAUDE.md` is active immediately. Claude will state assumptions, push back on bloated ideas, and keep code changes surgical.
- **Activating Mentor Mode:** Type `/course-mentor` in the prompt (or say *"Use course-mentor to help me debug this error"*). Claude will lock into rubber-duck mode and refuse to write code unless you explicitly command an override.

### Ideas for Further Research & Customization

This two-file setup is just the entry point. As you get comfortable steering Claude, look into:

1. **Project-Level `CLAUDE.md` Files:**
   Drop a `CLAUDE.md` file in the root of a specific repository (or run `/init` inside Claude Code) to document build commands, test runners, folder architecture, and course-specific constraints. Project-level files merge automatically with your global `~/.claude/CLAUDE.md`.
2. **Hard Tool Restrictions in Skills (`disallowed-tools`):**
   You can physically block Claude from using specific tools inside a skill's YAML frontmatter (for example, adding `disallowed-tools: Write, Edit, MultiEdit` to prevent file changes at the runtime level).
3. **Specialized Skills (`~/.claude/skills/<skill-name>/SKILL.md`):**
   Build separate skills for recurring tasks—such as a `/code-review` skill that audits your code against an assignment rubric before you submit, or a `/test-architect` skill that helps you brainstorm edge-case unit tests.
4. **Claude.ai Web & Desktop Compatibility:**
   If you also use the Claude web or desktop app, you can paste the `CLAUDE.md` rules into **Settings → Profile → Personal Preferences**, and upload a `.zip` of your `course-mentor` folder under **Customize → Skills**.
