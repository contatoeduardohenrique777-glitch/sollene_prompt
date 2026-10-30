# Sollene: Creative Hero with Liquid Background (rebuild command)

Build exactly the page described below. It is ONE full-screen hero section in a single `index.html` file: no framework, no build step, no extra libraries. Every value here is measured; do not round, restyle or "improve" anything. Appendix A holds the complete reference file. When prose and Appendix A disagree, Appendix A wins: write the file so it is byte-identical to Appendix A (UTF-8, LF line endings, exactly the lines between the fences plus one final newline: 35,492 bytes). In the prose, `Nk` is shorthand for `calc(N * var(--k))`.

**Assets (Path A, hosted on Cloudflare R2).** Use the URL below exactly. Do not download, re-encode, rename or hotlink any other image.

| Asset | URL | Size | Used by |
|---|---|---|---|
| Background photo | `https://assets.layerweb.com.br/sollene/images/hero-visor.webp` | 3200x1000 WebP, 148,870 bytes | `<img>` fallback + WebGL2 texture (same element) |

Fonts come from Google Fonts with these exact families and weights: Inter Tight 200, 300, 500; Instrument Serif italic 400; Inter 400, 500 (`display=swap`).

---

## 0. Global shell

- `<html lang="en">`, `<title>Sollene | Expression Built for Souls</title>`, favicon = the brand icon from section 4 as an SVG data URI.
- Reset `*{box-sizing:border-box;margin:0;padding:0}`. `html` and `body` background `#0a0201`, body text `#fff`, font Inter, `-webkit-font-smoothing:antialiased`, `overflow-x:hidden`.
- A first inline script in `<body>` adds the class `js` to `<html>`. A `<noscript>` style makes `.js .a,.js .bg` visible again.
- Preload the background with `<link rel="preload" as="image" crossorigin="anonymous">`. The `<img>` also has `crossorigin="anonymous"` (it is uploaded to WebGL). Never load this URL without CORS mode anywhere on the page.

## 1. Design canvas and layering

- Hero: `height:100svh` (fallback `100vh`), `min-height:620px`, `overflow:hidden`, `isolation:isolate`.
- **Content width limit 1440 px.** Only `.bg` (photo + liquid canvas), `.shade` and `.dust` are full-bleed. Everything else (vertical lines, rings, ghost word and all of `.ui`) sits inside `.frame`: `position:absolute; top:0; bottom:0; left:0; right:0; margin:0 auto; max-width:1440px; z-index:2`. Below 1100 px `.frame` becomes `position:relative; flex:1; display:flex; flex-direction:column; width:100%`.
- Reference canvas **1440 x 1060**. Scale unit `--k: min(calc(min(100vw, 1440px) / 1440), calc(100svh / 1060))`, so k never exceeds 1. Every desktop length is written `calc(N * var(--k))` where N is the canvas px value.
- Anchoring (relative to `.frame`): left-column items are positioned from the left, right-column items from the right, header items from the top, and the stats, ghost word, partner label, partner row and ring 2 from the bottom. The nav is centered with `left:0;right:0;margin:auto;width:max-content`.
- Small texts use a floor: `max(calc(N * var(--k)), Mpx)` with the M values from Appendix A (9, 10 or 11 px).
- Stacking order (z-index): `.bg` 0 → `.shade` 1 → `.dust` 1 (after the shade in the DOM) → `.frame` 2, which contains `.vline` and `.ring` 2 → `.ghost` 3 → `.ui` 4 (all text and controls). **The liquid effect lives only in `.bg`.** Text is never drawn into the canvas and is never distorted.

### 1.5 Absolute coordinate table (canvas px, desktop ≥ 1100 px)

| Element | Anchor | x / right | y / bottom | w x h | z |
|---|---|---|---|---|---|
| Logo (icon 48 + wordmark 31px) | left/top | left 22 | top 34 | icon 48x48, gap 12 | 4 |
| Nav pill | center/top | centered | top 32 | auto x 50 (padding 5/8) | 4 |
| Book a Call button | right/top | right 23 | top 30 | 123 x 44 | 4 |
| Badge (globe 28 + 1px divider 26 tall + 2-line text) | left/top | left 21 | top 206 | auto, gap 10 | 4 |
| h1 (3 lines) | left/top | left 17 | top 271 | width 6em, font 88 / line-height 1.045 | 4 |
| Sub paragraph (2 lines) | left/top | left 19 | top 562 | 16px / 1.69 | 4 |
| CTA pill | left/top | left 19 | top 652 | 199 x 64, circle 54 inset 5 | 4 |
| Clients (3 faces 38, overlap -12, text) | left/top | left 244 | top 665 | auto, gap 10 | 4 |
| Stat card 1 | left/bottom | left 17 | bottom 161 | 203 x 134, radius 14 | 4 |
| Stat card 2 | left/bottom | left 234 | bottom 161 | 203 x 134, radius 14 | 4 |
| Ghost word "MUSE" | left/bottom | left 8 | bottom -51 | Inter Tight 200, 230px | 3 |
| Bars (6) | right/top | right 191 | top 258 | height 133, bar 18 wide, gap 14.6 | 4 |
| Metric text | left = 100% - 154 | — | top 288 | 12px / 1.3 | 4 |
| Impact block | left = 100% - 346 | — | top 438 | width 300 | 4 |
| Ring 1 | right/top | right 218 | top 577 | 180 x 180 | 2 |
| Ring 2 | right/bottom | right 69 | bottom 74 | 128 x 128 | 2 |
| "Partner Labs" | right/bottom | right 43 | bottom 144 | 13px | 4 |
| Partner row (5 marks) | right/bottom | right -30 | bottom 66 | gap 40, mark 18px | 4 |
| Vertical lines | top 96 → bottom 0 | left 288, left 584, right 572, right 282 | — | 1px | 2 |

## 2. Background, shade and liquid cursor (PIXEL CONTRACT)

- `.bg` holds the `<img>` (object-fit cover, `object-position:60.4% 50%`; 55% below 1100 px; 58% at ≤ 600 px) and a `<canvas>`. When WebGL2 works the class `gl` goes on `<html>`, the canvas is shown and the img is `visibility:hidden`. Without WebGL2 the img stays visible and the page is complete without the effect.
- The WebGL2 shader reproduces the same cover framing: if canvasAspect < imageAspect, `uScale=(A/I,1)` and `uStart=(px*(1-A/I),0)` with px = .604 (≥ 1100 px), .55 (601–1099 px), .58 (≤ 600 px). Otherwise `uScale=(1,I/A)` and `uStart=(0,.5*(1-I/A))`.
- Canvas resolution: DPR capped at 1.75, and also capped so width x height ≤ 4.2 million px.
- **Ripple simulation (CPU height field):** one cell = 5 CSS px, grid capped at 360 x 260. Each step: `v = ((L+R+U+D) * .5 - prev) * 0.98`, with buffers swapped. The simulation runs at a fixed 60 steps/s (at most 3 steps per frame, dt capped at 50 ms).
- Packing into an RGBA8 texture (LINEAR, CLAMP): R = 128 + dx*480, G = 128 + dy*480, B = 128 + h*110 (clamped 0–255), where dx and dy are central differences.
- Fragment shader: `n=(rg-.5)*2`, `h=(b-.5)*2`, `off=n*0.018`. Sample the image at `uStart+(vUv+off)*uScale`. Add chromatic split `ca=off*uScale*.2` (R at +ca, B at −ca). Then `c *= 1+h*.12`, and add specular `pow(max(dot(normalize(vec3(-n*1.4,1)), normalize(vec3(-.35,.45,1))),0),60) * vec3(1,.72,.55) * .35`.
- Pointer: `pointermove` and `pointerdown` on `window` (so the effect works under the text too). Drops go every 7.5 px (1.5 cells) along the travelled segment, radius 3.2 cells, amplitude `(min(1,dist/40)*.55+.08)/sqrt(n)`, raised-cosine profile, subtracted from the height.
- Intro ring: 900 ms after init, one drop of radius 9 cells and amplitude .8 at (58% x, 30% y) on desktop, or (62%, 30%) below 1100 px.
- The loop stops when the max |height| < .002 and there has been no input for 1200 ms. Both buffers are then cleared and a clean frame is drawn. It restarts on the next input. Input is ignored while the hero is off-screen (IntersectionObserver).
- `prefers-reduced-motion: reduce`: draw once, attach no listeners, no intro ring.
- `.shade` (above the canvas, not distorted): `linear-gradient(90deg, rgba(8,1,0,.58) 0%, rgba(8,1,0,.3) 28%, rgba(8,1,0,0) 50%)`, `linear-gradient(0deg, rgba(8,1,0,.5) 0%, rgba(8,1,0,0) 32%)`, `linear-gradient(180deg, rgba(8,1,0,.35) 0%, rgba(8,1,0,0) 16%)`. Below 1100 px it is `linear-gradient(180deg, rgba(8,1,0,.7) 0%, rgba(8,1,0,.35) 38%, rgba(8,1,0,.1) 60%, rgba(8,1,0,.7) 100%)`.

