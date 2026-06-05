# /extract-prompt

> ## ⚠️ Before you start
>
> Installing a Claude Code skill is just copying one folder into `~/.claude/skills/`. Your AI agent can do all of it. **The only human step:** after install, **restart Claude Code** so it picks up the new skill, then type `/extract-prompt`.


> Analyzes a project and outputs a portable prompt that recreates it.

Part of [**vibekit**](https://ritual.industries) — a showcase of small tools and Claude Code skills that improve the experience of coding with AI.

## What it does

Point `/extract-prompt` at a project and it reverse-engineers the whole thing into a single, ordered **build instruction set** — not documentation, but a prompt precise enough that a fresh Claude session with zero prior context can one-shot rebuild the project from scratch.

It works by surveying every file, then mining the git history to find where the hard problems actually lived: the most-modified files, the workarounds, the non-obvious architecture calls, the platform quirks, and the exact styling values that define the feel. All of that gets distilled into an `ONE-SHOT-PROMPT.md` written to your project root — complete with environment setup, an ordered build plan with verification checkpoints, a numbered list of known pitfalls and *why* each fix exists, design specs down to the hex code, and a self-review checklist.

The result is portable. Hand the file to a teammate, drop it into a new repo, or feed it to a fresh Claude instance — anywhere the project's hard-won knowledge needs to travel without you in the loop.

## Install

This skill is a single `SKILL.md`. Install it like any other Claude Code skill — clone or download this repo, then copy the folder into your skills directory so that `~/.claude/skills/extract-prompt/SKILL.md` exists.

```bash
git clone https://github.com/brianharms/skill-extract-prompt.git
mkdir -p ~/.claude/skills/extract-prompt
cp skill-extract-prompt/SKILL.md ~/.claude/skills/extract-prompt/
```

Or, if you downloaded a ZIP, just move the unpacked `SKILL.md` into `~/.claude/skills/extract-prompt/`.

That's the whole install — there are no companion scripts or build steps. Once the file is in place, invoke it inside Claude Code by typing:

```
/extract-prompt
```

## Usage

Open Claude Code in the project you want to capture, then run the skill:

```
/extract-prompt
```

Claude will:

1. **Survey** the project — read the full file tree, every source file, `CLAUDE.md` / `README.md` / `package.json` / configs, and the git log (`git log --oneline -50`, most-modified files, and `-p` diffs of the top offenders).
2. **Identify the hard parts** — workarounds, non-obvious decisions, platform quirks, order dependencies, and styling precision.
3. **Generate the prompt incrementally** — writing `ONE-SHOT-PROMPT.md` section by section (constraints → environment → file structure → ordered build steps with verify checkpoints → design specs → pitfalls → self-review checklist) so large projects don't blow the output token limit.
4. **Quality-check and summarize** — then print how many files were analyzed, how many pitfalls/workarounds were captured, how many build steps were produced, the estimated token count of the prompt, and an honest flag on any areas where one-shot success is uncertain.

When it finishes, you'll have `ONE-SHOT-PROMPT.md` in your project root, ready to commit or hand off.

## Requirements / Dependencies

- **Claude Code CLI** — this is a Claude Code skill and runs inside it.
- **git** — the extraction leans heavily on `git log` history to find where the hard problems were. A non-git project still works, but the output will be weaker (no iteration history to mine).
- **Read / Write / Bash / Task tools** — used for surveying files, appending sections, and (for 10+ source-file projects) parallelizing the survey with Explore/Bash agents. These ship with Claude Code; no extra setup.

No OS restriction, no external services, no MCP servers, no sibling skills required.

## For AI coding agents

If you're an agent working **on** this skill, here's what you need to know.

**Repo layout:**

```
skill-extract-prompt/
├── SKILL.md      # the entire skill — the contract Claude reads at invocation
├── LICENSE       # MIT
├── .gitignore
└── README.md     # this file
```

**`SKILL.md` is the whole product.** It contains no code to execute — it's a natural-language instruction set that Claude follows when the user types `/extract-prompt`. Treat it as the contract: every behavior described above (the four phases, the section order, the output format) comes directly from this file. Edit `SKILL.md` to change behavior; there is nothing else.

**How to test changes:** install your modified copy into `~/.claude/skills/extract-prompt/SKILL.md`, open Claude Code in a real project (ideally one with a meaty git history), and run `/extract-prompt`. Verify it produces a well-formed `ONE-SHOT-PROMPT.md` in the project root and prints the closing summary (file count, pitfalls, build steps, token estimate, uncertainty flags).

**Invariants — do not break these:**

- **Output filename and location.** The skill must write to a file named exactly `ONE-SHOT-PROMPT.md` in the project root. Downstream workflows expect that name.
- **Incremental writing.** Never collapse the "write incrementally" guidance into a single-write instruction. Large projects produce prompts that exceed output token limits; the skill must `Write` section 1, then `cat >> ONE-SHOT-PROMPT.md << 'SECTION_EOF'` each subsequent section, keeping each append under ~3000 words. This is the load-bearing safety mechanism — removing it silently loses work on big repos.
- **Section order and contents.** Keep the canonical structure: constraints → environment setup → file structure → ordered build steps (each with a `Verify:` checkpoint and `PITFALL:` where relevant) → design specifications → known pitfalls/workarounds (each with *what goes wrong / fix / why*) → self-review checklist.
- **Git-history mining.** The skill's value comes from reading iteration history, not just current source. Don't strip the `git log` survey steps.
- **Output discipline.** The generated prompt must stay pure instruction — no editorializing, no vague language ("make it look nice"), exact dependency versions (never ranges or "latest"), and verbatim code for anything that took more than one iteration to get right. Preserve these rules; they're what make the prompt one-shot reliably.

When changing the skill, prefer tightening or extending these rules over loosening them — the entire premise is fidelity.

## License

MIT © 2026 Brian Harms / Ritual Industries — [ritual.industries](https://ritual.industries)
