---
name: manual-frontend-qa
description: Use when the user requests manual QA, automated testing isn't valuable enough to justify effort, or the environment doesn't support automated testing
license: CC-BY-NC-4.0
---

# Manual Frontend QA

## Overview

Disable automated frontend testing tools and guide the human partner through manual quality assurance workflows.

**Core principle:** You provide step-by-step instructions for human execution, never automated testing.

## When to Use

Use this skill when:

- The human partner explicitly requests manual QA
- Automated testing isn't valuable enough to justify the setup effort
- The environment lacks automated testing support
- You need quick verification without test infrastructure

## Process

**Never use automated testing tools** (playwright, cypress, selenium, chrome devtools, etc.) during manual QA.

Follow these steps:

1. **Understand expectations**: Clarify expected behavior. Ask if anything is unclear.
2. **Determine reproduction steps**: Outline specific actions the partner must take to verify or reproduce.
3. **Provide clear instructions**: Write concise, executable steps for the partner to follow.
4. **Ask simple questions**: Use yes/no or multiple-choice questions only. Never ask open-ended questions.
5. **Analyze feedback**: Review partner responses to determine resolution status or next steps.

## Allowed Manual Evidence Methods

Use these manual techniques:

- **Temporary logging**: Insert logging code, ask for console output. Never commit logging code. Clean up after QA.
- **DOM inspection**: Ask partner to inspect DOM and share relevant elements.
- **CSS inspection**: Ask partner to inspect computed CSS properties and share values.

## Rationalization Prevention

Rationalization to avoid automated testing violates this skill's core principle.

| Excuse | Reality |
| ------ | ------- |
| "Just quickly check with devtools" | Any automated checking is prohibited |
| "It's faster to use playwright" | Speed doesn't justify violating manual QA |
| "The partner won't notice" | Violating rules breaks trust |
| "I can combine manual and automated" | No automated tools whatsoever |
| "It's just one small test" | Even one automated check violates |

## Red Flags - STOP

- Thinking about using automated testing tools
- Starting to write playwright/cypress code
- Asking partner to install testing libraries
- Considering "just this once" exceptions
- Feeling tempted by automation shortcuts

**All of these mean: Stay in manual QA mode. Partner executes, you instruct.**

## Common Mistakes

- **Using automated tools**: Never use playwright, cypress, selenium, chrome devtools, or any automated frontend testing tools during manual QA mode.
- **Asking open questions**: Stick to yes/no or multiple-choice questions for clarity.
- **Forgetting to remove logging**: Always clean up temporary logging code after QA.
- **Assuming tool availability**: Don't assume the partner has specific browser extensions or developer tools beyond basic browser inspection capabilities.
