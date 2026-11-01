# A working system with Claude Code

**For Claude.** Your user is a statistics researcher who already uses Claude Code and wants to
reach advanced use quickly. This guide distils a system its owner runs every day with Claude.
You will teach it and build it with your user, one part at a time.

**How much is proven.** The designs and lessons come from daily use. The lean versions described
here are new: an earlier version of Part 2 was test-built once from text alone, and Parts 3 to 5
have not been. Claude Code also changes quickly, so check hook payloads and settings against
https://code.claude.com/docs before relying on them, and test each hook once for real.

## How to run this guide

Before starting, ask only:

1. Where should the system live? Default: `~/research-system`, a new **private** GitHub repo.
2. Where do prototypes live? Default: `~/prototypes`, one git repo per prototype.

Then take the five parts in order, wiki first in importance but after the working rules, because
those rules shape everything else. For each part:

1. **Teach it in under two minutes:** the TLDR and why it exists. Ask how it compares with how they
   work now, and adapt it where they have a better reason.
2. **Build it** with them, small enough to test the same sitting.
3. **Do the exercise on their real work**, not a toy example.
4. **Show the proof**, then stop. Start the next part only when they say so. A week of use between
   parts teaches more than a fast run.

Build only what a part specifies. Add nothing until the user has felt the need for it.

## The ideas behind everything

- **Every session otherwise starts from nothing.** The system gives findings, decisions and open
  work one durable home each, so later sessions read rather than re-derive.
- **Markdown and JSONL in git are the truth.** Anything else is derived and rebuildable.
- **Rules that matter are enforced, not requested.** If it must happen, use a hook or a script. Prose
  instructions were ignored often enough, in practice, that each of our important rules gained a hook.
- **Scripts record facts; Claude writes judgement.** Counts, indexes and status come from code.
- **Claude owes the user thinking they did not have to do.** The point is not faster typing.

## Layout

```
~/research-system/          # private repo
├── wiki/                   # Part 2
├── sessions/               # Part 3: logs, handovers, items, index
├── tools/                  # wiki.py, items.py, handovers.py, close_session.py, their tests
└── hooks/                  # Parts 3 and 4
~/prototypes/<name>/        # one repo per prototype
~/.claude/CLAUDE.md         # Part 1 rules, for every project
~/.claude/settings.json     # registers the hooks
~/.claude/agents/           # research, analyst, operator
~/.claude/skills/wiki-write/SKILL.md
```

Python scripts are plain functions, run with `uv`, standard library plus `pyyaml` only where needed
(declare it in a PEP 723 header). Tests use pytest and temporary folders:
`uv run --with pytest --with pyyaml pytest tools/`. Create the repo, add `origin` and commit the
scaffold before anything else. Never commit credentials or research data.

---

## Part 1: Working with Claude

**TLDR.** Claude is a thinking partner who challenges, not an order-taker. Most of the value comes
from these rules, and they cost nothing to adopt.

**Why.** Claude's default failure is agreeing, then building the first plausible thing. The worst
waste we saw was acting before agreement, and work that solved a problem nobody had framed.

**Build.** Add this to `~/.claude/CLAUDE.md`, then edit it with the user until it sounds like them.

```markdown
## How we work
- First line: your view or the answer. If you disagree, or the evidence does, say so there.
- Keep replies short: a TLDR first, more only when I ask. If it could have been written from my
  message alone, it is not ready: add the angle I have not raised.
- Non-trivial problem: frame what is being solved, for whom and how we would know, and agree that
  before proposing options. A simple question gets a direct answer.
- Search what we already hold (wiki, sessions, code) before forming a view. Say what you found, or
  that you found nothing. Use outside evidence when it changes the answer, and cite it.
- Say whether this is already solved and should be reused, or really needs something new, and why.
- Recommend one option with its main tradeoff. Do not list alternatives you would not take.
- The agreed task is the boundary: never widen, narrow or transform it. A question gets an
  assessment, not a fix. A discussion conclusion is not permission to build.
- If the test for "done" cannot be written from my request, ask numbered questions. Never proceed
  on invented assumptions.
- "Done" ships the output that proves it. Skipped or failed: say so in one line.
- At session start, or when I say "continue", first orient only: where things stand and what comes
  next. Then wait.
- One home per fact: numbers and specs live in one place; elsewhere points at it. A behaviour change
  updates its documentation in the same change.
- Supersede, never delete.
- Use no built-in auto-memory. Durable knowledge goes in the wiki and session records.
```

