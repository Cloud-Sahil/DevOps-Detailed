# 🚀 Git, GitHub & GitLab — Complete Interview Guide

> **Covers:** Basics · Branching · Advanced Commands · Workflows · GitHub/GitLab · Scenarios · Shell Scripts · Best Practices
> **Duplicates removed** — clean, modular, interview-ready.

---

## 📁 Table of Contents

| # | Section | Topics |
|---|---------|--------|
| 1 | [Basics & Fundamentals](#1-basics--fundamentals) | Git, GitHub, GitLab, VCS, Repos |
| 2 | [File States & Staging](#2-file-states--staging) | 3 states, HEAD, index, .gitignore |
| 3 | [Remote Operations](#3-remote-operations) | fetch, pull, push, remotes, upstream |
| 4 | [Branching & Merging](#4-branching--merging) | branches, merge, rebase, conflicts, stash |
| 5 | [Undoing & Recovery](#5-undoing--recovery) | reset, revert, reflog, cherry-pick |
| 6 | [Advanced Commands](#6-advanced-commands) | bisect, rebase -i, worktree, submodule |
| 7 | [Workflows & Collaboration](#7-workflows--collaboration) | Git Flow, GitHub Flow, PRs, forks |
| 8 | [GitHub & GitLab Specific](#8-github--gitlab-specific) | Actions, CI/CD, webhooks, runners |
| 9 | [Commit Best Practices](#9-commit-best-practices) | messages, amend, squash, hooks |
| 10 | [Security & Configuration](#10-security--configuration) | SSH, PAT, secrets, gc |
| 11 | [Scenario-Based Questions](#11-scenario-based-questions) | 35 real-world scenarios |
| 12 | [Shell Script Reference](#12-shell-script-reference) | 50+ commands, daily workflows |
| 13 | [Git Internals](#13-git-internals) | Object model, SHA, rerere, bare repos |

---

## 1. Basics & Fundamentals

### Q1. What is Git?
Git is a **Distributed Version Control System (DVCS)** that tracks changes in source code. Created by Linus Torvalds in 2005, it stores data as snapshots and allows multiple developers to collaborate, maintain history, and revert changes.

> 💡 Always mention "distributed" in interviews — it highlights offline capability.

---

### Q2. What is the difference between Git, GitHub, and GitLab?

| Feature | Git | GitHub | GitLab |
|---------|-----|--------|--------|
| Type | Local tool | Cloud service | Cloud / Self-hosted |
| Interface | CLI | GUI + CLI | GUI + CLI |
| CI/CD | ❌ | GitHub Actions | Built-in pipelines |
| Maintained by | Community | Microsoft | GitLab Inc. |
| Self-hostable | ✅ | ❌ | ✅ |

---

### Q3. Centralized vs Distributed VCS

| Feature | Centralized (SVN) | Distributed (Git) |
|---------|------------------|-------------------|
| Repository | Single central server | Every developer has full copy |
| Offline work | ❌ | ✅ |
| Speed | Slower (server round-trips) | Faster (local ops) |
| Branching | Heavy | Lightweight |
| Risk | Single point of failure | Redundant by nature |

---

### Q4. What is a Repository?
A repo is a directory tracked by Git, containing files + entire commit history in a `.git` folder.

- **Local repo** → on your machine (`git init`)
- **Remote repo** → on GitHub/GitLab for team collaboration

---

### Q5. What is a Bare Repository?
A bare repository has no working directory — contains only `.git` contents at root. Used as central server repos (GitHub stores bare repos).

```bash
git init --bare myrepo.git
```

---

### Q6. What is the difference between `git init` and `git clone`?

```bash
git init                                        # New empty repo
git clone https://github.com/user/repo.git      # Copy existing repo
```

- `git init` — starts fresh project
- `git clone` — copies all files, branches, and commit history

---

## 2. File States & Staging

### Q7. What are the 3 states of a file in Git?

```
Working Directory  →  Staging Area (Index)  →  Repository
   (Modified)            (Staged)               (Committed)
```

| State | Meaning |
|-------|---------|
| Modified | Changed but not staged |
| Staged | Marked for next commit |
| Committed | Safely stored in local repo |

---

### Q8. What is the Staging Area (Index)?
Intermediate area where changes are collected before committing. Allows fine-grained control — stage specific files or even specific lines.

```bash
git add file.txt       # Stage specific file
git add .              # Stage all changes
git add -p file.txt    # Stage specific hunks (patch mode)
git status             # See staged/unstaged files
```

---

### Q9. What is HEAD?
`HEAD` is a pointer to the current snapshot — usually the latest commit on the active branch. Switching branches moves `HEAD`.

**Detached HEAD** = `HEAD` points directly to a commit hash, not a branch.

```bash
git checkout abc1234       # Enters detached HEAD
git checkout -b new-branch # Save work by creating branch
```

---

### Q10. What is `.gitignore`?
Specifies files Git should NOT track — build artifacts, secrets, IDE files.

```gitignore
node_modules/
*.log
.env
dist/
target/
*.tfstate
.terraform/
*.pem
.DS_Store
```

---

## 3. Remote Operations

### Q11. What is the difference between `git fetch` and `git pull`?

| | git fetch | git pull |
|--|-----------|----------|
| Downloads changes | ✅ | ✅ |
| Auto-merges | ❌ | ✅ |
| Safe | Always | May cause conflicts |
| Equivalent | fetch only | fetch + merge |

```bash
git fetch origin            # Download, don't merge
git pull origin main        # Download + merge
git pull --rebase origin main  # Download + rebase (linear history)
```

---

### Q12. What is `git push`?
Uploads local commits to remote repository.

```bash
git push origin main              # Push to main
git push -u origin feature        # Push + set upstream
git push --force-with-lease       # Safe force push
git push --tags                   # Push all tags
```

---

### Q13. Managing Remotes

```bash
git remote -v                              # List remotes
git remote add origin <url>               # Add remote
git remote add upstream <original-url>    # Add upstream (fork)
git remote rename origin backup           # Rename
git remote remove origin                  # Remove
git remote set-url origin <new-url>       # Change URL
```

---

### Q14. What is an Upstream Repository?
In a forked workflow, **upstream** = the original repo you forked from.

```bash
git remote add upstream https://github.com/original/repo.git
git fetch upstream
git merge upstream/main    # Sync your fork
```

---

### Q15. SSH vs HTTPS for Git remotes

| | HTTPS | SSH |
|--|-------|-----|
| Auth | Username + PAT | SSH key pair |
| Setup | Easy | Requires key setup |
| Security | Good | Better |
| Firewall | Works through | May be blocked |

```bash
# HTTPS
git remote add origin https://github.com/user/repo.git

# SSH
git remote add origin git@github.com:user/repo.git

# Setup SSH keys
ssh-keygen -t ed25519 -C "your@email.com"
ssh-add ~/.ssh/id_ed25519
cat ~/.ssh/id_ed25519.pub    # Add to GitHub Settings → SSH Keys
ssh -T git@github.com        # Test connection
```

---

## 4. Branching & Merging

### Q16. What is a Branch?
A lightweight pointer to a commit. Allows diverged, independent development without affecting other branches.

```bash
git branch feature-login           # Create branch
git checkout -b feature-login      # Create + switch (old)
git switch -c feature-login        # Create + switch (new)
git branch -a                      # List all branches
git branch -d feature-login        # Delete merged branch
git branch -D feature-login        # Force delete
git push origin --delete feature   # Delete remote branch
```

---

### Q17. What is the difference between `git merge` and `git rebase`?

| | git merge | git rebase |
|--|-----------|-----------|
| History | Non-linear, creates merge commit | Linear, replays commits |
| Use case | Shared/team branches | Personal feature branches |
| Destructive | No | Rewrites history |

```bash
git merge feature-branch       # Merge (keeps all history)
git rebase main                # Rebase (linear, clean history)
git merge --no-ff feature      # Always create merge commit
```

> **Rule:** Use `merge` for shared branches; `rebase` to clean local feature branches.

---

### Q18. What is a Fast-Forward Merge?
Happens when target branch has no new commits — Git just moves the pointer forward, no merge commit created.

```bash
git merge feature-branch         # Fast-forward if no divergence
git merge --no-ff feature        # Force merge commit even if FF possible
```

---

### Q19. How do you resolve a merge conflict?

```bash
# 1. Trigger merge
git merge feature-branch

# 2. See conflicted files
git status

# 3. Open file — Git marks like this:
# <<<<<<< HEAD
# your changes
# =======
# incoming changes
# >>>>>>> feature-branch

# 4. Edit file to resolve, then:
git add resolved-file.js
git commit     # Complete the merge
git push origin main
```

---

### Q20. What is `git stash`?
Temporarily shelves uncommitted changes so you can switch tasks.

```bash
git stash                          # Save changes
git stash push -m "WIP login"      # Save with label
git stash list                     # View stashes
git stash pop                      # Apply + remove latest
git stash apply stash@{0}          # Apply but keep in list
git stash drop stash@{0}           # Delete specific stash
git stash clear                    # Delete all stashes
git stash branch new-branch        # Create branch from stash
```

**`pop` vs `apply`:** `pop` removes from stash list; `apply` keeps it.

---

### Q21. What is Detached HEAD?
`HEAD` points to a commit hash instead of a branch. Commits made here may be lost when switching.

```bash
git checkout abc1234              # Detached HEAD
git checkout -b save-work         # Rescue by creating branch
```

---

## 5. Undoing & Recovery

### Q22. `git reset` vs `git revert` vs `git checkout`

| Command | Effect | History | Use When |
|---------|--------|---------|----------|
| `git reset` | Moves HEAD, modifies index | Rewrites | Local/private branch |
| `git revert` | New commit that undoes | Safe, keeps | Shared branch |
| `git checkout` | Switch branch/restore file | No change | Exploring |

---

### Q23. Types of `git reset`

```bash
git reset --soft HEAD~1    # Undo commit, keep staged
git reset --mixed HEAD~1   # Undo commit, keep unstaged (default)
git reset --hard HEAD~1    # Undo commit, DISCARD all changes ⚠️
git reset --hard HEAD~3    # Undo last 3 commits
```

---

### Q24. What is `git reflog`?
Records every HEAD movement — safety net for recovering "lost" commits.

```bash
git reflog                  # See all history
git checkout HEAD@{3}       # Go to specific state
git branch recovered abc123 # Recover deleted branch
```

> Commits stay in reflog ~90 days before garbage collection.

---

### Q25. What is `git cherry-pick`?
Applies changes from a specific commit to another branch — without merging the whole branch.

```bash
git cherry-pick abc1234                 # Single commit
git cherry-pick abc123 def456           # Multiple commits
git cherry-pick feature~2..feature      # Range
```

---

### Q26. `git clean` — Remove Untracked Files

```bash
git clean -n      # Dry run — show what would be deleted
git clean -f      # Delete untracked files
git clean -fd     # Delete untracked files + directories
git clean -fdx    # Also delete .gitignore'd files
```

---

### Q27. Remove a File from Git Tracking (Keep Locally)

```bash
git rm --cached secrets.env
echo "secrets.env" >> .gitignore
git commit -m "stop tracking secrets.env"
```

---

## 6. Advanced Commands

### Q28. What is `git rebase -i` (Interactive Rebase)?

```bash
git rebase -i HEAD~5    # Edit last 5 commits
```

Actions in editor:
- `pick` — keep commit
- `squash` / `s` — combine with previous
- `reword` / `r` — change message
- `drop` / `d` — delete commit
- `edit` / `e` — pause to amend

---

### Q29. What is `git bisect`?
Binary search to find which commit introduced a bug. 1000 commits → found in ~10 steps.

```bash
git bisect start
git bisect bad                  # Current is broken
git bisect good v1.0.0          # v1.0 was working
# Test each midpoint, mark good/bad
git bisect good                 # or git bisect bad
git bisect reset                # End session

# Automate with test script
git bisect run npm test
```

---

### Q30. What is `git tag`?
Permanent markers for release points (unlike branches, tags don't move).

```bash
git tag v1.0.0                           # Lightweight tag
git tag -a v1.0.0 -m "Release 1.0.0"    # Annotated tag
git push origin v1.0.0                   # Push tag
git push origin --tags                   # Push all tags
git tag -d v1.0.0                        # Delete local tag
git push --delete origin v1.0.0          # Delete remote tag
```

**Semantic Versioning (SemVer):** `MAJOR.MINOR.PATCH`
- MAJOR = breaking changes
- MINOR = new features (backward-compatible)
- PATCH = bug fixes

---

### Q31. `git log` — Useful Formats

```bash
git log --oneline                            # Compact
git log --graph --all                        # ASCII branch graph
git log --oneline --graph --decorate --all   # Popular combo
git log -p                                   # Show diff per commit
git log --author="John"                      # Filter by author
git log --since="2 weeks ago"                # Filter by date
git log --grep="fix"                         # Search messages
git log main..feature --oneline              # Commits in feature, not main
```

---

### Q32. `git diff` Options

```bash
git diff                      # Working dir vs staging
git diff --staged             # Staging vs last commit
git diff HEAD                 # Working dir vs last commit
git diff branch1..branch2     # Between two branches
git diff abc123 def456        # Between two commits
```

---

### Q33. What is `git blame`?
Shows who last changed each line and in which commit.

```bash
git blame filename.js
git blame -L 10,20 filename.js    # Specific lines
git log -L 47,47:payment.js       # Full history of a line
```

---

### Q34. What is `git worktree`?
Check out multiple branches simultaneously in separate directories.

```bash
git worktree add ../hotfix hotfix-branch    # New dir for branch
git worktree list
git worktree remove ../hotfix
```

---

### Q35. What is `git submodule`?
Embed one Git repo inside another (for shared libraries).

```bash
git submodule add https://github.com/lib/repo libs/repo
git submodule update --init --recursive
```

---

## 7. Workflows & Collaboration

### Q36. Git Flow vs GitHub Flow

| | Git Flow | GitHub Flow |
|--|----------|-------------|
| Branches | main, develop, feature, release, hotfix | main + feature only |
| Complexity | High | Simple |
| Best for | Versioned releases | Continuous deployment |
| PR required | Yes | Yes |

**Git Flow branches:**
- `main` — production-ready
- `develop` — integration branch
- `feature/*` — new features
- `release/*` — release prep
- `hotfix/*` — emergency fixes

---

### Q37. What is a Pull Request (PR)?
Notifies team about pushed changes for review before merging. Central to collaborative workflows.

**PR Steps:**
```
1. Create feature branch
2. Push changes
3. Open PR on GitHub
4. Team reviews, comments, approves
5. CI/CD checks pass
6. Merge to main
```

---

### Q38. Fork vs Branch

| | Fork | Branch |
|--|------|--------|
| Location | Your GitHub account | Same repository |
| Access | Don't need write access | Need repo access |
| Use case | Open source contribution | Team collaboration |

---

### Q39. What are Protected Branches?
Prevent unauthorized changes to critical branches.

Rules:
- Require PR reviews before merging
- Require CI status checks to pass
- Restrict direct push
- Prevent force push / deletion

---

### Q40. What is a CODEOWNERS File?
Auto-assigns reviewers when their files are modified in a PR.

```
# .github/CODEOWNERS
*.js          @frontend-team
/src/api/     @backend-team
README.md     @docs-team
```

---

## 8. GitHub & GitLab Specific

### Q41. What is GitHub Actions?
Built-in CI/CD platform. Workflows defined in `.github/workflows/*.yml`.

```yaml
name: CI
on: [push]
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - run: npm test
```

---

### Q42. What is GitLab CI/CD?
Defined in `.gitlab-ci.yml`. GitLab Runners execute jobs in stages.

```yaml
stages:
  - build
  - test
  - deploy

test:
  stage: test
  script:
    - npm test
```

---

### Q43. What is a Webhook?
HTTP callback sent to a URL when events happen (push, PR, issue). Used to trigger Jenkins, Slack bots, or deployment tools.

---

### Q44. What is a GitLab Runner?
Agent that executes CI/CD jobs. Types:
- **Shared** — available to all projects
- **Group** — scoped to project group
- **Specific** — registered for single project

Executors: shell, Docker, Kubernetes.

---

### Q45. What is GitHub Packages?
Package hosting (npm, Maven, Docker, NuGet) integrated with GitHub — source code and packages in one place.

---

### Q46. What is GitLab Container Registry?
Built-in Docker image registry in GitLab. CI/CD pipelines push images; deployments pull — no need for Docker Hub.

---

## 9. Commit Best Practices

### Q47. What Makes a Good Commit Message?

```
<type>(<scope>): <summary>     ← 50 chars max

<explanation>                  ← wrap at 72 chars

Closes #89                     ← issue reference
```

**Conventional Commits types:**

| Type | Use for |
|------|---------|
| `feat` | New feature |
| `fix` | Bug fix |
| `docs` | Documentation only |
| `refactor` | Code restructure |
| `test` | Adding tests |
| `chore` | Build, config changes |

**Example:**
```
feat(auth): add JWT token refresh logic

Implemented token refresh endpoint to handle expiry gracefully.
Users are now automatically re-authenticated.

Closes #89
```

---

### Q48. How to Amend Last Commit

```bash
git add missed-file.js
git commit --amend                      # Edit message
git commit --amend -m "New message"     # Change message directly
git commit --amend --no-edit            # Keep message, add file
```

⚠️ Never amend commits already pushed to shared branch.

---

### Q49. How to Squash Commits

```bash
# Interactive rebase
git rebase -i HEAD~5
# Mark commits as 'squash'

# Soft reset + single commit
git reset --soft main
git commit -m "feat: clean single commit"
```

---

### Q50. What are Git Hooks?
Scripts that run automatically before/after Git events. Stored in `.git/hooks/`.

| Hook | Trigger |
|------|---------|
| `pre-commit` | Before commit created |
| `commit-msg` | Validate commit message |
| `pre-push` | Before pushing |
| `post-merge` | After merge |

Use **Husky** to share hooks across teams via `package.json`.

---

## 10. Security & Configuration

### Q51. Configure Git Globally

```bash
git config --global user.name "Your Name"
git config --global user.email "you@example.com"
git config --global core.editor "code --wait"
git config --global init.defaultBranch main
git config --global pull.rebase true
git config --global fetch.prune true
git config --global alias.lg "log --oneline --graph --all"
git config --list
```

Config levels: `--system` > `--global` > `--local`

---

### Q52. What is a Personal Access Token (PAT)?
Token used instead of password for HTTPS Git operations and API auth.

- Can be scoped to specific permissions
- Can be revoked individually
- Can have expiration dates

---

### Q53. How to Remove Sensitive Data Committed to Git

```bash
# 1. IMMEDIATELY rotate/revoke the exposed credential

# 2. Remove using BFG Repo Cleaner (recommended)
bfg --delete-files secrets.env my-repo.git
git reflog expire --expire=now --all
git gc --prune=now --aggressive
git push --force --all

# 3. Or use git filter-repo (modern)
git filter-repo --path secrets.env --invert-paths
git push origin --force --all

# 4. All collaborators must re-clone
```

---

### Q54. What is `git gc` (Garbage Collection)?
Housekeeping to optimize repository — compresses objects, removes unreachable objects, prunes old reflog entries.

```bash
git gc                   # Standard cleanup
git gc --prune=now       # Remove unreachable objects immediately
```

---

## 11. Scenario-Based Questions

### Scenario 1: Committed to Wrong Branch (Not Pushed)

```bash
git branch feature/my-feature      # Create correct branch at current state
git reset --hard HEAD~1            # Reset main back
git checkout feature/my-feature    # Switch to feature
```

---

### Scenario 2: Pushed Sensitive API Key

```bash
# 1. REVOKE the key immediately (assume compromised)
# 2. Remove from history
bfg --replace-text passwords.txt my-repo.git
git push --force --all
# 3. Add to .gitignore
# 4. Notify all collaborators to re-clone
```

---

### Scenario 3: Undo Last 3 Commits (Not Pushed)

```bash
git reset --mixed HEAD~3    # Keep changes unstaged
git reset --soft HEAD~3     # Keep changes staged
git reset --hard HEAD~3     # Discard everything ⚠️
```

---

### Scenario 4: Undo Pushed Commit on Shared Branch

```bash
git revert <bad-commit-hash>   # New commit that undoes it
git push origin main           # Safe — no history rewrite
```

---

### Scenario 5: Urgent Hotfix While Mid-Feature

```bash
git stash push -m "WIP: feature login form"
git checkout main
git checkout -b hotfix/critical-bug
# fix bug
git commit -m "fix: resolve payment null pointer"
git checkout main
git merge --no-ff hotfix/critical-bug
git push origin main
git branch -d hotfix/critical-bug
git checkout feature/login-form
git stash pop
```

---

### Scenario 6: Recover Deleted Branch

```bash
git reflog                              # Find last commit SHA
git checkout -b recovered-feature abc1234
```

---

### Scenario 7: Feature Branch Way Behind Main

```bash
git checkout feature/my-feature
git fetch origin
git rebase origin/main                  # Linear history
git push origin feature/my-feature --force-with-lease
```

---

### Scenario 8: Cherry-Pick Bug Fix to Release Branch

```bash
git log develop --oneline               # Find commit hash
git checkout release/v2.1
git cherry-pick abc1234                 # Apply just that commit
git push origin release/v2.1
```

---

### Scenario 9: Find Commit That Introduced a Bug

```bash
git bisect start
git bisect bad                          # Current is broken
git bisect good v2.0.0                  # Known good version
# Test each checkout, mark good/bad
git bisect reset                        # End session
```

---

### Scenario 10: Squash All WIP Commits Before PR

```bash
git reset --soft main
git commit -m "feat(payments): add Stripe checkout integration"
git push --force-with-lease origin feature/my-feature
```

---

### Scenario 11: Git Push Rejected — Non-Fast-Forward

```bash
# Someone pushed while you were working
git pull --rebase origin main
git push origin main
```

---

### Scenario 12: Rewrite Commit Message Already Pushed

```bash
# Most recent commit
git commit --amend -m "fix(auth): correct password validation"
git push --force-with-lease origin feature-branch

# Older commit
git rebase -i HEAD~3
# Change 'pick' to 'reword' for target commit
git push --force-with-lease
```

---

### Scenario 13: Stage Only Some Changes in a File

```bash
git add -p utils.js    # Patch mode — choose hunks interactively
# y = stage hunk
# n = skip
# s = split into smaller
# e = edit manually
```

---

### Scenario 14: Clean Up Merged Local Branches

```bash
git branch --merged main | grep -v "^\* main" | xargs git branch -d
git fetch --prune    # Remove remote-tracking refs that no longer exist
```

---

### Scenario 15: Check If Commit Is in a Branch

```bash
git branch --contains abc1234          # Which branches have this commit
git merge-base --is-ancestor abc1234 release/v3.0
echo $?    # 0 = included ✅, 1 = not included ❌
```

---

## 12. Shell Script Reference

### Daily Git Flow

```bash
#!/bin/bash
# Daily workflow
git status
git add .
git commit -m "feat: add new feature"
git push origin main

# Shortcut for tracked files
git commit -am "fix: quick patch"
```

---

### Complete Command Reference (50+)

```bash
#!/bin/bash

# === SETUP ===
git init                             # New repo
git clone <url>                      # Clone remote
git config --global user.name "Name"
git config --global user.email "email"

# === STAGING & COMMITTING ===
git status                           # Check status
git add .                            # Stage all
git add -p <file>                    # Stage partial (patch)
git commit -m "message"              # Commit
git commit -am "message"             # Stage + commit tracked

# === BRANCHING ===
git branch                           # List branches
git branch -a                        # All branches
git switch -c feature-branch         # Create + switch
git checkout -b feature-branch       # Create + switch (old)
git merge <branch>                   # Merge
git rebase <branch>                  # Rebase
git branch -d <branch>               # Delete merged
git branch -D <branch>               # Force delete

# === REMOTE ===
git remote -v                        # Show remotes
git remote add origin <url>          # Add remote
git fetch origin                     # Fetch all
git pull origin main                 # Pull
git push origin main                 # Push
git push -u origin feature           # Push + set upstream
git push --force-with-lease          # Safe force push

# === STASH ===
git stash                            # Stash changes
git stash pop                        # Restore + remove
git stash list                       # List stashes
git stash drop stash@{0}             # Delete stash

# === UNDO ===
git restore <file>                   # Restore file
git restore --staged <file>          # Unstage
git reset --soft HEAD~1              # Undo commit, keep staged
git reset --mixed HEAD~1             # Undo commit, keep files
git reset --hard HEAD~1              # Undo + discard ⚠️
git revert <commit>                  # Safe undo (new commit)
git cherry-pick <commit>             # Apply specific commit
git reflog                           # Recovery history

# === INSPECTION ===
git log --oneline --graph --all      # Visual history
git diff                             # Working vs staged
git diff --staged                    # Staged vs committed
git show <commit>                    # Commit details
git blame <file>                     # Line-by-line author
git bisect start                     # Start bug hunt

# === TAGS ===
git tag -a v1.0.0 -m "Release"      # Annotated tag
git push origin v1.0.0              # Push tag
git push --tags                      # All tags
git tag -d v1.0.0                    # Delete local tag

# === ADVANCED ===
git rebase -i HEAD~5                 # Interactive rebase
git clean -fd                        # Remove untracked
git gc                               # Optimize repo
git archive --format=zip HEAD > r.zip  # Export repo
git rev-parse HEAD                   # Get commit hash
git submodule add <url>              # Add submodule
git worktree add ../branch branch    # Linked worktree
git sparse-checkout set src/         # Partial checkout
```

---

## 13. Git Internals

### Q55. How Does Git Store Data?
All data stored as objects in `.git/objects/` — content-addressed by SHA hash.

| Object | Description |
|--------|-------------|
| **Blob** | Raw file content |
| **Tree** | Directory structure (refs blobs + trees) |
| **Commit** | Points to tree + metadata + parent commit |
| **Tag** | Named commit reference with message |

```bash
git cat-file -t <hash>    # Show object type
git cat-file -p <hash>    # Show object content
```

---

### Q56. What is `git rerere`?
**Re**use **Re**corded **Re**solution — remembers how you resolved conflicts and auto-applies next time.

```bash
git config rerere.enabled true
```

---

### Q57. What is `ORIG_HEAD`, `FETCH_HEAD`, `MERGE_HEAD`?

| Ref | Set When |
|-----|----------|
| `ORIG_HEAD` | Before merge/rebase/reset — lets you undo |
| `FETCH_HEAD` | Records last fetched branch |
| `MERGE_HEAD` | The commit being merged (in-progress merge) |

```bash
git reset --hard ORIG_HEAD    # Undo last merge/rebase
```

---

### Q58. `git switch` vs `git checkout`
`git switch` (Git 2.23+) is more focused — only for branch operations.

```bash
# Old way
git checkout feature-branch
git checkout -b new-branch
git checkout -- file.txt        # Restore file

# New way (clearer intent)
git switch feature-branch
git switch -c new-branch
git restore file.txt            # Restore file
```

---

### Q59. What is `git describe`?
Human-readable name for a commit based on nearest tag. Useful for build versioning.

```bash
git describe                  # e.g., v1.2.0-14-gabcdef7
git describe --abbrev=0       # Just tag name: v1.2.0
```

---

### Q60. Git Best Practices (Team Environment)

| Practice | Why |
|----------|-----|
| ✅ Commit often, small | Easy to review and revert |
| ✅ Meaningful commit messages | Conventional Commits standard |
| ✅ Feature branches always | Never commit directly to main |
| ✅ Short-lived branches | Less drift, less painful merges |
| ✅ Review via PR/MR | Never self-merge important changes |
| ✅ Protect main branches | Enforce reviews + CI checks |
| ✅ `.gitignore` always | No secrets or build artifacts |
| ✅ Tag every release | SemVer tags for traceability |
| ✅ `rebase` before merging | Clean linear history |
| ❌ Never commit secrets | Use env vars + secret managers |
| ❌ Never `--force` on shared | Use `--force-with-lease` |
| ❌ Never skip tests | Prod bug costs 10x more |

---

## 🧠 Quick Memory Tricks

```
3 States:    Modified → Staged → Committed
Git Flow:    main → develop → feature → release → hotfix
Reset:       soft (staged) | mixed (unstaged) | hard (gone)
Revert vs Reset: revert = safe (new commit) | reset = dangerous (rewrites)
PR Flow:     branch → commit → push → PR → review → merge
SSH Setup:   keygen → ssh-add → copy pub → paste GitHub → test
```

---

*© Git, GitHub & GitLab Complete Interview Guide — 60 Q&A + 35 Scenarios + 50+ Commands*
*Duplicates removed · Modular sections · Interview-ready*

---

## 11B. Additional Scenario-Based Questions (30–90)

---

### Scenario 16: You Need to Undo a `git rebase` That Went Wrong

**Problem:** You rebased your feature branch onto main but now the history looks wrong and you want to go back.

```bash
# Find the pre-rebase state in reflog
git reflog
# Look for: HEAD@{n}: rebase finished / rebase -i (finished)

# Reset back to before rebase
git reset --hard HEAD@{5}    # replace 5 with correct index

# Verify
git log --oneline
```

> `git reflog` saves you every time. Every rebase, reset, merge is recorded.

---

### Scenario 17: Abort a Rebase in Progress

**Problem:** You started `git rebase -i HEAD~10` and now you have massive conflicts on commit 4. You want to cancel the whole thing.

```bash
git rebase --abort        # Go back to exact pre-rebase state

# OR: skip just the problematic commit
git rebase --skip

# OR: resolve conflict and continue
git add <resolved-file>
git rebase --continue
```

---

### Scenario 18: Push to Two Remotes Simultaneously

**Problem:** Your company wants code mirrored to both GitHub and GitLab.

```bash
# Add both remotes
git remote add origin   git@github.com:user/repo.git
git remote add gitlab   git@gitlab.com:user/repo.git

# Push to both manually
git push origin main
git push gitlab main

# OR: configure one remote to push to multiple URLs
git remote add all git@github.com:user/repo.git
git remote set-url --add all git@gitlab.com:user/repo.git
git push all main    # pushes to both in one command
```

---

### Scenario 19: Rename the Default Branch from `master` to `main`

**Problem:** Your old repo uses `master`. Team wants to rename it to `main`.

```bash
# Step 1: Rename locally
git branch -m master main

# Step 2: Push new branch to remote
git push origin main

# Step 3: Set as default on GitHub
# GitHub → Settings → Branches → Default branch → main

# Step 4: Delete old master on remote
git push origin --delete master

# Step 5: Update local tracking
git branch -u origin/main main
git remote set-head origin main
```

---

### Scenario 20: You Committed a 500MB File — Push is Rejected

**Problem:** `git push` fails because a large binary file was committed.

```bash
# Step 1: Remove the large file from last commit
git rm --cached large-file.zip
git commit --amend --no-edit

# Step 2: If already in history, remove completely
git filter-repo --path large-file.zip --invert-paths

# Step 3: Force push
git push origin main --force

# Step 4: Prevent recurrence
echo "*.zip" >> .gitignore
echo "*.iso" >> .gitignore

# Step 5: Use Git LFS for large files in future
git lfs track "*.zip"
git add .gitattributes
```

---

### Scenario 21: Apply Changes from One Repo to Another

**Problem:** You have a fix in `repo-A` that needs to go into `repo-B`.

```bash
# Option A: Generate patch from repo-A
cd repo-A
git format-patch -1 abc1234        # Creates 0001-fix-commit.patch

# Apply to repo-B
cd repo-B
git am 0001-fix-commit.patch       # Applies with original author

# Option B: Direct diff apply
cd repo-A
git diff HEAD~1 HEAD > fix.patch
cd repo-B
git apply fix.patch
```

---

### Scenario 22: Sync a Fork with the Original Repo

**Problem:** Your fork is 30 commits behind the original repo. You need to sync it.

```bash
# Step 1: Add upstream (one-time setup)
git remote add upstream https://github.com/original/repo.git

# Step 2: Fetch upstream changes
git fetch upstream

# Step 3: Merge into your main
git checkout main
git merge upstream/main

# Step 4: Push synced fork to your GitHub
git push origin main
```

---

### Scenario 23: View All Changes Made by One Developer

**Problem:** Your manager wants to audit all changes made by "John" in the last month.

```bash
# All commits by John
git log --author="John" --oneline

# Last 30 days
git log --author="John" --since="30 days ago" --oneline

# Full diffs by John
git log --author="John" -p

# Files John changed
git log --author="John" --name-only --oneline

# Number of lines added/removed
git log --author="John" --stat
```

---

### Scenario 24: Check Out a PR Locally for Testing

**Problem:** You want to test PR #42 locally before approving it, but don't know the branch name.

```bash
# GitHub PR
git fetch origin pull/42/head:pr-42
git checkout pr-42

# GitLab MR
git fetch origin merge-requests/42/head:mr-42
git checkout mr-42

# After testing
git checkout main
git branch -d pr-42
```

---

### Scenario 25: Your Commit Has Wrong Author Name/Email

**Problem:** You committed with wrong `user.name` or `user.email` (e.g., root@ec2 instead of your real name).

```bash
# Fix last commit only
git commit --amend --author="Sahil <sahil@email.com>" --no-edit

# Fix multiple commits (all with wrong author)
git filter-repo --commit-callback '
  if commit.author_email == b"root@ec2":
    commit.author_name = b"Sahil"
    commit.author_email = b"sahil@email.com"
'
git push --force
```

---

### Scenario 26: Git Repo Corrupted — Object Errors

**Problem:** You see errors like `error: object file is empty` or `corrupt loose object`.

```bash
# Step 1: Check repo integrity
git fsck --full

# Step 2: Fetch a fresh copy of objects from remote
git fetch origin

# Step 3: Aggressive cleanup
git gc --aggressive --prune=now

# Step 4: If too broken, re-clone
git clone <remote-url> repo-clean
# Copy your uncommitted work manually
cp -r repo-broken/src/ repo-clean/src/
```

---

### Scenario 27: Find All Files Changed Between Two Releases

**Problem:** QA wants a list of every file changed between `v1.0` and `v2.0`.

```bash
# List changed files only
git diff v1.0..v2.0 --name-only

# With change type (Added/Modified/Deleted)
git diff v1.0..v2.0 --name-status

# Summary stats
git diff v1.0..v2.0 --stat

# Commits between the two tags
git log v1.0..v2.0 --oneline
```

---

### Scenario 28: Someone Force-Pushed and Overwrote Your Commits

**Problem:** A teammate force-pushed to `feature-branch` and your 3 commits are now missing from remote.

```bash
# Step 1: Your commits still exist locally — check reflog
git reflog

# Step 2: Find your lost commits
git log --oneline    # Your local still has them

# Step 3: Push your version back (if you own the branch)
git push origin feature-branch --force-with-lease

# Prevention: Always use --force-with-lease instead of --force
# It fails if someone else has pushed to the branch
```

---

### Scenario 29: Create a Release Branch and Hotfix Flow

**Problem:** You're on `v2.0` release branch and a critical bug needs to be fixed and deployed to production immediately.

```bash
# Step 1: Create hotfix from main/release
git checkout main
git checkout -b hotfix/v2.0.1

# Step 2: Fix the bug, commit
git commit -m "fix: critical auth bypass vulnerability"

# Step 3: Merge to main
git checkout main
git merge --no-ff hotfix/v2.0.1
git tag -a v2.0.1 -m "Hotfix v2.0.1"

# Step 4: Merge to develop too (don't lose the fix)
git checkout develop
git merge --no-ff hotfix/v2.0.1

# Step 5: Delete hotfix branch
git branch -d hotfix/v2.0.1
git push origin main develop --tags
```

---

### Scenario 30: Set Up Git in a Fresh EC2 Server for CI/CD

**Problem:** New EC2 instance — set up Git, clone repo, configure SSH for automated deployments.

```bash
# Step 1: Install Git
sudo apt update -y
sudo apt install git -y
git --version

# Step 2: Configure identity
git config --global user.name "deploy-bot"
git config --global user.email "deploy@company.com"

# Step 3: Generate deploy SSH key
ssh-keygen -t ed25519 -C "ec2-deploy" -f ~/.ssh/deploy_key -N ""

# Step 4: Add public key to GitHub (Deploy Keys → repo settings)
cat ~/.ssh/deploy_key.pub

# Step 5: Configure SSH to use this key
cat >> ~/.ssh/config << EOF
Host github.com
  IdentityFile ~/.ssh/deploy_key
  StrictHostKeyChecking no
EOF

# Step 6: Clone repo
git clone git@github.com:user/repo.git
cd repo

# Step 7: Create deploy script
cat > deploy.sh << 'EOF'
#!/bin/bash
git pull origin main
docker build -t myapp:latest .
docker run -d -p 80:80 myapp:latest
EOF
chmod +x deploy.sh
```

---

### Scenario 31: Diff Between What You Have and What's on Remote

**Problem:** Before pulling, you want to see what changes are on remote without downloading them yet.

```bash
# Step 1: Fetch remote (downloads but doesn't merge)
git fetch origin

# Step 2: See commits on remote not on local
git log HEAD..origin/main --oneline

# Step 3: See file changes
git diff HEAD origin/main

# Step 4: See only changed file names
git diff HEAD origin/main --name-only

# Now decide: merge or rebase?
git pull origin main            # merge
git pull --rebase origin main   # rebase
```

---

### Scenario 32: Export Specific Folder from Repo (No .git History)

**Problem:** Client wants only the `src/frontend/` folder as a zip, without any git history.

```bash
# Export specific folder as zip
git archive HEAD src/frontend/ --format=zip -o frontend.zip

# Export entire repo (no .git folder)
git archive HEAD --format=zip -o project.zip

# Export specific tag
git archive v2.0.0 --format=tar.gz -o release-v2.0.tar.gz
```

---

### Scenario 33: You Need to See the Exact Contents of a File at a Past Commit

**Problem:** A file `config.yaml` was working 3 commits ago. You want to see its content then.

```bash
# View file at specific commit (don't checkout branch)
git show HEAD~3:config/config.yaml

# View file at specific tag
git show v1.5.0:config/config.yaml

# View file at specific hash
git show abc1234:config/config.yaml

# Restore file from past commit into working directory
git checkout HEAD~3 -- config/config.yaml
```

---

### Scenario 34: Prevent Accidental Push to Main

**Problem:** Developers keep accidentally pushing directly to `main`. You want to block it locally.

```bash
# Create a pre-push hook
cat > .git/hooks/pre-push << 'EOF'
#!/bin/bash
branch=$(git rev-parse --abbrev-ref HEAD)
if [ "$branch" = "main" ]; then
  echo "❌ Direct push to main is not allowed!"
  echo "   Create a feature branch and open a PR."
  exit 1
fi
EOF
chmod +x .git/hooks/pre-push

# Share across team using Husky (Node.js projects)
npx husky add .husky/pre-push 'bash scripts/check-branch.sh'
```

---

### Scenario 35: Commit Only Part of a File's Changes

**Problem:** `utils.js` has two unrelated changes — a bug fix on line 10 and a new feature on line 80. You want separate commits.

```bash
# Patch mode — interactive hunk selection
git add -p utils.js

# For each hunk:
# y = stage this hunk
# n = skip this hunk
# s = split into smaller hunks
# e = manually edit the hunk

# Commit just the bug fix
git commit -m "fix: correct email validation regex"

# Then stage + commit the feature
git add utils.js
git commit -m "feat: add phone number formatter"
```

---

### Scenario 36: Maintain Long-Running Feature Branch with Frequent Main Updates

**Problem:** Your feature takes 3 weeks. Main gets 50+ commits. How do you stay updated without chaos?

```bash
# Strategy: Regular rebase onto main (every 2-3 days)
git fetch origin
git rebase origin/main

# If conflicts arise, resolve one by one
git add <resolved>
git rebase --continue

# Push updated branch
git push origin feature/long-feature --force-with-lease

# Alternative: Periodic merge (less clean but safer for shared branch)
git merge origin/main
git push origin feature/long-feature
```

---

### Scenario 37: Check Git Repository Health and Size

**Problem:** Repo clone is getting very slow. You want to check size and health.

```bash
# Check repo size on disk
du -sh .git/

# Count objects
git count-objects -vH

# Find large files in history
git rev-list --objects --all |
  git cat-file --batch-check='%(objecttype) %(objectname) %(objectsize) %(rest)' |
  sort -k3 -rn | head -20

# Optimize repo
git gc --aggressive --prune=now

# Remove large file from all history
git filter-repo --strip-blobs-bigger-than 50M
```

---

### Scenario 38: Handle Multiple GitHub Accounts on Same Machine

**Problem:** You have personal and work GitHub accounts on the same laptop.

```bash
# Generate separate SSH keys
ssh-keygen -t ed25519 -f ~/.ssh/personal_key -C "personal@email.com"
ssh-keygen -t ed25519 -f ~/.ssh/work_key    -C "work@company.com"

# Configure SSH to use different keys per host
cat >> ~/.ssh/config << EOF
# Personal GitHub
Host github-personal
  HostName github.com
  User git
  IdentityFile ~/.ssh/personal_key

# Work GitHub
Host github-work
  HostName github.com
  User git
  IdentityFile ~/.ssh/work_key
EOF

# Clone using specific account
git clone git@github-personal:personal-user/repo.git
git clone git@github-work:work-org/repo.git

# Set per-repo identity
cd work-repo
git config user.name "Work Name"
git config user.email "work@company.com"
```

---

### Scenario 39: Revert a Specific File to an Older Version

**Problem:** `app.js` was working fine 5 commits ago. Only revert that one file, not the whole repo.

```bash
# Find the commit where it was working
git log --oneline -- app.js

# Restore specific file from that commit
git checkout abc1234 -- app.js

# Stage and commit the restoration
git add app.js
git commit -m "revert: restore app.js to working state (abc1234)"
git push origin main
```

---

### Scenario 40: Auto-Deploy on Every Push Using Git Hooks (Server-Side)

**Problem:** Set up a bare repo on a server that auto-deploys when you push to it.

```bash
# On server: create bare repo
mkdir -p /var/repos/myapp.git
cd /var/repos/myapp.git
git init --bare

# Create post-receive hook
cat > hooks/post-receive << 'EOF'
#!/bin/bash
TARGET="/var/www/myapp"
GIT_DIR="/var/repos/myapp.git"
BRANCH="main"

while read oldrev newrev ref; do
  if [[ $ref = refs/heads/$BRANCH ]]; then
    echo "Deploying $BRANCH branch..."
    git --work-tree=$TARGET --git-dir=$GIT_DIR checkout -f $BRANCH
    cd $TARGET
    npm install
    pm2 restart myapp
    echo "Deploy complete!"
  fi
done
EOF
chmod +x hooks/post-receive

# On local machine: add server as remote
git remote add production user@server:/var/repos/myapp.git
git push production main    # Auto-deploys!
```

---

### Scenario 41: Fix "Your Branch Has Diverged" Error

**Problem:** `git status` shows "Your branch and 'origin/main' have diverged, and have X and Y different commits respectively."

```bash
# See what diverged
git log --oneline --graph HEAD origin/main

# Option A: Rebase local onto remote (clean linear history)
git pull --rebase origin main

# Option B: Merge (creates merge commit)
git pull origin main

# Option C: If you want to discard your local commits
git reset --hard origin/main    # ⚠️ Loses local commits

# Verify
git status    # Should show "up to date"
```

---

### Scenario 42: Use Git to Track Config Files (Dotfiles)

**Problem:** You want to version-control your `~/.bashrc`, `~/.vimrc`, etc. across machines.

```bash
# Initialize bare repo in home directory
git init --bare $HOME/.dotfiles

# Create alias to manage dotfiles
alias dotfiles='git --git-dir=$HOME/.dotfiles/ --work-tree=$HOME'

# Add to .bashrc permanently
echo "alias dotfiles='git --git-dir=\$HOME/.dotfiles/ --work-tree=\$HOME'" >> ~/.bashrc

# Track files
dotfiles add ~/.bashrc ~/.vimrc ~/.gitconfig
dotfiles commit -m "add: initial dotfiles"
dotfiles remote add origin git@github.com:user/dotfiles.git
dotfiles push origin main

# On new machine: clone and restore
git clone --bare git@github.com:user/dotfiles.git $HOME/.dotfiles
dotfiles checkout
```

---

### Scenario 43: Git Tag for Every Successful CI Build

**Problem:** You want to auto-tag every successful CI/CD build with the build number.

```bash
# In GitHub Actions workflow
- name: Tag build
  run: |
    git config user.name "github-actions"
    git config user.email "actions@github.com"
    git tag -a "build-${{ github.run_number }}" -m "CI build #${{ github.run_number }}"
    git push origin "build-${{ github.run_number }}"

# In Jenkins pipeline
sh """
  git tag -a "build-${BUILD_NUMBER}" -m "Jenkins build #${BUILD_NUMBER}"
  git push origin "build-${BUILD_NUMBER}"
"""

# List all build tags
git tag -l "build-*" | sort -V | tail -10
```

---

### Scenario 44: Work on Feature Without Committing (Worktree)

**Problem:** You need to review `hotfix` branch code while actively coding on `feature/auth` — without stashing.

```bash
# Create a linked worktree for hotfix
git worktree add ../hotfix-review hotfix/v2.1

# Now you have TWO working directories:
# /project/         → your feature/auth branch
# /hotfix-review/   → hotfix/v2.1 branch

# Work on each independently in different terminals
cd /hotfix-review && cat bugfix.js   # review here

# After review, remove worktree
git worktree remove ../hotfix-review
git worktree list    # Confirm removed
```

---

### Scenario 45: Find Deleted File and Restore It

**Problem:** A file `utils/helper.js` was deleted 10 commits ago. You need it back.

```bash
# Step 1: Find the commit that deleted it
git log --all --full-history -- utils/helper.js
# Shows: abc1234 "refactor: remove helper file"

# Step 2: Restore it from the commit BEFORE deletion
git checkout abc1234~1 -- utils/helper.js

# Step 3: Commit the restoration
git add utils/helper.js
git commit -m "restore: recover accidentally deleted helper.js"
```

---

### Scenario 46: Generate a Changelog from Git History

**Problem:** PM wants a list of all features and fixes since the last release.

```bash
# Changes since last tag
git log v1.5.0..HEAD --oneline

# Only feat and fix commits (Conventional Commits)
git log v1.5.0..HEAD --oneline | grep -E "^[a-f0-9]+ (feat|fix)"

# Format as markdown changelog
git log v1.5.0..HEAD --pretty=format:"- %s (%h)" | grep -E "(feat|fix)"

# Using git-changelog tool
npm install -g conventional-changelog-cli
conventional-changelog -p angular -i CHANGELOG.md -s
```

---

### Scenario 47: Lock Down Repository — Nobody Can Push Without Review

**Problem:** Junior developers keep pushing broken code directly to `main`. Set up rules.

```bash
# GitHub → Repository Settings → Branches → Add rule for main:
# ✅ Require pull request reviews before merging
# ✅ Require approvals: 1
# ✅ Dismiss stale reviews when new commits pushed
# ✅ Require status checks to pass (CI)
# ✅ Require branches to be up to date
# ✅ Do not allow bypassing above settings

# Pre-commit hook (local enforcement)
cat > .git/hooks/pre-commit << 'EOF'
#!/bin/bash
# Run linter before commit
npm run lint
if [ $? -ne 0 ]; then
  echo "❌ Linting failed. Fix errors before committing."
  exit 1
fi
EOF
chmod +x .git/hooks/pre-commit
```

---

### Scenario 48: Migrate Repository from GitLab to GitHub

**Problem:** Company is switching from GitLab to GitHub. Move all branches, tags, and history.

```bash
# Step 1: Clone GitLab repo as mirror (all branches + tags)
git clone --mirror git@gitlab.com:company/repo.git
cd repo.git

# Step 2: Create new GitHub repo (empty, no README)

# Step 3: Add GitHub as remote
git remote add github git@github.com:company/repo.git

# Step 4: Push everything (mirror push)
git push --mirror github

# Step 5: Verify on GitHub — all branches and tags should be there

# Step 6: Update local remotes for team
git remote set-url origin git@github.com:company/repo.git
```

---

### Scenario 49: Debug Why a File Keeps Showing as Modified

**Problem:** `git status` always shows `config/settings.json` as modified even after committing. Nothing actually changed.

```bash
# Check line ending issues
git diff config/settings.json
# If you see ^M characters → Windows/Unix line ending mismatch

# Fix: Set line ending normalization
git config --global core.autocrlf input    # On Mac/Linux
git config --global core.autocrlf true     # On Windows

# Fix: Normalize the file
git add --renormalize config/settings.json
git commit -m "fix: normalize line endings"

# Alternative: Ignore local changes (if file differs per machine)
git update-index --skip-worktree config/settings.json
```

---

### Scenario 50: Enforce Commit Message Format in CI

**Problem:** Team is not following Conventional Commits standard. Enforce it in GitHub Actions.

```bash
# .github/workflows/commit-lint.yml
cat > .github/workflows/commit-lint.yml << 'EOF'
name: Commit Lint
on: [pull_request]

jobs:
  commitlint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
        with:
          fetch-depth: 0
      - uses: wagoid/commitlint-github-action@v5
EOF

# commitlint.config.js
cat > commitlint.config.js << 'EOF'
module.exports = {
  extends: ['@commitlint/config-conventional'],
  rules: {
    'type-enum': [2, 'always', ['feat','fix','docs','style','refactor','test','chore']],
    'subject-max-length': [2, 'always', 72],
  }
}
EOF
```

---

### Scenario 51: Track When a Bug Was Introduced (Without bisect)

**Problem:** You know the bug is in `auth.js` but don't know when it appeared. Fast check without full bisect.

```bash
# See full history of changes to auth.js
git log --oneline -- src/auth.js

# See every change made to auth.js (with diffs)
git log -p -- src/auth.js

# Search for when a specific function was added/removed
git log -S "validateToken" -- src/auth.js

# Search for when a line pattern changed
git log -G "return null" -- src/auth.js

# See file at each commit
git log --oneline -- src/auth.js | while read hash msg; do
  echo "=== $hash: $msg ==="
  git show $hash:src/auth.js | grep -n "validateToken"
done
```

---

### Scenario 52: Set Different Git Config Per Directory (Work vs Personal)

**Problem:** All repos in `~/work/` should use work email. All in `~/personal/` use personal email.

```bash
# Global config (~/.gitconfig)
[user]
  name = Sahil
  email = personal@gmail.com

[includeIf "gitdir:~/work/"]
  path = ~/.gitconfig-work

# Work config (~/.gitconfig-work)
cat > ~/.gitconfig-work << 'EOF'
[user]
  name = Sahil (Work)
  email = sahil@company.com
[core]
  sshCommand = ssh -i ~/.ssh/work_key
EOF

# Verify
cd ~/work/some-repo
git config user.email    # shows work@company.com

cd ~/personal/some-repo
git config user.email    # shows personal@gmail.com
```

---

### Scenario 53: Reorder Commits Before Pushing

**Problem:** Your last 4 commits are in wrong order. You want to reorder them before opening a PR.

```bash
git rebase -i HEAD~4

# Editor shows:
# pick aaa1 feat: add login button
# pick bbb2 fix: typo in README
# pick ccc3 feat: add logout button
# pick ddd4 docs: update API docs

# Reorder by moving lines:
# pick bbb2 fix: typo in README
# pick aaa1 feat: add login button
# pick ccc3 feat: add logout button
# pick ddd4 docs: update API docs

# Save and close — Git replays in new order
git push origin feature-branch --force-with-lease
```

---

### Scenario 54: Set Up Monorepo with Multiple Projects

**Problem:** You have frontend, backend, and infra in one repo. Set up Git for clean management.

```bash
# Structure
mkdir monorepo && cd monorepo
git init

mkdir -p frontend backend infra docs

# .gitignore per folder
echo "node_modules/" > frontend/.gitignore
echo "target/" > backend/.gitignore
echo ".terraform/" > infra/.gitignore

# Root .gitignore
cat > .gitignore << 'EOF'
*.log
.DS_Store
.env
*.tfstate
EOF

# Sparse checkout for developers who only need frontend
git sparse-checkout init --cone
git sparse-checkout set frontend docs

# Verify
git sparse-checkout list
```

---

### Scenario 55: Show All Contributors and Their Commit Count

**Problem:** Project manager wants a summary of who contributed how much.

```bash
# Count commits per author (sorted)
git shortlog -sn

# Include all branches
git shortlog -sn --all

# With email
git shortlog -sne

# Lines added/removed per author
git log --format='%aN' | sort -u | while read author; do
  echo -n "$author: "
  git log --author="$author" --pretty=tformat: --numstat |
    awk '{ add += $1; del += $2 } END { print "+"add" -"del }'
done

# Since specific date
git shortlog -sn --since="2025-01-01"
```

---

### Scenario 56: Automatically Add Ticket Number to Commit Messages

**Problem:** Every commit must include Jira ticket number from branch name (e.g., `feature/PROJ-123-login`).

```bash
# Create prepare-commit-msg hook
cat > .git/hooks/prepare-commit-msg << 'EOF'
#!/bin/bash
COMMIT_MSG_FILE=$1
BRANCH_NAME=$(git symbolic-ref --short HEAD)

# Extract ticket number (e.g., PROJ-123)
TICKET=$(echo "$BRANCH_NAME" | grep -oE '[A-Z]+-[0-9]+')

if [ -n "$TICKET" ]; then
  # Prepend ticket to commit message
  sed -i.bak "1s/^/[$TICKET] /" "$COMMIT_MSG_FILE"
fi
EOF
chmod +x .git/hooks/prepare-commit-msg

# Now when you're on branch feature/PROJ-123-login
# git commit -m "add login form"
# becomes: [PROJ-123] add login form
```

---

### Scenario 57: List All Files Ever Committed (Including Deleted)

**Problem:** Security audit needs every file that ever existed in the repo, including deleted ones.

```bash
# List all files ever in history
git log --all --full-history --name-only --pretty=format: | sort -u | grep -v "^$"

# Only deleted files
git log --all --diff-filter=D --summary --pretty=format: | grep delete | awk '{print $4}'

# Files containing specific pattern (ever)
git grep -l "password" $(git rev-list --all)
```

---

### Scenario 58: Set Up Git Aliases for Common DevOps Workflows

**Problem:** You type the same long commands dozens of times per day.

```bash
# Add to ~/.gitconfig
git config --global alias.st "status"
git config --global alias.co "checkout"
git config --global alias.br "branch"
git config --global alias.lg "log --oneline --graph --all --decorate"
git config --global alias.undo "reset --soft HEAD~1"
git config --global alias.unstage "restore --staged"
git config --global alias.last "log -1 HEAD --stat"
git config --global alias.aliases "config --get-regexp alias"
git config --global alias.contributors "shortlog -sn"
git config --global alias.find "log --all --full-history --"

# Usage
git lg            # beautiful log graph
git undo          # undo last commit, keep changes
git unstage .     # unstage everything
git last          # see last commit details
git find file.js  # find all commits that touched file.js
```

---

### Scenario 59: Recover Commits After Accidental `git reset --hard`

**Problem:** You ran `git reset --hard HEAD~5` by mistake and lost 5 commits.

```bash
# Step 1: Don't panic — reflog has everything
git reflog
# Shows: abc1234 HEAD@{0}: reset: moving to HEAD~5
#        def5678 HEAD@{1}: commit: your last good commit

# Step 2: Reset back to before the accident
git reset --hard HEAD@{1}
# OR use the specific commit hash
git reset --hard def5678

# Step 3: Verify your commits are back
git log --oneline

# Step 4: Push (if needed)
git push origin feature-branch --force-with-lease
```

> ⚠️ Reflog only works for LOCAL history. Once `git gc` runs (after ~90 days), it's gone.

---

### Scenario 60: Clean Up Remote Tracking Branches

**Problem:** `git branch -a` shows dozens of remote branches that were deleted on GitHub months ago.

```bash
# See stale remote tracking refs
git remote show origin | grep "stale"

# Remove stale remote tracking branches (prune)
git fetch --prune
# OR
git remote prune origin

# Set auto-prune on every fetch
git config --global fetch.prune true

# See what will be pruned (dry run)
git fetch --prune --dry-run

# Delete all remote tracking refs at once
git branch -r | grep -v HEAD | while read branch; do
  git branch -dr "$branch"
done
git fetch
```

---

*© Git, GitHub & GitLab — 60 Additional Scenario Questions*
*Total scenarios: 60 | All real-world, DevOps-ready*
