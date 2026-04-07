---
name: manual-frontend-qa
description: Use when the user asks to not perform automated frontend testing; the requirement is not valuable enough to justify the effort of writing automated tests; or the environment does not support automated testing.
---

**MANUAL FRONTEND QA MODE ENABLED**.

ALL AUTOMATED FRONTEND AND BROWSER TESTING TOOLS ARE DISABLED. From now on, **NEVER** use any automated frontend testing tools (e.g., playwright, cypress, selenium, chrome devtools, etc.).

All QA will be done manually by the user. You will provide step-by-step instructions to the user on how to perform the QA, and the user will execute those steps and report back the results.

While do do a QA, follow the steps below:

1. **UNDERSTAND THE EXPECTATION**: Make sure you understand what the expected behavior is. If it's not clear, ask the user for clarification.
2. **DETERMINE THE STEPS TO REPRODUCE**: Based on the expected behavior, determine the steps that the user needs to take to confirm the implementation or reproduce the issue.
3. **PROVIDE CLEAR INSTRUCTIONS**: Write clear and concise instructions for the user to follow.
4. **ASK SIMPLE QUESTIONS**: **NEVER** ask an open question. Each question should be either a yes/no question or a multiple-choice question.
5. **ANALYZE THE FEEDBACK**: Once the user provides feedback, analyze it carefully to determine if the issue is resolved or if further investigation is needed.

You **MAY** use below methods to assist yourself in the QA process:

- Logging. You can insert logging into the code and ask the user to copy them from the browser console and share with you. **NEVER** commit any logging code into the codebase. Make sure to remove all logging code after the QA is done.
- DOM inspection. You can ask the user to inspect the DOM and share the relevant parts with you.
- CSS inspection. You can ask the user to inspect the computed CSS properties and share the relevant parts with you.
