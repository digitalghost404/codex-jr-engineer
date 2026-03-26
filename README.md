# Codex Jr. Engineer

Give Claude Code a junior engineer that works in the background.

This is a **skill** — a small instruction file that teaches Claude Code a new behavior. Once installed, Claude Code will automatically hand off simple, repetitive tasks to OpenAI's Codex CLI while it focuses on the hard stuff. You don't manage two tools. You don't switch windows. You just work, and more gets done.

## Why This Exists

If you use Claude Code heavily, you've probably seen the **rate limit** message — Claude pauses because you've used too many tokens in a short period. Every task you give Claude Code, whether it's designing a complex system or just adding comments to a file, uses the same budget.

The insight: **simple tasks don't need Claude Code's full power.** Writing tests, adding documentation, renaming variables — these are mechanical. OpenAI's Codex CLI can handle them just fine, and it runs on a completely separate budget.

This skill teaches Claude Code to recognize those simple tasks and automatically send them to Codex in the background, while it keeps working on the complex stuff. The result: you get more done before hitting rate limits, and you're never sitting idle.

## What It Looks Like In Practice

You won't see anything different in how you work. You talk to Claude Code the same way you always do. Behind the scenes, this is what happens:

```
You: "Execute this plan."

Claude Code: Using codex-jr-engineer to dispatch mechanical tasks to Codex.
             Tasks 2, 3, 5 are Codex-eligible. Dispatching while working
             on others directly.

             Dispatched to Codex: unit tests for pkg/config/.
             Continuing with Task 1 (command parser design).

  [Claude Code works on Task 1 — the complex architecture work]

             Codex completed: unit tests for pkg/config/.
             Reviewed — clean pass. Tests passing. Integrated.

  [You never switched windows. You never touched Codex.]
```

## How It Works

When Claude Code starts working through a plan (a list of tasks), the skill kicks in:

1. **Classifies** each task — "Is this simple enough for Codex, or does it need Claude Code's full attention?"
2. **Dispatches** simple tasks to Codex in the background — immediately, not after the complex work is done
3. **Reviews** everything Codex produces before it touches your code (spec compliance, naming conventions, security scan, test execution)
4. **Integrates** work that passes review, or throws it away and redoes it if it doesn't

Nothing Codex writes enters your project without Claude Code checking it first.

### What Goes to Codex (Simple, Mechanical Tasks)

- Writing tests for existing code
- Adding documentation comments
- Adding struct tags or type annotations
- Renaming variables across a file
- Converting code from one pattern to another
- Generating boilerplate (scaffolding, config files)

### What Stays in Claude Code (Complex Tasks)

- Designing new systems or APIs
- Debugging tricky issues
- Changes that touch many connected files
- Anything that needs internet access or external tools
- Work that requires back-and-forth conversation

## Installation

### What You Need First

1. **Claude Code** — Anthropic's coding CLI. If you don't have it yet, see the [Claude Code docs](https://docs.anthropic.com/en/docs/claude-code).

2. **Codex CLI** — OpenAI's coding CLI. Install it with:
   ```bash
   npm install -g @openai/codex
   ```
   You'll need an OpenAI API key set as `OPENAI_API_KEY` in your environment.

3. **A plan-based workflow** — This skill activates when Claude Code is working through a structured plan (a list of tasks). If you use the [superpowers](https://github.com/superpowers-ai/superpowers) skill pack, you're already set. If not, any workflow where Claude Code executes numbered tasks from a plan will work.

### Install the Skill

Run these two commands in your terminal:

```bash
mkdir -p ~/.claude/skills/codex-jr-engineer
```

```bash
curl -o ~/.claude/skills/codex-jr-engineer/SKILL.md \
  https://raw.githubusercontent.com/digitalghost404/codex-jr-engineer/main/SKILL.md
```

That's it. The skill is now installed globally — it works on every project, no setup needed per project.

### Verify It's Working

Start a new Claude Code session and ask:

```
What skills do you have available?
```

You should see `codex-jr-engineer` in the list.

## Security

A few things to know about how this works under the hood:

- **Codex runs in full-auto mode** — meaning it makes changes without asking you first. This is safe because Codex runs inside a **sandboxed environment with no internet access** by default. It can only read and write files in your project.
- **Everything gets reviewed** — Claude Code checks every line Codex produces before it enters your codebase. This includes a security scan for hardcoded secrets, unsafe functions, and unexpected file modifications.
- **Nothing is permanent until reviewed** — if Codex output doesn't pass review, it gets thrown away entirely. No partial work ever enters your code.
- **Always use version control** — run this on projects managed with git. If anything goes wrong, you can always revert.
- **Don't disable Codex's sandbox** — the safety model depends on it. Codex can't reach the internet, can't read files outside your project, and can't install packages.
- **Project files could influence Codex** — like any AI tool, Codex reads your project files to do its work. If a file contains adversarial instructions in comments, Codex might behave unexpectedly. The review step catches anomalous output, but be aware of this if you're working on untrusted codebases.

## Frequently Asked Questions

**Do I need to change how I use Claude Code?**
No. Just install the skill and keep working normally. When Claude Code is executing a plan and Codex is available, it will use the skill automatically.

**What if I don't have Codex installed?**
The skill checks for Codex at the start of every session. If it's not installed, the skill announces "Codex CLI not available" and Claude Code does everything itself. Nothing breaks.

**Does this cost money?**
Codex CLI uses your OpenAI API key, so Codex tasks are billed to your OpenAI account. Claude Code tasks are billed as normal through your Anthropic plan. The tradeoff: you spend a small amount on OpenAI to significantly extend your Claude Code session before hitting rate limits.

**What's the `.codex-dispatch.json` file that appears in my project?**
It's a temporary tracking file that records what tasks were sent to Codex. It's automatically added to `.gitignore` so it never gets committed. It's cleaned up when all tasks finish.

**Can Codex mess up my code?**
Codex can produce bad output — wrong logic, missing edge cases, style mismatches. That's expected. The skill's review step catches these issues. Bad output gets thrown away and Claude Code redoes the task itself. Your codebase only receives code that passes review.

**What's a "plan" in this context?**
A plan is a structured list of tasks that Claude Code works through step by step. Example:

```markdown
### Task 1: Build the user authentication module
Files: src/auth/login.go, src/auth/token.go
...

### Task 2: Write tests for the auth module
Files: src/auth/login_test.go
...

### Task 3: Add documentation to all exported functions
Files: src/auth/*.go
...
```

In this example, Task 1 is complex (Claude Code handles it). Tasks 2 and 3 are mechanical (dispatched to Codex).

## License

[MIT](LICENSE)
