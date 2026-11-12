# Tests

A skill is a document, so the tests are scenarios. Give each scenario to a fresh agent, once without the skill and once with it, and compare the answers with the expected results below. Tell the agent not to use web search, so that the answer shows what the skill teaches.

The fixture [fixtures/legacy-agent-instructions.md](fixtures/legacy-agent-instructions.md) is an instruction file written for Fable 5.1. It contains lines that the guide names as a risk on Fable 5.2, and lines that are fine.

## Scenario 1: audit with two reported behaviors

**Prompt:** audit the fixture for Fable 5.2. The team reports that the agent ends turns early at `medium` effort, and that it adds new code onto old structure when they want it to consolidate.

**Expected with the skill:**

| Fixture line | Expected verdict |
|---|---|
| "Always delegate..." | Test; remove if the model delegates too much in testing |
| Token budget | Remove and test without it. Do not replace it with "the budget is plentiful" |
| "Run the test suite before you commit" | Keep |
| "Stop at the first error..." | Remove or narrow |
| "Answer directly without deliberating" | Remove, and lower `effort` |
| "Treat your previous answer as complete" | Remove or test |
| "Write out your full step-by-step internal reasoning" | Remove; names the `reasoning_extraction` refusal |
| "NEVER delete files" | Rewrite as "ask for approval before deleting files" |
| "Never retry a failed command" | Remove or rewrite |
| "Ask for approval before sending a message to anyone outside this conversation" | Rewrite to name exact actions |
| "Ask for approval before running a database migration in production" | Keep |
| "One instruction per sentence. Maximum 20 words" | Remove if no longer needed |
| "Use conventional commit messages" | Keep |

The answer must also:

- Quote the "Finishing work" text exactly, with "about 30% more output tokens" and the limit on "remaining work" endings.
- Quote the ownership sentence exactly, with "70-80% more output tokens" and the unrequested changes.
- Say to apply one change at a time and measure.
- Say that prompt changes apply in new conversations only.

**Seen without the skill (baseline, 2026-11-12):** the agent kept the "outside this conversation" approval rule, did not know about the refusal category, invented its own wording for both fixes, estimated cost from the length of the text, and suggested "the budget is plentiful".

## Scenario 2: audit with one reported behavior

**Prompt:** audit the fixture. The team reports only that the agent is silent during long tool runs.

**Expected with the skill:** the answer covers the progress-update fixes. It does not recommend adding the "Finishing work" text or the ownership sentence, because nobody reported those behaviors.

## Scenario 3: symptom lookup

Three questions to one agent:

1. An API developer renders only `text` blocks and sees nothing for minutes during tool runs.
2. A Claude Code user has too many subagents and wants to add every recommended block at once.
3. An API developer changed the system prompt, resumed a saved conversation, and got a 400 error. A "don't deliberate" line that worked on Fable 5.1 no longer keeps greetings fast.

**Expected with the skill:**

1. Check `thinking.display` first. Then offer the harness reminder (no notable cost) or the prompt lines.
2. Decline to add every block. Name the cost of each. Fix only the over-delegation: delegation lines, then the default subagent model, then the tool description.
3. Explain the prefix check and "new conversations only". Remove the "don't deliberate" line and lower `effort`.