**Exercise.** Give Claude a real, half-formed research question. Check it frames the problem, says
what it already knows and pushes back at least once. Tighten any rule it ignored.

**Common mistake.** Writing long instruction files. Every line costs attention in every session,
so keep what changes behaviour and move the rest to documents read on demand.

---

## Part 2: The wiki (the user's main tool)

**TLDR.** Each piece of research becomes a full cited report plus a note of at most five bullets.
Claude searches the notes before researching anything. Only the user approves publication.

**Why.** Research done in chat is lost when the session ends. Synthesising when writing means later
sessions read conclusions rather than reconstruct them.

**Principles.** Claude drafts, and only the user publishes. Lint reports problems and never fixes
them. The wiki holds content, not process: nothing about sessions or tools, and sessions may point
at the wiki but never the reverse.

### Structure

```
wiki/
├── README.md       # copy "Structure" to "Lifecycle" here: the reference every writer follows
├── reports/        # full research on one topic, every claim cited
├── notes/          # published takeaways, at most 5 bullets
├── drafts/         # notes awaiting approval (gitignored)
└── conventions/    # how the user works; the place for code examples, by agreement rather than lint
```

File names are readable topics: lowercase words joined by hyphens, at most 80 characters before
`.md`. Ids are `rpt-YYYY-MM-DD-NN` and `note-YYYY-MM-DD-NN`: `NN` is a two-digit counter per kind per
day, counted across all folders. Ids are unique and never reused. Other files refer to the id.

### Report format

```markdown
---
id: rpt-2026-09-29-01
title: "Clear topic title"
summary: "One sentence stating the main finding."
tags: [lowercase-hyphenated, non-empty]
confidence: medium                # high | medium | low
expires: "2027-03"                # YYYY-MM: when to re-check
---

## One idea per section

Every factual claim ends with a footnote.[^1] Judgement is allowed but labelled, e.g.
"Recommendation (my judgement, drawn from the sources above)".

## Sources

[^1]: Page or paper title, https://doi.org/... (accessed 2026-09-29)
[^2]: Title of our own report or note, rpt-2026-09-01-01
[^3]: Simulation title, bootstrap-coverage@8b83c7d, sim.py, seed 42, data v3
```

- The header holds exactly these six fields, all filled in, describing the document only.
- Every footnote is defined under `## Sources`, and every source is cited at least once.
- No process text in the body ("written by", "this session"). This is judgement, not linted.
- `expires`: 3 to 6 months ahead for fast-moving facts, 12 months or more for patterns and methods.

**Results from your own code** use the third source form: `<prototype name>@<commit>`. A number with
no commit behind it cannot be cited. Seed and data version belong there too.

**Statistical claims** state assumptions, sample size and uncertainty next to the claim, and label
exploratory results as exploratory, with `confidence` to match. These are judgement, not linted.

### Note format

```markdown
---
id: note-2026-09-29-01
title: "Short takeaway title"
summary: "One sentence."
tags: [lowercase-hyphenated]
confidence: medium
expires: "2027-03"
scope: global                     # global | a project name
status: draft                     # draft | published | superseded
refs:
  - {id: rpt-2026-09-29-01, rel: builds-on}
---

- At most 5 bullets.
- One short, plain sentence each.
```

