---
title: Project manifest schema
description: Slots the installer fills from the target repository. The filled example is a generic service, not a reference product.
type: plan
---

# Project manifest schema

The installer writes this manifest in the target project. The kit's role text stays unchanged. The review job reads the manifest for every slot below. An empty optional slot means that input is absent. Do not invent a test runner, an issue tracker, or a forbidden import.

The normative match rules are [routing-policy.md](routing-policy.md). This file is the slot list. `policy_version` records which signal table was copied. `kit_revision` is the kit git SHA at install time.

## Slots

| Slot | Required | What the installer records |
| --- | --- | --- |
| `default_branch` | yes | The branch pull requests merge into. |
| `tests.unit` | no | Command for the gate's required test check. Empty skips that check. |
| `tests.affected` | no | Command the review job runs after the editing agent exits. An empty command means the job cannot commit a fix. |
| `tests.full` | no | Full-suite command. The review job does not have to run it. |
| `accepted_decisions` | yes | Path to accepted decisions, or `none`. |
| `active_scope` | yes | Path to the active scope document, or `none`. |
| `critical_paths` | yes | Prefixes for `critical-component-changed`. May be empty. |
| `gates.forbidden_imports` | no | Path, pattern, and message for an import the target already forbids. |
| `gates.required_checks` | no | Extra deterministic checks the target already has, besides `tests.unit`. |
| `issue_tracker` | no | `null` when the project has none. Otherwise an id format and how to list the single in-progress issue. |
| `policy_version` | yes | `policy-v1.2` unless the project changes the signal table. |
| `signals` | yes | The signal table copied from the routing policy, with this project's path overrides. |
| `instruction_entry` | yes | The file this agent host loads at session start. |
| `ci_job_file` | no | The review job file in this project, or `null`. |
| `ci_config_paths` | yes | Paths that raise `ci-config-changed`. Empty when the project has no CI config. |
| `public_api_paths` | no | When set, replaces the default public-API pattern. |
| `product_root` | yes | Default `src/`. |
| `hunter_paths` | yes | Subject prefixes for each hunter. |
| `defects_behavior_paths` | yes | Extra runtime files defects must see. |
| `gate_paths` | yes | Paths the pre-commit hook refuses until a human commits them by skipping the hook once. |
| `fixer_identity` | yes | Name and email used for one commit only when the checkout has no identity. |
| `kit_revision` | yes | Full git SHA of the kit that was copied. |
| `optional_signals` | yes | `true` or `false` for each optional signal. Default `true`. |

`issue_tracker`, when set, has `trailer` (a format string containing `{id}`), `id_pattern`, `list_in_progress` (a command that prints the in-progress ids), and `data_dir` (a directory omitted from routing and from hunter slices). The commit-message hook writes the trailer only when that command returns exactly one id. Several ids are not guessed. One format a project may record is `Bead: {id}` with a `data_dir` for that tracker. Leave the slot `null` when the project has no tracker.

`gates.forbidden_imports` entries have `path`, `pattern`, and `message`. Copy only rules the target already enforces.

## Filled example

This example is a generic HTTP service called `sample-api`. It is not a template of any one product's stack.

```yaml
default_branch: main
tests:
  unit: "pytest -m unit"
  affected: "pytest --affected"
  full: "pytest"
accepted_decisions: docs/decisions
active_scope: docs/scope/current.md
critical_paths:
  - src/payments/
gates:
  forbidden_imports:
    - path: src/domain/
      pattern: "from app.http"
      message: "Domain modules must not import the HTTP layer"
  required_checks: []
issue_tracker: null
policy_version: policy-v1.2
product_root: src/
public_api_paths:
  - src/http/routes/
ci_config_paths:
  - ci/**/*.yml
ci_job_file: ci/review.yml
instruction_entry: AGENTS.md
optional_signals:
  critical-component-changed: true
  cross-component-changed: true
  large-change: true
large_change_lines: 80
signals:
  - id: auth-boundary-changed
    required: true
    hunters: [security]
  - id: secrets-changed
    required: true
    hunters: [security]
  - id: ci-config-changed
    required: true
    hunters: [security]
  - id: deployment-script-changed
    required: true
    hunters: [security]
  - id: database-migration-changed
    required: true
    hunters: [defects, security]
  - id: persistent-model-changed
    required: true
    hunters: [defects]
  - id: public-api-changed
    required: true
    hunters: [defects, security]
  - id: schema-or-event-changed
    required: true
    hunters: [defects]
  - id: dependency-manifest-changed
    required: true
    hunters: [defects, security]
  - id: trust-boundary-changed
    required: true
    hunters: [security]
  - id: critical-component-changed
    required: false
    hunters: [scope, defects, security]
  - id: cross-component-changed
    required: false
    hunters: [scope, defects, security]
  - id: large-change
    required: false
    hunters: [scope, defects, security]
hunter_paths:
  scope: [src/, docs/decisions/, docs/scope/current.md]
  defects: [src/, pyproject.toml]
  security: [src/, ci/]
  readability: [src/]
  maintainability: [src/]
defects_behavior_paths:
  - Dockerfile
  - compose.yml
  - compose.yaml
gate_paths:
  - adversary/
  - ci/review.yml
fixer_identity:
  name: Review fixer
  email: review-fixer@localhost
kit_revision: "<40-character git sha of the kit at install time>"
```

Dependency manifests in this example are `pyproject.toml` for the defects subject. The `dependency-manifest-changed` pattern in the routing policy still matches `package.json` and the lockfiles it names. Add a project-specific manifest name by recording it in `signals` only together with a new `policy_version`. Replacing `public_api_paths` and `ci_config_paths` is allowed at `policy-v1.2`.
