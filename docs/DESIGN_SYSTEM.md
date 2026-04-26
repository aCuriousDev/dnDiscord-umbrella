# DnDiscord - Design System

> **Mood:** Arcane Grimoire - candlelight on old parchment under a midnight sky.
> **Status:** POC. Extends the existing purple → indigo / gold / Cinzel direction into a formal, token-driven system.
> **Audience:** Frontend devs working on `front/dndiscord-esp`, plus anyone generating brand assets.

---

## 1. Brand Foundation

### 1.1 Vision
DnDiscord is a grimoire you open inside Discord - a living spellbook where parties plan campaigns, roll dice, and fight monsters together. The interface should feel like **candlelight on old parchment under a midnight sky**: mysterious, warm enough to gather around, serious enough to trust with a four-hour session.

### 1.2 Personality
| Trait        | Is                                    | Isn't                          |
| ------------ | -------------------------------------- | ------------------------------ |
| Mood         | Arcane, scholarly, cinematic           | Gory, edgelord, cartoonish     |
| Voice        | Confident DM, dry wit                  | Snarky, gamified, childish     |
| Rhythm       | Deliberate, weighty                    | Bouncy, playful                |
| Texture      | Parchment, ink, gold leaf, candle glow | Plastic, neon, flat-minimalism |

### 1.3 Brand pillars
1. **Grimoire** - content feels written, inked, bound.
2. **Candlelight** - warm gold glows the eye toward what matters.
3. **Twilight** - deep indigo/plum backdrops hold focus without fatigue.
4. **Ritual** - interactions have weight; clicks feel like turning a page or rolling a die.

---

## 2. Color System

All colors are defined as tokens. Tailwind extends them; CSS exposes them as custom properties for non-Tailwind usage (Babylon.js HUD, SVG charts, etc.).

### 2.1 Core palette (ink surfaces)

| Token       | Hex       | Use                                              |
| ----------- | --------- | ------------------------------------------------ |
| `ink.950`   | `#070812` | Deepest background, Babylon viewport void        |
| `ink.900`   | `#0F0F1A` | App background (maps to current `game-darker`)   |
| `ink.800`   | `#14162B` | Elevated surface 1 (cards, modals)               |
| `ink.700`   | `#1A1A2E` | Elevated surface 2 (maps to current `game-dark`) |
| `ink.600`   | `#232544` | Borders, dividers, input fill                    |
| `ink.500`   | `#2E3150` | Hover border                                     |

### 2.2 Brand (purple → indigo)

| Token        | Hex       | Use                                        |
| ------------ | --------- | ------------------------------------------ |
| `plum.900`   | `#2B0F2E` | Brand shadow                               |
| `plum.700`   | `#4B1E4E` | **brandStart** - primary gradient origin   |
| `plum.500`   | `#6B2C6F` | Hover highlight on brand surfaces          |
| `plum.300`   | `#A968AE` | Text on ink, decorative runes              |
| `indigo.900` | `#0B1A2C` | Brand deep shadow                          |
| `indigo.700` | `#162C44` | **brandEnd** - primary gradient terminus   |
| `indigo.500` | `#2A4E78` | Info surfaces, secondary buttons           |
| `indigo.300` | `#6A90C0` | Info text, quiet links                     |

**Brand gradient:** `linear-gradient(135deg, plum.700 0%, indigo.700 100%)` - identical to current `bg-brand-gradient`.

### 2.3 Gold (accent / combat / loot)

| Token      | Hex       | Use                                          |
| ---------- | --------- | -------------------------------------------- |
| `gold.700` | `#8A6A1C` | Pressed/active gold                          |
| `gold.500` | `#C99A2C` | Borders on legendary items                   |
| `gold.400` | `#E3B23C` | Default accent (maps to current `game-gold`) |
| `gold.300` | `#F4C542` | Highlight, XP, crit, focus glow              |
| `gold.200` | `#FFE08A` | Gold text on deep ink                        |

### 2.4 Semantic status

| Token     | Hex       | Use              | Tailwind alias |
| --------- | --------- | ---------------- | -------------- |
| `success` | `#4ADE80` | Heal, save, win  | `game-green`   |
| `danger`  | `#EF4444` | Damage, HP loss  | `game-red`     |
| `warning` | `#F59E0B` | Low HP, timer    | new            |
| `info`    | `#38BDF8` | Tooltip, rumor   | new            |
| `crit`    | `#F4C542` | Nat 20, legendary| `gold.300`     |
| `fumble`  | `#9F1239` | Nat 1, cursed    | new            |