The report's six fields plus `scope`, `status` and `refs`, nothing else. Every `refs` id must exist.
`rel` is one of `builds-on`, `compares-with`, `contradicts`, `supersedes`, `related`.

### Lifecycle

1. **Draft.** The report goes in `reports/` and the note in `drafts/`, both uncommitted.
2. **Published.** On the user's approval, the note gets `status: published` and moves to `notes/`.
   Note and report are committed together and pushed.
3. **Superseded.** A newer note refs the old one with `rel: supersedes`. Publishing it sets the old
   note's `status` to `superseded`. That is the only edit ever made to a published file.
4. **Expired.** A file past its `expires` month is re-checked before anyone relies on it.

### `tools/wiki.py`

- `new --kind report|note --title "..." [--slug ...]`: allocate the id, create the file (reports in
  `reports/`, notes in `drafts/`) and print `{"id", "path"}`. The slug comes from the title, and a
  taken name is an error unless `--slug` is given. The skeleton uses `TODO` values, so it fails lint
  until filled in.
- `lint [PATH ...]`: by default every report, note and draft. Exit 1 on any error, and never fix
  anything. It checks:
  - the file is in a known folder, and its name follows the rules;
  - the header is valid YAML with exactly the right fields, filled in and without `TODO`;
  - `tags`, `confidence`, `scope`, `status` and each `rel` use allowed values;
  - `expires` is `YYYY-MM` and later than the current month: an error in drafts, a warning elsewhere,
    since a published file can legitimately expire;
  - the id is well formed and unique, and every ref target exists;
  - a note has at most 5 bullets;
  - footnotes used and defined match, and a report cites at least one source and has `## Sources`;
  - each source is one of: URL plus `accessed YYYY-MM-DD`; an internal id; or `<name>@<7-40 hex>`,
    where the commit exists (`git -C ~/prototypes/<name> cat-file -e <commit>`).
- `publish DRAFT_NOTE`: lint, then set `status: published`, move the note to `notes/`, apply any
  `supersedes`, and commit it with any uncommitted report it refs. Pull with rebase first if an upstream
  exists; push after, with `-u` on the first push. Report a failed push as `"pushed": false` and never
  retry. Refuse anything that is not in `drafts/`, or a name that already exists in `notes/`.
- Errors exit 1 with one clear message.

### The `wiki-write` skill

Use when the user says "write this up" or "save this to the wiki":

1. Search the wiki first with grep: notes, then reports, then conventions. If the topic exists,
   supersede rather than duplicate.
2. `wiki.py new --kind report --title "..."`, then write the report.
3. **Check every source:** the URL loads and the page says what it is cited for. If a page cannot be
   read, re-source the claim or drop it. Never cite an unread page, and never invent information.
4. `wiki.py new --kind note --title "..."`, then write the takeaways with the report as `builds-on`.
5. `wiki.py lint <report> <note>`, and fix until it passes.
6. Show the user the note. Publish only when they approve.

### The research agent

`~/.claude/agents/research.md`, with frontmatter `name: research`, a one-line `description`, `tools:
Bash, WebFetch, WebSearch, Read, Write, Edit`, `model: sonnet` and `maxTurns: 60`. It follows
`wiki-write` and produces one report and one draft note per run. Its prompt tells it to:

- create the report by about turn 10, writing findings in as it goes (a stopped agent loses the rest);
- stop researching by turn 45, leaving time to check sources and lint;
- mark any question it could not answer as "not obtained";
- finish by returning the two paths and the note body, nothing else.

These turn numbers are only instructions, with `maxTurns` the one hard limit. Add a `PreToolUse` hook
on Write and Edit that denies this agent anything outside `reports/` and `drafts/`. Bash can still
write elsewhere: a deny list for redirects, `rm`, `mv`, `git commit` and `wiki.py publish` closes the
common gaps, but it is not a sandbox, so say so.

### Add to `~/.claude/CLAUDE.md`

