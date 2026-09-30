<!-- Canonical source: ~/Sites/app-factory/factory/.claude/fragments/. Copied into
     each app repo by bootstrap.sh. Edit the factory copy only. -->

## Issue requirements

- All work belongs to a GitHub issue in the app's own repo (`comike011/<app>`). If
  none exists, create one with `gh issue create` and confirm the number with the user
  before starting.
- An issue spans **exactly one app repo**. Work that touches two apps, or an app and
  the factory, becomes one issue per repo, each linking the others.

## Worktree requirements

**Every change goes in a worktree. There is no size exemption.**

**The main checkout stays on the default branch, always.** The rule is about the
*checkout*, not the branch name: creating an `af-<n>/…` branch inside
`~/Sites/app-factory/<app>` and working there is just as forbidden as committing to
`main`.

- **Branch naming:** `af-<n>/<slug>`, or `no-issue/<slug>` for untracked work.
- **Claude Code sessions:** create the worktree with the native `EnterWorktree` tool
  (through `superpowers:using-git-worktrees`). It lands inside the sandbox write
  root, so builds and tests run without disabling the sandbox.
  - It does not produce the names you asked for. `EnterWorktree({name: "af-12/slug"})`
    creates `<repo>/.claude/worktrees/af-12+slug` on branch `worktree-af-12+slug`, so
    **rename the branch immediately**: `git branch -m af-12/slug`. From inside a
    worktree, that command writes the common `.git/config`, which the sandbox denies,
    so run it with the sandbox disabled.
  - **Then release it with `ExitWorktree({action: "keep"})`**, never `remove`.
- **Copy anything listed in `.worktreeinclude`** (`.env`, signing config,
  `GoogleService-Info.plist`, …). `EnterWorktree` does not do this for you.
- **iOS specifics.** A worktree is a second copy of the Xcode project, so:
  - Give each worktree its own DerivedData path (`-derivedDataPath build/` in the
    worktree) so two checkouts never share build products.
  - Run `pod install`, `npm ci` or the SwiftPM resolve step in the new worktree
    before the first build. Dependencies are not shared across checkouts.
  - Never change signing, bundle identifiers or entitlements as a side effect of a
    build fix. Those changes are contract changes (tier question 2).

**If you are already on a branch in the main checkout,** stop. Commit any work to that
branch first, run `git checkout main`, then
`git worktree add .claude/worktrees/<name> <that-branch>`. Do not use `git branch -m`
to recover: with one argument, it renames whichever branch is checked out.