DO NOT apply the distortion to any text, button, card or line. DO NOT use an SVG displacement filter or a CSS filter for the liquid. DO NOT use a float texture extension. DO NOT animate the ripples on their own: the only automatic ripple is the single intro ring. DO NOT swap the photo for a gradient or a solid colour.

## 3. Headline gradient (PIXEL CONTRACT, indispensable)

- `h1`: Inter Tight 300, `calc(88 * var(--k))`, line-height 1.045, letter-spacing -0.05em, width 6em.
- The three lines are literal: `Expression` / `Built for Souls` / `Not <em>Formulas</em>`. The `em` is Instrument Serif italic 400 at 1.02em, letter-spacing -0.02em.
- Each line is a mask `span.ln` (display block, overflow hidden, `padding:0 .2em .16em 0; margin-bottom:-.16em` so descenders and the italic are not cut). Inside it is a `span` that carries the gradient:
  `linear-gradient(112deg, #fff 0%, #fff 54%, #ffd5c2 66%, rgba(255,156,108,.62) 80%, rgba(255,110,60,.14) 100%)`, `background-size:100% 300%`, with `background-position` `0 0`, `0 50%`, `0 100%` on lines 1–3. Together the three lines read as ONE diagonal gradient across the whole block: white top-left, fading to peach and transparent orange at the end of "Souls" and across "Formulas". Use `background-clip:text` with `-webkit-` prefixes and `-webkit-text-fill-color:transparent`.

DO NOT use one horizontal gradient per line. DO NOT put `background-clip:text` on the h1 itself (the lines animate inside their masks). DO NOT make the headline plain white.

## 4. Brand

The icon is new and original: a crescent moon (sol + Selene) holding a small sun dot. It has one colour, `#ff4a22`, and renders at 48x48 canvas px.

```html
<svg viewBox="0 0 48 48" aria-hidden="true"><path fill="#ff4a22" d="M26.29 4.13A20 20 0 1 0 43.71 27.37A15 15 0 1 1 26.29 4.13Z"/><circle fill="#ff4a22" cx="32" cy="18" r="5"/></svg>
```

The wordmark `Sollene` is Inter Tight 500, 31px, letter-spacing -0.02em. The link has `aria-label="Sollene home"`, and the hero has `aria-label="Sollene creative studio"`.

DO NOT redraw, outline or add gradients to the icon. DO NOT change the spelling "Sollene".

## 5. Components (literal text and styles)

- **Nav** (`Services` + 10px chevron, `Our Approach`, `Story`, `Journal`, `Works`). Pill border 1px rgba(255,255,255,.14), radius 12, background rgba(20,6,3,.28), backdrop blur 10px, gap 10. Items are 40 tall with padding 0 15, radius 8, background rgba(255,255,255,.06), border rgba(255,255,255,.1), Inter 500 12.5px, `#f4ebe6`. Hover background is rgba(255,255,255,.14).
- **Book a Call**: white `#fff`, text `#1a0d08`, radius 7, Inter 500 13px. Hover background is `#ffe3d8`.
- **Badge** text: `Loved by creators from<br>all across the globe`, Inter 400 11px / 1.3, rgba(255,255,255,.8).
- **Sub**: `We design bold, emotive, and inclusive digital art <br>stories guided by the way real minds feel.`, rgba(255,255,255,.82). Keep the space before `<br>`.
- **CTA** `Start today`: see the pixel contract in section 5.5.
- **Clients**: three 38px original SVG avatars (orange `#ff7a3d→#b3200c`, plum `#5a1a40→#1c0a1a`, teal `#3fc4c9→#14606e`) with a 2px ring rgba(20,4,2,.9). Text `80K+ Loyal Viewers` (500, 11px) over `All Regions` (400, .92em, rgba(255,255,255,.5)).
- **Stat cards**: `240+` / `Campaigns launched` / mark `01` (glass: background rgba(255,255,255,.07), inset ring 1px rgba(255,255,255,.28)) and `96%` / `Partners who return` / mark `02` (red: background rgba(140,22,6,.62), inset ring rgba(255,120,80,.35)). Both have backdrop blur 14px, number Inter Tight 300 44px at -0.04em, `*` 24px top-right (14, 10), label 11px at bottom 18, mark 7px at right 16 / bottom 32.
- **Ghost** `MUSE`: transparent fill, `-webkit-text-stroke:1px rgba(255,255,255,.17)`, letter-spacing .02em.
- **Right column**: six bars at heights 30/42/55/68/84/100%, `linear-gradient(0deg, rgba(255,255,255,.03), rgba(255,255,255,.16))`, radius 3. Metric `+68%<br>Engagement<br>Growth Rate` at rgba(255,255,255,.45). Impact block: h2 `Lasting Brand Glow` 16px at .45 opacity, then p `Track the shift through honest, clear signals<br>moments that stay` 11.5px / 1.55 at .3 opacity, margin-top 12.
- **Rings**: SVG circles, r 49.5 in a 100 viewBox, `pathLength="100"`, stroke rgba(255,255,255,.14) 1px non-scaling. Ring 1 is rotated 170deg and draws to dashoffset 58. Ring 2 is rotated 180deg and draws to 62.
- **Partners**: `Partner Labs` (13px, .85 opacity). The row holds five fictional marks with inline SVG icons: `Northline` (double chevron), `kivora` (hollow diamond), `Orbit` (ring + dot), `Halcyon` (double wave), `Plume` (leaf). The text is Inter Tight 500 18px, gap 8, and the row fades out on the right with `mask-image:linear-gradient(270deg, transparent 0, #000 150k)`.
- **Vertical lines**: 1px, `linear-gradient(180deg, transparent, rgba(255,255,255,.1) 14%, rgba(255,255,255,.1) 86%, rgba(255,255,255,.03))`.

DO NOT add real company logos. DO NOT replace the avatars with photos. DO NOT change any literal text.

## 5.5 Main CTA (PIXEL CONTRACT)

- Markup: `<a class="cta" aria-label="Start today"><i aria-hidden="true"><span data-t="Start today">Start today</span></i><b aria-hidden="true"><svg/><svg/></b></a>`. The two arrows are identical, `M3 9h12M10 4l5 5-5 5`, with stroke-width 1.6.
- Base: 199 x 64k pill, `overflow:hidden; isolation:isolate`, gradient `linear-gradient(90deg,#ff5a2a,#e8230f)`, shadow `0 10k 30k rgba(232,35,15,.35)`, label padding-left 26k, Inter 500 14px, white circle 54k at right 5k / top 5k.
- `::before` (z -1) is a white fill: `clip-path:circle(27k at calc(100% - 32k) 50%)`, which sits exactly under the circle. On hover it grows to `circle(150% at ...)` in .75s cubic-bezier(.65,0,.25,1). Below 1100 px it uses 24px at `calc(100% - 29px)`.
- Label roll: the `i` is 1.2em tall with overflow hidden. The span's `::after` repeats `attr(data-t)` in `#1a0d08`. On hover the span moves `translate:0 -1.2em` (.6s, same easing) and turns `#1a0d08` instantly. On leave its colour returns to white over .3s after a .45s delay, so the label is never white on the white fill.
- Arrow swap: arrow 1 exits to `translate:260% 0` and arrow 2 enters from `-260%` (.55s). The circle turns `#e8230f` (.5s, delay .1s) and the strokes turn `#fff` (.4s, delay .1s). The hover shadow is `0 16k 40k rgba(232,35,15,.5)`, and `:active` is `scale:.97` (.4s cubic-bezier(.3,1.5,.5,1)).
- Magnetic pull (only with `(hover:hover)`, not with reduced motion): pointermove sets `--mx = (relX-.5)*12px` and `--my = (relY-.5)*10px`. The CTA has `translate:var(--mx,0) var(--my,0)` with .5s cubic-bezier(.2,.8,.2,1). Pointerleave removes both values.
- Entrance: the CTA uses `rise` with `animation-fill-mode:backwards` and `opacity:1` (so the static translate stays free for the magnetic pull). The circle pops `scale 0→1` over .8s cubic-bezier(.3,1.6,.5,1) at 1.75s.
- Shine: `::after` (z -1, left -40%, width 30%, `linear-gradient(100deg, transparent, rgba(255,255,255,.38), transparent)`) loops 6s from 2.3s. The keyframes are `0% translate -100%` and `18%–100% translate 560%`.

