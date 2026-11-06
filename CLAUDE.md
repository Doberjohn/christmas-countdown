# Christmas Countdown

A Christmas countdown website with three interactive scenes. Each scene is a
self-contained HTML page (inline CSS and JS, Canvas 2D, Web Audio, no build step).
The plan is one site that hosts all three scenes once each is finalised.

## Status

| Scene | Path | Status |
|---|---|---|
| Frosted window | `scenes/frosted-window/index.html` | **Finalised** (the reference for quality) |
| Paper village | `scenes/paper-village/index.html` | Shortlisted, first version, not yet polished |
| Bauble garland | `scenes/bauble-garland/index.html` | Shortlisted, first version, not yet polished |

**The frosted window is the home page (`/`).** There is no root `index.html`: `/` is
rewritten to `scenes/frosted-window/index.html` by `vercel.json` (Vercel) and
`serve.json` (local `npx serve`). Keep the two rewrites in sync. The URL stays `/`, so
the page must only use absolute or inline assets (it does today).

Two other concepts (a wind-up music box and a Greek-folklore "kallikantzaroi" world
tree) were built and **ruled out**. They are not in this repo.

Live prototypes (claude.ai artifacts, private to John):
- Frosted window: https://claude.ai/artifact/FyegFYe2VQ3TS3dk8DKCoJ
- Paper village: https://claude.ai/artifact/A76NyfPCWvoeSjM2SuAKtJ
- Bauble garland: https://claude.ai/artifact/UnU96BNyRv6erCcvsavSUq

## Next steps (in order)

1. **Polish the paper village** with John, the same way the frosted window was polished
   (he gives visual feedback, we iterate). Expect feedback like "looks cheap / 2D school
   graphics", "low res", "too distracting": aim for photographic or crafted realism and
   restraint.
2. **Polish the bauble garland** the same way.
3. **Combine into one site.** Decided: the frosted window is the default (home) scene.
   Not decided yet how visitors move between scenes. Options
   discussed: a scene switcher, or the scene changing with the season (garland in
   November, village through Advent from 1 December, frosted window on Christmas
   Eve/Day). Ask John before building.
4. **Deploy.** John deploys his projects to Vercel (GitHub user `Doberjohn`). A static
   site with no build step is fine.

## Shared behaviour (keep consistent across scenes)

- Count down to **00:00 on 25 December in the viewer's local time**. On 25 December show
  a Christmas Day state. From 26 December count to the next year.
- Seconds change detection must use `Math.ceil(now.getTime() / 1000)`. The countdown is
  `floor((target - now) / 1000)` and `target` is a whole second, so the displayed value
  changes exactly when `ceil(now / 1000)` changes. Using `floor` caused a visible bug
  (seconds freezing and then skipping one) whenever a frame landed exactly on the tick.
  All three pages are fixed.
- Respect `prefers-reduced-motion` (no snow, gusts, drips, twinkle or reveal animations;
  state changes are instant).
- A visually hidden `role="timer"` element carries the countdown text for screen readers,
  updated once a minute.
- Sound only starts after a user gesture (browser autoplay rules) and must always have a
  way to stop it.
- Visible copy uses no em dashes.

The paper village and bauble garland pages also have a "Preview a day" date slider and
a sound toggle. The frosted window had these and John asked to **remove all visible
text and buttons** from it; keep that in mind when polishing the other two (ask first).

## Frosted window: how it works

One four-pane window on a falu-red wall at night. The countdown (days, hours, minutes,
seconds, one per pane) is finger-written in the frost in Caveat 700, over a warm room
seen out of focus. No caption or buttons; only a faint mute icon bottom right.

**Render pipeline (each frame, in `composeGlass`):**
1. `drawInterior`: the room. Static parts are painted once per resize (`paintRoom`),
   lit with a light map and desaturated (`gradeRoom`), then pre-blurred into three focus
   levels (`roomLv.base` / `roomLv.xmas`, [near-sharp, soft, very soft]) with
   `blurCanvas` (canvas `filter: blur()` where supported, a down/up-sampling pyramid
   otherwise, e.g. Safari). Per frame: blend focus levels, then additive firelight
   flicker, low-res flames (`fireLow`), lamp and candle glows, and bokeh-disc lights.
2. Frost layer (`frostLayer`): blurred interior + `frostTex` (haze, grain, ferns) +
   additive light scatter + gust rime (`edgeFrost`) + glints + white rims around the text.
3. Holes are cut with `destination-out`: finger strokes (`renderWipe`), the written units
   (`drawUnit`), water drips (`drawDripHoles`), and the Christmas melt (`applyMelt`).
