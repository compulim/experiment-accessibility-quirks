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

> Could be related to [crbug#863375](https://bugs.chromium.org/p/chromium/issues/detail?id=863375).

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

| Assistive technology | Read as                                  |
| -------------------- | ---------------------------------------- |
| Narrator             | "Should read this"                       |
| VoiceOver            | "Should read this, should not read this" |
| NVDA                 | "Should read this, should not read this" |

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

| HTML                                                      | VoiceOver read as |
| --------------------------------------------------------- | ----------------- |
| `<article>Hello</article>`                                | "hello, article"  |
| `<article aria-roledescription="message">Hello</article>` | "hello"           |
| `<article aria-roledescription="">Hello</article>`        | "hello, article"  |

## TalkBack: `aria-roledescription` is ignored

Tested on Chrome + TalkBack

### Findings

Comparing three scenarios:

| HTML                                                      | VoiceOver read as |
| --------------------------------------------------------- | ----------------- |
| `<article>Hello</article>`                                | "hello, article"  |
| `<article aria-roledescription="message">Hello</article>` | "hello, article"  |

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

## `aria-label` sometimes ignored by scan mode with generic widget

The behavior is inconsistent across AT.

### Background

When focusing on `<button aria-label="Yay">Yes</button>`, it would read "Yay", instead of "Yes". This is consistent across AT.

When focusing on `<div aria-label="Yay" tabindex="0">Yes</div>`, it varies.

### Findings

| AT        | HTML                                           | Read as                                                  |
| --------- | ---------------------------------------------- | -------------------------------------------------------- |
| Narrator  | `<button aria-label="Yay">Yes</button>`        | <kbd>TAB</kbd>: "Yay button"<br />Scan: "Yay button Yes" |
| NVDA      | `<button aria-label="Yay">Yes</button>`        | <kbd>TAB</kbd>: "Yay button"<br />Scan: "button Yay"     |
| VoiceOver | `<button aria-label="Yay">Yes</button>`        | Select: "Yay button"<br />Double tap: "Yay"              |
| TalkBack  | `<button aria-label="Yay">Yes</button>`        | Select: "Yay button"<br />Double tap: nothing            |
| Narrator  | `<div aria-label="Yay" tabindex="0">Yes</div>` | <kbd>TAB</kbd>: "Yay group"<br />Scan: "Yes"             |
| NVDA      | `<div aria-label="Yay" tabindex="0">Yes</div>` | <kbd>TAB</kbd>: "Yes"<br />Scan: "Yes"                   |
| VoiceOver | `<div aria-label="Yay" tabindex="0">Yes</div>` | Select: "Yes"<br />Double tap: "Yes"                     |
| TalkBack  | `<div aria-label="Yay" tabindex="0">Yes</div>` | Select: "Yay"<br />Double tap: nothing                   |

ATs are not consistent on how to handle `aria-label` for roles that are not expected to be interactive, e.g. `role="generic"`.

It boils down to how ATs should handle `<p aria-label="Yay">Yes</p>`.

Generalizing the table above:

- TalkBack always use `aria-label`
- NVDA and VoiceOver use `aria-label` for interactive roles, and text node for static roles
  - [`role="separator"`](https://www.w3.org/TR/wai-aria/#separator) can be static or interactive, depends on whether it is focusable or not
- Narrator use `aria-label` when focused, but text node when scanned
  - This is somehow similar to NVDA and VoiceOver, but Narrator use a different strategy on deciding interactivity: <kbd>TAB</kbd> means interactive, scan means static
  - When Narrator consider the role is interactive, it concatenates both `aria-label` and text node into "Yay button Yes". This seems a bug 🪲
  - When `aria-labelledby` or `aria-label` is present, accessible name computation should not consider its content
  - Correct reading should be "Yay button", instead of "Yay button Yes"

### Conclusions

Don't use `aria-label` on static roles if possible.

Narrator seems bugged when computing accessible name for `<button aria-label="Yay">Yes</button>`.

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
<button aria-hidden="true" role="presentation none" style="pointer-events: none;" tabindex="-1"></button>
```

And

```html
<div aria-hidden="true" role="presentation none">
  <button>Hello</button>
</div>
```

## Repeated readings when using `aria-labelledby`

When using `aria-labelledby` on a widget, such as list item. The label will be narrated as part of the widget.

Then, continue to scan, the original label will be read again.

The repetitions should be expected.

### Findings

> We are using `role="listbox"` in the sample because the role is read even in scan mode.

```html
<div aria-labelledby="item-label-1" role="listbox">
  <span>Bot said:</span>
  <span id="item-label-1">Hello, World!</span>
  <span>Sent just now</span>
</div>
```

When using screen reader, it will roughly read the followings:

- Enter list, selected, selection contains zero item, "Hello, World!"
- "Bot said"
- "Hello, World!"
- "Sent just now"

Note: "Hello, World!" will be narrated twice.

However, when we remove "Bot said:" in the forementioned DOM tree. The screen reader would read:

- Enter list, selected, selection contains zero item, "Hello, World!"
- "Sent just now"

Note: "Hello, World!" is only narrated once, despite it is unchanged.

### Conclusions

`aria-labelledby` is a way to _briefly describe the type of content_. This should be very similar to `aria-roledescription` and could consider as an extension to role description mechanism. Thus, `aria-labelledby` should be used _very sparingly_.

To reduce repeated readings, we should refrain from using `aria-labelledby` to read the element content. Screen reader users should be able to pick up the content when scanning or asking the screen reader to read the whole document.

## Repeated readings in live region

🪲 Seems like a bug.

When using `aria-labelledby` in a live region, Windows Narrator will repeat the reading.

```html
<article aria-labelledby="id-00001">
  <div id="id-00001">Hello, World!</div>
</article>
```

### Narrations

| Screen reader                         | Version tested        | Narrations                           | Notes                                       |
| ------------------------------------- | --------------------- | ------------------------------------ | ------------------------------------------- |
| Windows Narrator + Edge 102.0.1245.41 | Windows 10 19043.1766 | Hello, World! Article. Hello, World! | "Hello, World!" is being narrated twice. 🪲 |
| Windows Narrator + Edge 102.0.1245.41 | Windows 11 22621.160  | Hello, World! Article. Hello, World! | "Hello, World!" is being narrated twice. 🪲 |
| NVDA + Chrome 102.0.5005.115          | 2022.1                | Hello, World!                        |                                             |
| NVDA + Firefox 101.0.1                | 2022.1                | Hello, World!                        |                                             |
| TalkBack                              | Android 11            | Hello, World!                        |                                             |
| VoiceOver                             | macOS 12.4            | Hello, World!                        |                                             |

### 🪲 Bug in Windows Narrator

https://user-images.githubusercontent.com/1622400/174435618-d51c9b0b-5855-4b7d-bc7b-4ecc7c1b17f5.mp4

Transcript:

- Bot said, do you like our service? Yes no.
- Article.
- Bot said, do you like our service? Yes button, no button.
- You said, yes.
- Article.
- You said, yes.

After the content is appended to a live region, Windows Narrator narrated the content twice, causing confusions.

However, when we press <KBD>CAPSLOCK</kbd> + <kbd>CTRL</kbd> + <kbd>I</kbd> to narrate the whole page, it did not repeat the content.

https://user-images.githubusercontent.com/1622400/174435816-2a9336ba-c8b4-4c1a-b26a-9c42fea9488f.mp4

Transcript:

- Bot said, do you like our service?
- Button, yes.
- Button, no.
- You said, yes.

### Workaround

We need to simplify the DOM tree for live region as much as possible and refrain from using `aria-labelledby`. We should use the following template:

```html
<article id="id-00001">Hello, World!</article>
```

On Windows Narrator, it will now correctly narrate as "Article. Hello, World!"

https://user-images.githubusercontent.com/1622400/174435972-ced75022-c525-41fa-8b9b-a82205e1fa38.mp4

Transcript:

- Article.
- Bot said, do you like our service?
- Yes, button.
- No, button.
- Article.
- You said, yes.

## `<ul>`/`<li>` vs. `role="list"`/`role="listitem"`

While we can use `<ul>` with `list-style-type: none;` to represents an accessible list but not visually a list, we need to be careful about its content.

For the following HTML:

```html
<ul style="list-style-type: none; padding: 0;">
  <li>
    <ul>
      <li>This will be level 2, regardless of list-style-type set earlier.</li>
    </ul>
  </li>
</ul>
```

The inner `<li>` will always be of level 2 and displayed as a hollow dot.

When we need a list that is not visually appears as a list, use `role="list"`/`role="listitem"` instead.
