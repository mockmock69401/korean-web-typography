---
name: korean-web-typography
description: Use when building or restyling any web UI that displays Korean text, or writing Korean copy for one - including when another design skill also sets fonts or a type scale. Covers the font stack, font sizes and small-text floors, heading line height, monospace, word-break keep-all, line spacing, em and en dashes in Korean body text, and when to use cards and tables and how to map text sizes to the h1-h6 hierarchy. Applies to every web project, not one codebase.
---

# Korean web typography

Latin-tuned defaults are wrong for Korean in ways that are easy to miss, because the page still
renders and nothing errors. These are the corrections. They apply to any web project that shows
Korean, including ones whose UI is mostly English but whose *data* is Korean — file paths, log
lines, user content.

**Announce at start:** "I'm using the korean-web-typography skill for the type rules."

**This skill wins on Korean text.** When another skill, design system or style guide also sets the
font, size, line breaking or line spacing of text that is or contains Korean, follow this skill for
those properties. Design skills commonly say "avoid Inter, use Geist" or ship a type scale tuned for
Latin; that advice was written for Latin text. The same goes for when to use cards and tables and
for the h1–h6 type scale (section 8). Keep their other rules (color, layout, motion).

**Before writing any CSS, ask the user whether large text is the priority** — see section 7. The
answer sets the size floor for the whole page.

## 1. No monospace. Ever.

Delete the monospace token from the theme so the utility is not reachable at all — **and override
the preflight rule, because the token alone does not cover it.**

```css
@theme {
  --font-mono: initial;   /* Tailwind v4: removes the font-mono utility */
}

@layer base {
  /* Preflight styles code/kbd/samp/pre from --default-mono-font-family, a *different* variable
     with a hardcoded fallback stack, so clearing --font-mono leaves them monospaced. Verified in
     Tailwind v4's compiled output, which emits:
       code,kbd,samp,pre{font-family:var(--default-mono-font-family, ui-monospace, SFMono-Regular,
       Menlo, Monaco, Consolas, "Liberation Mono", "Courier New", monospace)} */
  code, kbd, samp, pre { font-family: inherit; }
}
```

Check the compiled CSS rather than assuming either rule worked. The override wins on source order
within the same cascade layer, not on specificity, so it must come after preflight — which it does
when written in `@layer base` in your own stylesheet, but is worth re-checking if a Tailwind major
reorganises preflight.

**Plain CSS, no Tailwind.** There is no token to delete, but the browser's own stylesheet sets
`code`, `kbd`, `samp` and `pre` to `monospace`, so the override is needed just as much. Put it in
your stylesheet and make sure no rule names a monospace family:

```css
code, kbd, samp, pre { font-family: inherit; }
```

**Why.** No monospace family in common use ships Hangul. `SFMono-Regular`, `Menlo`, `Consolas`,
`Geist Mono`, `JetBrains Mono` — all of them fall back to whatever the OS picks for Hangul, which
on Windows is 맑은 고딕 or 굴림. The result is a single line rendered in two unrelated faces with
different metrics, weights and vertical alignment. It looks worse than a proportional font would,
and it defeats the only reason monospace was chosen: nothing lines up, because half the line is
not monospaced.

**What people actually wanted** when they reached for monospace is usually column alignment of
digits. Get that without changing the font:

```css
font-variant-numeric: tabular-nums;   /* Tailwind: tabular-nums */
```

That fixes digit widths in a proportional font, which is the whole benefit, and leaves Korean
rendering in the real UI face.

Apply `tabular-nums` **only where numbers are compared or animate in place** — a progress counter,
a duration, a table of sizes, a percentage that ticks. Not to body text, not to headings, not
"just in case". Elsewhere it makes digits slightly too wide for the surrounding text.

Log panes, file paths, terminal-ish output and code-adjacent UI are the usual places someone
reaches for monospace out of habit. For paths and logs that contain Korean, use the UI font.
Genuine source code in a code editor or a syntax-highlighted block is the one place monospace
still earns its keep — and even there, only if the content is actually code rather than log text.

