<!-- Canonical source: ~/Sites/app-factory/factory/.claude/fragments/. Copied into
     each app repo by bootstrap.sh. Edit the factory copy only. -->

## What every app repo declares

The harness is stack-neutral. Each app decides its own stack, and the `/af-*` commands
read that decision from the **`## App` section of the app's own `CLAUDE.md`**. That
section must give every field below. A stage that finds one missing stops and asks
for it rather than guessing.

| Field | Example (SwiftUI) | Example (Expo) |
|---|---|---|
| Stack | SwiftUI, iOS 18+, SwiftData | Expo SDK 54, React Native, TypeScript |
| Build | `xcodebuild -scheme App -destination 'platform=iOS Simulator,name=iPhone 16' -derivedDataPath build build` | `npx expo prebuild --platform ios && npx tsc --noEmit` |
| Test | `xcodebuild test -scheme App -destination '…' -derivedDataPath build` | `npx jest` |
| Lint | `swiftlint` | `npx eslint .` |
| Setup | `brew bundle && xcodebuild -resolvePackageDependencies` | `npm ci` |
| Bundle id | `com.comike011.<app>` | `com.comike011.<app>` |

These commands are the definition of "tests pass" in *On implementation complete*.
They are also what an execute brief's **Done means** field names. Build and test logs
are long, so run them through a subagent, or pipe them to `tail`/`grep`, rather
than into the main thread.

## Stack-specific guidance

A stack's conventions belong in the app's own `CLAUDE.md` below the `## App`
section, not in these fragments. When two apps on the same stack want the same rule,
promote it to a factory fragment named for that stack (`swiftui.md`, `expo.md`) and
have both apps import it.
