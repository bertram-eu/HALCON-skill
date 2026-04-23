# HALCON Vision Script Authoring Skill

> **Proof of Concept** — This skill is under active development and may change significantly.

An [Agent Skill](https://agentskills.io) for generating, refactoring, running, and reviewing [MVTec](https://www.mvtec.com) [HALCON](https://www.mvtec.com/products/halcon) vision scripts. It produces [HDevelop](https://www.mvtec.com/products/halcon/work-with-halcon/hdevelop) (`.hdev`) and [HDevelopEVO](https://www.mvtec.com/products/halcon/work-with-halcon/hdevelopevo) (`.hscript`) programs with accurate operator usage, parameter defaults and documentation.

This skill follows the open [Agent Skills](https://agentskills.io) standard and works with any compatible agent, including [Claude Code](https://claude.ai/code) and [OpenAI Codex](https://developers.openai.com/codex).

## What It Does

- **Generates complete, executable scripts** in both HDevelop XML (`.hdev`) and HDevelopEVO plain-text (`.hscript`) formats
- **Selects appropriate HALCON operators** with valid parameter ranges and sensible defaults
- **Builds structured vision pipelines** following a proven workflow: input → ROI → preprocessing → detection → decision
- **Creates reusable procedures** with full multi-language documentation (English, German, + user locale)
- **Runs and tests scripts headless** via `hrun` or `hscriptengine` for CI/testing workflows
- **Leverages HALCON's documentation** — 2,600+ operator reference pages, solution guides, and 1,700+ example scripts

## Prerequisites

- An AI coding agent that supports [Agent Skills](https://agentskills.io)
- [HALCON](https://www.mvtec.com/products/halcon) (or compatible version) installed with `HALCONROOT` and `HALCONEXAMPLES` environment variables set
- `hrun` and/or `hscriptengine` available on your PATH

## Installation

### Option A: Using `npx skills` (multi-agent)

Installs the skill for all supported agents at once (Claude Code, Cursor, Codex, Copilot, Gemini CLI, and more):

```bash
npx skills add https://github.com/bertram-eu/halcon-skill.git
```

Or using GitHub shorthand:

```bash
npx skills add bertram-eu/halcon-skill
```

To update the skill to the latest version:

```bash
npx skills update bertram-eu/halcon-skill
```

### Option B: Manual (git clone)

Clone the repository and copy the skill into your agent's skills directory. The exact path depends on your agent:

```bash
git clone https://github.com/bertram-eu/halcon-skill.git
```

## Usage

Once installed, the skill activates automatically when you ask for HALCON-related tasks. Examples:

```
> Write an HDevelop script that reads an image, applies a threshold, and measures the area of all detected regions.

> Create an .hscript procedure that performs template matching with subpixel accuracy.

> Debug my barcode reading script — it fails on low-contrast images.

> Convert this HDevelop script to HDevelopEVO format.
```

The skill defaults to `.hdev` (HDevelop XML) format. Mention "HDevelopEVO", "HScript", or ".hscript" to get plain-text output instead.

## What's Included

```
halcon-skill/
├── SKILL.md                             # Skill definition and authoring rules
└── references/
    ├── hdev_format_reference.md         # .hdev XML schema and documentation format
    ├── hscript_format_reference.md      # .hscript plain-text syntax
    ├── hrun_reference.md                # hrun CLI usage for headless .hdev execution
    └── hscriptengine_reference.md       # hscriptengine CLI usage for headless .hscript execution
```

## Limitations

This is a proof of concept. Known limitations:

- Requires a local HALCON installation and license for execution and documentation access
- The skill relies on HALCON's HTML operator docs at `HALCONROOT/doc/html/` — if your installation is incomplete, operator lookups may fail
- Generated scripts should always be reviewed and tested before production use

## License

See [LICENSE](LICENSE) for details.