4. Glass canvas: interior, frost layer on top, drip gloss, then `drawGlassFinish`
   (vignette, faint cool reflection, film grain).
5. A separate fixed full-page canvas draws the snow, cold gusts and powder blown off
   the ledges.

**Key behaviours and their constants:**
- Writing: each unit has two slots (hole + rim canvases) for a left-to-right "written"
  reveal and a fade of the old value. `WRITE` (load/mode change, staggered) and `TICK`.
- Strokes: one smoothed `Path2D` per finger stroke, width `strokeWidth()` (about 2.5% of
  the glass, 3.6% for touch). Clear for `HOLD` (2.6 s), then refreeze over `REGROW` (7 s):
  a crystal tile (`crystalPat`) is cut out of the cleared path so frost re-forms as
  crystals and grains, then the haze closes over it. Do not go back to per-segment
  narrowing: it looked like beaded tubes.
- Drips: spawn from the low points of the written numbers (found by painting the number
  into a small `probe` canvas and scanning columns) and from the lowest point of a stroke.
  Stop-start motion, regrow like strokes.
- Gusts: every 22 to 42 s (first after 8 to 14 s), about 5 s long. Snow streaks
  sideways, powder blows off `.snowcap` and `.sill-snow`, rime builds on the pane edges
  and lingers, wind sound swells with an ice hiss.
- Christmas Day: frost melts outward from each pane centre (corners keep some), the focus
  pulls from the glass into the room, and the room gains a "MERRY CHRISTMAS" bunting
  banner and more presents. Preview it with **`#christmas-day`** in the URL.
- Sound (`startAmbient`): Silent Night (Gruber, 1818, public domain) on a synthesized
  celesta with soft felt-piano chords, low-passed as if heard through glass, plus
  reverb, fire crackle and rumble, and wind. Starts on the first pointer or key press,
  fades in, loops about every 64 s, pauses when the tab is hidden. Mute icon or `M` key.
  Drawing on the glass also makes a soft squeak. Volume was not checked by ear yet.
- Performance: device pixel ratio capped at 1.5. The room is pre-blurred at resize, so
  per-frame work is mostly `drawImage` and a few gradients.

**Design decisions John made (don't undo without asking):**
- Photographic look: camera focused on the glass, room out of focus, bokeh lights,
  grade, vignette, grain. The earlier flat vector room read as "cheap 2D school graphics".
- Full-resolution, soft-edged strokes (an earlier quarter-resolution mask looked pixelated).
- Thinner strokes than the first version.
- No visible text or buttons apart from the mute icon.
- Subtle background sound and subtle gusts: atmosphere, never distracting.

## Paper village (to polish)

A layered paper-cut diorama of a snowy village at dusk: eight paper layers with fibre
texture and soft shadows, parallax on pointer move (drag on touch, gentle idle sway).
24 numbered Advent windows: window N lights on the night of N December (window 24 is in
the church tower); before December all are dark. On Christmas Eve every window opens; on
Christmas Day a gold star rises behind the spire. Tapping a lit window shows one of six
small paper scenes inside. Countdown on a letterpress-style card (Alfa Slab One, Karla).
No sound. Has a "Preview a day" slider. Device tilt is not used (blocked in the
artifact sandbox; it could work on a self-hosted site over HTTPS with permission).

## Bauble garland (to polish)

A pale wall in winter-morning light with gold threads in swags and one glass bauble per
sleep left (about 50 in early November), drawn on canvas with highlights, caps and
shadows; each date always gets the same colour and shape. Baubles swing on touch or drag
(pendulum physics) and chime when tapped (Web Audio). Today's bauble has a red bow and
opens to show one of 35 family activities chosen by date. Christmas Day: one big
sapphire bauble that opens to "Merry Christmas". Big countdown type (Bodoni Moda, Hanken
Grotesk), "Preview a day" slider, sound toggle.

## Hosting notes

The pages were built for claude.ai artifact hosting, whose content security policy only
allows Google Fonts and inline assets (no remote images, no fetch). On your own hosting
those limits go away: fonts can be self-hosted, and the code can be split into modules.
Keep each scene working as a single static page until the combined-site structure is
decided.

## Testing that was done, and what wasn't

- Rendered in headless Chrome at 1440x900 and 390x844 (fallback fonts, since Google
  Fonts did not load in that sandbox). No console errors.
- Not yet checked: real Caveat rendering on device, Safari and Firefox, low-end phones,
  the actual sound mix by ear, long-running stability.
- Useful dev hooks: `#christmas-day` (frosted window). To force a gust quickly, set
  `nextGust` to `performance.now() + 2500` in a local copy.
