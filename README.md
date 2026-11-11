# Skills

A collection of reusable AI skills, maintained by Sarang Kharche.

Each skill lives in its own folder. Its README explains what it does, how to use it and any requirements. The SKILL.md file contains the instructions that the assistant reads.

## Available skills

| Skill | Purpose |
| --- | --- |
| [Clear Technical English](clear-technical-english/README.md) | Clearer technical explanations and instructions |

## Install a skill

Clone this repository:

```sh
git clone https://github.com/sarangkharche/skills.git
cd skills
```

Choose a skill and copy its complete folder into the directory for your assistant. Replace `skill-name` below with the folder name. Run these commands from the cloned repository.

### Claude Code

```sh
mkdir -p ~/.claude/skills
cp -R ./skill-name ~/.claude/skills/
```

For a shared project installation, use the project's `.claude/skills/` directory instead.

### Codex

```sh
mkdir -p ~/.codex/skills
cp -R ./skill-name ~/.codex/skills/
```

Start a new chat after installation. Check the skill's README for any additional setup.

## Use a skill

In Claude Code:

```text
/skill-name Your request
```

In Codex:

```text
$skill-name Your request
```

Assistants may also select relevant skills automatically. This depends on the skill metadata, the client and your instructions.

## Update an installed skill

Pull the latest repository changes and copy the skill folder again. Save any local customisations first.

## Add a skill

Create a folder named after the skill. Include:

1. `SKILL.md` with its name, description and instructions
2. `README.md` with its purpose, installation, usage examples and limitations
3. Any supporting references, assets or scripts that the skill needs

Keep instructions and examples specific to each skill in its own folder. Add a link to its README in the table above.
