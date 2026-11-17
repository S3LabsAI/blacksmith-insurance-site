# BlackSmith Agency — website

One-page marketing site for BlackSmith Agency, an Allstate insurance agency in
The Colony, Texas. Static HTML/CSS, no build step, deployed on Vercel.

The agency is captive to Allstate, so the copy must not claim to shop or compare
carriers. An earlier draft (from the design prototype) was written for an
independent agency and sold exactly that — "multiple carriers", "we shop the
market", "we aren't tied to one company". Those claims were rewritten; keep them
out. Allstate's own brand and advertising rules govern how the name and any logo
may be used, and this site currently uses the name in text only, with no Allstate
logo or trademark asset.

Design source: [Blacksmith Insurance Demo Site](https://claude.ai/design/p/5326169f-8b33-41a5-ab46-818ac5599721?file=BlackSmith+Agency.dc.html) (Claude Design).

## Layout

```
index.html      the whole page
styles.css      all styles (design tokens in :root)
api/quote.js    retired: old quote form → AgencyZoom API function (unused)
scripts/        agencyzoom-ids.js — id lookup for the retired function
vercel.json     cache + security headers
robots.txt      / sitemap.xml
assets/         optimized art (logos, headshot, favicon, OG image)
```

## Local preview

No dependencies and no build. Open `index.html`, or serve it:

```bash
python3 -m http.server 8000
```

## Deploying

Vercel serves this repo as-is from the root — there is no framework and no build
command. Pushes to `main` publish to production; other branches get preview URLs.

The prototype exported from Claude Design (`.dc.html`) is **not** deployable on
its own: it renders through a client-side runtime that expects React to be
injected by the design host. This repo is the compiled, self-contained version —
same visual output, no runtime.

## Quote form → JotForm → AgencyZoom

The quote form keeps the site's own design and posts straight from the browser
to a **JotForm form backend**. JotForm stores each submission and forwards it to
AgencyZoom through the JotForm → AgencyZoom integration on the backing form.
There is no server code in this path.

| | |
| --- | --- |
| JotForm form | `263206076259055`, "BlackSmith Agency – Website Quote Request" (Tom's JotForm account) |
| Website id | `blacksmith-quote-form` |
| Submit URL | `https://api.jotform.com/form-backend/submit` |
| Submission key | in the submit script at the bottom of `index.html` |

The submission key is a public, form-scoped write key meant to sit in page code.
It can only add submissions to this one form; it can't read them and it isn't a
JotForm API key.

Field mapping (the HTML `name`s must match exactly):

| HTML name | JotForm qid | Required |
| --- | --- | --- |
| `first_name` | 2 | yes |
| `last_name` | 3 | yes |
| `email` | 4 | yes |
| `phone` | 5 | yes (10-digit US check in the browser) |
| `coverage` | 6 | yes |
| `details` | 7 | no |
| `consent` | 8 | always sent, `yes` or `no` |

The `website` honeypot is checked in the browser and never sent: a filled value
shows the normal thank-you and nothing is submitted. Any failure (network, a
JotForm error, anything other than `responseCode` 200 with `content.success`
true) shows the office phone number instead of a fake success.

**Changing fields:** add or rename the input in `index.html`, then update the
backend with the full new field list (JotForm's `update_form_backend`). Omitted
fields are removed from the JotForm form, and the submission key may change, so
paste the new key into the submit script.

### JotForm → AgencyZoom

AgencyZoom's JotForm web-lead integration gives the agency an inbound address,
`csmith58_jotform@myagency.email`. The JotForm form has a notification email
to that address on every submission, with all questions and answers, and
AgencyZoom turns each email into a lead. Lead source, pipeline and assignee
are set on the integration in AgencyZoom, not here.

Keep the JotForm question labels (First name, Last name, Email, Phone, …)
stable: AgencyZoom reads the notification email, so renaming a label can
break its field matching.

`api/quote.js` (the earlier direct AgencyZoom API function) is no longer called
by the page. Delete it and `scripts/agencyzoom-ids.js` once the JotForm path is
verified end to end.

## Known gaps

One thing needs a decision before this is a finished production site:

1. **The "Why BlackSmith" section uses a card, not a photo.** The design left an
   empty image slot there (captioned "DFW business owner, shop, or truck yard").
   Until a photo exists, that column is a bone-colored contact card. To swap in
   a real photo, replace the `<aside class="why-card">…</aside>` block with:

   ```html
   <aside class="why-media"><img src="/assets/why.jpg" alt="…"></aside>
   ```

   The `.why-media` styles are already in `styles.css`.

## Hero graphic

The hero uses `assets/texas-hero-transparent.webp` — a Texas map marking The Colony,
generated in Claude Design (`texas-hero-graphic.html`, D3 + us-atlas counties) and
rendered at 1040px for a ~496px display box.

Because the artwork is square and its "The Colony" callout runs to the right edge,
`.hero-media` is `aspect-ratio: 1` and the image uses `object-fit: contain` — a
non-square frame would clip the label.

The design project exports this artwork two ways. The **transparent** variant is the
one in use: the map sits directly on the hero ink, so the 1px frame and its 14px mat
are the only container. The dark-background variant carries its own vignette and dot
grid, which at ~496px reads as a second rectangle inside the frame rather than as
texture.

Anything under `/assets/` is served `immutable, max-age=31536000`. Changing an image
means changing its filename, not overwriting it, or cached clients keep the old one.

`styles.css` is served `max-age=0, must-revalidate`: browsers check it on every
page load (a cheap 304 when unchanged), so a CSS change shows up immediately. It
used to be cached for an hour, which left browsers that had visited before showing
the new `/team` page with the old stylesheet. Both pages link it as
`/styles.css?v=<date>`. Bump that date in `index.html` and `team.html` whenever the CSS
changes, so browsers that still hold the old one-hour copy fetch the new file.

The owner headshot is now used only in the About section.

## Editing notes

- Colors, fonts, and spacing are CSS custom properties at the top of `styles.css`.
- Type scales with `clamp()`; breakpoints are at 1180 / 1160 / 1024 / 900 / 720 / 560px.
  The nav drops at 1160 rather than the prototype's 900 — with six links, the phone
  number and the Get-a-Quote button in the same bar, it runs out of room near 1150.
- The phone number appears in six places (header, hero card, About, quote, footer,
  and JSON-LD) — grep for `9726240239` before changing it.
- The production hostname is hard-coded in `index.html` (canonical, `og:url`,
  `og:image`, JSON-LD `url`/`image`), `robots.txt`, and `sitemap.xml`. Moving to a
  custom domain means updating all three files — grep for `blacksmith-insurance-site`.

## URLs and deployment protection

Production (public, indexable):
**https://blacksmith-insurance-site.vercel.app**

The project has Vercel Authentication enabled at `all_except_custom_domains`. In
practice that gates the per-deployment URLs — the
`blacksmith-insurance-site-<hash>-…vercel.app` links shown in build output return
a 302 to Vercel SSO and carry `x-robots-tag: noindex`. The production alias above
is unaffected: it serves publicly with no auth and no noindex header.

So: share the production alias, not a build-output link. If you need to hand
someone a specific preview deployment, either add them to the Vercel team or turn
off Deployment Protection under Project → Settings.
