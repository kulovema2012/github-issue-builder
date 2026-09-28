---
name: "github-issue-builder"
description: "Turn a rough feature/bug idea into a GitHub issue (or epic + child issues) with spec, acceptance criteria, agent-ready plan, test plan, and branch/worktree setup, then optionally file it. Use for 'make an issue', 'write a ticket', or any raw idea meant for GitHub."
---

# GitHub Issue Builder

Take a rough idea (one line, a voice-note ramble, a screenshot, a half-written ticket) and turn it into a GitHub issue that an AI coding agent (Codex, Claude Code) can pick up and finish without coming back with questions — including exactly which branch and folder (worktree) to work in, and where its pull request goes.

The issue is always written in **English**, in plain, easy-to-read language. Short sentences. No clever or cryptic phrasing. The reader may be an agent with no memory of the conversation, so everything it needs must be in the issue itself.

## Workflow

1. Capture the idea
2. Look at the repo (when one is reachable)
3. Ask 2-4 clarifying questions (only if needed)
4. Check the size — one issue or an epic with child issues?
5. Write the draft(s)
6. Show the draft, then create it on GitHub (issues + epic branch) or hand it over for copy-paste

### 1. Capture the idea

Restate the idea to yourself as: *who* has *what problem*, and *what change* fixes it. Decide the type: `feature`, `bug`, `improvement`, `chore`, or `docs`. A bug needs different information (steps to reproduce, expected vs actual) than a feature does.

### 2. Look at the repo

A plan that names real files and real test commands is far more useful to an agent than a generic one, so check for repo context before writing:

