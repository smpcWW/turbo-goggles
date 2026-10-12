---
title: Lessons from the reference workflow
description: What the kit keeps, what stays product-specific, and what was editor packaging. Each lesson cites a reference path.
type: context
---

# Lessons from the reference workflow

The reference repository is the Hand Contracting showcase. Its review workflow is the source. Its product rules are not. Where the draft design and the running code disagree, the kit follows the workflow doc and the scripts.

The reference paths below are citations. They are not install inputs.

## Keep in every project

- The implementer does not load the report, hunter, or collate roles, and does not write routing or verdict files. Source: `.cursor/rules/adversarial-commit.mdc`, `.cursor/skills/adversarial-commit/SKILL.md`.
- The pre-commit hook classifies the staged diff, writes `.git/adversary-routing/<tree>.json` from `git write-tree`, and warns on high risk. It does not run hunters and it does not block an ordinary commit. Source: `.githooks/pre-commit`, `docs/workflow/adversarial-commit.md` (loop step 1).
- A staged change to the gate is refused before any bypass is considered. A human commits it by skipping the hook. `ADVERSARY_BYPASS` only fills the log field on an ordinary commit. Source: `.githooks/pre-commit`, `.cursor/skills/adversarial-commit/SKILL.md`.
- The merge check classifies the full pull-request diff again. A local routing file is not an input. Source: `scripts/adversary/classify-run.sh`, `azure-pipelines/adversary.yml`.
- Routing is deterministic. `decision_source` is `policy-v1.2`. The same diff yields the same risk, signals, and hunters. A long diff is not required for high risk. A docs-only change with no signal stays low. Source: `scripts/adversary/collate-decision.py` (`route_risk`, `POLICY`), `docs/workflow/adversarial-commit.md`.
- Deterministic gates run before hunters, including on a low decision. A gate failure fails the job before any hunter starts. Source: `scripts/adversary/deterministic-gates.sh`, `azure-pipelines/adversary.yml`.
- Selected hunters run at the same time. Each sees the ask, the gate report, and its own slice. None sees another hunter's report. Source: `scripts/adversary/run-agents.sh`.
- On a merge check the ask is the pull-request title plus every non-merge commit subject, newest first, duplicates removed. An early `adversary-request <tree>` commit keeps the ask in its body. Source: `scripts/adversary/classify-run.sh`, `scripts/adversary/collate-decision.py` (`extract_ask`).
- A blocking major includes `invariant`, `path`, `line` (integer at least 1), `failurePath`, and `verification`. A major missing any of those is demoted to a nit and does not block. The verdict counts evidenced majors, not `askSatisfied`. Source: `scripts/adversary/collate-decision.py` (`evidenced_major`, `counted_finding`, `decide`), `.cursor/skills/adversary-report/SKILL.md`.
- Readability and maintainability stay registered with vote `advisory`. The default policy does not select them. An advisory major is demoted to a nit. Source: `scripts/adversary/registry`, `scripts/adversary/collate-decision.py`.
- Collate is a decision procedure. The merge pipeline sets `ADVERSARY_SKIP_AGENT=1`, so the step does not call a model. Pass means every selected blocking hunter reported and none has an evidenced major. No selected blocking hunter is also a pass. A missing or mismatched blocking report is `missing` and fails. Source: `scripts/adversary/collate.sh`, `scripts/adversary/collate-decision.py` (`decide`), `azure-pipelines/adversary.yml`.
- The fixer runs only for an implementable evidenced major, pushes at most one `Adversary-Fix` commit per job, and allows at most two such commits on the pull request. It commits only after the affected tests pass. A skipped fixer exits success and leaves a failed verdict failed. A truncation claim whose only agent is `pipeline` is not implementable. Source: `scripts/adversary/fix-majors.sh`, `scripts/adversary/fix-plan.py`.
- A slice over 120,000 characters drops whole files, largest first, and the review fails. Lockfile bodies are summarized. `package.json` stays in full. Instructions in the diff, commit messages, and docs are untrusted data. Source: `scripts/adversary/collate-decision.py` (`fit_slice`, `summarize_lockfile`), `docs/adversarial-review-agent-contract.md`, `scripts/adversary/fix-plan.py` (`render_fix_prompt`).
- The agent process keeps a real home directory. An empty home makes a file read hang for about 16 minutes. Source: `scripts/adversary/run-cursor-agent.sh`.
- After three failed early reviews, stop and report. Do not queue another run. A pass resets the count. The same tree and ask are not sent twice. Source: `scripts/adversary/request-review.sh`.

## Where the running code won

