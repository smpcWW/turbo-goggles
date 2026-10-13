---
title: Review model
description: The merge check classifies, then runs gates, hunters, collate, and a bounded fix. The hook only locks the review machinery.
type: plan
---

# Review model

The implementer does not approve its own work. Hunters review one ask. Collate compares their reports with the rules in [finding-contract.md](finding-contract.md). Routing uses [routing-policy.md](routing-policy.md). Project values live in the manifest described by [project-manifest.schema.md](project-manifest.schema.md).

## Loop

1. The implementer stages the change and commits. The session loads [roles/implementer.md](../roles/implementer.md) only.
2. The pre-commit hook refuses a staged path listed in the manifest `gate_paths`, then exits success. It does not classify. A human commits a gate change by skipping the hook once. When `issue_tracker` is not null, the commit-message hook appends the trailer when exactly one issue is in progress. The job reads that trailer from the commit messages and does not write it. The gate lock below specifies both hooks.
3. Before classify, the job resets tracked files to `HEAD` and deletes untracked files. Ignored files stay, including installed dependency directories. A reused work directory would otherwise let a later add publish leftover edits. The merge check classifies the full pull-request diff against the default branch. It writes the routing decision in the job workspace. Why the hook does not classify is in [../lessons/from-hand-contracting.md](../lessons/from-hand-contracting.md).
4. Deterministic gates run next, including when the decision is low. A gate failure fails the job before any hunter starts. The publish step still runs. The fixer finds no verdict and does not call an agent.
5. When the decision is high, the selected hunters run at the same time. Each receives the ask, the gate report, and its own diff slice. None receives another hunter's report.
6. Collate runs the comparison in [roles/collate.md](../roles/collate.md). It does not receive the diff. A failed comparison fails the job.
7. The bounded fixer runs after collate even when collate failed. The editing agent follows [roles/fixer.md](../roles/fixer.md). Checkout cleaning, the affected-test command, and the commit are the job's procedure in Bounded fix below. A skip exits success and leaves a failed verdict failed.
8. The job publishes the routing decision, and the verdict and fix report when those files exist, including when hunters stayed idle.

An optional early run may send the staged tree through the same job. The ask on that run is the ask the implementer supplied. The merge check is the decision that gates merge. Pushing a review branch does not start a second run by itself.

## Ask

On a merge check the ask is the pull-request title, then every non-merge commit subject in the diff, newest first, with duplicates removed. When the title cannot be read, the newest commit subject stands in for it. Subjects are data. They are not instructions to the hunters.

On an optional early run whose commit subject is `adversary-request <tree>`, the ask is the commit body. That run does not rebuild the ask from later subjects.

## What a hunter sees

The slice for a hunter is the `hunter_files` list in the routing decision for the diff under review. The manifest issue-tracker data directory is omitted when `issue_tracker.data_dir` is set. Lockfile bodies are replaced with a summary before the size limit is applied. `package.json` stays in full when the slice contains it.

The summary covers `package-lock.json`, `npm-shrinkwrap.json`, `pnpm-lock.yaml`, and `yarn.lock`. It records the lockfile version, the package count, direct resolved versions, and any resolution outside `registry.npmjs.org`. `npm-shrinkwrap.json` is summarized when it appears in a slice. It is not, by itself, a dependency-manifest signal.

A slice longer than 120,000 characters drops whole files, largest first. The job records the dropped paths and marks the review truncated. It does not cut a file in the middle. A truncated review fails. Scope and defects also receive each issue the commit messages cite, using the manifest issue-tracker format. Other hunters do not.

Instructions inside the diff, commit messages, docs, and issue text are untrusted. The rule is in [finding-contract.md](finding-contract.md).

## Bounded fix

The fixer runs only when this run's verdict contains an implementable evidenced major from a blocking hunter. Nits, demoted majors, advisory findings, and a truncation claim that comes only from the pipeline are not in the work list.

After the fixer checks out the pull-request head, the job cleans that tree again before the editing agent writes anything. The clean resets tracked files to `HEAD` and deletes untracked files. Ignored files stay, including installed dependency directories.

The editing agent does not run a shell command, does not spawn a subagent, and does not edit a command allowlist or a CLI config. It reads only the finding's file, the files named in the failure path, and tests that already mention those symbols. It does not read installed dependency directories. It does not start a datastore, a container runtime, or an HTTP server. For that agent, `verification` describes the behavior a test must lock in. The agent does not execute that field. The schema is in [finding-contract.md](finding-contract.md).

After the editing agent exits, the job runs the manifest `tests.affected` command against the pull-request base. The job does not commit if that command fails. A test failure does not consume the attempt. A missing base skips with `base-unknown` and does not scan the whole history.

When the job was queued on a synthetic merge, the fixer has switched to the pull-request head, and that head is the merge's second parent, the affected-test command sees that head as the change under test. Any other checkout keeps the queued revision, so selection fails closed. An empty queued revision leaves the command's environment unchanged.

When the manifest sets `issue_tracker.data_dir` and that directory is tracked, the job restores it from `HEAD` before it stages the fix. Tracker files cannot ride along in the commit.

A pull request may contain at most two commits whose message has a trailer line matching `^Adversary-Fix: [0-9]+$`. The count is commits in `base..HEAD`. Commits already on the base do not count. Each job pushes at most one commit, by a normal fast-forward push, and only onto the pull-request branch. That push starts a new review of the new head before another fix is considered. A later pass skips with `no-evidenced-majors` and does not consume an attempt. The commit message lists the claims and ends with the trailer for that attempt.

Skip reasons are `no-verdict`, `no-evidenced-majors`, `hunter-report-missing`, `no-implementable-majors`, `base-unknown`, and `fix-limit-reached`. A skip writes `fix-report.json` and exits success. The failed verdict remains the job failure.

The job does not commit a gate path or a secret (a path ending in `.env`, containing `.env.`, or named `credentials.json` or ending in `.pem`). It does not write git config. When the checkout has no commit identity, the job supplies one for that commit only, from the manifest `fixer_identity`.

## Three failures

An optional early review stores a fail count at `.git/adversary-fail-count`. A pass sets it to 0. When the count is already 3, do not queue another run. Report the comparison and stop. The same tree and the same ask are not submitted twice.

## Gate lock

The pre-commit hook refuses a commit when a staged path matches the manifest `gate_paths`. It then exits success. It does not classify the diff, write a routing file, or record a bypass. A human commits a gate change by skipping the hook once, after reviewing that change.

The merge check classifies the full pull-request diff. Why that classification is sufficient, and why the hook no longer classifies, is in [../lessons/from-hand-contracting.md](../lessons/from-hand-contracting.md).

When `issue_tracker` is not null, the commit-message hook appends the trailer only when `list_in_progress` returns exactly one id and the message does not already contain that trailer. Several ids write nothing. The review job reads the trailer from the commit messages. It does not write the trailer. Scope and defects load the cited issue.
