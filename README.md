# Bowl

A dog feeding calculator that works out the day's food from breed, target adult
weight and age, splits it into meals, and lists the points that actually change
the number. Installable to a phone home screen, works offline, no build step.

## Deploy

```bash
git init
git add .
git commit -m "Bowl"
git branch -M main
git remote add origin git@github.com:<you>/bowl.git
git push -u origin main
```

Then **Settings → Pages → Source: Deploy from a branch → `main` / `/ (root)`**.
It lands at `https://<you>.github.io/bowl/`.

All paths are relative, so a project subpath works without configuration.
`.nojekyll` is present so nothing gets swallowed by Jekyll.

## Add to home screen

- **iOS** — open in Safari, Share → Add to Home Screen. Runs full screen, no
  browser chrome.
- **Android** — Chrome will offer Install, or use the menu → Add to Home screen.

## How the number is worked out

The output is **grams per day**, divided by the number of meals. Everything runs
through calories, so the one input that really matters is the food's energy
density (kcal/100 g, on the bag as metabolisable energy).

**Puppies** use two published equations against today's weight and the expected
adult weight:

- NRC (2006): `MER = 130 × BW^0.75 × 3.2 × (e^(−0.87p) − 0.1)`, where
  `p = current weight / expected adult weight`
- Klein et al. (2019), fitted to privately owned puppies rather than colony
  dogs: `ME (MJ) = (1.063 − 0.565p) × BW^0.75`, roughly 20% lower

Those two form the band. The point estimate sits between them, weighted by
breed size: NRC matched Labrador-sized puppies well but overshot Yorkshire,
Norfolk and miniature Schnauzer puppies in the trials, so small breeds land near
Klein and large breeds near NRC.

**Adults and seniors** use resting energy requirement, `RER = 70 × kg^0.75`,
times a maintenance factor — 1.8 intact, 1.6 neutered, ×0.9 for seniors. That
lands at 95–115 kcal/kg^0.75, which is where FEDIAF's guidance sits.

Activity (×0.88 / 1 / 1.14) and body condition (×0.85 / 1 / 1.1) scale the
result. For an overweight dog, enter the weight it should be.