- The first paragraph of `docs/workflow/adversarial-commit.md` says the hook accepts a commit only on a unanimous pass. The hook and loop step 1 do not. The kit follows the hook.
- The module docstring of `scripts/adversary/collate-decision.py` says a pass needs every registered hunter to be satisfied and to report no majors. `decide` uses the selected blocking hunters and evidenced majors, and it does not read `askSatisfied`. The kit follows `decide`.
- `docs/adversarial-review-agent-contract.md` names deployment, compatibility, performance, and compliance specialists, and a model orchestrator. The registry does not include them. Deployment and migration signals select `security` and `defects`. The kit does not add those specialists.
- That contract's finding schema (confidence, blocking flag, categories) is not what `report_errors` accepts. The kit uses the report object the script validates.
- The contract's pilot rule that every finding stays non-blocking is not what the registry does. `scope`, `defects`, and `security` are blocking.
- `collate.sh` can still start a narrative agent when `ADVERSARY_SKIP_AGENT` is unset. The merge pipeline always sets it. The kit keeps the pipeline behavior and does not describe a second model review.
- The workflow signal table lists `fetch(`, `axios(`, `spawn(`, and `exec(`. The classifier also matches `child_process(`. The kit includes that match.
- `npm-shrinkwrap.json` is in `LOCKFILE_NAMES` and is summarized. It is not in the `dependency-manifest-changed` path expression. The kit keeps both facts.
- The hunter prompt in `scripts/adversary/run-agents.sh` does not repeat the untrusted-content sentence. The contract states it, and `render_fix_prompt` treats repository files, commit messages, and finding text as data. The kit keeps the rule in one place, the finding contract.
- `run-cursor-fixer.sh` does not repeat the empty-home comment. It also does not clear the home directory. The kit states the constraint on every role process.

## Reference product

These stay in the reference. The manifest has a slot for each. The kit's filled example does not use the reference values.

- Precedence, the active phase, and accepted decisions. Source: `docs/README.md`. The reference scope hunter reads that index, `docs/phases/`, and `docs/adr/`. Source: `.cursor/skills/adversary-scope/SKILL.md`.
- Product checks in the scope and security skills: datastore and rate-limiter bans, TLS inside the Node process, framework imports from `src/domain/`, the business name, and review `contact_email`. Source: `.cursor/skills/adversary-scope/SKILL.md`, `.cursor/skills/adversary-security/SKILL.md`, `scripts/adversary/deterministic-gates.sh`.
- Critical path `src/domain/`. Source: `scripts/adversary/critical-paths`.
- Gate tests `npm test`, fixer tests `npm run test:affected`, and the full suite `npm run test:full`. Source: `scripts/adversary/deterministic-gates.sh`, `scripts/adversary/fix-majors.sh`, `docs/workflow/testing.md`.
- Issue ids as `Bead: <id>`, and only the single in-progress bead. Scope and defects receive cited beads. Other hunters do not. `.beads/` is omitted from slices. Source: `.githooks/prepare-commit-msg`, `scripts/adversary/bead-context.py`, `scripts/adversary/collate-decision.py` (`omitted_from_review`).
- Defects also sees `next.config.ts`, `next.config.js`, and `next.config.mjs`. Source: `scripts/adversary/collate-decision.py` (`defects_behavior_path`).
- The CI signal id `azure-pipeline-changed` matches `^azure-pipelines/.+\.ya?ml$`. The kit's portable id is `ci-config-changed`, and the path list is a manifest slot. Source: `scripts/adversary/collate-decision.py` (`PATH_SIGNALS`).
- Pipeline name, organization, project, and pool. Source: `docs/workflow/adversarial-commit.md`, `azure-pipelines/adversary.yml`. The kit's worked example uses placeholders.
- Early-review defaults for the organization URL, the project, and the pipeline name. Source: `scripts/adversary/request-review.sh`.

## Reference editor packaging

- Session text was an always-apply rule plus a skill directory loaded by name. Source: `.cursor/rules/adversarial-commit.mdc`, `scripts/adversary/lib.sh`.
- Hunters were read-only ask-mode processes. Defects and security used the reasoning model. Scope used the fast model. Limits were 180 seconds and 900 seconds. The fixer was a separate writable process with a 60 minute limit inside a 120 minute job. Source: `scripts/adversary/run-cursor-agent.sh`, `scripts/adversary/run-cursor-fixer.sh`, `azure-pipelines/adversary.yml`.
- The secret was a library variable consumed by those processes. The kit names a placeholder variable and does not copy the secret. Source: `azure-pipelines/adversary.yml`.
