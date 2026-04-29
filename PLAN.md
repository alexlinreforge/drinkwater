# Bloom — A Water Reminder That Grows With You

> A single-page web app where a hand-illustrated flower wilts as you forget to drink, and blooms when you do. Deployed on Vercel as a static site. No backend, no accounts — just you and your flower.

---

## 1. The concept

You arrive at the page and meet **one flower in a ceramic pot**, centered on a soft, warm background. A subtle ring around the pot is your hydration timer. As minutes pass without a drink, the flower's posture changes: petals soften, the stem gently bows, color desaturates. When the timer hits zero, the flower visibly droops and a soft chime plays. A **watering can** appears at the top of the screen, and you **click-and-drag it over the flower to pour** — water droplets fall, the flower springs back up, and a new petal, leaf, or bud may appear.

Over a day, your flower **grows from a sprout to a full bloom**. Hit your daily goal, and it produces a second flower, then a third — a small bouquet by week's end. Miss too many reminders in a row and a petal gently falls (recoverable, never punishing).

The reminder is the interaction. There is no "dismiss" button.

---

## 2. Goals & non-goals

**Goals**
- Look and feel like a thoughtfully-designed product, not a weekend hack
- Zero friction: open tab → it works. No login, no setup wizard.
- Reminders feel *delightful*, never naggy or guilt-inducing
- Runs entirely client-side, deploys to Vercel with one click

**Non-goals (for v1)**
- Accounts, sync across devices, or any backend
- Mobile app (responsive web is enough — but desktop is the primary target since reminders only fire when the tab is open)
- Push notifications when tab is closed (out of scope; the metaphor depends on the tab being visible)
- Health-data integrations, social features, leaderboards

---

## 3. The aesthetic — "not amateur looking"

This is the make-or-break section. Every choice below should be honored.

### Visual direction
- **Mood**: warm, calm, slightly nostalgic. Like a sunlit windowsill in the late afternoon.
- **Reference points**: Apple Health's bloom rings, Headspace's illustration style, Studio Ghibli backgrounds, Rifle Paper Co. botanicals, Things 3's restraint.
- **Avoid at all costs**: emoji as art, generic Material/Bootstrap components, neon gradients, dark-mode-by-default tech aesthetic, comic-sans-adjacent fonts, anything that looks "gamified."

### Color palette (warm botanical)
Define these as CSS custom properties in `:root`:
- `--cream`: `#FAF6EF` (background)
- `--cream-deep`: `#F2EADB` (card surface)
- `--terracotta`: `#C97B5C` (pot, accents)
- `--sage`: `#8FA98E` (stem, leaves base)
- `--sage-deep`: `#5C7A5E` (leaf shadow)
- `--bloom-pink`: `#E8A5A5` (primary flower)
- `--bloom-coral`: `#E89B7B` (secondary flower)
- `--bloom-butter`: `#F2D49B` (third flower)
- `--ink`: `#3A3A3A` (text — never pure black)
- `--ink-soft`: `#7A7570` (secondary text)

