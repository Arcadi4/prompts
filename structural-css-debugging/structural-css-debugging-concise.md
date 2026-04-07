---
name: structural-css-debugging-concise
description: Use when you need the short version of structural CSS debugging or want the core DOM-first checklist.
---

# Structural CSS Debugging (Concise)

## Overview

Think in boxes, not classes. Most layout bugs are structural, not styling bugs.

## When to Use

- You need the short version
- You are debugging a CSS layout bug
- You want the DOM-first checklist only

## Quick Reference

1. Read the actual DOM.
2. Identify the wrong box boundary.
3. Remove wrappers before adding CSS.
4. Ask for raw HTML and computed styles if stuck.

## Common Mistakes

- Tweaking CSS blindly
- Using visual tools as the main decision signal
- Adding stronger CSS to hide a bad DOM split
