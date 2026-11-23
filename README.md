# critique

An agent skill that has an independent reviewer attack an answer before you rely on it.

An assistant that checks its own work tends to agree with itself: the reasoning that produced the answer is
still in its context. `/critique` hands the answer to a **fresh subagent** that sees only the task and the
answer, tells it to find what is wrong and to verify load-bearing claims against the real code, data or
sources, then revises. It keeps the critiques that hold up, pushes back on the ones that don't, and stops
when a round finds nothing material. You get three parts: the initial answer, what the critic raised and how
each point changed the answer (or why it was rejected), and the final answer in full.

For a written document or analysis, such as a research write-up, a memo or a report, it runs three reviewers
built for that: one tests the claims the conclusion rests on, one fact-checks every figure and quote against
its source, and a "newcomer" reads only the page and lists everything someone new to the field couldn't
follow. You get the same three parts. It revises a document it wrote itself; for one you hand it, it lists
the findings and recommends changes, and rewrites your text only if you ask.

## Install

The skill is the folder itself. Clone it where your client looks for skills:

| Client | Command |
| --- | --- |
| Claude Code (all projects) | `git clone https://github.com/rballentine/critique ~/.claude/skills/critique` |
| Codex (all projects) | `git clone https://github.com/rballentine/critique ~/.agents/skills/critique` |
| One project only, or Claude Code on the web (claude.ai/code) | copy the files, without the `.git` folder, into the project's `.claude/skills/critique` (Claude Code) or `.agents/skills/critique` (Codex) and commit them; a cloud session reads skills from the repository |

Update a clone with `git -C <that folder> pull`, or copy the files again. It needs a client that can start a
subagent: Claude Code's Agent tool, or Codex with subagents enabled (the default in current releases). Where
none is available it says so rather than pretending to critique itself.

## Use

- `/critique` (Claude Code) or `$critique` (Codex) with no task critiques the assistant's last answer.
- `/critique <task>` does the task, then critiques and revises it.
- `rounds=N` allows up to N critique-and-revise rounds (default 1; for a document, one to three by its
  size); it stops early once a round finds nothing material.
- Ask it to review a document or analysis to get the claim-testing, fact-checking and newcomer reviewers.

A project's own instructions can make it a standing step, for example "run critique on the draft before
any hard piece of work is final". It then reports in one line what the critique changed instead of the
three-part summary.

Each critic is a separate model run, and a critic that checks sources reads a lot, so a round can cost more
than producing the answer did. Reviewing a document runs three reviewers a round.

The critic is told to stay read-only; unless your client offers a read-only agent type, that rests on the
instruction alone.

## Contributing

Pull requests are welcome.

## License

[MIT](LICENSE).
