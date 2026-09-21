---
name: korean-web-typography
description: Use when building or restyling any web UI that displays Korean text, or writing Korean copy for one - sets the font stack, forbids monospace, applies word-break keep-all, gives the order of operations for line spacing, and keeps dashes out of Korean body text. Applies to every web project, not one codebase.
---

# Korean web typography

Latin-tuned defaults are wrong for Korean in ways that are easy to miss, because the page still
renders and nothing errors. These are the corrections. They apply to any web project that shows
Korean, including ones whose UI is mostly English but whose *data* is Korean — file paths, log
lines, user content.

**Announce at start:** "I'm using the korean-web-typography skill for the type rules."

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

Headings in Korean want tighter tracking than body, not looser: `-0.02em` is reasonable at large
sizes. Long-form reading text can go to `line-height: 1.7`–`1.8`; dense UI can sit at `1.5`.

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

It covers Hangul, Latin and the numerals with one consistent set of metrics, so a mixed
Korean/English line does not change face mid-sentence. For Korean UI this is the right default.

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
metadata — so check those before body copy.

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

## 6. No dashes in Korean body text

The em dash `—`, the en dash `–`, and a hyphen or minus sign typed as a dash (` - `) are English
punctuation. Korean prose does not use them to set off an aside or to join two clauses, so in body
text they read as translated or machine-written copy. Don't swap one dash for another; rewrite the
sentence with Korean punctuation and structure — a comma, a colon after a noun phrase,
parentheses, or a second sentence.

| Don't | Do |
|---|---|
| `저장했어요 — 다시 열면 그대로 복원돼요.` | `저장했어요. 다시 열면 그대로 복원돼요.` |
| `세 가지 방법 – 복사, 이동, 삭제` | `세 가지 방법: 복사, 이동, 삭제` |
| `기본 글꼴 - Pretendard - 을 씁니다.` | `기본 글꼴(Pretendard)을 씁니다.` |
| `3-5일`, `3–5일` | `3~5일` |
| `기온이 -3도까지 내려가요.` | `기온이 영하 3도까지 내려가요.` |
| `매출 -12%` in a sentence | `매출 12% 감소` |

Ranges take the tilde `~`, the usual Korean range mark. A negative value in running text reads
better as words: 영하, 감소, 줄어든.

**The one exception is a short title.** A heading may use a dash for simple emphasis —
`Pretendard — 한국어 UI의 기본 글꼴` — because it stands alone and is read at a glance. The
exception ends at the heading. Body text, captions, labels, button text, error messages, alt text
and descriptions stay dash-free.

**Not punctuation, not affected:** code and CSS values (`-0.01em`, `keep-all`), identifiers and file
names, and numbers in tables, charts and data cells, where the minus sign is part of the value.

## Checklist

- [ ] `--font-mono` removed from the theme; no `font-mono` utility anywhere
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
- [ ] No `—`, `–` or ` - ` dash in Korean body text (a short title may use one for emphasis);
      ranges written with `~`, negative values in prose written as words
