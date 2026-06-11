# Agent Skills

Open-source [Agent Skills](https://skills.sh) for Claude Code and other coding agents.

## Install

```bash
npx skills add vineet-codes256/agent-skills
```

Or install a single skill:

```bash
npx skills add vineet-codes256/agent-skills --skill compose-and-push-commits
```

## Skills

### compose-and-push-commits

Turns a messy working tree into clean, atomic, reviewable git history — then pushes it.

- Surveys the full diff (staged, unstaged, untracked)
- Groups changes into logical units (deps, config, tests, migrations, features kept separate)
- Proposes a commit plan and waits for your approval before touching anything
- Commits each group with conventional, imperative subject lines
- Pushes to the remote only after you type `CONFIRM_REMOTE_CHANGE` — never force-pushes, never `git add -A`

Trigger with: *"compose and push commits"*, *"clean up commits"*, *"split commits"*, *"commit and push my changes"*.

## License

[MIT](LICENSE)
