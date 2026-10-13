---
title: Reference editor map
description: Non-normative map from the reference Cursor rules and skills onto the kit role files.
type: context
---

# Reference editor map

This file is not required to install the kit. The installer never reads it. It exists so a reader can see which reference file each behavior came from. The reference editor packaged the workflow as Cursor rules and skills. The kit stores the same jobs as markdown roles.

| Reference file | Kit role |
| --- | --- |
| `.cursor/rules/adversarial-commit.mdc` | [roles/implementer.md](../roles/implementer.md), as the session pointer |
| `.cursor/skills/adversarial-commit/SKILL.md` | [roles/implementer.md](../roles/implementer.md) |
| `.cursor/skills/adversary-report/SKILL.md` | [roles/report.md](../roles/report.md) |
| `.cursor/skills/adversary-scope/SKILL.md` | [roles/hunters/scope.md](../roles/hunters/scope.md) |
| `.cursor/skills/adversary-defects/SKILL.md` | [roles/hunters/defects.md](../roles/hunters/defects.md) |
| `.cursor/skills/adversary-security/SKILL.md` | [roles/hunters/security.md](../roles/hunters/security.md) |
| `.cursor/skills/adversary-readability/SKILL.md` | [roles/hunters/readability.md](../roles/hunters/readability.md) |
| `.cursor/skills/adversary-maintainability/SKILL.md` | [roles/hunters/maintainability.md](../roles/hunters/maintainability.md) |
| `.cursor/skills/adversary-collate/SKILL.md` | [roles/collate.md](../roles/collate.md) |

The reference loaded a skill by directory name through `scripts/adversary/lib.sh`, which concatenated `.cursor/skills/<name>/SKILL.md`. A review job in the kit passes a role path instead. `scripts/adversary/run-cursor-agent.sh` starts the read-only hunter process. `scripts/adversary/run-cursor-fixer.sh` starts the process that may edit. Those launchers are editor-specific. The binding's `invoke_role` field is the portable replacement.

The reference hunter launcher keeps the process home directory because an empty home makes a file read hang. That constraint is in the binding schema, not in a role.

The fixer launcher is editor packaging. On the reference host it uses the fast model for 10 minutes. The process receives a private command allowlist of `ls`, `npm`, and `node`. `git`, Docker, and the interactive CLI config stay out of reach. Hunters do not receive that directory. Those limits and the allowlist stay out of the role files. The editing agent's procedure is [../workflow/review-model.md](../workflow/review-model.md).

`scripts/adversary/lib.sh` (`adversary_collect_report`) starts a hunter once more when it exits non-zero or returns no valid report. `scripts/adversary/run-agents.sh` calls that collector. A second miss fails the hunter. The attempt count is 1 or 2, and the default is 2. One retry of the 900 second hunter limit still fits in the 120 minute job. The host-neutral retry is in [../ci/job-shape.md](../ci/job-shape.md). The reference directory restored before a fix commit is `.beads/`. The portable rule uses the manifest `issue_tracker.data_dir` slot.
