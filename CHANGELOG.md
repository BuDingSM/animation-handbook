# Changelog

The main steps of Animation Handbook, newest first, with the number of animations at each one.

## Unreleased

- The Animation finder: saw an animation on a website and don't know its name? Answer three short questions on the home page
  (what moved, when and how) and the animations that match play, with their names. Not here? opens a short GitHub form.
- A Star on GitHub button under every animation page's Copy prompt and, on screens 820px and wider, at the end of its
  top bar; a round star on the home page for phones.
- A Famous effects page: eleven effects people admire on well-known websites, each described in plain words with the
  handbook animations that make something similar and a prompt to build your own. It is linked from the home page's footer.
- A new animation, Ink Spread Reveal (167 in all): ink drops spread and merge until the picture shows, with See inside and
  Compare from the start.
- A new animation, Pixelate Transition (168 in all): the page breaks into big pixels, then sharpens into the next, with
  See inside and Compare from the start.
- A new animation, Before/After Slider (169 in all): drag a line across two pictures to compare them, with See inside and
  Compare from the start.
- A new animation, Magnifier Lens (170 in all): a lens follows your pointer and shows the picture up close, with See inside
  and Compare from the start.
- A new animation, Card Deck Swipe (171 in all): swipe the top card left or right and the next one rises, with See inside
  and Compare from the start.
- A new animation, Water Ripple (172 in all): ripples spread from your finger and bend the picture below, with See inside
  and Compare from the start.
- A new animation, Audio Visualizer (173 in all): bars dance to the music, from bass to treble, silent until Hear it is
  pressed, with See inside and Compare from the start.
- A new animation, Ken Burns Effect (174 in all): photos slowly zoom and pan, then fade into the next, with See inside and
  Compare from the start.
- A new animation, Fire Effect (175 in all): flames flicker and rise as heat climbs and cools, with See inside and Compare
  from the start.
- A new animation, Lightning (176 in all): jagged bolts strike from the clouds and light up the sky, with See inside and
  Compare from the start.
- A new animation, Preloader (177 in all): a logo and a percentage count up, then the screen lifts away, with See inside and
  Compare from the start.
- A new animation, Easing (178 in all): speed changes through a move instead of staying constant, with See inside and
  Compare from the start.
- A new animation, Overshoot (179 in all): a pointer swings past its target and settles back, with See inside and
  Compare from the start.
- A new animation, Squash and Stretch (180 in all): a ball squashes as it lands and stretches as it takes off, with See inside and
  Compare from the start.
- A new animation, Anticipation (181 in all): a shape pulls back before it shoots forward, with See inside and
  Compare from the start.
- A new animation, Follow-Through (182 in all): a flag and tail keep swinging after the body stops, with See inside and
  Compare from the start.
- A new animation, Bottom Sheet (183 in all): a panel slides up from the bottom of the screen, with See inside and
  Compare from the start.
- A new animation, Popover (184 in all): a small panel with an arrow pops out beside a button, with See inside and
  Compare from the start.
- A new animation, Tabs (185 in all): an underline slides across as tab contents swap, with See inside and
  Compare from the start.
- A new animation, Filter Chips (186 in all): chips toggle filters and fade the cards that no longer match, with See inside and
  Compare from the start.
- A new animation, Stepper (187 in all): plus and minus buttons change a number with a small pop, with See inside and
  Compare from the start.
- A new animation, Pagination (188 in all): a marker slides across page numbers as the list swaps, with See inside and
  Compare from the start.
- A new animation, Breadcrumbs (189 in all): a trail shows where you are and takes you back up, with See inside and
  Compare from the start.
- A new animation, Inline Validation (190 in all): a field says right or wrong while you type, with See inside and
  Compare from the start.
- A new animation, Lightbox (191 in all): a photo opens large over a dimmed page, with See inside and
  Compare from the start.
- A new animation, Hotspots (192 in all): pulsing dots on a picture open notes about each part, with See inside and
  Compare from the start.
