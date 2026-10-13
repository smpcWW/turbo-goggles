---
title: Review job shape
description: Host-neutral stages for classify, gates, hunters, collate, bounded fix, and publish, plus one worked CI example.
type: plan
---

# Review job shape

The review job is the merge check. It classifies the pull-request diff itself. It does not reuse a routing file from the developer's `.git` directory. Roles stay the files named in the binding. This document does not change those roles.

The agent process keeps the home directory from `agent_home` in the binding. An empty home makes file reads hang.

## Stages

The job checks out with full history and credentials so the fixer can fast-forward the pull-request branch. On a host that queues a synthetic merge, that first checkout is the merge, and the fixer then switches to the pull-request head. The job writes its workspace under a job output directory. The clean, the editing agent, and the affected-test command follow [../workflow/review-model.md](../workflow/review-model.md).

1. **Clean.** Before classify, reset tracked files to `HEAD` and delete untracked files. Leave ignored files, including installed dependency directories. A reused work directory would otherwise let a later add publish leftover edits.
2. **Classify.** Diff the head against the merge base of the default branch. Write `routing.json`, `base.txt`, and, when the decision is high, `ask.txt`. The ask is the pull-request title plus every non-merge commit subject, as the review model defines. A low decision sets the hunters and collate stages idle. Gates still run.
3. **Gates.** Run the manifest `tests.unit` command when it is set, then every `gates.forbidden_imports` rule and `gates.required_checks` entry. Write `gates.txt`. A failure fails the job. Do not start hunters.
4. **Hunters.** Run only when classify succeeded, the decision is high, and gates succeeded. Start every id in `selected_hunters` at the same time. Invoke each one with `invoke_role` and that hunter's role path. Pass the ask, the gate report, and the slice from `hunter_files`. Do not pass another hunter's report. Scope and defects also receive cited issues when the manifest has an issue tracker. A hunter that exits non-zero, or that finishes with no valid report, is started once more. A second miss fails that hunter. The attempt count is 1 or 2. The default is 2. Report collection follows [../workflow/finding-contract.md](../workflow/finding-contract.md).
5. **Collate.** Run only when the previous stages succeeded and classify did not idle this stage. Invoke the collate role with the reports and the routing decision. Do not pass the diff. Do not call a model to review the code. A fail verdict fails the job.
6. **Bounded fix.** Run even when collate failed, and also when hunters were idle. Invoke the fixer role only when the verdict has an implementable evidenced major and the pull request is under the attempt limit. Otherwise write `fix-report.json` with the skip reason and exit success, so a failed verdict stays the job failure. Idle hunters produce no verdict; the fixer skips with `no-verdict` and does not call an agent. After the fixer checks out the pull-request head, clean the tree again before the editing agent writes anything. The editing agent does not run a shell. After it exits, the job runs the manifest `tests.affected` command. The job does not commit when that command fails, and a test failure does not consume the attempt. When the job was queued on a synthetic merge and the fixer has switched to the pull-request head, and that head is the merge's second parent, the affected-test command sees that head as the change under test. Any other checkout keeps the queued revision, so selection fails closed. An empty queued revision leaves the command's environment unchanged. When the manifest sets `issue_tracker.data_dir` and that directory is tracked, restore it from `HEAD` before staging the fix.
7. **Publish.** Always run, including when hunters were idle and when an earlier stage failed. Publish `routing.json` when it exists, plus `verdict.json` and `fix-report.json` when they exist, as one review artifact.

Pushing a branch does not start this job. Opening or updating a pull request into `default_branch` does. An optional early run may start the same job by an explicit request. That request does not enable a second trigger on the same push.

## Worked example

This example is one host. Replace the example when the project uses another host. Do not rewrite the roles or the stages above.

Placeholders: `<organization>`, `<project>`, `<pool>`, `<pipeline-name>`, `<default-branch>`, `<agent-credential-group>`, `<agent-api-key>`, `<queued-revision>`.

The secret is the library variable `<agent-api-key>` in group `<agent-credential-group>`. Authorize the pipeline to use that group. Do not commit the secret. Do not print it. The hunter and fixer steps inherit it.

```yaml
# Pipeline name: <pipeline-name>
# Organization: https://dev.azure.com/<organization>/
# Project: <project>
# Pool: <pool>

trigger: none

pr:
  branches:
    include:
      - <default-branch>

variables:
  - group: <agent-credential-group>

jobs:
  - job: Review
    displayName: Adversarial review
    timeoutInMinutes: 120
    pool:
      name: <pool>
    steps:
      - checkout: self
        fetchDepth: 0
        persistCredentials: true
        displayName: Checkout review tree

      - bash: clean-checkout
        displayName: Clean the checkout

      - bash: classify-run
        displayName: Classify the change

      - bash: deterministic-gates
        displayName: Deterministic gates

      - bash: run-hunters
        displayName: Run selected hunters
        condition: and(succeeded(), ne(variables['skipHunters'], 'true'))
        env:
          AGENT_API_KEY: $(<agent-api-key>)

      - bash: collate
        displayName: Collate hunter reports
        condition: and(succeeded(), ne(variables['skipCollate'], 'true'))
        env:
          AGENT_API_KEY: $(<agent-api-key>)

      - bash: fix-majors
        displayName: Fix evidenced majors
        condition: succeededOrFailed()
        env:
          AGENT_API_KEY: $(<agent-api-key>)

      - bash: stage-review-artifact
        displayName: Stage review artifact
        condition: always()

      - publish: $(Build.ArtifactStagingDirectory)
        artifact: verdict
        displayName: Publish review artifact
        condition: always()
```

`classify-run` sets `skipHunters` when the decision is low. On a pull request it also sets `skipCollate` when the decision is low. `fix-majors` still runs and skips because there is no verdict. `stage-review-artifact` copies `routing.json`, `verdict.json`, and `fix-report.json` when each file exists. The job timeout has to cover hunter time, one hunter retry, and one fixer attempt. The reference host limits scope to 180 seconds and defects and security to 900 seconds. The fixer on that host uses the fast model for 10 minutes. One retry of the 900 second limit still fits in the 120 minute job. Put those limits in the host's own step configuration, not in the role files.

`<queued-revision>` is the revision the job was queued with. When this checkout is a synthetic merge and the fixer has switched to the pull-request head, and that head is the merge's second parent, the affected-test command sees that head as the change under test. Any other checkout keeps `<queued-revision>`. An empty `<queued-revision>` leaves the command environment unchanged.
