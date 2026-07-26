# Assignment 6 — Build an AI-Assisted Linux Health Check (AI-Assisted Linux Incident Triage)

Part of the DevOps Micro Internship (DMI) Cohort 3 with Agentic AI

---

## Purpose

In this assignment, you will build a read-only Bash triage script that checks the health of your Ubuntu server and Nginx application, connect it to Claude Code as a reusable `/linux-triage` skill, simulate a controlled Nginx incident, use the skill to gather and analyze evidence, recover the service manually, and verify recovery. The workflow follows the Agentic Loop: Gather → Analyze → Human Act → Verify.

---

# Task 1 — Confirm the Healthy Baseline and Create the Workspace

## Goal

Confirm that Nginx and the React application are healthy before building the automation.

### Evidence

#### Screenshot 1 — Output of `systemctl is-active nginx`, `ss -ltn | grep ':80'`, and `curl -I http://localhost`

![](./screenshots/ss3.6.1.png)

---

#### Screenshot 2 — Output of `pwd` and `find . -maxdepth 4 -type d | sort` showing the workspace folder structure

![](./screenshots/ss3.6.2.png)

---

### Notes

Answer the following in your own words:

**1. What proves that Nginx is running?**

The systemctl is-active nginx command returned active, which confirms that the Nginx service is running.

---

**2. What proves that the server is listening for HTTP traffic?**

The ss -ltn | grep ':80' command showed that port 80 is listening, which means the server is ready to accept HTTP requests.

---

**3. Why must you capture a healthy baseline before simulating an incident?**

A healthy baseline helps compare the server's normal state with the failed state, making it easier to identify and verify the actual problem.

---

# Task 2 — Create Project Context and Safety Rules in CLAUDE.md

## Goal

Tell Claude exactly what this project does and what it is not allowed to do.

### Evidence

#### Screenshot 3 — CLAUDE.md open in VS Code showing all four sections (Project Overview, Incident Workflow, Safety Rules, Output Rules)

![](./screenshots/ss3.6.3.png)

---

### Notes

Answer the following in your own words:

**1. Why should Claude receive project-specific operational rules?**

Project-specific rules help Claude follow the correct workflow and avoid unsafe actions during incident analysis.

---

**2. Why is the human required to execute the recovery command?**

The human reviews the evidence and decides whether the recovery action is safe before running the command.

---

**3. Which rule prevents Claude from making an unsupported diagnosis?**

The rule "Do not claim a root cause unless the report contains supporting evidence." prevents Claude from making an unsupported diagnosis.

---

# Task 3 — Use Agentic AI to Plan Before Writing the Script

## Goal

Use Claude Code to inspect the environment and produce a read-only plan before creating any Bash code.

### Evidence

#### Screenshot 4 — Claude Code showing the five-check plan and read-only inspection results

![](./screenshots/ss3.6.4.1.png)
![](./screenshots/ss3.6.4.2.png)

---

### Notes

Answer the following in your own words:

**1. Which part of this task represents the Gather phase?**

The read-only inspection commands that Claude ran to check the server represent the Gather phase because they collected system information.

---

**2. Did Claude follow the instruction not to create files? How did you verify this?**

Yes. Claude only ran read-only commands and clearly stated that no files were created or edited, so it followed the instruction.

---

**3. Why is planning before coding useful in DevOps automation?**

Planning helps identify the required checks first, making the script more accurate, organized, and easier to troubleshoot.

---

# Task 4 — Build the Linux Triage Bash Script

## Goal

Create one Bash script that gathers consistent Linux and Nginx health evidence.

### Evidence

#### Screenshot 5 — Top section of `linux-triage.sh` showing variables, thresholds, and the checks array

![](./screenshots/ss3.6.5.png)

---

#### Screenshot 6 — Middle section showing check functions and conditionals

![](./screenshots/ss3.6.6.1.png)
![](./screenshots/ss3.6.6.2.png)

---

#### Screenshot 7 — Bottom section showing the loop, summary function, and exit behavior

![](./screenshots/ss3.6.7.png)

---

#### Screenshot 8 — Output of `bash -n scripts/linux-triage.sh` (no syntax errors) and `ls -l scripts/linux-triage.sh` showing executable permission

![](./screenshots/ss3.6.8.png)

---

### Notes

Answer the following in your own words:

**1. What is stored in the checks array?**

The checks array stores the names of all the health check functions, like service, port, HTTP, disk, and memory checks.

---

**2. How does the `for` loop use that array?**

The for loop runs each function in the array one by one, so every health check is executed automatically.

---

**3. Why are the health checks separated into functions?**

