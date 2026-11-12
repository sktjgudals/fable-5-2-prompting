# Contributing

Thank you for helping keep this skill accurate.

## The one rule

Anthropic's [Prompting Claude Fable 5.2](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-fable-5-2) guide is the source of truth. Every claim, figure, and quoted prompt text in this skill must come from that guide. Do not add advice from other sources, personal experience, or guesses. If the guide gives no text or no figure for something, the skill must say so.

## Report a mismatch

Open an issue with:

1. The file and line in this repository.
2. The sentence in the guide that it disagrees with.
3. The date you read the guide.

## Send a change

1. Change the skill files.
2. Run the scenarios in [tests/README.md](tests/README.md) with a fresh agent that has the skill, and check the expected results.
3. If the guide changed, update the "read on" date in `SKILL.md`, `references/api-harness.md`, and both README files, and add an entry to `CHANGELOG.md`.
4. Open a pull request. Say which scenarios you ran and what the agent did.

## Style

- American English, short sentences, one idea per sentence.
- Quote the guide's prompt texts exactly. Do not reword them.
- Keep `SKILL.md` short. Put detail in `references/`.
- The `description` in `SKILL.md` says when to use the skill. It does not summarize what the skill does.

## Translations

`README.ko.md` is a Korean translation of `README.md`. Translations of the README into other languages are welcome. The skill files stay in English, because the agent answers in the user's language.