## 2. `word-break: keep-all` as the default

```css
:root, body { word-break: keep-all; overflow-wrap: anywhere; }
```

**Why.** The CSS default (`normal`) breaks CJK at *any* character, so Korean wraps mid-word:
`파이프라`/`인 실행`. `keep-all` makes the line break at spaces, keeping each 어절 whole, which is
how Korean is meant to wrap.

**Always pair it with `overflow-wrap: anywhere`.** `keep-all` alone will overflow its container on
a long unbreakable token — a URL, a Windows path, a hash. `anywhere` lets those break while
ordinary Korean still wraps at 어절 boundaries. Without the pair you trade mid-word breaks for
horizontal scrollbars.

## 3. Do not justify Korean

Never `text-align: justify` on Korean body text. Latin justification works because short words give
the engine many small gaps to distribute. Korean 어절 are long, so the same algorithm produces a few
enormous gaps per line — rivers of whitespace that are markedly harder to read than a ragged right
edge. Left-align.

## 4. Line spacing: the order of operations

The governing principle: **the vertical gap between lines must clearly exceed the horizontal gaps
within a line.** If they are comparable, the eye groups glyphs vertically and the reader loses the
line. Korean makes this easy to get wrong because Hangul syllables are dense, evenly-weighted
squares with no ascenders or descenders to visually separate the rows.

When a block of Korean looks cramped or the lines swim, work in this order and stop as soon as it
reads well:

1. **Tighten `letter-spacing` first.** Fonts hinted for Latin leave Hangul looking loose.
   `-0.01em` to `-0.02em` is the usable range; past `-0.03em` syllables start to collide.
2. **Then tighten `word-spacing`**, if 어절 still float apart. Small negative values only.
3. **Only then widen `line-height`.** Reach for this last, in small steps.

Doing it in the other order — widening leading first — makes the block taller and looser without
fixing the actual problem, and you end up with airy text that still reads badly.

Sensible starting values for Korean body text:

```css
line-height: 1.6;        /* vs ~1.5 for Latin */
letter-spacing: -0.01em;
```

Long-form reading text can go to `line-height: 1.7`–`1.8`; dense UI can sit at `1.5`.

### Headings

**Scope.** These rules apply to every `h1`–`h6` and to any text styled as a heading: section titles,
card titles, footer column titles. Size does not change that. A 15px footer title is still a heading
and follows the line-height rule below, not the body values above.

**Tracking.** Tighter than body, not looser: `-0.02em` to `-0.03em`. The `-0.03em` collision limit
applies to headings too, at every size. Large display sizes do not earn an exception.

**Line-height depends on whether the heading wraps**, at the width where it is rendered:

- **Fits on one line → `line-height: 1.15`.** With no second line there is no inter-line gap to
  protect, and extra leading only pushes the heading away from the content it labels.
- **Wraps to two or more lines → derive it from the word-space width.** The governing principle
  above says the gap between lines must clearly exceed the horizontal gaps within a line, and in a
  heading the widest horizontal gap is the space between 어절. Its width depends on the font, the
  size, the weight and the `letter-spacing`/`word-spacing` you just set, so measure it rather than
  guess. Hangul syllables fill nearly the whole em box, so the visible gap between two lines is
  roughly `(line-height − 1) × font-size`. Start from a gap of 1.5× the space width, then look at
  the rendered heading and adjust:

  ```js
  // Browser console: measure the rendered word space of a heading, suggest a line-height.
  const h = document.querySelector('h1');          // the heading to tune
  const s = getComputedStyle(h);
  const probe = document.createElement('span');
  Object.assign(probe.style, {
    fontFamily: s.fontFamily, fontSize: s.fontSize, fontWeight: s.fontWeight,
    letterSpacing: s.letterSpacing, wordSpacing: s.wordSpacing,
    whiteSpace: 'pre', position: 'absolute', visibility: 'hidden',
  });
  document.body.append(probe);
  probe.textContent = '가 가'; const withSpace = probe.getBoundingClientRect().width;
  probe.textContent = '가가';  const noSpace   = probe.getBoundingClientRect().width;
  probe.remove();
  const ratio = (withSpace - noSpace) / parseFloat(s.fontSize);   // space width in em
  console.log({ spaceEm: ratio.toFixed(3), lineHeight: (1 + 1.5 * ratio).toFixed(2) });
  ```