### 2.5 Text

| Token       | Hex                   | Contrast on `ink.900`     |
| ----------- | --------------------- | ------------------------- |
| `text.high` | `#F5F1E4` (parchment) | 14.2 : 1 AAA              |
| `text.mid`  | `#CBC6B3`             | 9.1 : 1 AAA               |
| `text.low`  | `#8A8574`             | 4.8 : 1 AA                |
| `text.mute` | `#5A5648`             | 2.6 : 1 - decorative only |

Never use pure `#FFFFFF` on ink - it vibrates. Use `text.high`.

### 2.6 Usage rules
- Brand gradient = navigation, hero CTAs, identity surfaces. **Not** for status or data viz.
- Gold = reward, success-in-combat, focus rings. Do not use for generic emphasis.
- Status colors = strictly reserved. Never decorative.
- Maximum 2 hues per surface (ink + one brand or one accent). Avoid rainbow combat panels.

---

## 3. Typography

### 3.1 Type families

| Role      | Family                 | Usage                                            |
| --------- | ---------------------- | ------------------------------------------------ |
| Display   | **Cinzel**             | Titles, hero, section headers, dice totals       |
| Secondary | **IM Fell English SC** | Flavor text, quotes, journal entries, item names |
| Body      | **Inter**              | UI labels, paragraphs, forms, chat               |
| Mono      | **JetBrains Mono**     | Dice expressions (`2d6+3`), code, hashes         |

Cinzel and IM Fell stay imported via Google Fonts as already configured. Add Inter + JetBrains Mono.

### 3.2 Scale (1.25 - major third)

| Token     | Size / Line-height | Weight | Family    | Use                     |
| --------- | ------------------ | ------ | --------- | ----------------------- |
| `display` | 48 / 56            | 700    | Cinzel    | Hero, splash            |
| `h1`      | 36 / 44            | 700    | Cinzel    | Page title              |
| `h2`      | 28 / 36            | 600    | Cinzel    | Section                 |
| `h3`      | 22 / 30            | 600    | Cinzel    | Card title              |
| `lead`    | 18 / 28            | 400    | IM Fell   | Flavor, quotes          |
| `body`    | 16 / 24            | 400    | Inter     | Default paragraph       |
| `small`   | 14 / 20            | 400    | Inter     | Meta, captions          |
| `micro`   | 12 / 16            | 500    | Inter     | Labels, tags            |
| `dice`    | 20 / 24            | 600    | JetBrains | Dice / stat expressions |

### 3.3 Rules
- Cinzel always in `letter-spacing: 0.04em` on sizes ≥ h2 - it breathes better.
- IM Fell reserved for narrative voice. Never use for data or forms.
- Line length: body paragraphs capped at **68ch**.
- Title glow is optional: `text-shadow: 0 0 14px rgba(88,33,90,.7), 0 0 28px rgba(28,57,91,.6)` (existing `.title-shine`).

---

## 4. Spacing, Radii, Shadows, Borders

### 4.1 Spacing (4px base)
`0, 4, 8, 12, 16, 20, 24, 32, 40, 48, 64, 80` - use Tailwind defaults (`0, 1, 2, 3, 4, 5, 6, 8, 10, 12, 16, 20`).

### 4.2 Radii
| Token         | Value | Use                     |
| ------------- | ----- | ----------------------- |
| `radius.sm`   | 6px   | Chips, pills, inputs    |
| `radius.md`   | 10px  | Buttons, badges         |
| `radius.lg`   | 14px  | Cards, panels           |
| `radius.xl`   | 20px  | Modals, hero            |
| `radius.full` | 9999  | Avatars, dot indicators |

### 4.3 Borders
- Default: `1px solid rgba(255,255,255,0.08)` (ink surfaces).
- Brand border: `2px solid rgba(255,255,255,0.18)` (menu badges).
- Focus ring: `2px solid gold.300` + `0 0 0 4px rgba(244,197,66,0.25)` outer glow.

