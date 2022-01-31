# Accessibility Quirks

## Table of content

- [Live region additions](#live-region-additions)

## Live region additions

Tested on Edge + Windows Narrator.

Live region with `aria-relevant="additions"` will only look at the last element in the container.

### Tests

Given:

```html
<section role="liveregion" aria-relevant="additions">
  <article>1</article>
  <article>2</article>
  <article>3</article>
</section>
```

When an element is appended after 3, Windows Narrator will narrate.

However, when an element is inserted before 3, Windows Narrator will ignore it.
