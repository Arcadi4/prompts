---
name: structural-css-debugging-concise
description: Use when you have a written implementation plan to execute in a separate session with review checkpoints.
---

# Structural CSS debugging

Do **not** tweak CSS back and forth.  
If you edit classes repeatedly to “try one more thing”, **STOP**.
Do **not** use visual tools (e.g. playwright) to decide layout fixes.  
They show symptoms, not structure

---

## THINK IN BOXES, NOT CLASSES

Before changing CSS, ask:

1. What boxes does the browser think exist here?
2. Which boundary is causing the bug?
3. Is that boundary necessary, or redundant?
4. Should these nodes be separate elements at all?

Most CSS bugs are not “missing rules”, They are:

- extra wrappers
- wrong box boundaries
- normal content split into multiple layout units
- transient state left in normal flow

The real question is not:

> “What CSS should I add?”

It is:

> “What DOM lets the browser solve this naturally?”

---

## DOM-FIRST RULES

### 1. Prefer natural flow

If something is conceptually one phrase / one unit / one label-value pair, keep it in the same normal flow first.
Do **not** jump straight to:

- `flex`
- `inline-flex`
- `grid`
- `inline-block`
- `whitespace-nowrap`

unless normal flow is clearly insufficient.

### 2. Separate normal content from exceptional state

Keep the main content in normal document flow.
Only pull out the exceptional parts:

- copied state
- overlays
- badges
- temporary indicators
- decorative layers

Common good pattern:

- main content stays natural
- transient state uses `absolute`, overlay, opacity, etc.

### 3. Remove redundancy before adding rules

Ask:

- Which wrapper exists only for styling convenience?
- Which element exists only because I assumed I needed another box?
- Which layout container is compensating for a bad DOM split?

Often the right fix is:

- remove a wrapper
- merge two siblings into one flow
- move an overlay deeper
- let one semantic element own the interaction

### 4. Never glue bad structure with stronger CSS

If elements keep splitting apart, do **not** keep escalating with:

- `nowrap`
- more wrappers
- bigger layout containers
- spacing hacks
That usually creates the opposite bug.
If you are gluing elements together with CSS, suspect the DOM first.

---

## DEBUG LOOP

1. **Define the unit**
   - What should stay together?
   - What may wrap internally?
   - What must not affect layout?
2. **Read the real DOM**
   - raw HTML
   - actual nesting
   - inline vs block vs positioned elements
   - where text nodes really are
3. **Check browser rules**
   Search official docs / MDN for:
   - inline formatting
   - whitespace / wrapping
   - flex sizing
   - inline-block behavior
   - positioning
4. **Change structure first**
   Make the smallest DOM change that removes the bad boundary.
   Only then add or simplify CSS.

---

## NEVER TRUST FIRST JUDGMENT

If the issue is tricky, your first idea is probably incomplete.
Before deciding:

- inspect raw HTML
- inspect computed styles
- search official references
- search the codebase for working structural analogies
- ask the user what boundary is actually wrong

Do not wait for the user to prove you wrong.

---

## WHEN STUCK

Stop editing and ask for evidence:

- What exactly is breaking?
- Which boundary is wrong?
- What does the raw HTML look like?
- What are the computed values for `display`, `white-space`, `position`?
- Does the whole unit fail to wrap, or do two parts split apart?

---

## GOLDEN RULE

Let the browser do the layout.
Your job is not to overpower it with more CSS.
Your job is to give it the correct DOM.