DO NOT animate `transform` on the CTA. DO NOT keep `fill-mode:both` on its entrance (it would block the magnetic translate). DO NOT let the label disappear on leave.

## 5.6 Background dust (PIXEL CONTRACT)

- A full-bleed 2D canvas `.dust` (z 1, after `.shade`, `pointer-events:none`). DPR is capped at 2. The count is `min(90, round(w*h/16000))`.
- Each particle: depth z ∈ [.3,1], radius r ∈ [.6,2], rising speed vy ∈ [6,20] px/s scaled by z, sway `sin(t*.0004*tw+ph)*sw` with sw ∈ [8,26] px and tw ∈ [.6,2.2]. It wraps from the top (y < -12) back to the bottom at a random x.
- Drawing uses `globalCompositeOperation:'lighter'` and a 32 px radial sprite (`rgba(255,214,186,1)` 0 → `rgba(255,170,120,.55)` .25 → `rgba(255,120,60,0)` 1), drawn at `r*6*z+2` px. Alpha = `edge * z * (.22 + .4 * twinkle)`, where edge fades the top 18% and the bottom 8%.
- Parallax: target `(.5 - clientX/innerWidth, .5 - clientY/innerHeight)` eased by .04 per frame, giving an offset of 16z px horizontally and 10z px vertically.
- The canvas fades in over 2.4s ease after a .6s delay once `ready` is set. The loop pauses while the hero is off-screen. With reduced motion it draws one still frame.

DO NOT draw the dust in the WebGL canvas or let the liquid bend it. DO NOT exceed 90 particles. DO NOT use bright white dots.

## 6. Entrance timeline

The class `ready` is added to `<html>` (inside a requestAnimationFrame) when the background image and `document.fonts.ready` have both resolved, or after 3000 ms, whichever comes first. Before that, `.js .a` and `.js .bg` have opacity 0 and `.js .vline` has `scale:1 0`. Animations use the individual `translate` / `scale` properties, never `transform`.

| Element | Keyframes | Duration / easing | Delay |
|---|---|---|---|
| `.bg` (photo + canvas) | opacity 0→1, scale 1.1→1, blur 18px + brightness .55 → none | 2.2s cubic-bezier(.22,.8,.24,1) | 0 |
| Vertical lines 1–4 | scale 1 0 → 1 1 (origin top) | 1.6s cubic-bezier(.65,0,.25,1) | .2 / .3 / .4 / .5s |
| Logo, nav, Book a Call (`down`) | opacity 0, translate 0 -14px, blur 6px → none | 1s cubic-bezier(.2,.8,.2,1) | .4 / .5 / .6s |
| Badge (`rise`) | opacity 0, translate 0 18px, blur 8px → none | 1s cubic-bezier(.2,.8,.2,1) | .7s |
| h1 lines (`line`) | translate 0 105%, opacity 0, blur 6px → none | 1.2s cubic-bezier(.16,1,.3,1) | .8 / .95 / 1.1s |
| Sub, CTA, clients (`rise`) | as badge | 1s | 1.3 / 1.45 / 1.55s |
| Bars | scale 1 0 → 1 1 (origin bottom) + opacity | .9s cubic-bezier(.2,.8,.2,1) | 1.5s + i*.07s |
| Ghost word (`fade`) | opacity 0, translate 0 40px → none | 1.8s | 1.6s |
| Stat cards (`card`) | opacity 0, translate 0 26px, scale .97, blur 10px → none | 1.1s | 1.65 / 1.75s |
| Metric / impact (`rise`) | as badge | 1s | 1.7 / 1.8s |
| Rings | stroke-dashoffset 100 → 58 / 62 | 2s cubic-bezier(.6,0,.2,1) | 1.8 / 2s |
| Partner label / marks | rise | 1s / .9s | 1.9s / 2s + i*.08s |
| Liquid intro ring | one drop on the visor | — | 900 ms after texture init |
| CTA circle pop / shine | see 5.5 | .8s / 6s loop | 1.75s / 2.3s |
| Dust canvas | opacity 0 → 1 | 2.4s ease | .6s |

## 7. Responsive

- **≥ 1100 px:** the canvas layout above. `.head`, `.row` and `.stats` are `display:contents`.
- **≤ 1099 px (tablet and phone, column in flow):**
  - Hero `height:auto; min-height:100svh`, flex column, padding 20px 24px 28px. `.ui` and the children of `.head`, `.row` and `.stats` become `position:relative` with every offset `auto`.
  - Header row: logo (icon 36, word 24px) left and Book a Call (40 tall, padding 0 18) right. The nav is hidden.
  - Badge margin-top `clamp(48px,9vh,96px)`. h1 `clamp(46px,9.2vw,84px)`, margin-top 18. Sub 15px / 1.6, max 30em. CTA row margin-top 26, gap 18, wraps; CTA 184x58 with a 48 circle.
  - Stats margin-top auto (pinned to the bottom), cards `min(203px, 50% - 6px)` x 124. Partner label margin-top 28; the row wraps with gap 26 and has no mask.
  - Bars, metric, impact and rings are hidden. Ghost size `clamp(130px,30vw,260px)` at bottom -0.22em. Only vertical lines 1 and 4 stay, at 33.33%. The ghost word stays absolute (z 3), so it sits behind the in-flow content (z 4), as intended.
- **≤ 600 px:** padding 16px 16px 24px, Book a Call 36 tall / 12px, badge margin-top 44, cards 112 tall with 34px numbers, partner gap 14px 22px, marks 15px.

## 8. Accessibility

- `prefers-reduced-motion: reduce` removes every entrance animation (everything is shown at once) and the liquid becomes a still image.
- The `<img>` has `alt="Man wearing a glowing visor, lit by a red halo"`. The canvas is `aria-hidden`, decorative SVGs are `aria-hidden`, and the nav is `aria-label="Main"`.
- If WebGL2 is missing or the texture fails, the `<img>` stays visible and nothing shifts.

## 9. Final self-check (run before answering)

1. The file is byte-identical to Appendix A (35,492 bytes). After load in a WebGL2 browser without reduced motion, `<html>` has the classes `js gl ready`.
2. At 1440x900 the h1 box sits at about x 14, y 230, w 448, h 235, the sub at about x 16, y 477, and the CTA at about x 16, y 554, 169 x 54. At 2560x1080 the logo starts at x 582 (the 1440 column starts at 560).
3. Exactly one image request (the R2 URL above, status 200 with `Access-Control-Allow-Origin`). Fonts load from Google. The console shows no errors and no 404.
4. All content stays inside a centred 1440 px column on wider screens, while the photo and dust stay full-bleed. Hovering the CTA fills it white from the arrow circle and rolls the label. Moving the pointer bends the photo only. Text, cards, buttons and lines stay sharp, and the ripples settle within about 2 s.
5. The headline shows the diagonal white → peach → transparent orange gradient across all three lines.
6. No real brand names appear. The logo is the crescent + dot icon with the word `Sollene`.

---

# Appendix A: complete index.html

