# Expo SDK Upgrade Playbook

This playbook gives the long form of `SKILL.md`. A person or an agent can follow it.
It applies to any Expo app. Replace `<N>` with the current SDK and `<N+1>` with the target.

## 0. Scope

- One SDK version per upgrade.
- No feature work in the upgrade branch.
- No unrelated major library upgrades in the upgrade branch.

## 1. Preflight

Do each check before you change anything. Stop if a check fails.

| #   | Check               | Command                                                       | Pass condition                                       |
| --- | ------------------- | ------------------------------------------------------------- | ---------------------------------------------------- |
| 1   | Clean git tree      | `git status --short`                                          | No output                                            |
| 2   | Current SDK         | `node -p "require('expo/package.json').version"`              | Matches the SDK you think you have                   |
| 3   | Lockfile is in sync | Install with the frozen lockfile flag of your package manager | No change to the lockfile                            |
| 4   | Doctor baseline     | `npx expo-doctor`                                             | Save the output. Note old failures.                  |
| 5   | Type check baseline | Project command, for example `npx tsc --noEmit`               | Save the error count                                 |
| 6   | Lint baseline       | Project command                                               | Save the warning and error count                     |
| 7   | Test baseline       | Project command                                               | All tests pass. Save the count.                      |
| 8   | Bundle baseline     | `npx expo export --platform all`                              | Completes                                            |
| 9   | Native folders      | `git ls-files ios android \| head`                            | Know if they are committed or generated              |
| 10  | EAS / CI config     | Read `eas.json` and CI files                                  | Note pinned versions (Node, Xcode, JDK, image names) |

Save all baselines in a scratch file. You will compare them at the end.

Record the rollback point:

```sh
git rev-parse HEAD
```

## 2. Read before you edit

1. Open the Expo SDK `<N+1>` release notes.
2. Open the React Native release notes for the React Native version that SDK `<N+1>` uses.
3. Write a list called "breaking changes that touch this app". For each item, write:
   - the change,
   - the files in this app that it touches (`grep` for the API),
   - the planned fix.
4. Check each third-party library that has native code. Look for a release that supports SDK `<N+1>`.
   If none exists, decide now: wait, replace, or patch.

Do not use API names from memory. Names change between SDKs.

## 3. Order of operations

Follow this order. Each stage ends with a commit that the baselines can check.

### 3.1 Branch

```sh
git switch -c chore/expo-sdk-<N+1>
```

### 3.2 Bump Expo

```sh
npx expo install expo@^<N+1>.0.0
```

### 3.3 Align dependencies

```sh
npx expo install --fix
npx expo-doctor
```

- Read every line that `--fix` changes.
- If a package has no compatible version, handle it now (see section 2, step 4).
- Do not hand-edit versions to silence a warning.

### 3.4 Align config

Check each file. Change only what the release notes require.

- `app.json` or `app.config.*`: removed or renamed keys, plugin options.
- Config plugins: version support, option names.
- `babel.config.js`: preset name, plugin order.
- `metro.config.js`: base config import, resolver options.
- `tsconfig.json`: the Expo base config it extends.
- `eas.json` and CI: Node version, Xcode version, build image.

### 3.5 Regenerate native code

- If `ios/` and `android/` are generated: `npx expo prebuild --clean`. Do not commit the output.
- If they are committed: run `npx expo prebuild --no-install` in a temp copy. Diff it against your folders. Apply the needed changes by hand. Then run a full native build.

Never fix a problem in generated output. Fix the plugin, the config, or the template. Then regenerate.

### 3.6 Fix application code

Work through the list from section 2. Commit one topic at a time.
Typical topics: renamed APIs, removed modules, new React or React Native behavior, changed defaults, navigation library changes.

### 3.7 Clean up

- Remove polyfills or patches that the new SDK makes unneeded.
- Remove `patch-package` patches for versions that no longer exist. Check each patch still applies.
- Delete the old lockfile entries only through the package manager.

## 4. Verify

Compare with the baselines from section 1.

| Check                            | Pass condition                                                                  |
| -------------------------------- | ------------------------------------------------------------------------------- |
| `npx expo-doctor`                | No new failures                                                                 |
| Type check                       | No new errors                                                                   |
| Lint                             | No new errors                                                                   |
| Unit tests                       | Same pass count. No test weakened or deleted to pass.                           |
| `npx expo export --platform all` | Completes                                                                       |
| Dev build, iOS                   | App opens. No red screen.                                                       |
| Dev build, Android               | App opens. No red screen.                                                       |
| Native modules                   | One screen per module opens and works                                           |
| Release build                    | Both platforms build. If you did not run this, say so.                          |
| Cold start                       | App starts from a fresh install                                                 |
| Upgrade path                     | App updates from the previous release without data loss (storage, auth session) |

Smoke test list. Fill it in for your app:

- [ ] Launch and first screen
- [ ] Sign in and sign out
- [ ] Main app flow
- [ ] Deep link (if used)
- [ ] Push notification (if used)
- [ ] Camera, location, or other permission prompt (if used)
- [ ] Offline start

## 5. Roll back

Choose by state.

| State                | Action                                                                                  |
| -------------------- | --------------------------------------------------------------------------------------- |
| Branch only          | `git switch <base>`. Delete the branch. Reinstall.                                      |
| One bad commit       | `git revert <sha>`. Run the checks again.                                               |
| Merged, not released | `git revert -m 1 <merge-sha>`. Reinstall. Rebuild.                                      |
| Released             | Decide with the team: fix forward, or revert and release again. Raise the build number. |

After any rollback:

```sh
<package-manager> install          # use the frozen lockfile flag
npx expo start --clear
npx expo prebuild --clean          # only if the project generates native folders
```

Then run the baseline checks. They must match section 1.

## 6. Hand-off report

Use this shape. Keep each line short.

- **Result:** one sentence. What works. What blocks.
- **Ran:** the checks you ran, with pass or fail.
- **Not run:** the checks you did not run, and why.
- **Needs you:** device builds, store checks, risky areas.
- **Changed tests:** each test you changed, and why.
- **Rollback:** the exact commit or command.

## 7. Risky areas

Give extra care to these areas. Read the release notes twice for them.

- Authentication and session storage.
- Consent and privacy SDK setup.
- Push notifications.
- Deep links and universal links.
- Any library with native code.
- New Architecture, Hermes, or JS engine changes.
- Minimum OS versions (iOS deployment target, Android `minSdk`).
