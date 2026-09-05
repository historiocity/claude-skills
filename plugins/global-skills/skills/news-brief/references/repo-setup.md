# Setting up the brief repo

The skill runs anywhere with no setup. This file covers the optional full pipeline:
a repo that archives every issue, serves them over GitHub Pages, and feeds reader
feedback back into the next morning's run.

The reference repo is `historiocity/News-Agent`.

## Layout

```
News-Agent/
├── .github/workflows/daily-brief.yml   ← cron 08:00 UTC (4am EDT / 3am EST)
├── config.md                           ← beats, anti-topics, sources, watchlist
├── issues/YYYY-MM-DD.html              ← one issue per day, standalone
├── index.html                          ← archive, regenerated each run
└── .nojekyll                           ← serve files as-is
```

There is deliberately no copy of the skill in this repo. The workflow checks out
`historiocity/claude-skills` at run time and stages the skill onto the runner, so the
marketplace copy stays the single source of truth. That works because `claude-skills`
is public; if it is ever made private again, the checkout step needs a fine-grained
PAT with read access, supplied as a secret.

## One-time setup

1. **Seed the repo.** Copy `config.template.md` to `config.md` at the root, and
   `daily-brief.yml` to `.github/workflows/daily-brief.yml`. Create an empty
   `issues/` directory and a `.nojekyll` file at the root.

2. **Create the feedback label.** The skill filters on it and the feedback button
   applies it:

   ```
   gh label create brief-feedback --description "Reader response to a brief issue" --color 0E8A16
   ```

   The skill treats a missing label as "no feedback" rather than failing, so this is
   about making the loop work, not about avoiding a crash.

3. **Add the auth secret.** Run `claude setup-token` locally, then add the result as
   the repo secret `CLAUDE_CODE_OAUTH_TOKEN` under Settings → Secrets and variables →
   Actions. Runs bill to the Claude subscription rather than the API.

4. **Install the Claude GitHub App** on the repo: https://github.com/apps/claude

5. **Enable Pages.** Settings → Pages → deploy from `main`, root. Note that Pages on
   a private repo requires a paid plan; the public path is free.

6. **First run.** Actions tab → Daily Brief → Run workflow. This produces issue one
   without waiting for cron, and surfaces any auth problem while you're awake to see
   it.

## What publishing means

A public brief repo is world-readable, including `config.md` and every feedback
issue. `config.md` describes the reader — their beats, employer focus, and locality —
and feedback issues record their reactions to individual stories. That is the
tradeoff for free Pages hosting. Anything the reader would not want indexed belongs
in a private override, not in `config.md`.

## Feedback loop

Each issue renders radio controls per story. Answering them and pressing the button
at the bottom opens a prefilled GitHub issue labeled `brief-feedback`. The next run
reads open feedback issues, folds them into `config.md`, closes them with a one-line
note saying what changed, and commits the updated config alongside the new issue.

That is the whole mechanism — no server, no database, no external service. The
archive is the repo and the feedback queue is the issue tracker.
