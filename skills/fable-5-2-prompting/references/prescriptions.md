# Prescriptions

The quoted blocks are from Anthropic's "Prompting Claude Fable 5.2" guide. The measurements are Anthropic's own test results. Results on other tasks can differ, so the user must measure.

## Contents
1. Finishing work
2. Approvals
3. Code ownership
4. Progress updates
5. Subagents
6. Caution that piles up
7. Clear reports

## 1. Finishing work

**Symptom:** the turn ends before the task is done. The model lists remaining work, offers to continue, or stops at an ordinary obstacle such as a timeout. This happens more often at `low` and `medium` effort.

**Add to the system prompt:**

```text
# Finishing work
Please avoid stopping while you still owe work the user asked for. Be persistent and do not stop unless needed.

While status notes are welcome, as is clarifying with the user on open decisions, carry on with work that does not depend on the user's answer without unnecessary stops.

If you notice yourself inviting the user to redirect you or offering to wait, instead proceed on the next part of the task. You may meet ordinary obstacles like errors, timeouts, locked files, empty results or failing tools. Diagnose them first, and when they are not real blockers, work through them with the access you have (wait and retry, fix the request, use another tool or source) rather than stopping unnecessarily.

If someone clearly decided something is a hard blocker, such as a file marked not to be touched, access that was intentionally withheld, or a safety guardrail, leave it alone, say plainly what you found, and look for another safe way to finish the task. Stopping before the task is complete is very rarely merited, as in when the task can't move forward without user input, or where the blocker is deliberate and should not be worked past. This does not override the need for confirmation on risky or destructive actions.
```

**Measured:** in recorded agentic coding sessions replayed at `medium`, a version of this text cut by about half the number of sessions that ended with unfinished work that could have been continued. Output tokens rose about 30%.

**Limits:** it removed most endings that ask permission or offer to continue. It did not remove endings that list remaining work and then stop.

**Keep:** the model asks less often with this text. Keep the user's own confirmation step for risky or irreversible actions.

## 2. Approvals

- When the user tells the model to go on, the message must say what to do. "Yes, run the migration on the staging database" is correct. "Continue" or a thumbs-up is not enough.
- An approval rule must name the exact actions. In a simulated task with no approver, a rule that listed "sending messages to people outside this conversation" made the model hold back about half of its routine requests to colleagues and other teams.

## 3. Code ownership

**Symptom:** the coding agent adds to the existing structure and duplicates code. It reports a problem but does not fix it.

**Add to the system prompt when the user wants the model to improve structure as it works:**

```text
You are the owner and long-term maintainer of this codebase, and its design is up to you.
```

**Measured (small coding tasks):** the model consolidated duplicated code in under 10% of runs without the sentence and in about 75% of runs with it.

**Cost:** output tokens rose 70-80%. The model made changes nobody asked for, many of them new tests and docs. In some runs it changed how existing features behave. The final PR summary often left out small changes, so the user must review the diff and the summary.

**Do not use** when the user owns the code and wants minimal changes. A scope rule does not cancel the sentence: with "No drive-by refactors, renames or cleanups" earlier in the same prompt, the model still consolidated in over half of runs.

## 4. Progress updates

**Symptom:** the agent seems silent during long tool runs, or makes its first tool call without saying what it will do.

**Step 1, check the client.** Updates between tool calls come back as `thinking` blocks. With the default `thinking.display: "omitted"` their text is empty, so a client that renders only `text` blocks shows nothing. See `api-harness.md`.

**Step 2, pick one.** Use the harness reminder where the platform supports it, because it has no notable cost. Use the system prompt sentence where it does not. Add the opener-and-recap text when a human is in the loop and wants an opening line and a closing recap.

Harness reminder (needs mid-conversation system messages). After several steps in a row with no readable text, append this after the latest tool result as a turn-scoped system message:

```text
The user hasn't heard from you in a while: say in a few words what you're doing, then continue.
```

Measured: the longest silent stretches were cut roughly in half, with no notable rise in tokens or time. Leave each reminder in `messages` on later requests.

Opener, updates, and recap (system prompt, for a human in the loop):

```text
Before you start, say in a line what you're about to do; brief updates while you work help the user follow along. Close with a short recap that stands on its own (what you found, what you did, and what's next), so a reader who only sees the last message has the full picture.
```

Measured: the model wrote a message before the first tool call in every run. Output tokens per task rose roughly 20%.

System prompt only (no harness change possible):

```text
Every few tool calls, give the user a one-line update on where things stand.
```

Measured: this sentence removed most long silent stretches in workflow automation tasks. The guide gives no token figure for it.

**Do not use** for unattended work where nobody needs frequent updates.

## 5. Subagents

**Symptom:** the session starts more subagents than the task needs.

1. Read the system prompt, skills, and agent files for lines that tell the model to delegate. Review those lines first.
2. Check which model the subagents use by default. Subagents can be a large part of session cost.
3. Change the subagent tool description only if over-delegation remains. The new description must state what a subagent adds to session cost, say when delegation is and is not worth it, and tell the model how to brief a subagent.

The guide gives no sample description. Write one for the user's harness from the three requirements above, and tell the user it is a draft to test.

**Measured (`high` effort):** a longer description focused on cost roughly halved the number of subagents compared with a shorter one.

**Do not change** a plain tool description when nothing else in the prompt asks for delegation.

**Note:** at `xhigh` and `max` the model can use subagents to review its own work. Early testers got good results with Fable 5.2 as planner and orchestrator for other models.

## 6. Caution that piles up

**Symptom:** the agent asks for approvals it does not need or withholds routine work. Seen mostly when several agents work together. Cautious lines collect in memory, notes, and instruction files.

- Remove notes that are no longer needed, such as an old "never retry".
- Name the routine steps that need no approval. "Act more freely" is too vague.
- Remove or narrow stop conditions such as "stop at the first error and review".
- Write exceptions explicitly. Use "ask for approval before deleting files", not "never delete files".

Keep the user's own confirmation step for risky or irreversible actions.

## 7. Clear reports

**Symptom:** the final report or handover is dense, or needs context the reader does not have.

**Add to the system prompt, and adapt it to the reader and product:**

```text
# Communicating clearly
Write for an intelligent but low-context reader. The person reading has not seen your tool calls, your reasoning or the files you opened, and may be switching in from other work. Messages should generally be self-contained: the reader should understand what you mean without having to dig for other context. Put the key points in the first few sentences. Practice progressive disclosure in your sentence order, and do not use undefined references and ideas in line 1 that won't make sense until the reader reaches line 12.

Use plain words, and write in clear, understandable full sentences. Do not use a term or abbreviation that you made up during the work. Say what the thing is in a way any reader would understand. Write mostly in the style of ASD-STE100 Simplified Technical English: short sentences, one idea per sentence, active voice, and consistently using the same word for the same thing. Use a paragraph of flowing prose where ideas connect, and keep bullets for lists of separate items. Leave out unnecessary details unless the reader asks and practice concision, but do ensure that your messages have the key context. Aim to make your user-facing text as human-readable and clear as possible.
```

**Measured:** final reports became less dense and used fewer semicolons. Sentences became shorter in workflow automation tasks.

**First** remove strict style rules that are no longer needed (see `audit-checklist.md`, item 6).
