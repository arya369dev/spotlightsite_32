# Spotlight website v2

*Where marketing meets intelligence.*

The site opens on a dark, cinematic stage where your intro film plays. When you scroll, it moves into a light, pastel world lit by stage-light colours. It is a static site with two small Vercel functions. You can deploy it to Vercel as it is.

---

## 1. Folder layout

Put the files exactly like this. Everything sits in the root folder except the two files inside `api/`.

```
spotlight/
├── index.html            Home page
├── privacy.html          Privacy policy        → /privacy
├── terms.html            Terms & conditions    → /terms
├── 404.html              Custom "not found" page
├── style.css             All styles
├── main.js               Motion, cursor, consent, analytics, form
├── posts.json            Blog / News / Events content (source of truth)
├── robots.txt            Crawler rules (AI crawlers allowed)
├── llms.txt              Summary for AI assistants
├── site.webmanifest      App icon / colour info
├── vercel.json           HTTPS, security headers, clean URLs, routes
├── package.json
├── env-example.txt       List of secret settings for Vercel
├── intro.mp4 / intro.webm        Intro film (with sound), compressed, two formats for every browser
├── hero-loop.mp4 / hero-loop.webm Silent slow-motion loop of the beam, seamless
├── hero-poster.jpg       First frame shown while video loads
├── intro-poster.jpg
├── og-image.jpg          Social preview (1200×630)
├── logo-full.png  logo-mark.png  logo-wordmark.png
├── favicon-32.png  favicon-180.png  favicon-512.png
├── font-cormorant-500.woff2  font-cormorant-600.woff2  font-manrope.woff2  font-cinzel-500.woff2
│                         Self-hosted fonts (faster + no Google Fonts privacy issue)
└── api/
    ├── contact.js        Audit form handler (validation, spam, n8n, email, Meta CAPI)
    └── content.js        Insights pages, sitemap.xml, feed.xml, llms-full.txt
```

`sitemap.xml`, `feed.xml`, `llms-full.txt` and every `/insights/...` page are **generated on the fly** by `api/content.js`. Each new post is added to them automatically, so there are no files for these.

---

## 2. Deploy to Vercel (about 10 minutes)

1. Create a new GitHub repository and upload all the files, keeping `api/` as a folder.
2. In Vercel, choose **Add New → Project**, import the repo, and keep the framework preset as **Other**. You don't need a build command.
3. In **Settings → Environment Variables**, add the values from `env-example.txt`. At minimum, add:
   - `SITE_URL`: your domain
   - `CONTACT_WEBHOOK_URL`, **or** `RESEND_API_KEY` + `LEAD_NOTIFY_EMAIL`. Until one of these is set, the form shows visitors your email address instead.
4. Deploy. Then add your domain under **Settings → Domains**.

### Replace the placeholder domain and email
The code uses `spotlight.agency` and `hello@spotlight.agency` as placeholders. Use find & replace across all files to swap in your real domain and email. Also fill in the bracketed items `[registered company name]`, `[registered address]`, `[city]` and `[name]` in `privacy.html` and `terms.html`. Ask a lawyer to review both pages before launch.

### Turn on analytics and Meta ads
Open `main.js` and fill in the `CONFIG` block at the top:

```js
GA4_ID: "G-XXXXXXX",          // Google Analytics 4
META_PIXEL_ID: "123456789",   // Meta Pixel
TURNSTILE_SITE_KEY: "",       // optional Cloudflare Turnstile
```

These IDs are **public by design**, so they are safe in the browser. The secret keys (the Meta CAPI token and the Turnstile secret) go **only** in Vercel environment variables.

---

## 3. The launch checklist, and where each item lives

| Item | How it's done |
|---|---|
| Privacy policy | `privacy.html`, written for India's DPDP Act 2023 + GDPR, with a cookie table |
| Terms & conditions | `terms.html` |
| Remove frontend secrets | No keys in HTML/JS. All secrets are env vars read only by `api/*.js` (see `env-example.txt`) |
| Enforce HTTPS | Vercel auto-redirects HTTP → HTTPS. `vercel.json` adds HSTS (2 years, preload) + CSP `upgrade-insecure-requests` |
| Cookie consent banner | Accept / Reject / Choose. GA4 and the Meta Pixel **don't load** until consent is given. "Cookie settings" in the footer re-opens it |
| Meta titles/descriptions | A unique title and description on every page, including each generated post |
| Social preview image | `og-image.jpg` 1200×630 + Open Graph + X/Twitter tags + `og:video` |
| Favicon | 32, 180 (Apple) and 512 px + `site.webmanifest` |
| Sitemap & robots.txt | `/sitemap.xml` (auto, includes every published post) + `robots.txt` |
| Image alt text | Every content image has alt text. Decorative images use `alt=""` |
| Image compression | PNGs quantised (~40% smaller), JPGs optimised. Intro video 1.8 MB → 0.7 MB (MP4) / 0.39 MB (WebM), background loop 63–112 KB, fonts self-hosted as WOFF2 |
| Page load speed | Lighthouse (tested before delivery): desktop **100** performance, mobile **89** on a plain local server with no compression (Vercel adds Brotli + edge caching, so expect higher). Accessibility, Best practices and SEO all **100**. Poster and fonts preloaded, fonts `display=swap`, insights load only when scrolled near, videos pause off-screen, no frameworks |
| Colour contrast | Text colours chosen and checked for WCAG AA (4.5:1+) on both light and dark backgrounds |
| Mobile responsiveness | Fluid type, 3 breakpoints, touch-friendly 44px+ targets, no sideways scroll |
| Custom 404 page | `404.html`, with a swinging spotlight that follows the cursor |
| Broken link fixes | All internal links checked automatically before delivery |
| Form validation | Live, friendly errors in the browser **and** strict validation on the server |
| Spam protection | Hidden honeypot field, time trap, link-count filter, rate limit, optional Cloudflare Turnstile |
| Analytics setup | GA4 + Meta Pixel (consent-gated) + Meta Conversions API (server side, deduplicated) |
| Single clear CTA | Every button leads to one action: **Get your free growth audit** |

