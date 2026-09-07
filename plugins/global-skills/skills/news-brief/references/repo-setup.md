# Setting up the brief repo

The skill runs anywhere with no setup. This file covers the optional full pipeline:
a repo that archives every issue, serves them over GitHub Pages, and feeds reader
feedback back into the next morning's run.

The reference repo is `historiocity/News-Agent`.

## Layout

```
News-Agent/
├── .github/workflows/daily-brief.yml   ← 5am local, with backstops
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

## Scheduling, and why it is not one cron line

GitHub's scheduler is not a guarantee. Scheduled workflows are delayed under load
and are sometimes dropped entirely — never queued, never run, no failure to see.
The top of the hour is the worst slot for this. A single `0 8 * * *` line will
silently skip days.

`daily-brief.yml` therefore schedules five crons and gates them:

| Cron (UTC) | Role |
|---|---|
| `0 9 * * *` | 5am EDT — fires mid-Mar → early Nov |
| `0 10 * * *` | 5am EST — fires early Nov → mid-Mar |
| `23 13 * * *` | backstop, ~9:23am EDT |
| `47 17 * * *` | backstop, ~1:47pm EDT |
| `11 22 * * *` | backstop, ~6:11pm EDT — last chance for the local day |

Cron is always UTC and never shifts for daylight saving, so 5am local needs two
lines. Both fire year-round; the gate step reads `github.event.schedule` together
with the current UTC offset and stands down whichever one is not 5am today.

The backstops exist because the 5am run may never happen. Each one checks whether
`issues/YYYY-MM-DD.html` exists for the current local day and exits in seconds if
it does, so on a normal day they cost nothing. On a day GitHub dropped the 5am
run, the first backstop produces the brief instead.

That existence check is also what makes the whole thing idempotent: no trigger can
produce a second issue for a day that already has one, so overlapping or
badly-delayed runs cannot double-post. `concurrency` queues rather than cancels, so
a slow 5am run is never killed by a backstop starting behind it.

Changing the reader's timezone means changing `BRIEF_TZ` **and** the two 5am cron
lines. The crons cannot read the env var.

## Notification

The workflow verifies the issue actually reached the default branch — a run that
wrote a file but failed to push is red, not silently green — and then notifies.

**Default, no setup.** It opens a GitHub issue titled `📰 Brief ready — <date>`,
assigned to the repo owner, containing a link to the published page and the
issue's story titles, then closes it immediately. GitHub emails the assignee and
pushes to the GitHub mobile app; closing keeps the tracker clean without
suppressing the notification. Labels `brief-ready` and `brief-failed` are created
on first use.

**Optional phone push.** Set a repository variable `NTFY_TOPIC` (Settings →
Secrets and variables → Actions → Variables) to a topic name of your choosing, and
subscribe to that topic in the free ntfy app. The push carries the issue date
and deep-links to the issue. Unset, the step skips silently.

**Failures notify too.** A failed run opens its own issue naming the run log, so a
broken pipeline never looks like a quiet news day. The usual causes are an expired
`CLAUDE_CODE_OAUTH_TOKEN` or a push rejection.

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