### 4.4 Shadows
| Token          | Value                                                            | Use                |
| -------------- | ---------------------------------------------------------------- | ------------------ |
| `shadow.soft`  | `0 10px 30px rgba(0,0,0,0.25)`                                   | Buttons, cards     |
| `shadow.lift`  | `0 14px 36px rgba(0,0,0,0.5)`                                    | Hover elevation    |
| `shadow.modal` | `0 40px 80px rgba(0,0,0,0.6), 0 0 0 1px rgba(255,255,255,0.06)`  | Modals             |
| `shadow.glow`  | `0 0 20px rgba(244,197,66,0.35), 0 0 40px rgba(75,30,78,0.4)`    | Focus, crit, magic |
| `shadow.inset` | `inset 0 0 40px rgba(148,163,184,0.2)`                           | Panels (existing)  |

### 4.5 Surfaces & vignette
Every primary page gets the existing `.vignette` inner shadow. Elevated surfaces stack `ink.800` → `ink.700` → `ink.600` with a 1px top hairline (`linear-gradient(180deg, rgba(255,255,255,0.05), transparent 1px)`).

---

## 5. Motion

All transitions obey `cubic-bezier(0.2, 0.8, 0.2, 1)` ("grimoire ease") unless noted.

| Token          | Duration | Use                            |
| -------------- | -------- | ------------------------------ |
| `motion.xs`    | 120ms    | Hover color shifts             |
| `motion.sm`    | 200ms    | Button lift, badge glow        |
| `motion.md`    | 350ms    | Page fade, art crossfade       |
| `motion.lg`    | 600ms    | Modal reveal, ritual openings  |
| `motion.shine` | 1800ms   | Gradient pan on brand surfaces |
| `motion.title` | 6000ms   | Title shine loop               |

Preserve existing animations: `btnGradientPan`, `titleShineMove`, `fadeInUp`, `fadeOutRight`.

**Reduced motion:** if `prefers-reduced-motion: reduce`, drop all shine loops and art crossfades; keep focus/hover color transitions only.

---

## 6. Component Specifications

### 6.1 Button - primary
- Background: `bg-brand-gradient`
- Border: transparent → `rgba(255,255,255,0.2)` on hover
- Text: Cinzel 600 / 16px / letter-spacing 0.04em, color `text.high`
- Padding: `14px 20px`
- Radius: `radius.md`
- Shadow: `shadow.soft` → `shadow.lift`
- Hover: `translateY(-1px)` + `btnGradientPan` shine + brightness(1.05)
- Focus: `shadow.glow` + `2px gold.300` ring
- Disabled: 40% opacity, no shine, cursor `not-allowed`

### 6.2 Button - secondary
- Background: `ink.700` with `1px solid ink.500`
- Text: `text.high`, Inter 600
- Hover: border → `plum.500`, background → `ink.600`

### 6.3 Button - ghost
- Transparent, `text.mid`, hover: `ink.800`, no border until hover.

### 6.4 Menu row (badge + button)
Already implemented; lock spec:
- 56×56 rotated badge overlapping button at `left: 0; translate(-50%,-50%) rotate(45deg)`
- Inner icon rotated `-45deg`, 24×24, `text.high`
- Button: 320px min width on desktop, full-width on mobile, 56px height
- Conic glow ring on badge (existing `::after`)

### 6.5 Card / Panel
- Background: `ink.800`
- Border: `1px rgba(255,255,255,0.08)` + 1px top hairline
- Radius: `radius.lg`
- Padding: 20px
- Shadow: `shadow.soft`
- Optional title bar: Cinzel h3 + gold hairline `1px gold.400` beneath

### 6.6 Character sheet panel (combat)
- Background: `ink.700/95` + backdrop blur
- Gold trim: `1px solid gold.400/30`, hover `gold.400`
- Stat rows: label in Inter `small` `text.mid`, value in Cinzel `h3` `text.high`
- Maps to existing `.panel-game`

### 6.7 Stat bar (HP / MP / XP)
- Track: `ink.900`, 12px tall, `radius.sm`, `1px` inner border `rgba(255,255,255,0.08)`
- Fill: solid color - HP `danger`, MP `indigo.500`, XP `gold.300`
- Transition: width `motion.md`
- Optional: gold glow on fill when value ≥ 90% (crit zone)