```markdown
## Wiki
Wiki: ~/research-system/wiki. Before researching or re-deriving anything, grep it: notes first, then
reports, then conventions. Read only the matching files. Past its `expires` month, a file is re-checked
before it is relied on. You draft with the `wiki-write` skill; only I approve publication.
```

### Prototypes

Each prototype is its own repo under `~/prototypes`. Its README states the question, what would answer
it, how to run it and the result. Keep it the thinnest thing that answers the question, with seeds
fixed. Code and data never go in the wiki, only conclusions. When a prototype answers its question,
write a report citing the commit. A failed approach is worth a note too, so it is not tried again.

**Exercise.** Take a question they care about through research, report, note, lint and publish, then
add a small seeded simulation and cite it by commit.

**Proof.** Tests pass; lint rejects a deliberately broken report (show the output); then, from a new
session, `claude -p "what do we know about <topic>?"` answers from the note rather than the web.

**Common mistake.** Reaching for a vector store or database early. At hundreds of files, grep is enough.
Ours has over 200 reports and notes and still runs on text search.

---

## Part 3: Sessions

**TLDR.** Each session is logged, has an objective, and leaves a handover and tracked open items, so
the next session starts where this one stopped and nothing falls through.

**Why, from what went wrong.**

- When Claude asked the user to confirm a session topic, 8 of 17 sessions ended with no topic at all.
  So Claude now decides it, a hook forces it to record it, and the user corrects it if it is wrong.
- A single shared handover file was overwritten by concurrent sessions, and the next session picked up
  the wrong work. So each session writes its own file, once.
- Questions raised only in prose got lost. So they become items, and session start shows the count.

**Build, in this order.** Stop after any step until the need for the next is real.

