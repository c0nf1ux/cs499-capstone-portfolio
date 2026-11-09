# CS-499 Capstone — Code Review & Enhancement Decisions

> A narrated code review of three real coursework artifacts, followed by the trade-off reasoning behind every enhancement planned for them.

**Status:** Milestone One (Code Review) complete | **Program:** SNHU CS-499 Computer Science Capstone | **Author:** Heath Davis

## What this is

Three projects from my coursework, reviewed on camera the way I'd walk a teammate through a legacy codebase: what the code actually does, where it breaks down, and what I'd change and why. This repo is the write-up version of that review — the real findings and the real reasoning, without needing to watch 28 minutes of video to get them.

The three artifacts, and the course outcome category each maps to:

| Artifact | Category | Course |
|---|---|---|
| ABCU Advising Program — course catalog lookup over a `std::map` | Algorithms & Data Structures | CS-300 |
| REST service + standalone TLS server (merged as the enhancement) | Software Design & Engineering | CS-305 |
| Grazioso Salvare animal rescue dashboard — PyMongo wrapper + Dash app | Databases | CS-340 |

## Read this first

- **[PORTFOLIO-CODE-REVIEW.md](PORTFOLIO-CODE-REVIEW.md)** — the findings: what each artifact gets right, where it breaks, and the specific bugs (not hypothetical ones — exact lines, exact failure modes).
- **[PORTFOLIO-DECISIONS.md](PORTFOLIO-DECISIONS.md)** — the fix: every planned enhancement, why it's the right call, what it costs, and which course outcome it serves.

## The actual code

Nothing here is invented to pad out a write-up — every claim traces back to a real file in **[engineering-portfolio](https://github.com/c0nf1ux/engineering-portfolio)**, my full SNHU coursework repo. Direct links to each file reviewed are in both documents above.

## Video

Narrated walkthrough (28 min): **[Watch on YouTube](#)** — *link pending upload.*

## Why this exists as its own repo

`engineering-portfolio` is the coursework itself — three years of assignments, as submitted. This repo is the reflection on it: the critical distance to say "this works, but here's what's actually wrong with it" about my own code, in writing a recruiter or reviewer can skim without watching a video. Same pattern as my other `*-portfolio` repos — documentation only, no runnable code, the real implementation always lives in its real repo.