### 6.8 Dice display
- Container: 64×64 square, `radius.md`, `plum.900` → `indigo.900` gradient, gold 2px border
- Number: JetBrains Mono `dice`, centered, `gold.300`
- Rolling state: 350ms tumble (rotate + scale) using existing motion curve
- Crit: `shadow.glow` pulse

### 6.9 Chat / journal entry
- Background: transparent on `ink.900`
- Speaker name: IM Fell English SC, `plum.300`
- Body: Inter `body`, `text.mid`
- System messages: `small`, italic, `text.low`, inline rune separator ✦

### 6.10 Input / form field
- Background: `ink.600`, inset shadow `0 1px 0 rgba(0,0,0,0.3)`
- Border: `1px ink.500`
- Radius: `radius.sm`
- Padding: `10px 14px`
- Focus: border `gold.300`, outer glow `shadow.glow`, no default browser outline
- Placeholder: `text.low`
- Error: border `danger`, helper text `danger` `small`

### 6.11 Modal
- Overlay: `rgba(7,8,18,0.75)` + backdrop blur 4px
- Surface: `ink.800`, `radius.xl`, `shadow.modal`, max-width 560px
- Reveal: `fadeInUp` at `motion.lg`, overlay fades at `motion.md`

### 6.12 Tooltip
- `ink.950` / 95% + `1px rgba(255,255,255,0.08)`
- Text `small` `text.high`
- Shadow `shadow.lift`
- Delay 400ms in, 80ms out

