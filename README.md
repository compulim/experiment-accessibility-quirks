# Accessibility Quirks

## Table of content

- [Live region additions](#live-region-additions)

## Live region additions

Tested on Edge + Windows Narrator.

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

When an element is appended after 3, Windows Narrator will narrate it.

However, when an element is inserted before 3, Windows Narrator will ignore it.

## Live region with same content

Tested on Edge + Windows Narrator.

In a live region, the elements are "ABC", "DEF" and "XYZ". Then, after 2 seconds, remove all of them. Then, after a few more seconds, add back "ABC", "DEF", and "XYZ".

Narrator fails recognize the first "ABC" is a new one. And at the time the elements is reappear in the DOM tree, it only narrate "DEF" and "XYZ".

In our case, we are using `aria-labelledby` to point to the narrative content of "ABC", "DEF", and "XYZ".
