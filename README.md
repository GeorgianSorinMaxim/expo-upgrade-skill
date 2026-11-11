# expo-upgrade-skill

A Claude Code skill and a playbook. They take an Expo app forward by one SDK version.

## Files

- `SKILL.md` — the skill. Short rules and the procedure.
- `playbook.md` — the long form. Preflight, order of operations, verification, rollback, hand-off report.

## Install

Project skill:

```sh
mkdir -p .claude/skills/expo-upgrade-skill
cp SKILL.md playbook.md .claude/skills/expo-upgrade-skill/
```

Personal skill: copy the same files to `~/.claude/skills/expo-upgrade-skill/`.

## Use

Ask Claude Code: "Upgrade this app to the next Expo SDK." The skill loads when the task matches.
You can also follow `playbook.md` by hand.

## Notes

- The skill does not store API details for any SDK. It tells the agent to read the release notes for the target SDK.
- Edit the "Risky areas" section in `playbook.md` for your app.
- Add your own smoke-test list in section 4.