**Also included:** structured data (Organization, FAQPage, VideoObject, BlogPosting, NewsArticle, Event, Breadcrumbs), `llms.txt` + `llms-full.txt`, an RSS feed, a skip link, visible keyboard focus, and `prefers-reduced-motion` support.

---

## 4. Blog / News / Events and automation (n8n or an AI agent)

All content lives in **`posts.json`**. Each item looks like this:

```json
{
  "slug": "my-post-url",
  "type": "blog",                 // blog | news | event
  "status": "pending_review",     // draft | pending_review | published
  "title": "…",
  "excerpt": "One or two sentences for cards and search results",
  "date": "2026-10-03",
  "author": "Spotlight Team",
  "tags": ["SEO"],
  "readingTime": 5,
  "cover": "",                    // optional image URL
  "coverAlt": "",
  "body": "Markdown text: ## headings, lists, **bold**, [links](https://…)",
  "faq": [{ "q": "…", "a": "…" }],   // optional, becomes FAQ rich results
  "event": { "start": "2026-11-12T16:00:00+05:30", "end": "…", "mode": "online", "location": "…", "registerUrl": "…", "price": "Free" },
  "generatedBy": "ai"             // human | ai
}
```

**Only `"status": "published"` ever appears on the site.** That status is the human approval gate.

### Recommended n8n workflow
1. **Trigger:** a schedule (e.g. every Monday), an RSS/news source, or a form.
2. **AI node** (OpenAI / Anthropic): drafts the post in the JSON shape above, with `status: "pending_review"`.
3. **GitHub node:** reads `posts.json`, appends the new item, and commits it back.
4. **Notify** (email, Slack or WhatsApp): "New draft ready for review".
5. **A person reviews and edits** the post on GitHub, changes the status to `published`, and commits.
6. Vercel redeploys automatically within about 30 seconds. The post goes live at `/insights/<slug>`, and the sitemap, RSS feed, `llms-full.txt` and homepage cards all update by themselves.

The Markdown converter escapes all HTML, so AI-written text can't inject scripts into your pages.

### Leads into n8n
Set `CONTACT_WEBHOOK_URL` to an n8n **Webhook** node. Each audit request arrives as:

```json
{ "type": "audit_request", "received_at": "…", "lead": { "name": "…", "email": "…", "phone": "…", "website": "…", "service": "…", "message": "…" },
  "source": { "page": "…", "utm_source": "…", "utm_campaign": "…", "fbclid": "…" }, "event_id": "lead-…" }
```

If you set `CONTACT_WEBHOOK_SECRET`, check the `x-spotlight-signature` header (HMAC-SHA256 of the body) in n8n. From there you can push to a CRM or Google Sheets, auto-reply, or start the audit research agent.

---

## 5. Meta ads readiness
- Pixel `PageView`, `ViewContent` (when someone flips a service card), `Contact` (when someone clicks a CTA) and `Lead` (when someone submits the form).
- Conversions API `Lead` is sent from the server with the **same `event_id`**, so Meta counts each lead once. Emails and phone numbers are SHA-256 hashed first.
- UTM tags, `fbclid` and `gclid` are captured on arrival and attached to the lead.
- Visitors who click an ad (any UTM or fbclid in the URL) **skip the intro film** and land straight on the offer.
- Use `META_TEST_EVENT_CODE` to watch events in Events Manager → Test events.

---

## 6. Motion & accessibility notes
- **Intro film:** plays once per visit and starts muted. It has *Sound on* and *Skip intro* buttons, and **Esc** skips it. When it ends, the film "irises out" into the lamp. *Watch the intro with sound* replays it.
- **Custom cursor:** a gold ring plus a soft coloured stage-light glow that changes colour with each section. It only appears on mouse/trackpad devices, so touch devices keep normal behaviour.
- **Playcards:** they tilt with the cursor and flip with a click, tap, Enter or Space. **Esc** flips back. The hidden side is made `inert` so screen readers only read the visible side.
- If a visitor's device asks for reduced motion, the film, particles, tilt and scroll effects are switched off.

---

## 7. Editing quickly
- **Text:** edit `index.html` directly. The FAQ appears twice, once on the page and once in the JSON-LD at the top, so update both.
- **Colours:** change the `:root` variables at the top of `style.css`.
- **Services:** each card is an `<article class="playcard">` in `index.html`.
- **The sample posts and events** in `posts.json` are placeholders. Replace them, or set their status to `draft`, before launch.
