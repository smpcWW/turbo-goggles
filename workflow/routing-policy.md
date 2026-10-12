---
title: Routing policy
description: Deterministic policy-v1.2 signal table, required and optional signals, and hunter slice membership.
type: plan
---

# Routing policy

`decision_source` is `policy-v1.2`. The classifier does not call a model. The same numstat, name-status, patch, critical-path list, and manifest patterns always produce the same JSON.

A change is high risk when any signal matches. A new file is not a signal. A long diff is not required. A documentation-only diff that matches no signal stays low, and no hunter runs. Gates still run.

Path signals use the whole changed path. Content signals use added lines only (a diff line that starts with `+` and is not `+++`). Content signals skip `docs/` and skip omitted paths. The omitted path is the manifest `issue_tracker.data_dir` when that slot is set. Omitted paths raise no signal and appear in no slice.

## Decision

```json
{
  "risk": "high",
  "signals": ["database-migration-changed"],
  "selected_hunters": ["defects", "security"],
  "hunter_files": {
    "defects": ["db/migrations/20260101_add_status.sql"],
    "security": ["db/migrations/20260101_add_status.sql"]
  },
  "decision_source": "policy-v1.2"
}
```

`risk` is `high` when `signals` is non-empty, otherwise `low`. Signal order is the table order below. `selected_hunters` is the registry order `scope`, `defects`, `security`, `readability`, `maintainability`, keeping a hunter only when a selected signal lists it and the manifest still registers it. Optional signals that the manifest turns off are not selected.

## Signal table

`required` means a project that claims `policy-v1.2` keeps the signal. It may replace a path pattern in the manifest. It may not drop the signal. `optional` means the manifest may set the signal to off. The default for every optional signal is on.

| Signal | Use | Hunters |
| --- | --- | --- |
| `auth-boundary-changed` | required | `security` |
| `secrets-changed` | required | `security` |
| `ci-config-changed` | required | `security` |
| `deployment-script-changed` | required | `security` |
| `database-migration-changed` | required | `defects`, `security` |
| `persistent-model-changed` | required | `defects` |
| `public-api-changed` | required | `defects`, `security` |
| `schema-or-event-changed` | required | `defects` |
| `dependency-manifest-changed` | required | `defects`, `security` |
| `trust-boundary-changed` | required | `security` |
| `critical-component-changed` | optional | `scope`, `defects`, `security` |
| `cross-component-changed` | optional | `scope`, `defects`, `security` |
| `large-change` | optional | `scope`, `defects`, `security` |

Default expressions, copied as written. A manifest path list replaces the expression for `ci-config-changed`, `public-api-changed`, and `critical-component-changed` when that list is set. `ci-config-changed` has no built-in path: it matches only `ci_config_paths`.

```text
auth-boundary-changed path, case insensitive:
(^|/)(auth|session|permission|rbac|tenant)([./]|$)
auth-boundary-changed added line:
\b(authorize|authorise|authentication|permission|tenant_id|tenantId|isAdmin|requireRole)\b

secrets-changed path, case insensitive:
(^|/)\.env|(^|/)(credentials|secrets?)([./]|$)
secrets-changed added line, case insensitive:
\b(api[_-]?key|secret|password|token)\b\s*[:=]

deployment-script-changed path:
(^|/)(Dockerfile|docker-compose.*|helm/.+|.*\.(tf|bicep))$

database-migration-changed path:
(^|/)migrations?/|.*\.sql$
database-migration-changed added line, case insensitive:
\b(CREATE|ALTER|DROP)\s+TABLE\b

persistent-model-changed path, case insensitive:
^src/.+(schema|model)

public-api-changed path, unless public_api_paths is set:
^src/app/api/|(^|/)route\.(ts|tsx|js|jsx)$

schema-or-event-changed path, case insensitive:
(^|/)(openapi|dto)([./]|$)|(^|/)events/|.*\.proto$

dependency-manifest-changed path:
(^|/)(package\.json|package-lock\.json|pnpm-lock\.yaml|yarn\.lock)$

trust-boundary-changed added line:
\b(fetch|axios|child_process|spawn|exec)\s*\(
```

`persistent-model-changed` replaces the `src/` prefix when the manifest sets another product root. `critical-component-changed` matches `critical_paths`. An empty list never matches. `cross-component-changed` matches product files under two or more first-level directories of the product root. `large-change` matches 80 or more added plus deleted product lines. The manifest may set another threshold.

`large-change` and `cross-component-changed` are additional triggers. A one-line auth or CI-config change is high risk without them.

Product files are paths under the product root, default `src/`, excluding tests. A test path contains `/__tests__/`, starts with `__tests__/`, or has `.test.`, `.spec.`, or `.tests.` in the filename. Tests do not count toward `large-change` or `cross-component-changed`.

Name-status `R*` (rename) and `C*` (copy) use the destination path. A copy counts as an add. A rename counts as a modification.

## Hunter slices

For each selected hunter, `hunter_files` contains:

- changed paths that match that hunter's subject paths in the manifest, and
- changed paths that caused a signal which selects that hunter.

The defects slice also contains changed tests and any changed path listed in the manifest `defects_behavior_paths`. The reference defaults for that list are `Dockerfile`, `compose.yml`, `compose.yaml`, `next.config.ts`, `next.config.js`, `next.config.mjs`, and any filename starting with `docker-compose`.

Subject paths use the same prefix rule: a pattern ending in `/` is a prefix, otherwise it is an exact path.

Readability and maintainability stay registered with vote `advisory`. `policy-v1.2` does not select them. A project that wants them on a signal is changing the policy and must change `policy_version`.

## Registry votes

| Hunter | Vote | Default subject |
| --- | --- | --- |
| `scope` | blocking | Product root, the active-scope path, and the accepted-decisions path |
| `defects` | blocking | Product root and dependency manifests |
| `security` | blocking | Product root, `ci_config_paths`, and `scripts/` when that directory exists |
| `readability` | advisory | Product root |
| `maintainability` | advisory | Product root |

A docs-only change outside those subjects and outside every signal runs no hunter.
