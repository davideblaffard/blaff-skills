---
name: blaff-rea
description: Reverse-engineer a public website - measure its type, colour, spacing and components as design tokens, build an unofficial brand guide, then apply the findings to the user's own project.
---

# blaff-rea · reverse-engineer a website

Turn a public website into an **unofficial, art-directed brand guide**: every value is measured, tokenised and rebuilt to prove the tokens work. Then, if the user wants, carry those design decisions into one of their own projects, rewritten in their own terms. Never copy the site's code.

Talk to the user in their own language. Ask one clear question at a time. Use AskUserQuestion for multiple-choice questions when it is available; otherwise ask in plain text.

---

## Phase 0 · Intake (before touching anything)

Only ask for what is missing. If the user has already said it, don't ask again.

1. **URL.** "Send me the URL you want to work on." Check that it is a public http(s) site and derive the host. Outputs go to `out/<host>/`.
2. **Images.** "Do you want images in the guide?"
   - `none`: no images. The gradient fallbacks are the design. This is the recommended default.
   - `api`: generated through a provider (Google Gemini / Nano Banana, OpenAI Images, fal, Replicate…). Ask which one. The key goes in `.env` and **must never be pasted into chat**. If the user pastes it anyway, tell them to revoke it and put a new one in `.env`.
   - `manual`: you print the prompts. The user generates the images in any tool and drops them into `images/<slot>.png`.
3. **Outputs.** "What do you need?" (multiple choice)
   - Brand guide + tokens (always included)
   - Social slides (Instagram + X)
   - 60 fps presentation video
4. **Target project** (can also be asked at the end, in Phase 10): "Do you want to apply what we take from <host> to one of your projects? If so, which one: a folder, a GitHub repo or a Vercel project?"

Before starting, check the environment: `node`, Playwright, an installed Chrome, and `ffmpeg` (only if the video is needed). Install what is missing, or say what is missing. If the shell cannot reach the site (restricted network, allowlist), say so straight away and offer option (b) from the bot-check rule below.

Sum up the plan in three lines (URL, images, outputs, project) and start. Ask for explicit confirmation only before steps that cost money: before generating images through an API, show the estimated cost.

---

## Non-negotiables

- Mark everything **"Unofficial analysis"**: the guide's cover, top bar and footer, every slide, and the video. Add: "Not affiliated with or endorsed by <brand>. Trademarks belong to their owners."
- Never copy logos, font files or the brand's images. Describe them and link to the source pages. A logo becomes a dashed placeholder with its measured size and colour.
- Never bypass bot checks or CAPTCHAs. If headless capture gets a challenge, stop and offer two options:
  - (a) `--headed`: a visible browser opens and the user completes the check;
  - (b) a paste-into-DevTools snippet that runs the same extractor in same-origin iframes at 1440px and 390px and downloads `snapshot.json`.
- AI images are generated **from text prompts only**, never with the brand's photos as input. Each one is labelled "AI-generated image" wherever it appears.
- Copy is plain and technical. Section titles are standard brand-book names: Overview, Contents, Typography, Fonts, Colour, Layout and spacing, Components, Imagery, Voice and tone, Design tokens, Method. No manifestos or slogans.

---

## Technical goal

Build `reverse-brand <url>`, a reusable CLI, then run it on the user's URL. The guide should look and feel like the brand's own site, not like a report.

## Phase 1 · Capture

- Use Playwright with an installed Chrome, launched with `--disable-gpu` (some machines hang on the first canvas call without it).
- Capture the home page plus 3–5 key pages, chosen by scoring same-origin links:
  - nav links and product, pricing, business and company pages score highest;
  - skip legal, help, login and locale duplicates;
  - take one page per top-level path.
- Use a 1440×900 viewport, plus a 390×844 pass of the home page for responsive type.
- Scroll each page to trigger lazy content. Then walk every visible element and record computed values:
  - font, size, weight, line-height and tracking;
  - text, background, border and fill colours, normalised to hex through a 1px canvas (so `oklch()` and `lab()` work);
  - padding, margin and gap, radius, shadows;
  - max-width containers, grid columns and media stats.
- Also record:
  - button and card candidates (the painted layer can be a child of the `<a>`);
  - header and nav (height, sticky, link style, logo size and colour);
  - form fields, h1–h3, paragraphs and CTA labels;
  - `:root` custom properties and media-query breakpoints.
- Exclude cookie and consent UI by id/class and by dialog text.

## Phase 2 · Analyse

- **Dedupe repeated chrome:** any count that is identical on every page (header and footer) is counted once.
- **Type:**
  - cluster heading sizes within 12%, then assign roles: display, h1–h4, lead, body, small;
  - for each role, record size, line-height ratio, weight and tracking in em;
  - match the mobile size per role and emit `clamp()`;
  - report the scale ratio and its nearest named interval.
