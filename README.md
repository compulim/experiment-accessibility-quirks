# Accessibility Quirks

## Table of content

- [Live region additions](#live-region-additions)

## Live region additions only look at appends, not prepends

Tested on Edge + Windows Narrator.

### Findings

Live region with `aria-relevant="additions"` will only look at the last element in the container. It should also applies to when `aria-relevant` is not present or is presented with other token, such as `"additions text"`.

### Tests

Given:

```html
<section role="liveregion" aria-relevant="additions">
  <article>1</article>
  <article>2</article>
  <article>3</article>
</section>
```

When an element is appended after 3, Windows Narrator will read it.

However, when an element is inserted before 3, Windows Narrator will ignore it.

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

## iOS Safari: `aria-activedescendant` supports is YMMV

Tested on iOS Safari + VoiceOver.

### Background

When VoiceOver is running, users will not be able to focus by tapping. They need to select the item, then double-tap on the screen.

### Findings

Widgets designed with `aria-activedescendant`, usually designed with a keyboard navigation pattern to move the active descendant ring.

However, on iOS, keyboard is uncommon and direct tap is disabled. Thus, moving the active descendant ring is not an easy task: the user need to swipe left/right to select what they want to set as active descendant, then double tap on the screen.

## iOS Safari + VoiceOver: `aria-roledescription` will mute the role description

Tested on iOS Safari + VoiceOver

### Findings

Comparing three scenarios:

| HTML | VoiceOver read as |
| - | - |
| `<article>Hello</article>` | "hello, article" |
| `<article aria-roledescription="message">Hello</article>` | "hello" |
| `<article aria-roledescription="">Hello</article>` | "hello, article" |

## iOS Safari + VoiceOver: `aria-label` seems mistreated as `aria-roledescription`

Tested on iOS Safari + VoiceOver

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
- "end, message" is read after the last element in the article.