- **Judge wrapping per breakpoint, by looking.** A heading that is one line on desktop often wraps
  at 375px. For fixed copy you wrote, render the page at each breakpoint and count the lines: `1.15`
  where it is one line, the derived value inside the media query where it wraps. Do not assume it
  might wrap; check.
- **Dynamic text only: assume wrapping.** When the heading's content comes from data or the user
  (a city name, a product title, a search term), you cannot see every value, so use the derived
  value at every width.

**Balance the lines.** Put `text-wrap: balance` on headings and `text-wrap: pretty` on body
paragraphs. `keep-all` moves whole 어절, so a heading easily ends with one short 어절 alone on the
last line (`것`, `요`). Browsers that do not support these values ignore them.

## 5. Font stack — pick by how many languages you actually need

The choice turns on one question: **is the text Korean (plus Latin), or is it genuinely
multilingual CJK?** Getting this wrong is not a style mistake; it produces holes in the rendering.

Ask it about the text you *author* — labels, copy, headings, error messages — not about every
string the app might ever display. An app whose entire UI is Korean but which shows arbitrary
filenames, user-pasted text or third-party titles will meet Japanese or Chinese eventually, and
that alone does not make it a multilingual app. Incidental foreign text in data is a reasonable
thing to leave to the OS fallback: it stays legible, it is rare, and it costs nothing. Reach for a
pan-CJK font when those languages are part of what the product is *for* — a dictionary, a
translation tool, a subtitle editor, anything a Japanese or Chinese speaker is meant to use.

Weigh it honestly rather than defensively. Noto Sans CJK unsubsetted is tens of megabytes, and
carrying that so a filename renders in one face instead of two is a poor trade for a tool whose
users are all Korean.

### Korean and Latin only → Pretendard

```bash
npm install pretendard
```

```css
@import "pretendard/dist/web/variable/pretendardvariable.css";

@theme {
  --font-sans: 'Pretendard Variable', system-ui, sans-serif;
  --font-mono: initial;
}
```

With a bundler but no Tailwind, set the same stack on the page yourself:

```css
@import "pretendard/dist/web/variable/pretendardvariable.css";

:root { --font-sans: 'Pretendard Variable', system-ui, sans-serif; }
body  { font-family: var(--font-sans); }
```

It covers Hangul, Latin and the numerals with one consistent set of metrics, so a mixed
Korean/English line does not change face mid-sentence. For Korean UI this is the right default.

**One family for everything, digits and Latin included.** Do not give a Latin-only face (Inter,
Geist, SF Pro, Roboto, a display serif) to numbers, prices, dates, codes or English words inside a
Korean UI. It is the defect from rule 1 without the monospace: the row switches face mid-line, with
a different x-height, weight and baseline next to the Hangul. Pretendard's Latin and numerals are
Inter-derived and already drawn to sit with its Hangul. For column alignment use `tabular-nums`,
not a second font. This holds even when another skill recommends a Latin face.

The package also ships `pretendardvariable-dynamic-subset.css`, which splits the range into
unicode-range chunks fetched on demand. Prefer it for a public website, where the full file is a
real download. Prefer the single variable file for a bundled desktop app, where everything is local
and one asset is simpler than a hundred.

**No bundler → load it from the CDN.** For a plain HTML page with no build step, link the
stylesheet in `<head>`:

```html
<link rel="stylesheet" as="style" crossorigin href="https://cdn.jsdelivr.net/gh/orioncactus/pretendard@v1.3.9/dist/web/static/pretendard.min.css" />
```