- **Colour:**
  - report roles as hex plus OKLCH, with usage share: background, surface, surface-inverse, text, text-muted, text-inverse, action, on-action, action-alt, accent, border;
  - weight text by characters and backgrounds by painted area;
  - list gradients and the WCAG contrast of the most-read pairs.
- **Layout:**
  - derive the spacing base unit (8/4/2) and separate on-grid from off-grid values;
  - record radii (pill vs px), container width, columns and gutter, and breakpoints;
  - record the home page's section rhythm (light and dark bands).
- **Components:** button variants (filled, outline, translucent), card variants (fill, backdrop, radius, padding, media) and the nav. If no forms were found, infer an input and flag it "inferred".
- **Imagery:** describe it in words: formats, crops, full-bleed share, object-fit, media share of page area, and alt-text themes.
- **Voice:**
  - quote 5 lines from the site, each 15 words or fewer, with source links;
  - derive 3 rules from measurable stats: headline length, case, "you" vs "we", numbers, the "X, Y" pattern, CTA verbs and exclamation marks;
  - show the stats.

## Phase 3 · Tokens

Export CSS custom properties for fonts, type roles, colour roles, spacing, layout, radii, button, card, input and nav. Export the same set as W3C design-tokens JSON, with OKLCH next to hex. Every brand sample in the guide uses only these variables.

## Phase 4 · The guide (most of the value is here)

Derive the art direction from the brand's own tokens instead of hard-coding it: its fonts, its alternating light and dark grounds, its card radius, its button shape, its gradient and its declared motion easing. The guide's own controls are the brand's rebuilt buttons.

Build it as a sequence of full pages that mirror the site's own layout patterns:

1. **Cover:**
   - a full-bleed photo, or the brand-gradient fallback;
   - "<Brand> brand guidelines" as a big headline on the image, with a one-line factual lede and a pill CTA;
   - a frosted-glass card showing one token, like the site's balance cards;
   - the Unofficial pill and the disclaimer at the bottom.
2. **Overview:** a centred headline and 4 big stats (typefaces, colours, spacing base, button variants).
3. **Contents:** 8 image cards styled like the site's product cards: title bottom-left, number top-left, arrow top-right, slow zoom on hover.
4. **Token rebuild test:** a button and two cards built only from tokens, on the site's light and dark grounds, with real images in the card media.
5. **Typography:** a full-bleed chapter photo with a glass specimen card and chip tabs (Display, Heading, Subhead, Lead, Body). Each tab shows the role at its **real** size, weight, leading and tracking, never one size for all.
6. **Fonts:**
   - a family card per typeface with a giant "Aa", weights, usage share, and a check for whether the font is installed locally;
   - below it: the character set, the type scale with mobile→desktop values, the sizes in use, and the site's declared type variables.
7. **Colour** (dark page):
   - a chrome or product image next to a proportional palette bar;
   - role tiles with hex, OKLCH and contrast;
   - the gradient and the contrast pairs;
   - the full palette in a collapsible panel.
8. **Layout and spacing:** big facts (base, columns, gutter, container), a column diagram, a spacing staircase, radius tiles, breakpoints and the section rhythm.
9. **Components** (dark page): an ordered material image beside the title, then a gallery of measured button variants on their grounds, the nav mock, the cards and the form input.
10. **Imagery:** a photo mosaic with labelled AI images, the measured facts, and links to the source pages.
11. **Voice and tone:** a split page (photo plus facts), then the quoted lines set large and 3 numbered rule cards.
12. **Design tokens** (dark page): copy, copy-JSON and download buttons, plus code panels.
13. **Method:** assumptions, the run log and the image prompts table.

Motion:
- subtle rise-in reveals and a slow zoom on photos, using the brand's easing;
- honour `prefers-reduced-motion`.

Layout:
- a sticky top bar that is transparent over the cover and frosted after;
- responsive, with no horizontal scroll at 390px.

Write **two editions**:
- `brand-guide.html`: no images, with CSS-gradient fallbacks in every image slot. At most 150 KB, works offline.
- `brand-guide-hero.html`: the same guide with images embedded as WebP, each embedded once and reused through CSS classes.

## Phase 5 · Images (optional, depends on the Phase 0 choice)

- First study the site's photography: look at it, don't copy it. Then write `prompts.json` with about 9 slots:
  - `hero` (cover), `horizon` (typography chapter), `object` (studio render for colour), `street` (lifestyle for voice);
  - `sky` (clean background), `glass` (orderly material for components), `grid` (layout), `tokens` (tokens);
  - plus whatever the brand needs.
