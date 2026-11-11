# Clear Technical English

A practical writing skill inspired by [ASD Simplified Technical English](https://www.asd-ste100.org/index.html) and the Medium article linked in [SOURCES.md](SOURCES.md).

It helps an AI assistant use short sentences, consistent technical terms and clear instructions while preserving the intended meaning.

## When to use it

Use it for technical explanations, documentation and rewrites. It is a writing aid rather than a reasoning audit or a formal STE compliance checker.

## With and without the skill

These are unchanged excerpts from the same rewrite request in two fresh Haiku 5.5 sessions. The baseline had no skills enabled. The other session had only Clear Technical English enabled.

### Without the skill

> Before starting the staging deployment, the operator must confirm that all 12 local tests have passed and that CI has completed successfully. If either condition is not met, the deployment must not proceed.

### With the skill

> Before you start the staging deployment, confirm both of these conditions:
>
> 1. All 12 local tests have passed.
> 2. CI has completed successfully.
>
> Do not deploy if either condition is not met.

The skill separates the two prerequisites into a numbered list and addresses the reader directly. Both excerpts retain the test and CI conditions. This illustrates one result, not a guarantee that every answer will improve.

## Install

Clone the repository and enter its directory:

```sh
git clone https://github.com/sarangkharche/skills.git
cd skills
```

### Claude Code

```sh
mkdir -p ~/.claude/skills
cp -R ./clear-technical-english ~/.claude/skills/
```

For a shared project installation, copy the folder into the project's `.claude/skills/` directory instead.

### Codex

```sh
mkdir -p ~/.codex/skills
cp -R ./clear-technical-english ~/.codex/skills/
```

Start a new chat after installation. No additional services or credentials are required by the skill.

## Use

In Claude Code:

```text
/clear-technical-english Rewrite this explanation for an engineer who is new to the service
```

In Codex:

```text
$clear-technical-english Explain this service
```

Provide your text after a rewrite request. Follow your company's templates and review important material before use.

Automatic selection is enabled in the Codex skill metadata. Whether the skill is selected depends on the request and your assistant's instructions. Installing it alone does not force it to be used on every technical answer.

## Validation

The skill was tested on a sample rewrite in Claude Code 2.1.311 with Haiku 5.5. The comparison used two fresh sessions, with all other skills disabled. The skill improved paragraph structure and retained the supplied technical details and health check recommendation in that example.

Automatic selection was also observed in one fresh local Codex chat. These small checks do not establish consistent improvement across all tasks or models.

## Sources and scope

[Source coverage](SOURCES.md) records the article, official website review and limits of the standard review. See the [writing notes](writing-notes.md) for intentional adaptations and [original examples](examples.md) for sample rewrites.

This skill contains original writing guidance and examples. It does not bundle the Medium article, the ASD standard, its dictionary or third party branding. It is independent of ASD and makes no claim of endorsement or full compliance.
