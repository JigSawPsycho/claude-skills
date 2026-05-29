# byteblast-skills

A personal [Claude Code](https://code.claude.com) **plugin marketplace** — a git repo of skills you can `git pull`/`push` and install/update across machines with the built-in `/plugin` commands.

## Install (per machine, once)

```text
/plugin marketplace add <your-org-or-user>/<this-repo>
/plugin install git-tools@byteblast-skills
```

Replace `<your-org-or-user>/<this-repo>` with the GitHub slug you push this to (e.g. `mike/byteblast-skills`). You can also add a local clone by path:

```text
/plugin marketplace add /absolute/path/to/byteblast-skills
```

## Update

```text
/plugin marketplace update byteblast-skills
```

This pulls the latest marketplace + plugin contents. Reload if prompted.

## Using the skills

Plugin skills are **namespaced by plugin**, so invoke them as `/<plugin>:<skill>`:

```text
/git-tools:git-catchup
```

## What's inside

| Plugin      | Skill         | What it does |
|-------------|---------------|--------------|
| `git-tools` | `git-catchup` | A functional briefing of what changed on the current branch since *your* last commit — important code changes, breaking changes, and how new/changed functionality works. |

## Repo layout

```
byteblast-skills/
├── .claude-plugin/
│   └── marketplace.json          # marketplace catalog (lists the plugins)
├── plugins/
│   └── git-tools/
│       ├── .claude-plugin/
│       │   └── plugin.json        # plugin manifest
│       └── skills/
│           └── git-catchup/
│               └── SKILL.md       # the skill itself
└── README.md
```

## Adding a new skill

1. Drop it under an existing plugin: `plugins/<plugin>/skills/<skill-name>/SKILL.md`.
   (Each skill is one folder with a `SKILL.md`; folder name = skill name. Skills are
   discovered exactly one level under `skills/` — no category subfolders.)
2. Or add a new plugin: create `plugins/<new-plugin>/.claude-plugin/plugin.json` plus its
   `skills/`, then add an entry to `.claude-plugin/marketplace.json`.
3. Bump the `version` in the affected `plugin.json` (and `marketplace.json`) so machines
   pick up the update, commit, and push.
