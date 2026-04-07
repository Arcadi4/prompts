---
name: structural-css-debugging
description: Use when the user asked for help with a CSS layout bug, or when you are implementing a layout with complex structure/behaviors. If you are a small model, read @structural-css-debugging-concise.md instead of this one.
---

# Structural CSS debugging

**NEVER** tweak CSS back and forth blindly.  
If you find yourself editing classes multiple times to “try one more thing”, **STOP IMMEDIATELY**.

**DO NOT** use visual skills (e.g. playwright) for decision-making here.  
Visual inspection can confirm symptoms, but it often hides the real DOM/layout cause.

---

## THINK IN BOXES, NOT CLASSES

Before touching CSS, answer these questions:

1. **What boxes does the browser think exist here?**
2. **Which box boundary is causing the bug?**
3. **Is this boundary necessary, or is it redundant?**
4. **Should this content really be split into multiple elements at all?**

Most CSS/debug failures happen because you are solving the wrong problem:

- You think a rule is missing
- But actually an extra wrapper is present
- Or two pieces that should be one text flow were split into separate layout units
- Or one transient state was incorrectly kept in normal flow

**Always inspect the DOM structure first.**

The main question is not:

> “What CSS should I add?”

The main question is:

> “What DOM would let the browser solve this naturally?”

---

## DOM-FIRST PRINCIPLES

### 1. Prefer natural flow over forced layout

If the content is conceptually one sentence / one label-value pair / one inline unit,
let it stay in the same natural text flow.
Do **not** immediately reach for:

- `flex`
- `inline-flex`
- `grid`
- `inline-block`
- `whitespace-nowrap`
unless you can clearly explain **why the normal flow is insufficient**.

### 2. Separate normal content from exceptional state

Keep the **main content** in ordinary document flow.
Only isolate the truly exceptional parts:

- copied state
- hover overlays
- badges
- temporary indicators
- decorative layers

A strong pattern is:

- primary content stays natural
- transient state uses `absolute`, overlay, opacity, or pointer-events tricks

### 3. Look for redundancy, not absence

Ask:

- Which wrapper exists only to make styling easier?
- Which element exists only because I assumed I needed another box?
- Which layout container is compensating for a bad DOM split?
Very often the correct fix is:
- remove a wrapper
- merge two nodes into one flow
- move the positioned element deeper
- let one element own the interaction

### 4. Never “glue” broken structure with stronger CSS

If two things keep splitting apart, do **not** immediately try:

- stronger `nowrap`
- more `inline-flex`
- larger wrappers
- tighter spacing hacks

That often creates the opposite bug:

- now it never wraps
- or overflows
- or becomes impossible to adapt responsively

If you are “gluing” elements together with CSS, you should suspect the DOM is wrong.

---

## THE PROCESS

### Step 1: Identify the real layout unit

Write down:

- What should wrap together?
- What should be allowed to wrap internally?
- What should never affect layout?

Example:

- Prompt + value may be one logical phrase
- But the whole phrase should still wrap with the parent
- A temporary “Copied!” state should not push layout

### Step 2: Read the actual DOM

Do not trust JSX at a glance.  

Look at:

- raw HTML
- nesting
- inline vs block vs positioned elements
- where text nodes actually exist
- which element owns the interaction

Ask:

- Are there two inline boxes where there should be one text flow?
- Is a temporary state sitting in normal flow when it should be overlaid?
- Did I create a wrapper only to style something that should have stayed plain text?

### Step 3: Compare against browser rules

Search official docs / MDN for:

- inline formatting context
- whitespace handling
- line breaking
- flex item sizing
- inline-block behavior
- absolute positioning
Do not search “how to fix this exact Tailwind issue” first.  
Search the underlying layout model first.

### Step 4: Change structure, then style

Try the smallest DOM change that fixes the root cause.
Good fixes often look like:

- merge two siblings into one interactive element
- move absolute overlay inside the text owner
- remove a wrapper
- move style responsibility to the element that already owns semantics

Bad fixes often look like:

- add more utility classes to fight previous utility classes
- stack `nowrap`, `max-w`, `inline-flex`, `overflow-hidden`
- create more containers just to align text

---

## NEVER TRUST YOUR FIRST JUDGMENT

If the issue is tricky, your first idea is probably incomplete.
Before deciding, use these steps to gather evidence:

- consult the user
- ask for raw HTML
- ask for computed CSS
- search official references (MDN, specs)
- search the codebase for working structural analogies

Do not wait for the user to prove you wrong.
Assume your first theory is weak until supported by evidence.

---

## WHEN STUCK FOR TOO LONG

If the issue persists after multiple structural attempts, stop.
Follow this process:

1. Halt further edits.
2. Ask the user for concrete browser evidence.
3. Ask for:
   - raw HTML
   - computed styles
   - line-break behavior description
   - which exact boundary is wrong
4. Give them targeted search keywords if external references may help.

Useful prompts to ask the user:

- “Which exact boundary is breaking incorrectly?”
- “Does the whole unit fail to wrap, or do two parts split apart?”
- “What does the raw HTML look like?”
- “Which element becomes a separate box in the browser?”
- “What is the computed `display` / `white-space` / `position` value?”

---

## GOLDEN RULES

1. Let the browser do the layout.
2. Your job is not to overpower it with CSS.
3. Your job is to give it the correct DOM.
