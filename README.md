# skills

My personal Claude Code skills. Skills are grouped by category, and each skill is a folder with its own `SKILL.md`, such as `git/commit-format/`.

## Setup

Clone this repo to `~/skills`, then link every skill into Claude Code:

    find ~/skills -name SKILL.md -exec dirname {} \; | while read -r skill; do ln -sfn "$skill" ~/.claude/skills/; done

Run the same command after adding a new skill. To update, run `git pull`.
