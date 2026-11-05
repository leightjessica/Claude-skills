# Econ research skills for Claude Code

Three [Claude Code](https://claude.com/claude-code) skills developed in the course of empirical
development-economics research (randomized evaluations, survey data, Stata pipelines). They are
shared as-is in the hope that they are useful to other researchers.

| Skill | What it does |
|---|---|
| `prepping-replication-packages` | Seven-stage audit that gets a Stata replication package ready for external researchers or a journal data editor: master-script verification, path harmonization, do-file audit, output checks, README completeness, PII audit and removal, and final cleanup. |
| `code-paper-alignment` | Re-runs the analysis pipeline and reconciles the manuscript against it: every number and table, every methodology statement (text, footnotes, table notes, appendices), and hard-coded numbers that should be code-generated macros. Fixes minor discrepancies and flags major ones for a decision. |
| `policy-presentations` | Conventions for PowerPoint decks aimed at donors, governments, and implementing partners: one figure per slide, interpretation in speaker notes, section summaries, and plain-language framing of methods. |

## Install

As a plugin (recommended, so you receive updates):

```
/plugin marketplace add <github-user>/econ-research-skills
/plugin install econ-research-skills@econ-research-skills
```

Or copy any folder under `skills/` into `~/.claude/skills/`.

## Notes

- The replication and alignment skills assume Stata and LaTeX; adjust the example commands (for
  example, the Stata executable path) for your own machine.
- Both audit skills are designed to report before making destructive changes, and never to modify
  data. Review their proposed edits as you would a coauthor's.
- `policy-presentations` works alongside a PowerPoint-generation skill (such as Anthropic's `pptx`
  skill) and will use a branded `.pptx` template if you provide one.

## License

MIT. See [LICENSE](LICENSE).
