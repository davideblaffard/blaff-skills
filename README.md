# blaff-skills

The Claude skills I use in my videos, all in one place and ready to install with one command.

If one of them is useful to you, leave a ⭐. It helps me decide what to make next.

## Skills

| Skill | What it does | Video |
| --- | --- | --- |
| [`blaff-rea`](plugins/blaff/skills/blaff-rea/SKILL.md) | Reverse-engineers a public website: measures type, colour, spacing and components, turns them into design tokens and builds an unofficial brand guide. It can then apply those decisions to your own project, on a branch, one change at a time. | _coming soon_ |

## Install

### Claude Code

```text
/plugin marketplace add davideblaffard/blaff-skills
/plugin install blaff@blaff-skills
```

Then ask Claude something like: "use blaff-rea on https://example.com".

### Claude (app and web)

Download the [`plugins/blaff/skills/blaff-rea`](plugins/blaff/skills/blaff-rea) folder, zip it, and upload it under Skills in Claude's settings.

## How blaff-rea works

1. **It asks what you need:**
   - the URL;
   - whether you want images (none, generated through an API, or made by you);
   - whether you also want social slides and a video.
2. **It measures the site** with Playwright: the home page plus 3–5 key pages, at 1440px and 390px.
3. **It analyses the site and creates tokens:**
   - type roles with `clamp()`;
   - colour roles in hex and OKLCH;
   - the spacing base unit and radii;
   - buttons, cards and voice.

   It exports them as `tokens.css` and `tokens.json`.
4. **It builds the brand guide** using only those tokens, in two editions: a light offline one and one with images.
5. **It verifies the result** with automated checks and screenshots.
6. **It applies the result to your project**, if you want. It first writes a recipe in its own words, then works on a branch, with one commit per change and preview deploys only.

You need Node and Playwright with Chrome installed. `ffmpeg` is only needed for the video.

## Ground rules

These skills are for studying a design, not cloning it.

- Everything they produce is labelled **"Unofficial analysis"**, with a note that there is no affiliation with the brand.
- They never download or copy logos, font files, images, text or code from the site.
- They never bypass CAPTCHAs or bot checks.
- AI images come from text prompts only and are always labelled.

Trademarks belong to their owners.

## License

[MIT](LICENSE)
