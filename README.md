# claude-skills

Justin's **global skills** — a personal plugin marketplace for Claude Code. These
are reusable, use-case-independent skills meant to be available in *any* project
on *any* device. This repo is the **single source of truth** for them.

## What's here

```
claude-skills/
├── .claude-plugin/marketplace.json     ← the marketplace catalog ("jd-skills")
└── plugins/global-skills/              ← one plugin bundling all global skills
    ├── .claude-plugin/plugin.json
    └── skills/
        ├── grill-me/                   scoping-interview skill
        ├── linkedin-post/              LinkedIn post development skill
        └── news-brief/                 personalized news brief
            └── references/             config template, workflow, repo setup
```

## The skills

**grill-me** and **linkedin-post** are pure conversation skills: no setup, no state
on disk, no external services. Install and use.

**news-brief** works the same way by default — ask for a brief anywhere and you get
one. It additionally *scales up* when it finds a configured repo around it: an
archive of past issues, a watchlist that persists, and a reader-feedback loop through
GitHub issues. `historiocity/News-Agent` is that repo; `skills/news-brief/references/`
holds everything needed to stand up another one.

## Install on any device (Claude Code)

```sh
/plugin marketplace add historiocity/claude-skills
/plugin install global-skills@jd-skills
```

After that, the skills are available globally. Update later with:

```sh
/plugin marketplace update jd-skills
```

## Relationship to projects

Project-specific skills do **not** live here — they live in their own project
repos (e.g. `trip-planner` holds `trip-shortlist`). When a project needs a global
skill, it gets a **renamed fork** copied into that project repo so the project is
self-contained (e.g. `grill-me-trip-planner`). Those forks are free to drift; this
repo stays the canonical seed. Genuinely general improvements made in a fork get
promoted back here deliberately.

`news-brief` is the exception, and deliberately so. Rather than forking a copy into
`News-Agent`, that repo's workflow checks this repo out at run time and stages the
skill onto the runner. There is only ever one copy, so it cannot drift. This works
because this repo is public — if it goes private, that checkout step needs a
read-scoped PAT as a secret.
