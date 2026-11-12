# fable-5-2-prompting

An **unofficial** agent skill for auditing and tuning prompts for Claude Fable 5.2 and Claude Mythos 5.2. It is based on Anthropic's [Prompting Claude Fable 5.2](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-fable-5-2) guide, read on 2026-11-12. This project is not affiliated with or endorsed by Anthropic.

[한국어 README](README.ko.md)

## What it does

Fable 5.2 behaves differently from earlier models in a few specific ways. Instructions written for Fable 5.1 or Opus 5.5 can now cost more, cause side effects, or do nothing. This skill gives your agent two modes:

- **Audit.** The agent reads your system prompt, `CLAUDE.md`, skills, agent files, and memory. It reports each line that is a problem on Fable 5.2, with the reason and a proposed replacement. It does not edit your files.
- **Symptom lookup.** You describe what you see. The agent returns the fix from the guide, where to put it, and what it costs.

| You see | The skill points to |
|---|---|
| The agent stops before the task is done | The "Finishing work" text (about 30% more output tokens) |
| The agent asks for approval it does not need | Stale caution in memory and rules; approval rules that are too broad |
| The agent adds to old code and does not consolidate | The ownership sentence (70-80% more output tokens) |
| The agent is silent during long turns | `thinking.display`, then a reminder or a prompt line |
| Too many subagents | Standing delegation lines; the subagent tool description |
| Slow replies to simple requests | The `effort` level, not prompt text |
| `stop_reason: "refusal"` | The refusal category; requests to write reasoning in the response |

## Why it is not an "always-on" prompt

The guide says to apply one change at a time, for a behavior you observed, and to measure the result. Most fixes have a token cost. The skill tells the agent to follow that rule, so the agent does not paste every block into your prompt.

## Install

**Claude Code plugin:**

```
/plugin marketplace add sktjgudals/fable-5-2-prompting
/plugin install fable-5-2-prompting@fable-5-2-prompting
```

**Manual:** copy `skills/fable-5-2-prompting/` into `~/.claude/skills/` (or the skills directory of any harness that supports the Agent Skills format).

## Use

```
Audit my CLAUDE.md and agent files for Fable 5.2.
```

```
My Fable 5.2 agent keeps ending its turn with "want me to continue?". What do I change?
```

## Files

```
skills/fable-5-2-prompting/
├── SKILL.md                      symptom table and the two modes
└── references/
    ├── prescriptions.md          exact text, placement, cost, and limits for each fix
    ├── audit-checklist.md        lines to find in prompts written for earlier models
    └── api-harness.md            effort, progress display, preserved thinking, refusals
tests/                            test scenarios and a sample instruction file
```

## How it was tested

Each scenario in [tests/README.md](tests/README.md) was given to a fresh agent without the skill and then with it. Without the skill, the agent kept a broad approval rule that the guide warns about, did not know about the `reasoning_extraction` refusal, invented its own wording for the fixes, and estimated costs from the length of the text. With the skill, it gave the guide's text and measured costs, and it declined to add every block at once.

## Accuracy

- The guide is the source of truth. If it changes, this skill can be out of date. Check the guide for beta header names before you rely on one.
- The measurements are Anthropic's own test results. Measure on your own tasks.
- Found a mismatch with the guide? See [CONTRIBUTING.md](CONTRIBUTING.md). Changes are listed in [CHANGELOG.md](CHANGELOG.md).

## License

MIT for the original text in this repository. The example prompt texts are quoted from Anthropic's documentation and remain Anthropic's property. See [LICENSE](LICENSE).
