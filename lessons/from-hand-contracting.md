---
title: Lessons from the reference workflow
description: What the kit keeps, what stays product-specific, and what was editor packaging. Each lesson cites a reference path.
type: context
---

# Lessons from the reference workflow

The reference repository is the Hand Contracting showcase. Its review workflow is the source. Its product rules are not. Where a workflow document and a script disagree, the kit follows the script and records the disagreement in this file.

The kit was read on 2026-10-13 at reference `6bd1f7c`. That tip is past `1ed74c0` and already includes `d3792c5`, `490af3e`, and `1ed74c0`. A later edit removed classification from the pre-commit hook. The kit follows `.githooks/pre-commit` and `scripts/adversary/classify-run.sh` after that edit. Product rules stay citations.

The reference paths below are citations. They are not install inputs.

## Keep in every project

- The implementer does not load the report, hunter, or collate roles, does not classify the diff, and does not write verdict files. Source: `.cursor/rules/adversarial-commit.mdc`, `.cursor/skills/adversarial-commit/SKILL.md`.
- The pre-commit hook refuses a staged change to the review machinery and then exits success. An ordinary commit proceeds. The value is that this lock stops the commit from being created, and the pull-request job cannot, because the job runs only after the commit exists. A human commits that change by skipping the hook once. Source: `.githooks/pre-commit`, `docs/workflow/adversarial-commit.md`.
- The commit-message hook appends an issue trailer when exactly one issue is in progress. Scope and defects load that trailer from the commit messages. The job does not write the trailer. Source: `.githooks/prepare-commit-msg`, `scripts/adversary/bead-context.py`, `scripts/adversary/run-agents.sh`.
- The merge check classifies the full pull-request diff. The hook does not. A local routing file is not an input. Source: `scripts/adversary/classify-run.sh`, `azure-pipelines/adversary.yml`, `.githooks/pre-commit`.
- Routing is deterministic. `decision_source` is `policy-v1.2`. The same diff yields the same risk, signals, and hunters. A long diff is not required for high risk. A docs-only change with no signal stays low. Source: `scripts/adversary/collate-decision.py` (`route_risk`, `POLICY`), `docs/workflow/adversarial-commit.md`.
- Deterministic gates run before hunters, including on a low decision. A gate failure fails the job before any hunter starts. Source: `scripts/adversary/deterministic-gates.sh`, `azure-pipelines/adversary.yml`.
- Selected hunters run at the same time. Each sees the ask, the gate report, and its own slice. None sees another hunter's report. Source: `scripts/adversary/run-agents.sh`.
- On a merge check the ask is the pull-request title plus every non-merge commit subject, newest first, duplicates removed. An early `adversary-request <tree>` commit keeps the ask in its body. Source: `scripts/adversary/classify-run.sh`, `scripts/adversary/collate-decision.py` (`extract_ask`).
- A blocking major includes `invariant`, `path`, `line` (integer at least 1), `failurePath`, and `verification`. A major missing any of those is demoted to a nit and does not block. The verdict counts evidenced majors, not `askSatisfied`. Source: `scripts/adversary/collate-decision.py` (`evidenced_major`, `counted_finding`, `decide`), `.cursor/skills/adversary-report/SKILL.md`.
- Readability and maintainability stay registered with vote `advisory`. The default policy does not select them. An advisory major is demoted to a nit. Source: `scripts/adversary/registry`, `scripts/adversary/collate-decision.py`.
- Collate is a decision procedure. The merge pipeline sets `ADVERSARY_SKIP_AGENT=1`, so the step does not call a model. Pass means every selected blocking hunter reported and none has an evidenced major. No selected blocking hunter is also a pass. A missing or mismatched blocking report is `missing` and fails. Source: `scripts/adversary/collate.sh`, `scripts/adversary/collate-decision.py` (`decide`), `azure-pipelines/adversary.yml`.
- The fix stage runs only for an implementable evidenced major, pushes at most one `Adversary-Fix` commit per job, and allows at most two such commits on the pull request. The job commits only after the affected tests pass. A skipped fixer exits success and leaves a failed verdict failed. A truncation claim whose only agent is `pipeline` is not implementable. Source: `scripts/adversary/fix-majors.sh`, `scripts/adversary/fix-plan.py`.
- Before classify, the job resets tracked files to `HEAD` and deletes untracked files. Ignored files stay, including installed dependency directories. A self-hosted agent reuses its work directory, so a later `git add` would otherwise publish leftover edits. After the fixer checks out the pull-request head, it cleans that tree again before the editing agent writes anything. Source: `scripts/adversary/lib.sh` (`adversary_clean_checkout`), `scripts/adversary/clean-checkout.sh`, `azure-pipelines/adversary.yml`, `scripts/adversary/fix-majors.sh`.
- The editing agent does not run a shell command, does not spawn a subagent, and does not edit a command allowlist or CLI config. It reads only the finding's file, the files named in the failure path, and tests that already mention those symbols. It does not read installed dependency directories. It does not start a datastore, a container runtime, or an HTTP server. Source: `scripts/adversary/fix-plan.py` (`render_fix_prompt`).
- The `verification` field on an evidenced major is a way to reproduce the failure. For the editing agent it describes the behavior a test must lock in. It is not a command to execute. The review job runs the manifest `tests.affected` command after the editing agent exits, and it does not commit if that command fails. A test failure does not consume the attempt. Source: `scripts/adversary/fix-plan.py`, `scripts/adversary/fix-majors.sh`.
- When the job was queued on a synthetic merge and the fixer has switched to the pull-request head, and that head is the merge's second parent, the affected-test command sees that head as the change under test. Any other checkout keeps the queued revision, so selection fails closed. An empty queued revision leaves the command's environment unchanged. Source: `scripts/adversary/fix-majors.sh` (`affected_source_version`).
- When the manifest issue tracker has a data directory and that directory is tracked, the job restores it from `HEAD` before staging the fix commit. Tracker files cannot ride along. The reference directory is `.beads/`. Source: `scripts/adversary/fix-majors.sh`.
- A hunter that exits non-zero, or that finishes with no valid report, is started once more. A second miss fails that hunter. The attempt count is 1 or 2. The default is 2. The job reads the last `<<<REPORT>>>` … `<<<END>>>` block. Invalid JSON writes no report file. Source: `scripts/adversary/lib.sh` (`adversary_collect_report`, `adversary_write_report`), `scripts/adversary/run-agents.sh`.
- A slice over 120,000 characters drops whole files, largest first, and the review fails. Lockfile bodies are summarized. `package.json` stays in full. Instructions in the diff, commit messages, and docs are untrusted data. Source: `scripts/adversary/collate-decision.py` (`fit_slice`, `summarize_lockfile`), `docs/adversarial-review-agent-contract.md`, `scripts/adversary/fix-plan.py` (`render_fix_prompt`).
- The agent process keeps a real home directory. An empty home makes a file read hang for about 16 minutes. Source: `scripts/adversary/run-cursor-agent.sh`.
- After three failed early reviews, stop and report. Do not queue another run. A pass resets the count. The same tree and ask are not sent twice. Source: `scripts/adversary/request-review.sh`.