```html
<!doctype html>
<html lang="en">
<head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>Sollene | Expression Built for Souls</title>
<link rel="icon" href="data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 48 48'%3E%3Cpath fill='%23ff4a22' d='M26.29 4.13A20 20 0 1 0 43.71 27.37A15 15 0 1 1 26.29 4.13Z'/%3E%3Ccircle fill='%23ff4a22' cx='32' cy='18' r='5'/%3E%3C/svg%3E">
<link rel="preconnect" href="https://fonts.googleapis.com">
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
<link href="https://fonts.googleapis.com/css2?family=Inter+Tight:wght@200;300;500&family=Instrument+Serif:ital@1&family=Inter:wght@400;500&display=swap" rel="stylesheet">
<link rel="preload" as="image" href="https://assets.layerweb.com.br/sollene/images/hero-visor.webp" crossorigin="anonymous">
<style>
/* ---------- Global shell ---------- */
*,*::before,*::after{box-sizing:border-box;margin:0;padding:0}
html{background:#0a0201}
body{background:#0a0201;color:#fff;font-family:Inter,system-ui,sans-serif;-webkit-font-smoothing:antialiased;overflow-x:hidden}
a{color:inherit;text-decoration:none}
svg{display:block}

/* ---------- Hero frame ---------- */
.hero{position:relative;height:100vh;height:100svh;min-height:620px;overflow:hidden;isolation:isolate;
  --k:min(calc(min(100vw, 1440px) / 1440),calc(100svh / 1060));--accent:#ff4a22}
.bg{position:absolute;inset:0;z-index:0;background:#0a0201}
.bg img,.bg canvas{position:absolute;inset:0;width:100%;height:100%}
.bg img{object-fit:cover;object-position:60.4% 50%}
.bg canvas{display:none}
.gl .bg canvas{display:block}
.dust{position:absolute;inset:0;z-index:1;width:100%;height:100%;pointer-events:none}
.frame{position:absolute;top:0;bottom:0;left:0;right:0;margin:0 auto;max-width:1440px;z-index:2;pointer-events:none}
.gl .bg img{visibility:hidden}
.shade{position:absolute;inset:0;z-index:1;pointer-events:none;
  background:linear-gradient(90deg,rgba(8,1,0,.58) 0%,rgba(8,1,0,.3) 28%,rgba(8,1,0,0) 50%),
             linear-gradient(0deg,rgba(8,1,0,.5) 0%,rgba(8,1,0,0) 32%),
             linear-gradient(180deg,rgba(8,1,0,.35) 0%,rgba(8,1,0,0) 16%)}
.vline{position:absolute;z-index:2;top:calc(96 * var(--k));bottom:0;width:1px;background:linear-gradient(180deg,rgba(255,255,255,0),rgba(255,255,255,.1) 14%,rgba(255,255,255,.1) 86%,rgba(255,255,255,.03));pointer-events:none;transform-origin:50% 0}
.vline.v1{left:calc(288 * var(--k))}
.vline.v2{left:calc(584 * var(--k))}
.vline.v3{right:calc(572 * var(--k))}
.vline.v4{right:calc(282 * var(--k))}
.ui{position:absolute;inset:0;z-index:4;pointer-events:none}
.ui a,.ui button{pointer-events:auto}

/* ---------- Header ---------- */
.logo{position:absolute;left:calc(22 * var(--k));top:calc(34 * var(--k));display:flex;align-items:center;gap:calc(12 * var(--k))}
.logo svg{width:calc(48 * var(--k));height:calc(48 * var(--k))}
.logo span{font:500 calc(31 * var(--k))/1 "Inter Tight",sans-serif;letter-spacing:-.02em}
.nav{position:absolute;top:calc(32 * var(--k));left:0;right:0;margin:auto;width:max-content;display:flex;gap:calc(10 * var(--k));padding:calc(5 * var(--k)) calc(8 * var(--k));
  border:1px solid rgba(255,255,255,.14);border-radius:calc(12 * var(--k));background:rgba(20,6,3,.28);backdrop-filter:blur(10px);-webkit-backdrop-filter:blur(10px)}
.nav a{display:flex;align-items:center;gap:calc(7 * var(--k));height:calc(40 * var(--k));padding:0 calc(15 * var(--k));border-radius:calc(8 * var(--k));
  background:rgba(255,255,255,.06);border:1px solid rgba(255,255,255,.1);font:500 max(calc(12.5 * var(--k)),10px)/1 Inter,sans-serif;color:#f4ebe6;transition:background .3s}
.nav a:hover{background:rgba(255,255,255,.14)}
.nav svg{width:calc(10 * var(--k));height:calc(10 * var(--k))}
.btn-top{position:absolute;right:calc(23 * var(--k));top:calc(30 * var(--k));width:calc(123 * var(--k));height:calc(44 * var(--k));display:grid;place-items:center;
  background:#fff;color:#1a0d08;border-radius:calc(7 * var(--k));font:500 max(calc(13 * var(--k)),10px)/1 Inter,sans-serif;transition:background .3s}
.btn-top:hover{background:#ffe3d8}

/* ---------- Left column ---------- */
.badge{position:absolute;left:calc(21 * var(--k));top:calc(206 * var(--k));display:flex;align-items:center;gap:calc(10 * var(--k))}
.badge svg{width:calc(28 * var(--k));height:calc(28 * var(--k))}
.badge i{width:1px;height:calc(26 * var(--k));background:rgba(255,255,255,.35)}
.badge p{font:400 max(calc(11 * var(--k)),9px)/1.3 Inter,sans-serif;color:rgba(255,255,255,.8)}
h1{position:absolute;left:calc(17 * var(--k));top:calc(271 * var(--k));width:6em;font:300 calc(88 * var(--k))/1.045 "Inter Tight",sans-serif;letter-spacing:-.05em}
h1 .ln{display:block;overflow:hidden;padding:0 .2em .16em 0;margin-bottom:-.16em}
h1 .ln>span{display:block;color:transparent;-webkit-text-fill-color:transparent;-webkit-background-clip:text;background-clip:text;
  background-image:linear-gradient(112deg,#fff 0%,#fff 54%,#ffd5c2 66%,rgba(255,156,108,.62) 80%,rgba(255,110,60,.14) 100%);background-size:100% 300%}
h1 .ln:nth-child(1)>span{background-position:0 0}
h1 .ln:nth-child(2)>span{background-position:0 50%}
h1 .ln:nth-child(3)>span{background-position:0 100%}
h1 em{font:italic 400 1.02em/1 "Instrument Serif",serif;letter-spacing:-.02em}
.sub{position:absolute;left:calc(19 * var(--k));top:calc(562 * var(--k));font:400 max(calc(16 * var(--k)),11px)/1.69 Inter,sans-serif;color:rgba(255,255,255,.82)}
/* Main CTA: white fill grows out of the arrow circle, label rolls, arrow swaps, magnetic pull, shine every 6 s. */
.cta{position:absolute;left:calc(19 * var(--k));top:calc(652 * var(--k));width:calc(199 * var(--k));height:calc(64 * var(--k));border-radius:999px;overflow:hidden;isolation:isolate;
  background:linear-gradient(90deg,#ff5a2a,#e8230f);display:flex;align-items:center;padding-left:calc(26 * var(--k));
  font:500 max(calc(14 * var(--k)),10px)/1 Inter,sans-serif;color:#fff;box-shadow:0 calc(10 * var(--k)) calc(30 * var(--k)) rgba(232,35,15,.35);
  translate:var(--mx,0) var(--my,0);transition:translate .5s cubic-bezier(.2,.8,.2,1),scale .4s cubic-bezier(.3,1.5,.5,1),box-shadow .6s}
.cta::before{content:"";position:absolute;inset:0;z-index:-1;background:#fff;clip-path:circle(calc(27 * var(--k)) at calc(100% - 32 * var(--k)) 50%);
  transition:clip-path .75s cubic-bezier(.65,0,.25,1)}
.cta::after{content:"";position:absolute;top:0;bottom:0;left:-40%;width:30%;z-index:-1;background:linear-gradient(100deg,transparent,rgba(255,255,255,.38),transparent);
  translate:-100% 0;pointer-events:none}
.cta i{position:relative;display:block;height:1.2em;overflow:hidden;font-style:normal}
.cta i span{display:block;line-height:1.2em;transition:translate .6s cubic-bezier(.65,0,.25,1),color .3s .45s}
.cta i span::after{content:attr(data-t);display:block;color:#1a0d08}
.cta b{position:absolute;right:calc(5 * var(--k));top:calc(5 * var(--k));width:calc(54 * var(--k));height:calc(54 * var(--k));border-radius:50%;background:#fff;overflow:hidden;
  transition:background .5s .1s,scale .5s cubic-bezier(.3,1.6,.5,1)}
.cta b svg{position:absolute;left:0;right:0;top:0;bottom:0;margin:auto;width:calc(18 * var(--k));height:calc(18 * var(--k));transition:translate .55s cubic-bezier(.65,0,.25,1)}
.cta b svg path{stroke:#1a0d08;transition:stroke .4s .1s}
.cta b svg+svg{translate:-260% 0}
.cta:hover{box-shadow:0 calc(16 * var(--k)) calc(40 * var(--k)) rgba(232,35,15,.5)}
.cta:hover::before{clip-path:circle(150% at calc(100% - 32 * var(--k)) 50%)}
.cta:hover i span{translate:0 -1.2em;color:#1a0d08;transition:translate .6s cubic-bezier(.65,0,.25,1),color 0s}
.cta:hover b{background:#e8230f}
.cta:hover b svg{translate:260% 0}
.cta:hover b svg+svg{translate:0 0}
.cta:hover b svg path{stroke:#fff}
.cta:active{scale:.97}
.clients{position:absolute;left:calc(244 * var(--k));top:calc(665 * var(--k));display:flex;align-items:center;gap:calc(10 * var(--k))}
.faces{display:flex}
.faces svg{width:calc(38 * var(--k));height:calc(38 * var(--k));border-radius:50%;box-shadow:0 0 0 calc(2 * var(--k)) rgba(20,4,2,.9)}
.faces svg+svg{margin-left:calc(-12 * var(--k))}
.clients p{font:500 max(calc(11 * var(--k)),9px)/1.35 Inter,sans-serif}
.clients p span{display:block;font-weight:400;color:rgba(255,255,255,.5);font-size:.92em}
.stat{position:absolute;bottom:calc(161 * var(--k));width:calc(203 * var(--k));height:calc(134 * var(--k));border-radius:calc(14 * var(--k));padding:calc(14 * var(--k)) calc(16 * var(--k));
  backdrop-filter:blur(14px);-webkit-backdrop-filter:blur(14px)}
.stat.s1{left:calc(17 * var(--k));background:rgba(255,255,255,.07);box-shadow:inset 0 0 0 1px rgba(255,255,255,.28)}
.stat.s2{left:calc(234 * var(--k));background:rgba(140,22,6,.62);box-shadow:inset 0 0 0 1px rgba(255,120,80,.35)}
.stat strong{display:block;font:300 calc(44 * var(--k))/1 "Inter Tight",sans-serif;letter-spacing:-.04em}
.stat sup{position:absolute;right:calc(14 * var(--k));top:calc(10 * var(--k));font:300 calc(24 * var(--k))/1 "Inter Tight",sans-serif;color:rgba(255,255,255,.85)}
.stat small{position:absolute;left:calc(16 * var(--k));bottom:calc(18 * var(--k));font:400 max(calc(11 * var(--k)),9px)/1 Inter,sans-serif;color:rgba(255,255,255,.75)}
.stat em{position:absolute;right:calc(16 * var(--k));bottom:calc(32 * var(--k));font:400 max(calc(7 * var(--k)),7px)/1 Inter,sans-serif;font-style:normal;color:rgba(255,255,255,.45)}
.ghost{position:absolute;z-index:3;left:calc(8 * var(--k));bottom:calc(-51 * var(--k));font:200 calc(230 * var(--k))/1 "Inter Tight",sans-serif;letter-spacing:.02em;
  color:transparent;-webkit-text-stroke:1px rgba(255,255,255,.17);pointer-events:none;white-space:nowrap}

/* ---------- Right column ---------- */
.bars{position:absolute;right:calc(191 * var(--k));top:calc(258 * var(--k));height:calc(133 * var(--k));display:flex;align-items:flex-end;gap:calc(14.6 * var(--k))}
.bars i{width:calc(18 * var(--k));border-radius:calc(3 * var(--k));background:linear-gradient(0deg,rgba(255,255,255,.03),rgba(255,255,255,.16));transform-origin:50% 100%}
.metric{position:absolute;left:calc(100% - 154 * var(--k));top:calc(288 * var(--k));font:400 max(calc(12 * var(--k)),9px)/1.3 Inter,sans-serif;color:rgba(255,255,255,.45)}
.impact{position:absolute;left:calc(100% - 346 * var(--k));top:calc(438 * var(--k));width:calc(300 * var(--k))}
.impact h2{font:400 max(calc(16 * var(--k)),11px)/1.2 Inter,sans-serif;color:rgba(255,255,255,.45)}
.impact p{margin-top:calc(12 * var(--k));font:400 max(calc(11.5 * var(--k)),9px)/1.55 Inter,sans-serif;color:rgba(255,255,255,.3)}
.ring{position:absolute;z-index:2;pointer-events:none}
.ring circle{fill:none;stroke:rgba(255,255,255,.14);stroke-width:1;vector-effect:non-scaling-stroke}
.ring.r1{right:calc(218 * var(--k));top:calc(577 * var(--k));width:calc(180 * var(--k));height:calc(180 * var(--k))}
.ring.r2{right:calc(69 * var(--k));bottom:calc(74 * var(--k));width:calc(128 * var(--k));height:calc(128 * var(--k))}
.partners-label{position:absolute;right:calc(43 * var(--k));bottom:calc(144 * var(--k));font:400 max(calc(13 * var(--k)),10px)/1.2 Inter,sans-serif;color:rgba(255,255,255,.85)}
.partners{position:absolute;right:calc(-30 * var(--k));bottom:calc(66 * var(--k));display:flex;align-items:center;gap:calc(40 * var(--k));
  -webkit-mask-image:linear-gradient(270deg,transparent 0,#000 calc(150 * var(--k)));mask-image:linear-gradient(270deg,transparent 0,#000 calc(150 * var(--k)))}
.partners span{display:flex;align-items:center;gap:calc(8 * var(--k));font:500 calc(18 * var(--k))/1 "Inter Tight",sans-serif;letter-spacing:-.01em;color:#fff;white-space:nowrap}
.partners svg{width:calc(18 * var(--k));height:calc(18 * var(--k))}

/* ---------- Entrance timeline ---------- */
.js .a,.js .bg{opacity:0}
.ready .bg{animation:bgIn 2.2s cubic-bezier(.22,.8,.24,1) both}
.ready .a{animation:rise 1s cubic-bezier(.2,.8,.2,1) both;animation-delay:var(--d,0s)}
.ready .a.down{animation-name:down}
.ready .a.card{animation-name:card;animation-duration:1.1s}
.ready .a.fade{animation-name:fade;animation-duration:1.8s}
.ready .vline{animation:grow 1.6s cubic-bezier(.65,0,.25,1) both;animation-delay:var(--d,0s)}
.ready h1.a{animation:none;opacity:1}
.ready h1 .ln>span{animation:line 1.2s cubic-bezier(.16,1,.3,1) both;animation-delay:var(--d)}
.ready .bars.a{animation:none;opacity:1}
.ready .bars i{animation:bar .9s cubic-bezier(.2,.8,.2,1) both;animation-delay:calc(1.5s + var(--i) * .07s)}
.ready .ring.a{animation:none;opacity:1}
.ready .ring circle{stroke-dasharray:100;animation:draw 2s cubic-bezier(.6,0,.2,1) both;animation-delay:var(--d)}
.ready .partners.a{animation:none;opacity:1}
.ready .partners span{animation:rise .9s cubic-bezier(.2,.8,.2,1) both;animation-delay:calc(2s + var(--i) * .08s)}
.ready .cta.a{animation-fill-mode:backwards;opacity:1}
.ready .cta b{animation:pop .8s cubic-bezier(.3,1.6,.5,1) 1.75s backwards}
.ready .cta::after{animation:shine 6s cubic-bezier(.4,0,.2,1) 2.3s infinite}
.js .dust{opacity:0}
.ready .dust{opacity:1;transition:opacity 2.4s ease .6s}
.js .vline{scale:1 0}
@keyframes pop{from{scale:0}to{scale:1}}
@keyframes shine{0%{translate:-100% 0}18%,100%{translate:560% 0}}
@keyframes bgIn{from{opacity:0;scale:1.1;filter:blur(18px) brightness(.55)}to{opacity:1;scale:1;filter:none}}
@keyframes rise{from{opacity:0;translate:0 18px;filter:blur(8px)}to{opacity:1;translate:0 0;filter:none}}
@keyframes down{from{opacity:0;translate:0 -14px;filter:blur(6px)}to{opacity:1;translate:0 0;filter:none}}
@keyframes card{from{opacity:0;translate:0 26px;scale:.97;filter:blur(10px)}to{opacity:1;translate:0 0;scale:1;filter:none}}
@keyframes fade{from{opacity:0;translate:0 40px}to{opacity:1;translate:0 0}}
@keyframes line{from{translate:0 105%;opacity:0;filter:blur(6px)}to{translate:0 0;opacity:1;filter:none}}
@keyframes grow{from{scale:1 0}to{scale:1 1}}
@keyframes bar{from{scale:1 0;opacity:0}to{scale:1 1;opacity:1}}
@keyframes draw{from{stroke-dashoffset:100}to{stroke-dashoffset:var(--off,0)}}

/* ---------- Tablet and phone: column in flow ---------- */
@media (max-width:1099px){
  .hero{height:auto;min-height:100svh;display:flex;flex-direction:column;padding:20px 24px 28px}
  .bg img{object-position:55% 50%}
  .shade{background:linear-gradient(180deg,rgba(8,1,0,.7) 0%,rgba(8,1,0,.35) 38%,rgba(8,1,0,.1) 60%,rgba(8,1,0,.7) 100%)}
  .frame{position:relative;top:auto;bottom:auto;flex:1;display:flex;flex-direction:column;width:100%}
  .ui{position:relative;inset:auto;flex:1;display:flex;flex-direction:column}
  .ui>*,.head>*,.row>*,.stats>*{position:relative!important;left:auto!important;right:auto!important;top:auto!important;bottom:auto!important}
  .vline.v1{left:33.33%}.vline.v4{right:33.33%}.vline.v2,.vline.v3{display:none}
  .vline{top:64px}
  .head{display:flex;align-items:center;justify-content:space-between}
  .logo{gap:10px}.logo svg{width:36px;height:36px}.logo span{font-size:24px}
  .nav{display:none}
  .btn-top{width:auto;height:40px;padding:0 18px;border-radius:7px;font-size:13px}
  .badge{margin-top:clamp(48px,9vh,96px);gap:10px}.badge svg{width:26px;height:26px}.badge i{height:24px}.badge p{font-size:11px}
  h1{margin-top:18px;font-size:clamp(46px,9.2vw,84px)}
  .sub{margin-top:18px;font-size:15px;line-height:1.6;max-width:30em}
  .row{display:flex;align-items:center;gap:18px;flex-wrap:wrap;margin-top:26px}
  .cta{position:relative;left:auto;top:auto;width:184px;height:58px;padding-left:24px;font-size:14px}
  .cta b{right:5px;top:5px;width:48px;height:48px}.cta b svg{width:16px;height:16px}
  .cta::before{clip-path:circle(24px at calc(100% - 29px) 50%)}
  .cta:hover::before{clip-path:circle(150% at calc(100% - 29px) 50%)}
  .clients{gap:10px}.faces svg{width:34px;height:34px;box-shadow:0 0 0 2px rgba(20,4,2,.9)}.faces svg+svg{margin-left:-11px}.clients p{font-size:11px}
  .stats{display:flex;gap:12px;margin-top:auto;padding-top:40px}
  .stat{position:relative;left:auto!important;bottom:auto;width:min(203px,calc(50% - 6px));height:124px;border-radius:14px;padding:14px 16px}
  .stat strong{font-size:40px}.stat sup{right:14px;top:10px;font-size:22px}.stat small{left:16px;bottom:16px;font-size:11px}.stat em{right:16px;bottom:30px;font-size:7px}
  .ghost{position:absolute!important;left:0!important;bottom:-0.22em!important;font-size:clamp(130px,30vw,260px)}
  .bars,.metric,.impact,.ring{display:none}
  .partners-label{margin-top:28px;font-size:12px;color:rgba(255,255,255,.7)}
  .partners{margin-top:12px;gap:26px;flex-wrap:wrap;-webkit-mask-image:none;mask-image:none}
  .partners span{font-size:16px;gap:7px}.partners svg{width:16px;height:16px}
}
@media (max-width:600px){
  .hero{padding:16px 16px 24px}
  .bg img{object-position:58% 50%}
  .btn-top{height:36px;padding:0 14px;font-size:12px}
  .badge{margin-top:44px}
  .stat{height:112px}.stat strong{font-size:34px}
  .partners{gap:14px 22px}
  .partners span{font-size:15px}
}
@media (min-width:1100px){.head,.row,.stats{display:contents}}

/* ---------- Reduced motion ---------- */
@media (prefers-reduced-motion:reduce){
  .js .a,.js .bg,.js h1 .ln>span{opacity:1!important;animation:none!important}
  .js .vline{scale:1;animation:none!important}
  .ready .bars i,.ready .partners span,.ready .ring circle,.ready .cta b,.ready .cta::after{animation:none!important}
  .js .dust{opacity:1}
}
</style>
<noscript><style>.js .a,.js .bg{opacity:1}</style></noscript>
</head>
<body>
<script>document.documentElement.classList.add('js')</script>
<section class="hero" aria-label="Sollene creative studio">
  <div class="bg">
    <img src="https://assets.layerweb.com.br/sollene/images/hero-visor.webp" alt="Man wearing a glowing visor, lit by a red halo" crossorigin="anonymous" decoding="async" fetchpriority="high">
    <canvas aria-hidden="true"></canvas>
  </div>
  <div class="shade"></div>
  <canvas class="dust" aria-hidden="true"></canvas>
  <div class="frame">
  <i class="vline v1" style="--d:.2s"></i><i class="vline v2" style="--d:.3s"></i><i class="vline v3" style="--d:.4s"></i><i class="vline v4" style="--d:.5s"></i>
  <svg class="ring r1 a" viewBox="0 0 100 100" aria-hidden="true"><circle cx="50" cy="50" r="49.5" pathLength="100" transform="rotate(170 50 50)" style="--d:1.8s;--off:58"/></svg>
  <svg class="ring r2 a" viewBox="0 0 100 100" aria-hidden="true"><circle cx="50" cy="50" r="49.5" pathLength="100" transform="rotate(180 50 50)" style="--d:2s;--off:62"/></svg>
  <div class="ghost a fade" style="--d:1.6s" aria-hidden="true">MUSE</div>

  <div class="ui">
    <div class="head">
      <a class="logo a down" href="#" aria-label="Sollene home" style="--d:.4s">
        <svg viewBox="0 0 48 48" aria-hidden="true"><path fill="#ff4a22" d="M26.29 4.13A20 20 0 1 0 43.71 27.37A15 15 0 1 1 26.29 4.13Z"/><circle fill="#ff4a22" cx="32" cy="18" r="5"/></svg>
        <span>Sollene</span>
      </a>
      <nav class="nav a down" aria-label="Main" style="--d:.5s">
        <a href="#">Services <svg viewBox="0 0 10 10" aria-hidden="true"><path d="M2 3.5l3 3 3-3" fill="none" stroke="currentColor" stroke-width="1.3" stroke-linecap="round" stroke-linejoin="round"/></svg></a>
        <a href="#">Our Approach</a>
        <a href="#">Story</a>
        <a href="#">Journal</a>
        <a href="#">Works</a>
      </nav>
      <a class="btn-top a down" href="#" style="--d:.6s">Book a Call</a>
    </div>

    <div class="badge a" style="--d:.7s">
      <svg viewBox="0 0 28 28" aria-hidden="true" fill="none" stroke="rgba(255,255,255,.85)" stroke-width="1.2"><circle cx="14" cy="14" r="12"/><ellipse cx="14" cy="14" rx="5.2" ry="12"/><path d="M2 14h24M4 8h20M4 20h20"/></svg>
      <i></i>
      <p>Loved by creators from<br>all across the globe</p>
    </div>

    <h1 class="a">
      <span class="ln"><span style="--d:.8s">Expression</span></span>
      <span class="ln"><span style="--d:.95s">Built for Souls</span></span>
      <span class="ln"><span style="--d:1.1s">Not <em>Formulas</em></span></span>
    </h1>

    <p class="sub a" style="--d:1.3s">We design bold, emotive, and inclusive digital art <br>stories guided by the way real minds feel.</p>

    <div class="row">
      <a class="cta a" href="#" style="--d:1.45s" aria-label="Start today"><i aria-hidden="true"><span data-t="Start today">Start today</span></i><b aria-hidden="true"><svg viewBox="0 0 18 18"><path d="M3 9h12M10 4l5 5-5 5" fill="none" stroke-width="1.6" stroke-linecap="round" stroke-linejoin="round"/></svg><svg viewBox="0 0 18 18"><path d="M3 9h12M10 4l5 5-5 5" fill="none" stroke-width="1.6" stroke-linecap="round" stroke-linejoin="round"/></svg></b></a>
      <div class="clients a" style="--d:1.55s">
        <div class="faces" aria-hidden="true">
          <svg viewBox="0 0 38 38"><defs><linearGradient id="fa" x1="0" y1="0" x2="1" y2="1"><stop offset="0" stop-color="#ff7a3d"/><stop offset="1" stop-color="#b3200c"/></linearGradient></defs><rect width="38" height="38" fill="url(#fa)"/><circle cx="19" cy="15" r="6.5" fill="#3a0d05"/><path d="M6 38c1.5-8 7-12 13-12s11.5 4 13 12z" fill="#3a0d05"/></svg>
          <svg viewBox="0 0 38 38"><defs><linearGradient id="fb" x1="0" y1="0" x2="1" y2="1"><stop offset="0" stop-color="#5a1a40"/><stop offset="1" stop-color="#1c0a1a"/></linearGradient></defs><rect width="38" height="38" fill="url(#fb)"/><circle cx="19" cy="15" r="6.5" fill="#e9a07a"/><path d="M6 38c1.5-8 7-12 13-12s11.5 4 13 12z" fill="#e9a07a"/></svg>
          <svg viewBox="0 0 38 38"><defs><linearGradient id="fc" x1="0" y1="0" x2="1" y2="1"><stop offset="0" stop-color="#3fc4c9"/><stop offset="1" stop-color="#14606e"/></linearGradient></defs><rect width="38" height="38" fill="url(#fc)"/><circle cx="19" cy="15" r="6.5" fill="#0b2a31"/><path d="M6 38c1.5-8 7-12 13-12s11.5 4 13 12z" fill="#0b2a31"/></svg>
        </div>
        <p>80K+ Loyal Viewers<span>All Regions</span></p>
      </div>
    </div>

    <div class="stats">
      <div class="stat s1 a card" style="--d:1.65s"><strong>240+</strong><sup>*</sup><em>01</em><small>Campaigns launched</small></div>
      <div class="stat s2 a card" style="--d:1.75s"><strong>96%</strong><sup>*</sup><em>02</em><small>Partners who return</small></div>
    </div>

    <div class="bars a" aria-hidden="true"><i style="--i:0;height:30%"></i><i style="--i:1;height:42%"></i><i style="--i:2;height:55%"></i><i style="--i:3;height:68%"></i><i style="--i:4;height:84%"></i><i style="--i:5;height:100%"></i></div>
    <p class="metric a" style="--d:1.7s">+68%<br>Engagement<br>Growth Rate</p>
    <div class="impact a" style="--d:1.8s">
      <h2>Lasting Brand Glow</h2>
      <p>Track the shift through honest, clear signals<br>moments that stay</p>
    </div>

    <p class="partners-label a" style="--d:1.9s">Partner Labs</p>
    <div class="partners a" aria-label="Partners">
      <span style="--i:0"><svg viewBox="0 0 18 18" aria-hidden="true" fill="none" stroke="#fff" stroke-width="1.8" stroke-linejoin="round"><path d="M3 12l6-6 6 6M3 16l6-6 6 6"/></svg>Northline</span>
      <span style="--i:1"><svg viewBox="0 0 18 18" aria-hidden="true" fill="#fff"><path d="M9 1l8 8-8 8-8-8z M9 5.5L5.5 9 9 12.5 12.5 9z" fill-rule="evenodd"/></svg>kivora</span>
      <span style="--i:2"><svg viewBox="0 0 18 18" aria-hidden="true" fill="none" stroke="#fff" stroke-width="1.8"><circle cx="9" cy="9" r="6.5"/><circle cx="9" cy="9" r="1.8" fill="#fff" stroke="none"/></svg>Orbit</span>
      <span style="--i:3"><svg viewBox="0 0 18 18" aria-hidden="true" fill="none" stroke="#fff" stroke-width="1.8" stroke-linecap="round"><path d="M1.5 8c2-3 4-3 6 0s4 3 6 0 2.5-1.5 3-1M1.5 13c2-3 4-3 6 0s4 3 6 0 2.5-1.5 3-1"/></svg>Halcyon</span>
      <span style="--i:4"><svg viewBox="0 0 18 18" aria-hidden="true" fill="#fff"><path d="M15.5 2.5C9 2.5 4 7 4 13.5V16h1.6v-2.4C7.5 13.6 15.5 10 15.5 2.5z"/></svg>Plume</span>
    </div>
  </div>
  </div>
</section>

<script>
/* Sollene hero: entrance trigger, CTA pull, drifting dust and liquid cursor on the background only. */
(() => {
  const root = document.documentElement;
  const hero = document.querySelector('.hero');
  const img = hero.querySelector('.bg img');
  const canvas = hero.querySelector('.bg canvas');
  const reduce = matchMedia('(prefers-reduced-motion: reduce)').matches;

  // Entrance starts once the background image and fonts are ready (3 s cap).
  const imgReady = img.complete ? Promise.resolve() : new Promise(r => { img.onload = img.onerror = r; });
  const fontsReady = (document.fonts ? document.fonts.ready : Promise.resolve()).catch(() => {});
  Promise.race([Promise.all([imgReady, fontsReady]), new Promise(r => setTimeout(r, 3000))])
    .then(() => requestAnimationFrame(() => root.classList.add('ready')));

  // ---- Main CTA: small magnetic pull toward the pointer ----
  const cta = hero.querySelector('.cta');
  if (!reduce && matchMedia('(hover: hover)').matches) {
    cta.addEventListener('pointermove', e => {
      const r = cta.getBoundingClientRect();
      cta.style.setProperty('--mx', (((e.clientX - r.left) / r.width - .5) * 12).toFixed(1) + 'px');
      cta.style.setProperty('--my', (((e.clientY - r.top) / r.height - .5) * 10).toFixed(1) + 'px');
    });
    cta.addEventListener('pointerleave', () => { cta.style.removeProperty('--mx'); cta.style.removeProperty('--my'); });
  }

  // ---- Dust: slow warm particles drifting up behind the text ----
  const dust = hero.querySelector('.dust'), dctx = dust.getContext('2d');
  const sprite = document.createElement('canvas');
  sprite.width = sprite.height = 32;
  { const s = sprite.getContext('2d'), g = s.createRadialGradient(16, 16, 0, 16, 16, 16);
    g.addColorStop(0, 'rgba(255,214,186,1)'); g.addColorStop(.25, 'rgba(255,170,120,.55)'); g.addColorStop(1, 'rgba(255,120,60,0)');
    s.fillStyle = g; s.fillRect(0, 0, 32, 32); }
  let parts = [], dw = 0, dh = 0, ddpr = 1, dOn = true, pmx = 0, pmy = 0, tmx = 0, tmy = 0, dThen = 0;
  const seed = () => {
    const r = dust.getBoundingClientRect(); dw = Math.max(1, r.width); dh = Math.max(1, r.height);
    ddpr = Math.min(window.devicePixelRatio || 1, 2);
    dust.width = Math.round(dw * ddpr); dust.height = Math.round(dh * ddpr);
    parts = Array.from({ length: Math.min(90, Math.round(dw * dh / 16000)) }, () => ({
      x: Math.random() * dw, y: Math.random() * dh, z: .3 + Math.random() * .7, r: .6 + Math.random() * 1.4,
      vy: 6 + Math.random() * 14, ph: Math.random() * 6.283, tw: .6 + Math.random() * 1.6, sw: 8 + Math.random() * 18 }));
    paint(performance.now(), 0);
  };
  const paint = (t, dt) => {
    dctx.setTransform(ddpr, 0, 0, ddpr, 0, 0); dctx.clearRect(0, 0, dw, dh);
    dctx.globalCompositeOperation = 'lighter';
    pmx += (tmx - pmx) * .04; pmy += (tmy - pmy) * .04;
    for (const p of parts) {
      p.y -= p.vy * p.z * dt;
      if (p.y < -12) { p.y = dh + 12; p.x = Math.random() * dw; }
      const x = p.x + Math.sin(t * .0004 * p.tw + p.ph) * p.sw + pmx * 16 * p.z, y = p.y + pmy * 10 * p.z;
      const edge = Math.min(1, y / (dh * .18), (dh - y) / (dh * .08));
      dctx.globalAlpha = Math.max(0, edge) * p.z * (.22 + .4 * (.5 + .5 * Math.sin(t * .001 * p.tw + p.ph)));
      const s = p.r * 6 * p.z + 2;
      dctx.drawImage(sprite, x - s / 2, y - s / 2, s, s);
    }
  };
  const dLoop = t => {
    if (!dOn) return;
    const dt = Math.min(.05, (t - (dThen || t)) / 1000); dThen = t;
    paint(t, dt); requestAnimationFrame(dLoop);
  };
  seed();
  new ResizeObserver(seed).observe(dust);
  if (!reduce) {
    addEventListener('pointermove', e => { tmx = .5 - e.clientX / innerWidth; tmy = .5 - e.clientY / innerHeight; }, { passive: true });
    new IntersectionObserver(([en]) => { const was = dOn; dOn = en.isIntersecting; if (dOn && !was) { dThen = 0; requestAnimationFrame(dLoop); } }).observe(hero);
    requestAnimationFrame(dLoop);
  }

  const gl = canvas.getContext('webgl2', { alpha: false, antialias: false, premultipliedAlpha: false });
  if (!gl) return;

  // ---- Shaders ----
  const VS = `#version 300 es
  in vec2 p; out vec2 vUv;
  void main(){ vUv = vec2(p.x * .5 + .5, .5 - p.y * .5); gl_Position = vec4(p, 0., 1.); }`;
  const FS = `#version 300 es
  precision highp float;
  in vec2 vUv; out vec4 o;
  uniform sampler2D uImg, uWave;
  uniform vec2 uScale, uStart;
  uniform float uAmp;
  void main(){
    vec4 w = texture(uWave, vUv);
    vec2 n = (w.rg - .5) * 2.;
    float h = (w.b - .5) * 2.;
    vec2 off = n * uAmp;
    vec2 iu = uStart + (vUv + off) * uScale;
    vec2 ca = off * uScale * .2;
    vec3 c = vec3(texture(uImg, iu + ca).r, texture(uImg, iu).g, texture(uImg, iu - ca).b);
    vec3 N = normalize(vec3(-n * 1.4, 1.));
    float spec = pow(max(dot(N, normalize(vec3(-.35, .45, 1.))), 0.), 60.);
    c *= 1. + h * .12;
    c += spec * vec3(1., .72, .55) * .35;
    o = vec4(c, 1.);
  }`;
  const sh = (type, src) => { const s = gl.createShader(type); gl.shaderSource(s, src); gl.compileShader(s); return s; };
  const prog = gl.createProgram();
  gl.attachShader(prog, sh(gl.VERTEX_SHADER, VS)); gl.attachShader(prog, sh(gl.FRAGMENT_SHADER, FS));
  gl.bindAttribLocation(prog, 0, 'p'); gl.linkProgram(prog);
  if (!gl.getProgramParameter(prog, gl.LINK_STATUS)) return;
  gl.useProgram(prog);
  gl.bindBuffer(gl.ARRAY_BUFFER, gl.createBuffer());
  gl.bufferData(gl.ARRAY_BUFFER, new Float32Array([-1, -1, 3, -1, -1, 3]), gl.STATIC_DRAW);
  gl.enableVertexAttribArray(0); gl.vertexAttribPointer(0, 2, gl.FLOAT, false, 0, 0);
  const U = n => gl.getUniformLocation(prog, n);
  const uScale = U('uScale'), uStart = U('uStart'), uAmp = U('uAmp');
  gl.uniform1i(U('uImg'), 0); gl.uniform1i(U('uWave'), 1);

  const tex = (unit) => { const t = gl.createTexture(); gl.activeTexture(gl.TEXTURE0 + unit); gl.bindTexture(gl.TEXTURE_2D, t);
    gl.texParameteri(gl.TEXTURE_2D, gl.TEXTURE_WRAP_S, gl.CLAMP_TO_EDGE); gl.texParameteri(gl.TEXTURE_2D, gl.TEXTURE_WRAP_T, gl.CLAMP_TO_EDGE);
    gl.texParameteri(gl.TEXTURE_2D, gl.TEXTURE_MAG_FILTER, gl.LINEAR); gl.texParameteri(gl.TEXTURE_2D, gl.TEXTURE_MIN_FILTER, gl.LINEAR); return t; };
  const imgTex = tex(0), waveTex = tex(1);
  let imgW = 0, imgH = 0;

  // ---- Ripple simulation (CPU height field, 1 cell = CELL css px) ----
  const CELL = 5, DAMP = .98;
  let gw = 0, gh = 0, cur, prev, pack;
  let cw = 1, ch = 1, energy = 0, lastInput = 0, running = false, visible = true;

  const resize = () => {
    const r = canvas.getBoundingClientRect();
    cw = Math.max(1, r.width); ch = Math.max(1, r.height);
    let dpr = Math.min(window.devicePixelRatio || 1, 1.75);
    dpr = Math.min(dpr, Math.sqrt(4.2e6 / (cw * ch)));
    canvas.width = Math.round(cw * dpr); canvas.height = Math.round(ch * dpr);
    gl.viewport(0, 0, canvas.width, canvas.height);
    gw = Math.min(360, Math.ceil(cw / CELL)); gh = Math.min(260, Math.ceil(ch / CELL));
    cur = new Float32Array(gw * gh); prev = new Float32Array(gw * gh); pack = new Uint8Array(gw * gh * 4).fill(128);
    gl.activeTexture(gl.TEXTURE1);
    gl.texImage2D(gl.TEXTURE_2D, 0, gl.RGBA, gw, gh, 0, gl.RGBA, gl.UNSIGNED_BYTE, pack);
    cover(); draw();
  };

  // Same framing as the <img>: object-fit cover + object-position (px% 50%).
  const cover = () => {
    if (!imgW) return;
    const A = cw / ch, I = imgW / imgH, px = cw < 601 ? .58 : cw < 1100 ? .55 : .604;
    if (A < I) { const s = A / I; gl.uniform2f(uScale, s, 1); gl.uniform2f(uStart, px * (1 - s), 0); }
    else { const s = I / A; gl.uniform2f(uScale, 1, s); gl.uniform2f(uStart, 0, .5 * (1 - s)); }
  };

  const drop = (x, y, radius, amp) => {
    const gx = x / CELL, gy = y / CELL, r = radius, r2 = r * r;
    for (let j = Math.max(1, Math.floor(gy - r)); j <= Math.min(gh - 2, Math.ceil(gy + r)); j++)
      for (let i = Math.max(1, Math.floor(gx - r)); i <= Math.min(gw - 2, Math.ceil(gx + r)); i++) {
        const d2 = (i - gx) ** 2 + (j - gy) ** 2;
        if (d2 < r2) cur[j * gw + i] -= amp * .5 * (1 + Math.cos(Math.PI * Math.sqrt(d2) / r));
      }
  };

  const step = () => {
    let e = 0;
    for (let j = 1; j < gh - 1; j++) for (let i = 1; i < gw - 1; i++) {
      const k = j * gw + i;
      const v = ((cur[k - 1] + cur[k + 1] + cur[k - gw] + cur[k + gw]) * .5 - prev[k]) * DAMP;
      prev[k] = v; const a = v < 0 ? -v : v; if (a > e) e = a;
    }
    const t = prev; prev = cur; cur = t; energy = e;
  };

  const upload = () => {
    for (let j = 0; j < gh; j++) for (let i = 0; i < gw; i++) {
      const k = j * gw + i, o = k * 4;
      const dx = (cur[i < gw - 1 ? k + 1 : k] - cur[i > 0 ? k - 1 : k]);
      const dy = (cur[j < gh - 1 ? k + gw : k] - cur[j > 0 ? k - gw : k]);
      pack[o] = Math.max(0, Math.min(255, 128 + dx * 480));
      pack[o + 1] = Math.max(0, Math.min(255, 128 + dy * 480));
      pack[o + 2] = Math.max(0, Math.min(255, 128 + cur[k] * 110));
    }
    gl.activeTexture(gl.TEXTURE1);
    gl.texSubImage2D(gl.TEXTURE_2D, 0, 0, 0, gw, gh, gl.RGBA, gl.UNSIGNED_BYTE, pack);
  };

  const draw = () => { if (imgW) { gl.uniform1f(uAmp, .018); gl.drawArrays(gl.TRIANGLES, 0, 3); } };

  // Pointer trail: drops every ~1.5 cells along the travelled segment.
  let px0 = null, py0 = null;
  const queue = [];
  const onMove = e => {
    if (!visible) return;
    const r = canvas.getBoundingClientRect();
    const x = e.clientX - r.left, y = e.clientY - r.top;
    if (x < 0 || y < 0 || x > r.width || y > r.height) { px0 = null; return; }
    queue.push([x, y]); lastInput = performance.now(); start();
  };
  const flush = () => {
    for (const [x, y] of queue) {
      if (px0 === null) { px0 = x; py0 = y; continue; }
      const dx = x - px0, dy = y - py0, dist = Math.hypot(dx, dy);
      const amp = Math.min(1, dist / 40) * .55 + .08;
      const n = Math.max(1, Math.ceil(dist / (CELL * 1.5)));
      for (let s = 1; s <= n; s++) drop(px0 + dx * s / n, py0 + dy * s / n, 3.2, amp / Math.sqrt(n));
      px0 = x; py0 = y;
    }
    queue.length = 0;
  };

  let acc = 0, then = 0;
  const tick = now => {
    if (!running) return;
    const dt = Math.min(50, now - (then || now)); then = now; acc += dt;
    flush();
    let steps = 0;
    while (acc >= 1000 / 60 && steps < 3) { step(); acc -= 1000 / 60; steps++; }
    if (steps) { upload(); draw(); }
    if (energy < .002 && now - lastInput > 1200) { running = false; cur.fill(0); prev.fill(0); upload(); draw(); return; }
    requestAnimationFrame(tick);
  };
  const start = () => { if (!running && imgW && !reduce) { running = true; then = 0; acc = 0; requestAnimationFrame(tick); } };

  const init = () => {
    imgW = img.naturalWidth; imgH = img.naturalHeight;
    if (!imgW) return;
    gl.activeTexture(gl.TEXTURE0);
    gl.texImage2D(gl.TEXTURE_2D, 0, gl.RGBA, gl.RGBA, gl.UNSIGNED_BYTE, img);
    gl.generateMipmap(gl.TEXTURE_2D);
    gl.texParameteri(gl.TEXTURE_2D, gl.TEXTURE_MIN_FILTER, gl.LINEAR_MIPMAP_LINEAR);
    if (gl.getError() !== gl.NO_ERROR) return;
    root.classList.add('gl');
    resize();
    new ResizeObserver(resize).observe(canvas.parentElement);
    if (reduce) return;
    addEventListener('pointermove', onMove, { passive: true });
    addEventListener('pointerdown', onMove, { passive: true });
    new IntersectionObserver(([en]) => { visible = en.isIntersecting; }).observe(hero);
    // One soft ring on the visor while the background appears.
    setTimeout(() => { if (!visible) return; drop(cw * (cw < 1100 ? .62 : .58), ch * .3, 9, .8); lastInput = performance.now(); start(); }, 900);
  };
  imgReady.then(() => (img.decode ? img.decode().catch(() => {}) : 0)).then(init);
})();
</script>
</body>
</html>
```