**Cross-check.** The card also converts the chart-style bands from
[PetMD's feeding guide](https://www.petmd.com/dog/nutrition/are-you-feeding-your-dog-right-amount)
into grams of your food and shows whether the two agree. They usually do for
adults; for puppies the printed brackets are wide enough (one row covers six
weeks to three months) that they often don't, which is the reason for the
calculation.

Life stage: puppy under 12 months, or under 24 for large and giant breeds still
below adult weight; senior from 7 years for giant, 8 large, 10 medium, 11 small,
12 toy. Meal frequency follows PetMD: three a day under four months, four to
five for toy puppies, two to three thereafter by size.

### The day: sleep and exercise

The clock card divides 24 hours into overnight sleep, daytime naps, meals and
walks. Meals are spaced evenly through the waking hours; each one is followed by
a walk or a play session, then naps sized to make up whatever sleep the night
doesn't cover. Gaps are ordinary awake time. Wake and bed times and the number
of walks are set under *The day*.

Sleep totals use the standard veterinary bands — 18–20 h under three months,
16–18 h to six months, 14–16 h to a year, 12–14 h adult, 16–18 h senior.

Puppy exercise uses the Royal Kennel Club's five minutes per month of age, per
outing, up to twice a day. That rule has no trial evidence behind it and the
PDSA says so plainly, so Bowl treats it as a **ceiling on formal lead walking**
rather than a target, and pairs it with an equal amount of unhurried sniffing,
as suggested in *Veterinary Practice* (2024). Adults get a size-based band,
scaled down for seniors and shifted by the activity setting.

### Sources

- National Research Council (2006), *Nutrient Requirements of Dogs and Cats*
- Klein, Thes, Böswald & Kienzle (2019), *J Anim Physiol Anim Nutr* 103:1952
- Alexander, Colyer & Morris (2017), *J Nutr Sci* 6:e26 — Yorkshire terrier growth
- Bradley et al. (2021), *Animals* 11(5):1380 — Norfolk terrier growth
- FEDIAF Nutritional Guidelines (95 and 110 kcal/kg^0.75 adult maintenance)
- PetMD, *How Much To Feed Your Dog* — the cross-check charts
- The Royal Kennel Club — puppy and dog walking tips (the five-minute rule)
- PDSA, and *Veterinary Practice* (July 2024) — on the rule's evidence base
- Krontveit et al. (2012) — exercise, terrain and hip dysplasia in large breeds
- AKC, VCA and Vets4Pets — sleep requirements by life stage

## Saved data

Dogs are stored in `localStorage` on the device, so they persist across visits
and app launches; nothing is sent anywhere and there's no account. Clearing site
data clears them, so there's a **Copy backup** / **Restore backup** pair at the
bottom of the *Your dog* panel that round-trips everything as JSON. The app also
calls `navigator.storage.persist()` on load, which asks the browser not to evict
the data under storage pressure.

For a puppy, the weight is the input that goes stale. If it hasn't been updated
in ten days the card says so and offers a shortcut to change it.

## Layout

One responsive layout, three shapes:

| Width | Shape |
|---|---|
| under 820px | Single column. Setup lives in an off-canvas drawer behind the **Setup** button; closes on tap-outside, Esc, the ✕, or on save |
| 820px and up | Sidebar becomes a permanent sticky column, ~296px, widening to 352px past 1080px |
| 1180px and up | Content splits in two — food and notes on the left, the day's clock on the right |
| 1500px and up | Notes flow into two columns |

The breakpoint is 820px so iPad portrait (834pt) gets the sidebar rather than the
drawer. It's defined twice — in the CSS media queries and in the `mqWide`
matchMedia in the script — so change both together if you move it.

## Files

| | |
|---|---|
| `index.html` | Everything — markup, styles, energy equations, breed table |
| `manifest.webmanifest` | Install metadata |
| `sw.js` | Offline shell cache |
| `icons/` | App icons — `icon.svg` (rounded), `icon-square.svg` and `icon-maskable.svg` are the sources; the PNGs are rasterised from them |

## Changing things

**Breeds** — the `BREEDS` array near the top of the script: `["Name", size,
[min lb, max lb]]`, where size is `toy` / `small` / `medium` / `large` /
`giant`. Size drives meal frequency and the senior threshold; the pounds set the
weight slider's bounds.

**Equations** — `puppyKcal` and `adultKcal` hold the whole engine; `NRC_W` sets
how far each breed size leans toward NRC. `ADULT`, `SENIOR` and `PUP` are only
the cross-check chart, in cups per day.

**After any edit**, bump `CACHE` in `sw.js` (`bowl-v1` → `bowl-v2`) or installed
copies will keep serving the old version.

## Caveat

It's a starting point, not a prescription. Check the chart on your own food's
bag, weigh the dog regularly, feel for ribs, and let your vet have the final
word — particularly for a growing large-breed puppy, where feeding for fast
growth causes real joint problems.

## Licence and attribution

The app icon is supplied artwork, kept as authored. `icons/icon.svg` (rounded),
`icon-square.svg` (iOS applies its own mask) and `icon-maskable.svg` (art at 78%
for Android's safe zone) are the three sources; the four PNGs are rasterised
from them, so re-export after any edit. If the artwork derives from a licensed
source, add that source's credit next to the comment at the top of each SVG.

Note the icon's background is `#143A2E` while the app chrome is `#0F2A26` — two
shades of the same green. If the splash screen looks off against the home-screen
tile, change `theme_color` and `background_color` in `manifest.webmanifest` to
match the icon.

Everything else here is yours to do as you like with.