or import it from CSS — at the very top of the stylesheet, since an `@import` that follows other
rules is silently ignored:

```css
@import url("https://cdn.jsdelivr.net/gh/orioncactus/pretendard@v1.3.9/dist/web/static/pretendard.min.css");
```

This is the **static** build, and it declares the family as `Pretendard` (weights 100–900), not
`Pretendard Variable`. Copying the npm snippet's `font-family` next to the CDN link names a face
that was never loaded, and the page falls back to the OS font with no error:

```css
font-family: Pretendard, system-ui, sans-serif;
```

**Pretendard runs small — size for legibility.** At the same `font-size`, Pretendard renders
slightly smaller than other Korean sans faces (맑은 고딕, Apple SD Gothic Neo, Noto Sans KR). Sizes
carried over from a design tuned for another font therefore come out smaller than intended. Don't
copy the px values across when switching fonts; look at the rendered text and raise sizes where it
stops reading comfortably. Small text suffers first — captions, labels, table cells, secondary
metadata — so check those before body copy, against the floor in section 7.

### Japanese or Chinese in the mix → Noto Sans CJK KR

**Pretendard is a Korean font, and its CJK coverage is incidental.** This is easy to miss because
it does contain *some* kana and hanja — the ones that appear inside Korean text, in loanwords and
citations — so a casual look suggests it handles Japanese. It does not. Measured against
`pretendard` 1.3.9's own declared unicode ranges:

- Missing `と` `で` `に` `ひ` `ふ` `ほ` `ゲ` `ソ` — among the most common kana in Japanese. The
  hiragana run is literally `…3066, 3069…`, skipping 3067 and 3068.
- 388 Han codepoints in total. Japanese needs ~2,136 jōyō kanji as a floor; Chinese needs several
  thousand.

Partial coverage is **worse than none**. A missing font falls back cleanly for the whole string; a
font with holes renders some glyphs in Pretendard and the rest in the OS fallback, switching face
mid-word. That is the exact defect rule 1 exists to prevent, arriving through the front door.

The Pretendard project's answer is separate fonts per language — `pretendard`, `pretendard-jp`,
`pretendard-std`, `pretendard-gov` are four distinct npm packages. Loading two of them and
switching by language gives up the single-metric consistency that was the reason to choose
Pretendard in the first place.

For genuinely multilingual work use **Noto Sans CJK KR**: one family covering Korean, Japanese and
Chinese, where the `KR` suffix selects which regional glyph forms win for the Han characters the
three languages share — the same codepoint is drawn differently in Korean, Japanese and Chinese
convention, and the regional variant is how you choose.

**Do not confuse `Noto Sans KR` with `Noto Sans CJK KR`.** They are different fonts and the names
are nearly identical:

- `Noto Sans KR` — Korean only. This is what `@fontsource/noto-sans-kr` and Google Fonts ship. It
  has the same gap as Pretendard for Japanese and Chinese.
- `Noto Sans CJK KR` — the pan-CJK superfamily. Distributed by Google/Adobe as OTF/OTC rather than
  through fontsource; self-host the `.otf`, or use the Source Han Sans build, which is the same
  font under Adobe's name.

Because it is pan-CJK, it is a large file — tens of megabytes unsubsetted. Subset it to the ranges
you actually serve, or accept the weight in a bundled desktop app where it loads from disk.

### No dependency available

`system-ui` alone is tolerable on Korean Windows (맑은 고딕) and macOS (Apple SD Gothic Neo), but the
two differ enough that a design reviewed on one will look off on the other. It is a fallback, not
a choice.

## 6. Go easy on em and en dashes in Korean body text

The em dash `—` and the en dash `–` are rare in Korean prose. Used to set off an aside or to join
two clauses, they make body text read as translated or machine-written copy. Keep them to a
minimum: when a sentence reaches for one, first try Korean punctuation and structure instead — a
comma, a colon after a noun phrase, parentheses, or a second sentence.

