---
layout: post
title:  "GitHub commit hooks for better release notes"
date:   2019-11-16 16:32:36 -0500
categories: GitHub, Automation
---

I love using automation for improving the team processes.

If you need to have a clean commit history and be able to use it for release notes, you want to use GitHub local hook.
It gives ability to add ticket tracking system number(in my case JIRA) to every commit message automatically.

The trick is to name your branch as feature/PROJ-XXXX-branch-description and for every commit, `PROJ-XXXX` prepends
automatically to the message as `[PROJ-XXXX] commit description`

Find scripts and installation instruction in [https://github.com/PavelSusloparov/git-hook](https://github.com/PavelSusloparov/git-hook)
