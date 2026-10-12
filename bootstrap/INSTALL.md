---
title: Install the review workflow
description: Procedure an agent follows to equip one named project. It refuses to edit the kit or any other project.
type: agent
---

# Install the review workflow

You are installing the kit into one project. The kit stays the upstream text. You write a copy, a filled manifest, a binding, a short pointer, and a classify hook. You write only under the named project.

Read [../workflow/project-manifest.schema.md](../workflow/project-manifest.schema.md), [../workflow/routing-policy.md](../workflow/routing-policy.md), [../workflow/review-model.md](../workflow/review-model.md), and [../bindings/binding.schema.md](../bindings/binding.schema.md) before you write. Do not read [../bindings/reference-cursor.md](../bindings/reference-cursor.md) to succeed. That map is not an input.

## Stop before you write

Stop and report, without editing, when any of these are true:

- The operator has not named both the target project directory and the session entry file. Ask for the entry file. Ask which file this agent host loads at session start. Do not assume a host.
- The target's git toplevel is the kit's git toplevel, or either directory is inside the other.
- The target is not a git repository.
- The target already has `adversary/project-manifest.md`. Report the recorded `kit_revision` and stop. Do not upgrade unless the operator asks.
- `core.hooksPath` is already set to a path other than `adversary/hooks`. Report that path and stop. Do not replace another hook setup.
- You are about to edit the kit, the reference repository, or any project that is not the named target.

## 1. Discover the manifest inputs

Work inside the target toplevel. Record only what you can show from that repo. Leave a slot empty or `none` rather than copying another product's rules.

- `default_branch`: `git symbolic-ref --short refs/remotes/origin/HEAD` when that ref exists, otherwise ask.
- `tests.unit`, `tests.affected`, `tests.full`: commands the repo already documents. If you cannot find them, ask. If the project has no tests, leave the commands empty.
- `accepted_decisions`: a directory of accepted decisions if one exists, otherwise `none`.
- `active_scope`: the scope document the repo marks as current, otherwise `none`.
- `critical_paths`: prefixes the repo already treats as critical. If you cannot tell, use an empty list and say so in the install report.
- `gates`: forbidden imports and required checks the repo already enforces. Do not add checks from the kit's reference notes.
- `issue_tracker`: `null` unless the repo has a tracker. If it does, record `trailer`, `id_pattern`, `list_in_progress`, and `data_dir`. A `Bead: {id}` trailer is valid only when that tracker is the one this repo uses.
- `ci_job_file` and `ci_config_paths`: the review job file if you are adding one, and the CI config paths that exist. Empty `ci_config_paths` when the project has no CI config.
- `public_api_paths`: set this when the repo's public routes are not the default pattern in the routing policy. Otherwise omit it and the default pattern applies.
- `product_root`: the repo's source root, or `src/` when that is the source root.
- `instruction_entry`: the file the operator named. When they said the project has no entry file, you will create `AGENTS.md` and record that path.
- `policy_version`: `policy-v1.2`.
- `signals` and `optional_signals`: copy the table from the routing policy. Turn an optional signal off only when the operator says so.
- `hunter_paths`: the registry subjects in the routing policy, with this repo's product root, decisions path, scope path, and CI paths substituted.
- `defects_behavior_paths`: the default list in the routing policy, unless the operator replaces it.
- `gate_paths`: `adversary/` and the `ci_job_file` when that slot is set.
- `fixer_identity`: `Review fixer` and `review-fixer@localhost`.
- `kit_revision`: `git -C <kit-dir> rev-parse HEAD`.

## 2. Copy the role pack

Copy, unchanged:

- `<kit>/roles/` to `<target>/adversary/roles/`
- `<kit>/workflow/` to `<target>/adversary/workflow/`

Those relative links are part of the pack. Do not edit the copies to name the project, the host, or a vendor. Project values belong in the manifest.

## 3. Write the manifest and the binding

Write `<target>/adversary/project-manifest.md` with the slots you discovered. Include YAML front matter with `title`, `description`, and `type: plan` when you write Markdown.

Write `<target>/adversary/binding.md` with the fields in the binding schema:

- `session_entry` is the entry file you were given, or `AGENTS.md` if you create it in the next step.
- `roles_dir` is `adversary/roles`.
- `workflow_dir` is `adversary/workflow`.
- `manifest` is `adversary/project-manifest.md`.
- `invoke_role` is the host command the operator names. It must pass `{role}` as a path under `adversary/roles/`. If the operator has not named a command, ask. Do not invent a vendor CLI.
- `classify_hook.install` is `git config core.hooksPath adversary/hooks`.
- `classify_hook.entry` is `adversary/hooks/pre-commit`.
- `agent_home` states that the review job inherits a real home directory and must not set that directory empty.

## 4. Point the session entry at the implementer

If the operator said there is no entry file, create `<target>/AGENTS.md` and record `session_entry: AGENTS.md` in the binding. Say in your install report that you created it.

If the entry file exists, add this pointer once, and do not duplicate it:

```markdown
## Review

Before you commit, follow `adversary/roles/implementer.md`. Do not load `adversary/roles/report.md`, `adversary/roles/collate.md`, `adversary/roles/fixer.md`, or anything under `adversary/roles/hunters/` while implementing.
```

Do not paste hunter instructions into the entry file.

## 5. Install the classify hook

Write `<target>/adversary/hooks/pre-commit` so that it:

1. Refuses the commit when a staged path matches `gate_paths`. Tell the human to review that change and commit it by skipping the hook once. Do this before any bypass check. `ADVERSARY_BYPASS` must not get past this step.
2. Classifies the staged diff with [../workflow/routing-policy.md](../workflow/routing-policy.md) and the manifest. Implement that policy here. Do not copy a script from the reference repository.
3. Writes `.git/adversary-routing/<tree>.json` for `git write-tree`, prints the decision, warns on high risk, and appends the log line from the review model.
4. Exits success for an ordinary commit, including a high-risk commit.

When `issue_tracker` is not `null`, write `adversary/hooks/prepare-commit-msg` that appends the trailer only when `list_in_progress` returns exactly one id and the message does not already have that trailer. Several ids write nothing.

Run `git config core.hooksPath adversary/hooks` in the target. Make the hook files executable.

## 6. Report and stop

Tell the operator the target path, the entry file, whether you created `AGENTS.md`, the `kit_revision`, and any slot you left empty. Do not commit unless the operator asks. Do not push. Do not start a review. Do not edit the kit.