| Instead of | Try |
|---|---|
| `저장했어요 — 다시 열면 그대로 복원돼요.` | `저장했어요. 다시 열면 그대로 복원돼요.` |
| `세 가지 방법 – 복사, 이동, 삭제` | `세 가지 방법: 복사, 이동, 삭제` |
| `기본 글꼴 — Pretendard — 을 씁니다.` | `기본 글꼴(Pretendard)을 씁니다.` |
| `3–5일` | `3~5일` |

For ranges, the tilde `~` is the usual Korean mark.

**A short title is where a dash sits comfortably.** A heading may use one for simple emphasis —
`Pretendard — 한국어 UI의 기본 글꼴` — because it stands alone and is read at a glance. In body
text, captions, labels, button text, error messages, alt text and descriptions, reach for the
alternatives above first.

This rule covers `—` and `–` only. The hyphen-minus `-` is outside it.

## 7. Small-text size floor: ask first

The smallest text on the page is where Korean legibility fails first: captions, labels, helper and
error text, table and calendar cells, badges, price footnotes, footer legal text. It is also what
gets shrunk again inside a mobile media query after the body was sized with care. Pick a floor and
hold it at every breakpoint.

| Context | Small-text range |
|---|---|
| Large text is the priority (readability first) | 14–16px |
| Otherwise | 12–13px |

Nothing on the page goes below the chosen range's lower bound. That includes the mobile media
query, digit-only text (prices, dates) and text squeezed into dense components like calendars. If a
component cannot fit its text at the floor, change the component (fewer columns, shorter wording,
stacking), not the size.

**Ask before building.** Which row applies is the user's decision, not an inference from the brief.
Before writing CSS, ask once:

> 큰 글씨(가독성) 우선인가요? 우선이면 작은 글씨를 14~16px, 아니면 12~13px로 맞춥니다.

Skip the question when the request or the project already answers it: a design spec with sizes, or
the user saying the audience is older readers or that text should be large. If you cannot ask
(running as a subagent, or non-interactive with no answer given), use 14–16px and say so in your
report.

## 8. Structure by hierarchy: cards, tables and type sizes

Cards and tables are the two components most often added for looks. A box around every group and a
grid around every list flatten the page: everything gets the same visual weight, and the heading
hierarchy readers scan by is lost. Use them only when the content needs them.

**Card: only when it acts as a button.** A card is a surface the user clicks or taps as a whole: a
product to open, a plan to choose, a project to enter. Build it as one link or button (`<a>` or
`<button>`) with hover, focus and pressed states, and put no other link or button inside it. HTML
does not allow interactive content inside `<a>` or `<button>`, and a second target makes it unclear
what a click does. If nothing happens when the surface is clicked, or if it holds its own buttons,
it is not a card. Show it as a section: a heading, the content under it, and spacing or a divider
between groups.

**Table: only when comparing data is the point.** A table is for reading across rows and down
columns to compare values: plans against features, versions against sizes, months against figures.
One record's details, a list of items or a set of label and value pairs is not a comparison. Use a
heading with a list, or a definition list (`dl`).

**Everything else: avoid both. Place content by hierarchy.** Order, spacing, headings,
lists and dividers carry the structure. Do not nest a card in a card, and do not wrap a table in a
card.

**Type sizes: only h1–h6, matched to hierarchy.**

- Define the `h1`–`h6` sizes once, as tokens, and use them for every piece of text that works as a
  heading, whatever its element (section titles, card titles, footer column titles).
- Pick the tier by the content's depth in the page, not by how big it should look. The page title is
  `h1`, its sections `h2`, their subsections `h3`, and so on. Do not skip a tier when a level in
  between exists.
- The same level gets the same size everywhere on the page.
- **If no tier fits, use the nearest one.** Do not add a one-off size, and do not nudge a tier with
  `font-size: 17px` or a `scale` utility to make it fit. A level deeper than `h6` uses `h6`. A text
  that seems to want a size between two tiers takes whichever tier is closer.
- Body text uses the body size. The smallest tier and the body size still sit above the floor from
  section 7.
