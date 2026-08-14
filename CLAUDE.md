# Maya Family Daycare — Website Build Spec

Context file for Claude Code. Read this before writing any code in this repo.

---

## What this is

A small marketing website for **Maya Family Daycare**, a home-based daycare in Palmdale, CA (Antelope Valley). Built as a favor for a family member — not a portfolio piece, but the code should still be clean.

The site's only job: a parent finds it on their phone, decides in ~30 seconds whether this place is worth a call, and calls.

**Current phase:** first sample build with placeholder content. The owner has not returned the intake questionnaire yet. Everything marked `[PLACEHOLDER]` is invented and must be visibly flagged in the code so it gets replaced later.

---

## Confirmed facts (safe to hard-code)

Pulled from the business's public Google Business Profile.

| Field | Value |
|---|---|
| Name | Maya Family Daycare |
| Address | 5051 Bell Ave, Palmdale, CA 93552 |
| Phone | (661) 313-8262 |
| Email | mayafamilydaycare@gmail.com |
| License # | **197701136** — confirmed by the owner, already public on her own flyer |
| Hours | Mon–Fri 7:00 AM – 5:30 PM · Sat/Sun closed |
| Services listed | Educational services, homework assistance, meal preparation, rest periods |
| Age range | **2 weeks to 12 years** — confirmed on her flyer |

**Do not invent:** license number, license type, prices, staff names, credentials, years in business, testimonials, review counts, capacity, or anything about specific children. If a section needs one of these, use an obvious placeholder and flag it.

---

## Hard constraints

### License number — RESOLVED
The owner supplied it directly: **197701136**. It also already appears on her own printed flyer, so it is public information by her choice.

- Put it in the footer slot that was reserved for it.
- California requires licensed child care to display the license number, and parents look for it — this is a trust signal, not fine print.
- **License *type* is still unconfirmed** (Small vs. Large Family Child Care Home vs. Child Care Center). Do not state the type. The number alone is enough.
- Verify the number against the CDSS Community Care Licensing public search before launch — a typo here is worse than an omission.

### Sensitive information
Before publishing anything in these categories, it gets checked with the repo owner first:
- License number
- ~~Exact street address~~ — **RESOLVED.** Her own flyer publishes 5051 Bell Ave, Palmdale, CA 93552. She distributes it herself, so the address can go on the site. The `settings.showStreetAddress` flag can be flipped to `true`.
- Staff names or photos
- Anything identifying specific children
- Prices

**No photographs of children.** No image showing an identifiable child's face may be committed or used, under any circumstance. See the photo handling section — a committed image cannot be fully removed from a public repo's history.

---

## Architecture

### Single file
One `index.html` with CSS and JS inline. No build step, no framework, no bundler. It gets hosted on GitHub Pages and edited by hand months from now by someone who may have forgotten how it works.

### All editable content lives in one config object at the top
This is the most important structural rule. Everything a human might want to change — every string, every image path — goes in a single `CONTENT` object at the top of the file. The rendering code below never contains a hard-coded string or image path.

Rationale: avoids hunting through markup to change text, avoids the same value drifting out of sync in two places.

Shape:

```js
const CONTENT = {
  images: {
    fachada:     "assets/fachada.jpg",      // front of the house
    salonBebes:  "assets/salon-bebes.jpg",  // infant room, cribs
    salonMain:   "assets/salon-principal.jpg",
    patio:       "assets/patio.jpg",        // covered patio, shade sails
  },
  es: { heroTitle: "...", heroSub: "...", /* ... */ },
  en: { heroTitle: "...", heroSub: "...", /* ... */ },
};
```

Every key present in `es` must exist in `en` and vice versa.

### Images
- Live in `assets/` in this same repo, served by GitHub Pages alongside the HTML.
- Stable, descriptive, lowercase filenames — `patio.jpg`, not `IMG_2084.PNG`.
- Each entry in `CONTENT.images` carries a comment saying what the photo shows.
- **Replacement workflow:** to swap a photo, overwrite the file keeping the same filename. No HTML edit needed. Preserve this property — it's the whole point of the naming scheme.

### Photo handling — required before any image is committed

The owner has supplied 14 real photos. They arrive as raw camera files with meaningless names and must be processed before use.

**1. Look at every image.** Do not work from filenames. Open each one and identify what it actually shows before naming or placing it.

**2. Screen for children's faces — hard stop.**
No photograph showing a child's face may be committed, used, or referenced. If any image contains an identifiable child, flag it by filename, exclude it, and report it. Do not crop or blur to make it usable — exclude it and let the repo owner decide.

This matters more than usual here: **once an image is committed and pushed to a public repo, it remains in git history permanently.** Deleting the file in a later commit does not remove it. There is no clean undo. Screen first.

**3. Compress.** iPhone originals run 3–5MB. Fourteen of those makes a slow site on cell data, which is how nearly every visitor arrives. Target **under 300KB each, max 1600px wide**. `sips` is available on macOS and is sufficient:

```bash
sips -Z 1600 -s format jpeg -s formatOptions 70 input.jpg --out assets/output.jpg
```

**4. Rename descriptively.** Lowercase, hyphenated, describing content: `patio-cubierto.jpg`, `salon-bebes.jpg`, `fachada.jpg`, `area-lectura.jpg`.

**5. Place each photo deliberately**, in the section it actually belongs to. Do not dump them into a generic gallery grid.