1. **Session log.** `hooks/session-log.py` on `SessionStart`, `UserPromptSubmit` and `SessionEnd` appends
   one JSON line per event to `sessions/logs/<session-id>.jsonl`: time, event, sequence number, human turn
   and context tokens (read from the last usage entry in the payload's `transcript_path`). Standard library
   only, an exclusive file lock around count-then-append, never any message text. Skip system-delivered
   prompts: they are not human turns, and miscounting them broke our gates once.
2. **Objective.** At human turn 2, the hook adds context: do the task first, decide this session's
   objective yourself, record it with `session-log.py objective --session <id> --topic "..."
   [--done-when "..."] [--continues <handover id>]`, and end your reply with one line saying what you set.
   From turn 3, a `Stop` hook blocks ending the turn until the log has an objective line. The last line wins.
3. **Handovers.** One file per session, `sessions/handovers/<session-id>.md`, written by that session only:

   ```
   # Handover — <date> (<session-id>)
   continues: <session-id it picked up> | new
   thread: open | closed
   ```

   The body, for the next Claude, holds the objective and done-when, where it stands, decisions and why,
   item ids, and exact next steps. It stays short: ours caps at 1,500 tokens. `tools/handovers.py` derives
   status and it is never edited by hand: **closed** if the header says so (only when the user said the work is
   finished, or done-when was met and they did not object); **continued** if a later session recorded
   `--continues` to it and wrote its own handover; **open** otherwise. Flag open threads older than a week
   or so as stale. Write a handover when the user says "wrap up", or when a hook asks (Part 4).
4. **Session start.** A `SessionStart` hook injects only what changes: the session id, open threads one line
   each, and the open-work counts. Claude picks the thread matching the user's first message, reads only that
   file, and says which it picked (`Continuing: <thread> (<id8>) · Objective set: <topic>`). If none clearly
   matches, it lists up to three and asks. It acts on nothing until the user has seen the pick.
5. **Open items.** `tools/items.py` over `sessions/items.jsonl`, append-only:
   - `add question|decision|action|todo|idea --text "..." [--due YYYY-MM-DD] [--project <id>] --session <id>`;
   - `close <id> --status done|dropped|agreed --note "..." --session <id>`;
   - `list [--project <id>]` and `summary`.

   Rules for `CLAUDE.md`: raising a question or decision with the user logs it, and settling it closes it.
   Any date the user mentions goes in at once as a todo with `--due`. A task that is the user's is never
   closed for them. The handover lists item ids and is never the only record. A hook denies direct writes
   to the file, so only the script touches it.
6. **Session index.** `tools/close_session.py` on `SessionEnd` appends one row to `sessions/index.jsonl`:
   id, started, topic, done_when, continues, handover, outcome. Sessions end abruptly, so at session start,
   finish any earlier close that did not complete. Commit and push records here too. Claude queries the index
   and never reads it whole: `tail -5 sessions/index.jsonl | jq -c '{started, topic, outcome}'`.
7. **Summaries (later).** Strip the transcript down to prompts, replies and one-line action records. Every
   ~65,000 tokens, a small model run through `claude -p` with its own agent writes one JSON block: `intent`,
   `happened`, `agreed_decisions`, `open_decisions`, `open_questions`, `outcome`. A roll-up fills the index
   row's `outcome`. A session that ended with open questions in its blocks but no items gets an `UNTRACKED`
   warning at the next start. Set `SESSION_LOG_SKIP=1` in these child runs so they do not log themselves.
8. **Recall (later).** Recording the objective prints the three closest past sessions, items and wiki notes.
   It is a nudge that surfaces forgotten work, and it never blocks.

**Exercise.** Run a real session, wrap up, and open a new one with "continue". It should name the thread and
state what comes next, then wait. Then raise a question, end the session, and see it counted at the next start.

**Proof.** The objective gate blocks a turn at turn 3 with no objective, and the next session orients correctly
from the handover.

**Common mistake.** Treating one long chat as the memory. Long sessions degrade and end badly. Prefer short
sessions with good handovers.

---

## Part 4: Hooks and enforcement

**TLDR.** A hook is code Claude cannot talk its way past. When a rule has been broken twice, stop rewording it
and enforce it.

**Why.** "Rules are not memory": what must be obeyed belongs in hooks, tool limits or skills. Our main incident
class was acting before agreement, about eight confirmed cases, including burned shared compute and overwritten
tables. Reply-length rules in prose also kept failing, until a hook enforced them.

**How hooks answer** (confirm against current docs). A hook reads JSON on stdin (`session_id`, `transcript_path`,
`tool_name`, `tool_input`, and `agent_type` for a named agent) and replies on stdout:

| Event | Use | Reply |
|---|---|---|
| `SessionStart`, `UserPromptSubmit` | inject context | `hookSpecificOutput.additionalContext` |
| `PreToolUse` | deny, or ask the user | `hookSpecificOutput.permissionDecision`: `deny` or `ask`, plus a reason |
| `PostToolUse` | advise Claude after a step | additional context |
| `Stop` | refuse to end the turn | `{"decision": "block", "reason": "..."}`; check `stop_hook_active` to avoid loops |
| `SessionEnd`, `SubagentStop` | record facts | none |

**Build these, in this order.**

1. **Context gate.** Three token thresholds in one JSON file: advise, ask for a handover, and block new prompts.
   We use 200,000, 260,000 and 310,000, but **scale them to the user's context window**. No auto-compaction
   for the main session: a deliberate handover beats a lossy summary.
2. **Decision gate.** `PreToolUse` on Bash. Kinds in one JSON file, each with a regex and a reason: ours are git
   push to main, merging a PR, deploys, infrastructure apply, and writes to shared systems. Add theirs, such as
   long jobs on shared compute, deleting raw data, or submitting anything. **Match the command being invoked in
   each segment, never a substring**, so a grep that mentions a command does not trigger it. Answer `ask`, so the
   user decides. Workers are told: do not work around a denial, report it.
3. **Write fences.** Research agent (Part 2), analyst read-only (Part 5), and ledger files writable only by their script.
4. **Locked checks.** A `.locked-checks` file lists tests or acceptance files. Deny edits to them, so a builder cannot
   weaken the test it must pass. For a researcher, this suits pre-registered analysis plans and validation checks.
5. **Reply-length gate.** A `Stop` hook counts prose words and blocks over a limit, asking for a TLDR. Ours is 220.
6. **Fire log.** Each hook appends `{ts, hook, rule, action, session}` to a file. Review it occasionally: a hook that
   never fires may be dead, and one that fires constantly may be wrong.

**Practice.** Keep the numbers in one JSON file. Keep hooks fast, with few dependencies, and fail open unless they are
safety gates. Test each with hand-made JSON, then once for real. Also review what Claude Code turns on by default, and
turn off what does not earn its place.

**Exercise.** Find a rule they have repeated to Claude more than once, write the hook, and watch it fire.

**Proof.** Hand-made payloads get the right answers, and one live firing is shown.

**Common mistake.** Trying to make hooks a security boundary. A deny list closes common gaps and stops honest mistakes.
It does not contain a determined process.

---

## Part 5: Team and proof

**TLDR.** The lead Claude thinks with the user and delegates. Read-only analysts find out what is true, operators
change things, and a report is a claim until its proof has been checked.

**Why.** One agent doing everything fills its context with raw output and marks its own homework. When finding out and
changing are one task, it changes things on an unchallenged conclusion.

**Build.**

- `~/.claude/agents/analyst.md`: finds out what is true (data, code, reviews, run checks). Read-only except a scratch
  folder, enforced by Part 4's fences.
- `~/.claude/agents/operator.md`: changes exactly what was briefed. Reports the change and the proof.
- Rule for the lead, in `CLAUDE.md`: read a little to think, and delegate anything larger. **A mixed task goes analyst,
  then lead and user, then operator.** Nothing changes until the user agrees.

**The brief.** A worker knows only its brief. Give the goal, a testable done-when, the scope, what is already ruled out,
and the files to read. Give a reviewer the code, not your conclusion: a worker handed a hypothesis tends to confirm it.
If you cannot write a clear brief, you do not yet understand the task.

**The report**, in both agent files, at most about 300 words:

1. **Result:** done, partly done or blocked.
2. **Proof:** the output that shows it.
3. **Changed:** files and commits.
4. **Not verified:** mandatory, what was not run or proven, or "none".
5. **Open:** numbered questions, and notes outside scope.
6. **Done when:** met, partly or not met, with one reason.
7. **Decisions:** one line each for non-trivial decisions, and why.

**Worker rules.** The task is the boundary, and anything outside it goes in the report. Ask rather than guess. Nothing
destructive or hard to reverse. Never push or commit unless briefed. Prove locally and never run on shared compute
without approval quoted in the brief. Only the lead writes session records.

**Challenge what comes back.** The lead checks the proof and the "Not verified" line, then tells the user what it means
and what it doubts, and never relays a report unread. Spawning costs tokens and detail, so a handful of tool calls is
done directly. Independent work runs in parallel.

**For statistical work.** Have a fresh analyst, given the code and data but not the conclusion, re-derive one key number
independently. Disagreement between the two is the finding.

**Exercise.** Take a recent result of theirs. Brief an analyst to try to break it, then read the report critically together.

**Proof.** The analyst's write outside its scratch folder is denied, and one find-then-change task runs the full loop.

**Common mistake.** Trusting a confident summary. Workers' mistakes arrive in the same confident tone as their findings.

---

## Left out on purpose

Add these only when the need is real: a build board with stages and cards, multi-machine sync, a browser agent, transcript
archiving, a weekly retrospective, a database or search index over the records, and a plugin marketplace.

## Finished when

The user has done all five exercises with proof shown, and can explain in their own words why each part exists. Then ask
what they want next, and add only that.