Functions keep the script clean, organized, and make it easier to update or reuse each health check.

---

**4. What is the purpose of `$(...)` in this script?**

$(...) runs a command and stores its output in a variable, such as the date, hostname, or memory value.

---

**5. Why does the script use different exit codes for HEALTHY, WARN, and FAIL?**

Different exit codes make it easy to identify the server's health. 0 means healthy, 1 means there is a warning, and 2 means there is a failure that needs attention.

---

# Task 5 — Run and Understand the Healthy-State Report

## Goal

Run the Bash script against the healthy server and verify that it creates a report.

### Evidence

#### Screenshot 9 — Output of `./scripts/linux-triage.sh` showing your Full Name and all five check results

![](./screenshots/ss3.6.9.png)

---

#### Screenshot 10 — Output showing the captured exit code and final summary

![](./screenshots/ss3.6.10.png)

---

### Notes

Answer the following in your own words:

**1. What is the overall status of your healthy baseline?**

The overall status was HEALTHY, as all the required checks passed successfully.

---

**2. Which exact Linux evidence proves the application is serving traffic?**

The curl -I http://localhost command returned HTTP 200 OK, proving the application is serving traffic.

---

**3. Did your script return exit code 0 or 1? Explain why.**

My script returned exit code 0 because all health checks passed and there were no warnings or failures.

Note: If your report showed Overall Status: WARN, then write:
My script returned exit code 1 because there was a warning (such as low memory or high disk usage), but no failed checks.

---

**4. What is the difference between a warning and a failure in this script?**

A warning means the system is still working but needs attention. A failure means a critical check failed and the service may not be working properly.

---

# Task 6 — Create and Run the /linux-triage Skill

## Goal

Turn the Bash script into a reusable, manually invoked Agentic AI workflow.

### Evidence

#### Screenshot 11 — `SKILL.md` showing the frontmatter, allowed tool restrictions, and safety rules

![](./screenshots/ss3.6.11.png)

---

#### Screenshot 12 — `/linux-triage` output for the healthy server

![](./screenshots/ss3.6.12.png)

---

### Notes

Answer the following in your own words:

**1. Why does this skill have Bash, Read, and Grep, but not Write?**

The skill is designed to collect and analyze information only. It does not need Write permission because it should not modify any files or the server.

---

**2. Why is `disable-model-invocation: true` useful for this skill?**

It makes the skill follow the predefined steps and keeps the workflow consistent and safe.

---

**3. What part is performed by Bash, and what part is performed by Claude?**

Bash collects the system health data, while Claude reads the report, explains the findings, and suggests a safe recovery command.

---

**4. Why is this better than asking Claude "Is my server healthy?" without giving it evidence?**

Because Claude makes its analysis based on real system data instead of guessing, making the result more accurate and reliable.

---

# Task 7 — Simulate an Nginx Incident and Let the Skill Diagnose It

## Goal

Create a controlled service failure, gather evidence through Bash, and let Claude analyze the evidence without taking recovery action.

### Evidence

#### Screenshot 13 — Output showing Nginx is inactive and the HTTP request fails

![](./screenshots/ss3.6.13.png)

---

#### Screenshot 14 — `/linux-triage` output showing failed evidence, most likely cause, and a suggested recovery command

![](./screenshots/ss3.6.14.1.png)
![](./screenshots/ss3.6.14.2.png)

---

#### Screenshot 15 — `incident-failure-report.txt` showing the failed checks and your Full Name

![](./screenshots/ss3.6.15.png)

---

### Notes

Answer the following in your own words:

**1. Which three checks failed?**

The Nginx service check, port 80 check, and local HTTP check failed.

---

**2. What evidence supports the conclusion that Nginx is unavailable?**

The report showed that the Nginx service was inactive, port 80 was not listening, and the HTTP request did not return 200 OK.

---

**3. Did Claude execute the recovery command? Why is that important?**

No. Claude only recommended the recovery command. This is important because the final decision and action should always be approved by a human.

---

**4. Which phase of the Agentic Loop is represented by the Bash report?**

The Bash report represents the Gather phase because it collects system health information.

---

**5. Which phase is represented by Claude's explanation?**

Claude's explanation represents the Analyze phase because it reviews the collected evidence and explains the likely issue.

---

# Task 8 — Recover Manually, Verify Again, and Write the Incident Summary

## Goal

Recover the service as the human operator and prove that the system is healthy again.

### Evidence

#### Screenshot 16 — Output showing Nginx is active and `curl -I http://localhost` returns 200 OK

![](./screenshots/ss3.6.16.png)

---

#### Screenshot 17 — Second `/linux-triage` output showing successful recovery with no FAIL results

