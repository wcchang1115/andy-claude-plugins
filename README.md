# andy-claude-plugins

A [Claude Code](https://claude.com/claude-code) plugin marketplace.

## Plugins

### `dev-writing-guides`

Three guides for the text a developer writes around code. Each one is a
checklist that Claude follows when it drafts or reviews that text.

| Skill | Covers |
| --- | --- |
| `wcc-commit-message-guide` | commit subject and body |
| `wcc-pr-description-guide` | PR title and description |
| `wcc-code-comments-guide` | code comments and docstrings in the current diff |

The guides share one idea: say what the change decides, and cut anything the
diff already shows. They also move issue IDs, session IDs, log references and
customer names out of code and commits, into the tracker issue.

#### Optional companion skill

The guides look for a skill named `simple-english` and use it to tighten the
text they write. The guides work without it. To get that step, install a skill
with that name separately.

#### Repo-specific rules

The guides read your repository's `CLAUDE.md` or `AGENTS.md` for two things:

- a fixture contract, which says a test fixture must cite the real trace or log
  it came from
- tracker formatting limits, for trackers that reject some markdown

If your repo documents neither, the guides fall back to their general rules.

## Install

```
/plugin marketplace add wcchang1115/andy-claude-plugins
/plugin install dev-writing-guides@andy-claude-plugins
```

## Add a new plugin

1. Create `plugins/<name>/.claude-plugin/plugin.json`.
2. Put its `skills/`, `commands/`, or `agents/` next to that folder.
3. Add an entry to the `plugins` list in `.claude-plugin/marketplace.json`.

## License

MIT. See [LICENSE](LICENSE).
