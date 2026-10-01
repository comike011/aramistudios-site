<!-- Canonical source: ~/Sites/app-factory/factory/.claude/fragments/. bootstrap.sh
     COPIES this file into each app repo's .claude/fragments/, where the app's
     CLAUDE.md imports it. Edit the factory copy, never an app's copy: an app's copy
     is overwritten on the next bootstrap, and `bootstrap.sh --check` reports drift. -->

## Workflow transitions

Work in App Factory runs in three stages, **one stage per session**:

> `/af-plan <issue>` → `/af-execute <issue>` → `/af-settle <pr>`

An *issue* is a GitHub issue number in the **current app's repo** (`comike011/<app>`).
Numbers repeat across apps, so an issue number means nothing without the repo it
belongs to. Every command resolves the repo from its cwd, and every session name
carries the app (`<app>-<n>-PLAN`).

Issue state is kept by GitHub, not by hand. Look an issue up with
`gh issue view <n> --json title,body,labels,comments`: that is one call, with no subagent and
no MCP. The PR body says `Closes #<n>`, so the merge closes the issue. There is no
status board to move and no transition to run.

**On work started: pick the ceremony tier before anything else.** The tier decides
whether the spec and plan triggers below fire at all. Ask four questions and state the
answer in one line ("Tier 0: the issue specifies the change verbatim, no persisted-data
change"). Any **yes** means full ceremony (Tier 2).

1. Is there a design decision with **more than one defensible answer**?
2. Does it change a **contract**: a backend API the app calls, an App Store
   entitlement, capability, privacy manifest or permission prompt, or a
   deep-link / URL scheme?
3. Does it change **persisted data**: an on-device schema (SwiftData, Core Data,
   SQLite, AsyncStorage), a Keychain item's shape, or anything an installed build
   must migrate?
4. Will someone need the **reasoning** in three months, not just the diff?

If all four are no, write no spec and no plan doc. One more question picks the lane:

- **Are the steps coupled enough that doing them out of order breaks something?**
  - No → **Tier 0, just go**: worktree, change, tests, PR.
  - Yes → **Tier 1**: an inline task list, made durable in the execute brief.

Rules that keep this honest:

- **Q1 is bound to something observable.** If you had to ask a clarifying question
  about *approach*, Q1 is **yes**, because needing to ask means there was more than
  one defensible answer.
- **Size is not a criterion.** Line and file counts are unknowable when you must
  decide.
- **Promote upward mid-flight, never demote.** If you discover a real decision while
  implementing, stop and write the spec.
- **The tier line goes in the PR body.** For Tier 0 and Tier 1 work, the PR is the
  design record.
- **Tiering never skips the worktree, the local review, or the security review.** It
  decides only whether a spec and plan are written.

**On plan requested (Tier 1 or 2).** Before drafting a Tier 2 spec, read
`~/Sites/app-factory/factory/specs/<app>/` for prior designs on the same surface. A
Tier 1 task list stays in the session until the brief makes it durable.

**On spec written (`factory/specs/<app>/YYYY-MM-DD-<slug>.md`).** The spec is reviewed
**locally, before it is committed**, and that review is its only feedback point.

1. **Ask which review path. This is blocking, and do not choose for the user.** Use
   `AskUserQuestion` with two options:
   - **Walkthrough**: present the spec section by section, and for each one apply
     `~/Sites/app-factory/factory/rubrics/spec-review-rubric.md`, state your findings,
     and take the user's call before moving on.
   - **Auto-apply**: run the rubric without the walkthrough, apply its findings to the
     doc, and report what changed as one short list.
   - If the originating request already said which way ("just draft the spec", "skip
     the review", "auto-apply"), that **is** the answer. Honor it and do not ask.
2. **Finish that path before committing.** Whatever the review surfaces is applied
   first. There is no second pass later.
3. **Commit the spec straight to `factory` `main` and push.** Do not open a PR. The
   commit is the audit record. If the review was skipped, the commit message says so
   in one line. `factory/CLAUDE.md` explains why `specs/` is exempt from the worktree
   rule.
4. **Then resume where the spec was requested.** In a plan session, continue
   `/af-plan` at approval and handoff. In an execute session (a mid-flight
   promotion), continue implementing in the same session.

**On plan approved.** Choose inline or subagent-driven execution yourself and do not
ask. State which you chose in one line. Use **inline** (`superpowers:executing-plans`) when
the steps are coupled or fit in one context, and **subagent-driven**
(`superpowers:subagent-driven-development`) when the plan splits into independently
verifiable tasks. When it is borderline, go inline. Either way the work lands on one
branch in one worktree.

**On plan session handed off.** The plan session **ends** when the worktree exists, has
been **released**, and `execute.md` is written. Implementation belongs to a new
background session, which this session starts itself.

1. **Release the worktree with `ExitWorktree({action: "keep"})`** as soon as it is
   provisioned, *before* writing the brief. `EnterWorktree` writes a `locked` file
   naming this session's pid, and a locked worktree makes this session impossible to
   delete cleanly. Deleting the session anyway is how worktrees go missing from under
   later stages. `keep` drops the lock and leaves the worktree on disk. Never use
   `remove`, which deletes the branch and does not protect git-excluded files.
2. **Write `execute.md` under `<repo>/.claude/briefs/<worktree-name>/`**, in the
   **main checkout**, never inside the worktree. `<worktree-name>` is the worktree's
   directory basename (`af-<n>+<slug>`). This is the template:

   ```markdown
   # Execute brief — <app>#<n>

   ## Where you are
   Repo root: /Users/<you>/Sites/app-factory/<app>
   Worktree:  /Users/<you>/Sites/app-factory/<app>/.claude/worktrees/af-<n>+<slug>
   Branch:    af-<n>/<slug>
   Base:      main @ <sha>

   ## Launch — the line that starts this stage
   ~/Sites/app-factory/factory/scripts/af_launch.sh execute <n>

   ## Model profile — which model this stage runs on, and why
   <default | profile name> — <the rubric rule that fired>

   ## Prerequisites — what must be true on this machine before a build
   - <Xcode version, simulator runtime, Node/Ruby version, running services, secrets>

   ## What to build
   Tier: <0|1|2>
   Spec: ~/Sites/app-factory/factory/specs/<app>/<file>.md
   Task list (Tier 1): <the inline task list>

   ## Scope fence — implementation may not touch anything outside this list
   - <explicit file list>

   ## Deliberate decisions — choices that look wrong and are not
   - <decision> — chosen because <reason>. Do NOT re-litigate.

   ## Review path
   - <auto | skip-everything | unset — unset means execute asks>
   ## Security review
   - <run | skip | unset — unset means execute runs the gate>

   ## Done means
   <exit condition, including the app's build and test commands from its CLAUDE.md>
   ```

   **Deliberate decisions** is the field everything depends on. It holds the choices a
   cold execute session would otherwise re-litigate and "fix". It matters more when
   the next stage runs on a cheaper model, which lacks both the conversation and the
   judgement to tell a deliberate constraint from a mistake. Never leave it empty.
   **Prerequisites** exists because iOS builds fail on machine state (a missing
   simulator runtime, the wrong Xcode selected, an unset signing team) that a cheap
   execute session would misdiagnose as a code bug. For Tier 0 and Tier 1 work, omit
   the Spec line rather than point at a doc that does not exist.
3. **Exclude the briefs directory through the common git dir,** once per repo:

   ```sh
   CD=$(git rev-parse --git-common-dir)
   git check-ignore -q .claude/briefs/ || echo '/.claude/briefs/' >> "$CD/info/exclude"
   git check-ignore -v .claude/briefs/
   ```

   A non-zero exit on the verify step means the exclude did not take, so stop. Never
   use the worktree's own `.git/worktrees/<name>/info/exclude`, which git never reads,
   and never the app's tracked `.gitignore`.
4. **Do not implement here.**
5. **Start the execute stage, then stop,** with the sandbox disabled (inside it,
   `claude --bg` cannot write `~/.claude/jobs`):

   ```sh
   ~/Sites/app-factory/factory/scripts/af_launch.sh execute <n>
   ```

   Report the worktree, the brief, and the `<id> <name>` line it prints. If it exits
   non-zero, nothing started. Relay its output verbatim and stop. Never implement
   inline as a fallback, and do not use `/clear` instead: this session stands in the
   main checkout, and only a new process starts in the worktree.

**On implementation complete (all tasks done and the tests pass).**

- Summarize in one short paragraph: the branch, the commit count, the files touched,
  and the result of the app's build and test commands.
- If a test is failing or the build was not run, do **not** proceed
  (`superpowers:verification-before-completion`).
- **Ask which review path. This is blocking**, unless the brief pre-set it or the request
  already said:
  - **Auto-apply**: run `/code-review` on the branch diff, apply the clear blocking
    and important findings, and note them in the PR body.
  - **Skip everything**: skip both the code review and the security review.
- **Security review before push. This gate is blocking.** Run `/security-review` on the
  branch diff. A clean result advances with no report. A High or Medium finding is a
  hard stop: fix it or triage it as a false positive in one line, then re-run. A Low
  finding goes in the PR body. `Security review: skip` in the brief skips the gate,
  and the PR body notes the skip.
- Then proceed with no confirmation: push the branch, then open a **regular, non-draft**
  PR with `gh pr create --assignee @me`. The body carries the tier line,
  `Closes #<n>`, and any review or security notes.

**On PR opened.** The open PR **ends the implementing session**.

1. **Write `settle.md`** beside `execute.md`, resolving the directory from inside the
   worktree as `dirname "$(git rev-parse --git-common-dir)"` plus
   `basename "$(git rev-parse --show-toplevel)"`. The template:

   ```markdown
   # Settle brief — <app>#<n>

   ## Where you are
   Repo root: /Users/<you>/Sites/app-factory/<app>
   Worktree:  /Users/<you>/Sites/app-factory/<app>/.claude/worktrees/af-<n>+<slug>
   Branch:    af-<n>/<slug>
   PR:        #<p>  https://github.com/comike011/<app>/pull/<p>

   ## Launch
   ~/Sites/app-factory/factory/scripts/af_launch.sh settle <n> --pr <p>

   ## Model profile
   <same as execute> — <why>

   ## Escalations — blocking findings this stage declined to judge
   <appended by /af-settle; empty at write time. Never overwrite an entry.>

   ## Scope fence — a fix may not touch anything outside this list
   - <explicit file list>

   ## Deliberate divergences — constraints that look wrong and are not
   - <constraint> — chosen because <reason>. Do NOT "fix" this; rebut it.

   ## Settled means
   CI green (or no checks exist), every bot that reviews this repo has reported for
   the current round, and every blocking finding is answered. Nits do not earn a round.
   ```

2. **Verify the exclude** with `git check-ignore -v .claude/briefs/`. Do not append it
   here. A non-zero exit means the plan stage's exclude never took, so stop.
3. **Do not settle here.**
4. **Start the settle stage, then stop** (sandbox disabled):
   `~/Sites/app-factory/factory/scripts/af_launch.sh settle <n> --pr <p>`. Report the
   PR URL and the `<id> <name>` line. On a non-zero exit, relay the output verbatim and
   stop. For untracked (`no-issue/…`) work there is no issue number to launch with, so
   print `cd <worktree> && claude --add-dir <repo root> --permission-mode auto "/af-settle <p>"`
   for the user and stop.

**On settle session started (`/af-settle <pr>`).** This session inherits none of the
implementing conversation, so read the settle brief before judging anything. The
brief's key is the worktree directory's own name:

```sh
[ -e "$(git rev-parse --git-dir)/commondir" ] || { echo 'main checkout — stop'; exit 1; }
REPO=$(dirname "$(git rev-parse --git-common-dir)")
KEY=$(basename "$(git rev-parse --show-toplevel)")
```

A worktree sits three levels below the repo root, so never count `..` to reach the root.
Once the brief is read, clear the earlier stages (sandbox disabled):
`~/Sites/app-factory/factory/scripts/af_launch.sh reap <n> PLAN EXEC SETTLE`. A failure
there is one line of output, not a stop.

- **The reviewer roster is derived, never assumed.** Codex reviews a repo only if
  its GitHub app is installed on it. The Claude action reviews only if
  `.github/workflows/` holds a Claude review workflow. `pr_findings.sh` reports each
  bot as `reported`, `absent` or `errored`. A repo with no reviewer installed has an
  empty roster, and `absent` there is the correct permanent answer.
- **No App Factory repo has a reviewer installed at this time.** The roster is therefore
  empty everywhere: `absent` is terminal, the session posts no `@codex review` and waits
  no ~6 minutes. Install a reviewer on a repo, then delete this bullet.
- **Watch CI to completion:** `gh pr checks <pr> --watch`. Fix a failing check at most
  **3 times**, then write `needs input:`. A repo with no checks returns instantly, and
  that counts as green, not reviewed.
- **Read review state only through**
  `~/Sites/app-factory/factory/scripts/pr_findings.sh <pr>`. Never hand-roll `gh`
  queries: the API has three traps (comments re-anchor to head, an empty bot review
  looks like a clean pass, and `gh --jq` ignores `--arg`). If the script fails, write
  `needs input:`. Allow up to **~6 minutes** for a first report *when a reviewer is in the
  roster*; with an empty roster, read once and proceed. A rostered Codex that has not
  appeared gets one `@codex review` and one more wait, then is noted as absent.
- **Tier every finding where `answered_by_me` is false.** A finding is a **nit** if and
  only if acting on it would change **none** of: runtime behavior, a contract, persisted
  data, or a test outcome. Everything else is **blocking**. For an iOS app, blocking
  also includes main-thread violations, retain cycles, unhandled permission-denied
  paths, and anything that would fail App Review.
- **Nits never earn a round.** Answer them in one batched reply with a disposition
  each. Never push for nits alone.
- **Escalate before you reply.** A blocking finding the divergences list does not
  cover is not a cheaper stage's call. Append it to the brief's `## Escalations`,
  stop, and relaunch with `af_launch.sh settle <n> --pr <p> --opus`. A stage already
  on the strong model judges the finding itself.
- **A blocking round** means one commit fixing every blocking finding (or a reasoned
  rebuttal), then `pr_reply.sh <pr> --from replies.json`, then a re-trigger that
  captures the round boundary in the same command:
  `ROUND=$(date -u +%Y-%m-%dT%H:%M:%SZ) && gh pr comment <pr> --body '@codex review' && echo "boundary: $ROUND"`.
  Every later `pr_findings.sh` call in that round passes `--since "$ROUND"`. Judge Codex
  on `codex.status` and the Claude action on `claude.status_at_head`. With an empty roster
  no finding ever exists, so this never fires.
- **At two blocking rounds, report and ask whether to continue.** The cap is a
  reporting point, not a stop.
- **A clean pass is terminal. Never reply to one.** An errored review is not a clean
  pass.
- **Never touch human reviewer comments.** Summarize them for the user.
- **Exit** when CI is green, every rostered bot has reported or been noted absent, and
  no blocking finding is open. Confirm the CI status, the blocking findings fixed
  versus rebutted, the nits declined, and any absent bot.
- **The merge is the user's call.** Nothing here merges an app PR on its own.

**On merge confirmed.** Confirm the issue closed (`gh issue view <n> --json state`) and
close it with a comment if `Closes #<n>` did not take. Remove the worktree with
`git -C <repo> worktree remove <worktree>`, then run `rm -rf <repo>/.claude/briefs/<key>/`.
This is the only place a worktree or a brief is removed. Pull `main`, delete the
merged branch (`-D` after a squash merge), and confirm all of it in one short message.

**Context hygiene.** The plan-to-execute and execute-to-settle handoffs are new
background sessions, so there is nothing to clear at those boundaries. At the end of
a settle session, or after the merge cleanup, recommend `/clear` before the next
issue. `/clear` and `/compact` are CLI built-ins that no tool can invoke, so recommend
them and never claim to have run one. Send high-volume reads (wide searches, build
logs, crash logs) to a subagent rather than the main thread.

**Right-repo guard.** Before implementing, confirm that the issue belongs to the repo
you are standing in. If it does not, stop and name the right repo.
