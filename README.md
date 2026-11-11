# Skills

Reusable AI skills for technical work, maintained by Sarang Kharche.

## Available skills

| Skill | Purpose |
| --- | --- |
| [Clear Technical English](clear-technical-english/SKILL.md) | Clearer technical explanations, consistent terminology and readable instructions |

## Install

Clone this repository:

```sh
git clone https://github.com/sarangkharche/skills.git
```

### Claude Code

Copy the skill folder into your personal skills directory:

```sh
mkdir -p ~/.claude/skills
cp -R skills/clear-technical-english ~/.claude/skills/
```

Start a new chat and use:

```text
/clear-technical-english Explain this service
```

For a shared project installation, copy the folder into your project's `.claude/skills/` directory instead.

### Codex

Copy the same folder into your personal skills directory:

```sh
mkdir -p ~/.codex/skills
cp -R skills/clear-technical-english ~/.codex/skills/
```

Start a new chat and use:

```text
$clear-technical-english Explain this service
```

Automatic selection is enabled in the skill metadata. Whether it is selected depends on the request and your assistant's instructions. Installing it alone does not force it to be used on every technical answer.

## Validation

Clear Technical English was tested on a sample rewrite in Claude Code 2.1.311 with Haiku 5.5. The comparison used two fresh sessions, with all other skills disabled. The skill improved paragraph structure and retained the supplied technical details and health check recommendation in that example.

Automatic selection was also observed in one fresh local Codex chat. These small checks do not establish consistent improvement across all tasks or models.

## Sources and scope

[Source notes](clear-technical-english/SOURCES.md) document the article, official website review and limits of the standard review.

This repository contains original writing guidance and examples. It does not bundle the Medium article, the ASD standard, its dictionary or third party branding. It is independent of ASD and makes no claim of endorsement or full compliance.