**Priority guidance — the outdoor space leads.** The covered patio with shade sails, the playhouse, the ride-on toys, and the water table are the strongest assets she has. In Palmdale summer heat, shaded outdoor play is a genuine differentiator that most home daycares cannot offer. Give the outdoor photos prominence and room; don't bury them in a thumbnail row.

Second priority: the infant room (cribs, changing table) and the main classroom, since those speak to the two distinct parent audiences.

---

## Bilingual (English / Spanish)

Both languages in one file, driven by the `CONTENT.es` / `CONTENT.en` blocks.

**Behavior:**
1. On load, read `navigator.language`. If it starts with `es`, open in Spanish. Otherwise English.
2. A visible **"English / Español"** toggle stays on screen at all times so detection can be overridden. This is non-negotiable — never trap a visitor in a guessed language.
3. Toggling swaps text in place. No page reload, no separate URL.
4. Set `<html lang>` to match the active language and update it on toggle.

**Default is English.** Most parents in Palmdale search in English even when Spanish is the home language — but the ones who want Spanish really want it, which is why the button has to be obvious.

Voice note: the Spanish should read like a Palmdale mom wrote it, not like a translation. Warm, plain, no institutional Spanish.

---

## Responsive — phone first

Design for the phone, then let it expand. Nearly every visitor arrives from Google Maps on a phone; the desktop layout is secondary.

- Base styles target narrow screens. Media queries add complexity for wider ones, not the reverse.
- Tap targets comfortably large — the call button especially.
- Body text readable with no pinch-zoom.
- Single column on phone; multi-column only where it genuinely helps on desktop.
- Test on a real iPhone before calling it done, not just a resized browser window.

---

## Calls to action — static-compatible only

No server, no database, no backend. These work on a static host and are the only ones to build:

- **Call** — `tel:` link. Primary CTA, most prominent thing on the page.
- **WhatsApp** — `wa.me` link with a pre-written message so the parent doesn't have to compose one.
- **Email** — `mailto:`.
- **Directions** — link out to the Google Maps listing.
- **Social links** — if she has them.

**Do not build:** contact forms, booking calendars, waitlist signup, newsletter, parent login/portal. These were explicitly cut. They need a backend and she won't maintain them.

---

## Content structure (first draft — adjust once the questionnaire comes back)

1. **Hero** — name, one-line positioning, big Call button, language toggle.
2. **At a glance** — hours, ages served, location, subsidy acceptance status.
3. **The space** — photo-led. The outdoor area is the strongest asset: shade sails, covered patio, playhouse, ride-on toys, water table. In Palmdale summer heat, shaded outdoor play is a real differentiator — give it room.
4. **Programs / ages** — **2 weeks to 12 years.** Accepting infants from two weeks old is unusual and is a genuine differentiator — most centers start at 6 weeks or later, and parents returning from short leave struggle to find placement. Give it weight. At the other end, 12 years means after-school and homework help. The site must speak to two distinct parents and must not blur them together.
5. **Meals** — meal preparation is offered; specifics TBD.
6. **About the owner** — `[PLACEHOLDER]` until the questionnaire returns.
7. **Testimonials** — `[PLACEHOLDER]`. She currently has zero Google reviews.
8. **Footer** — address, phone, hours, reserved empty license slot, language toggle.

---

## Local context

- **Palmdale is Antelope Valley, not central LA.** An hour north, distinct market. Don't reference LA neighborhoods or LA-wide services as if they're local.
- **Subsidies:** the relevant agency is **CCRC (Child Care Resource Center)**, Lancaster office — Alternative Payment Program, CalWORKs stages, vouchers. Crystal Stairs and Options do *not* serve this area; don't mention them.
- **Transitional Kindergarten** is pulling 4-year-olds out of private daycare statewide. If she has a position on TK-age care, it belongs on the site.
- **Licensing body:** California Department of Social Services, Community Care Licensing (CCL). Not DCF — that's Florida.

---

## Visual direction

Static has nothing to do with looking dated — the ceiling here is set by design effort, not by the architecture. Aim for something that looks intentional rather than templated.

Anchor points: she has an existing pink/purple/blue cartoon logo, a jungle/tree wall motif inside, and bright primary-colored outdoor play equipment. The palette should acknowledge the logo without copying its exact saturation. Warm and human; avoid the generic corporate-daycare stock look.

Final visual direction waits on the questionnaire's style section and any reference sites she sends.

---

## Deployment

- Static site on **GitHub Pages**, served from this repo.
- GitHub Pages requires a **public repo** on the free plan. Keep unconfirmed sensitive data (license number especially) out of the repo until cleared.
- URL will be `username.github.io/repo-name` until a custom domain is bought (~$15/yr, optional, later).
- Editing can be done entirely through github.com in a browser — no local git required for simple text or image swaps.

---

## Open items

- [ ] Owner returns the intake questionnaire (`cuestionario-daycare.html`)
- [x] License number confirmed — 197701136
- [x] Street address cleared for publication
- [x] Hours corrected — Mon–Fri 7:00–5:30 (owner fixed the Friday time on Google too)
- [x] Age range confirmed — 2 weeks to 12 years
- [ ] Real photos received (14 sent) — compress, rename, move into `assets/`. **Check every one for children's faces before committing.**
- [ ] License *type* — still unknown, don't state it
- [ ] Pricing decision — publish or "call for rates"
- [ ] Subsidy/CCRC contract status confirmed
- [ ] Google Business Profile category check — currently "Day care center," may need to be a Family Child Care Home
- [ ] Owner collects first Google reviews (higher enrollment impact right now than the website itself)
