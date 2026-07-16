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
        └── linkedin-post/              LinkedIn post development skill
```

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