### 6.13 Tag / chip
- `ink.700` with `1px ink.500` border
- `micro` text, padding `2px 8px`, `radius.full`
- Variants: `class-<classname>` tints using class colors (see [Section 8](#8-class-color-accents))

### 6.14 Settings button (existing)
Lock current `.settings-btn` - fixed top-right, conic-gradient border, padding-box/border-box clip. No changes.

---

## 7. Iconography

### 7.1 System icons
- 24×24 grid, 2px stroke, round joins/caps
- Stroke color inherits `currentColor`
- Source: **Lucide** (MIT, tree-shakeable) - wide coverage, matches tone
- Custom icons only when Lucide lacks a D&D primitive (d20, spellbook, scroll, grimoire)

### 7.2 Class / action icons (brand)
- 48×48 square base, drawn inside a rotated diamond to match `.menu-badge`
- Monoline ink style, gold fills optional on rewards
- Exported as SVG + 2x PNG
- Set: Campaign, New Character, Party, Battle Map, Dice Tray, Inventory, Grimoire, Settings, Invite, Save, Load, End Turn, Initiative, Level Up, Shop

### 7.3 Rules
- Never mix filled + stroked icons inside the same panel.
- Icons inside `.menu-badge` always white with `drop-shadow(0 2px 6px rgba(0,0,0,0.35))`.
- Disabled icons at 40% opacity.

---

## 8. Class Color Accents

D&D classes used for avatars, portraits, tags. Pulled from class artwork in `dist/assets/classes/`.

| Class     | Hex       |
| --------- | --------- |
| Barbarian | `#B33A3A` |
| Bard      | `#C97BB4` |
| Cleric    | `#E6C35C` |
| Druid     | `#6BAA3F` |
| Fighter   | `#8B5A2B` |
| Monk      | `#6EC1D1` |
| Paladin   | `#E8D28C` |
| Ranger    | `#3F7A4E` |
| Rogue     | `#5A5260` |
| Sorcerer  | `#D94F4F` |
| Warlock   | `#7A3CAE` |
| Wizard    | `#3F6BD1` |

Used only as 8% tint backgrounds, 100% on small accent bars, and in tag borders. Never as full panel colors.

---

## 9. Accessibility

- All interactive text ≥ `text.mid` (4.5:1 minimum).
- Focus visible on every interactive element - gold ring, never removed.
- Target size ≥ 44×44 on touch.
- `prefers-reduced-motion`: strip shine loops, crossfades, badge glow pulses.
- Color never the sole carrier of meaning (pair with icon or label - HP bar shows numeric too).
- Discord embedded mode: respect safe-area insets (existing `.pt-safe-top` etc.).

---

## 10. Token → Tailwind Mapping (drop-in)

```js
// tailwind.config.js - extend block
colors: {
  ink:    { 950:'#070812', 900:'#0F0F1A', 800:'#14162B', 700:'#1A1A2E', 600:'#232544', 500:'#2E3150' },
  plum:   { 900:'#2B0F2E', 700:'#4B1E4E', 500:'#6B2C6F', 300:'#A968AE' },
  indigo: { 900:'#0B1A2C', 700:'#162C44', 500:'#2A4E78', 300:'#6A90C0' },
  gold:   { 700:'#8A6A1C', 500:'#C99A2C', 400:'#E3B23C', 300:'#F4C542', 200:'#FFE08A' },
  parchment: '#F5F1E4',
  // legacy aliases (keep working)
  brandStart: '#4B1E4E', brandEnd: '#162C44',
  'game-dark':'#1A1A2E','game-darker':'#0F0F1A','game-accent':'#E94560',
  'game-gold':'#F4C542','game-blue':'#0F3460','game-blue-light':'#1E5A8E',
  'game-green':'#4ADE80','game-red':'#EF4444',
},
fontFamily: {
  fantasy: ['Cinzel','Georgia','serif'],
  display: ['Cinzel','serif'],
  old:     ['IM Fell English SC','serif'],
  body:    ['Inter','system-ui','sans-serif'],
  mono:    ['JetBrains Mono','ui-monospace','monospace'],
},
boxShadow: {
  soft:      '0 10px 30px rgba(0,0,0,0.25)',
  lift:      '0 14px 36px rgba(0,0,0,0.5)',
  modal:     '0 40px 80px rgba(0,0,0,0.6), 0 0 0 1px rgba(255,255,255,0.06)',
  glow:      '0 0 20px rgba(244,197,66,0.35), 0 0 40px rgba(75,30,78,0.4)',
  insetGlow: 'inset 0 0 40px rgba(148,163,184,0.2)',
},
backgroundImage: {
  'brand-gradient':     'linear-gradient(135deg, #4B1E4E 0%, #162C44 100%)',
  'grimoire-vignette':  'radial-gradient(ellipse at center, transparent 55%, rgba(0,0,0,0.65) 100%)',
},
```

---

## 11. Nano Banana Prompts - Logo

Nano banana (Gemini 2.5 Flash Image) responds best to **descriptive, scene-level prompts** with explicit composition, lighting, materials, and a clear "avoid" clause. Each prompt below is self-contained.

### 11.1 Primary mark - the "Arcane D20"

> A centered logo mark on a pure black background. A twenty-sided die (icosahedron, d20) seen three-quarters front, carved from obsidian with glowing violet runes etched deep into each visible face. The top face shows a stylized "D" fused with a chat-bubble tail, as if the die were speaking. Molten gold seeps into the rune channels and drips one slow drop from the lowest vertex. Rim lighting in deep indigo from behind, warm candle highlight from upper-left, soft volumetric smoke at the base. Ultra sharp, symmetrical, studio product photography feel, 1:1 square composition, generous negative space around the die. Color palette strictly limited to deep plum (#4B1E4E), midnight indigo (#162C44), obsidian black, and candle gold (#F4C542). Avoid: text, watermarks, multiple dice, cartoon style, neon, chromatic aberration, reflections of a studio.

### 11.2 Wordmark - "DnDiscord"

> A horizontal wordmark reading exactly "DnDiscord" on a pure black background. Cinzel-style serif capitals, custom-carved letterforms inked in warm candle gold (#F4C542) with faint violet runic underlines beneath the baseline. The central "i" is dotted with a tiny six-pointed arcane star. Subtle parchment texture inside the gold strokes, as if the letters were gilded onto aged vellum and photographed under candlelight. Extremely high legibility, perfectly kerned, symmetrical, 3:1 aspect ratio, generous margins. Avoid: shadows beneath the text, glow halos, sans-serif fonts, misspellings, extra characters, watermark, background imagery.

### 11.3 Monogram - "Dd" in diamond

> A square monogram logo on a pure black background. A diamond (square rotated 45 degrees) with a gradient fill going from deep plum (#4B1E4E) at the top-left corner to midnight indigo (#162C44) at the bottom-right, framed by a thin gold hairline border. Inside the diamond, upright (not rotated), two interlocked letterforms "D" and "d" in a Cinzel-style serif, carved in candle gold with a faint inner glow. A single six-pointed arcane star nests in the gap between the two letters. Clean, minimal, heraldic feel, 1:1 square, generous black negative space around the diamond. Avoid: text other than "D" and "d", outer glow beyond the border, bevels, 3D extrusion, cartoon shading.

### 11.4 App icon (Discord activity tile)

> A 1:1 square app icon on a rich dark indigo-to-plum radial gradient background (center #4B1E4E, edges #0F0F1A). Centered, a glowing obsidian d20 die with violet rune etchings on every face and molten candle-gold (#F4C542) pooled inside the rune grooves, catching warm light from above. Subtle volumetric candle smoke drifting behind. Composition is centered, heroic, confident, with approximately 12% padding on all sides so the mark remains clear at small sizes (down to 48px). Avoid: any text, letters, numbers on the die faces, multiple objects, busy background, lens flares, people, hands, frames, borders.

### 11.5 Favicon - simplified

> A minimal 1:1 square icon for tiny favicon use. A single geometric obsidian d20 silhouette, front-facing, with ONE visible face glowing in candle gold (#F4C542) against a flat deep plum (#4B1E4E) background. No runes, no text, no gradient, no shadow. Maximum clarity at 16×16 pixels. Avoid: detail, texture, text, numbers, multiple colors beyond the two listed.

---

## 12. Nano Banana Prompts - Icon Set

All icons share these **global style rules**, included at the top of each prompt:

> Style: monoline ink illustration, 2px stroke weight, round line caps, drawn in warm candle gold (#F4C542) on a pure black background, centered in a 1:1 square, 12% padding, no fill except where noted, no text, no watermark, crisp vector-ready edges, symmetrical composition.

### 12.1 Campaign (open grimoire)
> [Global style rules above.] An open leather-bound grimoire seen from above, two pages facing, a feather quill resting diagonally across the right page, a small six-pointed arcane star hovering above the left page. The book's spine and corners have gold filigree. Avoid: text on the pages, readable words, hands, ink spills.

### 12.2 New character (hooded figure silhouette)
> [Global style rules.] A small hooded adventurer silhouette from the chest up, head slightly turned, a faint arcane star mark on the forehead. Simple geometric shapes, no facial detail. Avoid: weapons, background, multiple figures, gender markers.

### 12.3 Party (three linked figures)
> [Global style rules.] Three abstract adventurer silhouettes standing side by side, linked by a continuous gold line running across their shoulders like a bond. Different heights to suggest variety. Avoid: faces, weapons, text, more than three figures.

### 12.4 Battle map (hex grid with pin)
> [Global style rules.] A stylized hex-grid rectangle seen in three-quarter perspective with a single location pin (drop marker) standing on a central hex. A compass rose sits in the lower-right corner. Avoid: labels, terrain details, shadows, multiple pins.

### 12.5 Dice tray (d20 + d6)
> [Global style rules.] A twenty-sided die in the foreground with one six-sided die tucked behind it to the lower right. The d20 shows three visible faces. Avoid: numbers or pips on the faces, multiple extra dice, shadows, text.

### 12.6 Inventory (satchel + items)
> [Global style rules.] An open adventurer's satchel viewed front-on, a potion vial peeking from the left pocket, a scroll peeking from the right, a coin on top. Avoid: text on scroll, faces on coin, realistic shading.

### 12.7 Grimoire (spellbook with star)
> [Global style rules.] A closed spellbook standing upright, a six-pointed arcane star glowing on the cover, two leather clasps on the right edge. Avoid: text, title, hands, pages sticking out.

### 12.8 Settings (cog with rune)
> [Global style rules.] An eight-tooth gear with a small six-pointed arcane star etched at its center. Avoid: screws, multiple gears, 3D shading, text.

### 12.9 Invite (envelope + arcane star seal)
> [Global style rules.] A closed envelope seen front-on, diagonal flap at the back, sealed in the center with a six-pointed arcane star wax seal. Avoid: text, stamps, addresses.

### 12.10 Save / load (scroll + star)
> [Global style rules.] A rolled scroll lying horizontally with a ribbon tied around its middle and a small six-pointed arcane star floating just above the ribbon knot. Avoid: text on the scroll, hands, shadows.

### 12.11 End turn (hourglass)
> [Global style rules.] A slim hourglass, upper bulb half full, lower bulb catching falling sand in an arcane star shape at the bottom. Avoid: text, numbers, multiple hourglasses.

### 12.12 Initiative (curved arrow + pulse)
> [Global style rules.] A curved forward-pointing arrow with a single sharp pulse peak near its midpoint, suggesting quickness. Avoid: text, multiple arrows, shadows.

### 12.13 Level up (upward chevron + star)
> [Global style rules.] A bold upward chevron with a six-pointed arcane star centered just above its peak. Avoid: text, numerals, multiple stars.

### 12.14 Shop (coin stack)
> [Global style rules.] Three stacked coins seen from a slight three-quarter angle, the top coin bearing a six-pointed arcane star. Avoid: currency symbols, numerals, hands, shadows.

### 12.15 Class set (12 portraits)
For each of the 12 classes in [Section 8](#8-class-color-accents), reuse this template:

> [Global style rules.] A stylized portrait bust of a D&D {CLASSNAME} seen from the chest up, three-quarter view, wearing characteristic {CLASS_GEAR}. Monoline ink with the gold accent replaced by the class color {CLASS_HEX}. Heraldic, iconic, readable as a small avatar. Avoid: background, text, multiple figures, readable faces (use geometric minimalism).

Fill-ins per class:

| Class     | `{CLASS_GEAR}`                                              | `{CLASS_HEX}` |
| --------- | ----------------------------------------------------------- | ------------- |
| Barbarian | fur-trimmed shoulder pauldrons, a great axe hilt rising     | `#B33A3A`     |
| Bard      | a feathered cap and a lute neck crossing the chest          | `#C97BB4`     |
| Cleric    | a holy symbol medallion and chainmail collar                | `#E6C35C`     |
| Druid     | a leafy hood and a wooden staff topped with a crescent      | `#6BAA3F`     |
| Fighter   | heavy plate shoulders and a sword pommel at the hip         | `#8B5A2B`     |
| Monk      | a cloth headband and simple robe with a rope belt           | `#6EC1D1`     |
| Paladin   | winged-helm crown and a gleaming breastplate with a sigil   | `#E8D28C`     |
| Ranger    | a hooded cloak and a quiver strap across the chest          | `#3F7A4E`     |
| Rogue     | a half-mask covering nose and mouth, dark leather hood      | `#5A5260`     |
| Sorcerer  | wild hair with faint arcane sparks, an open collar tunic    | `#D94F4F`     |
| Warlock   | a high collar and a floating eldritch sigil above the head  | `#7A3CAE`     |
| Wizard    | a pointed hat and star-covered robes, long beard optional   | `#3F6BD1`     |

---

## 13. Implementation Notes

1. **Install fonts** - add Inter + JetBrains Mono to the existing Google Fonts import in `index.html`.
2. **Extend Tailwind** - paste Section 10 into `tailwind.config.js`. Keep legacy aliases for zero-breaking-change migration.
3. **Refactor `index.css` tokens** - replace `:root` custom properties with the new palette. Keep `.menu-*`, `.settings-btn`, `.class-art-*`, `.vignette` as-is.
4. **Add utility classes** - `.surface-1`, `.surface-2`, `.text-high/mid/low`, `.focus-ring-gold`.
5. **Add `prefers-reduced-motion`** media-query block stripping shine animations.
6. **Generate logo/icon assets** via nano banana using Section 11 / Section 12 prompts. Store under `front/dndiscord-esp/public/brand/`.
7. **No component rewrite.** POC scope - the existing Solid components keep working because legacy color tokens remain valid.

---

## 14. Verification

After implementation:

- `npm run dev` from `front/dndiscord-esp/` - app boots, menu page renders, badges + gradient buttons look identical or better.
- Open the menu route visually; confirm focus ring is gold and visible on tab navigation.
- DevTools → Rendering → emulate `prefers-reduced-motion: reduce`; confirm title shine and button shine halt.
- Lighthouse accessibility ≥ 95 on the menu page.
- Tailwind build does not error on new tokens; legacy `bg-game-dark`, `text-brandStart`, `bg-brand-gradient` still resolve.
- Generated brand assets exist under `public/brand/` and display in `index.html` as favicon.

---

## 15. Out of Scope (POC)

- No dark/light mode toggle (app is dark-only).
- No i18n typographic variants.
- No Storybook / component library extraction.
- No animation library (Framer Motion, GSAP).
- No redesign of Babylon.js 3D scene lighting.
- No marketing site.

These are tracked for post-POC in a follow-up design iteration.