## What left the hook

- Classification, a local routing file, a high-risk warning, and a bypass note left the pre-commit hook. The pull-request job already classifies the full diff and writes its own routing decision. `scripts/adversary/classify-run.sh` computes that decision from the pull-request diff. `scripts/adversary/request-review.sh` and `azure-pipelines/adversary.yml` do not read a hook routing file or a bypass variable. The hook no longer classifies. Source: `.githooks/pre-commit`, `scripts/adversary/classify-run.sh`, `scripts/adversary/request-review.sh`, `azure-pipelines/adversary.yml`.

## Where the running code won

- The opening paragraph of `docs/workflow/adversarial-commit.md` now matches the hook. The hook refuses a staged gate change and does not accept a commit based on a unanimous hunter pass. That pass is the job's collated verdict in `scripts/adversary/check-verdict.sh`. Source: `.githooks/pre-commit`, `docs/workflow/adversarial-commit.md`, `scripts/adversary/check-verdict.sh`.
- The module docstring of `scripts/adversary/collate-decision.py` says a pass needs every registered hunter to be satisfied and to report no majors. `decide` uses the selected blocking hunters and evidenced majors, and it does not read `askSatisfied`. The kit follows `decide`.
- `docs/adversarial-review-agent-contract.md` names deployment, compatibility, performance, and compliance specialists, and a model orchestrator. The registry does not include them. Deployment and migration signals select `security` and `defects`. The kit does not add those specialists.
- That contract's finding schema (confidence, blocking flag, categories) is not what `report_errors` accepts. The kit uses the report object the script validates.
- The contract's pilot rule that every finding stays non-blocking is not what the registry does. `scope`, `defects`, and `security` are blocking.
- `collate.sh` can still start a narrative agent when `ADVERSARY_SKIP_AGENT` is unset. The merge pipeline always sets it. The kit keeps the pipeline behavior and does not describe a second model review.
- The workflow signal table lists `fetch(`, `axios(`, `spawn(`, and `exec(`. The classifier also matches `child_process(`. The kit includes that match.
- `npm-shrinkwrap.json` is in `LOCKFILE_NAMES` and is summarized. It is not in the `dependency-manifest-changed` path expression. The kit keeps both facts.
- The hunter prompt in `scripts/adversary/run-agents.sh` does not repeat the untrusted-content sentence. The contract states it, and `render_fix_prompt` treats repository files, commit messages, and finding text as data. The kit keeps the rule in one place, the finding contract.
- `run-cursor-fixer.sh` does not repeat the empty-home comment. It also does not clear the home directory. The kit states the constraint on every role process.
- `docs/workflow/adversarial-commit.md` says the fix prompt reads only the finding's files. `render_fix_prompt` also allows tests that already mention those symbols, and it forbids a shell, a subagent, and edits to a command allowlist. The kit follows the prompt.
- `docs/workflow/adversarial-commit.md` says the affected-test command receives the pull-request head when that head is the merge's second parent, and otherwise keeps the queued revision. It does not mention an empty queued revision. `affected_source_version` in `scripts/adversary/fix-majors.sh` leaves the command environment unchanged in that case. The kit follows the script.
- The fix prompt forbids a shell. `scripts/adversary/run-cursor-fixer.sh` still writes a private allowlist that permits `ls`, `npm`, and `node`. The kit keeps the prohibition on the editing agent and treats the allowlist as editor packaging.

