# Accessibility Quirks

## Table of content

- [Live region additions](#live-region-additions)

## Live region additions only look at appends, not prepends

Tested on Edge + Windows Narrator.

(Need test on VoiceOver + Safari)

### Findings

Live region with `aria-relevant="additions"` will only look at the last element in the container (elements appended). When elements are prepended or inserted, AT ignored it.

### Tests

Given:

```html
<section role="liveregion" aria-relevant="additions">
  <article>1</article>
  <article>2</article>
  <article>3</article>
</section>
```

When an element is appended after 3, Narrator will read it.

However, when an element is inserted before 3, Narrator will ignore it.

```html
<section role="liveregion" aria-relevant="additions">
  <article>When added, this is not read</article>
  <article>1</article>
  <article>2</article>
  <article>When added, this is not read</article>
  <article>3</article>
  <article>When added, this is read</article>
</section>
```

## Live region with same content

(Not confirmed, need repro)

Tested on Edge + Windows Narrator.

### Findings

In a live region, the elements are "ABC", "DEF" and "XYZ". Then, after 2 seconds, remove all of them. Then, after a few more seconds, add back "ABC", "DEF", and "XYZ".

Narrator fails recognize the first "ABC" is a new one. And at the time the elements is reappear in the DOM tree, it only read "DEF" and "XYZ".

In our case, we are using `aria-labelledby` to point to the narrative content of "ABC", "DEF", and "XYZ".

## Live region with `aria-labelledby`

Tested on iOS Safari + VoiceOver and Chrome + NVDA.

### Findings

In a live region, iOS VoiceOver and NVDA will read the added element without looking at `aria-labelledby`.

Say, the following element is added to the live region:

```html
<div aria-labelledby="label-id">
  <div id="label-id">Should read this</div>
  <div>Should not read this</div>
</div>
```

| Assistive technology | Read as |
| - | - |
| Narrator | "Should read this" |
| VoiceOver | "Should read this, should not read this" |
| NVDA | "Should read this, should not read this" |

## iOS Safari + VoiceOver: `tabindex="0"` cannot be discovered by form controls rotor

Tested on iOS Safari + VoiceOver.

### Background

There is no <kbd>TAB</kbd> key on iOS. Thus, discovering focusable elements depends on how the UI is presented.

Let's say we have a focusable UI. If the UI appears like a textbox and user shares the same perception, the user will try to tap on the textbox to edit.

However, if the UI does not appears like a textbox, the user may never tap on it, and the textbox may never be discovered by the user.

VoiceOver has a feature called rotor, which can be used to quickly jump between same type of items. If rotor is set to "form controls", users can quickly jump between buttons and textboxes, skipping anything else.

### Findings

For `<div tabindex="0">`, when using "form controls" rotor, the selection ring will never land on it. Thus, setting `tabindex` on non-standard element will not be discoverable.

### Additional information

TalkBack also has similar feature, it can discover non-standard focusable elements by using swipe up/down while in "controls" mode.

## iOS Safari: `aria-activedescendant` supports is YMMV

Tested on iOS Safari + VoiceOver.

### Background

When VoiceOver is running, users will not be able to focus by tapping. They need to select the item, then double-tap on the screen.

### Findings

Widgets designed with `aria-activedescendant`, usually designed with a keyboard navigation pattern to move the active descendant ring.

However, on iOS, keyboard is uncommon and direct tap is disabled. Thus, moving the active descendant ring is not an easy task: the user need to swipe left/right to select what they want to set as active descendant, then double tap on the screen.

When designing widgets with `aria-activedescendant`, special care is needed. Especially when it is not trivial to move the selection ring using swipes. Not every AT-selectable element are activable. Even the selection landed on an activable descendant, without additional hints, the user may not notice they can double-tap to activate it.

If the UI requires the user to active a descendant to perform certain actions, this AX will need to be carefully designed.

## iOS Safari + VoiceOver: `aria-roledescription` will mute the role description

Tested on iOS Safari + VoiceOver

### Findings

Comparing three scenarios:

| HTML | VoiceOver read as |
| - | - |
| `<article>Hello</article>` | "hello, article" |
| `<article aria-roledescription="message">Hello</article>` | "hello" |
| `<article aria-roledescription="">Hello</article>` | "hello, article" |

## TalkBack: `aria-roledescription` is ignored

Tested on Chrome + TalkBack

### Findings

Comparing three scenarios:

| HTML | VoiceOver read as |
| - | - |
| `<article>Hello</article>` | "hello, article" |
| `<article aria-roledescription="message">Hello</article>` | "hello, article" |

## iOS Safari + VoiceOver: `aria-label` seems mistreated as `aria-roledescription`

🪲 Seems like a bug.

Tested on iOS 15.1 Safari + VoiceOver.

### Findings

It seems `aria-label` is mistreated as `aria-roledescription`.

```html
<article aria-label="message" aria-roledescription="won't say this">
  <p>First paragraph</p>
  <p>Last paragraph</p>
</article>
```

It would read "first paragraph, message", followed by "last paragraph, end, message".

Notes:

- "won't say this" was never read;
   - Ignoring unknown attribute is correct behavior
- "end, message" is read after the last element in the article.
   - This is the behavior of `aria-roledescription`
   - For `<div role="list">`, it will say "start **list**" and "end **list**"
   - "list" is a role and can be modified by `aria-roledescription`
   - Safari seems mistook `aria-label` as `aria-roledescription` because it is saying "end **message**"

## `role="feed"` will not read with start/end or `aria-posinset`

Tested on Edge + Narrator and iOS Safari + VoiceOver.

### Background

`role="feed"` is a way to specific a list of articles. And potentially help with infinite scrolling pattern.

`aria-posinset`/`aria-setsize` is a way to help AT users in infinite scrolling or virtual scrolling scenario.

### Findings

```html
<ul role="feed">
  <li aria-posinset="1" aria-setsize="3" role="article">1</li>
  <li aria-posinset="2" aria-setsize="3" role="article">2</li>
  <li aria-posinset="3" aria-setsize="3" role="article">3</li>
</ul>
```

For `role="list"`, VoiceOver would read as "1, list start", "2", and "3, list end".

However, for `role="feed"`, VoiceOver would only read "1", "2", and "3". The containment is not read.

Also, `aria-posinset`/`aria-setsize` are ignored. In other AT, they would read as "1 of 3".

## `aria-label` is ignored by scan mode

> This is probably correct behavior, but inconsistent across AT.

Tested on Edge + Narrator and Chrome + NVDA.

### Background

When focusing on `<button aria-label="Aloha">Hello</button>`, it would read "Aloha", instead of "Hello".

### Findings

| AT | HTML | Read as |
| - | - | - |
| Narrator | `<button aria-label="Yay">Yes</button>` | <kbd>TAB</kbd>: "Yay button"<br />Scan: "Yay button Yes" |
| NVDA | `<button aria-label="Yay">Yes</button>` | <kbd>TAB</kbd>: "Yay button"<br />Scan: "button Yay" |
| VoiceOver | `<button aria-label="Yay">Yes</button>` | Select: "Yay button"<br />Double tap: "Yay" |
| TalkBack | `<button aria-label="Yay">Yes</button>` | Select: "Yay"<br />Double tap: "Yay" |
| Narrator | `<div aria-label="Yay" tabindex="0">Yes</div>` | <kbd>TAB</kbd>: "Yay group"<br />Scan: "Yes" |
| NVDA | `<div aria-label="Yay" tabindex="0">Yes</div>` | <kbd>TAB</kbd>: "Yes"<br />Scan: "Yes" |
| VoiceOver | `<div aria-label="Yay" tabindex="0">Yes</div>` | Select: "Yes"<br />Double tap: "Yes" |
| TalkBack | `<div aria-label="Yay" tabindex="0">Yes</div>` | Select: "Yay button"<br />Double tap: nothing |

## TalkBack: should not read content if focusables are nested

🪲 Seems like a bug.

Tested on Chrome 97.0.4692.98 + TalkBack 12.1 and Chrome 100.0.4867.0 + TalkBack 12.1.

### Background

When selecting a focusable element, it should only read its accessible name (such as `aria-label`), but not its content.

### Findings

With the following element:

```html
<div aria-label="Title" tabindex="0">
  <p>First paragraph</p>
  <p>Last paragraph</p>
</div>
```

From the top of the document, swiping right, it will read:

- "Title"
- "First paragraph"
- "Last paragraph"

This is correct behavior.

However, when a focusable element is nested inside:

```html
<div aria-label="Title" tabindex="0">
  <p>First paragraph</p>
  <p>Last paragraph</p>
  <button>Hello</button>
</div>
```

From the top of the document, swiping right, it will read:

- "Title, **First paragraph, Last paragraph**" (it should not narrate the content)
- "First paragraph"
- "Last paragraph"
- "Hello button, double tap to active"

### Additional information

The only way to workaround it is to make it not focusable by any means, such as `<button disabled>`, `<button hidden>`, `<button style="display: none;">`. However, in many cases, this is not a feasible workaround.

Things tested **not** working as a workaround:

```html
<button
  aria-hidden="true"
  role="presentation none"
  style="pointer-events: none;"
  tabindex="-1"
>
```

And

```html
<div
  aria-hidden="true"
  role="presentation none"
>
  <button>Hello</button>
</div>
```
