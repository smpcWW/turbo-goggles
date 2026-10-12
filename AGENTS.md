---
title: Kit instructions
description: Session entry for an assistant that can read this kit and install it into one project.
type: agent
---

# Kit instructions

This directory is the Adversarial Agent Workflow Optimization kit. It is not an installed project. Do not treat these role files as the session rules of whatever repo you were already editing.

To equip one project, follow [bootstrap/INSTALL.md](bootstrap/INSTALL.md). Name the target project and the file that its agent host loads at session start before you write anything. Write only under that project. Do not edit this kit while installing. Do not edit any other project.

The workflow text is [workflow/review-model.md](workflow/review-model.md). Role files under [roles/](roles/) are the jobs a review runs. While you are implementing a change, load [roles/implementer.md](roles/implementer.md) only. Do not load [roles/report.md](roles/report.md), [roles/collate.md](roles/collate.md), [roles/fixer.md](roles/fixer.md), or anything under [roles/hunters/](roles/hunters/) on an implementation turn.

Hunter and collate roles run in the review job, on a fresh agent, after the change is committed. They must not inherit the implementer's reasoning.
