# github-issue-builder

An agent skill for **Claude Code** and **Codex**. It turns a rough idea (one line, a voice-note ramble, a screenshot, a half-written ticket) into a GitHub issue that an AI coding agent can pick up and finish without coming back with questions. That includes exactly which branch and folder (worktree) to work in, and where its pull request goes.

## Install

```bash
npx github-issue-builder install            # both tools, user scope (~/.claude/skills and ~/.agents/skills)
npx github-issue-builder install --only claude
npx github-issue-builder install --scope project --project ./my-repo
npx github-issue-builder verify             # check the installed copies match the package
npx github-issue-builder uninstall          # removes the skill, keeping a backup
```

Add `--dry-run` to see what would change.

**Updating.** Run `npx github-issue-builder@latest install` again. The installer:
- backs up any existing copy to `~/.github-issue-builder-backups/` and replaces it, or says "already up to date";
- keeps a link between the two install folders (for example `~/.claude/skills/github-issue-builder` pointing at `~/.agents/skills/github-issue-builder`) and updates the folder it points to;
- removes old copies from `~/.codex/skills` (and from `$CODEX_HOME/skills`), so Codex doesn't load the skill twice. Pass `--keep-legacy` to leave them;
- warns if your **claude.ai account** also has this skill (synced into `~/.claude/skills/synced/`). Only claude.ai can update that copy, so replace or delete it in claude.ai → Settings → Capabilities → Skills.

`verify` fails while an outdated duplicate is still around. To file issues directly, the skill uses an authenticated [`gh`](https://cli.github.com/) CLI or a GitHub connector. Without one, it hands you the drafts to copy and paste.

## Usage

Ask your agent something like "make an issue for …", "write a ticket: …", or paste any raw idea meant for GitHub. The skill:

1. **Captures the idea**: who has what problem, what change fixes it, and its type (feature, bug, improvement, chore, docs).
2. **Looks at the repo**: README, agent docs, build and test commands, the default branch, duplicate issues and existing labels.
3. **Asks up to 4 clarifying questions, in one round**, and only when the answer changes the issue. One of them is the question behind most disappointing features: *is this a one-off, or will there be more of them?*
4. **Checks the size**: one issue, or an epic plus child issues with a dependency graph and a parallel build order.
5. **Writes the draft**: summary, spec, out of scope, testable acceptance criteria (Given/When/Then only where needed), branch and worktree setup, a step-by-step plan with a check after each step, a test plan, and risks and open questions.
6. **Delivers it**: files the issue with `gh` after you confirm, creates the epic branch for epics, and replaces the placeholders with real issue numbers.

Branch conventions: single issues branch from the default branch. For an epic, the skill creates `epic/<n>-<slug>`, and each child branches from it in its own worktree (`../wt-<n>`). Child PRs merge into the epic branch, and the final epic PR closes every child issue together.

## Tested

The "one-off or pattern?" question came from real client feedback: a single "lifetime package" feature that should have been a reusable package system. It was kept only after a blind A/B test. Six raw ideas, including a bug and a chore as controls, were run twice through each version and judged without the judges knowing which version wrote which draft. The new version won 9 of 12 pairs.

## License

MIT