- The heading rules in section 4 (tracking, line height, `text-wrap: balance`) apply to every tier.

## 9. Verify the rendered page before reporting

The checklist is self-reported, and intended values drift from rendered ones once media queries and
component styles pile up. Before calling the work done, open the page in a browser at desktop width
and again at about 375px, and run:

```js
// Browser console: smallest rendered text, which families it uses, and whether the font loaded.
const rows = [];
const walker = document.createTreeWalker(document.body, NodeFilter.SHOW_TEXT);
while (walker.nextNode()) {
  const el = walker.currentNode.parentElement;
  const text = walker.currentNode.textContent.trim();
  if (!text || el.closest('script, style, noscript, template')) continue;
  const cs = getComputedStyle(el);
  rows.push({ px: parseFloat(cs.fontSize), family: cs.fontFamily.split(',')[0], text: text.slice(0, 24) });
}
rows.sort((a, b) => a.px - b.px);
console.table(rows.slice(0, 10));
console.log('families:', [...new Set(rows.map(r => r.family))]);
console.log('sizes:', [...new Set(rows.map(r => r.px))].sort((a, b) => b - a));
const first = getComputedStyle(document.body).fontFamily.split(',')[0].trim().replace(/["']/g, '');
console.log(first, 'loaded:', [...document.fonts].some(f => f.family.replace(/["']/g, '') === first && f.status === 'loaded'));
```

Pass when:

- the smallest `px` is at or above the floor chosen in section 7, at both widths;
- every value in `sizes` is one of the h1–h6 tiers or the body size at that width (section 8);
- `families` holds one UI family (genuine code blocks excepted), with no Latin-only or monospace face;
- the body font reports `loaded: true`. `false` usually means the `font-family` name does not match
  the face the stylesheet declares (section 5).

Report the measured minimum size, not the intended one. Without a browser, search the CSS for every
`font-size`, including inside media queries, and report the lowest.

## Checklist

- [ ] Asked whether large text is the priority before writing CSS (or found the answer in the
      request or spec)
- [ ] `--font-mono` removed from the theme; no `font-mono` utility anywhere; `code, kbd, samp, pre`
      inherit the UI font (plain CSS too)
- [ ] One font family for all text; no Latin-only face on digits, codes or English words
- [ ] `tabular-nums` only on compared or animating numbers
- [ ] `word-break: keep-all` **and** `overflow-wrap: anywhere` set globally
- [ ] Nothing justified
- [ ] `letter-spacing` tightened before `line-height` widened
- [ ] Font chosen by language scope: Pretendard for Korean + Latin, Noto Sans **CJK** KR if
      Japanese or Chinese is genuinely in scope — and not `Noto Sans KR`, which is Korean only
- [ ] Font set as `--font-sans`, loaded from the npm package in a bundled project or from the CDN
      link on a plain HTML page — with the family name that CSS declares (`Pretendard Variable`
      for the npm variable file, `Pretendard` for the CDN static file)
- [ ] Sizes checked for legibility in Pretendard itself, not carried over from another font
- [ ] Smallest text within the chosen floor at every breakpoint: 14–16px when large text is the
      priority, 12–13px otherwise
- [ ] Every heading (any size, including footer titles) tracked `-0.02em`–`-0.03em`;
      `line-height: 1.15` where the rendered heading is one line at that breakpoint, derived from the
      word-space width where it wraps or where its text is dynamic; `text-wrap: balance`
- [ ] `—` and `–` kept to a minimum in Korean body text, alternatives tried first (a short title
      may use one for emphasis); ranges written with `~`
- [ ] Cards only where the whole surface is a button; tables only where comparing data is the point;
      everywhere else content placed by hierarchy, no card or table
- [ ] Text sizes come only from the h1–h6 tiers and the body size, chosen by hierarchy depth; no
      one-off sizes, and a missing tier is answered with the nearest tier
- [ ] Rendered page checked at desktop and ~375px with the section 9 script; measured minimum size
      reported
