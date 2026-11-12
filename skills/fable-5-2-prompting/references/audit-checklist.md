# Audit checklist

Use this list on every file the model reads: system prompt, CLAUDE.md, skills, agent files, memory, notes, and tool descriptions. For each matching line, give the verdict and the reason below.

Verdicts: **remove**, **rewrite**, **test** (the guide says to measure before deciding), **keep**.

| # | Find lines like | Verdict | Reason from the guide |
|---|---|---|---|
| 1 | "Always delegate", "spawn a subagent for every..." | Test. Remove if the model delegates too much in testing | Fable 5.2 delegates readily when the prompt encourages it. Subagents can be a large part of session cost. |
| 2 | A token budget shown to the model, or the amount remaining | Remove, then test without a budget | The size of the budget changes how much the model spends. Removing a large budget lowered cost with no loss of performance at `high` and `xhigh`. Do not replace it with "the budget is plentiful". This is about prompt text, not the API task budgets feature. |
| 3 | "Treat your previous answer as complete", "focus thinking on the new message" (from the Opus 5.5 guide) | Remove if the model must keep checking earlier answers. Otherwise test. | At `high` these lines reduced thinking on follow-ups and made the model much less likely to point out its own earlier mistakes. |
| 4 | "Answer directly without deliberating", "do not overthink" | Remove. If simple requests are slow, lower `effort` | These lines do not stop thinking at `medium` or above. |
| 5 | "Write your full reasoning in the answer", "show your step-by-step thinking" | Remove | Can return `stop_reason: "refusal"` with category `reasoning_extraction`. To see reasoning, set `display: "summarized"` and read the thinking blocks. |
| 6 | Strict style limits: "one instruction per sentence", "20 words or fewer" | Remove if no longer needed | The model follows these closely and the prose becomes choppy. |
| 7 | "Never X" with no scope or reason: "never delete files", "never retry" | Remove if the reason no longer holds. Otherwise rewrite with scope and reason, or as "ask for approval before X" | The model treats "never" as absolute. |
| 8 | "Stop at the first error and wait for review" | Remove or narrow | Blocks autonomous work. |
| 9 | Broad approval rules: "ask before messaging anyone outside this conversation" | Rewrite to name the exact actions | This rule made the model hold back about half of its routine requests to colleagues and other teams. |
| 10 | Old cautions in memory or notes, including ones the model wrote | Delete those whose reason no longer holds | Caution builds up across sessions. If a system is already stuck, consider restarting it with its memory and notes trimmed. |
| 11 | "No drive-by refactors" used together with an ownership sentence | Test. Tell the user that the two lines conflict | The scope rule does not cancel the ownership sentence. |

## Keep

- Specific approval rules for risky or irreversible actions, such as "ask before a production database migration".
- Project facts: commands, paths, conventions, test requirements.

## Harness checks (ask the user; these are not in the prompt files)

- Does harness code edit the system prompt, tools, or earlier messages between requests? See `api-harness.md`, preserved thinking.
- Does the client render non-empty `thinking` blocks between tool calls? See `api-harness.md`, progress display.
- Were effort settings copied from an earlier model? Run the sweep again.

## Report format

One row per line reviewed:

| Line | Verdict | Reason | Proposed text |
|---|---|---|---|

After the table, state:
1. Which findings come straight from the guide and which need the user's own test.
2. The order to apply them: one at a time, with a measurement after each.
3. That prompt changes apply in new conversations only.
