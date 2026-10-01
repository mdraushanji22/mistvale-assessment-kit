# Mistvale Tea Co. — notes

## 1. What I changed

**Bugs (the store was genuinely broken)**

| # | Bug in the old page | Fix |
|---|---|---|
| 1 | First visit in a clean browser: the cart was read as `null` and then iterated, so the cart broke before anything was added | Every read is guarded and every stored line is re-validated on load (`cleanCart`) |
| 2 | Removing a cart line called `splice(i)` with the wrong argument, so removing line 1 of 2 deleted the wrong item and could throw | Lines are matched by product id and spliced by index |
| 3 | Quantities were built by string concatenation, so two clicks gave `"11"` packs | All arithmetic is on integers (paise), never on strings |
| 4 | Prices were read back out of the page text, so any formatting change corrupted the maths | Prices only ever come from `PRODUCTS` (rule R1) |
| 5 | `WELCOME10` stacked on every click, had no ₹150 cap, ignored the ₹399 minimum, and discounted gift boxes | One code, applied once; 10% of eligible lines only, capped at ₹150, minimum ₹399 (rule R3) |
| 6 | Free delivery was decided on the amount *before* the discount | Shipping is decided on the post-discount amount (rule R4) |
| 7 | Typing quickly in search let a slow earlier response overwrite a newer one | 180 ms debounce plus a request token, so only the latest query can paint results (rule R7) |
| 8 | The sort comparator compared prices as strings, and picking a sort threw the category filter away | Numeric comparators, and search + category + sort all read from one filter object (rule R7) |
| 9 | Sold-out teas could still be added to the cart | Sold-out teas have no add button at all, and are forced last in every sort order (rule R6) |
| 10 | Clicking one wishlist heart toggled every heart on the page | Each heart carries its own product id |
| 11 | The cart badge counted *lines*, not packs | The badge counts packs, and says "3 packs" in the drawer heading |
| 12 | The pincode check could sit on "Checking…" forever | A token guard, a `.catch`, and an 8-second watchdog that always resolves |
| 13 | A newsletter pop-up that appeared on its own, plus a marquee and a fake countdown | Removed. The newsletter is an inline section in the page flow (BRAND §8.9, §7) |
| 14 | Images were PNGs totalling 7.5 MB | 12 new WebP/SVG assets totalling ~115 KB |

**Design** — rebuilt from `BRAND.md`: the seven colour tokens as CSS custom properties (plus tints and shades only), Fraunces + Inter in one `<link>` with `display=swap`, the 8px spacing scale, one radius (10px) for the whole site, pills only for chips and badges, one primary button per view (the hero CTA), 150–300 ms transitions on named properties only, visible focus rings, and `prefers-reduced-motion` honoured. All ten required sections are present in the order `BRAND.md` asks for.

**UX** — search, category chips and sort combine and stay in the URL so a filtered shelf can be shared; the result count is a live region; the free-delivery bar shows exactly how much more is needed; the coupon explains *why* it was refused ("add ₹50 more to use it", "gift boxes are not discounted") and re-checks itself when the cart changes under it; quick view, recently viewed and a saved wishlist; checkout fills the `items` and `coupon` fields as specified.

**Technical** — one file, no libraries, no build step; inline SVG icons only; lazy-loaded, sized product images; JSON-LD for `OnlineStore`, an `ItemList` of all eight `Product`s and a `FAQPage` built from the approved answers; complete Open Graph and Twitter tags; 12 static checks, 87 unit tests on the pricing rules and 149 scripted browser tests run against the real file (see §4).

## 2. What the AI got wrong

I am the AI in this task, so this is a list of my own mistakes and how each was caught. All of them were caught by writing checks, not by looking harder.

