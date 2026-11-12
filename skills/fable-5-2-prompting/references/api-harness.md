# API and harness notes

Read this file when the user builds on the Claude API or maintains a harness. Beta header names were read from the guide on 2026-11-12. Check the guide before you rely on a header name. The guide links to the API reference pages that show the request format for each feature. This file does not reproduce them.

## Effort

Effort sets how much the model thinks, how many tool calls it makes, how long a turn takes, and how many tokens it uses. It has little effect on the length of the reply. Use prompting to change length and tone.

| Level | Use for |
|---|---|
| `low` | Fast back-and-forth, well-defined tasks, high volume, fast subagents |
| `medium` | Interactive coding, chat, everyday knowledge work |
| `high` (API default) | Hard coding, research, agentic work, long unattended runs |
| `xhigh` | The hardest long-running tasks, only where the user measured a gain |
| `max` | Measuring the model's limits; open problems in science or mathematics |

- Start at `high`. Then run evals at `medium` and `xhigh`.
- Do not carry over settings from Fable 5.1 or Opus 5.5. The levels have the same order across models, but the amount of thinking at each level can differ.
- Most of the capability gain comes at `high`. Above `high` the gain is uneven and depends on the task.
- At `low` or `medium`, add the "Finishing work" text for agentic work.
- At `high` and above the model thinks before nearly every reply, including a reply to a greeting. Lower the effort for that turn. A prompt instruction does not stop thinking at `medium` or above.
- To change effort during a conversation and keep the prompt cache, use per-message effort (beta, `mid-conversation-output-config-2026-07-01`). Without it, a new top-level `effort` value on the next request starts the prompt cache over.

## Prompt changes and the prefix check

The API applies a prefix check to all accounts on Fable 5.2. If `system`, `tools`, or an earlier message has changed, a request that sends back earlier thinking blocks can return a 400 error.

- Apply prompt and tool-description changes in new conversations only.
- To change instructions during a conversation, put them in the most recent turn, or append a mid-conversation system message where the platform supports it.

## Preserved thinking and history edits

If a harness changes the system prompt, tools, or earlier messages between requests, the thinking blocks after the changed point become invalid. The Messages API then rejects the request by default. The Message Batches API drops the blocks.

- Keep history append-only, apart from compaction and context editing. Cache hits stay high and thinking is not dropped.
- To shorten history, use context editing, or compaction that keeps no earlier turns after the summary.
- `thinking.block_binding.prefix_mismatch_behavior: "drop_block"` (beta, `thinking-binding-controls-2026-08-01`) makes the Messages API drop the blocks instead. The model then thinks again and uses more output tokens. Measure output tokens before you rely on it.

## Progress display

- `thinking.display: "omitted"` is the default. The update text is empty.
- `"updates"` (beta, `thinking-display-updates-2026-08-18`) returns a short summary of each update.
- `"summarized"` returns updates and summarized reasoning.
- Render the non-empty `thinking` blocks between tool calls.
- Send the "user hasn't heard from you" reminder as a turn-scoped system message (`clear_at: "next_user_message"`, beta, `mid-conversation-system-clear-at-2026-08-21`). Set `display` to `"updates"` or `"summarized"` before the harness counts the steps that gave the user nothing to read, because at `"omitted"` every `thinking` block is empty. Do not remove or reword a reminder on later requests, because that edits an earlier turn.

## Refusals

A declined request returns `stop_reason: "refusal"` with `stop_details.category`:

| Category | Meaning |
|---|---|
| `cyber` | Could enable cyber harm, such as malware or exploit development |
| `bio` | Could enable biological harm, such as dangerous lab methods |
| `frontier_llm` | Could help develop a competing AI model |
| `reasoning_extraction` | Asks the model to reproduce its internal reasoning in the response |
| `general_harms` | Another usage policy area |
| `null` | No named category |

- Benign security and life-sciences work can trigger `cyber` and `bio`. Anthropic has a Cyber Verification Program and a life sciences verification program for organizations with legitimate work. Claude Mythos 5.2 offers the same capabilities as Claude Fable 5.2, and it is available only to organizations verified through these programs.
- For `reasoning_extraction`, remove any instruction in a prompt, skill, or tool description that asks for reasoning in the response. To see how the model reasoned, set `display: "summarized"` and read the thinking blocks.
- To retry a declined request on another model, see "Refusals, fallback and billing" in the "What's new in Claude Fable 5.2" page.

## Where to start a migration

1. Set `effort` to `high` explicitly. Run evals at `medium` and `xhigh` too.
2. Read every prompt, skill, agent file, memory file, note, and harness text that you carry over to the new model. Apply `audit-checklist.md`.
3. Run the longest agentic task. Record where the model stops early, asks for unneeded approval, goes quiet, over-delegates, or adds to old code when a restructure was wanted.
4. For each recorded behavior, apply one fix at a time. Measure its effect and its token cost.
