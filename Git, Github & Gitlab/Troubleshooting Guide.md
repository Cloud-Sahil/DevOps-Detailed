# Git, GitHub & GitLab Troubleshooting Guide

## Table of Contents

1. Git Not Installed
2. Git Command Not Found
3. Git Authentication Failed
4. Permission Denied (Public Key)
5. Repository Not Found
6. Failed to Push Some Refs
7. Merge Conflict
8. Detached HEAD State
9. Branch Not Found
10. Remote Origin Already Exists
11. Changes Not Staged for Commit
12. Nothing to Commit
13. Undo Last Commit
14. Accidentally Committed Wrong Files
15. Git Ignore Not Working
16. Large File Push Failed
17. SSL Certificate Problem
18. Host Key Verification Failed
19. Git Clone Slow
20. Git Pull Overwrites Local Changes
21. Rejected Push Due to Diverged Branch
22. GitLab Runner Offline
23. GitLab Pipeline Failed
24. GitHub Actions Failed
25. GitHub Authentication Using Token
26. Invalid Username or Password
27. Branch Protection Error
28. Cannot Delete Branch
29. Git Fetch vs Pull Issues
30. Corrupted Repository
31. Scenario-Based Troubleshooting Questions
32. Useful Commands

---

# 1. Git Not Installed

## Error

```bash
git: command not found
```

## Cause

Git package is not installed.

## Solution

### Ubuntu

```bash
sudo apt update
sudo apt install git -y
```

### Amazon Linux

```bash
sudo yum install git -y
```

### Verify

```bash
git --version
```

---

# 2. Git Command Not Found

## Error

```bash
git: command not found
```

## Cause

Git binary path missing.

## Solution

```bash
which git
```

Add Git to PATH if required.

```bash
export PATH=$PATH:/usr/bin
```

---

# 3. Git Authentication Failed

## Error

```bash
Authentication failed
```

## Cause

Wrong credentials or expired token.

## Solution

Generate Personal Access Token (PAT).

GitHub:

```bash
Settings
→ Developer Settings
→ Personal Access Tokens
```

Use token instead of password.

---

# 4. Permission Denied (Public Key)

## Error

```bash
Permission denied (publickey)
```

## Cause

SSH key missing or not configured.

## Solution

Generate SSH Key

```bash
ssh-keygen -t rsa -b 4096
```

Start SSH Agent

```bash
eval "$(ssh-agent -s)"
```

Add Key

```bash
ssh-add ~/.ssh/id_rsa
```

Copy Public Key

```bash
cat ~/.ssh/id_rsa.pub
```

Add to GitHub/GitLab.

Test

```bash
ssh -T git@github.com
```

---

# 5. Repository Not Found

## Error

```bash
Repository not found
```

## Cause

Wrong repository URL.

## Solution

Check remote URL.

```bash
git remote -v
```

Update URL.

```bash
git remote set-url origin <repo-url>
```

---

# 6. Failed To Push Some Refs

## Error

```bash
failed to push some refs
```

## Cause

Remote branch contains newer commits.

## Solution

```bash
git pull origin main
git push origin main
```

---

# 7. Merge Conflict

## Error

```bash
CONFLICT (content)
```

## Cause

Same file modified by multiple users.

## Solution

Check conflict.

```bash
git status
```

Edit file manually.

```text
<<<<<<< HEAD
Code A
=======
Code B
>>>>>>> branch
```

Remove markers.

Commit changes.

```bash
git add .
git commit -m "Resolved conflict"
```

---

# 8. Detached HEAD State

## Error

```bash
HEAD detached
```

## Cause

Checked out commit instead of branch.

## Solution

```bash
git checkout main
```

Or create branch.

```bash
git checkout -b new-branch
```

---

# 9. Branch Not Found

## Error

```bash
pathspec did not match
```

## Solution

List branches.

```bash
git branch -a
```

Fetch latest branches.

```bash
git fetch --all
```

---

# 10. Remote Origin Already Exists

## Error

```bash
remote origin already exists
```

## Solution

Remove existing remote.

```bash
git remote remove origin
```

Add again.

```bash
git remote add origin <url>
```

---

# 11. Changes Not Staged For Commit

## Check

```bash
git status
```

## Stage Files

```bash
git add .
```

---

# 12. Nothing To Commit

## Error

```bash
nothing to commit
```

## Cause

No changes detected.

## Verify

```bash
git status
```

---

# 13. Undo Last Commit

## Keep Changes

```bash
git reset --soft HEAD~1
```

## Remove Commit Completely

```bash
git reset --hard HEAD~1
```

---

# 14. Accidentally Committed Wrong Files

## Solution

```bash
git reset HEAD~
```

Remove unwanted files.

```bash
git add .
git commit -m "Correct commit"
```

---

# 15. Git Ignore Not Working

## Cause

File already tracked.

## Solution

```bash
git rm -r --cached .
git add .
git commit -m "Updated gitignore"
```

---

# 16. Large File Push Failed

## Error

```bash
File exceeds GitHub size limit
```

## Solution

Use Git LFS.

```bash
git lfs install
```

Track file.

```bash
git lfs track "*.zip"
```

---

# 17. SSL Certificate Problem

