---
title: Environment binding schema
description: Fields an agent host fills in so a review job can load the shared role files by path.
type: plan
---

# Environment binding schema

A binding is not a second copy of the roles. The role files under `roles/` are what every host executes. The binding records how this host finds those files and how it starts one role.

The installer writes the binding inside the target project. Paths in the binding are relative to that project's git toplevel.

## Fields

| Field | Required | Meaning |
| --- | --- | --- |
| `session_entry` | yes | File this host loads at session start. The installer creates `AGENTS.md` when the project has none, and records that choice here. |
| `roles_dir` | yes | Directory of the installed role pack, relative to the project root. |
| `workflow_dir` | yes | Directory of the installed workflow text the roles link to. |
| `manifest` | yes | Path of the filled project manifest. |
| `invoke_role` | yes | Command or job line that passes one role file path to the agent. The path is an argument. It is not copied into a host-specific skill folder. |
| `gate_hook.install` | yes | Command that points this clone at the gate hook. |
| `gate_hook.entry` | yes | Path of the pre-commit hook the installer wrote. The hook implements the gate lock in [../workflow/review-model.md](../workflow/review-model.md). |
| `agent_home` | yes | The review job must inherit a real home directory. It must not set the home directory to an empty directory. |

`invoke_role` uses the token `{role}` for the role path. A review job replaces `{role}` with one of `roles/implementer.md`, `roles/report.md`, `roles/collate.md`, `roles/fixer.md`, or `roles/hunters/<id>.md` under `roles_dir`. Hunters also receive the ask, the gate report, and that hunter's slice. Collate does not receive the diff. The fixer receives the implementable majors, not a request to review.

The installer succeeds without [reference-cursor.md](reference-cursor.md). That file is a historical map, not a binding.

## Filled example

Paths below are inside the target project. The host command is a placeholder for whatever program that project uses to start an agent.

```yaml
session_entry: AGENTS.md
roles_dir: adversary/roles
workflow_dir: adversary/workflow
manifest: adversary/project-manifest.md
invoke_role: "review-agent --workspace . --manifest adversary/project-manifest.md --role {role}"
gate_hook:
  install: "git config core.hooksPath adversary/hooks"
  entry: adversary/hooks/pre-commit
agent_home: "inherit the invoking user's home directory; do not set it to an empty directory"
```

A second host replaces this file. It does not rewrite the role pack.
