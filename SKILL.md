---
name: expo-upgrade-skill
description: Use when an Expo app must move forward one SDK version, when `npx expo install --fix` or `expo-doctor` reports version mismatches after a bump, or when a build fails after an SDK upgrade.
---

# Expo SDK Upgrade (one step)

## Rule

Move one SDK version at a time. Never skip a version.
To go from SDK N to N+2, finish N+1 first. Merge it or tag it. Then start N+2.

Do not skip a step in this file because the change looks small.

## Inputs

Ask the user for these items. Use the defaults if they do not answer.

| Item            | Default                                                                 |
| --------------- | ----------------------------------------------------------------------- |
| Target SDK      | Current SDK + 1                                                         |
| Package manager | The one that owns the lockfile                                          |
| Native folders  | Check if `ios/` and `android/` are in git                               |
| Test command    | `package.json` `test` script                                            |
| Build check     | User runs device builds. Do not claim a build passes unless you ran it. |

## Procedure

Use the full detail in `playbook.md`. Short form:

1. **Preflight.** Clean git tree. Green tests on the current SDK. Record the baseline.
2. **Branch.** One branch, named for the target SDK.
3. **Read the changelog.** Read the Expo SDK release notes and the React Native notes for the target version. Write a list of breaking changes that touch this app.
4. **Bump Expo.** `npx expo install expo@^<target>`.
5. **Fix dependencies.** `npx expo install --fix`, then `npx expo-doctor`.
6. **Fix config.** Update `app.json` / `app.config.*`, config plugins, `babel.config.js`, `metro.config.js`, `tsconfig.json`.
7. **Regenerate native code.** If the project uses Continuous Native Generation, run `npx expo prebuild --clean`. Never edit generated folders by hand.
8. **Fix code.** Work through the breaking-change list. One commit per topic.
9. **Verify.** Run the checks in the Verify section.
10. **Hand off.** Report what passed, what the user must test on a device, and the rollback command.

## Verify

Run each check. Record the result.

- Type check. Lint. Unit tests. Compare with the baseline.
- `npx expo-doctor` shows no new failures.
- `npx expo export` (or the project bundle command) completes for each platform.
- Dev build starts. App opens. No red screen.
- Check each native module the app uses. Open one screen that uses it.
- Release build: run it if you can. If you cannot, say so plainly.

## Rollback

Before step 4, record the current commit: `git rev-parse HEAD`.

- Not merged: `git switch <base-branch>` and delete the upgrade branch.
- Merged and not released: revert the merge commit.
- Released: ship a fix-forward or revert, then release again. Do not edit native folders by hand to "patch" the problem.

After any rollback: reinstall with the lockfile, clear caches (`npx expo start --clear`), and run `prebuild --clean` if the project uses it.

## Common mistakes

| Mistake                                          | Fix                                                        |
| ------------------------------------------------ | ---------------------------------------------------------- |
| Edit `package.json` versions by hand             | Use `npx expo install`. It picks versions for the SDK.     |
| Skip an SDK version                              | Do one version at a time.                                  |
| Fix generated `ios/` or `android/` files         | Fix the config plugin or `app.json`. Then re-run prebuild. |
| Trust old API names from memory                  | Read the changelog for the target SDK.                     |
| Say "build passes" with no build run             | Say what you ran and what you did not run.                 |
| Upgrade other major libraries in the same branch | Keep them in a separate branch.                            |