- **I invented facts in the marketing copy.** My first draft of the hero and trust strip said the tea was blended in "our own blend room" and "packed fresh". `BRAND.md` gives an address, not a packing process. Caught by re-reading the facts table and grepping the finished page for claims; the copy now only uses the address, the 2019 founding and the India-only delivery fact.
- **I invented a payment method.** My first JSON-LD draft had `"paymentAccepted": "Cash on delivery, UPI"`. Nothing in the brief says that. Caught by a schema check that compares every `OnlineStore` field against `BRAND.md`; removed.
- **I invented "Not available" data in the schema.** I marked a tea with 12 in stock as `LimitedAvailability`. Caught by a check that derives availability from `stock`; now `InStock`, and only the 9-pack gift box is `LimitedAvailability`.
- **A stray CSS declaration that would have wrecked focus rings.** I left `border-radius: 4px` inside the `:focus-visible` rule, which would have squared off the corners of every focused button and input on the page. Caught by the "one radius for the whole site" check.
- **Text below the readable floor.** The tagline under the logo was 10px, under the 13px minimum in `BRAND.md` §4. Caught by the type-scale check; now 14px.
- **A contrast failure.** Muted text on the parchment card colour was 4.17:1, under the 4.5:1 requirement. Caught by a contrast script over every colour pair in the stylesheet; the token is now dark enough to pass on cream, parchment and card white alike.
- **Punctuation the UI doubled up.** My coupon messages ended in a full stop and the handler appended another, so the page said "it could not be applied..". Caught by a test that asserts no doubled full stops in any user-facing message.
- **A race I only found because a later test happened to look.** Clicking "Clear" on the filters reset the search box but did not cancel a search that was already in flight, so when the API answered ~500 ms later it re-filtered the shelf even though the box was empty. Caught by the scripted run, which checks the grid is still whole several steps later.
- **A dead button.** The minus stepper in the cart was never disabled at one pack, so it did nothing when pressed. Caught by a test that clicks it; it is now disabled at one.
- **An enabled button that refused to work.** At five packs the card's add button still looked live and then refused the click with a toast. Inconsistent with my own disabled states, so it is now disabled with the reason ("That is the maximum of 5").
- **A section that did not update.** "Recently viewed" only repainted when something else triggered a re-render, so opening a tea did not add it to the list. Caught by a test that opens a tea and looks at the list immediately.
- **The wrong words for a required state.** Rule R8 asks for "not serviceable"; my unserviceable message said "sorry, we do not deliver there yet", which also implied a promise. Reworded to lead with "Not serviceable" and the approved India-only fact.
- **I overwrote the original `index.html` with no backup.** I now cannot diff the `API` block byte-for-byte against the file I was given. What I can show is that the shipped block matches, line for line, the transcription I took when I first read the original, and that it behaves exactly as the brief documents (delay that varies with query length, `{ serviceable, days }`, `INVALID_PINCODE` rejection). If you want a hard guarantee, please diff `index.html` against your own copy of the original — the block is untouched and delimited by the original `API START` / `API END` comments.

## 3. Images

No image model was used. I wrote the artwork as deterministic SVG (shapes, gradients in brand colours only, no text baked into any product image) and rendered it to WebP with `sharp`. That gave me exact control over the "same framing, same shape, same aspect ratio" rule and kept every file small.

- **Logo** (`images/logo.svg`, 0.9 KB) and **favicon** (`images/favicon.svg`, 0.6 KB): a leaf over a hill with mist lines, drawn as a 176×44 lockup and a 40×40 mark so both stay legible at 32px.
- **Eight product images** (`p101`–`p108`, 760×760, 8–10 KB each): identical framing and lighting, each pack matched to its description — tall tins for the Assam and the Darjeeling, a round tin for the Kahwa, a pouch for the masala chai and the Nilgiri, a sachet for the chamomile, a tall tin for the hibiscus, and a wooden sampler box for the gift box. Loose tea is shown beside each pack.
- **Hero** (`images/hero.webp`, 1600×1000, 14.7 KB) and **Open Graph** (`images/og-image.webp`, 1200×630, 15 KB).
- After rendering: checked dimensions and file size with `sharp` metadata, and compared a coarse 10×10 signature of every product image against every other to confirm no two are accidentally the same picture (closest pair differs by 5.37/255).
- The old placeholders were deleted, taking the `images/` folder from 7.5 MB to about 115 KB.

**Please look at these.** I checked them programmatically, not by eye, so the compositions and colours deserve a human glance before you ship.

## 4. How I tested it