## Error

```bash
SSL certificate problem
```

## Solution

Update CA certificates.

```bash
sudo apt install ca-certificates -y
```

Verify.

```bash
git config --global http.sslVerify true
```

---

# 18. Host Key Verification Failed

## Error

```bash
Host key verification failed
```

## Solution

Remove old key.

```bash
ssh-keygen -R github.com
```

Reconnect.

```bash
ssh -T git@github.com
```

---

# 19. Git Clone Slow

## Causes

- Slow internet
- Large repository

## Solution

Shallow Clone

```bash
git clone --depth 1 <repo-url>
```

---

# 20. Git Pull Overwrites Local Changes

## Error

```bash
local changes would be overwritten
```

## Solution

Commit changes.

```bash
git add .
git commit -m "backup"
```

Or stash.

```bash
git stash
git pull
git stash pop
```

---

# 21. Rejected Push Due To Diverged Branch

## Solution

```bash
git pull --rebase origin main
git push origin main
```

---

# 22. GitLab Runner Offline

## Check Runner

```bash
gitlab-runner status
```

Restart Runner.

```bash
sudo gitlab-runner restart
```

---

# 23. GitLab Pipeline Failed

## Check Logs

```bash
CI/CD → Pipelines
```

Validate YAML.

```bash
.gitlab-ci.yml
```

Test locally.

```bash
gitlab-ci-lint
```

---

# 24. GitHub Actions Failed

## Check Workflow

```bash
.github/workflows/
```

View logs.

```bash
Actions → Workflow Run
```

Validate YAML syntax.

---

# 25. GitHub Authentication Using Token

## Clone

```bash
git clone https://TOKEN@github.com/user/repo.git
```

## Store Credentials

```bash
git config --global credential.helper store
```

---

# 26. Invalid Username Or Password

## Cause

GitHub removed password authentication.

## Solution

Use PAT Token.

```bash
Personal Access Token
```

---

# 27. Branch Protection Error

## Error

```bash
protected branch hook declined
```

## Cause

Direct push blocked.

## Solution

Create Pull Request.

```bash
Feature Branch
→ Pull Request
→ Merge
```

---

# 28. Cannot Delete Branch

## Error

```bash
Cannot delete checked out branch
```

## Solution

Switch branch.

```bash
git checkout main
```

Delete.

```bash
git branch -D feature
```

---

# 29. Git Fetch vs Pull Issues

## Fetch

Downloads changes only.

```bash
git fetch
```

## Pull

Downloads + merges.

```bash
git pull
```

---

# 30. Corrupted Repository

## Solution

Verify.

```bash
git fsck
```

Reclone repository.

```bash
git clone <repo-url>
```

---

# 31. Scenario-Based Troubleshooting Questions

## Scenario 1

Developer cannot push code.

### What will you check?

- Remote URL
- Git credentials
- SSH keys
- Branch permissions

---

## Scenario 2

Pipeline suddenly fails after code merge.

### Steps

- Check pipeline logs
- Validate YAML
- Check runner status
- Verify environment variables

---

## Scenario 3

GitHub Actions stuck in queued state.

### Check

- Available runners
- Workflow permissions
- GitHub service status

---

## Scenario 4

Merge conflict during release deployment.

### Steps

- Identify conflicting files
- Resolve manually
- Test locally
- Merge again

---

## Scenario 5

Developer deleted important branch accidentally.

### Recovery

```bash
git reflog
```

Restore commit.

```bash
git checkout -b recovered-branch <commit-id>
```

---

## Scenario 6

Repository size becomes very large.

### Solution

- Remove unwanted files
- Use Git LFS
- Clean history

---

## Scenario 7

GitLab Runner not picking jobs.

### Check

```bash
gitlab-runner verify
```

- Tags
- Runner registration
- Network connectivity

---

## Scenario 8

Git pull takes too long.

### Check

- Repository size
- Network
- Large binaries

---

## Scenario 9

Production branch protected.

Developer needs urgent fix.

### Solution

- Create Hotfix Branch
- Raise PR
- Get approval
- Merge

---

## Scenario 10

SSH works yesterday but fails today.

### Check

- SSH agent
- Key expiry
- Known hosts
- Firewall rules

---

# 32. Useful Commands

## Check Version

```bash
git --version
```

## Configure Username

```bash
git config --global user.name "Sahil"
```

## Configure Email

```bash
git config --global user.email "sahil@example.com"
```

## Clone Repository

```bash
git clone <repo-url>
```

## Check Status

```bash
git status
```

## Add Changes

```bash
git add .
```

## Commit

```bash
git commit -m "message"
```

## Push

```bash
git push origin main
```

## Pull

```bash
git pull origin main
```

## Branches

```bash
git branch
```

## Logs

```bash
git log --oneline
```

## Stash

```bash
git stash
git stash pop
```

## Reflog

```bash
git reflog
```

---

# Git Troubleshooting Workflow

```text
Issue Occurs
      |
      v
Check Error Message
      |
      v
git status
      |
      v
git log / git reflog
      |
      v
Verify Credentials
      |
      v
Verify Remote URL
      |
      v
Verify Branch
      |
      v
Fix Issue
      |
      v
Test Again
```
