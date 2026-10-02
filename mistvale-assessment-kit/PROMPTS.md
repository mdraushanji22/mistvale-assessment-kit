# Prompts

## Honest note first

This file is the prompt log the brief asks for, and I want it to be accurate rather than
impressive-looking. Two things you should know before you read it:

1. **I did not use an image-generation model.** Every image in `images/` was written by me
   as SVG code and rendered to WebP with `sharp`. There is therefore no image-model prompt to
   log — what I have instead is the art direction I wrote for myself, reproduced in §3. Those
   are not instructions I sent to an outside service, and I have labelled them as such rather
   than dressing them up as a model transcript.
2. **The build ran in a single session against the repo's own brief.** The brief is
   `README.md` and `BRAND.md` in `D:\mistvale-assessment-kit\mistvale-assessment-kit`. Those two
   files were my actual specification, and the checks in §4 are derived from them line by line.
   If the graders want a verbatim transcript, the form below is the more reliable route — see
   the note at the end.

## 1. Prompts typed by the user

**Prompt 1 — the brief (summarised; see the caveat below).**

> Read `README.md` and `BRAND.md` in `D:\mistvale-assessment-kit\mistvale-assessment-kit`. They
> are the specification. Turn `index.html` into the premium, accessible Mistvale Tea Co. store
> described there. Preserve the protected blocks exactly, replace the placeholder images with
> your own, and write `NOTES.md` and `PROMPTS.md`.

**Prompt 2 — the follow-up.**

> What did we do so far?

**Prompt 3 — the current one.**

> Continue if you have next steps, or stop and ask for clarification if you are unsure how to
> proceed.

**Prompt 4 — the account feature (summarised; same caveat as above).**

> Add accounts to the store: a way to sign up, log in and log out, plus guest checkout for
> people who do not want an account.

**Caveat, stated plainly:** I cannot reproduce prompts 1 and 4 verbatim. They are not in the
context I am working from any more — I have a summary of what they asked for, not the exact
wording. Rather than invent a plausible-looking transcript, I have written the summary above and
left a space for the real text. **Please paste your original brief into the form**, or paste it
here and I will add it verbatim.

## 2. How I worked, in order

For transparency, since the prompt count is so low:

1. Read `README.md` (requirements R1–R8, protected contracts, deliverable list) and `BRAND.md`
   (visual system, palette, type, motion, accessibility, facts, FAQs, SEO, art direction).
2. Grepped `index.html` for the protected blocks and copied them out verbatim before touching
   anything.
3. Wrote the markup and stylesheet in one pass to `BRAND.md`.
4. Wrote the store logic against `PRODUCTS` and the `API` as the only sources of truth.
5. Wrote the image generator, produced the assets, verified them.
6. Wrote `test-rules.js`, then `check.js`, then `contrast.js`, then the jsdom shopper tests —
   and fixed what they found. Most of the real bugs in §2 of `NOTES.md` came out of this step,
   not out of the build.
7. Added the account feature last, after a second reading of the brief confirmed it was an
   extension and not a requirement — then spent most of that hour on the password handling and
   on `mutate.js` / `mutate-check.js`, which break the finished page ten and sixteen ways on
   purpose to prove the tests can actually fail. That step found four checks of mine that were
   passing without checking anything (§2 of `NOTES.md`).

The equivalent of many small prompts was applied as many small checks.

## 3. Art direction I wrote for the artwork

Self-authored, not sent to an image model. Recorded here because they are the design decisions
the files encode.

**Shared, all eight products and the hero**

- Flat geometric illustration. No photographic realism, no text or lettering inside the image,
  no faces, no hands, no teacups as the subject.
- Brand colours only: cream `#F7F3EA`, parchment `#EFE8DA`, card white `#FFFDF8`, forest
  `#1F3A2E`, maroon `#7A2E3B`, brass `#C9A227`, sage `#6F7F63`.
- Identical framing, camera angle, lighting direction and aspect ratio across the set so the
  shelf reads as one shoot. Subject centred, generous margin, soft long shadow falling to the
  right, faint radial cream glow behind the subject.
- One distinguishing silhouette per product, so the eight are recognisable at thumbnail size.

**Per product**

| File | Brief |
|---|---|
| `p101.webp` | Tall round Assam tin, maroon band, brass rim; loose black tea spilling beside it |
| `p102.webp` | Rectangular masala chai pouch, maroon field; scattered whole spices — cardamom pods, clove, cinnamon — beside it |
| `p103.webp` | Short wide brass-rimmed tin for the Kahwa; loose tea and a saffron-coloured steam wisp |
| `p104.webp` | Kraft-and-cream pouch for the chamomile; three chamomile flowers beside it |
| `p105.webp` | Forest-green tin for the Nilgiri; loose tea and two whole leaves |
| `p106.webp` | Maroon tin for the hibiscus; deep red petals beside it |
| `p107.webp` | Grey-sage tin, lid off and set to one side, empty — sold out, no loose tea |
| `p108.webp` | Wooden sampler box, lid leaning against it, four compartments of different teas |
| `hero.webp` | 16:10. Low hills in three mist layers over a cream sky, sun low and warm, brass horizon line, a small leaf mark; generous empty sky on the left for the headline |

**Output rule:** render at 2× the display size, WebP quality 82, `sharp` metadata checked
afterwards, plus a 10×10 mean-signature comparison across all eight to prove none is a
duplicate of another.

## 4. The specification I worked from, for the record

The brief in this repo is unusually good, and most of the value in the finished page comes from
following it literally rather than from any prompt. The constraints that drove the build:

- One file. Inline HTML, CSS and JavaScript. No libraries, no frameworks, no build step.
- Do not change `PRODUCTS` ids, names or prices. Do not touch the `API` block.
- Do not change the checkout contract or the legal text.
- Business rules R1–R8: prices only from `PRODUCTS`, quantities capped at 5 and at stock, one
  coupon with a minimum and a cap, free delivery measured after the discount, all money in
  paise, sold-out teas unavailable and sorted last, search and filter and sort combining
  correctly, and a pincode check that cannot get stuck.
- `BRAND.md`: the palette, Fraunces and Inter in one request, the type scale, the 15px text
  floor, the 10px corner radius used once, motion limited to named properties and disabled
  under `prefers-reduced-motion`, visible focus, real landmarks, one primary button per view,
  no pop-ups, no countdown, no marquee, and the ten sections in a fixed order.
- Facts, FAQ answers and SEO fields restricted to what `BRAND.md` actually says.