## Reference product

These stay in the reference. The manifest has a slot for each. The kit's filled example does not use the reference values.

- Precedence, the active phase, and accepted decisions. Source: `docs/README.md`. The reference scope hunter reads that index, `docs/phases/`, and `docs/adr/`. Source: `.cursor/skills/adversary-scope/SKILL.md`.
- Product checks in the scope and security skills: datastore and rate-limiter bans, TLS inside the Node process, framework imports from `src/domain/`, the business name, and review `contact_email`. Source: `.cursor/skills/adversary-scope/SKILL.md`, `.cursor/skills/adversary-security/SKILL.md`, `scripts/adversary/deterministic-gates.sh`.
- Critical path `src/domain/`. Source: `scripts/adversary/critical-paths`.
- Gate tests `npm test`. The fix stage runs `npm run test:affected` after the editing agent exits. The full suite is `npm run test:full`. Source: `scripts/adversary/deterministic-gates.sh`, `scripts/adversary/fix-majors.sh`, `docs/workflow/testing.md`.
- Issue ids as `Bead: <id>`, and only the single in-progress bead. Scope and defects receive cited beads. Other hunters do not. `.beads/` is omitted from slices. Source: `.githooks/prepare-commit-msg`, `scripts/adversary/bead-context.py`, `scripts/adversary/collate-decision.py` (`omitted_from_review`).
- Defects also sees `next.config.ts`, `next.config.js`, and `next.config.mjs`. Source: `scripts/adversary/collate-decision.py` (`defects_behavior_path`).
- The CI signal id `azure-pipeline-changed` matches `^azure-pipelines/.+\.ya?ml$`. The kit's portable id is `ci-config-changed`, and the path list is a manifest slot. Source: `scripts/adversary/collate-decision.py` (`PATH_SIGNALS`).
- Pipeline name, organization, project, and pool. Source: `docs/workflow/adversarial-commit.md`, `azure-pipelines/adversary.yml`. The kit's worked example uses placeholders.
- Early-review defaults for the organization URL, the project, and the pipeline name. Source: `scripts/adversary/request-review.sh`.

## Reference editor packaging

- Session text was an always-apply rule plus a skill directory loaded by name. Source: `.cursor/rules/adversarial-commit.mdc`, `scripts/adversary/lib.sh`.
- Hunters were read-only ask-mode processes. Defects and security used the reasoning model. Scope used the fast model. Limits were 180 seconds and 900 seconds. A hunter that returns no report is started once more. One retry of the 900 second limit still fits in the 120 minute job. Source: `scripts/adversary/run-cursor-agent.sh`, `scripts/adversary/lib.sh` (`adversary_collect_report`), `azure-pipelines/adversary.yml`, `docs/workflow/adversarial-commit.md`.
- The fixer was a separate writable process on the fast model, with a 10 minute limit inside that 120 minute job. That process receives a private command allowlist of `ls`, `npm`, and `node`. `git`, Docker, and the interactive CLI config stay out of reach. Hunters do not receive that directory. Source: `scripts/adversary/run-cursor-fixer.sh`, `azure-pipelines/adversary.yml`, `docs/workflow/adversarial-commit.md`.
- The secret was a library variable consumed by those processes. The kit names a placeholder variable and does not copy the secret. Source: `azure-pipelines/adversary.yml`.
