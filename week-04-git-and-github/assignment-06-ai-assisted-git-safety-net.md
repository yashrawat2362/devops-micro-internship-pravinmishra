# Assignment 6 — Building an AI-Assisted Git Safety Net (PR Ready Check)

Part of the DevOps Micro Internship (DMI) Cohort 3 with Agentic AI

---

## Purpose

In Week 2 you built Claude Code hooks that block a dangerous action *before* it happens (`PreToolUse`), and a restricted skill that could look but not touch (`allowed-tools` without `Write`). In this assignment you will discover that Git has the exact same idea, decades older: a **pre-commit hook** that blocks a commit before it's created.

You will build both halves of a real "PR Ready" workflow:

1. A **Git hook that follows fixed rules** — scans staged changes for hardcoded secrets and oversized files and refuses the commit. No AI involved, no guessing, just a rule that gives the same answer every time.
2. A **restricted Claude Code skill** (`/pr-ready`) that reads your staged diff and drafts a Pull Request title, description, and a short list of things worth a second look — the kind of judgment a fixed rule can't make (mixed changes, missing context, unclear intent). The skill never commits, pushes, or opens the PR. You do that yourself, using its draft as a starting point.

This mirrors the Agentic Loop from Week 3's Linux triage assignment: **Gather → Analyze → Human Act → Verify**. The hook and the skill both gather and analyze; only you act.

---

# Task 0 — Confirm Your Fork and Create a Feature Branch

## Goal

Confirm you are working in your own fork, then create a dedicated branch for this assignment.

### Evidence

#### Screenshot 1 — Output of git remote -v and git branch showing the new branch

![ss-1](./screenshots/ss4.6.1.png)

---

### Notes

**1. Why create a dedicated branch instead of doing this work on main?**

A dedicated branch keeps the main branch stable, isolates changes, and makes it easier to review, test, and merge the work safely.

---

# Task 1 — Stage a Change With Realistic Risk

## Goal