- Use new subjects in the brand's photographic style, and append the suffix: "photorealistic, no text, letters, logos, brand marks, cards or app screens".
- `none`: skip this phase. The gradient fallbacks are the design.
- `api`:
  - write a small provider adapter `generate({prompt, aspectRatio, resolution}) → image bytes`;
  - read the key from `.env` and never log it;
  - check the balance first and **show the user the total estimated cost before generating**;
  - save job ids before polling, so a crash never loses a paid image;
  - stream downloads to disk and log the cost per image.
- `manual`: print the prompts and tell the user: "Generate the images in any tool you like and save them as `images/<slot>.png`. Tell me when you're done." Then run the `--import` step, which builds the manifest from those files.
- Re-encode everything to WebP (2000px for wide images, 1400px for others) and review the results yourself. Regenerate anything off-brand or noisy.

## Phase 6 · Social slides (only if requested)

Export an Instagram carousel (8 slides at 1080×1350: cover, typography, colour, layout, components, voice, tokens, outro) and X images (4 at 1600×900).
- They use the same tokens and images.
- Every slide carries an Unofficial badge, plus an AI credit when it shows a photo.
- Put text-heavy content beside the image, not on top of busy areas.

## Phase 7 · Video (only if requested)

Produce a 1920×1080 presentation at **real 60 fps**: about 45 seconds, 11 scenes from intro to outro.
- Scenes cross-dissolve with a slight scale and blur.
- Photos get a slow Ken Burns zoom.
- Text, bars and columns animate in with a staggered ease-out.

Render it **frame by frame**:
- keep all CSS animations paused;
- for each frame, seek every animation to `i/fps` through `document.getAnimations()` and capture it;
- pipe the frames into ffmpeg: libx264, CRF 14, `yuv420p`, TV range, BT.709, faststart.

Also write `--stills` (one frame per scene) for review.

## Phase 8 · Verify, then review visually

Write `verify.js` and make it pass. It checks that:
- the guide opens offline with zero network requests;
- the size budgets hold (150 KB light, 4 MB hero);
- the required sections exist, with the Unofficial label and AI labels on every image;
- the token JSON parses;
- there are five quotes and three rules;
- the rebuilt button's and cards' computed styles equal the token values;
- there is no horizontal scroll at 390px.

Then screenshot every section, slide and video still, look at them, and fix what looks wrong before you finish: truncation, low contrast, collisions, bad crops, washed-out images.

## Phase 9 · Log

Write `run-log.md` with:
- the timing per phase;
- pages and elements captured;
- file sizes;
- the cost of each image;
- LLM token usage, if known;
- every assumption (viewports, page picks, role inference, inferred components, fallbacks).

### Guide deliverables

`out/<host>/`:
- `brand-guide.html`, `brand-guide-hero.html`
- `tokens.css`, `tokens.json`, `snapshot.json`, `prompts.json`
- `images/`, `slides/`
- `video/<host>-presentation-1920x1080-60fps.mp4`
- `run-log.md`
- the screenshots from verification

Sum up in a few lines what you measured, what you assumed and what it cost. Then move on to Phase 10.

---

## Phase 10 · Apply it to the user's project

If the project hasn't been named yet, ask: "Tell me which project you want to apply what we took from <host> to: a folder, a GitHub repo or a Vercel project." If the user doesn't want to apply it, stop here.

1. **Read the project before proposing anything:**
   - its stack (framework; Tailwind, CSS modules or plain CSS);
   - where its tokens or global variables live, and how they are named;
   - the fonts already in use;
   - the commands to run and build it.
   If it already has a design system, follow its naming.
2. **Ask what to bring over** (multiple choice): typography, colour, spacing and grid, radii and shadows, buttons and cards, motion. Recommend at most three: a small, recognisable change beats a full redesign.
3. **Write the recipe** in `recipe.md` in the project, in your own words. For each chosen item, write one or two lines with the values from `tokens.json`, renamed to fit the project's conventions. Show it to the user and get confirmation before changing any code.
4. **Rules while applying:**
   - work on a new branch (e.g. `blaff-rea/<host>`) and don't touch `main`;
   - one change at a time: apply, run, check, then make one commit per change;
   - take the decisions, not the code: don't paste the site's CSS or JS, and don't copy its text or content structure;
   - don't use the brand's font files, logos or images. If a font is paid, suggest a free alternative with similar proportions (e.g. from Google Fonts) and explain why;
   - add `prefers-reduced-motion` handling to any motion you introduce;
   - don't touch unrelated things (old bugs, refactors), and never display or read out `.env` or other secrets.
5. **Verify:**
   - before and after screenshots at 1440px and 390px;
   - contrast of the new colours on text;
   - no horizontal scroll;
   - the build passes.
6. **Publish** to a preview deployment only, if the project has one. Push or merge to `main`, or deploy to production, only when the user explicitly asks.

Finish with:
- the changes made (one commit per change);
- the before and after screenshots;
- the preview link;
- what you left out and why.