![](./screenshots/ss3.6.17.png)

---

#### Screenshot 18 — Output of `ls -lah reports` showing both `incident-failure-report.txt` and `recovery-report.txt`

![](./screenshots/ss3.6.18.png)

---

#### Screenshot 19 — `incident-summary.md` showing all required sections and your Full Name

![](./screenshots/ss3.6.19.png)

---

### Notes

Answer the following in your own words:

**1. What action did you execute manually?**

I manually started the Nginx service using sudo systemctl start nginx.

---

**2. What evidence proves that the service recovered?**

systemctl is-active nginx returned active, curl -I http://localhost returned 200 OK, and the second triage report showed no failed checks.

---

**3. Why is the second triage run necessary?**

It confirms that the recovery worked and the server is healthy again.

---

**4. What could go wrong if an AI agent automatically restarted every failed service?**

It could restart the wrong service, hide the real issue, or cause unnecessary downtime.

---

**5. In one sentence, explain the difference between using AI as a chatbot and using AI in this agentic workflow.**

A chatbot gives answers, while an agentic AI workflow analyzes real system evidence and helps the user make safe decisions.

---

# Incident Summary

Fill in all seven sections below in your own words.

**Full Name:** Yash Rawat

**Date:** 17/07/2026

---

**1. Reported Symptom**

The website was not opening because the Nginx service had stopped.

---

**2. Evidence Collected**

The Bash report showed that the Nginx service was inactive, port 80 was not listening, and the local HTTP check did not return 200 OK.

---

**3. Most Likely Cause**

The most likely cause was that the Nginx service was stopped, so the web server was unable to serve the application.

---

**4. Human-Approved Recovery Action**

After reviewing the report, I manually ran sudo systemctl start nginx to restore the service.

---

**5. Verification**

I confirmed the recovery by checking that Nginx was active, curl -I http://localhost returned 200 OK, and the second triage report showed no failed checks.

---

**6. Safety Decision**

The AI skill was only allowed to collect and analyze evidence. The recovery action was performed manually to keep the system safe and under human control.

---

**7. Agentic Loop Mapping**

The incident followed the workflow: Gather (Bash collected evidence) → Analyze (Claude explained the issue) → Human Act (I started Nginx manually) → Verify (the second triage confirmed the service was healthy).

---

# LinkedIn Post (Required)

## Evidence

#### LinkedIn Post URL

Paste your LinkedIn post URL here:

`https://www.linkedin.com/posts/yashrawat2362_dmibypravinmishra-linux-bash-activity-7483928519337590784-O1V7?utm_source=share&utm_medium=member_desktop&rcm=ACoAADkdLvUBSPhBiWFg4xDHAf9vp3Ws4aR12mQ`

---

#### Screenshot — Published LinkedIn post

![linkedin-post](./screenshots/ss3.6.20.png)

---

# GitHub Repository URL

Paste the URL of your GitHub folder or repository containing the assignment files here:

`https://github.com/yashrawat2362/devops-micro-internship-pravinmishra/tree/main/week-03-linux-and-bash-for-devops`

---

# Submission Instructions

- Add all required screenshots in your submission
- Full Name must be visible in required screenshots and the Bash report
- All written answers must be in your own words
- Do not expose sensitive information (keys, passwords, AWS account IDs, tokens)
- GitHub URL must be included in this document

---

# Completion Checklist

- [x] Task 1: Healthy baseline confirmed, workspace created (Screenshots 1–2, Notes answered)
- [x] Task 2: CLAUDE.md created with all four sections (Screenshot 3, Notes answered)
- [x] Task 3: Five-check plan produced by Claude using read-only tools (Screenshot 4, Notes answered)
- [x] Task 4: `linux-triage.sh` created, syntax validated, executable permission set (Screenshots 5–8, Notes answered)
- [x] Task 5: Healthy-state report generated with no FAIL result (Screenshots 9–10, Notes answered)
- [x] Task 6: `/linux-triage` skill created and run successfully on healthy server (Screenshots 11–12, Notes answered)
- [x] Task 7: Nginx incident simulated, failed evidence captured, Claude did not execute recovery (Screenshots 13–15, Notes answered)
- [x] Task 8: Nginx recovered manually, recovery verified, reports saved, incident summary complete (Screenshots 16–19, Notes answered)
- [x] Incident summary contains all seven required sections
- [x] LinkedIn post published and URL submitted
- [x] Full Name visible in all required screenshots and the Bash report
- [x] Skill does not have Write permission
- [x] Skill did not execute any recovery commands
- [x] No sensitive data exposed

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