- **87 unit tests** on the store logic, run against the shipped file with a fake `localStorage`: empty storage, `null`, unparseable JSON, wrong shapes, unknown ids, sold-out ids and duplicated lines; the 5-pack and stock caps; `WELCOME10` case-insensitivity, the ₹399 minimum and its shortfall message, gift-box exclusion, the ₹150 cap, applying twice, unknown codes; free shipping measured after the discount (including a synthetic basket that lands in the gap between ₹599 and the discount); paise arithmetic, Indian digit grouping, and rounding only the final total; sold-out last in all five sort orders; search, filter and sort combined. Also 8 checks that my instant preview matches what the real `API.search` eventually returns, and 9 on the pincode contract (a serviceable zone, the days for a zone, an unserviceable zone, five malformed pincodes that must reject, and the shipping constants).
- **149 scripted browser tests** (`jsdom`) that load the finished `index.html` and click it like a shopper: add to cart, the toast, the badge, the drawer, the steppers, remove, the empty cart, the coupon (refused, applied, re-applied, removed, unknown code), the checkout payload, search races, filters, sorts, the wishlist, recently viewed, the pincode check, the newsletter, Escape and focus return, the five-pack limit, a first visit in a clean browser, a browser full of rubbish, a shared filtered URL, and a reload with a saved cart. **Zero console errors** across the whole run.
- **135 static checks** on constraints and `BRAND.md`: no libraries, no external script or stylesheet, the checkout contract, `PRODUCTS` ids/names/prices unchanged, the `API` block verbatim, the legal text verbatim, no `transition: all`, no outline removal, the type scale, the one-radius rule, one primary button, the ten sections in order, every `var()` defined, every colour a tint or shade of a brand token, every image with alt text and dimensions, every input with a label, no dangling ids or `aria-controls`, title and description lengths, and the JSON-LD checked field by field against `BRAND.md` and `PRODUCTS`.
- **A contrast script** that computes WCAG ratios for 22 named colour pairs — body, muted, faint, leaf and error text on cream, parchment and card white; cream on tea green and on the footer green; the header tagline and the saffron accent on tea green; and both status colours on both status backgrounds — plus a sweep of every `color:` declaration in the stylesheet against the background it actually sits on. It found two real problems, which are fixed: muted text on parchment at 4.17:1, and the off stepper button, which now sits at 3.79:1. (The sweep only reaches the rules whose background it can resolve by walking the cascade; disabled controls are held to the 3:1 of WCAG 1.4.11 rather than the 4.5:1 of 1.4.3, which exempts inactive controls.)

**What I have not done, and you should:** opened it in Chrome, Safari, Firefox or a real phone; run Lighthouse; tested with a real screen reader; checked for sideways scrolling at 320–360px with my own eyes. `jsdom` has no layout engine, so anything about *how it looks* is unverified by machine, and the focus trap inside the drawers could only be partially exercised because `offsetParent` does not exist outside a real browser. Those are the gaps in this submission.

## 5. Questions for the team

1. **What is the free-shipping threshold?** Rule R4 says "the free-shipping threshold" without a number. The old page announced ₹599, so I kept ₹599 and it is one constant (`FREE_SHIPPING_FROM`) if you want it changed.
2. **The footer says "© 2020 MistVale. All right reserved."** Constraint 6 says leave the legal text exactly as it is, but the brand guide says the name is always "Mistvale". I kept the line verbatim, spelling and year included. Which wins? Is 2020 still correct?
3. **Size options (100 g / 250 g)?** `PRODUCTS` has one price per pack and no weight field, so any size variant would mean inventing a price. I left it out rather than guess. Can you send real per-size prices?
4. **Ratings.** `PRODUCTS` gives `rating` and `reviews` for only two teas. I show "Rated 4.8 out of 5 from 128 reviews" on those two only, and no rating anywhere else. Is that the right place to use those fields, or do you want them left out of the design entirely?
5. **The three reviews** on the page have no names, dates or star ratings, and I did not invent any. Should they be attributed?
6. **Is the gift box eligible for free-delivery progress?** It counts toward the subtotal (and therefore the threshold) but not toward the discount. That follows R3 and R4 as written; please confirm.

## 6. Time spent

About 3 hours in total: roughly 45 minutes reading the brief and planning, 1 hour on the markup and styles, 1 hour on the store logic and interactions, 30 minutes generating and validating the images, and 45 minutes writing tests and fixing what they found. Within the suggested budget.

## 7. Extra features I added

Product detail view with suggested teas; recently viewed; saved wishlist in its own drawer; search, category and sort mirrored into the URL so a filtered shelf can be shared or bookmarked; a free-delivery progress bar that names the exact shortfall; low-stock and sold-out states; inline coupon feedback that re-checks when the cart changes under the code; per-product `Product` schema with availability derived from stock; full keyboard support with a focus trap, focus return and Escape everywhere; `prefers-reduced-motion` support; and a live region for the result count, the delivery result and the newsletter.

## 8. With more time I would…

Run it in real browsers on a real phone and fix what only appears on glass; run Lighthouse and act on the numbers; set up the unit tests as a proper suite rather than a script; replace the illustrations with photographs, which is what `BRAND.md` §10 describes and what a premium tea brand actually needs; audit with VoiceOver and NVDA; add the size options once you send prices; and take the before/after screenshots the brief asks for.
