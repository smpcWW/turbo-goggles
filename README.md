---
title: Adversarial Agent Workflow Optimization
description: A portable kit that equips one project in a workspace with an adversarial review workflow.
type: readme
---

# Adversarial Agent Workflow Optimization

This repository is a kit. Copy it into a workspace so an assistant can read it. Copying does not install the workflow. Installation writes into one named project in that workspace and leaves every other project alone.

The kit is the upstream text. An install copies the role pack into the target and records the kit git revision. Later edits to a review belong in the target's project manifest, not in a rewritten role.

## Install

1. Copy this repository into the workspace as its own directory. Do not put it inside the project you want to equip.
2. Name the target project and the file that project's agent host loads at session start. When the project has no such file, the installer creates `AGENTS.md`.
3. Run the installer in [bootstrap/INSTALL.md](bootstrap/INSTALL.md). It writes only under the named project.

The same role files are what every agent host executes. A host binding only records where that host reads session instructions, where the installed roles live, how a review job passes one role path to an agent, and how the gate hook is installed (`gate_hook` in [bindings/binding.schema.md](bindings/binding.schema.md)).

Start with [AGENTS.md](AGENTS.md) when an assistant is reading this folder.
