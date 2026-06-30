# DevOps Mastery — Complete Interview Question Bank
---

## 📑 Table of Contents

1. [Git](#1-git)
2. [GitHub](#2-github)
3. [GitLab](#3-gitlab)
4. [Python (for DevOps)](#4-python-for-devops)
5. [Shell Scripting](#5-shell-scripting)
6. [Docker](#6-docker)
7. [Kubernetes](#7-kubernetes)
8. [Jenkins](#8-jenkins)
9. [Terraform](#9-terraform)
10. [Networking](#10-networking)
11. [Azure](#11-azure)
12. [Prometheus](#12-prometheus)
13. [Grafana](#13-grafana)
14. [Datadog](#14-datadog)
15. [SonarQube](#15-sonarqube)
16. [Trivy](#16-trivy)

---

## 1. Git

### 🟢 Beginner

**1. What is Git, and how is it different from a centralized version control system like SVN?**
Git is a distributed version control system (DVCS) where every developer has a full copy of the repository, including its entire history, on their local machine. In contrast, SVN is centralized — only the server holds the full history, and developers only check out a working copy. This means Git allows offline commits, faster operations (since most actions are local), and better branching/merging support.

**2. What is the difference between `git init` and `git clone`?**
`git init` creates a brand-new, empty Git repository in the current directory — used when starting a project from scratch. `git clone` copies an existing remote repository (along with its full history) to your local machine, automatically setting up a remote called `origin`.

**3. What is a commit in Git?**
A commit is a snapshot of your staged changes at a point in time, along with metadata (author, timestamp, commit message, and a pointer to the parent commit). Each commit has a unique SHA-1 hash that identifies it.

**4. What is the staging area (index) in Git?**
The staging area is an intermediate space between your working directory and the repository. Files added using `git add` move into staging, where you can review and group exactly what will be included in the next commit before committing.

**5. What is the difference between `git pull` and `git fetch`?**
`git fetch` downloads new commits/branches from the remote but does not merge them into your local branch. `git pull` does a `git fetch` followed immediately by a `git merge` (or rebase), updating your current branch automatically.

**6. What does `git status` show?**
It shows the current state of the working directory and staging area — which files are modified, staged, untracked, or in conflict.

**7. What is a branch in Git?**
A branch is simply a movable pointer to a specific commit. The default branch is usually `main` or `master`. Branches let you work on features or fixes in isolation without affecting the main codebase.

**8. How do you create and switch to a new branch?**
```bash
git branch feature-x        # create
git checkout feature-x      # switch
# or combined:
git checkout -b feature-x
# modern syntax:
git switch -c feature-x
```

**9. What is the difference between `git merge` and `git rebase`?**
`git merge` combines two branches by creating a new "merge commit" that has two parents, preserving the full history of both branches. `git rebase` rewrites history by moving (replaying) your branch's commits on top of another branch, producing a linear history without a merge commit.

**10. What is a `.gitignore` file used for?**
It tells Git which files or directories to intentionally ignore (not track) — such as build artifacts, `node_modules/`, `.env` files, or IDE config files.

**11. What is the difference between `git add .` and `git add -A`?**
`git add .` stages new and modified files in the current directory and subdirectories, but historically did not stage deletions outside the current directory in old Git versions. `git add -A` stages all changes (new, modified, deleted) across the entire repository, regardless of current directory. In modern Git (2.x+), `git add .` from the repo root behaves almost the same as `-A`.

**12. What does `git log` do, and how do you see a compact view?**
`git log` shows the commit history. For a one-line-per-commit compact view:
```bash
git log --oneline --graph --decorate --all
```

**13. What is `HEAD` in Git?**
`HEAD` is a pointer to the current branch reference, which in turn points to the latest commit on that branch. It represents "where you currently are" in the repository.

**14. How do you undo the last commit but keep the changes?**
```bash
git reset --soft HEAD~1
```
This moves HEAD back one commit but keeps all changes staged.

**15. What is the difference between `git reset` and `git revert`?**
`git reset` moves the branch pointer backward, effectively rewriting history (dangerous on shared branches). `git revert` creates a *new* commit that undoes the changes of a previous commit, preserving history — safe for shared/public branches.

---

### 🟡 Intermediate

**16. Explain the three states of a file in Git.**
Modified (changed but not staged), Staged (marked to go into the next commit), and Committed (safely stored in the local repository database).

**17. What is a detached HEAD state, and how do you fix it?**
A detached HEAD occurs when you checkout a specific commit (not a branch). Any new commits made here aren't attached to a branch and can be lost. Fix it by creating a branch at that point: `git checkout -b new-branch-name`.

**18. What's the difference between `git reset --soft`, `--mixed`, and `--hard`?**
- `--soft`: Moves HEAD only; staged changes are kept.
- `--mixed` (default): Moves HEAD and unstages changes, but keeps them in the working directory.
- `--hard`: Moves HEAD and discards all changes in staging and working directory — irreversible without reflog.

**19. What is `git stash` used for?**
It temporarily shelves (stashes) uncommitted changes so you can switch branches cleanly, then reapply them later with `git stash pop` or `git stash apply`.

**20. How do you resolve a merge conflict?**
Git marks conflicting sections in the file with `<<<<<<<`, `=======`, `>>>>>>>` markers. You manually edit the file to resolve the conflict, then run `git add <file>` and `git commit` to finalize the merge.

**21. What is `git cherry-pick`?**
It applies the changes from a specific commit (identified by its hash) onto your current branch, without merging the entire branch. Useful for porting a single bug fix to another branch.

**22. What is the difference between a lightweight tag and an annotated tag?**
A lightweight tag is just a pointer to a commit (like a branch that doesn't move). An annotated tag is a full Git object with its own metadata (tagger name, date, message) and can be GPG-signed — recommended for releases.

**23. What does `git rebase -i` (interactive rebase) allow you to do?**
It lets you edit commit history interactively — squash multiple commits into one, reword commit messages, reorder commits, or drop commits entirely, before they're pushed.

**24. What is the difference between `git fetch origin` and `git fetch --all`?**
`git fetch origin` fetches updates only from the `origin` remote. `git fetch --all` fetches updates from every configured remote in the repository.

**25. How do you remove a file from Git tracking but keep it locally?**
```bash
git rm --cached filename
```
This untracks the file while leaving it untouched on disk (commonly used after adding something to `.gitignore` retroactively).

**26. What is the reflog, and why is it useful?**
`git reflog` records every change to the tip of branches (commits, resets, rebases, checkouts) on your local machine, even ones not reachable from any branch. It's a safety net to recover "lost" commits after a `reset --hard` or a bad rebase.

**27. What's the difference between `origin` and `upstream`?**
`origin` conventionally refers to the remote repository you cloned from (often your fork). `upstream` is a convention for the original repository you forked from — used in open-source contribution workflows to sync your fork with the source project.

**28. How do you squash the last 3 commits into 1?**
```bash
git rebase -i HEAD~3
# then mark commits 2 and 3 as "squash" (s) in the editor that opens
```

**29. What is a fast-forward merge?**
When the target branch has no new commits since the feature branch diverged, Git can simply move the branch pointer forward without creating a merge commit — this is a fast-forward merge.

**30. How do you find which commit introduced a bug?**
Use `git bisect` — a binary search tool. You mark a known good commit and a known bad commit, and Git checks out commits in between, you test and mark each as good/bad, until it isolates the exact offending commit.

---

### 🔴 Advanced

**31. Explain Git's internal object model (blob, tree, commit, tag).**
Git stores everything as content-addressed objects in `.git/objects`:
- **Blob**: stores raw file content (no filename/metadata)
- **Tree**: represents a directory — maps filenames to blob/tree SHAs (like a folder structure)
- **Commit**: points to a tree (project snapshot), parent commit(s), author, and message
- **Tag**: a named pointer to a specific commit (annotated tags are also objects themselves)

**32. How does Git determine file changes internally — does it store diffs?**
No — contrary to popular belief, Git does not store deltas/diffs between commits. Each commit stores a full snapshot (tree) of the project. However, Git compresses and deduplicates identical blobs (via SHA-1 content hashing), and uses delta compression only when packing objects into `.pack` files for storage efficiency — not as its core model.

**33. What is the difference between `git merge --no-ff` and a regular merge?**
A regular merge does a fast-forward if possible (no merge commit created). `--no-ff` forces a merge commit to be created even when a fast-forward is possible, preserving the historical context that a feature branch existed.

**34. Explain Git's three-way merge algorithm.**
Git uses the common ancestor (merge base) of the two branches along with the tips of both branches to compute a three-way diff. It compares changes from the ancestor to each tip and automatically merges non-overlapping changes; overlapping changes to the same lines produce a conflict requiring manual resolution.

**35. 🎯 Scenario: Your team accidentally pushed a large 2GB file in a commit 50 commits ago. The repo is now bloated. How do you clean it up?**
Use `git filter-repo` (the modern, recommended replacement for the deprecated `filter-branch`) or BFG Repo-Cleaner to rewrite history and strip the file from all commits:
```bash
git filter-repo --path path/to/large-file --invert-paths
```
After rewriting, force-push to the remote, have all collaborators re-clone (since history hashes change), and run `git gc --aggressive --prune=now` to reclaim space. Also add the file pattern to `.gitignore` going forward, and consider Git LFS for large binaries.

**36. What is Git LFS, and when would you use it?**
Git Large File Storage replaces large files (videos, datasets, binaries) with small text pointers in the repo, while storing the actual file content on a remote LFS server. This keeps the Git repo small and fast while still versioning large assets.

**37. Explain how `git gc` works and why it matters at scale.**
`git gc` (garbage collect) compresses loose objects into packfiles, removes unreachable/orphaned objects (after the reflog expiry), and optimizes the repository for storage and performance. In large enterprise monorepos, scheduled `git gc` is essential to prevent repo bloat and slow clone/fetch times.

**38. What's the difference between `git rebase` and `git merge` in terms of commit hash stability, and why does this matter for shared branches?**
Rebase rewrites commit hashes (since it replays commits with new parents), which breaks history for anyone who already pulled the old commits — this is why you should *never* rebase a branch others are actively basing work on. Merge preserves all original commit hashes, making it safe for shared/public branches. The rule of thumb: rebase local/feature branches before sharing; merge for integrating shared branches.

**39. 🎯 Scenario: Two team members rebased the same shared feature branch independently and pushed with `--force`. The branch history is now diverged and conflicting. How do you recover safely?**
First, do not panic-force-push further. Use `git reflog` on each affected machine to identify the last known-good commit SHA before either rebase. Communicate with the team, pick one canonical version (usually whoever pushed first or has the most complete work), reset the remote branch to that commit, and have others rebase their local work on top using `git rebase --onto`. Going forward, enforce branch protection rules (no force-push allowed on shared branches) to prevent recurrence.

**40. What is `git worktree`, and what production problem does it solve?**
`git worktree` lets you check out multiple branches of the same repository into separate directories simultaneously, sharing the same `.git` database. This avoids the need for multiple full clones when you need to, for example, hotfix `main` while still having your feature branch checked out — saving disk space and avoiding stash juggling.

**41. Explain signed commits and why they matter in a security-conscious enterprise.**
GPG/SSH-signed commits cryptographically prove that a commit was made by the holder of a specific private key, preventing identity spoofing in the commit author field (which is otherwise just plaintext and trivially fakeable). Enterprises enforce signed commits via branch protection rules to ensure auditability and non-repudiation, especially in regulated industries (finance, healthcare).

**42. What is the `.git/hooks` directory, and how can it be used in an enterprise CI/CD strategy?**
It contains scripts that Git executes automatically at certain points (pre-commit, pre-push, post-merge, etc.). While client-side hooks aren't version-controlled by default (a limitation), enterprises often use tools like `pre-commit` framework or `husky` to standardize and distribute hooks across the team — enforcing linting, secret-scanning, or commit message format before code ever reaches CI.

**43. 🎯 Scenario: A sensitive API key was committed and pushed to a public GitHub repo 6 months ago, across hundreds of commits since. What's your full remediation plan?**
1. **Immediately rotate/revoke the exposed credential** at the source (this is non-negotiable — assume it's compromised regardless of cleanup).
2. Use `git filter-repo` or BFG to strip the secret from all historical commits.
3. Force-push the rewritten history and instruct all clones to be deleted and re-cloned.
4. Since the repo was public, also request GitHub support to purge cached forks/clones where possible, and check if it was indexed by secret-scanning services.
5. Add the file/pattern to `.gitignore`, and implement pre-commit secret-scanning (e.g., `gitleaks`, `truffleHog`) plus GitHub's push protection to prevent recurrence.

**44. What is the difference between `git merge-base` and how is it used in CI pipelines?**
`git merge-base branch1 branch2` finds the common ancestor commit of two branches. CI pipelines use this to compute diffs only against the actual divergence point (rather than the full branch history) — useful for "changed files since branch point" logic in monorepo build optimization.

**45. How does Git handle binary files differently from text files internally?**
Internally, Git treats all files as blobs regardless of type — it doesn't inherently distinguish binary from text at the object level. However, Git's diff/merge tools attempt line-based diffing, which fails meaningfully for binaries; Git detects likely-binary files (via NUL byte heuristics or `.gitattributes` rules) and instead reports "Binary files differ" rather than attempting a line diff, and merge conflicts on binaries cannot be auto-resolved.

---

## 2. GitHub

### 🟢 Beginner

**1. What is GitHub, and how is it different from Git?**
Git is the version control *tool/software*. GitHub is a cloud-based hosting platform built around Git that adds collaboration features — pull requests, issues, project boards, code review, CI/CD via Actions, and access control. You can use Git without GitHub, but GitHub requires Git underneath.

**2. What is a repository (repo) on GitHub?**
A repository is a project's storage space on GitHub — it holds the code, commit history, branches, issues, and settings for that project.

**3. What is a Pull Request (PR)?**
A PR is a request to merge changes from one branch (or fork) into another. It provides a space for code review, automated checks (CI), and discussion before the changes are merged into the target branch.

**4. What is the difference between forking and cloning a repository?**
Cloning creates a local copy of a repo on your machine, connected to the original remote. Forking creates a full copy of the repo *under your own GitHub account*, allowing you to make changes independently and propose them back via a Pull Request — the standard open-source contribution model.

**5. What is a GitHub Issue?**
An issue is a tracker entry used to report bugs, request features, or discuss tasks. It supports labels, assignees, milestones, and can be linked to PRs that resolve it.

**6. What is a README.md file used for?**
It's the front-page documentation of a repository, automatically rendered on the repo's homepage — typically containing project description, setup instructions, usage examples, and contribution guidelines.

**7. What does "starring" a repository do?**
Starring bookmarks a repo to your GitHub profile for quick access later and signals appreciation/popularity — it has no functional effect on the code.

**8. What is a GitHub organization?**
An organization is a shared account for multiple people to collaborate across many repositories, with centralized billing, team-based permissions, and security policies — used by companies and open-source projects.

**9. What's the difference between a public and private repository?**
Public repos are visible to anyone on the internet (read access); private repos are visible only to people explicitly granted access. Both can still have fine-grained write/admin permissions.

**10. How do you add a collaborator to a repository?**
Repo Settings → Collaborators and teams → Add people, then specify their permission level (Read, Triage, Write, Maintain, or Admin).

---

### 🟡 Intermediate

**11. What are GitHub Actions?**
A native CI/CD platform built into GitHub. Workflows are defined in YAML files under `.github/workflows/`, triggered by events (push, PR, schedule, manual dispatch), and run jobs composed of steps on hosted or self-hosted runners.

**12. What's the difference between a GitHub Action and a GitHub Workflow?**
A **workflow** is the overall automated process (the YAML file) — e.g., "build and deploy." An **action** is a reusable unit of code that performs a specific task within a workflow step (e.g., `actions/checkout`, `actions/setup-node`) — analogous to a function being called inside the workflow.

**13. What are branch protection rules?**
Repository settings that enforce conditions before code can be merged into a protected branch (like `main`) — e.g., requiring PR reviews, passing status checks, signed commits, or disallowing force-pushes and direct pushes.

**14. What is the difference between GitHub Actions `on: push` and `on: pull_request` triggers?**
`push` triggers the workflow when commits are pushed directly to a branch/tag. `pull_request` triggers when a PR is opened, synchronized (new commits pushed to it), or other PR-related events — commonly used to run CI checks before merge, using the PR's merge commit context.

**15. What is a GitHub Secret, and how is it used in Actions?**
Secrets are encrypted environment variables stored at the repo, environment, or organization level, accessible in workflows via `${{ secrets.SECRET_NAME }}`. They're masked in logs and never exposed in plaintext, used for API keys, cloud credentials, tokens, etc.

**16. What is a self-hosted runner, and why would you use one over GitHub-hosted runners?**
A self-hosted runner is a machine you manage yourself (on-prem or cloud VM) that executes GitHub Actions jobs, instead of using GitHub's ephemeral cloud-hosted runners. Used when you need: access to internal/private networks, specific hardware (GPUs), custom software pre-installed, or to avoid per-minute billing for high-volume workloads.

**17. What is the difference between `GITHUB_TOKEN` and a Personal Access Token (PAT)?**
`GITHUB_TOKEN` is an automatically generated, short-lived token scoped to the specific repository and workflow run — used for most in-workflow API calls. A PAT is a manually created, longer-lived token tied to a user account with broader, user-configurable scopes — needed when a workflow must access resources outside its own repo (e.g., triggering workflows in other repos).

**18. What are GitHub Environments?**
Environments (e.g., `staging`, `production`) let you define deployment-specific protection rules (required reviewers, wait timers) and environment-specific secrets, used in Actions to gate sensitive deployment jobs.

**19. What is the difference between `git push` rejected due to "non-fast-forward" and how do you resolve it on GitHub?**
This happens when the remote branch has commits your local branch doesn't have. Resolve by pulling/rebasing first (`git pull --rebase`) then pushing, rather than blindly force-pushing, which would overwrite remote history.

**20. What is CODEOWNERS file used for?**
A `CODEOWNERS` file defines which individuals or teams are automatically requested as reviewers when specific files/paths are modified in a PR — enforcing domain-expert review on critical code areas.

**21. What's the difference between merge commit, squash merge, and rebase merge in a GitHub PR?**
- **Merge commit**: preserves all individual commits + adds a merge commit.
- **Squash and merge**: combines all PR commits into a single commit on the target branch — keeps history clean.
- **Rebase and merge**: replays each individual commit from the PR onto the target branch without a merge commit, preserving granular history linearly.

**22. How do GitHub Actions matrix builds work?**
A matrix strategy runs the same job multiple times with different variable combinations (e.g., multiple OS/language versions) in parallel:
```yaml
strategy:
  matrix:
    os: [ubuntu-latest, windows-latest]
    node: [16, 18, 20]
```
This generates 6 parallel job runs (2×3 combinations).

---

### 🔴 Advanced

**23. Explain how GitHub Actions caching works and how it improves pipeline performance.**
`actions/cache` stores dependencies (like `node_modules`, `.m2`, pip caches) keyed by a hash (often of the lockfile) between workflow runs, in GitHub's cache storage (10GB limit per repo, LRU eviction). On a cache hit, the restore step skips re-downloading/rebuilding dependencies, significantly cutting pipeline time — but stale caches can mask dependency issues, so cache keys must be invalidated correctly via hash-based keys.

**24. 🎯 Scenario: Your GitHub Actions workflow needs to deploy to AWS, but you don't want to store long-lived AWS access keys as GitHub Secrets. What's the modern, secure approach?**
Use **OpenID Connect (OIDC) federation** between GitHub Actions and AWS IAM. You configure an IAM Identity Provider trusting GitHub's OIDC token issuer, and an IAM role with a trust policy scoped to your specific repo/branch. The workflow then requests a short-lived, automatically-rotated token via `aws-actions/configure-aws-credentials` with `role-to-assume`, eliminating the need for any stored long-lived AWS secrets entirely — removing a major credential leakage risk.

**25. How do reusable workflows differ from composite actions in GitHub Actions?**
**Reusable workflows** (`workflow_call` trigger) are entire `.yml` workflow files that other workflows can call, supporting full job-level structure, secrets, and outputs — good for standardizing entire CI/CD pipelines across repos. **Composite actions** bundle multiple steps into a single reusable action (a `action.yml` with `using: composite`), invoked as a single step within a job — good for smaller, step-level reuse (like a standardized checkout+setup+lint sequence).

**26. What are GitHub Apps, and how do they differ from OAuth Apps and PATs for automation at scale?**
GitHub Apps are first-class integrations with fine-grained, repository-specific permissions and short-lived installation tokens (auto-expiring, typically 1 hour), installable at the org or repo level without being tied to a single user's identity. This is more secure and scalable than OAuth Apps (which act on behalf of a user with broader scope) or PATs (long-lived, tied to a person who may leave the company) — enterprises use GitHub Apps for bots, integrations, and automation that shouldn't depend on any individual's account.

**27. 🎯 Scenario: Your organization has 200 repositories, and you need to enforce that every single one requires 2 PR approvals and passing CI before merge — without manually configuring each repo. How do you do this at scale?**
Use a combination of: (1) **Organization-level ruleset** (GitHub's newer Rulesets feature, which supersedes per-repo branch protection) applied to a repository pattern like `*` to enforce required reviews and status checks org-wide; (2) for repos created via templates, bake the rules into a **repository template**; (3) for ongoing governance and drift detection, use **Terraform with the GitHub provider** to declaratively manage branch protection/rulesets as code across all repos, run via a scheduled CI job that reconciles any manual deviations.

**28. How does GitHub's CodeQL static analysis integrate into the security pipeline, and what's its underlying approach?**
CodeQL treats code as queryable data — it converts source code into a relational database representing the code's structure and data flow, then runs declarative QL queries against it to find vulnerability patterns (SQL injection, XSS, etc.) including complex *taint-tracking* paths across functions, not just simple pattern matching. It runs as a GitHub Actions workflow (`github/codeql-action`) on PRs and scheduled scans, surfacing results directly in the Security tab and as PR annotations.

**29. What is artifact retention and how does it impact compliance/audit requirements in regulated industries?**
GitHub Actions artifacts (build outputs, test reports, logs) have a configurable retention period (default 90 days, adjustable). For regulated industries (finance, healthcare) requiring longer audit trails, teams typically export artifacts to long-term storage (S3, Azure Blob) via a workflow step before GitHub's retention window expires, since GitHub itself isn't designed as a permanent compliance archive.

**30. 🎯 Scenario: A workflow run took 45 minutes and you need to cut it to under 10 minutes without losing test coverage. What's your optimization strategy?**
1. **Parallelize** the test suite using a matrix strategy splitting tests across multiple runners (e.g., by test file or `--shard`).
2. **Cache aggressively** — dependencies, build outputs, Docker layers (`actions/cache` or a registry-based cache).
3. **Use `needs` and conditional jobs** so unrelated jobs (e.g., docs build) don't block the critical path and can run in parallel rather than sequentially.
4. Switch to **larger/faster runners** for compute-heavy steps if budget allows.
5. **Fail fast** — run lint/unit tests (fast) before integration tests (slow), so failures surface earlier and don't waste runner time on doomed builds.
6. Use `paths` filters so the workflow doesn't even trigger for unrelated file changes (e.g., docs-only PRs).

---

## 3. GitLab

### 🟢 Beginner

**1. What is GitLab, and how does it differ from GitHub?**
GitLab is a DevOps platform offering Git repository hosting plus built-in CI/CD, issue tracking, container registry, and security scanning — all in a single application. While GitHub focuses heavily on its Actions marketplace ecosystem, GitLab's CI/CD (`.gitlab-ci.yml`) is more deeply integrated natively, and GitLab offers a full self-hosted Community/Enterprise Edition with feature parity to GitLab.com.

**2. What is `.gitlab-ci.yml`?**
It's the YAML configuration file at the root of a GitLab repository that defines the CI/CD pipeline — stages, jobs, scripts, and rules for when they run.

**3. What is a GitLab Runner?**
A Runner is the agent that executes the jobs defined in `.gitlab-ci.yml`. Runners can be shared (provided by GitLab.com), group-specific, or specific to a single project, and can run on different executors (Docker, Shell, Kubernetes, etc.).

**4. What is a Merge Request (MR) in GitLab?**
GitLab's equivalent of a GitHub Pull Request — a request to merge changes from a source branch into a target branch, with code review and pipeline checks before merging.

**5. What are stages and jobs in a GitLab CI pipeline?**
A **stage** is a logical grouping (e.g., `build`, `test`, `deploy`) that runs sequentially relative to other stages. A **job** is an individual task within a stage (e.g., `unit-test`, `lint`) — jobs within the same stage run in parallel by default.

**6. What is GitLab's built-in Container Registry?**
A private Docker registry included with every GitLab project, letting you push/pull container images directly tied to your repo, with access control matching the project's permissions.

**7. What does `stage: deploy` mean in a pipeline configuration?**
It assigns a job to the `deploy` stage, meaning it will only run after all jobs in earlier stages (e.g., `build`, `test`) have completed successfully.

**8. What's the difference between GitLab.com and self-managed GitLab?**
GitLab.com is the SaaS-hosted version managed entirely by GitLab Inc. Self-managed GitLab is installed and maintained on your own infrastructure (on-prem or your cloud account), giving full control over data residency, customization, and integration with internal systems.

---

### 🟡 Intermediate

**9. What is the difference between `only`/`except` and `rules` in `.gitlab-ci.yml`?**
`only`/`except` is the older, simpler syntax for controlling when a job runs (based on branch, tag, etc.). `rules` is the modern, more powerful and flexible replacement, supporting conditional logic (`if`), changes detection (`changes`), and dynamic behavior (`when: manual`, `allow_failure`) — GitLab recommends `rules` for all new pipelines.

**10. What are GitLab CI/CD variables, and what are the different scopes?**
Variables can be defined at the project, group, or instance level, or inline in `.gitlab-ci.yml`. They can be marked **protected** (only available on protected branches/tags) and **masked** (hidden in job logs) — critical for handling secrets securely.

**11. What is a `.gitlab-ci.yml` `include` used for?**
It lets you import external YAML configuration into your pipeline — from another file in the same repo, a different project, a remote URL, or a CI/CD template — enabling DRY, reusable pipeline definitions across multiple projects.

**12. Explain caching vs artifacts in GitLab CI.**
**Cache** is used to speed up jobs by reusing dependencies (e.g., `node_modules`) between pipeline runs — it's best-effort and not guaranteed. **Artifacts** are files explicitly passed between stages/jobs within the *same* pipeline run (e.g., a compiled binary from `build` passed to `deploy`) and can also be downloaded after the pipeline completes; they're guaranteed and tracked.

**13. What is `needs` in GitLab CI, and how does it change pipeline execution?**
By default, jobs run stage-by-stage sequentially. `needs` lets a job specify exact dependencies on other jobs (regardless of stage order), enabling a **Directed Acyclic Graph (DAG)** pipeline where jobs start as soon as their specific dependencies finish, rather than waiting for the entire previous stage — significantly speeding up pipelines.

**14. What is the difference between a shared runner, group runner, and specific runner?**
**Shared runners** are available to all projects in the GitLab instance. **Group runners** are scoped to all projects within a specific group. **Specific runners** are registered to and usable by only one designated project — typically used when a job needs special hardware or network access unique to that project.

**15. What is GitLab Auto DevOps?**
A feature that automatically detects your application's language/framework and builds a complete CI/CD pipeline (build, test, code quality, security scanning, container build, deploy to Kubernetes) with zero manual pipeline configuration — useful for quickly bootstrapping standardized pipelines.

**16. How do you trigger a pipeline manually for a specific job?**
Set `when: manual` on the job in `.gitlab-ci.yml`. The job then appears with a "play" button in the GitLab UI and must be manually triggered rather than running automatically — commonly used for production deployment gates.

---

### 🔴 Advanced

**17. Explain how GitLab's DAG pipelines (`needs`) differ from `dependencies`, and when to use which.**
`dependencies` controls *artifact passing* between jobs (which job's artifacts to download) but does not affect execution order beyond stage sequencing. `needs` controls *both* execution order (breaking the strict stage-by-stage model to create a DAG) *and* implicitly artifact access from the needed jobs. Use `needs` when you want true parallelism and out-of-order execution; use `dependencies` alone only when you need fine-grained artifact control within the standard stage model.

**18. 🎯 Scenario: Your monorepo has 15 microservices, but every push triggers a full pipeline rebuilding all 15 services, taking 40 minutes. How do you fix this in GitLab CI?**
Use `rules: changes` to conditionally trigger each service's build/test job only if files under its specific directory changed:
```yaml
service-a-build:
  stage: build
  rules:
    - changes:
        - services/service-a/**/*
  script: ./build.sh service-a
```
For more complex dependency graphs (where service-a depends on a shared library), combine this with a **dynamic child pipeline** generated by a script that calculates the actual affected services based on the diff, using `trigger` with `include` referencing a generated YAML artifact.

**19. What are GitLab parent-child pipelines, and what production problem do they solve?**
A parent pipeline can trigger one or more **child pipelines** (via `trigger: include:`), each with its own independent stages/jobs. This decomposes a massive monolithic pipeline into smaller, independently-visualized, independently-retriable pipelines — essential for monorepos or multi-component systems where a single giant pipeline becomes unmanageable and hard to debug.

**20. How does GitLab's Kubernetes integration (GitLab Agent) work for GitOps-style deployments?**
The GitLab Agent for Kubernetes runs inside the target cluster and establishes a secure, outbound-only connection back to GitLab (no inbound cluster access needed, which is a major security improvement over older certificate-based cluster integration). It supports both CI/CD-driven deployment *and* GitOps reconciliation, where the agent continuously syncs cluster state to match manifests defined in a Git repository, detecting and correcting drift automatically.

**21. 🎯 Scenario: A `.gitlab-ci.yml` deploy job has access to production AWS credentials as a CI/CD variable, and a developer's MR (not yet merged) modifies the pipeline file itself to print all environment variables. How do you prevent this from leaking your production secrets?**
Mark the secret variable as **protected**, which makes it available *only* on protected branches/tags — an unmerged feature branch MR pipeline does not have access to protected variables at all, regardless of what the pipeline YAML tries to print. Additionally, enable **masking** so even if accidentally echoed on an authorized branch, the value is redacted in logs. As a deeper control, use a short-lived OIDC-based cloud credential (GitLab supports OIDC for AWS/GCP/Azure) instead of long-lived static secrets, and restrict who can modify protected branches/`.gitlab-ci.yml` via CODEOWNERS and merge approval rules.

**22. What is the difference between GitLab's SAST, DAST, and Dependency Scanning, and how do they fit into a DevSecOps pipeline?**
- **SAST** (Static Application Security Testing): analyzes source code itself for vulnerability patterns without executing it (e.g., insecure function usage).
- **DAST** (Dynamic Application Security Testing): tests the *running* application (typically in a review/staging environment) by simulating attacks against live endpoints — catches runtime/config issues SAST can't see.
- **Dependency Scanning**: checks third-party libraries/packages against known CVE databases for vulnerable versions.
Together, these run as automatic stages in GitLab's Auto DevOps or custom pipelines, shifting security left into the CI process and surfacing results as MR widgets before code is merged.

**23. How would you architect GitLab Runner scaling for a high-throughput CI environment using Kubernetes executor?**
Deploy the GitLab Runner Helm chart configured with the **Kubernetes executor**, where each CI job spins up as an ephemeral pod in the cluster rather than a persistent VM. Combine this with cluster autoscaling (e.g., Karpenter or Cluster Autoscaler on EKS) so the underlying node pool scales out/in based on pending job pod demand. Use resource requests/limits per job to ensure fair scheduling, and consider a separate node pool with taints/tolerations for CI workloads to isolate them from production application pods.

---

## 4. Python (for DevOps)

### 🟢 Beginner

**1. Why is Python widely used in DevOps, despite Bash also being available?**
Python offers better readability, structured error handling (try/except), rich data structures (dicts, lists), and a massive ecosystem of libraries (`boto3` for AWS, `requests` for APIs, `paramiko` for SSH) — making it far better suited than Bash for complex logic, API integrations, and maintainable automation scripts beyond simple command chaining.

**2. What is the difference between a list and a tuple in Python?**
A list (`[]`) is mutable — you can add, remove, or change elements after creation. A tuple (`()`) is immutable — once created, its contents cannot change. Tuples are slightly faster and used when data shouldn't be modified (e.g., fixed config values).

**3. What is a dictionary in Python, and where would you use it in DevOps scripting?**
A dictionary stores key-value pairs (`{"name": "server1", "ip": "10.0.0.1"}`). In DevOps, it's commonly used to represent parsed JSON/YAML configs, API responses, or inventory data (e.g., mapping hostnames to IP addresses).

**4. How do you read a file in Python?**
```python
with open('config.txt', 'r') as f:
    content = f.read()
```
The `with` statement ensures the file is properly closed automatically, even if an exception occurs.

**5. What is the difference between `==` and `is` in Python?**
`==` checks value equality (do two objects have the same content?). `is` checks identity (are two variables pointing to the exact same object in memory?). For comparing values, always use `==`; `is` is mainly used for `None` checks (`if x is None`).

**6. How do you install third-party Python packages, and what is `pip`?**
`pip` is Python's package manager. Install with `pip install package-name`. Project dependencies are typically listed in a `requirements.txt` file and installed via `pip install -r requirements.txt`.

**7. What is a virtual environment, and why is it important?**
A virtual environment (`venv`) creates an isolated Python installation with its own packages, separate from the system Python and other projects. This prevents dependency version conflicts between different projects on the same machine.
```bash
python3 -m venv myenv
source myenv/bin/activate
```

**8. What is the difference between a function and a method in Python?**
A function is a standalone block of reusable code (`def my_func():`). A method is a function defined *inside a class*, called on an object instance (`obj.my_method()`) — it implicitly receives the object itself as the first argument (`self`).

**9. How do you handle exceptions in Python?**
```python
try:
    result = 10 / 0
except ZeroDivisionError as e:
    print(f"Error: {e}")
finally:
    print("This always runs")
```

**10. What is the purpose of `if __name__ == "__main__":` in Python scripts?**
It ensures that certain code only runs when the script is executed directly (not when it's imported as a module into another script) — a common pattern for making scripts both runnable and importable.

---

### 🟡 Intermediate

**11. How would you use Python's `subprocess` module to run a shell command and capture its output?**
```python
import subprocess
result = subprocess.run(['ls', '-la'], capture_output=True, text=True)
print(result.stdout)
print(result.returncode)
```
`subprocess.run` is preferred over the older `os.system()` because it gives structured access to stdout, stderr, and the exit code, and avoids shell injection risks when arguments are passed as a list rather than a single string.

**12. How do you parse a YAML file in Python, and why is this relevant to DevOps?**
```python
import yaml
with open('config.yaml') as f:
    data = yaml.safe_load(f)
```
This is core to DevOps because most infrastructure configs (Kubernetes manifests, Ansible playbooks, CI/CD pipelines, Docker Compose) are YAML — scripts that generate, validate, or template these files rely on this.

**13. What is the `boto3` library, and what's a basic example of using it?**
`boto3` is the official AWS SDK for Python, used to programmatically interact with AWS services.
```python
import boto3
ec2 = boto3.client('ec2')
instances = ec2.describe_instances()
for reservation in instances['Reservations']:
    for instance in reservation['Instances']:
        print(instance['InstanceId'], instance['State']['Name'])
```

**14. What is a list comprehension, and how does it improve on a traditional loop?**
```python
# Traditional loop
squares = []
for x in range(10):
    squares.append(x**2)

# List comprehension — more concise
squares = [x**2 for x in range(10)]
```
List comprehensions are more readable and often faster due to internal optimizations, but should be avoided when the logic becomes too complex to read on one line.

**15. How do you make an HTTP API call in Python, and how do you handle errors from it?**
```python
import requests
try:
    response = requests.get('https://api.example.com/health', timeout=5)
    response.raise_for_status()
    data = response.json()
except requests.exceptions.RequestException as e:
    print(f"API call failed: {e}")
```
`raise_for_status()` raises an exception for 4xx/5xx responses, which would otherwise silently succeed at the `requests.get()` level.

**16. What is the Global Interpreter Lock (GIL), and why does it matter for DevOps automation scripts?**
The GIL allows only one thread to execute Python bytecode at a time in CPython, meaning true CPU-bound parallelism via threading is limited. For DevOps scripts (mostly I/O-bound — API calls, SSH connections, file operations), this rarely matters since threads release the GIL during I/O waits; for CPU-bound work, `multiprocessing` (separate processes, separate GILs) is used instead of threading.

**17. How would you write a Python script to check if a list of servers are reachable (ping)?**
```python
import subprocess

servers = ['10.0.0.1', '10.0.0.2', '10.0.0.3']
for server in servers:
    result = subprocess.run(['ping', '-c', '1', '-W', '2', server],
                             stdout=subprocess.DEVNULL, stderr=subprocess.DEVNULL)
    status = "UP" if result.returncode == 0 else "DOWN"
    print(f"{server}: {status}")
```

**18. What is the difference between `*args` and `**kwargs` in a function definition?**
`*args` collects extra positional arguments into a tuple. `**kwargs` collects extra keyword arguments into a dictionary. Used when writing flexible functions/wrappers (common in decorators and automation tooling that wraps other functions).
```python
def deploy(*args, **kwargs):
    print(args)     # (val1, val2)
    print(kwargs)   # {'env': 'prod', 'dry_run': True}
deploy('val1', 'val2', env='prod', dry_run=True)
```

**19. How do you schedule a Python script to run periodically without using `cron`?**
Using the `schedule` library or `APScheduler` for in-process scheduling:
```python
import schedule, time
def job():
    print("Running health check...")
schedule.every(10).minutes.do(job)
while True:
    schedule.run_pending()
    time.sleep(1)
```
In production, `cron` or a workflow scheduler (Airflow, Jenkins) is usually preferred for reliability, since an in-process scheduler dies if the script crashes.

**20. What is the difference between `json.dumps()` and `json.dump()`?**
`json.dumps()` converts a Python object to a JSON-formatted *string* (returned in memory). `json.dump()` writes the JSON directly to a file object. Similarly `json.loads()` parses a string, while `json.load()` reads directly from a file.

---

### 🔴 Advanced

**21. How would you design a Python script that safely manages secrets (e.g., DB passwords) without hardcoding them?**
Never hardcode secrets in source. Use environment variables (`os.environ.get('DB_PASSWORD')`) loaded from a `.env` file (via `python-dotenv`) for local dev, but in production, fetch secrets at runtime from a secrets manager (AWS Secrets Manager, HashiCorp Vault, Azure Key Vault) using the appropriate SDK, with the script's IAM role/identity granting access — ensuring secrets never touch disk or version control, and can be rotated without code changes.

**22. Explain Python decorators and give a DevOps-relevant example (e.g., retry logic).**
A decorator wraps a function to extend its behavior without modifying its code. A common DevOps use case is automatic retry logic for flaky network calls:
```python
import time, functools

def retry(max_attempts=3, delay=2):
    def decorator(func):
        @functools.wraps(func)
        def wrapper(*args, **kwargs):
            for attempt in range(1, max_attempts + 1):
                try:
                    return func(*args, **kwargs)
                except Exception as e:
                    print(f"Attempt {attempt} failed: {e}")
                    if attempt == max_attempts:
                        raise
                    time.sleep(delay)
        return wrapper
    return decorator

@retry(max_attempts=5, delay=3)
def call_flaky_api():
    response = requests.get('https://flaky-service.com/health')
    response.raise_for_status()
    return response.json()
```

**23. 🎯 Scenario: You need to write a Python automation script that provisions 100 EC2 instances in parallel, but boto3's default `describe_instances`/`run_instances` calls are sequential and slow. How do you parallelize this safely?**
Use `concurrent.futures.ThreadPoolExecutor` since this is I/O-bound (waiting on AWS API responses), not CPU-bound:
```python
from concurrent.futures import ThreadPoolExecutor, as_completed
import boto3

def launch_instance(ec2_client, config):
    return ec2_client.run_instances(**config)

ec2 = boto3.client('ec2')
configs = [build_config(i) for i in range(100)]

with ThreadPoolExecutor(max_workers=10) as executor:
    futures = {executor.submit(launch_instance, ec2, cfg): cfg for cfg in configs}
    for future in as_completed(futures):
        try:
            result = future.result()
        except Exception as e:
            print(f"Failed to launch instance: {e}")
```
Important production considerations: respect AWS API rate limits (use a bounded `max_workers`, implement exponential backoff via `botocore`'s built-in retry config), and ensure idempotency (e.g., using client tokens) so a partial failure doesn't result in duplicate resources on retry.

**24. How does Python's `multiprocessing` differ from `threading`, and when would a DevOps engineer choose one over the other?**
`threading` uses multiple threads within a single process, sharing memory, but limited by the GIL for CPU-bound work — ideal for I/O-bound tasks (API calls, file I/O, SSH sessions). `multiprocessing` spawns separate OS processes, each with its own Python interpreter and memory space (bypassing the GIL), enabling true parallel CPU execution — used for CPU-heavy tasks like parsing/processing large log files or running computational data transformations across multiple cores.

**25. 🎯 Scenario: You're building a Python-based Infrastructure-as-Code validation tool that must parse Terraform plan JSON output and fail the CI pipeline if any resource is being destroyed in production. Outline your approach.**
```python
import json, sys

def check_destructive_changes(plan_file, environment):
    with open(plan_file) as f:
        plan = json.load(f)

    destructive = []
    for change in plan.get('resource_changes', []):
        actions = change['change']['actions']
        if 'delete' in actions:
            destructive.append(change['address'])

    if destructive and environment == 'production':
        print(f"BLOCKED: {len(destructive)} resource(s) will be destroyed in production:")
        for resource in destructive:
            print(f"  - {resource}")
        sys.exit(1)  # non-zero exit fails the CI job

    print("No destructive changes detected (or non-prod environment).")
    sys.exit(0)

if __name__ == "__main__":
    check_destructive_changes(sys.argv[1], sys.argv[2])
```
This is generated via `terraform show -json tfplan > plan.json` upstream in the pipeline, then this script gates the `apply` step — a common pattern for policy-as-code guardrails before native tools like OPA/Sentinel are introduced.

**26. What are Python type hints, and why do they matter for large-scale DevOps tooling maintained by multiple engineers?**
Type hints (`def deploy(env: str, replicas: int = 3) -> bool:`) document expected types without enforcing them at runtime (Python remains dynamically typed). In large internal tooling/codebases, they enable static analysis via `mypy`, better IDE autocomplete, and self-documenting function signatures — substantially reducing bugs and onboarding time in shared automation codebases used across a DevOps team.

**27. How would you implement structured logging in a Python automation tool meant to run in production and integrate with a log aggregator like Datadog or ELK?**
Avoid `print()` statements. Use the `logging` module configured to output structured JSON (so log aggregators can parse fields automatically):
```python
import logging, json_log_formatter

formatter = json_log_formatter.JSONFormatter()
handler = logging.StreamHandler()
handler.setFormatter(formatter)

logger = logging.getLogger('deploy-tool')
logger.addHandler(handler)
logger.setLevel(logging.INFO)

logger.info('deployment_started', extra={'environment': 'prod', 'version': 'v1.2.3'})
```
This outputs structured JSON logs to stdout, which container runtimes/log shippers (Fluentd, Datadog Agent) collect and forward — enabling searchable, filterable fields rather than unstructured text grepping.

---

## 5. Shell Scripting

### 🟢 Beginner

**1. What is a shebang (`#!/bin/bash`) and why is it needed at the top of a script?**
The shebang tells the operating system which interpreter to use to execute the script. `#!/bin/bash` specifies Bash; without it, the script may be run by the wrong shell, causing syntax errors if the script uses Bash-specific features.

**2. How do you make a shell script executable?**
```bash
chmod +x script.sh
./script.sh
```

**3. What is the difference between single quotes `' '` and double quotes `" "` in Bash?**
Single quotes preserve the literal value of everything inside (no variable expansion). Double quotes allow variable expansion (`$var`) and command substitution (`` `cmd` `` or `$(cmd)`) while still preserving spacing/special characters.

**4. How do you declare and use a variable in Bash?**
```bash
name="DevOps"
echo "Hello, $name"
```
Note: no spaces are allowed around `=` when assigning a variable in Bash.

**5. What is the difference between `$@` and `$*`?**
Both represent all positional arguments passed to a script. `"$@"` expands to each argument as a *separate* quoted string (preserves arguments with spaces correctly). `"$*"` expands to all arguments as a *single* string joined by the first character of `$IFS` (usually a space). `"$@"` is almost always the correct choice when looping over arguments.

**6. How do you read user input in a shell script?**
```bash
read -p "Enter your name: " username
echo "Hello, $username"
```

**7. What is the purpose of `exit` codes in shell scripts?**
Every command returns an exit code (0–255) upon completion; `0` means success, any non-zero value indicates an error. Scripts can check `$?` (the exit code of the last command) and explicitly `exit 1` to signal failure to whatever called the script (important for CI/CD pipelines, which treat non-zero exit as a failed step).

**8. How do you write an if-else condition in Bash?**
```bash
if [ "$status" == "running" ]; then
    echo "Service is up"
elif [ "$status" == "stopped" ]; then
    echo "Service is down"
else
    echo "Unknown status"
fi
```

**9. What is the difference between `[ ]` and `[[ ]]` in Bash conditionals?**
`[ ]` is the POSIX-standard test command, compatible with all shells (`sh`, `dash`), but more limited and prone to word-splitting/globbing issues with unquoted variables. `[[ ]]` is a Bash (and zsh/ksh) builtin keyword offering safer string comparisons, pattern matching (`==` with wildcards), and logical operators (`&&`, `||`) directly inside the brackets — generally preferred in Bash-specific scripts.

**10. How do you write a `for` loop in Bash to iterate over a list of files?**
```bash
for file in *.log; do
    echo "Processing $file"
done
```

---

### 🟡 Intermediate

**11. What is the difference between `>`, `>>`, and `2>` in shell redirection?**
`>` redirects stdout to a file, overwriting it. `>>` redirects stdout, appending to the file. `2>` redirects stderr specifically (file descriptor 2). To redirect both stdout and stderr to the same file: `command > output.log 2>&1`.

**12. What does `set -euo pipefail` do at the top of a script, and why is it considered best practice?**
- `set -e`: exits immediately if any command returns a non-zero exit code.
- `set -u`: treats use of an undefined variable as an error (instead of silently substituting empty string).
- `set -o pipefail`: makes a pipeline (`cmd1 | cmd2`) return the exit code of the *last command that failed*, not just the final command in the pipe — otherwise a failure in `cmd1` would be masked if `cmd2` succeeds.
Together, these make scripts "fail loudly" instead of silently continuing after an error — critical for reliable automation/CI scripts.

**13. What is command substitution, and what are the two syntaxes?**
Command substitution captures the output of a command into a variable.
```bash
current_dir=$(pwd)        # modern, preferred
current_dir=`pwd`         # legacy backtick syntax, avoid (harder to nest/read)
```

**14. How do you check if a file or directory exists in Bash?**
```bash
if [ -f "/etc/config.yaml" ]; then
    echo "File exists"
fi
if [ -d "/var/log/app" ]; then
    echo "Directory exists"
fi
```

**15. What's the difference between a shell function and a script, in terms of execution?**
A function runs within the *current* shell process — it can modify the calling shell's environment variables and is faster to invoke (no new process). A separate script, when executed (`./script.sh`), runs in a *new* subshell/process — changes to variables inside it do not affect the parent shell unless the script is "sourced" (`source script.sh` or `. script.sh`).

**16. How do you use `trap` in Bash, and what's a practical use case?**
`trap` lets you catch signals (like script exit, interrupt) and run cleanup code:
```bash
trap 'echo "Cleaning up temp files..."; rm -f /tmp/myapp_*' EXIT

echo "data" > /tmp/myapp_temp.txt
# ... rest of script
```
This ensures temp files are always removed, even if the script exits early due to an error — common in CI scripts that create temporary working directories.

**17. How would you parse command-line flags/arguments in a Bash script (e.g., `--env=prod --dry-run`)?**
```bash
while [[ $# -gt 0 ]]; do
    case $1 in
        --env=*)
            ENV="${1#*=}"
            shift
            ;;
        --dry-run)
            DRY_RUN=true
            shift
            ;;
        *)
            echo "Unknown option: $1"
            exit 1
            ;;
    esac
done
echo "Environment: $ENV, Dry run: $DRY_RUN"
```

**18. What is the difference between `kill`, `kill -9`, and `kill -15`?**
`kill` (default) sends `SIGTERM` (15) — a graceful termination request that the process can catch and handle (clean up, save state) before exiting. `kill -9` sends `SIGKILL`, which the OS forcibly terminates immediately — the process cannot intercept or clean up, used only as a last resort for unresponsive processes, since it can leave resources (locks, temp files) in an inconsistent state.

**19. How would you write a script to find and delete log files older than 7 days?**
```bash
find /var/log/myapp -name "*.log" -type f -mtime +7 -exec rm -f {} \;
```
`-mtime +7` matches files last modified more than 7 days ago.

**20. What is the difference between `awk`, `sed`, and `grep`, and when would you use each?**
- **grep**: searches/filters lines matching a pattern (find).
- **sed**: stream editor — primarily for find-and-replace and line-based text transformation.
- **awk**: a full pattern-scanning and processing language — best for column-based data extraction, calculations, and structured text reports.
Example: `grep "ERROR" app.log | awk '{print $1, $5}'` finds error lines, then extracts the 1st and 5th columns.

---

### 🔴 Advanced

**21. 🎯 Scenario: You have a production deployment script that must run multiple independent health checks against 10 different microservices, but running them sequentially takes too long. How do you parallelize this safely in Bash?**
```bash
#!/bin/bash
set -euo pipefail

services=("auth" "payments" "orders" "inventory" "notifications")
pids=()

for svc in "${services[@]}"; do
    (
        if curl -sf "http://$svc.internal/health" > /dev/null; then
            echo "$svc: HEALTHY"
        else
            echo "$svc: UNHEALTHY"
            exit 1
        fi
    ) &
    pids+=($!)
done

failed=0
for pid in "${pids[@]}"; do
    wait "$pid" || failed=1
done

if [ "$failed" -eq 1 ]; then
    echo "One or more services failed health checks"
    exit 1
fi
echo "All services healthy"
```
This backgrounds each check (`&`), tracks PIDs, and `wait`s for all of them, aggregating failures — turning a 10×sequential check into a parallel check bounded by the slowest single service.

**22. What is the difference between a subshell `( )` and a command group `{ }` in Bash?**
A subshell `( commands )` runs in a *forked child process* — variable changes, `cd`, and `exit` inside it do not affect the parent shell. A command group `{ commands; }` runs in the *current shell* — changes persist in the calling shell's environment. Subshells are useful for isolating side effects (e.g., temporarily `cd`-ing without affecting the rest of the script); command groups are useful for grouping commands to apply a single redirection or `&&`/`||` to all of them at once.

**23. How does Bash's process substitution `<( )` work, and what problem does it solve?**
Process substitution lets you treat the *output of a command* as if it were a file, without creating an actual temp file:
```bash
diff <(curl -s http://service-a/version) <(curl -s http://service-b/version)
```
This compares the live output of two commands directly, which would otherwise require manually writing each to a temp file first, comparing, then cleaning up — process substitution does this implicitly via `/dev/fd/` file descriptors.

**24. 🎯 Scenario: A cron job running a backup script silently fails every few weeks, with no clear error in the logs. How would you debug and harden this script?**
1. Add `set -euxo pipefail` at the top — the `-x` flag prints every command before executing it, giving a full execution trace in logs.
2. Ensure `PATH` is explicitly set at the top of the script — cron runs with a minimal environment, and scripts that rely on commands found via an interactive shell's `PATH` often silently fail under cron because the binary isn't found.
3. Redirect all output (stdout + stderr) explicitly to a timestamped log file rather than relying on cron's mail (`* * * * * /script.sh >> /var/log/backup_$(date +\%Y\%m\%d).log 2>&1`).
4. Add explicit exit-code checks after critical commands and `trap ERR` to capture exactly which line failed.
5. Consider replacing fragile shell logic with a more robust tool (or a monitored systemd timer with `OnFailure=` triggering an alert) so failures are proactively surfaced rather than silently discovered weeks later.

**25. Explain how Bash handles word splitting and globbing, and why unquoted variables are a common source of bugs/security issues.**
When a variable is unquoted (`$myvar` instead of `"$myvar"`), Bash performs word splitting (breaking it into multiple words on `$IFS` characters, typically spaces) and filename globbing (expanding `*`, `?` if present) on its value *before* passing it to a command. This means a variable containing a filename with a space (`my file.txt`) gets split into two separate arguments, or a value containing `*` could unexpectedly expand to a list of files — a classic source of bugs and even injection vulnerabilities in scripts that build commands from user/external input. Best practice: always quote variable expansions (`"$myvar"`) unless you specifically need splitting/globbing.

**26. How would you write an idempotent shell script (safe to run multiple times without unintended side effects) — give an example pattern.**
Idempotency means re-running the script produces the same end state, without errors or duplication, regardless of how many times it's run.
```bash
# Idempotent user creation
if ! id "appuser" &>/dev/null; then
    useradd appuser
fi

# Idempotent directory creation
mkdir -p /opt/myapp/data   # -p doesn't error if dir already exists

# Idempotent line-in-file
grep -qxF "export PATH=/opt/myapp/bin:\$PATH" ~/.bashrc || \
    echo "export PATH=/opt/myapp/bin:\$PATH" >> ~/.bashrc
```
This pattern (check-before-act, or use inherently idempotent flags like `-p`) is essential for provisioning scripts that may run during repeated deployments or configuration management runs.

**27. What is the performance and reliability difference between writing complex logic in Bash versus calling out to Python/Go from within a Bash wrapper, in an enterprise CI/CD context?**
Bash excels at orchestrating commands and simple control flow but lacks robust data structures, real error handling (try/except), unit testing frameworks, and becomes exponentially harder to maintain and debug as complexity grows (e.g., nested loops with associative arrays, complex JSON parsing). Enterprise teams commonly use Bash only as a thin orchestration layer (sourcing env vars, calling tools in sequence) while delegating any non-trivial logic (parsing, validation, API orchestration, retries) to Python or Go binaries — striking a balance between Bash's lightweight ubiquity (available everywhere, no runtime needed) and the maintainability of a proper programming language for complex logic.

---

## 6. Docker

### 🟢 Beginner

**1. What is Docker, and what problem does it solve?**
Docker is a containerization platform that packages an application along with all its dependencies (libraries, runtime, config) into a single, portable unit called a container. It solves the "it works on my machine" problem by guaranteeing the exact same environment runs identically across development, testing, and production.

**2. What is the difference between a container and a virtual machine (VM)?**
A VM virtualizes an entire machine, including its own OS kernel, running on a hypervisor — heavy (GBs), slow to boot (minutes). A container shares the host machine's OS kernel and only packages the application and its dependencies — lightweight (MBs), starts in seconds, and far more resource-efficient since it doesn't duplicate the OS.

**3. What is a Docker image?**
A read-only template containing the application code, runtime, libraries, and configuration needed to run a container. Images are built from a `Dockerfile` and stored in layers; a container is a running (writable) instance of an image.

**4. What is a Dockerfile?**
A text file containing a sequence of instructions (`FROM`, `RUN`, `COPY`, `CMD`, etc.) that Docker reads to automatically build an image.

**5. What are the basic Docker CLI commands to build and run a container?**
```bash
docker build -t myapp:1.0 .
docker run -d -p 8080:80 --name myapp-container myapp:1.0
docker ps
docker stop myapp-container
docker logs myapp-container
```

**6. What is the difference between `CMD` and `ENTRYPOINT` in a Dockerfile?**
`CMD` specifies default arguments/command for the container, which can be easily overridden at `docker run` time. `ENTRYPOINT` specifies the fixed executable that always runs; any `CMD` or runtime arguments are appended to it as parameters. They're often combined: `ENTRYPOINT ["python", "app.py"]` with `CMD ["--port", "8080"]` as default arguments.

**7. What is the purpose of the `-d` flag in `docker run`?**
`-d` runs the container in detached mode (in the background), returning control to the terminal immediately instead of attaching to the container's output stream.

**8. What is a Docker volume, and why is it needed?**
A volume is a persistent storage mechanism managed by Docker, used to store data that should survive beyond a container's lifecycle (container filesystems are ephemeral — deleted when the container is removed). Volumes are also used to share data between containers or between the host and a container.

**9. What is the difference between `docker stop` and `docker kill`?**
`docker stop` sends `SIGTERM` first (graceful shutdown, allowing the app to clean up), then `SIGKILL` after a grace period (default 10s) if it hasn't exited. `docker kill` sends `SIGKILL` immediately, forcibly terminating the container without any graceful shutdown opportunity.

**10. What is Docker Hub?**
A public cloud registry for storing and sharing Docker images — the default registry Docker pulls from unless configured otherwise (e.g., `docker pull nginx` pulls from Docker Hub).

---

### 🟡 Intermediate

**11. Explain Docker image layers and how caching works during a build.**
Each instruction in a Dockerfile creates a new read-only layer, stacked on top of previous layers. Docker caches each layer; if a layer's instruction and its inputs haven't changed since the last build, Docker reuses the cached layer instead of rebuilding it. This is why the *order* of Dockerfile instructions matters — placing rarely-changing instructions (like installing dependencies) before frequently-changing ones (like copying source code) maximizes cache hits and speeds up builds.

**12. What is a multi-stage build, and why is it used?**
A multi-stage build uses multiple `FROM` statements in a single Dockerfile, where each stage can use a different base image, and you selectively copy artifacts from one stage into another using `COPY --from=stage_name`. This lets you use a heavy build environment (with compilers, build tools) in an early stage, then copy only the final compiled artifact into a minimal runtime image — drastically reducing final image size and attack surface.
```dockerfile
FROM golang:1.21 AS builder
WORKDIR /app
COPY . .
RUN go build -o myapp

FROM alpine:3.18
COPY --from=builder /app/myapp /usr/local/bin/myapp
CMD ["myapp"]
```

**13. What is the difference between `COPY` and `ADD` in a Dockerfile?**
`COPY` simply copies files/directories from the build context into the image. `ADD` does everything `COPY` does, plus it can automatically extract local tar archives and fetch files from remote URLs. Docker's official best practice is to prefer `COPY` for clarity/predictability and only use `ADD` when its extra extraction behavior is specifically needed.

**14. How does Docker networking work — explain bridge, host, and none network drivers.**
- **bridge** (default): creates an isolated internal network; containers get their own IP and communicate via NAT through the host, with explicit port mapping (`-p`) needed for external access.
- **host**: the container shares the host's network namespace directly — no isolation, no port mapping needed, but less secure and can't run multiple containers on the same port.
- **none**: the container has no network access at all — used for fully isolated, network-restricted workloads.

**15. What is the difference between `ENV` and `ARG` in a Dockerfile?**
`ARG` defines a build-time variable, only available during the image build process (not in the running container, unless explicitly passed to `ENV`). `ENV` defines an environment variable available both during the build *and* persisted into the running container at runtime.

**16. How do you reduce Docker image size? List several techniques.**
Use a minimal base image (`alpine`, `distroless`, or `scratch`); combine `RUN` commands with `&&` to reduce layer count; clean up package manager caches in the same `RUN` layer they were created (`apt-get clean && rm -rf /var/lib/apt/lists/*`); use multi-stage builds to exclude build tools from the final image; use `.dockerignore` to exclude unnecessary files from the build context; and avoid installing unnecessary debugging tools in production images.

**17. What is a `.dockerignore` file used for?**
Similar to `.gitignore`, it excludes specified files/directories from being sent to the Docker daemon as part of the build context — speeding up builds and preventing sensitive files (`.env`, `.git`) from accidentally being copied into the image.

**18. What is the difference between `docker-compose up` and `docker-compose up -d`?**
Without `-d`, Compose runs in the foreground, streaming all container logs to the terminal (Ctrl+C stops everything). With `-d` (detached), containers start in the background and control returns immediately to the terminal.

**19. How does Docker handle persistent data — explain named volumes vs bind mounts.**
A **named volume** is fully managed by Docker, stored in Docker's own managed area on the host (`/var/lib/docker/volumes/`), portable and decoupled from host directory structure — the recommended approach for most production use cases. A **bind mount** maps a specific host filesystem path directly into the container — useful for local development (live code editing) but tightly couples the container to the host's exact filesystem layout, less portable.

**20. What is the purpose of `HEALTHCHECK` in a Dockerfile?**
It defines a command Docker periodically runs inside the container to determine if the application is actually healthy (not just running). Orchestrators like Docker Swarm/Kubernetes use this status to decide whether to route traffic to or restart the container.
```dockerfile
HEALTHCHECK --interval=30s --timeout=3s CMD curl -f http://localhost/health || exit 1
```

---

### 🔴 Advanced

**21. Explain how Docker achieves isolation using Linux namespaces and cgroups.**
Docker uses Linux **namespaces** to provide isolated views of system resources per container — PID namespace (isolated process tree), network namespace (isolated network stack/interfaces), mount namespace (isolated filesystem view), UTS namespace (isolated hostname), and user namespace (isolated user/group IDs). **cgroups** (control groups) enforce resource *limits* (CPU, memory, I/O) per container, preventing one container from starving others on the same host. Together, namespaces provide isolation (what a process can see) while cgroups provide resource governance (what a process can consume).

**22. 🎯 Scenario: A container is consuming excessive memory and getting OOM-killed unpredictably in production. How do you diagnose and fix this?**
1. Check current resource usage and limits: `docker stats` for live metrics, and inspect configured limits with `docker inspect <container> --format='{{.HostConfig.Memory}}'`.
2. If no memory limit was set, the container could consume unbounded host memory — set explicit limits: `docker run -m 512m --memory-swap 512m ...` (disabling swap prevents thrashing).
3. Check `dmesg` or `journalctl -k` on the host for OOM-killer log entries confirming which process was killed and why.
4. Profile the application itself for memory leaks (e.g., using language-specific profilers) — a resource limit only masks the symptom; if the app genuinely leaks memory, it will eventually hit any limit.
5. In Kubernetes, distinguish between `requests` (scheduling guarantee) and `limits` (hard ceiling, triggers OOMKill if exceeded) and ensure the limit reflects realistic peak usage with headroom, set via load testing.

**23. What is the security risk of running a Docker container as root, and how do you mitigate it?**
By default, processes inside a container run as root *within the container's user namespace*, but without user namespace remapping, the container's root UID 0 maps directly to the host's root UID 0. If an attacker breaks out of the container (via a kernel vulnerability or container escape), they'd have root privileges on the host itself. Mitigations: explicitly run as a non-root user in the Dockerfile (`USER appuser`), enable Docker's user namespace remapping (`--userns-remap`) so container root maps to an unprivileged host UID, and use `--security-opt=no-new-privileges` plus drop unnecessary Linux capabilities (`--cap-drop=ALL --cap-add=<only what's needed>`).

**24. 🎯 Scenario: Your CI pipeline builds a Docker image on every commit, but builds are taking 8+ minutes due to repeated dependency downloads. How do you optimize build caching in a CI environment where each build may run on a fresh/ephemeral runner?**
Since ephemeral CI runners don't have a local Docker build cache from previous runs, use **registry-based cache** with BuildKit:
```bash
docker buildx build \
  --cache-from=type=registry,ref=myregistry/myapp:buildcache \
  --cache-to=type=registry,ref=myregistry/myapp:buildcache,mode=max \
  -t myregistry/myapp:latest --push .
```
This pushes/pulls cache layers to/from a remote registry, so even a brand-new runner can warm its build cache from the registry. Additionally, restructure the Dockerfile to copy dependency manifests (`package.json`, `requirements.txt`) and install dependencies in an early layer *before* copying the full source code, maximizing cache hits on the (usually unchanged) dependency-install layer across commits that only change application code.

**25. Explain the difference between Docker's overlay2 storage driver and how copy-on-write (CoW) affects container performance.**
`overlay2` (the modern default storage driver) implements a union filesystem where the read-only image layers are stacked, and the container gets a thin writable layer on top. When a container modifies a file that exists in a lower (read-only) layer, the *entire file* is copied up to the writable layer first (copy-on-write), then modified — this means workloads that frequently modify large files can suffer performance overhead from repeated copy-up operations. For write-heavy workloads (e.g., databases), it's strongly recommended to use a Docker volume (bypassing the union filesystem entirely for that path) rather than writing directly to the container's writable layer.

**26. How does Docker Content Trust (DCT) work, and why does it matter for supply chain security?**
Docker Content Trust uses Notary (based on The Update Framework, TUF) to cryptographically sign images at push time and verify signatures at pull time (`DOCKER_CONTENT_TRUST=1`). This guarantees that an image pulled is exactly what the publisher signed and pushed — unmodified and from a verified source — protecting against man-in-the-middle tampering or a compromised/malicious registry serving a backdoored image, a critical control in supply-chain-security-conscious enterprise pipelines.

**27. 🎯 Scenario: You need to run a legacy application inside Docker that requires a specific kernel module not available in the container's isolated environment. What are your options?**
Containers share the host kernel — they cannot load their own separate kernel modules, since kernel modules operate below the container isolation boundary. Options: (1) ensure the required kernel module is loaded on the **host** itself before the container starts (containers can then use it via shared kernel access, e.g., `--privileged` or specific `--device` mounts if it's a device-level dependency); (2) if the workload fundamentally requires a different kernel or kernel-level isolation, it may not be a good fit for standard containers — consider a lightweight VM (Firecracker, gVisor for partial syscall-level isolation, or a full VM) instead; (3) re-architect the application to remove the kernel-module dependency if feasible, since relying on host kernel state is itself an anti-pattern that breaks container portability.

**28. What is the difference between Docker Swarm and Kubernetes, and in what scenario might Swarm still be a reasonable choice today?**
Both orchestrate containers across multiple hosts, but Kubernetes is far more feature-rich (advanced scheduling, auto-scaling, vast ecosystem of operators/CRDs, multi-cloud support) at the cost of significant operational complexity. Docker Swarm is simpler to set up and operate (built into the Docker CLI, minimal learning curve) but has a much smaller feature set and ecosystem, and its long-term development/community momentum has significantly slowed compared to Kubernetes. Swarm can still be a reasonable choice for small teams/simple deployments where Kubernetes' complexity isn't justified, but for any serious production scale, multi-team, or cloud-native ecosystem integration, Kubernetes is the industry-standard choice today.

---

## 7. Kubernetes

### 🟢 Beginner

**1. What is Kubernetes, and what problem does it solve?**
Kubernetes (k8s) is a container orchestration platform that automates the deployment, scaling, networking, and management of containerized applications across a cluster of machines. It solves the problem of running containers reliably at scale — handling failures, scaling up/down based on demand, load balancing, rolling updates, and self-healing — which becomes unmanageable to do manually beyond a handful of containers.

**2. What is a Pod in Kubernetes?**
The smallest deployable unit in Kubernetes — a Pod wraps one or more tightly-coupled containers that share the same network namespace (same IP, can communicate via `localhost`) and storage volumes. Most commonly, a Pod runs a single container.

**3. What is the difference between a Pod and a container?**
A container is a single packaged application instance. A Pod is Kubernetes' wrapper around one or more containers, adding a shared network/storage context and a Kubernetes-managed lifecycle (scheduling, restarts, health checks at the Pod level).

**4. What is a Node in Kubernetes?**
A worker machine (physical or virtual) in the cluster that runs Pods. Each Node runs essential components: the `kubelet` (agent communicating with the control plane), a container runtime (e.g., containerd), and `kube-proxy` (networking).

**5. What is the Kubernetes control plane?**
The set of components managing the overall state of the cluster: `kube-apiserver` (the front-end API), `etcd` (the cluster's key-value data store), `kube-scheduler` (assigns Pods to Nodes), and `kube-controller-manager` (runs controllers that reconcile desired vs actual state).

**6. What is a Deployment in Kubernetes?**
A higher-level object that manages a set of identical Pods (via a ReplicaSet), providing declarative updates, rolling updates/rollbacks, and self-healing (automatically replacing failed Pods to maintain the desired replica count).

**7. What is a Service in Kubernetes, and why is it needed?**
Since Pods are ephemeral (they get new IPs when recreated), a Service provides a stable, persistent network endpoint (virtual IP + DNS name) that load-balances traffic across a dynamic set of matching Pods (selected via labels) — decoupling clients from the constantly-changing underlying Pod IPs.

**8. What is the difference between `ClusterIP`, `NodePort`, and `LoadBalancer` Service types?**
- **ClusterIP** (default): exposes the Service only within the cluster, on an internal virtual IP.
- **NodePort**: exposes the Service on a static port on every Node's IP, accessible from outside the cluster.
- **LoadBalancer**: provisions an external cloud load balancer (on supported cloud providers) that routes external traffic to the Service.

**9. What is a Namespace in Kubernetes?**
A virtual cluster-within-a-cluster used to logically isolate and organize resources (e.g., separate `dev`, `staging`, `prod` namespaces), supporting separate resource quotas, access controls, and avoiding naming collisions between teams/environments.

**10. What is `kubectl`, and what are some basic commands?**
`kubectl` is the command-line tool for interacting with the Kubernetes API server.
```bash
kubectl get pods
kubectl describe pod my-pod
kubectl logs my-pod
kubectl apply -f deployment.yaml
kubectl delete pod my-pod
```

**11. What is a YAML manifest in Kubernetes?**
A declarative configuration file describing the desired state of a Kubernetes resource (Pod, Deployment, Service, etc.) — applied via `kubectl apply -f file.yaml`, after which Kubernetes continuously reconciles the actual cluster state to match it.

---

### 🟡 Intermediate

**12. What is a ReplicaSet, and how does it relate to a Deployment?**
A ReplicaSet ensures a specified number of identical Pod replicas are running at all times, replacing any that fail or are deleted. A Deployment manages ReplicaSets on your behalf — when you update a Deployment (e.g., new image version), it creates a new ReplicaSet and gradually shifts Pods from the old ReplicaSet to the new one (rolling update), while retaining old ReplicaSets for rollback capability.

**13. What is the difference between a Deployment and a StatefulSet?**
A Deployment treats all Pod replicas as interchangeable/stateless — they get random names and any Pod can be replaced by any other. A StatefulSet is used for stateful applications (databases, message queues) where Pods need stable, unique network identities (predictable naming like `pod-0`, `pod-1`), stable persistent storage tied to each specific Pod, and ordered, graceful deployment/scaling/termination.

**14. What is a DaemonSet, and when would you use one?**
A DaemonSet ensures exactly one copy of a Pod runs on every (or a selected subset of) Node in the cluster — used for node-level infrastructure agents like log collectors (Fluentd), monitoring agents (Datadog/Prometheus Node Exporter), or network plugins, which need to run on every node regardless of cluster scaling.

**15. What are Liveness and Readiness probes, and how do they differ?**
A **Liveness probe** checks if a container is still alive/functioning; if it fails, Kubernetes kills and restarts the container. A **Readiness probe** checks if a container is ready to receive traffic; if it fails, the Pod is removed from Service endpoints (no traffic routed to it) but is *not* restarted — useful for temporary states like warming up a cache, where restarting wouldn't help.

**16. What is a ConfigMap, and how does it differ from a Secret?**
A ConfigMap stores non-sensitive configuration data (key-value pairs) that can be injected into Pods as environment variables or mounted files. A Secret is structurally similar but intended for sensitive data (passwords, tokens, keys) — it's base64-encoded (not encrypted by default at the API level, though encryption-at-rest can be enabled in etcd) and has additional access restrictions.

**17. What is a PersistentVolume (PV) and a PersistentVolumeClaim (PVC)?**
A PersistentVolume is a piece of actual storage in the cluster (provisioned by an admin or dynamically via a StorageClass). A PersistentVolumeClaim is a request for storage by a user/Pod, specifying size and access mode — Kubernetes binds a matching PV to the PVC, and the Pod references the PVC (abstracting away the underlying storage implementation).

**18. What is an Ingress, and how does it differ from a Service of type LoadBalancer?**
An Ingress is an API object that manages external HTTP/HTTPS access to Services within the cluster, supporting host/path-based routing, SSL termination, and virtual hosting — all through a *single* external load balancer/entry point (managed by an Ingress Controller like NGINX or Traefik). Using a `LoadBalancer` Service per application, by contrast, provisions a *separate* expensive cloud load balancer for each Service, which doesn't scale well or support advanced routing rules.

**19. What are Kubernetes Labels and Selectors, and how are they used?**
Labels are arbitrary key-value pairs attached to objects (e.g., `app: frontend`, `env: prod`) for identification. Selectors query objects by their labels — this is the core mechanism Services use to find which Pods to route traffic to, and Deployments use to know which Pods belong to them.

**20. What is the difference between `kubectl apply` and `kubectl create`?**
`kubectl create` imperatively creates a resource — running it again on an existing resource throws an error (resource already exists). `kubectl apply` is declarative — it creates the resource if it doesn't exist, or intelligently updates it to match the YAML if it does, making it idempotent and safe to run repeatedly (the standard approach for GitOps/CI-driven deployments).

**21. What is a rolling update, and how do `maxSurge` and `maxUnavailable` control it?**
A rolling update gradually replaces old Pods with new ones with zero downtime. `maxSurge` controls how many *extra* Pods (above the desired replica count) can be created during the update. `maxUnavailable` controls how many Pods can be unavailable (below the desired count) during the update. Tuning these controls the trade-off between update speed and resource headroom/availability during the rollout.

**22. How do you check the resource usage (CPU/memory) of Pods in a cluster?**
```bash
kubectl top pods
kubectl top nodes
```
(Requires the Metrics Server to be installed in the cluster.)

---

### 🔴 Advanced

**23. Explain the full reconciliation loop Kubernetes uses, and why it's described as "level-triggered" rather than "edge-triggered."**
Every Kubernetes controller continuously watches the actual state of resources and compares it against the *desired* state declared in the API (etcd). If they differ, the controller takes action to reconcile them — and crucially, this loop runs continuously/repeatedly, not just once in response to a single change event. This makes it "level-triggered": even if an event is missed or the controller restarts, on its next reconciliation pass it will still detect and correct any drift, simply by comparing current state to desired state — unlike "edge-triggered" systems that only react to a specific transition and can permanently miss a state change if the triggering event is lost.

**24. 🎯 Scenario: A Pod is stuck in `CrashLoopBackOff`. Walk through your full debugging process.**
1. `kubectl describe pod <pod-name>` — check the Events section for scheduling issues, image pull errors, or OOMKilled status.
2. `kubectl logs <pod-name>` — check the current container's logs for an application-level error/stack trace.
3. `kubectl logs <pod-name> --previous` — critically, check the logs of the *previous* (crashed) instance, since the current container may have just restarted with no logs yet.
4. Check resource limits — if the container is being OOMKilled, `describe` will show `Reason: OOMKilled`; increase memory limits or fix the underlying memory leak.
5. Check the Liveness probe configuration — an overly aggressive probe (too short initial delay/timeout) can kill a legitimately slow-starting application before it's ready, causing a crash loop that isn't actually an application bug.
6. If it's a config/secret issue, verify mounted ConfigMaps/Secrets exist and have correct keys (`kubectl exec` into a working sibling Pod, or check `describe` for mount errors).

**25. What is a Custom Resource Definition (CRD), and how does the Operator pattern extend Kubernetes?**
A CRD lets you define your own custom Kubernetes resource type (beyond built-ins like Pod/Deployment), extending the Kubernetes API itself. An **Operator** pairs a CRD with a custom controller that encodes operational/domain-specific knowledge to manage that resource's full lifecycle automatically — for example, a `PostgresCluster` CRD with an Operator that handles automated backups, failover, and version upgrades, effectively codifying a human DBA's runbook into a continuously-running automated controller.

**26. 🎯 Scenario: Your application experiences sudden traffic spikes 3x normal load during specific hours, and you need it to scale automatically without manual intervention. Design the solution.**
Use a **Horizontal Pod Autoscaler (HPA)** targeting the Deployment, scaling based on CPU/memory utilization or custom metrics (e.g., requests-per-second via a metrics adapter like Prometheus Adapter):
```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: frontend-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: frontend
  minReplicas: 3
  maxReplicas: 30
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
```
If the underlying Nodes themselves also need to scale (not just Pods within existing capacity), pair this with a **Cluster Autoscaler** (or Karpenter on AWS) to add/remove Nodes based on pending unschedulable Pods. Also set appropriate `requests`/`limits` on the Pod spec so the scheduler and HPA make accurate decisions, and consider predictive/scheduled scaling (e.g., KEDA cron scaler) if the spike timing is known in advance, to pre-warm capacity before the HPA's reactive metric-based scaling would otherwise kick in.

**27. Explain how Kubernetes networking works at a low level — how does a Pod on Node A communicate with a Pod on Node B?**
Kubernetes mandates a flat networking model where every Pod gets its own unique cluster-wide IP, and all Pods can communicate with all other Pods without NAT, regardless of which Node they're on. This is implemented by a **CNI (Container Network Interface) plugin** (Calico, Cilium, Flannel, etc.), which sets up the actual networking — commonly using either an overlay network (encapsulating Pod traffic in a tunnel like VXLAN to route across the underlying physical network) or, in more advanced setups, direct BGP-based routing of Pod CIDRs between Nodes without encapsulation overhead. `kube-proxy` on each Node additionally programs iptables/IPVS rules to implement Service virtual IPs, transparently load-balancing traffic to the correct backend Pod IPs.

**28. 🎯 Scenario: You need to ensure a critical Pod is never scheduled on the same Node as another specific Pod (e.g., two replicas of a database for high availability), and is preferably spread across different Availability Zones. How do you configure this?**
Use **Pod Anti-Affinity** with `topologyKey`:
```yaml
affinity:
  podAntiAffinity:
    requiredDuringSchedulingIgnoredDuringExecution:
    - labelSelector:
        matchExpressions:
        - key: app
          operator: In
          values:
          - database
      topologyKey: "kubernetes.io/hostname"   # never same Node
    preferredDuringSchedulingIgnoredDuringExecution:
    - weight: 100
      podAffinityTerm:
        labelSelector:
          matchExpressions:
          - key: app
            operator: In
            values:
            - database
        topologyKey: "topology.kubernetes.io/zone"   # prefer different AZ
```
`requiredDuringScheduling` is a hard constraint (scheduling fails if it can't be satisfied); `preferredDuringScheduling` is a soft constraint (best-effort).

**29. What is the difference between `requests` and `limits` in Pod resource specifications, and how do they affect the Kubernetes scheduler and QoS classes?**
`requests` is the guaranteed minimum amount of CPU/memory a container needs — the scheduler uses this value to decide which Node has sufficient capacity to place the Pod. `limits` is the hard maximum a container is allowed to consume — exceeding the memory limit triggers an OOMKill; exceeding the CPU limit results in throttling (not killing). Based on how `requests` and `limits` are set, Kubernetes assigns a **QoS class**: `Guaranteed` (requests == limits for all containers — highest eviction priority protection), `Burstable` (requests < limits — some protection), or `BestEffort` (no requests/limits set — first to be evicted under Node resource pressure).

**30. 🎯 Scenario: A rolling deployment caused a production outage because the new version had a bug, and by the time it was noticed, 100% of Pods had already rolled over. How would you architect deployments to prevent this in the future?**
1. Implement a **Canary deployment** strategy (via a service mesh like Istio, or tools like Argo Rollouts/Flagger) that routes only a small percentage of traffic (e.g., 5%) to the new version initially, automatically analyzing error rates/latency metrics, and only progressively shifting more traffic if the canary remains healthy — automatically rolling back if metrics degrade.
2. Ensure robust **Readiness probes** that genuinely validate application health (not just "process is running") so a broken Pod never receives traffic in the first place.
3. Tune `maxUnavailable: 0` and a conservative `maxSurge` for standard rolling updates of critical services, ensuring old healthy Pods aren't removed until new Pods are confirmed ready.
4. Integrate automated smoke tests / synthetic monitoring as a required gate immediately post-deployment, before the rollout is allowed to continue past an initial small batch.
5. Maintain fast, well-tested rollback procedures (`kubectl rollout undo`) as a fallback, and ensure alerting on key SLIs (error rate, latency) is sensitive and fast enough to catch regressions within seconds/minutes, not after full rollout completion.

---

## 8. Jenkins

### 🟢 Beginner

**1. What is Jenkins, and what is it primarily used for?**
Jenkins is an open-source automation server used primarily for building CI/CD (Continuous Integration/Continuous Delivery) pipelines — automating the build, test, and deployment of software whenever code changes occur.

**2. What is the difference between Continuous Integration (CI) and Continuous Deployment/Delivery (CD)?**
CI is the practice of frequently merging code changes into a shared repository, with each merge automatically built and tested to catch integration issues early. Continuous Delivery automates the release process up to production, requiring a manual approval to actually deploy. Continuous Deployment goes further — every change that passes all tests is automatically deployed to production with no manual gate.

**3. What is a Jenkins job/project?**
A configured unit of work in Jenkins — defining what to build, how (source code location, build steps), and when (triggers like SCM polling, webhooks, or schedules).

**4. What is a Jenkins plugin, and why is the plugin ecosystem important?**
Plugins extend Jenkins' core functionality — adding support for specific source control systems, build tools, cloud providers, notification systems, and more. Jenkins' core is intentionally minimal; nearly all real-world functionality (Git integration, Docker support, Slack notifications) comes through its extensive plugin ecosystem (1800+ plugins).

**5. What is a Jenkinsfile?**
A text file (checked into source control alongside the application code) that defines a Jenkins Pipeline as code, using either Declarative or Scripted Pipeline syntax — enabling version-controlled, reviewable, reproducible CI/CD pipeline definitions instead of manually configuring jobs through the UI.

**6. What is the difference between Freestyle projects and Pipeline projects in Jenkins?**
Freestyle projects are configured entirely through the Jenkins web UI — simple but not version-controlled and hard to scale/reuse. Pipeline projects are defined as code (Jenkinsfile), version-controlled with the application, support complex logic, and are the modern recommended approach.

**7. What is a Jenkins agent (formerly called "slave")?**
A machine (physical, VM, or container) that connects to the Jenkins controller (master) and executes the actual build/job work, allowing builds to be distributed across multiple machines rather than all running on a single controller.

**8. How do you trigger a Jenkins build automatically when code is pushed to a Git repository?**
Configure a webhook on the Git hosting platform (GitHub/GitLab) pointing to the Jenkins server's webhook endpoint, combined with the "GitHub hook trigger for GITScm polling" option in the job config — Jenkins receives the push event and starts the build immediately, rather than relying on periodic polling.

---

### 🟡 Intermediate

**9. What is the difference between Declarative and Scripted Pipeline syntax in Jenkins?**
**Declarative Pipeline** uses a structured, opinionated syntax (`pipeline { agent { } stages { } }`) that's easier to read, write, and validate, with built-in support for directives like `post`, `when`, and `parameters`. **Scripted Pipeline** uses full Groovy code within a `node { }` block, offering maximum flexibility and programmatic control but requiring more Groovy knowledge and being harder to maintain. Declarative is recommended for most use cases; Scripted is used when you need logic beyond what Declarative directives support.

**10. What is the purpose of the `agent` directive in a Jenkinsfile?**
It specifies *where* (which executor/node) the Pipeline or a specific stage should run — e.g., `agent any` (any available agent), `agent { label 'docker' }` (a specific labeled agent), or `agent { docker 'node:18' }` (run inside a specific Docker container).

**11. What is a Jenkins shared library, and why is it used?**
A shared library is a separate Git repository containing reusable Groovy code (common pipeline steps, functions) that multiple Jenkinsfiles across different projects can import and call. This avoids duplicating complex pipeline logic across dozens of repositories — promoting DRY, centrally-maintained CI/CD standards across an organization.

**12. How do you pass parameters to a Jenkins Pipeline, and how do you access them in the Jenkinsfile?**
```groovy
pipeline {
    agent any
    parameters {
        string(name: 'ENVIRONMENT', defaultValue: 'staging', description: 'Target environment')
        booleanParam(name: 'DRY_RUN', defaultValue: false, description: 'Dry run mode')
    }
    stages {
        stage('Deploy') {
            steps {
                echo "Deploying to ${params.ENVIRONMENT}, dry run: ${params.DRY_RUN}"
            }
        }
    }
}
```

**13. What is the difference between `post { always { } }`, `post { success { } }`, and `post { failure { } }`?**
The `post` block defines actions that run after all stages complete, based on the final pipeline status: `always` runs regardless of outcome (commonly used for cleanup/notifications), `success` runs only if the pipeline succeeded, and `failure` runs only if it failed (commonly used for alerting the team).

**14. How does Jenkins handle credentials securely (e.g., API keys, passwords)?**
Credentials are stored encrypted in Jenkins' Credentials Store (not in the Jenkinsfile itself), referenced by an ID, and injected into the pipeline only at runtime via the `credentials()` helper or `withCredentials` block — they're masked in console output and never exposed in plaintext logs.
```groovy
withCredentials([usernamePassword(credentialsId: 'docker-hub-creds', usernameVariable: 'USER', passwordVariable: 'PASS')]) {
    sh 'docker login -u $USER -p $PASS'
}
```

**15. What is a Jenkins Multibranch Pipeline?**
A project type that automatically discovers, manages, and creates a separate pipeline run for every branch (and PR) in a repository that contains a Jenkinsfile — automatically adding new branches and removing pipelines for deleted branches, without manual job configuration per branch.

**16. What is the difference between `sh` and `bat` steps in a Jenkinsfile?**
`sh` executes a shell command on Unix/Linux agents. `bat` executes a batch command on Windows agents — used depending on the agent's operating system.

**17. How do you archive build artifacts in a Jenkins Pipeline?**
```groovy
stage('Build') {
    steps {
        sh 'mvn clean package'
        archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
    }
}
```
This saves the specified files with the build record, making them downloadable from the Jenkins UI and available to subsequent pipeline stages/jobs.

---

### 🔴 Advanced

**18. Explain how Jenkins' master-agent (controller-agent) architecture scales for large enterprises, and what's the role of dynamic/ephemeral agents.**
The Jenkins controller handles scheduling, the web UI, and orchestration but should not run heavy build workloads itself (to avoid resource contention affecting the entire system's responsiveness). Static agents are pre-provisioned, persistent machines. At enterprise scale, **dynamic/ephemeral agents** are preferred — using plugins like the Kubernetes Plugin or EC2 Plugin, Jenkins provisions a fresh agent (pod or instance) on-demand for each build, runs the job, then destroys the agent. This eliminates "works on this agent but not that one" drift, scales elastically with build demand, and avoids paying for idle capacity.

**19. 🎯 Scenario: Your Jenkins controller crashed and lost its job configuration history. How do you architect Jenkins to be resilient against this?**
1. Treat all pipeline definitions as code (Jenkinsfiles in source control) — the actual pipeline logic is never solely stored in Jenkins itself, only build *history/metadata* is.
2. Use **Configuration as Code (JCasC)** plugin to define the entire Jenkins controller configuration (plugins, credentials structure, system settings) declaratively in a YAML file, version-controlled and re-applicable to rebuild a fresh controller from scratch.
3. Regularly back up `$JENKINS_HOME` (which contains build history, credentials store, plugin data) to external storage (S3, etc.) using a scheduled job or backup plugin.
4. Run the Jenkins controller itself in a container/Kubernetes Deployment with persistent volume-backed `$JENKINS_HOME`, enabling fast recovery by simply rescheduling the pod against the same persistent volume.
5. For true high availability, consider Jenkins' (Enterprise-supported via CloudBees) HA configurations, though most teams mitigate risk primarily through fast, automated rebuild capability rather than true active-active HA, given Jenkins' architecture isn't natively designed for it.

**20. What is the difference between `input` step blocking behavior and how do you implement a manual approval gate before production deployment?**
```groovy
stage('Approve Production Deploy') {
    steps {
        timeout(time: 24, unit: 'HOURS') {
            input message: 'Deploy to production?', ok: 'Deploy',
                  submitter: 'release-managers'
        }
    }
}
stage('Deploy to Prod') {
    steps {
        sh './deploy.sh production'
    }
}
```
The `input` step pauses pipeline execution, waiting for a human to approve via the Jenkins UI; `submitter` restricts who's authorized to approve, and wrapping it in `timeout` prevents a pipeline from hanging indefinitely (and tying up an agent executor) if no one responds.

**21. 🎯 Scenario: A Jenkinsfile shared across 50 microservice repositories needs a small but breaking change (e.g., a new mandatory parameter). How do you roll this out without breaking all 50 pipelines simultaneously?**
1. If using a **Jenkins Shared Library**, version it using Git tags/branches (e.g., `@Library('my-lib@v2.0.0')`) rather than always pulling the latest — this lets each repo opt into the new version on its own timeline rather than being forced to update instantly when the library's default branch changes.
2. Implement the breaking change as backward-compatible first (e.g., make the new parameter optional with a sensible default, log a deprecation warning if unset), giving teams a migration window.
3. Communicate the deprecation timeline, update repos in batches (starting with non-critical ones), and only remove the old behavior after confirming all 50 repos have migrated (trackable via a quick grep/search across repos, or a centralized pipeline-config dashboard if available).
4. For organization-wide governance at this scale, consider whether some logic belongs in a centrally-versioned reusable workflow/template system with stronger compatibility guarantees, rather than a single shared library version controlling all consumers simultaneously.

**22. How does the Jenkins Kubernetes plugin work, and what is a Pod Template?**
The Kubernetes plugin allows Jenkins to dynamically provision build agents as Kubernetes Pods. A **Pod Template** defines the spec of the agent Pod — which container image(s) to use (e.g., a `maven` container for Java builds, a `docker` container with DinD for image builds), resource requests/limits, volumes, and labels. When a pipeline stage specifies `agent { kubernetes { ... } }` or references a pod template label, Jenkins requests the Kubernetes API to schedule a Pod matching that spec, runs the build inside it, and tears it down afterward — giving each build a clean, isolated, purpose-built environment.

**23. What are the security implications of running `docker` commands directly within a Jenkins pipeline agent, and how do you mitigate them?**
Running Docker-in-Docker (DinD) or mounting the host's Docker socket (`/var/run/docker.sock`) into a Jenkins agent container grants that agent (and by extension, anyone who can write a Jenkinsfile) effective root access to the host, since Docker socket access is equivalent to root on the host machine — a significant security risk in a multi-tenant Jenkins environment. Mitigations: use rootless Docker or tools like `kaniko`/`buildah` that can build container images without requiring a Docker daemon or privileged access at all; if DinD is unavoidable, run it in a dedicated, isolated agent Pod (not shared with other tenants) with the socket scoped as narrowly as possible; and enforce Jenkinsfile review/approval (via "Script Approval" or restricting who can modify Jenkinsfiles in protected branches) since a Jenkinsfile is effectively arbitrary code execution on whatever agent it runs on.

---

## 9. Terraform

### 🟢 Beginner

**1. What is Terraform, and what problem does it solve?**
Terraform is an Infrastructure as Code (IaC) tool by HashiCorp that lets you define cloud and on-prem infrastructure (servers, networks, databases, etc.) in declarative configuration files, then automatically creates, updates, or destroys real infrastructure to match that defined state. It solves the problem of manual, error-prone, undocumented infrastructure changes by making infrastructure version-controlled, repeatable, and auditable.

**2. What is the difference between declarative and imperative Infrastructure as Code?**
Declarative IaC (Terraform, CloudFormation) describes the *desired end state* ("I want 3 servers"), and the tool figures out how to get there. Imperative IaC (a Bash/Python script using AWS CLI) describes the *exact steps* to take ("create server 1, then server 2, then server 3") — Terraform's declarative model is generally preferred because it's idempotent and the tool itself handles dependency ordering and drift detection.

**3. What is a Terraform provider?**
A plugin that lets Terraform interact with a specific platform's API (AWS, Azure, GCP, Kubernetes, etc.), translating Terraform configuration into actual API calls to create/manage resources on that platform.

**4. What is the basic Terraform workflow (the core commands)?**
```bash
terraform init      # download providers, initialize backend
terraform plan       # preview changes
terraform apply       # execute changes
terraform destroy      # tear down managed infrastructure
```

**5. What is a Terraform resource block?**
The fundamental building block defining a single piece of infrastructure to manage:
```hcl
resource "aws_instance" "web" {
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t2.micro"
}
```

**6. What is the Terraform state file, and why is it important?**
The state file (`terraform.tfstate`) is a JSON file that records the current mapping between your Terraform configuration and the real-world resources it manages (including resource IDs and metadata). Terraform uses it to determine what changes are needed on each `plan`/`apply` — without it, Terraform wouldn't know what already exists versus what needs to be created.

**7. What is the difference between `terraform plan` and `terraform apply`?**
`terraform plan` shows a preview of what changes *would* be made (create/update/destroy) without actually making them — a dry run. `terraform apply` executes those changes against real infrastructure (typically showing the same plan output first, then asking for confirmation).

**8. What is a Terraform variable, and how do you define one?**
Variables parameterize a configuration so it isn't hardcoded:
```hcl
variable "instance_type" {
  type    = string
  default = "t2.micro"
}

resource "aws_instance" "web" {
  instance_type = var.instance_type
}
```

**9. What is a Terraform output?**
A way to extract and display specific values from your infrastructure after `apply` (e.g., a created resource's IP address or ARN), useful for displaying important information or passing values to other Terraform configurations.
```hcl
output "instance_ip" {
  value = aws_instance.web.public_ip
}
```

---

### 🟡 Intermediate

**10. What is Terraform's dependency graph, and how does it determine resource creation order?**
Terraform automatically builds a dependency graph based on implicit references between resources (e.g., if `resource_b` references an attribute of `resource_a`, Terraform infers `resource_a` must be created first). Resources without dependencies on each other are created in parallel for efficiency; you can also declare an explicit dependency using `depends_on` when no direct attribute reference exists but an ordering requirement still does.

**11. What is a Terraform module, and why is it used?**
A module is a reusable, encapsulated package of Terraform configuration (a directory with its own resources, variables, and outputs) that can be called multiple times with different inputs — promoting DRY infrastructure code (e.g., a "vpc" module reused across dev/staging/prod with different CIDR ranges as input variables).
```hcl
module "vpc" {
  source = "./modules/vpc"
  cidr_block = "10.0.0.0/16"
}
```

**12. What is a Terraform backend, and why should you avoid storing state locally in a team environment?**
A backend determines where Terraform's state file is stored and how operations are executed. Storing state locally on a single engineer's machine creates serious problems in a team: no shared visibility into current state, risk of state file loss, and no locking (two people running `apply` simultaneously could corrupt state or create conflicting changes). A **remote backend** (S3, Azure Storage, Terraform Cloud) provides centralized, shared, and often automatically locked state storage.

**13. What is state locking, and which backends support it?**
State locking prevents two simultaneous `terraform apply` operations from running against the same state file at the same time, which could cause corruption or conflicting changes. Locking is supported by remote backends like S3 (when paired with a DynamoDB table for lock coordination), Terraform Cloud, and Azure Storage (via blob leases) — local state has no locking protection.

**14. What is the difference between `terraform refresh` and `terraform plan`?**
`terraform refresh` (now effectively folded into `plan`/`apply` by default in modern Terraform via `-refresh=true`) reconciles the state file with the *actual* real-world infrastructure, detecting drift (changes made outside Terraform). `terraform plan` does this refresh implicitly and then additionally compares the refreshed state against your *desired* configuration to compute the needed changes.

**15. What is the purpose of `terraform.tfvars`?**
A file used to supply actual values for declared variables, separate from the variable *definitions* themselves — keeping environment-specific values (like `instance_type = "m5.large"` for prod vs `t2.micro` for dev) out of the reusable `.tf` configuration files. Terraform automatically loads `terraform.tfvars` if present, or you can specify a custom file with `-var-file=prod.tfvars`.

**16. What does `terraform import` do, and when would you use it?**
It brings an already-existing piece of infrastructure (created manually or by another tool) under Terraform's management, by mapping it to a resource block in your configuration and recording it in the state file — without recreating the actual resource. Used when migrating manually-created infrastructure into IaC management.

**17. What is the difference between `count` and `for_each` in Terraform?**
Both create multiple instances of a resource from a single block. `count` uses a numeric index (`0, 1, 2...`) — if an item in the middle of the list is removed, Terraform may unnecessarily destroy/recreate resources due to index shifting. `for_each` uses a map or set of strings, giving each resource a stable, named key — meaning removing one item only affects that specific resource, not others. `for_each` is generally preferred for collections of distinct, named resources.

---

### 🔴 Advanced

**18. Explain the Terraform state file's role in detecting drift, and how you would handle drift discovered in a production environment.**
Drift occurs when real infrastructure is changed outside of Terraform (e.g., someone manually modifies a security group rule in the AWS Console). On the next `terraform plan`, Terraform refreshes state against real infrastructure and will show a diff trying to revert the manual change back to match the declared configuration (since Terraform assumes its configuration is the source of truth). To handle this safely: investigate *why* the drift occurred (was it an emergency fix that should now be codified, or unauthorized?), then either update the Terraform configuration to reflect the legitimate new desired state (if the manual change should persist) or apply Terraform to revert it back to the codified state (if the manual change was unauthorized/incorrect) — never just ignore drift, since it erodes trust in Terraform as the single source of truth.

**19. 🎯 Scenario: Two engineers ran `terraform apply` simultaneously on a local state file (no remote backend configured), and the state is now corrupted/inconsistent with real infrastructure. How do you recover?**
1. Immediately migrate to a remote backend with locking (S3 + DynamoDB, or Terraform Cloud) to prevent recurrence — this should have been in place already.
2. Use `terraform state list` and compare against the actual cloud console to identify which resources are missing, duplicated, or misrepresented in state.
3. Use `terraform import` to bring any real resources back under management that are missing from state, and `terraform state rm` to remove any state entries for resources that don't actually exist (e.g., a duplicate Terraform thinks exists but the API call actually failed).
4. As a more surgical alternative for complex cases, manually edit a *backup* of the state JSON (extremely carefully, only as a last resort) or use `terraform state mv`/`terraform state pull` + `push` workflows to reconcile.
5. Once consistent, run `terraform plan` and carefully review the full diff before any further `apply`, to ensure it now accurately reflects only the intended changes, not artifacts of the corruption.

**20. What is the difference between Terraform Workspaces and using separate state files/directories per environment, and which is recommended for managing dev/staging/prod?**
Terraform Workspaces let you maintain multiple distinct state files using the *same* configuration code, switched via `terraform workspace select`. While convenient for quick experimentation, HashiCorp itself and most enterprise practitioners recommend **separate directories (or separate root modules) per environment** instead for dev/staging/prod, because workspaces make it dangerously easy to accidentally run `apply` against the wrong environment (since the same code/CLI is used, distinguished only by an easily-overlooked workspace name), and don't naturally support meaningfully different configurations (e.g., prod needing a completely different instance count/size structure, not just different variable values) — separate directories with shared modules provide clearer separation and reduce blast-radius risk.

**21. 🎯 Scenario: Your team wants to enforce that no Terraform plan can be applied that would delete a production RDS database, even by mistake. How do you implement this guardrail?**
1. Use `prevent_destroy` lifecycle meta-argument directly on the critical resource as a baseline protection:
```hcl
resource "aws_db_instance" "prod" {
  # ...
  lifecycle {
    prevent_destroy = true
  }
}
```
This causes `terraform apply`/`destroy` to fail immediately if it would destroy this specific resource.
2. For broader, policy-driven enforcement across many resources/teams, implement **Sentinel** (Terraform Cloud/Enterprise) or **Open Policy Agent (OPA)** policies that inspect the `terraform plan` JSON output in CI and block the pipeline if any `delete` action is detected on resources tagged `environment=production` or matching specific resource type patterns, requiring explicit human override/approval to proceed.
3. Combine with strict CI/CD gating — production applies should never run from a local machine, only through a pipeline with mandatory plan review and approval gates before `apply`.

**22. What is the difference between a Terraform provisioner and a configuration management tool (like Ansible), and why is provisioner use generally discouraged as a primary approach?**
Provisioners (`local-exec`, `remote-exec`) let Terraform run scripts on a resource after creation (e.g., SSH into a new VM and run a setup script). HashiCorp explicitly discourages relying on provisioners as a primary mechanism because they break Terraform's core declarative/idempotent model — Terraform has no real visibility into what the script does, can't accurately detect drift in whatever the script configured, and provisioner failures can leave resources in a partially-applied, hard-to-reconcile state. The recommended pattern is to use Terraform purely for provisioning infrastructure, and a dedicated configuration management tool (Ansible) or, better, pre-baked immutable images (built via Packer) for configuring the software/state *within* that infrastructure — keeping each tool focused on what it does best.

**23. How would you structure Terraform code for a large enterprise managing 50+ AWS accounts across multiple teams, balancing reusability with blast-radius isolation?**
1. Use a **module registry** (private or Terraform Cloud's registry) of well-tested, versioned shared modules (VPC, IAM roles, standard tagging) that all teams consume — ensuring consistency without duplicating logic.
2. Structure root configurations **per account/environment**, not as one giant monolithic state covering everything — this limits blast radius (a bad `apply` in one team's dev account can't accidentally affect another team's production state) and keeps `plan`/`apply` times manageable (smaller state = faster operations).
3. Use a **multi-account AWS Organizations structure** with Terraform managing account-level resources (via assumed IAM roles per account) from centralized CI/CD pipelines, with remote state stored per-account/environment in a centralized S3 bucket with account-specific key prefixes and tight IAM bucket policies.
4. Enforce policy-as-code (Sentinel/OPA) centrally across all pipelines for guardrails (cost limits, mandatory tagging, restricted regions/instance types), while still giving individual teams autonomy over their own application-level resource definitions within those guardrails.

---

## 10. Networking

### 🟢 Beginner

**1. What is the OSI model, and why is it relevant to DevOps?**
The OSI model is a 7-layer conceptual framework describing how network communication happens: Physical, Data Link, Network, Transport, Session, Presentation, Application. DevOps engineers most commonly work at Layer 3 (Network/IP), Layer 4 (Transport/TCP-UDP, load balancing), and Layer 7 (Application/HTTP, routing) — understanding which layer a problem occurs at speeds up troubleshooting significantly (e.g., "is this a DNS issue, a TCP connection issue, or an application-level HTTP error?").

**2. What is the difference between TCP and UDP?**
TCP (Transmission Control Protocol) is connection-oriented, reliable, and ordered — it establishes a handshake, retransmits lost packets, and guarantees delivery order, at the cost of more overhead. UDP (User Datagram Protocol) is connectionless and unreliable — no handshake, no retransmission, no ordering guarantee, but much lower latency/overhead, making it suitable for use cases like video streaming or DNS where occasional loss is acceptable but speed matters more.

**3. What is an IP address, and what's the difference between IPv4 and IPv6?**
An IP address uniquely identifies a device on a network. IPv4 uses a 32-bit address (e.g., `192.168.1.1`), providing about 4.3 billion unique addresses — largely exhausted globally. IPv6 uses a 128-bit address (e.g., `2001:0db8::1`), providing a vastly larger address space designed to solve IPv4 exhaustion, though adoption is still gradual.

**4. What is a subnet, and why is subnetting used?**
A subnet is a logical subdivision of a larger network, defined by a network address and subnet mask (e.g., `10.0.1.0/24`). Subnetting is used to organize networks logically (e.g., separate public/private subnets), improve security (isolating segments), and efficiently allocate IP address ranges.

**5. What is DNS, and how does it work at a basic level?**
DNS (Domain Name System) translates human-readable domain names (`example.com`) into IP addresses that computers use to route traffic. At a basic level: your computer queries a DNS resolver, which checks its cache, then queries root servers → TLD servers (`.com`) → authoritative name servers for the specific domain, returning the IP address.

**6. What is the difference between a public IP and a private IP address?**
A public IP is globally unique and routable across the internet. A private IP (ranges like `10.0.0.0/8`, `172.16.0.0/12`, `192.168.0.0/16`) is only valid within a private network (like a home network or VPC) and isn't directly routable on the internet — devices using private IPs typically reach the internet via NAT.

**7. What is a firewall?**
A security system that monitors and controls incoming/outgoing network traffic based on defined rules, typically used to block unauthorized access while allowing legitimate traffic (e.g., allowing port 443 for HTTPS, blocking all other inbound ports by default).

**8. What is the difference between HTTP and HTTPS?**
HTTP transmits data in plaintext. HTTPS adds a layer of encryption (TLS/SSL) on top of HTTP, encrypting data in transit so it can't be read or tampered with by an intermediary — essential for protecting sensitive data like passwords and payment information.

**9. What are common HTTP status codes, and what do the ranges generally mean?**
- **2xx**: Success (200 OK, 201 Created)
- **3xx**: Redirection (301 Moved Permanently, 304 Not Modified)
- **4xx**: Client error (400 Bad Request, 401 Unauthorized, 404 Not Found)
- **5xx**: Server error (500 Internal Server Error, 502 Bad Gateway, 503 Service Unavailable)

**10. What is a port, and what are some common port numbers?**
A port is a numbered endpoint (0–65535) on a device that identifies a specific service/process for network communication. Common ports: 22 (SSH), 80 (HTTP), 443 (HTTPS), 3306 (MySQL), 5432 (PostgreSQL), 6379 (Redis).

---

### 🟡 Intermediate

**11. What is NAT (Network Address Translation), and why is it commonly used in cloud VPCs?**
NAT translates private IP addresses to a public IP (and back) so devices on a private network can communicate with the internet without each having its own public IP. In cloud VPCs, a **NAT Gateway** placed in a public subnet allows instances in private subnets to initiate outbound internet connections (e.g., downloading updates) while remaining unreachable from inbound internet traffic — a common security pattern.

**12. What is the difference between a Load Balancer operating at Layer 4 vs Layer 7?**
A **Layer 4 load balancer** makes routing decisions based on IP address and TCP/UDP port only, without inspecting the actual application data — faster, but can't route based on content (e.g., URL path). A **Layer 7 load balancer** operates at the application layer, inspecting HTTP headers/paths/cookies to make more intelligent routing decisions (e.g., routing `/api/*` to one backend and `/static/*` to another), at the cost of slightly more processing overhead.

**13. What is the TCP three-way handshake?**
The process establishing a TCP connection: (1) client sends `SYN`, (2) server responds `SYN-ACK`, (3) client responds `ACK` — after which the connection is established and data transfer can begin.

**14. What is the difference between a Forward Proxy and a Reverse Proxy?**
A **forward proxy** sits in front of clients, forwarding their requests to the internet on their behalf (often used for anonymity, content filtering, or caching from a client's perspective). A **reverse proxy** sits in front of servers, receiving client requests on the servers' behalf and forwarding them to the appropriate backend — used for load balancing, SSL termination, and hiding backend infrastructure details (e.g., Nginx as a reverse proxy in front of application servers).

**15. What is the purpose of a VPC (Virtual Private Cloud)?**
A VPC is an isolated, logically-defined virtual network within a cloud provider, where you control IP address ranges, subnets, route tables, and gateways — providing network isolation and security boundaries for your cloud resources, similar to having your own private datacenter network within the shared cloud infrastructure.

**16. What is the difference between a Security Group and a Network ACL (in AWS, for example)?**
A **Security Group** operates at the instance level, is *stateful* (if you allow inbound traffic, the corresponding outbound return traffic is automatically allowed without an explicit rule), and only supports "allow" rules. A **Network ACL** operates at the subnet level, is *stateless* (inbound and outbound rules must be explicitly defined separately, even for return traffic), and supports both "allow" and "deny" rules, evaluated in order by rule number.

**17. What is DNS round-robin, and what's a limitation of it as a load-balancing technique?**
DNS round-robin returns multiple IP addresses for a single domain in rotating order, distributing client connections across multiple servers. Its key limitation: DNS has no awareness of server health or actual load — if a server goes down, DNS may still hand out its IP to clients (until TTL expires and health-aware mechanisms intervene), making it a weak, blunt load-balancing mechanism compared to dedicated load balancers with active health checks.

**18. What is the difference between A, CNAME, and MX DNS records?**
**A record** maps a domain name directly to an IPv4 address. **CNAME** (Canonical Name) maps a domain name to *another domain name* (an alias), which is then further resolved — commonly used for subdomains pointing to a service (`www` → `mycdn.cloudprovider.com`). **MX record** specifies the mail server(s) responsible for receiving email for the domain.

**19. What is a VPN, and how does it provide secure remote access?**
A VPN (Virtual Private Network) creates an encrypted tunnel between a client and a private network over the public internet, making the client appear as if it's directly on that private network — used for securely accessing internal company resources remotely without exposing them directly to the public internet.

---

### 🔴 Advanced

**20. 🎯 Scenario: Users report that your application is intermittently slow, but server-side CPU/memory metrics look completely normal. How would you investigate whether this is a networking issue?**
1. Check for **packet loss/latency** between client and server using `mtr` (combines ping + traceroute) or `ping`/`traceroute` to identify where in the network path delay is introduced.
2. Inspect **TCP retransmission rates** on the server (`netstat -s` or `ss -s`) — high retransmits indicate an unstable network path causing the OS to resend lost packets, adding latency invisible to application-level CPU metrics.
3. Check for **DNS resolution delays** — slow or flaky DNS lookups (especially with misconfigured/missing caching) can add significant latency before a connection even begins; tools like `dig +trace` help diagnose this.
4. Examine **connection pool exhaustion** — if the app or a downstream dependency (database) has a small connection pool, requests may queue waiting for an available connection, appearing as "slowness" while CPU is idle.
5. Check load balancer / proxy layer metrics (Layer 7 LB logs, target response time vs total request time) to isolate whether delay is happening at the LB→backend hop specifically, versus client→LB.
6. Rule out **MTU/fragmentation issues**, particularly relevant in VPN/overlay network (e.g., Kubernetes CNI) setups, which can cause specific packet sizes to silently drop or fragment, causing inconsistent/intermittent symptoms that correlate with payload size.

**21. Explain how BGP (Border Gateway Protocol) works at a high level, and why it matters for cloud networking/multi-region architectures.**
BGP is the routing protocol that determines how data travels across the internet between different autonomous systems (ASes — large networks like ISPs, cloud providers). It works by routers exchanging reachability information (which IP prefixes they can route to, and via which path) and selecting the best path based on attributes like AS path length. It matters for DevOps because: cloud providers use BGP for Direct Connect/ExpressRoute connections to on-prem datacenters, anycast IP addressing (used by CDNs and DNS providers for global load distribution) relies on BGP routing the same IP to the geographically nearest healthy location, and BGP misconfigurations/hijacks are a real-world cause of major internet outages.

**22. What is the difference between TLS termination at the Load Balancer versus end-to-end TLS (re-encryption) to the backend, and what are the trade-offs?**
**TLS termination at the LB**: the load balancer decrypts incoming HTTPS traffic and forwards it as plain HTTP to backend servers within the (presumably trusted) internal network — simpler, offloads CPU-intensive TLS work from backend servers, but means traffic is unencrypted on the internal network segment. **End-to-end TLS (re-encryption)**: the LB decrypts incoming traffic, then re-encrypts a new TLS connection to the backend — ensures encryption across the entire path, required for compliance frameworks (PCI-DSS, HIPAA) mandating encryption in transit even within internal networks, at the cost of additional CPU overhead and certificate management complexity on backend servers too.

**23. 🎯 Scenario: Your Kubernetes application running across multiple Availability Zones is experiencing higher-than-expected cross-AZ data transfer costs. How do you diagnose and reduce this at the networking level?**
1. Identify the source: check if Pod-to-Pod or Pod-to-database traffic is frequently crossing AZ boundaries — visualize using a service mesh's traffic metrics (e.g., Istio/Linkerd) or VPC Flow Logs analysis.
2. Implement **topology-aware routing** (Kubernetes `Service Internal Traffic Policy` or `topology.kubernetes.io/zone`-aware service routing) so traffic preferentially routes to a Pod replica in the *same* AZ as the caller, rather than being randomly load-balanced across all AZs.
3. For stateful dependencies (databases, caches), consider per-AZ read replicas or AZ-local caching layers to reduce the need for application Pods to repeatedly query a database instance in a different AZ.
4. Review NAT Gateway usage — NAT Gateway data processing charges apply per-AZ; ensure private subnets use a NAT Gateway in their *own* AZ rather than routing through a single NAT Gateway in a different AZ for high-volume traffic.
5. Weigh the trade-off explicitly: cross-AZ traffic costs money but cross-AZ *redundancy* is what provides high availability — the goal is to reduce unnecessary cross-AZ chatter, not eliminate the redundancy itself.

**24. Explain how Anycast IP routing works and why it's used for global services like DNS resolvers (e.g., 8.8.8.8) and CDNs.**
With Anycast, the *same* IP address is announced via BGP from multiple physically distinct locations around the world simultaneously. Internet routing (via BGP's shortest-AS-path selection) automatically directs a client's traffic to whichever announcing location is "closest" from a routing perspective (typically correlating with lowest latency), without the client needing any awareness of multiple locations or DNS-based geo-routing. This provides both low-latency global access (clients reach the nearest edge automatically) and resilience (if one location fails/withdraws its BGP announcement, traffic automatically reroutes to the next nearest location) — which is why it's foundational to how CDNs, public DNS resolvers, and DDoS-mitigation services operate at global scale.

---

## 11. Azure

### 🟢 Beginner

**1. What is Microsoft Azure?**
Azure is Microsoft's public cloud computing platform, offering a broad range of services — virtual machines, storage, databases, networking, AI/ML, and managed Kubernetes — billed on a pay-as-you-go model, competing primarily with AWS and Google Cloud Platform.

**2. What is an Azure Resource Group?**
A logical container that holds related Azure resources (VMs, storage accounts, databases) for a project or environment — used for organizing, applying access control, and managing the lifecycle (e.g., deleting a Resource Group deletes everything inside it) of related resources together.

**3. What is Azure Virtual Machine (VM), and how does it compare conceptually to AWS EC2?**
An Azure VM is an on-demand, scalable virtual server — Azure's equivalent of AWS EC2. Both let you choose an OS image, instance size (CPU/RAM), attach storage, and configure networking, billed per usage.

**4. What is the difference between Azure Blob Storage and Azure Files?**
**Blob Storage** is object storage optimized for unstructured data (images, backups, logs) accessed via REST API/SDK — Azure's equivalent of AWS S3. **Azure Files** provides fully managed file shares accessible via the standard SMB/NFS protocol, usable as a mounted network drive — suited for legacy applications that expect a traditional file share rather than object storage APIs.

**5. What is Azure Active Directory (Azure AD / Microsoft Entra ID)?**
Microsoft's cloud-based identity and access management service, used for authenticating users and managing access to Azure resources and Microsoft 365 applications — Azure's equivalent of AWS IAM, though Azure AD also serves as a broader enterprise identity platform beyond just cloud resource access.

**6. What is the Azure Portal, and what's the alternative for automation?**
The Azure Portal is the web-based GUI for managing Azure resources. For automation, the **Azure CLI** (`az` command), **Azure PowerShell**, or Infrastructure as Code tools (Terraform, ARM templates, Bicep) are used instead — essential for any production/repeatable workflow rather than manual clicking.

**7. What is an Azure Subscription?**
A billing and access management boundary in Azure — all resources exist within a subscription, which is tied to a specific billing account and can have its own access policies, quotas, and cost management settings. Large enterprises often use multiple subscriptions to separate environments (dev/prod) or business units.

**8. What is Azure App Service?**
A fully managed Platform-as-a-Service (PaaS) for hosting web applications, REST APIs, and mobile backends — handling infrastructure, scaling, and patching automatically, letting developers focus purely on application code rather than managing servers.

---

### 🟡 Intermediate

**9. What is Azure Virtual Network (VNet), and how does it relate to subnets and Network Security Groups (NSGs)?**
A VNet is Azure's isolated private network (equivalent to AWS VPC), within which you create **subnets** to logically segment resources. **Network Security Groups (NSGs)** are applied to subnets or individual network interfaces to control inbound/outbound traffic via allow/deny rules — functioning similarly to a combination of AWS Security Groups and NACLs.

**10. What is Azure Kubernetes Service (AKS)?**
Azure's managed Kubernetes offering — Azure handles the control plane (master nodes, etcd, API server) at no extra charge, while you manage and pay for the worker nodes. It integrates natively with Azure AD for authentication, Azure Monitor for observability, and Azure Load Balancer/Application Gateway for ingress.

**11. What is the difference between Azure Container Instances (ACI) and AKS?**
**ACI** runs individual containers directly without needing to manage any underlying VMs or a Kubernetes cluster — ideal for simple, short-lived, or burst workloads. **AKS** is a full Kubernetes cluster, suited for complex, multi-container, orchestrated applications requiring features like auto-scaling, service discovery, and rolling deployments across many services.

**12. What is an Azure Resource Manager (ARM) template?**
A JSON-based declarative Infrastructure as Code format native to Azure, describing the resources to deploy and their configuration — conceptually similar to AWS CloudFormation. **Bicep** is Microsoft's newer, more concise domain-specific language that compiles down to ARM JSON, offering a much friendlier authoring experience while deploying through the same underlying ARM engine.

**13. What is Azure DevOps, and how does it differ from GitHub?**
Azure DevOps is Microsoft's integrated suite for the full software delivery lifecycle — Azure Repos (Git hosting), Azure Pipelines (CI/CD), Azure Boards (work item/project tracking), Azure Artifacts (package management), and Azure Test Plans. While GitHub has converged toward similar capabilities (especially with GitHub Actions), Azure DevOps has traditionally been the more enterprise/Microsoft-ecosystem-oriented choice, with deeper native Azure integration and more mature work-item tracking (Boards) out of the box.

**14. What is the difference between Azure's Managed Identity types — System-Assigned vs User-Assigned?**
A **System-Assigned Managed Identity** is tied directly to a single Azure resource's lifecycle (created and deleted along with that resource, e.g., a specific VM) — used when only that one resource needs the identity. A **User-Assigned Managed Identity** is created as a standalone resource, independent of any single resource's lifecycle, and can be attached to multiple resources simultaneously — used when several resources need to share the same identity/permission set.

**15. What is Azure Key Vault used for?**
A centralized, secure store for secrets, encryption keys, and certificates — applications retrieve these at runtime via the Key Vault SDK/API (authenticated via Managed Identity, avoiding hardcoded credentials), and Key Vault also handles key rotation and audit logging of all access.

**16. What is the difference between Azure Load Balancer and Azure Application Gateway?**
**Azure Load Balancer** operates at Layer 4 (TCP/UDP) — fast, protocol-agnostic load balancing without inspecting application content. **Application Gateway** operates at Layer 7 (HTTP/HTTPS) — supports URL-path-based routing, SSL termination, cookie-based session affinity, and integrates a built-in Web Application Firewall (WAF) for application-layer threat protection.

---

### 🔴 Advanced

**17. Explain Azure's Role-Based Access Control (RBAC) model and how role assignments inherit through the resource hierarchy.**
Azure RBAC grants permissions by assigning a **Role** (a collection of permissions, e.g., "Contributor", "Reader", or a custom role) to a **Security Principal** (user, group, service principal, or managed identity) at a specific **Scope** (Management Group → Subscription → Resource Group → individual Resource). Permissions assigned at a higher scope automatically inherit down to everything beneath it — e.g., a role assigned at the Subscription level applies to every Resource Group and resource within it, unless more restrictive scoping is deliberately used at a lower level. This hierarchical inheritance model is key to designing least-privilege access at enterprise scale without manually assigning permissions resource-by-resource.

**18. 🎯 Scenario: Your organization needs to deploy infrastructure consistently across dozens of Azure subscriptions belonging to different business units, while enforcing organization-wide security/compliance baselines. What Azure-native tooling would you use?**
1. **Management Groups** to organize subscriptions hierarchically (e.g., by business unit or environment), allowing policies and RBAC to be applied at the appropriate level of the hierarchy rather than per-subscription.
2. **Azure Policy** to define and enforce compliance rules declaratively (e.g., "all storage accounts must have encryption enabled", "only allow VMs in approved regions") — policies can be set to `Audit` (report violations) or `Deny` (block non-compliant deployments) and apply automatically across the assigned Management Group/Subscription scope.
3. **Azure Blueprints** (or the increasingly preferred **Terraform/Bicep + CI/CD pipeline** approach) to package and consistently deploy a standard set of resources, policies, and RBAC assignments as a versioned artifact across multiple subscriptions.
4. Centralized **Azure Landing Zones** architecture (Microsoft's reference implementation) providing a pre-designed, scalable multi-subscription foundation with networking, identity, governance, and security baked in from day one.

**19. What is the difference between Azure Front Door and Azure Application Gateway, and when would you use each in a global architecture?**
**Azure Front Door** is a global, Anycast-based Layer 7 load balancer and CDN operating at Microsoft's edge network — used for routing traffic across *multiple regions/backends globally*, with built-in CDN caching, WAF, and automatic failover between regions. **Application Gateway** is a *regional* Layer 7 load balancer, used for routing/load-balancing traffic *within* a single region/VNet. In a global multi-region architecture, Front Door typically sits at the outermost edge (routing users to the nearest healthy region), with an Application Gateway inside each region handling more granular routing to backend services within that region.

**20. 🎯 Scenario: An AKS cluster needs to securely pull container images from Azure Container Registry (ACR) without storing any credentials in Kubernetes Secrets. How do you architect this?**
Use **AKS-to-ACR integration via Managed Identity**, attaching the role `AcrPull` to the AKS cluster's kubelet managed identity at the ACR resource scope:
```bash
az aks update -n myAKSCluster -g myResourceGroup --attach-acr myACRRegistry
```
This grants the AKS cluster's nodes (via their managed identity) direct, credential-less authorization to pull images from the specified ACR — Kubernetes' `imagePullSecrets` becomes entirely unnecessary, eliminating both the security risk of stored registry credentials and the operational burden of credential rotation.

**21. Explain Azure's shared responsibility model in the context of AKS specifically — what does Microsoft manage versus what remains the customer's responsibility?**
Microsoft manages and is responsible for the **AKS control plane** — the API server, etcd, scheduler, controller manager — including its availability, patching, and security, at no additional charge for the control plane itself. The **customer remains responsible** for: worker node OS patching (unless using node image auto-upgrade features), application-level security (RBAC within the cluster, Pod security standards, network policies), workload configuration (resource limits, secrets management), and the worker nodes' VM-level security configuration. Understanding this boundary is critical for accurately scoping security audits and incident response — a control-plane-level vulnerability is Microsoft's responsibility to patch, while a misconfigured Kubernetes RBAC role or an unpatched container image is the customer's responsibility.

**22. How would you design a disaster recovery strategy for a critical application running on Azure across multiple regions?**
1. Use **Azure Traffic Manager** or **Azure Front Door** for DNS/global traffic routing with health-probe-based automatic failover to a secondary region if the primary region becomes unhealthy.
2. For data tier resilience, use **geo-replication** features native to the data service in use — e.g., Azure SQL Database's Auto-failover groups, Cosmos DB's multi-region writes, or Storage Account's Geo-Redundant Storage (GRS) — choosing between active-passive and active-active replication based on RPO/RTO requirements and consistency needs.
3. Define and regularly test explicit **RTO (Recovery Time Objective)** and **RPO (Recovery Point Objective)** targets — DR architecture decisions (cold standby vs warm standby vs active-active) should be driven by these business requirements, not arbitrarily chosen, since active-active multi-region is significantly more expensive and complex than periodic backup/restore for workloads that can tolerate longer recovery times.
4. Automate infrastructure provisioning for the secondary region via IaC (Terraform/Bicep) so a full region rebuild is reproducible and not dependent on manual runbooks or tribal knowledge during an actual disaster event.

---

## 12. Prometheus

### 🟢 Beginner

**1. What is Prometheus, and what is it used for?**
Prometheus is an open-source monitoring and alerting toolkit that collects and stores time-series metrics data (numeric measurements with timestamps) by periodically *pulling/scraping* metrics from configured targets, then lets you query, visualize, and alert on that data using its own query language, PromQL.

**2. What is a time-series in the context of Prometheus?**
A stream of timestamped data points for a specific metric with a specific set of labels — e.g., `http_requests_total{method="GET", status="200"}` recorded every 15 seconds forms one time-series.

**3. What is the difference between Prometheus's pull model and a push-based monitoring system?**
In Prometheus's **pull model**, the Prometheus server actively scrapes (HTTP GET requests) metrics endpoints exposed by applications at regular intervals. In a **push model** (like StatsD or Datadog's agent), applications/agents actively send metrics data to a central collector. Prometheus's pull model makes it easy to detect if a target is down (a failed scrape is itself informative) and centralizes scrape configuration, but requires targets to be discoverable/reachable by the Prometheus server (for which it provides a separate Pushgateway for genuinely short-lived/batch jobs that can't be scraped).

**4. What are the four core Prometheus metric types?**
- **Counter**: a cumulative value that only increases (e.g., total HTTP requests served) — resets to 0 only on restart.
- **Gauge**: a value that can go up or down (e.g., current memory usage, number of active connections).
- **Histogram**: samples observations (e.g., request durations) into configurable buckets, allowing calculation of quantiles/percentiles.
- **Summary**: similar to Histogram but calculates configurable quantiles directly on the client side rather than via buckets.

**5. What is a Prometheus exporter?**
A small program that exposes metrics from a third-party system (which doesn't natively expose Prometheus-format metrics) in a Prometheus-compatible `/metrics` HTTP endpoint — e.g., `node_exporter` exposes host-level OS metrics (CPU, memory, disk), `mysqld_exporter` exposes MySQL database metrics.

**6. What is `prometheus.yml`?**
Prometheus's main configuration file, defining scrape targets/intervals, alerting rules locations, and Alertmanager connection settings.
```yaml
scrape_configs:
  - job_name: 'node'
    static_configs:
      - targets: ['localhost:9100']
```

**7. What is PromQL?**
Prometheus's functional query language, used to select and aggregate time-series data — e.g., `http_requests_total` returns the raw metric, while `rate(http_requests_total[5m])` calculates the per-second average rate of increase over the last 5 minutes.

---

### 🟡 Intermediate

**8. What is the difference between `rate()` and `irate()` in PromQL?**
`rate()` calculates the average per-second rate of increase over the specified time range, smoothing out short-term fluctuations — recommended for alerting and general dashboards. `irate()` calculates the rate based only on the *last two data points* in the range, making it more sensitive to very recent, rapid changes — useful for fast-moving graphs but can be noisy/misleading for alerting due to its sensitivity to brief spikes.

**9. What is the difference between a Counter and a Gauge in practice, and why does it matter for query functions?**
Because Counters only increase (and reset to 0 on restart), you should *never* use a raw Counter value directly in a dashboard — always wrap it with `rate()` or `increase()` to get a meaningful rate of change. Gauges, since they represent a current point-in-time value (already meaningful on their own), are queried directly without `rate()`. Applying `rate()` to a Gauge, or reading a raw Counter value, are both common beginner mistakes that produce misleading dashboards.

**10. What is Alertmanager, and how does it relate to Prometheus?**
Alertmanager is a separate component that receives alerts fired by Prometheus (based on alerting rules evaluating PromQL expressions) and handles the downstream logic: deduplication, grouping related alerts together, silencing, and routing to the correct notification channel (Slack, PagerDuty, email) based on configurable routing rules.

**11. What is a Prometheus alerting rule, and what does the `for` clause do?**
An alerting rule defines a PromQL condition that, when true, fires an alert:
```yaml
- alert: HighErrorRate
  expr: rate(http_requests_total{status="500"}[5m]) > 0.05
  for: 10m
  labels:
    severity: critical
```
The `for: 10m` clause requires the condition to remain true continuously for 10 minutes before the alert actually fires (transitioning from "pending" to "firing") — preventing alert flapping/noise from brief, self-resolving spikes.

**12. What is service discovery in Prometheus, and why is it important in dynamic environments like Kubernetes?**
Static target lists (`static_configs`) don't work well in environments where Pods/instances are constantly created and destroyed (autoscaling, rolling deployments). **Service discovery** (e.g., `kubernetes_sd_configs`) lets Prometheus automatically discover and update its scrape target list by querying the Kubernetes API for Pods/Services matching certain criteria, without manual reconfiguration every time the infrastructure changes.

**13. What is the purpose of `relabel_configs` in Prometheus?**
`relabel_configs` lets you dynamically modify, filter, or drop labels on discovered targets before scraping — commonly used in Kubernetes service discovery to filter which Pods to scrape (e.g., only Pods with a specific annotation), or to rewrite Kubernetes-generated labels into more readable metric label names.

**14. What is cardinality in Prometheus, and why is "high cardinality" considered a problem?**
Cardinality refers to the number of unique time-series generated by a metric and its label combinations. High cardinality (e.g., adding a `user_id` or raw `request_path` as a label, generating millions of unique combinations) dramatically increases Prometheus's memory usage and query performance, since each unique label combination is stored as a completely separate time-series — a well-known operational pitfall that can crash or severely degrade a Prometheus server if not avoided through careful label design.

---

### 🔴 Advanced

**15. Explain Prometheus's local storage engine (TSDB) and its 2-hour block structure.**
Prometheus's TSDB writes incoming samples to an in-memory write-ahead log (WAL) for durability, and periodically (every 2 hours by default) flushes data into immutable on-disk **blocks**, each containing a chunk of time-series data plus an index for efficient querying. Older blocks are eventually merged together (compaction) to reduce the number of files and improve query efficiency, and data is deleted entirely once it exceeds the configured retention period. This design optimizes for the specific access pattern of time-series data (mostly append-only writes, range-based reads) far more efficiently than a general-purpose database would.

**16. 🎯 Scenario: Your Prometheus server is consuming excessive memory and occasionally OOM-crashing. How do you diagnose and fix this?**
1. Check for **high cardinality metrics** as the most common root cause — query `topk(10, count by (__name__)(({__name__=~".+"})))` (or use `prometheus_tsdb_symbol_table_size_bytes` and per-metric series counts) to identify which metrics have an unexpectedly large number of unique time-series.
2. Identify and fix the offending label(s) — typically an unbounded label value like a raw user ID, timestamp, or unique request path being used as a label, which should instead be removed or replaced with a bounded/aggregated value.
3. Use `relabel_configs` with `drop` actions to filter out genuinely unnecessary metrics/labels at scrape time, reducing ingest volume.
4. If the data volume is legitimately large (not a cardinality mistake) and a single Prometheus instance can no longer handle it, consider horizontal scaling via **Prometheus federation** (a higher-level Prometheus scraping aggregated data from multiple lower-level Prometheus servers) or migrating to a remote-write-compatible long-term storage/scaling solution like **Thanos**, **Cortex**, or **Mimir**.
5. Tune `--storage.tsdb.retention.time` to a shorter window if long retention isn't actually needed locally (offloading long-term storage to remote-write targets instead).

**17. What is the difference between Prometheus Federation and remote_write-based long-term storage solutions like Thanos?**
**Federation** has a higher-level Prometheus server periodically scrape a *subset* of aggregated/important metrics from lower-level Prometheus servers — useful for hierarchical rollups (e.g., per-cluster Prometheus instances federated up to a global view) but isn't designed for full long-term storage or true global query capability across all raw data. **Thanos** (or similar tools like Cortex/Mimir) instead has each Prometheus instance continuously **remote_write** *all* its data to a shared, horizontally-scalable object storage backend (S3, GCS), with a separate Querier component providing a unified, deduplicated global query view across all instances and historical data — purpose-built for true long-term retention and globally-scalable multi-cluster Prometheus deployments, which Federation alone cannot provide.

**18. 🎯 Scenario: You need to alert when error rate exceeds a threshold, but you want to avoid alert fatigue from noisy, flapping conditions. Design a robust alerting rule.**
```yaml
- alert: HighErrorRate
  expr: |
    (
      sum(rate(http_requests_total{status=~"5.."}[5m]))
      /
      sum(rate(http_requests_total[5m]))
    ) > 0.05
  for: 15m
  labels:
    severity: critical
  annotations:
    summary: "Error rate above 5% for 15+ minutes"
    description: "Current error rate: {{ $value | humanizePercentage }}"
```
Key design decisions: (1) using a **ratio** (error rate / total rate) rather than an absolute count, since absolute thresholds don't scale correctly with traffic volume; (2) the `for: 15m` clause requires sustained breach, filtering out brief blips; (3) using `rate()` over a 5-minute window already smooths short-term noise before the ratio is even computed; (4) routing via Alertmanager's grouping/inhibition rules to avoid firing dozens of near-duplicate alerts if multiple related symptoms occur simultaneously (e.g., suppress a "high latency" alert if a more fundamental "service down" alert is already firing for the same service).

**19. Explain PromQL subqueries and give a practical example of when they're necessary.**
A subquery evaluates an instant-vector-producing expression repeatedly over a range, effectively turning it into a range-vector for further processing — syntax: `<instant_query>[<range>:<resolution>]`. Example: calculating the maximum 5-minute request rate observed over the past hour:
```promql
max_over_time(rate(http_requests_total[5m])[1h:1m])
```
This is necessary when you need to apply a range-vector function (like `max_over_time`) to the *result* of another range-vector function (like `rate`), which can't be directly nested without a subquery — useful for "what was the peak rate" or "what was the worst value of a derived metric" style questions that simple instant queries can't answer.

**20. What is the role of `staleness` handling in Prometheus, and why does it matter for ephemeral workloads like Kubernetes Jobs/short-lived Pods?**
If Prometheus stops receiving new samples for a given time-series (e.g., a Pod is deleted and its scrape target disappears) for longer than the staleness threshold (default 5 minutes), it marks that series as "stale" and queries return no data for it going forward, rather than incorrectly extrapolating/continuing to show the last known value indefinitely. This matters significantly for dynamic, ephemeral environments — without staleness handling, dashboards and alerts could show misleadingly "frozen" data for resources that no longer exist, masking real state changes (like a Pod being deleted as part of an intentional scale-down) as if they were still actively reporting their last metric value.

---

## 13. Grafana

### 🟢 Beginner

**1. What is Grafana, and what is it used for?**
Grafana is an open-source visualization and dashboarding platform used to query, visualize, and explore metrics, logs, and traces from various data sources (Prometheus, Elasticsearch, MySQL, CloudWatch, etc.) through customizable graphs, charts, and dashboards.

**2. What is a Grafana Data Source?**
A configured connection to a backend system that stores the actual data Grafana visualizes (e.g., a Prometheus server, an Elasticsearch cluster, a PostgreSQL database) — Grafana itself doesn't store metric data; it queries data sources in real-time (or near-real-time) to render visualizations.

**3. What is a Grafana Dashboard, and what is a Panel?**
A Dashboard is a collection of visualizations organized on a single screen. A Panel is an individual visualization within a dashboard (a graph, a table, a single-stat number, a gauge) — each panel has its own query against a data source and its own visualization settings.

**4. What is the difference between Grafana and Prometheus?**
Prometheus is the data collection and storage engine (it scrapes, stores, and queries time-series metrics via PromQL). Grafana is purely a visualization layer — it doesn't collect or store metrics itself but connects *to* Prometheus (or many other data sources) to display that data in dashboards. They're commonly used together but serve entirely different purposes.

**5. How do you create a basic graph panel in Grafana?**
Add a new Panel → select a Data Source → write a query (e.g., a PromQL expression if using Prometheus) → choose a visualization type (Time series, Bar chart, Gauge, etc.) → configure display options (legend, axis, thresholds).

**6. What is a Grafana variable, and why is it useful?**
A variable (e.g., `$environment`, `$service`) lets you create dynamic, reusable dashboards where users can select different values from a dropdown (like switching between `prod`/`staging`, or different hosts) without needing a separate dashboard for each option — the underlying queries automatically use the selected variable's value.

**7. What is a Grafana Alert?**
A rule configured within Grafana (Grafana's native unified alerting, separate from Prometheus's own Alertmanager-based alerting) that evaluates a query against a threshold and triggers notifications (Slack, email, PagerDuty) when conditions are met.

---

### 🟡 Intermediate

**8. What is the difference between Grafana's built-in alerting and Prometheus's Alertmanager-based alerting?**
**Grafana's native alerting** (Grafana Unified Alerting) evaluates queries against any connected data source (not just Prometheus) directly within Grafana, and has its own notification routing system. **Prometheus + Alertmanager** evaluates PromQL rules within Prometheus itself, with Alertmanager handling routing/deduplication/silencing separately. Many teams use Prometheus/Alertmanager as the primary alerting pipeline (since it's tightly coupled with the metrics collection and works even if Grafana is down) while still using Grafana for visualization, though Grafana's unified alerting has become increasingly capable and is sometimes used as the sole alerting layer, especially for non-Prometheus data sources.

**9. What is "Explore" mode in Grafana?**
A dedicated ad-hoc query/troubleshooting interface (separate from building permanent dashboards) for quickly running queries, exploring logs, and debugging issues across any connected data source — designed for fast, iterative investigation rather than building a polished, reusable dashboard.

**10. How do you provision Grafana dashboards and data sources as code (rather than manually through the UI)?**
Grafana supports **provisioning** via YAML configuration files (for data sources) and JSON dashboard definitions placed in specific provisioning directories, automatically loaded on Grafana startup — this lets dashboards and data sources be version-controlled in Git and deployed consistently across environments, instead of manually recreating them through the UI (which doesn't scale and isn't reproducible).

**11. What is the difference between a Grafana "Time series" panel and a "Stat" panel?**
A **Time series panel** shows how a value changes over time as a line/area graph — best for trends and patterns. A **Stat panel** shows a single current (or aggregated) numeric value prominently, often with a sparkline and color-coded thresholds — best for at-a-glance current status (e.g., "current error rate: 2.3%") rather than historical trend analysis.

**12. What are Grafana annotations, and how are they used?**
Annotations are markers overlaid on graphs at specific points in time, typically representing events (deployments, incidents, config changes) — letting you visually correlate a metric change (e.g., a latency spike) with an event that likely caused it (e.g., a deployment that happened at the same timestamp).

**13. How does Grafana handle multiple data sources in a single dashboard?**
Different panels within the same dashboard can each query entirely different data sources independently (e.g., one panel showing Prometheus metrics, another showing Elasticsearch logs, another querying a SQL database) — Grafana doesn't require a dashboard to be tied to a single data source, enabling unified observability views combining metrics, logs, and other data types.

---

### 🔴 Advanced

**14. 🎯 Scenario: You manage Grafana dashboards across 30+ microservices, and manually maintaining each dashboard is unsustainable. How do you scale dashboard management?**
1. Use **dashboard provisioning as code** — store dashboard JSON definitions in Git, deployed automatically via CI/CD when changes are merged, rather than relying on manual UI edits that can drift or be lost.
2. Build **dashboard templates with variables** (e.g., a `$service` variable) so a single template dashboard can represent any of the 30+ services by simply switching the variable, instead of maintaining 30 nearly-identical dashboard copies.
3. Use tools like **Grafonnet** (Jsonnet library for Grafana) or **grafana-dashboard-managers** to programmatically *generate* dashboard JSON from reusable code patterns, ensuring consistency (same panel layout, same thresholds, same naming conventions) across all services rather than hand-crafting each one.
4. Consider the **Grafana Terraform provider** to manage dashboards, data sources, alert rules, and folder structure entirely as Terraform-managed infrastructure, integrated into the same IaC pipeline as the rest of the platform.

**15. What is the Grafana plugin architecture, and how do panel plugins extend default functionality?**
Grafana supports a plugin system with three main types: **Panel plugins** (new visualization types beyond the built-in graph/table/stat panels — e.g., a flame graph or a custom map visualization), **Data source plugins** (connectors to systems Grafana doesn't natively support), and **App plugins** (full mini-applications embedded within Grafana, often bundling related panels/data sources/pages for a specific use case). Plugins are installed via the Grafana CLI or plugin catalog and extend Grafana's capabilities without modifying its core codebase — enabling integration with virtually any data system through community or vendor-built plugins.

**16. How would you design Grafana dashboards to support effective incident response (i.e., what makes a dashboard "good" for on-call engineers)?**
1. Structure dashboards hierarchically — a high-level **overview dashboard** (RED metrics: Rate, Errors, Duration, or USE metrics: Utilization, Saturation, Errors) that surfaces overall service health at a glance, with drill-down links to more detailed per-component dashboards rather than a single dashboard cramming everything together.
2. Use consistent, well-chosen **thresholds and color coding** (red/yellow/green) so an on-call engineer can identify a problem within seconds without needing to interpret raw numbers under stress at 3 AM.
3. Correlate metrics with **annotations** for deployments/config changes, so a sudden anomaly can immediately be checked against "did something change recently?"
4. Include direct links from dashboard panels to relevant logs (e.g., a Loki/Elasticsearch data source link scoped to the same time range and service) so engineers can pivot from "metric looks wrong" to "here's the actual error" without manually re-navigating to a different tool.
5. Avoid dashboard sprawl/duplication — a small number of well-maintained, trusted dashboards is far more valuable during an incident than dozens of stale, inconsistent ones where engineers aren't sure which one reflects current reality.

**17. 🎯 Scenario: A Grafana dashboard query against Prometheus is timing out / loading extremely slowly for a 30-day time range, though it loads fine for 1 hour. How do you diagnose and fix this?**
1. Check the underlying PromQL query's **step/resolution** — Grafana auto-calculates a query resolution based on the panel width and time range; for very long ranges, ensure the dashboard isn't forcing an unnecessarily fine-grained step that causes Prometheus to compute and return far more data points than the panel can even visually display.
2. Verify whether the query involves **high-cardinality** metrics or expensive aggregations (e.g., `sum by (very_high_cardinality_label)`) that become disproportionately expensive over a longer time range as more raw samples must be scanned.
3. Consider whether the underlying Prometheus instance's **retention/storage performance** is the actual bottleneck (e.g., querying 30 days of raw 15-second-interval data is fundamentally more expensive than 1 hour) — for genuinely long-range dashboard use cases, consider using **recording rules** in Prometheus to pre-compute and store expensive aggregations at a coarser interval, so Grafana queries a much smaller, pre-aggregated dataset instead of re-computing the full raw aggregation on every dashboard load.
4. If using a long-term storage solution (Thanos/Cortex/Mimir), verify the query is hitting an appropriately downsampled resolution tier for long time ranges, rather than scanning raw high-resolution data unnecessarily.

---

## 14. Datadog

### 🟢 Beginner

**1. What is Datadog, and what is it used for?**
Datadog is a commercial, SaaS-based observability and monitoring platform that unifies metrics, logs, traces (APM), security monitoring, and synthetic testing into a single product — collected via a lightweight Agent installed on hosts/containers, eliminating the need to self-host and integrate separate tools (like Prometheus + Grafana + ELK) for each observability pillar.

**2. What is the Datadog Agent?**
A lightweight background process installed on hosts, containers, or Kubernetes Nodes that collects metrics, logs, and traces from the system and running applications, then forwards them securely to Datadog's cloud backend for storage, visualization, and alerting.

**3. What is the difference between Datadog and Prometheus + Grafana?**
Prometheus + Grafana is a self-hosted, open-source combination requiring you to manage infrastructure, storage, and scaling yourself, but offering full control and no per-host licensing cost. Datadog is a fully-managed commercial SaaS platform — no infrastructure to manage, broader out-of-the-box integrations (700+) and unified metrics/logs/traces/APM in one product, but with ongoing subscription costs (typically priced per host/container).

**4. What is a Datadog Dashboard?**
A customizable visualization screen combining multiple widgets (graphs, timeseries, top-lists, heatmaps) displaying metrics, logs, or APM data — conceptually similar to a Grafana dashboard but native to the Datadog platform.

**5. What is a Datadog Monitor?**
Datadog's equivalent of an alerting rule — a configured condition (e.g., "CPU usage > 90% for 5 minutes") that, when breached, triggers a notification through configured channels (Slack, PagerDuty, email, etc.).

**6. What are Datadog Tags, and why are they important?**
Tags are key-value labels (`env:production`, `service:checkout`, `team:payments`) attached to hosts, containers, and metrics — used extensively throughout Datadog to filter, group, and aggregate data across dashboards, monitors, and APM traces. A consistent tagging strategy is foundational to making Datadog usable at scale.

---

### 🟡 Intermediate

**7. What is Datadog APM (Application Performance Monitoring), and how does distributed tracing work within it?**
Datadog APM instruments application code (via language-specific tracing libraries) to capture **traces** — a record of a single request's full journey across all the services it touches, broken into **spans** (individual operations like a database query or an internal function call). This lets engineers see exactly where time is spent across a distributed system for any given request, identifying which specific downstream service or query caused slowness, rather than just knowing "the request was slow" at an aggregate level.

**8. What is the difference between a Datadog Metric Monitor and a Log Monitor?**
A **Metric Monitor** alerts based on numeric time-series data crossing a threshold (e.g., CPU, request latency). A **Log Monitor** alerts based on the volume or pattern of log events matching a specific query (e.g., "alert if more than 50 logs matching `status:error service:payments` occur in 5 minutes") — useful for alerting on application-level error conditions that aren't naturally expressed as a numeric metric.

**9. What is the Datadog Cluster Agent in a Kubernetes deployment?**
A separate, centralized component (distinct from the per-Node Datadog Agent DaemonSet) that reduces load on the Kubernetes API server by centralizing API queries (instead of every node-level Agent querying the API independently), handles Cluster-level checks (like Kubernetes Events and the External Metrics Provider for HPA autoscaling based on Datadog metrics), and improves overall scalability of Datadog's Kubernetes integration in large clusters.

**10. What is the difference between Datadog's "Logs without Limits" and standard log ingestion?**
Standard log management ties cost/retention directly to ingestion volume. **Logs without Limits** decouples *ingestion* (collecting and briefly retaining all logs for live tailing/search) from *indexing* (long-term, fully-searchable storage), letting teams apply filters to selectively index only the logs that matter for long-term retention/search, while still being able to ingest (and briefly access) the full volume — controlling cost without losing visibility into the full log stream during live troubleshooting.

**11. What is a Datadog Synthetic Test?**
An automated, scheduled test that simulates user interactions (API calls, full browser-based multi-step user journeys) from various global locations, proactively detecting availability or performance issues *before* real users are affected — distinct from passive monitoring, which only reacts to data generated by actual production traffic.

---

### 🔴 Advanced

**12. 🎯 Scenario: Your Datadog monthly bill has grown 5x in the past 6 months without a proportional increase in infrastructure. How do you diagnose and control costs?**
1. Review **custom metrics** usage first — Datadog bills custom metrics based on the number of unique metric+tag combinations; a common cost driver is unintentionally high-cardinality tags (e.g., tagging a metric with a unique `request_id` or `pod_name` that constantly changes) causing massive proliferation of billed custom metric series.
2. Audit **log indexing volume** — apply exclusion filters for high-volume, low-value logs (e.g., health check endpoint logs, debug-level logs in production) so they're ingested for live visibility but not indexed/retained long-term, since indexed log volume is a major cost driver.
3. Review **host/container count** — check for orphaned/zombie Agent installations on decommissioned infrastructure still reporting and being billed.
4. Use Datadog's own **Usage and Cost Management** dashboards to break down spend by product (APM, Logs, Infrastructure, Custom Metrics) and team/tag, identifying exactly which area drove the increase rather than guessing.
5. Implement **tag governance** policies (Datadog supports tag-based access/usage restrictions) to prevent uncontrolled custom metric/tag cardinality growth going forward, treating tagging conventions as a reviewed, intentional practice rather than ad-hoc.

**13. Explain how Datadog's APM trace sampling works, and why intelligent sampling matters at high request volumes.**
At high throughput, capturing and storing 100% of traces becomes prohibitively expensive and often unnecessary for getting representative performance insight. Datadog uses a combination of **head-based sampling** (a decision made at the start of a trace, before knowing the outcome) and intelligent strategies that ensure **all error traces and traces exceeding a latency threshold are always retained** (even when overall sampling rate is low), while routine, fast, successful requests are sampled at a much lower rate — this preserves visibility into exactly the traces that matter most for debugging (errors, slow requests) without the cost/overhead of storing every single successful, fast request.

**14. 🎯 Scenario: You need to correlate a spike in application errors with a specific code deployment, across metrics, logs, and traces simultaneously. How do you architect this in Datadog?**
1. Ensure **consistent tagging** across all telemetry types — specifically tagging metrics, logs, and traces with the same `version` or `git.commit.sha` tag at deployment time (often automated via CI/CD injecting the tag as an environment variable consumed by the Datadog Agent/tracer).
2. Use Datadog's **Deployment Tracking** / Events API to programmatically post a deployment marker event from your CI/CD pipeline at the moment of each deploy — this automatically overlays a vertical marker on dashboards and APM views, letting you visually correlate a metric/error spike with the exact deployment timestamp.
3. Leverage **Datadog's unified service tagging** (`env`, `service`, `version` as standardized tag keys) which Datadog's UI specifically recognizes to automatically link related metrics, logs, and traces for the same service/version together in a single correlated view, rather than requiring manual cross-referencing across separate, disconnected dashboards.

**15. What is the difference between Datadog's Infrastructure monitoring "Live Containers" view and standard host-based metrics, particularly in a Kubernetes environment?**
Standard host-based metrics treat each VM/Node as the primary unit, which becomes insufficient in Kubernetes where many ephemeral containers/Pods run on each Node with constantly changing identities. **Live Containers** provides a real-time, container/Pod-level view (refreshed every 2 seconds) showing resource usage, health, and metadata at the granularity that actually matters in a containerized environment — letting engineers see, for example, exactly which specific Pod replica out of 50 is consuming excessive CPU, rather than only seeing aggregate Node-level metrics that obscure which individual workload is responsible.

---

## 15. SonarQube

### 🟢 Beginner

**1. What is SonarQube, and what problem does it solve?**
SonarQube is an open-source (with commercial editions) platform for continuous code quality and security analysis — it performs static code analysis to automatically detect bugs, vulnerabilities, code smells, and security hotspots in source code, before they reach production, integrated directly into the CI/CD pipeline.

**2. What is the difference between a "Bug," a "Vulnerability," and a "Code Smell" in SonarQube terminology?**
A **Bug** is a coding error likely to cause incorrect behavior at runtime (e.g., a null pointer dereference). A **Vulnerability** is a security-related weakness that could be exploited (e.g., SQL injection risk, hardcoded credentials). A **Code Smell** is a maintainability issue — code that works correctly but is poorly structured, hard to understand, or violates best practices (e.g., excessive complexity, duplicated code) — not an immediate bug, but a long-term maintenance/technical-debt risk.

**3. What is a SonarQube Quality Gate?**
A set of pass/fail conditions (e.g., "no new bugs," "code coverage on new code ≥ 80%," "no new critical vulnerabilities") that a project's analysis results must meet. If a Quality Gate fails, it's typically used to block a CI/CD pipeline from proceeding (e.g., blocking a PR merge or deployment) until the issues are addressed.

**4. What is SonarScanner?**
The command-line tool (or build-tool-specific plugin like the Maven/Gradle SonarScanner) that analyzes your source code locally/in CI and sends the analysis results to the SonarQube server for processing and display.

**5. What is the difference between SonarQube and SonarCloud?**
SonarQube is the self-hosted version, installed and managed on your own infrastructure. SonarCloud is SonarSource's fully-managed cloud SaaS offering, with native integrations for GitHub/GitLab/Bitbucket/Azure DevOps, requiring no infrastructure management — functionally similar analysis engine, different deployment/hosting model.

---

### 🟡 Intermediate

**6. How does SonarQube's static analysis differ fundamentally from dynamic analysis (like DAST tools)?**
Static analysis examines source code *without executing it* — analyzing syntax, structure, and data flow patterns to identify potential issues. This means it can catch issues early (even before the code compiles/runs) and analyze 100% of code paths theoretically, but it can also produce false positives (flagging something that isn't actually exploitable in practice) since it lacks runtime context. Dynamic analysis (DAST) tests the actual running application, catching real runtime/configuration issues static analysis can't see, but only covers the code paths actually exercised during testing.

**7. What is "New Code" vs "Overall Code" analysis in SonarQube, and why does this distinction matter practically?**
**Overall Code** analysis covers the entire codebase, including legacy code that may have accumulated significant technical debt over years. **New Code** analysis (typically defined as a specific time period or comparison against a baseline branch) focuses only on recently changed/added code. This distinction matters because demanding an immediate 0-issues standard across an entire large legacy codebase is often impractical — the common, pragmatic approach is to enforce strict Quality Gates on **New Code** (preventing new technical debt from being introduced) while gradually addressing the Overall Code backlog separately, rather than blocking all development until the entire historical codebase is "clean."

**8. How do you integrate SonarQube analysis into a Jenkins/GitLab CI pipeline, and how does the Quality Gate check work asynchronously?**
After the SonarScanner step runs and uploads analysis results, SonarQube processes them on its server side asynchronously (not instantly within the same pipeline step). The pipeline must then explicitly **wait for and poll** the Quality Gate status (e.g., using the `waitForQualityGate` step in Jenkins's SonarQube Scanner plugin, configured via a webhook from SonarQube notifying the CI server when analysis/Quality Gate evaluation completes) before deciding whether to proceed or fail the build — a common mistake is checking the Quality Gate status too early, before SonarQube has finished processing.

**9. What is a Security Hotspot, and how does it differ from a Vulnerability in SonarQube?**
A **Vulnerability** is a confirmed, exploitable security weakness. A **Security Hotspot** flags security-sensitive code (e.g., use of cryptography, deserialization, file path handling) that *requires manual human review* to determine whether it's actually exploitable in context — SonarQube can't always automatically determine if surrounding code properly mitigates the risk, so Hotspots are explicitly routed to a developer/security reviewer for a manual "Safe" or "Fix needed" determination, rather than being automatically scored as a guaranteed issue.

**10. What is the role of a Quality Profile in SonarQube?**
A Quality Profile defines the specific set of rules (and their severity) that SonarQube's analysis engine applies for a given programming language — different teams/projects can use different Quality Profiles (e.g., a stricter profile for a new greenfield project, a more lenient one for a legacy codebase being gradually improved) rather than being forced into one universal rule set.

---

### 🔴 Advanced

**11. 🎯 Scenario: Your team's SonarQube Quality Gate is blocking nearly every PR due to a huge backlog of pre-existing code smells in a 5-year-old legacy codebase, frustrating developers and creating pressure to bypass the gate entirely. How do you fix this?**
1. Reconfigure the Quality Gate to evaluate only **New Code** conditions (e.g., "0 new bugs," "new code coverage ≥ 80%"), explicitly excluding pre-existing Overall Code issues from blocking new PRs — this is the standard, intended SonarQube usage pattern for legacy codebases, not a workaround.
2. Set a clear **New Code definition** (e.g., "code changed since the previous release" or "code changed in the last 30 days") aligned with the team's actual release cadence.
3. Separately track and prioritize **technical debt paydown** for the legacy backlog as its own deliberate, scheduled work (not blocking unrelated feature PRs), using SonarQube's technical debt estimate to help prioritize which legacy issues offer the highest risk reduction per unit of remediation effort.
4. This reframes the gate from "you must fix 5 years of debt before merging your bugfix" (which inevitably leads to teams disabling the gate entirely, losing all value) to "you cannot introduce *new* debt," which is both achievable and actually improves the codebase trajectory over time.

**12. How does SonarQube's taint analysis work for detecting vulnerabilities like SQL injection, and what are its limitations?**
Taint analysis tracks the flow of data from an untrusted **source** (e.g., user input from an HTTP request parameter) through the code to a sensitive **sink** (e.g., a raw SQL query execution), flagging a vulnerability if untrusted data reaches a sensitive sink without passing through a recognized **sanitizer** (e.g., parameterized query binding, input validation) along the path. Limitations: taint analysis is inherently a static approximation — it can produce false positives when a sanitizer pattern isn't recognized by the engine's rule set even though the code is actually safe, and false negatives when data flows through complex/dynamic code paths (e.g., reflection, certain framework "magic") that the static analyzer can't fully trace — which is precisely why Security Hotspots exist for human judgment rather than 100% automated, blind enforcement.

**13. 🎯 Scenario: SonarQube analysis is adding 8+ minutes to every CI pipeline run, significantly slowing down developer feedback loops. How do you optimize this?**
1. Verify whether **incremental analysis** is being leveraged correctly — ensure the SonarScanner is configured to analyze only relevant changed files where supported, rather than always performing a full project re-analysis if avoidable for the language/build tool.
2. Check if code coverage report generation (a common prerequisite/input to SonarQube analysis) is itself the actual bottleneck rather than SonarQube analysis proper — profile the pipeline stage timings precisely before optimizing the wrong component.
3. Consider running the full, strict SonarQube analysis (with Quality Gate blocking) only on PR/merge-to-main events, while using a lighter-weight local pre-commit linter for instant per-commit feedback during active development — not every single commit needs to wait on a full server-side SonarQube round-trip.
4. Ensure adequate SonarQube server resources (CPU/memory, and database performance if using an underpowered backing database) — server-side processing capacity is a common, often-overlooked bottleneck distinct from the scanning step itself.
5. If using SonarQube Community Edition, confirm whether features like more granular/faster incremental analysis are available only in Developer/Enterprise editions, which may justify a licensing upgrade if analysis speed is a genuine organizational bottleneck at scale.

**14. What is the relationship between SonarQube's technical debt ratio and the maintainability rating (A–E), and how should this metric be used (and not misused) by engineering leadership?**
SonarQube calculates a **Technical Debt Ratio** — the estimated remediation cost (in time) of all maintainability issues, divided by the estimated cost to develop the codebase from scratch — and maps this ratio to a letter rating (A = ≤5%, through E = >50%). This is a useful **directional, relative trend indicator** (is the codebase's maintainability improving or degrading release-over-release?) but should not be treated as a precise, objective absolute measurement, since the underlying "cost to develop from scratch" and "remediation time" estimates are themselves rough heuristics, not empirically measured values specific to your team/domain. Misuse occurs when leadership treats the letter grade as a rigid KPI/performance metric for individual developers (incentivizing gaming the metric) rather than as one input among several for understanding overall codebase health trends.

---

## 16. Trivy

### 🟢 Beginner

**1. What is Trivy, and what is it used for?**
Trivy is an open-source, all-in-one security scanner (by Aqua Security) primarily used to detect vulnerabilities in container images, filesystems, and Git repositories — scanning for known CVEs in OS packages and application dependencies, as well as misconfigurations, exposed secrets, and software license issues.

**2. What types of scans can Trivy perform?**
Trivy supports several scan targets: **Image** scanning (container images for OS package and application dependency vulnerabilities), **Filesystem** scanning (local directories/source code), **Repository** scanning (remote Git repos directly), **Kubernetes** scanning (live cluster resources), and **IaC/Config** scanning (Terraform, Dockerfile, Kubernetes YAML misconfigurations).

**3. How do you run a basic Trivy scan on a Docker image?**
```bash
trivy image nginx:1.21
```
This downloads (or uses cached) vulnerability databases and reports any known CVEs found in the OS packages and application libraries within the specified image, along with their severity (LOW, MEDIUM, HIGH, CRITICAL).

**4. What is a CVE?**
CVE (Common Vulnerabilities and Exposures) is a standardized identifier (e.g., `CVE-2023-12345`) for a publicly known security vulnerability, maintained in a shared database referenced by virtually all vulnerability scanners — providing a common reference point across different tools and vendors for the same underlying issue.

**5. What is the difference between scanning for vulnerabilities and scanning for misconfigurations in Trivy?**
**Vulnerability scanning** checks installed software packages/libraries against known CVE databases (is this *specific version* of OpenSSL known to be vulnerable?). **Misconfiguration scanning** checks configuration files (Dockerfiles, Kubernetes manifests, Terraform) against security best-practice rules, regardless of any specific CVE (e.g., "this Dockerfile runs as root," "this Kubernetes Pod has no resource limits," "this S3 bucket is publicly readable") — two fundamentally different classes of risk that Trivy can detect within a single unified tool.

---

### 🟡 Intermediate

**6. How would you integrate Trivy into a CI/CD pipeline to fail the build on critical vulnerabilities?**
```bash
trivy image --exit-code 1 --severity CRITICAL,HIGH myapp:latest
```
The `--exit-code 1` flag makes Trivy return a non-zero exit code if any matching vulnerabilities are found (rather than its default behavior of always exiting 0 regardless of findings) — this non-zero exit code is what causes a CI/CD pipeline step to actually fail and block the pipeline, rather than just printing a report that gets ignored.

**7. What is the difference between scanning an image for OS package vulnerabilities versus application/language dependency vulnerabilities?**
**OS package vulnerabilities** come from the base image's package manager (e.g., `apt`/`apk` packages like `openssl`, `glibc`) — Trivy detects the OS distribution and checks installed package versions against distro-specific vulnerability feeds. **Application dependency vulnerabilities** come from language-specific package managers (e.g., `npm`/`package-lock.json`, Python's `requirements.txt`, Java's `pom.xml`/`.jar` files bundled inside the image) — Trivy parses these language-specific lockfiles/manifests and checks them against language-ecosystem-specific vulnerability databases, a separate detection mechanism from OS package scanning, both running as part of the same `trivy image` scan.

**8. How does Trivy detect hardcoded secrets in code/images, and what's a practical use case?**
Trivy includes a built-in secret scanner that uses regex-based pattern matching against common secret formats (AWS access keys, private keys, API tokens, etc.) to detect accidentally hardcoded credentials within scanned filesystems, repositories, or container image layers. A practical use case: scanning a Docker image before pushing it to a registry to catch a scenario where a developer accidentally left a `.env` file or hardcoded API key inside a `COPY . .` instruction, which would otherwise silently ship a live credential inside a publicly distributable image layer.

**9. What is the difference between `trivy image` and `trivy fs` (filesystem scan)?**
`trivy image` scans a built container image (including all its layers, OS packages, and any artifacts already baked into the image). `trivy fs` scans a local directory/filesystem directly — useful for scanning source code repositories *before* they're even built into an image (e.g., as an early CI step right after checkout, catching vulnerable dependency declarations in `package.json`/`requirements.txt` before spending time building a Docker image that would contain them).

**10. How does Trivy handle its vulnerability database, and why does database freshness matter for accurate scanning?**
Trivy downloads and locally caches a vulnerability database (aggregated from multiple sources: NVD, GitHub Security Advisories, distro-specific security trackers) which it refreshes periodically (`trivy image --download-db-only` to pre-fetch). If the local database is stale, newly disclosed CVEs won't be detected even if the actual scan runs successfully — in CI/CD environments, especially with ephemeral runners that don't persist a cache between runs, this means every pipeline run may re-download the database, which can be optimized via a shared cache or a dedicated Trivy server mode to avoid repeated full downloads and ensure consistent up-to-date results.

---

### 🔴 Advanced

**11. 🎯 Scenario: Your CI pipeline blocks every build due to Trivy detecting hundreds of LOW and MEDIUM severity vulnerabilities in a base image you don't control (e.g., a vendor-provided image), making the gate impractical. How do you handle this?**
1. First, distinguish genuine risk reduction from noise — scope the Quality Gate to only fail on **CRITICAL/HIGH** severity findings that have an available fix (`--ignore-unfixed` flag excludes vulnerabilities with no available patch yet, since blocking a build over an unfixable issue provides no actionable remediation path and just creates pipeline friction).
2. Use a `.trivyignore` file to explicitly document and suppress specific accepted-risk CVEs (with a clear justification/expiry comment) that have been manually reviewed and assessed as not exploitable in your specific deployment context — making risk acceptance an explicit, auditable decision rather than just disabling the scanner entirely.
3. If the base image itself is the root cause, evaluate switching to a smaller, better-maintained base image (e.g., Distroless or an actively-patched minimal image) that has substantially fewer OS packages and therefore fewer potential CVEs by design, rather than only treating symptoms in each derived image downstream.
4. Establish a recurring (not just CI-blocking) vulnerability review process for the LOW/MEDIUM findings that aren't blocking builds, so they aren't simply ignored forever — tracked separately as backlog/technical debt rather than either blocking all velocity or being silently dropped.

**12. How does Trivy's Kubernetes scanning mode work, and how does it differ from scanning a static Kubernetes YAML manifest?**
`trivy k8s` connects directly to a **live Kubernetes cluster** (via the configured kubeconfig context) and scans actually-running resources — Pods, Deployments, ConfigMaps, RBAC configuration — checking both the container images currently running (vulnerability scanning) and the live resource configurations (misconfiguration scanning, e.g., checking actual deployed RBAC bindings for overly permissive grants). This differs from scanning a *static* YAML manifest file (`trivy config k8s-manifest.yaml`), which only validates the configuration as written in source control — it can't detect drift where the live cluster state has diverged from the committed manifest (e.g., someone manually `kubectl edit`-ed a Deployment in production), making live cluster scanning a necessary complement to pre-deployment static scanning, not a replacement for it.

**13. 🎯 Scenario: You need to implement a "shift-left" security strategy using Trivy across the full software delivery lifecycle — from a developer's local machine through to a running production cluster. Design this end-to-end.**
1. **Local/IDE stage**: developers run `trivy fs .` or use a pre-commit hook to catch obvious issues (hardcoded secrets, critically vulnerable dependencies) before even committing code — fastest, cheapest point to catch an issue.
2. **CI pipeline stage**: on every PR, run `trivy fs` (source/dependency scan) and `trivy config` (Dockerfile/Terraform/K8s manifest misconfiguration scan) as fast, early gates; after the image is built, run `trivy image` for the full OS+application vulnerability scan, gating merge/deployment on CRITICAL/HIGH findings with available fixes.
3. **Registry stage**: integrate Trivy (or a registry's native scanning, often Trivy-based under the hood, like Harbor or AWS ECR's enhanced scanning) to continuously re-scan images already stored in the registry, since a CVE can be newly disclosed *after* an image was originally scanned and pushed — yesterday's "clean" image might be vulnerable today purely because the vulnerability database has been updated, not because the image itself changed.
4. **Runtime/cluster stage**: schedule periodic `trivy k8s` scans against the live production cluster to catch drift and continuously monitor currently-running workloads against the latest vulnerability data, feeding findings into a centralized dashboard/ticketing system for ongoing remediation tracking.
5. Centralize all stages' findings into a unified reporting format (Trivy supports SARIF/JSON output) feeding into a security dashboard, so the security posture is visible holistically across the entire pipeline rather than siloed per-stage with no aggregate visibility.

**14. What is the difference between Trivy's vulnerability scanning and Software Bill of Materials (SBOM) generation, and how do they relate to supply chain security?**
Vulnerability scanning answers "does this software currently contain any known vulnerabilities?" at the time of the scan. **SBOM generation** (`trivy image --format cyclonedx` or `--format spdx-json`) produces a comprehensive, structured inventory of every software component/dependency (with exact versions) present in an artifact — independent of whether any of them are *currently* known to be vulnerable. The critical supply-chain-security value of an SBOM is that it enables **retroactive vulnerability matching**: when a brand-new CVE is disclosed in the future for some library, an organization with archived SBOMs can instantly query "which of our deployed artifacts, built months or years ago, contain this now-vulnerable component?" without needing to re-scan every historical image — a capability pure point-in-time vulnerability scanning alone doesn't provide, since it only reflects the vulnerability landscape known *at scan time*.

**15. 🎯 Scenario: Trivy flags a CRITICAL vulnerability in a deeply transitive dependency (a dependency-of-a-dependency-of-a-dependency) that your team has no direct control over and no upstream fix is yet available. What's your remediation strategy?**
1. **Assess actual exploitability in context** — determine whether the vulnerable code path within that transitive dependency is actually reachable/exercised by your application's usage pattern (many CVEs are in code paths never actually invoked by a given consumer), using Trivy's reachability/usage analysis where available, or manual code review.
2. Check if a **direct dependency override** is possible — many package managers (npm's `overrides`/`resolutions`, Maven's dependency management section) allow forcing a specific transitive dependency to a patched version even if the direct parent dependency hasn't officially updated its declared requirement yet.
3. If no fix or safe override exists, evaluate **compensating controls** — e.g., network segmentation limiting exposure, WAF rules blocking the specific exploit pattern, or runtime monitoring specifically watching for exploitation attempts of that known vulnerability — while formally documenting the accepted risk (with `.trivyignore` and a clear expiry/review date) rather than leaving it as an indefinitely silent gap.
4. Track the issue proactively upstream — open or follow an issue with the maintainer of the direct dependency to push for them updating their own transitive dependency, and set a recurring reminder to re-evaluate once a fix becomes available, rather than treating the ignore as permanent.

---
