# Skills

Reusable AI skills maintained by Sarang Kharche. Each folder contains a `SKILL.md` file with instructions, plus any supporting documents, templates and examples.

## Available skills

| Skill | Purpose | Skill name |
| --- | --- | --- |
| [Clear Technical English](clear-technical-english/README.md) | Clearer technical explanations and instructions | `clear-technical-english` |
| [Pastel Technical Diagrams](pastel-technical-diagrams/README.md) | Editable Draw.io diagrams using a consistent pastel palette, typography and layout | `pastel-technical-diagrams` |

## Quick install

You need Node.js 22.20 or later and Git. Run this command to choose skills and an assistant interactively:

```sh
npx skills add sarangkharche/skills
```

The `skills` package is the shared installer from npm. It fetches the skill folders from this GitHub repository. You do not need to clone this repository yourself or publish a separate npm package.

The default installation applies to the current project. Add `-g` to make a skill available across projects for your user account.

### Codex

Install Pastel Technical Diagrams across projects:

```sh
npx skills add sarangkharche/skills --skill pastel-technical-diagrams -a codex -g
```

Install Clear Technical English across projects:

```sh
npx skills add sarangkharche/skills --skill clear-technical-english -a codex -g
```

Install every skill in this repository for Codex:

```sh
npx skills add sarangkharche/skills --skill '*' -a codex -g
```

### Claude Code

Install every skill in this repository for Claude Code:

```sh
npx skills add sarangkharche/skills --skill '*' -a claude-code -g
```

To install only the diagram skill, replace `'*'` with `pastel-technical-diagrams`.

### Install for a single project

Run this from your project directory. Omit `-g`:

```sh
npx skills add sarangkharche/skills --skill pastel-technical-diagrams -a codex
```

### Command options

| Option | What it does |
| --- | --- |
| `--skill <name>` | Select one skill. Use `'*'` to select every skill in the repository |
| `-a codex` | Install for Codex |
| `-a claude-code` | Install for Claude Code |
| `-g` | Install across projects for the current user |
| `--list` | Show available skills without installing |
| `--copy` | Copy files instead of creating links |
| `-y` | Skip installer confirmation prompts |

Start a new chat after installation. Read each skill's README for any additional requirements. Installing the diagram skill does not install or connect Draw.io; the skill documents a local file fallback when its connector is unavailable.

## Browse and manage skills

List the skills available in this repository without installing them:

```sh
npx skills add sarangkharche/skills --list
```

List your installed global skills for Codex:

```sh
npx skills list -g -a codex
```

Update the diagram skill installed through the CLI:

```sh
npx skills update pastel-technical-diagrams -g
```

Update all global skills managed by the CLI:

```sh
npx skills update -g
```

Remove the global diagram skill from Codex:

```sh
npx skills remove pastel-technical-diagrams -a codex -g
```

## Use a skill

Ask naturally or name the skill explicitly. Automatic selection depends on the assistant, its configuration and whether the request matches the skill.

In Codex:

```text
$pastel-technical-diagrams Create a diagram showing how an agent uses tools.
```

```text
$clear-technical-english Rewrite this technical explanation in plain English.
```

In Claude Code:

```text
/pastel-technical-diagrams Create a diagram showing how an agent uses tools.
```

The diagram skill is configured for automatic selection on Sarang's diagram requests. An explicit request for a different style or format takes precedence.

## Manual installation

If you prefer copying files yourself, clone the repository:

```sh
git clone https://github.com/sarangkharche/skills.git
cd skills
```

Copy the complete skill folder, including its documents and assets. For Codex:

```sh
mkdir -p ~/.codex/skills
cp -R ./pastel-technical-diagrams ~/.codex/skills/
```

For Claude Code:

```sh
mkdir -p ~/.claude/skills
cp -R ./pastel-technical-diagrams ~/.claude/skills/
```

For a shared Claude Code project installation, use the project's `.claude/skills/` directory. To update a manually copied skill, pull the repository and copy its folder again. Keep a backup of local customisations before replacing files. Use the CLI update commands for installations managed by the CLI.

## Add a skill to this repository

Create a folder named after the skill. Include:

1. `SKILL.md` with its name, description and instructions
2. `README.md` with its purpose, installation, usage and limitations
3. Supporting references, assets or examples that the skill needs

Add its README link to the table above. Keep each skill self contained. The installer discovers valid `SKILL.md` files in the repository.

## Installer documentation

See the [skills CLI documentation](https://github.com/vercel-labs/skills#install-a-skill) for setup requirements and the [command reference](https://github.com/vercel-labs/skills/blob/main/docs/commands.md) for all options.