On your own fork of this repository (the one you've been submitting your DMI work in since onboarding), create a new branch and stage a change that a real reviewer should catch: a hardcoded-looking secret and a leftover debug statement.

### Evidence

#### Screenshot 1 — Output of  `git status` showing the staged file on feature/ai-pr-ready

![ss-2](./screenshots/ss4.6.2.png)

---

### Notes

**1. Why does this assignment use an obviously fake key instead of a real one?**

The assignment uses a fake key to safely demonstrate how the pre-commit hook detects secrets without exposing any real credentials or creating a security risk.

---

# Task 2 — Write a Real Git Pre-Commit Hook

## Goal

Create a tracked, shareable pre-commit hook that blocks a commit containing secret-like patterns or files over 1MB.

### Evidence

#### Screenshot 2 — `hooks/pre-commit` open in VS Code showing the full script

![ss-3](./screenshots/ss4.6.3.png)

---

#### Screenshot 3 — Output of `git config core.hooksPath` confirming it points to `hooks`

![ss-4](./screenshots/ss4.6.4.png)

---

### Notes

**1. Why is `hooks/pre-commit` tracked in the repo instead of living only in `.git/hooks/`?**

Tracking `hooks/pre-commit` in the repository allows every team member to use the same shared hook, ensuring consistent checks across the project. Hooks stored only in `.git/hooks/` are local to one machine and are not shared with others.

---

**2. Compare this to `PreToolUse` from Week 2 Assignment 6. What does each one intercept, and what do they have in common?**

`hooks/pre-commit` intercepts Git commits before they are created, while `PreToolUse` intercepts tool actions before they are executed by the AI. Both act as safety checks that prevent risky operations before they happen.

---

# Task 3 — Prove the Hook Blocks the Risky Commit

## Goal

Attempt to commit the staged file from Task 1 and show the hook rejecting it.

### Evidence

#### Screenshot 4 — Terminal showing `git commit` rejected with the hook's "BLOCKED" message naming the exact file

![ss-5](./screenshots/ss4.6.5.png)

---

### Notes

**1. Which line in `hooks/pre-commit` matched your fake key, and why did it match?**

The line containing `AWS_ACCESS_KEY_ID=AKIAABCDEFGHIJKLMNOP` matched because the hook searches for the pattern `AKIA[0-9A-Z]{16}`, which is the format of an AWS access key.

---

**2. Could this hook have caught a poorly-named variable that stores a secret without the `AKIA` prefix? What does that tell you about the limits of a fixed rule like this?**

No. It would not detect a secret without the `AKIA` pattern. This shows that fixed-rule checks only catch known patterns and cannot identify every possible secret or security issue.

---

# Task 4 — Build the `/pr-ready` Skill

## Goal

Create a manually invoked Claude Code skill that reads your staged changes and produces a PR-readiness report and a draft PR description — without writing, committing, or pushing anything itself.

### Evidence

#### Screenshot 5 — `SKILL.md` frontmatter showing `allowed-tools: Bash, Read, Grep` (no `Write`) and `disable-model-invocation: true`

![ss-6](./screenshots/ss4.6.6.png)

---

#### Screenshot 6 — `/pr-ready` output while the risky file is still staged, showing it flagged the secret and/or debug statement

![ss-7](./screenshots/ss4.6.7.png)

---

### Notes

**1. Why does `/pr-ready` have `Bash` and `Read` but not `Write`?**

`/pr-ready` uses Bash to inspect the staged Git changes and Read to examine the files, but it does not have Write permission so it cannot modify files or perform Git actions. This ensures Claude only reviews and suggests changes, while the human remains responsible for all edits and Git operations.

---

**2. The pre-commit hook and `/pr-ready` both looked at the same staged diff. Did they flag the same things? What did one catch that the other didn't?**

Both detected the hardcoded AWS key in the staged changes. The pre-commit hook blocked the commit based on its fixed rule for secret patterns, while `/pr-ready` also identified the leftover debug statement and explained why these changes were risky, providing a draft PR title and description for human review.

---

# Task 5 — Fix the Issues and Re-Verify

## Goal

Remove the secret and debug statement, then prove both gates now pass clean.

### Evidence

#### Screenshot 7 — `git commit` succeeding after the fix (no BLOCKED message)

![ss-8](./screenshots/ss4.6.8.png)

---

#### Screenshot 8 — Second `/pr-ready` run showing a clean risk report and a drafted PR title + description

![ss-9](./screenshots/ss4.6.9.png)

---

### Notes

**1. What exactly did you change to satisfy the pre-commit hook?**

I removed the fake AWS access key and the debug echo statement from `scripts/notify.sh`, then staged the updated file again. After these changes, the pre-commit hook passed and the commit was allowed.

---

# Task 6 — Push and Open a Pull Request Using the AI Draft

## Goal

Push your branch and open a real Pull Request, using `/pr-ready`'s drafted title and description as your starting point — read it critically and edit before you use it.

**Important:** Open this Pull Request with base repository set to **your own fork** — not the shared upstream `pravinmishraaws/devops-micro-internship-pravinmishra` repository. This assignment's hook and skill files are your own practice work, not a change meant for the shared class repo.

### Evidence

#### Screenshot 9 — Your Pull Request showing the base repository is your own fork, plus the title and description, with the `/pr-ready` draft visible for comparison (paste it in the PR conversation or your notes below)

![ss-10](./screenshots/ss4.6.10.png)

---

#### PR Link

`https://github.com/pravinmishraaws/devops-micro-internship-interviews/pull/405`

---

### Notes

**1. What, if anything, did you edit in the AI's drafted PR description before using it? Why?**

I reviewed and updated the PR description to ensure it accurately described my changes and removed or corrected any unclear or inaccurate details before submitting it.

---

**2. If you had blindly copy-pasted the AI's draft without reading it, what could go wrong?**

The PR description could contain incorrect, incomplete, or misleading information, which might confuse reviewers or misrepresent the actual changes.

---

**3. Why does this PR need to target your own fork instead of the shared upstream repository?**

The assignment requires the PR to target my own fork because I do not have direct write access to the shared upstream repository, and it keeps my assignment work separate from the official project.

---

# Task 7 — Map the Workflow to the Agentic Loop

## Goal

Explain this assignment's workflow using the same Gather → Analyze → Human Act → Verify structure from Week 3.

### Notes

**1. Which step(s) represent Gather?**

The pre-commit hook and `/pr-ready` gathering the staged changes using `git diff --cached` and `git status` represent the Gather phase.

---

**2. Which step(s) represent Analyze?**

The pre-commit hook analyzes the staged files for secret patterns and oversized files, while `/pr-ready` analyzes the changes for risks, debug code, and drafts a PR title and description.

---

**3. Which step is Human Act, and why must a human — not Claude — run `git commit`, `git push`, and open the PR?**

The Human Act step is when I review the findings, fix the issues, and manually run `git commit`, `git push`, and create the Pull Request. A human must perform these actions to verify the changes and maintain full control over the repository.

---

**4. Which step is Verify?**

The Verify step is re-running the pre-commit hook and `/pr-ready` after fixing the issues, confirming the commit succeeds and the PR is ready.

---

**5. In one or two sentences: why do you need *both* the fixed-rule pre-commit hook and the AI skill? Isn't one enough?**

The pre-commit hook enforces fixed security rules and blocks risky commits automatically, while `/pr-ready` provides contextual analysis and PR guidance. Together they offer stronger protection than either one alone because they handle different types of checks.

---

# Task 8 — LinkedIn Post

## Goal

Publish a LinkedIn post summarizing what you built and what you learned about combining fixed-rule safety checks with AI-assisted review.

### Evidence

#### LinkedIn Post URL

`https://www.linkedin.com/posts/yashrawat2362_dmibypravinmishra-devops-git-activity-7486492380280410112-KFMo?utm_source=share&utm_medium=member_desktop&rcm=ACoAADkdLvUBSPhBiWFg4xDHAf9vp3Ws4aR12mQ`

---

## Key Learnings

Add 3-5 bullet points on what you learned this week.

- Learned the complete Git workflow, including repository setup, staging, committing, branching, merging, and pull requests.
- Gained hands-on experience with GitHub collaboration using forks, upstream repositories, syncing changes, and contributing through Pull Requests.
- Deployed a static website to an AWS EC2 instance with Nginx and connected Git version control with a real deployment workflow.
- Built a Git pre-commit hook and an AI-assisted /pr-ready skill, understanding how fixed-rule automation and AI review complement each other.
- Improved confidence in following professional DevOps practices such as writing meaningful commits, maintaining clean Git history, and applying the Gather → Analyze → Human Act → Verify workflow.

---

# Submission Instructions

- Ensure `hooks/pre-commit` and `.claude/skills/pr-ready/SKILL.md` are committed to your GitHub repository
- Add all required screenshots to your submission
- All written answers must be in your own words
- Do not use a real secret or credential anywhere in your submission — the fake key in Task 1 is intentional and must stay clearly fake
- Open your Pull Request against your own fork, not the shared upstream repository
- Push your final changes to your forked repository
- Include your PR link and LinkedIn post URL

---

## GitHub Repository URL

Paste your forked repository URL here:

`https://github.com/yashrawat2362/devops-micro-internship-interviews`

---

# Completion Checklist

- [x] Branch `feature/ai-pr-ready` created with a staged file containing a fake secret and a debug statement
- [x] `hooks/pre-commit` created and tracked in the repo (not only in `.git/hooks/`)
- [x] `core.hooksPath` configured to point at `hooks/`
- [x] Pre-commit hook shown blocking the risky commit
- [x] `.claude/skills/pr-ready/SKILL.md` created with correct `allowed-tools` (no `Write`) and `disable-model-invocation: true`
- [x] `/pr-ready` run against the risky diff and shown flagging issues
- [x] Risky file fixed; `git commit` succeeds cleanly
- [x] `/pr-ready` re-run showing a clean report and drafted PR title/description
- [x] Pull Request opened using the AI draft as a starting point, with your own fork as the base repository (not upstream), PR link included
- [x] Agentic Loop mapping (Task 7) completed in your own words
- [x] LinkedIn post published and URL submitted
- [x] All required screenshots added
- [x] GitHub repository URL provided

---

## 📌 About DMI & CloudAdvisory

DevOps Micro Internship (DMI) is a project-based DevOps program run by Pravin Mishra (The CloudAdvisory) focused on real-world execution, systems thinking, and career readiness.

It helps learners build strong DevOps foundations with hands-on experience.

---

## 📌 Resources

- 🌐 DMI Official Website: https://dmi.pravinmishra.com?utm_source=github&utm_medium=readme  
- 🎓 University: https://university.pravinmishra.com?utm_source=github&utm_medium=readme  
- 💬 Discord Community: https://discord.pravinmishra.com?utm_source=github&utm_medium=readme  
- 📝 Blog: https://dmi.pravinmishra.com/blog?utm_source=github&utm_medium=readme  
- ▶️ YouTube Playlist: https://www.youtube.com/playlist?list=PLFeSNDtI4Cho  
- 🔗 Pravin Mishra (LinkedIn): https://www.linkedin.com/in/pravin-mishra-aws-trainer/  
- 🏢 CloudAdvisory (LinkedIn): https://www.linkedin.com/company/thecloudadvisory/

---

*This submission is part of DevOps Micro Internship (DMI) Cohort 3 — Agentic AI Track.*
