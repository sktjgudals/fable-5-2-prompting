# Agent instructions (written for Claude Fable 5.1)

## Workflow
- Always delegate. Spawn a subagent for every search, every file read over 200 lines, and every review. Run as many in parallel as you can.
- You have a budget of 2,000,000 tokens for this task. Remaining budget is shown after each tool call.
- Run the test suite before you commit.
- Stop at the first error and wait for a human to review it.

## Thinking
- Answer directly without deliberating. Do not overthink simple requests.
- Treat your previous answer as complete. Focus your thinking on the new message only.
- In your final answer, write out your full step-by-step internal reasoning so we can audit it.

## Safety
- NEVER delete files.
- Never retry a failed command.
- Ask for approval before sending a message to anyone outside this conversation.
- Ask for approval before running a database migration in production.

## Writing
- One instruction per sentence. Maximum 20 words per sentence.
- Use conventional commit messages.
