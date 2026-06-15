# android-emarsys-sdk (Action fork)

Fork of [emartech/android-emarsys-sdk](https://github.com/emartech/android-emarsys-sdk) maintained by the Action mobile team.

## Why this fork exists

The official Emarsys SDK has an unhandled `UnrecoverableKeyException` crash in `SharedPreferenceCrypto.getOrCreateSecretKey()`. After a phone migration or backup/restore, the Android Keystore alias can exist but the key material becomes unrecoverable — `containsAlias()` returns `true` but `getKey()` throws, crashing the app in `Application.onCreate` before any screen renders.

The fix has been submitted upstream at [emartech/android-emarsys-sdk#63](https://github.com/emartech/android-emarsys-sdk/pull/63) but has not been merged yet. Once it is merged and released, this fork can be dropped in favour of the official SDK.

## What's different from the official SDK

This fork is based on the official `3.11.0` tag with 4 extra commits on top:

| Commit | Author | Purpose |
|--------|--------|---------|
| `UnrecoverableKeyException` fix | [Steven Roebert](https://github.com/sroebert) | Catches the exception, deletes the corrupted keystore entry, and creates a fresh key so the app recovers gracefully |
| Disable Required Signing | [Julian Falcionelli](https://github.com/julianfalcionelli) | Disables GPG signing requirement so JitPack can build without a signing key |
| Disable signing for subprojects | [Julian Falcionelli](https://github.com/julianfalcionelli) | Disables signing tasks on all submodules for JitPack compatibility |
| JitPack compat + static version | Action mobile team (adapted from [Julian Falcionelli](https://github.com/julianfalcionelli)) | Replaces Grgit dynamic versioning with a static version so JitPack can build without a full git history |

## Usage in action-app-android

In `gradle/libs.versions.toml`:

```toml
emarsys-fix = "<commit-hash>"

emarsys = { group = "com.github.actionbtdm", name = "android-emarsys-sdk", version.ref = "emarsys-fix" }
emarsys-firebase = { group = "com.github.actionbtdm.android-emarsys-sdk", name = "emarsys-firebase", version.ref = "emarsys-fix" }
```

In `settings.gradle`, make sure JitPack is in the repositories block:

```groovy
maven { url = "https://jitpack.io" }
```

## Updating to a new official version

When a new official Emarsys SDK version is released:

1. Fetch the new tag from upstream:
   ```bash
   git fetch upstream --tags
   ```
2. Create a new branch from the new official tag:
   ```bash
   git checkout -b chore/update-to-<new-version> <new-version-tag>
   ```
3. Cherry-pick the Action team's extra commits from `master` on top:
   ```bash
   git cherry-pick <fix-commit> <jitpack-commit> <signing-commit-1> <signing-commit-2>
   ```
4. Resolve any conflicts, keeping the Action team's extra commits intact
5. Push and open a PR into `master`
6. Update the commit hash in `libs.versions.toml` on `action-app-android`

## JitPack

This fork is distributed via [JitPack](https://jitpack.io/#actionbtdm/android-emarsys-sdk).

### Pre-compiling an artifact

JitPack builds artifacts on demand the first time they are requested, which can take a few minutes. To pre-compile an artifact before updating the app:

**Option 1 — via the JitPack website:**
1. Go to [jitpack.io/#actionbtdm/android-emarsys-sdk](https://jitpack.io/#actionbtdm/android-emarsys-sdk)
2. Find the commit, branch, or tag you want
3. Click **"Get it"** — JitPack will start building immediately

**Option 2 — via direct URL:**

Open the following URL in your browser (replacing the hash with your commit):

```
https://jitpack.io/com/github/actionbtdm/android-emarsys-sdk/<commit-hash>
```

The build log will show progress. Once complete, Gradle sync in the app will resolve instantly.