- A new animation, Countdown Timer (193 in all): digits roll as the time left ticks down, with See inside and
  Compare from the start.
- A new animation, Text Mask (194 in all): heavy letters cut out to show a moving seascape inside, with See inside and
  Compare from the start.
- A new animation, Underline Animation (195 in all): an underline grows from the left when you hover a link, with See inside and
  Compare from the start.
- A new animation, Wiggle (196 in all): a bell jitters in small irregular shakes, with See inside and
  Compare from the start.
- A new animation, Ping-Pong Loop (197 in all): a shape travels out and comes back along the same path, with See inside and
  Compare from the start.
- A new animation, Repeater (198 in all): one shape is copied around a circle and turns, with See inside and
  Compare from the start.
- A new animation, Kaleidoscope (199 in all): an eight-way mirrored pattern turns and spreads, with See inside and
  Compare from the start.
- See inside and Compare on every animation page: See inside shows the parts that make the effect (guides, hidden layers,
  the numbers made visible), and Compare shows it beside a version without its key idea.
- Famous effects becomes a gallery: each effect opens in a pop-up with our own working version of it, and its address
  opens it directly. The page is linked from the home page's top bar too.
- Every card preview on the home page is drawn again in one bright, consistent style, with one color for each category.
- The home page: a row of chips under Good places to start for the seven groups (press one to see that group's
  animations; its address is `?cat=` and the group's name), a Famous effects band, and How it works in new words
  (Pick an animation, See it move, Copy the prompt).
- New README pictures: a wall of six animations, three of them new, and a recording of today's home page.
- The home page's search knows more everyday words: other names for an animation (lightbox, snackbar, navbar, tabs), common
  misspellings (paralax, accordian) and British spellings (colour, grey). When nothing matches, Ask for it opens a short
  GitHub form with your words filled in.
- After you copy a prompt, the Star on GitHub button under it lights up once.
- Without JavaScript, the home page lists every animation as a plain link under its group, so visitors and search engines
  that run no script can still reach every page.
- The AI skill (`npx skills add`) and the Claude Code plugin, both added after 1.0.0, are removed: every page's Copy prompt
  does that job. `llms.txt` stays.

## 1.0.0 — 2026-10-10

The same 166 animations, easier to reach and to work on.

- The README's catalog links every animation to its live demo, and each page's README starts with a link to its demo.
- The 12 demos that follow the pointer or react to a hover also work from the keyboard.
- New pictures in the README.
- Opened from disk, without a server, the pages link to each other.
- A 404 page: a mistyped address leads to the right page, to the full list or to the search. GitHub Pages now serves
  the files as they are, so no extra pages are built from the repository's notes.
- A code of conduct, a security policy and the site font's license, and the tests run on GitHub Actions for every push
  to main and every pull request.

## 2026-10-02 — 166 animations

- Eight new animations, among them Dynamic Island, Liquid Glass, Shatter Effect and Text Particles.
- The README opens with a moving wall of six animations.

## 2026-10-01 — 158 animations and a new home page

- 29 new animations.
- The home page asks "What do you want to animate?": a search box, nine places to start from (Buttons, Text,
  Backgrounds and more) and a Copy prompt button on every card.

## 2026-09-30 — the guided-steps pages

- Every demo page has the same three steps: Watch it, Try it with a few plain settings, and Copy the prompt to paste
  into an AI assistant. The pages share one stylesheet and one script, and the site font is Schibsted Grotesk.
- Tests for every page and its README, and a browser check that loads each page at several screen sizes.

## 2026-08-21 — 129 animations

- Five new scroll animations, and fixes to the scroll demos.

## 2026-07-13 — 124 animations and a new name

- 24 new animations.
- Animation Atlas becomes Animation Handbook. The home page shows a small live preview on every card, and every demo
  takes the same look.

## 2026-05-22 — 100 animations

- The first 100 animations, in seven categories, live on GitHub Pages.

## 2026-05-19 — the start

- The project starts with Parallax Depth-of-Field and Parallax Scrolling.