### Typography
- **Display / numbers**: [Fraunces](https://fonts.google.com/specimen/Fraunces) (variable, with optical sizing — feels editorial and handcrafted)
- **UI / body**: [Inter](https://fonts.google.com/specimen/Inter) at 15–16px, generous line-height (1.6)
- Use Fraunces for the timer numerals and flower's "name." Use Inter for everything else.
- Never use system-ui as a fallback alone — it betrays the un-designed feel.

### Layout
- Single viewport, no scrolling on desktop
- Flower centered, 60% of viewport height
- Timer ring around the pot (thin, ~3px, `--terracotta` at 30% opacity)
- Top-left: tiny day counter ("Day 3 · 4 of 8 cups")
- Top-right: settings cog (opens a small panel for interval, sound on/off, daily goal)
- Bottom: nothing. Resist the urge to add a footer.

### Motion (this is where polish lives)
Use **Framer Motion** or hand-rolled CSS with cubic-bezier easings. Motion principles:
- **Breathing**: the flower has a 4-second idle sway (subtle rotate ±1.5°, ease-in-out)
- **Wilting**: over the timer's last 25%, the stem rotates from 0° → 18°, petals scale 1 → 0.94, saturation drops 100% → 70%. Use a long ease-out so it feels gradual, not animated.
- **Pouring**: water droplets are SVG circles with a path animation; on hit, the flower does a spring-back (overshoot then settle, ~600ms)
- **Petal fall**: when a streak breaks, one petal detaches and floats down with a slight horizontal drift over ~3 seconds
- **Growth**: when the flower levels up, a new leaf/petal scales in from 0 with a gentle bounce
- All transitions: prefer `cubic-bezier(0.34, 1.56, 0.64, 1)` (a soft overshoot) for state changes

### The flower itself
- **Custom SVG, not an emoji, not a stock asset.** This is the centerpiece — it deserves to be hand-drawn.
- Build it as a layered SVG: pot → soil → stem → leaves (2–4) → flower head → petals (5–7, each its own path so they can fall independently)
- Slight imperfection in the linework — avoid perfectly geometric shapes. Petals should look painted, not vectorized.
- Three flower variants for the bouquet (pink peony, coral ranunculus, butter daisy — all in the palette above)
- If hand-drawing is a blocker, pull from a high-quality illustrator like Streamline's "Plump" set or commission one — but **do not ship with default Material icons or emoji.**

### Sound (optional but worth it)
- A single soft chime when the timer fires (think: a wooden wind chime, not a notification ding)
- A gentle water-trickle loop while pouring
- Both off by default; toggle in settings. Use [howler.js](https://howlerjs.com/) or native `Audio`.
- Source from freesound.org with proper licensing, or generate with a tool like Soundraw.

---

## 4. MVP feature list

1. **Timer**: configurable interval (default 45 min), visualized as a ring around the pot
2. **Wilt animation**: gradual over the final 25% of the interval
3. **Pour interaction**: drag the watering can over the flower to confirm a drink
4. **Growth**: flower gains a stage (sprout → bud → bloom → full bloom → +second flower) every N successful pours
5. **Daily goal**: default 8 cups, resets at local midnight
6. **Streak tracking**: consecutive days hitting the goal (shown subtly in settings, not on main view)
7. **Persistence**: all state in `localStorage`
8. **Settings panel**: interval, daily goal, sound on/off, reset flower
9. **Tab-title timer**: when tab is unfocused, title becomes "🌸 5:23 until next sip" (the one place an emoji is OK, since favicons/titles are constrained)
10. **Reduced-motion support**: respect `prefers-reduced-motion` — swap animations for crossfades

---

## 5. Tech stack

- **Framework**: Next.js 15 (App Router) — overkill for a single page, but Vercel-native and gives you image optimization + font loading for free. Alternative: Vite + React if you want lighter.
- **Language**: TypeScript, strict mode
- **Styling**: Tailwind CSS v4 + CSS custom properties for the palette. No component library — every element is hand-built.
- **Animation**: Framer Motion (`motion/react`)
- **State**: React state + a single `useLocalStorage` hook. No Redux, no Zustand — this app is too small.
- **Audio**: native `Audio` API (don't pull howler unless you need it)
- **Fonts**: `next/font/google` for Fraunces + Inter
- **Deployment**: Vercel (drop the repo in, done)

---

## 6. Data model

```ts
type FlowerStage = 'sprout' | 'bud' | 'bloom' | 'full' | 'bouquet-2' | 'bouquet-3';

type BloomState = {
  // Settings
  intervalMinutes: number;        // default 45
  dailyGoal: number;              // default 8
  soundEnabled: boolean;          // default false

  // Today's progress
  cupsToday: number;
  lastDrinkAt: number;            // unix ms
  todayDate: string;              // 'YYYY-MM-DD' for midnight reset

  // Long-term
  flowerStage: FlowerStage;
  streakDays: number;
  lastGoalHitDate: string | null;

  // Aesthetics
  primaryFlowerColor: 'pink' | 'coral' | 'butter';
};
```

Persist to `localStorage` under key `bloom:state:v1`. Version it so future migrations are clean.

---

## 7. File structure

```
drinkwater/
├── README.md                       ← short, with a screenshot once it exists
├── PLAN.md                         ← this file
├── package.json
├── tsconfig.json
├── next.config.ts
├── tailwind.config.ts
├── postcss.config.mjs
├── .gitignore
├── public/
│   ├── favicon.svg                 ← the flower head, simplified
│   └── sounds/
│       ├── chime.mp3
│       └── water.mp3
├── app/
│   ├── layout.tsx                  ← font loading, metadata
│   ├── page.tsx                    ← the only page
│   └── globals.css                 ← CSS vars, base styles, Tailwind
├── components/
│   ├── Flower.tsx                  ← the SVG flower with stage + wilt props
│   ├── Pot.tsx                     ← terracotta pot SVG (separated for layering)
│   ├── TimerRing.tsx               ← SVG ring around the pot
│   ├── WateringCan.tsx             ← draggable can with droplet emitter
│   ├── SettingsPanel.tsx           ← slide-in panel from the right
│   ├── DayCounter.tsx              ← "Day 3 · 4 of 8 cups"
│   └── ReducedMotionFlower.tsx     ← static fallback
├── hooks/
│   ├── useLocalStorage.ts
│   ├── useTimer.ts                 ← the countdown + wilt-progress calculation
│   └── useDragToPour.ts            ← pointer events for the watering can
├── lib/
│   ├── state.ts                    ← BloomState type, defaults, reducer, migration
│   ├── stages.ts                   ← rules for stage progression
│   └── time.ts                     ← midnight reset, date helpers
└── assets/
    └── flower-sketches/            ← reference images / WIP SVGs
```

---

## 8. Build order (what to build first)

1. **Scaffold**: `npx create-next-app@latest`, strip the boilerplate, set up fonts and color tokens
2. **Static flower**: build the SVG `<Flower />` in its "full bloom" state. Get this looking *beautiful* before anything else moves. **If the flower looks amateur, the whole app does.** Spend real time here.
3. **Wilt states**: parameterize the flower with a `wiltProgress: 0..1` prop and design 4 keyframes (fresh, slightly tired, wilting, drooped)
4. **Timer ring + countdown logic** in `useTimer`
5. **Pour interaction**: drag the watering can, emit droplets, trigger the spring-back animation
6. **State + persistence**: wire up `BloomState`, localStorage, midnight reset
7. **Stage progression**: sprout → bud → bloom → full → bouquet
8. **Settings panel**
9. **Sound + reduced-motion polish**
10. **Deploy to Vercel**

---

## 9. User flows

**First visit**
- Page loads, flower is a small sprout in fresh soil
- A one-time tooltip near the watering can: "Drag me onto the flower whenever you take a sip"
- Timer starts immediately at the default interval

**Reminder fires**
- Chime plays (if enabled), tab title updates
- Flower visibly droops over the previous few minutes (so it's not a sudden jump)
- Watering can gently bobs at the top of the screen to draw the eye

**User pours**
- Drag the can — droplets fall in real-time following the cursor
- On release over the flower: spring-back animation, +1 cup, timer resets
- If this push hits a stage threshold, the new petal/leaf grows in

**End of day**
- At local midnight, `cupsToday` resets, `todayDate` updates
- If goal was hit, `streakDays` increments and a third flower may join the pot
- If goal was missed, one petal falls from the flower (recoverable next day)

**Returning the next day**
- Flower retains its long-term stage; today's cup count is fresh
- A subtle "Day N" appears in the corner

---

## 10. Open questions for the build phase

- **Flower art**: hand-draw in Figma/Illustrator, or commission? If hand-drawing isn't realistic, what's the fallback that doesn't look generic?
- **Should the watering can be persistent on screen, or only appear when it's time to drink?** (Lean toward: always visible but dimmed, brightens when due)
- **What happens if the tab is closed for hours?** Probably: on return, show the flower in its current wilt state, no guilt-trip, just "ready when you are"
- **Accessibility**: keyboard-only users need a way to "pour" — spacebar? Add this in v1.1.
- **Daily goal at 8 cups feels arbitrary** — make it adjustable from the start, default 8.

---

## 11. Definition of done for MVP

- [ ] Deployed to Vercel at a real URL
- [ ] Looks intentional on a 13" laptop screen at first glance (the "designer test")
- [ ] Flower SVG is custom and beautiful
- [ ] Timer, wilt, pour, and growth all work end-to-end
- [ ] Survives a tab refresh (state persists)
- [ ] Survives midnight (date resets)
- [ ] No console errors, no layout shift on load
- [ ] Lighthouse score 95+ on performance and accessibility

---

*When you share this file back, ask me to "build the Bloom MVP from PLAN.md" and I'll start from step 1 of the build order.*
