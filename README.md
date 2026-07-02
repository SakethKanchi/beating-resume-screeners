# Resume Radar

A [Claude Agent Skill](https://docs.claude.com/en/docs/agents-and-tools/agent-skills/overview)
that audits and optimizes a resume against automated **LLM resume-screening
agents**, reproducing the exact 120-point rubric from HackerRank's open-source
[`interviewstreet/hiring-agent`](https://github.com/interviewstreet/hiring-agent).

Modern screeners are not keyword ATS filters. They read the resume PDF, pull the
candidate's **live GitHub via API**, classify every repo as open-source vs
personal, and score four weighted categories plus bonuses and deductions. This
skill teaches Claude to see what the agent sees and to fix the highest-leverage
gaps, using **real signals and honest framing only**.

## What it does

- Reproduces the agent's GitHub classification (which caps the open_source
  score) with a zero-dependency Python script.
- Scores a resume against the full reverse-engineered rubric.
- Returns a gap table sorted by point-swing, plus concrete edits.
- Refuses to fabricate experience, PRs, stars, or metrics.

## Install

**Claude Code / claude.ai / API:** place this folder so `SKILL.md` sits at the
skill root (for Claude Code, drop it in `~/.claude/skills/resume-radar/`).
The skill auto-activates on prompts like "simulate my resume score" or "help me
beat the resume bot".

## Use

```
python scripts/github_enrich.py <github-username-or-url>
```

Set `GITHUB_TOKEN` to raise the GitHub API limit from 60/hr to 5000/hr. Then ask
Claude to `audit`, `simulate`, or `optimize` a resume.

## Example

**You:** "Simulate the hiring-agent score for my resume and tell me where I'm losing points."

**Claude** runs the enrichment script, which reports the same classification the
screener computes:

```json
{
  "username": "example-dev",
  "public_repos": 23,
  "open_source_repos": 0,
  "self_project_repos": 15,
  "all_repos_are_self_project": true,
  "open_source_cap_triggered": true
}
```

Then it scores the resume and returns a gap table sorted by point-swing:

| Category | Score | Ceiling | Highest-leverage fix |
| --- | --- | --- | --- |
| open_source | 8 / 35 | +25 | land one merged PR to a 1000+ star repo (personal repos are capped at ~10) |
| self_projects | 23 / 30 | +7 | add a working live-demo URL to each project |
| production | 20 / 25 | +5 | quantify scale/impact of shipped work |
| technical_skills | 9 / 10 | +1 | already near max |
| bonus | 6 / 20 | +3 | add the portfolio/blog URL that is missing from the resume |

Every suggestion is a real, honest action; the skill never invents experience.

## Contents

| Path | Purpose |
| --- | --- |
| `SKILL.md` | Skill instructions and workflow (loaded when triggered) |
| `references/RUBRIC.md` | Full 120-point rubric (loaded on demand) |
| `scripts/github_enrich.py` | Reproduces the agent's live GitHub classification |
| `scripts/test_github_enrich.py` | Offline unit tests for the script |
| `evaluations.json` | Behavioral evals for the skill |

## Ethics

This skill optimizes what a candidate genuinely has: it surfaces real merged
pull requests, adds working demo links, cleans up a cluttered GitHub, and
ensures real profile URLs are present. It never invents experience or
contributions. The screening agent reads the live repo and a human reads the
PDF, so dishonest signals get caught.

## Credit

Rubric reverse-engineered from the MIT-licensed
[`interviewstreet/hiring-agent`](https://github.com/interviewstreet/hiring-agent)
by HackerRank. This project is not affiliated with or endorsed by HackerRank.

## License

MIT