- If the user named a repo or the working directory is a git repo: read the README, the top-level layout, contributing/agent docs (`CONTRIBUTING.md`, `AGENTS.md`, `CLAUDE.md`), the package/build file to learn the test and lint commands, and grep for the code areas the idea touches.
- If `gh` is available and authenticated: `gh repo view <repo> --json defaultBranchRef` gives the default branch name (don't assume `main`), `gh issue list --search "<keywords>"` spots duplicates, and `gh label list` shows which labels exist.
- Keep this quick — you are looking for the files, patterns and commands the plan should point at, not reviewing the codebase.

If no repo is reachable, continue without it, assume the default branch is `main`, and mark any file paths in the plan as guesses (see the writing rules).

If you find an existing issue that already covers the idea, tell the user and ask whether to update that one instead of creating a duplicate.

### 3. Ask clarifying questions

Only ask when the answer would change the issue. Ask **at most 4** questions, in one round, using the multiple-choice question tool when available (offer concrete options, recommended one first). Good targets:

- **Who and why** — who uses this, and what problem it solves (if the idea only states a solution)
- **Scope edges** — which obvious neighbouring features are in or out
- **What "done" looks like** — the one behaviour that must work for the user to be happy
- **One-off or pattern** — when the request names a single instance of something that usually comes in plurals (one package, one plan, one report, one role), ask whether it stays the only one or more are coming. The answer decides between a one-off and a small reusable version, such as packages kept in a list an admin can add to. This is the most common reason a finished feature disappoints ("but I wanted more than one"), so ask it whenever it applies and the request doesn't already answer it.
- **Hard constraints** — must not break X, must use library Y, deadline, platform
- For bugs: steps to reproduce, expected vs actual result, environment

Do not ask about things you can decide sensibly yourself (naming, file layout, which test framework the repo already uses). Do not ask about things the repo already answers.

If the idea is already clear, skip this step. If the user is not around to answer, pick the most reasonable reading, and list each assumption under *Risks & open questions* so a human can correct it. Put an unanswered one-off-or-pattern question first there, phrased so it can be answered in one word, because it changes the scope more than anything else.

### 4. Check the size

An issue is too big when any of these is true:

- It contains several deliverables that could ship and be reviewed separately
- The plan would need more than about 8 steps
- It touches several unrelated parts of the system (e.g. new DB schema + new admin UI + public API + billing)
- An agent would likely need more than one focused session / one reasonably sized PR

When it is too big, **propose a split** before writing: one epic issue plus child issues that each deliver one working, testable slice. For each child, work out what it depends on:

- **Depends on nothing** → can start as soon as the epic branch exists
- **Depends on another child** → must wait until that child is merged into the epic branch
- Children with no dependency on each other can be built **in parallel**, each in its own worktree

Show the proposed split as a short list with the dependencies and the build order (which children can run in parallel), and let the user confirm or adjust it. Then write the epic and every child with the templates below.

### 5. Write the draft

Use the templates in the next sections. Then reread each draft as if you were the agent that has to implement it with nothing else to go on: Is anything ambiguous? Could two people read a criterion and disagree about whether it passes? Is it clear which branch to start from and where the PR goes? Fix those spots before showing it.

Issue numbers don't exist until the issues are created, so in the draft use placeholders such as `#<epic>` and `#<data-model>` and name branches with a slug only (e.g. `epic/<epic>-order-export`). Replace them with real numbers after creation (step 6).

### 6. Deliver

Show the full draft(s) (title + body) to the user in Markdown code blocks so they are easy to copy. Then:

**If you can create issues** (an authenticated `gh` CLI, or a GitHub connector/MCP tool), ask "Create this on `<owner/repo>`?" and confirm the repo. On yes:

*Single issue*
1. Write the body to a file and run `gh issue create --repo <owner/repo> --title "<title>" --body-file <file>` (a body file avoids shell-escaping problems with backticks and quotes).
2. Replace the `<n>` placeholders in its Branch & worktree section with the real number (`gh issue edit <n> --body-file <file>`).
3. Don't create the working branch — the agent creates it from the latest default branch when it starts.

*Epic + children*
1. Create the epic issue first, then each child issue with `Part of #<epic>` at the top of its body.
2. Create the **epic branch** on GitHub from the default branch — this is the only branch the skill creates:
   ```
   SHA=$(gh api repos/<owner>/<repo>/git/ref/heads/<default-branch> --jq .object.sha)
   gh api repos/<owner>/<repo>/git/refs -f ref=refs/heads/epic/<epic>-<slug> -f sha="$SHA"
   ```
3. Edit every issue body so all placeholders become real issue numbers and real branch names (epic child list, `Depends on #…`, branch names, the final `Closes …` line).
4. **Don't create the child branches or worktrees.** Each agent creates its own when it starts, from the latest epic branch. If they were created now, later children would miss the code merged by earlier ones.

For both: only add labels that already exist in the repo (`gh label list`); mention any suggested label that doesn't exist instead of creating it. Reply with the created issue URL(s), the epic branch name, and which child issue(s) can start right away.

**If you cannot create issues** (no `gh`, not authenticated, no connector): hand over the drafts as copy-paste text, plus the one command to create the epic branch if there is one, and say in one line what would let you file them directly next time (e.g. "run `gh auth login`" or connecting GitHub).

Never create an issue or branch without the user's go-ahead — once created it is visible to everyone on that repo.

## Branch and worktree conventions

These keep every agent's work separated and make the flow predictable. Use them unless the repo already has its own convention (check `CONTRIBUTING.md` and recent branch names) — then follow the repo.

| Thing | Name | Created by | Starts from | Merges into |
|---|---|---|---|---|
| Single issue branch | `<n>-<short-slug>` | the agent, when it starts | default branch | default branch |
| Epic branch | `epic/<epic>-<short-slug>` | this skill, when filing | default branch | default branch (once, at the end) |
| Child issue branch | `<n>-<short-slug>` | the agent, when it starts | latest epic branch | epic branch |
| Worktree folder | `../wt-<n>` (next to the repo folder) | the agent, when it starts | — | removed after merge |

Why an epic branch: all children share one goal, so their work collects in one place and the default branch never gets a half-built feature. Children with no dependency on each other can run in parallel in separate worktrees.

Important GitHub behaviour to reflect in the issues: `Closes #n` in a PR only auto-closes the issue when the PR merges into the **default** branch. Child PRs merge into the epic branch, so their issues stay open. The final epic → default-branch PR must list `Closes #<child1>, #<child2>, …, #<epic>` so they all close together.

## Issue template (single issue or child issue)

**Title:** a short imperative sentence under ~70 characters that says what changes, e.g. `Add CSV export to the orders table`. If the repo's existing issues use a prefix style (like `feat:` or `[Bug]`), follow it.

**Body:**

```markdown
<Child issues only: first line> Part of #<epic>

## Summary
<2-4 sentences: who has what problem, and what this issue changes to fix it. A reader should understand the point from this section alone.>

## Background
<Optional. Links, related issues, current behaviour, screenshots, why now. Delete the section if there is nothing useful to say.>

## Spec
<What the finished change does, described as behaviour, not code. Cover what applies:>
- **User-facing behaviour:** what the user sees and does, step by step
- **Inputs / outputs:** fields, formats, limits, defaults
- **Data / API changes:** new endpoints, schema or config changes, migrations
- **Edge cases:** empty state, errors, permissions, large inputs, concurrency
- **Constraints:** things that must not change or break

## Out of scope
- <Things someone might reasonably expect this issue to do, that it deliberately does not do. Each one short. Point to a follow-up issue if one exists.>

## Acceptance criteria
- [ ] <One observable, testable behaviour per line.>
- [ ] <...>

**Key scenarios**
<Only for the behaviours that are tricky or easy to get wrong — usually 1-3. Skip this sub-section if none.>

**Scenario: <name>**
- **Given** <starting state>
- **When** <action>
- **Then** <observable result>

## Branch & worktree
- **Depends on:** <#n — do not start until it is merged into the epic branch> / <nothing — can start now>
- **Start from:** `<epic/<epic>-<slug>` for a child | `<default-branch>` for a single issue>
- **Your branch:** `<n>-<slug>`
- **Set up (run from the repo folder):**
  ```
  git fetch origin
  git worktree add ../wt-<n> -b <n>-<slug> origin/<start-from branch>
  cd ../wt-<n>
  ```
- **Open your pull request into:** `<epic branch for a child | default branch for a single issue>` — <child: "not into <default-branch>">
- **PR description:** <child: "Part of #<epic>. Implements #<n>." | single issue: "Closes #<n>">
- **After merge:** `git worktree remove ../wt-<n>` and delete the branch `<n>-<slug>`

## Implementation plan
<Ordered, small steps an AI coding agent can follow one at a time. Each step leaves the project building and its tests passing.>

1. **<Step goal>**
   - Files: `<path>` <mark `(verify)` if not confirmed in the repo>
   - Do: <what to change, in a sentence or two>
   - Check: `<command to run>` / <what to confirm>
2. ...

**Notes for the implementer**
- Commands: install `<...>`, test `<...>`, lint `<...>`
- Follow the existing pattern in `<file>` for <...>
- Do not <constraint, e.g. change the public API of X>

## Test plan
- **Unit:** <what to cover>
- **Integration / end-to-end:** <what to cover, if relevant>
- **Manual check:** <short steps a human can do to see it working>

## Risks & open questions
- <Unknowns, risky parts, decisions still pending, and any assumptions made while writing this issue.>
```

For a **bug**, add a `## Bug details` section right after Summary with *Steps to reproduce*, *Expected result*, *Actual result*, and *Environment* (version, OS/browser, logs). The first acceptance criterion should be "the steps to reproduce no longer produce the bug", and the plan should start by writing a failing test that reproduces it.

## Epic template

```markdown
## Summary
<The overall goal, who it's for, and why it's split into several issues.>

## Spec
<High-level behaviour of the finished feature. Details live in the child issues.>

## Out of scope
- <...>

## Acceptance criteria (whole feature)
- [ ] <End-to-end behaviours that are only true once all children are merged.>

## Epic branch
- Branch: `epic/<epic>-<slug>` — created from `<default-branch>`
- Every child issue branches from this epic branch and opens its PR into it.
- `<default-branch>` is not touched until the final merge below.

## Child issues (build order)
| Issue | What it delivers | Depends on | Can run in parallel with |
|---|---|---|---|
| #<a> | <...> | — | — |
| #<b> | <...> | #<a> | #<c> |
| #<c> | <...> | #<a> | #<b> |

- [ ] #<a> <title>
- [ ] #<b> <title>
- [ ] #<c> <title>

## Keeping the epic branch up to date
If `<default-branch>` gets other changes while this epic is in progress, merge it into the epic branch every few days so the final merge stays small:
```
git fetch origin
git switch epic/<epic>-<slug>
git merge origin/<default-branch>
git push
```

## Finishing the epic
- [ ] All child PRs merged into `epic/<epic>-<slug>`
- [ ] Whole-feature acceptance criteria above pass on the epic branch
- [ ] Open the final PR: `epic/<epic>-<slug>` → `<default-branch>`, with this description line: `Closes #<a>, #<b>, #<c>, #<epic>`
- [ ] After it merges, delete the branch `epic/<epic>-<slug>`

## Risks & open questions
- <...>
```

## Writing rules

**Acceptance criteria**
- Describe *what* is observable, not *how* it is built. "Clicking Export downloads a CSV with one row per visible order" — not "add an `exportCsv()` function".
- One behaviour per line, specific enough that two people would agree whether it passes. Replace words like "fast", "user-friendly", "properly" with something measurable or remove them.
- Include the unhappy paths that matter: errors, empty data, missing permissions, invalid input.
- Use Given/When/Then only where a checklist line would be ambiguous (multi-step flows, state-dependent behaviour, tricky edge cases). Most criteria should stay as plain checklist items.
- Aim for roughly 4-10 criteria. More than ~12 is usually a sign the issue should be split.

**Implementation plan (for AI agents)**
- Order steps so each one can be finished, built, and tested before the next — e.g. data model → logic → API → UI → docs. An agent works best with small verified steps rather than one big change.
- Name real files and real commands when you have seen them in the repo. When you haven't, still suggest the likely location but add `(verify)` so the agent checks before editing — an agent will trust a confident wrong path.
- Each step gets a concrete *Check*: a test command, a build, or a specific thing to observe.
- Tell the agent which existing code to copy patterns from; it keeps the change consistent with the codebase.
- Keep steps to what the issue needs. Don't add refactors, extra features, or "nice to have" steps — put those under Out of scope or Risks.

**Branch & worktree section**
- Always present, in every single and child issue. Say plainly where the PR goes; for child issues add "not into `<default-branch>`", because opening the PR against the default branch is the most common mistake.
- If the issue depends on another, say so in the first line so an agent doesn't start too early.

**General**
- Don't invent facts about the repo, the users, or the business. If something is unknown, say so under *Risks & open questions*.
- Keep the issue self-contained. Summarise anything important from the conversation instead of writing "as discussed".
- Delete template sections that truly don't apply rather than filling them with "N/A" — except Out of scope, Acceptance criteria, Branch & worktree, Test plan, and Risks & open questions, which are always present (write "None known" in Risks if that is true).

## Example

**Input idea:** "users should be able to export their orders to excel"

After a quick repo look (React frontend in `web/`, Express API in `api/`, orders page at `web/src/pages/Orders.tsx`), good questions would be: *Which orders — all, or only the current filtered view? CSV or real .xlsx? Who can export — every user or admins only?* Suppose the answers are: current filtered view, CSV is fine, any logged-in user.

This fits in one issue, so no epic. **Title:** `Add CSV export of the filtered orders list`

**Sample acceptance criteria:**
- [ ] The Orders page shows an **Export CSV** button to any logged-in user
- [ ] Clicking it downloads `orders-YYYY-MM-DD.csv` containing exactly the orders matching the current filters
- [ ] The CSV has a header row and the columns: Order ID, Date, Customer, Status, Total
- [ ] With no matching orders, the button is disabled and shows a tooltip "No orders to export"
- [ ] Exporting 10,000 orders completes without the page freezing

**Sample out of scope:** Excel (.xlsx) format; scheduled or emailed exports; exporting order line items.

**Sample Branch & worktree (after the issue was created as #57):**
- Depends on: nothing — can start now
- Start from: `main`
- Your branch: `57-orders-csv-export`
- Set up: `git fetch origin && git worktree add ../wt-57 -b 57-orders-csv-export origin/main && cd ../wt-57`
- Open your pull request into: `main`
- PR description: `Closes #57`
- After merge: `git worktree remove ../wt-57` and delete the branch