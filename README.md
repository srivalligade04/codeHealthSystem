Code Health Radar
A dashboard that surfaces what standard engineering metrics miss — dependency exposure, knowledge concentration, pull request risk, CI pipeline health, and the structural gaps between a passing build and a safe merge.
Built as a working prototype from the article We Built a Code Health Radar. Here Is What We Learned About the Gaps Every Team Has.
---
What it does
Most engineering teams measure build status, deployment frequency, and sprint velocity. None of these tell you whether the code is actually healthy.
Code Health Radar answers four questions standard metrics do not:
Which dependencies are actively exposing you to known vulnerabilities — not in a quarterly scan but right now across the repositories you are actively shipping from
Who actually knows this code — not who is listed in a CODEOWNERS file but who has committed in the last twelve months and what happens if that person leaves
What is slipping through the review process — changes that touch single-owner files with no domain expert assigned, high-risk PRs where the first review takes forty-one hours, CVE fixes with no regression test added
What CI shows versus what it misses — a passing build is not the same as a healthy pipeline
---
Six views
View	What it answers
Overview	Aggregate health across all repositories with a risk map plotting bus factor against vulnerability count
Repositories	Every scanned repository with health score, vulnerable dependencies, bus factor, and staleness flag
Pull Requests	Each PR with a risk score, CI check status, Code Sheriff findings, and uncaught issues
Jenkins CI	Pipeline health per repository — stages, test counts, coverage trend, and last run time
Ownership & Reviews	Knowledge concentration by commit percentage and review load distribution across the team
Code Sheriff	Automated review pass that flags structural gaps CI does not catch — before merge
---
Code Sheriff
The most useful thing in the tool is something we almost did not build.
CI passes on pull requests that have real problems. Not security problems that require human judgment. Obvious structural problems any automated check could catch.
The Code Sheriff runs after CI and before merge and looks for:
Tests passing but coverage dropped significantly on a high-risk change
A CVE fix landed with no regression test for the affected code path
A configuration change that affects downstream services with no notification sent
A pull request open eleven days with a failing build and no reviewer assigned
Each PR gets a status — Approved, Flagged, or Blocked — and a specific list of uncaught issues. The findings appear in the bot comment alongside CI results. They do not block merging. They inform the human making the merge decision.
Warn-only mode is a deliberate design decision. A gate creates friction on every merge including the ones that should move fast. It creates pressure to game the threshold rather than address the finding. The goal is not to prevent merges. It is to make sure the person who clicks merge has seen the specific things that could go wrong.
---
What the pilot revealed
Running the tool across eight repositories revealed problems the team already knew about in a vague sense but had never needed to act on. The dashboard made them specific and undeniable.
legacy-reports — health score 29. Last commit 410 days ago. CI failing for 14 days with nobody watching. Two critical CVEs open since 2015 and 2017. One person holding 92 percent of commits. That person had left the organization.
payments-service — health score 54. CI passing. Open PR for a partial refund endpoint with three failing unit tests, no integration test coverage on the downstream payment flow, and a dependency addition carrying a known high CVE. The reviewer had approved it.
data-pipeline — health score 47. One person holding 78 percent of commits. Largest change of the quarter at 610 lines with first review taking 41 hours and coverage at 61 percent.
auth-gateway — health score 81. Recently merged PR to reduce token lifetime. Build passed. Tests passed. Code Sheriff finding: downstream services had not been notified. The code was correct. The operational communication was missing.
The tool did not discover these problems. The problems existed before we built anything. The dashboard made them visible and undeniable in a way that a conversation could not.
---
Getting started
No server required. No dependencies. No login.
```bash
git clone https://github.com/srivalligade04/codeHealthSystem.git
cd codeHealthSystem
open code-health-radar-interactive.html
```
Or open `code-health-radar-interactive.html` directly in any browser.
The prototype runs entirely in the browser with sample data from the pilot. Click any repository in the risk map or any pull request in the table to see the full detail view.
---
Live demo
👉 Open the interactive prototype
---
Replacing the sample data
All data lives in the `<script>` block at the top of the HTML file. Replace the `repos`, `prs`, `mrs`, `jenkinsJobs`, and `reviewLoad` arrays with output from your own tooling — your GitHub or GitLab API, your Jenkins API, your dependency scanner, and your commit history.
The tool is intentionally a single HTML file with no build step and no external dependencies so it is straightforward to embed the data generation in a script that regenerates the file on a schedule.
---
Technical decisions
Health score as a single number. A single number gets the conversation started. The nuance lives in the detail view.
Risk score from multiple signals. PR risk incorporates the ownership profile of the files being changed, the dependency changes in the PR, and whether domain experts are assigned. A change to a file with a single owner for fourteen months scores higher than the same change to a file with four active contributors.
Ownership by commit not by CODEOWNERS. CODEOWNERS reflects organizational intent. Commit history reflects who actually knows the code. These are frequently different and the difference matters when you are trying to understand what happens if a specific person leaves.
Warn-only Code Sheriff. Teams that encounter hard blocks on every high-risk PR learn to work around the block rather than address the finding. Warn-only mode puts the information in front of the engineer and leaves the decision with the human.
---
What to build next
Real-time integration — PR hooks that run the Code Sheriff at open time and update findings as the PR evolves
Trend analysis — health score over time, ownership concentration trend, average time to first review trend. The trend is often more important than the current value
IDE integration — a developer who sees Code Sheriff findings while writing the code addresses them differently than one who sees them in a bot comment after the PR is open
---
Article
Full writeup of what we built, what it revealed, and the design decisions that mattered:
We Built a Code Health Radar. Here Is What We Learned About the Gaps Every Team Has.
---
License
MIT
