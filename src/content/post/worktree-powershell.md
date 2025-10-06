---
title: "Worktree.ps: A Small PowerShell Helper for Git Worktrees"
description: "Parallelize safely with Git worktrees using a tiny PowerShell script that makes creating, switching, and cleaning worktrees effortless."
publishDate: "2025-10-06"
tags: ["git", "worktree", "powershell", "productivity", "devtools"]
---

### Context
Lately while using AI coding tools (Claude Code, Codex), I keep running into pockets of waiting. When several tasks pile up, I’m tempted to spin up another terminal to work in parallel—but I worry about conflicts.

That led me to Git’s worktree feature, which turns out to be perfect for this. Just create a new working directory for the same repo and you can safely run multiple tasks side by side; when you’re done, merge and move on.

### Worktree.ps
Using `git worktree` directly isn’t always convenient—there are a bunch of flags to remember. So, while learning the workflow, I used Codex to write a small PowerShell script that streamlines the experience:

https://github.com/littlesmilelove/worktree.ps

### Core Commands

```
wtx init                 # run inside the repo you want to manage
wtx add 3000             # creates ../<repo>.3000, branch feat/3000, optional env/deps/dev
wtx 3000                 # jump to that worktree (via shell hook)
wtx main                 # print main repo path; with shell hook, cd there
wtx rm 3000 --yes        # remove worktree and branch
wtx clean                # remove numerically named worktrees
```

This lets you easily create and manage multiple worktree-based working folders, and hop between them without friction—so you can stay focused on the coding itself.

For more details, check the project README. Enjoy coding!

