# 4-Row Horizontal Swipe & Multi-Column Peeking Gallery Architecture

Transform the public gallery on `index.html` into a fixed 4-row horizontal swipe carousel across all device form-factors. Prevents indefinite vertical page stretching while presenting an optimal viewport matrix (8 photos on mobile, 16 photos on desktop) with edge peeking and dual swipe/button navigation.

---

### User Review & Critical Decisions

> [!IMPORTANT]
> The following parameters were confirmed by the user in Phase 1 clarification:
> - **Unified Vertical Height (Fixed 4 Rows)**: Across mobile, tablet, and desktop, the gallery is locked to exactly **4 rows vertically**. All additional photos overflow and scroll **horizontally** to the side.
> - **Initial Viewport Photo Matrix**:
>   - **Mobile (`< 640px`)**: Displays **2 columns x 4 rows = 8 photos** in the initial view, with the 3rd column peeking on the right edge to visually cue horizontal swipeability.
>   - **Tablet (`640px - 1023px`)**: Displays **3 columns x 4 rows = 12 photos** with the 4th column peeking.
>   - **Desktop / Laptop (`>= 1024px`)**: Displays **4 columns x 4 rows = 16 photos** with the 5th column peeking.
> - **Navigation & Interaction**: Full touch swipe on mobile, smooth mouse drag-to-scroll, plus prominent **desktop Left/Right navigation arrow buttons** (`<` and `>`).
> - **Language Requirement**: All user communication in Malayalam; UI elements in English.

---

## 1. Overview & Core Concept

### What It Does
Replaces full-page vertical image expansion with a **high-density, 4-row horizontal swipe gallery**. Institutional event photos flow sideways in clean columns, preventing long-scroll fatigue while showcasing large photo albums smoothly.

### Target Audience & Persona
- Visitors, students, and parents browsing school galleries on smartphones (via one-thumb swipe) or desktop computers (via arrow buttons or mouse drag).

### Key Value
- **Zero Vertical Page Bloat**: Regardless of whether there are 10 or 150 photos, the gallery stays within a compact vertical footprint (~380px on mobile, ~580px on desktop).
- **Affordance by Design**: By peeking the next column off the right edge (e.g. 2.2 columns visible on mobile, 4.25 on desktop), users instantly recognize that more photos exist horizontally.

---

## 2. User Experience & Visual Design

### 4-Row Responsive Horizontal Viewport Matrix

```
[ MOBILE VIEWPORT (< 640px) - 2.2 COLUMNS VISIBLE (8 Photos + Peeking 3rd Col) ]
┌───────────────────────────────────────────────┐
│ GALLERY                                       │
│ ┌──────────────┐ ┌──────────────┐ ┌───[PEEK]─┐│
│ │ Photo 1      │ │ Photo 5      │ │ Photo 9  ││
│ └──────────────┘ └──────────────┘ └───[PEEK]─┘│
│ ┌──────────────┐ ┌──────────────┐ ┌───[PEEK]─┐│
│ │ Photo 2      │ │ Photo 6      │ │ Photo 10 ││
│ └──────────────┘ └──────────────┘ └───[PEEK]─┘│
│ ┌──────────────┐ ┌──────────────┐ ┌───[PEEK]─┐│
│ │ Photo 3      │ │ Photo 7      │ │ Photo 11 ││
│ └──────────────┘ └──────────────┘ └───[PEEK]─┘│
│ ┌──────────────┐ ┌──────────────┐ ┌───[PEEK]─┐│
│ │ Photo 4      │ │ Photo 8      │ │ Photo 12 ││
│ └──────────────┘ └──────────────┘ └───[PEEK]─┘│
│ ◄ Swipe horizontally to view more photos ►    │
└───────────────────────────────────────────────┘

[ DESKTOP / LAPTOP (>= 1024px) - 4.25 COLUMNS VISIBLE (16 Photos + Peeking 5th Col) ]
┌─────────────────────────────────────────────────────────────────────────────────┐
│ GALLERY                                              [ < Prev ]   [ Next > ]    │
│ ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──[PEEK]──┐                │
│ │ Photo 1  │ │ Photo 5  │ │ Photo 9  │ │ Photo 13 │ │ Photo 17 │                │
│ └──────────┘ └──────────┘ └──────────┘ └──────────┘ └──[PEEK]──┘                │
│ ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──[PEEK]──┐                │
│ │ Photo 2  │ │ Photo 6  │ │ Photo 10 │ │ Photo 14 │ │ Photo 18 │                │
│ └──────────┘ └──────────┘ └──────────┘ └──────────┘ └──[PEEK]──┘                │
│ ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──[PEEK]──┐                │
│ │ Photo 3  │ │ Photo 7  │ │ Photo 11 │ │ Photo 15 │ │ Photo 19 │                │
│ └──────────┘ └──────────┘ └──────────┘ └──────────┘ └──[PEEK]──┘                │
│ ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──[PEEK]──┐                │
│ │ Photo 4  │ │ Photo 8  │ │ Photo 12 │ │ Photo 16 │ │ Photo 20 │                │
│ └──────────┘ └──────────┘ └──────────┘ └──────────┘ └──[PEEK]──┘                │
└─────────────────────────────────────────────────────────────────────────────────┘
```

### Spatial Dimensions & Ratios
- **Grid Engine**:
  - `display: grid;`
  - `grid-auto-flow: column;` (causes columns to append horizontally to the right).
  - `grid-template-rows: repeat(4, ...);` (locks vertical rows strictly to 4).
- **Row Heights & Column Widths**:
  - **Mobile**: Rows `repeat(4, 88px)`, column width `calc((100% - 10px) / 2.2)` (~155px), gap `8px`. Total height ~380px.
  - **Tablet**: Rows `repeat(4, 115px)`, column width `calc((100% - 24px) / 3.2)` (~210px), gap `12px`. Total height ~500px.
  - **Desktop**: Rows `repeat(4, 135px)`, column width `calc((100% - 45px) / 4.25)` (~245px), gap `15px`. Total height ~585px.
- **Micro-Interactions**:
  - Smooth hover scale `transform: scale(1.05);` on desktop.
  - Floating Left/Right navigation arrow buttons near the header on desktop devices for 1-click column advancement.
  - Smooth drag-to-scroll listener for desktop mouse users.

---

## 3. Key Technical Decisions & Trade-Offs

- **CSS Grid with `grid-auto-flow: column` vs. Multiple Flex Wrappers**:
  - *Chosen Approach*: Pure CSS Grid with `grid-template-rows: repeat(4, ...); grid-auto-flow: column; overflow-x: auto;`.
  - *Why*: Eliminates complex DOM splitting into sub-divs; the browser naturally tiles images top-to-bottom then left-to-right.
- **Scroll Snap & Edge Peeking (`2.2` / `4.25` divisor)**:
  - *Chosen Approach*: Setting column width using fractional viewport divisors so the subsequent column is cut in half at the container boundary.
  - *Why*: Visual affordance principle—users naturally swipe when they see an element partially sliced at the right margin.

---

## 4. Technical Architecture & File Modifications

### Files to Modify
1. **`index.html`**:
   - Add Left and Right desktop navigation buttons in `#gallery-container` header:
     `<button id="gallery-nav-prev" ...><i class="fas fa-chevron-left"></i></button>`
     `<button id="gallery-nav-next" ...><i class="fas fa-chevron-right"></i></button>`
2. **`styles/main.css`**:
   - Configure `.gallery-swipe-container`:
     - Mobile: `grid-template-rows: repeat(4, 88px); grid-auto-columns: calc((100% - 10px) / 2.2); gap: 8px;`
     - Tablet (`@media (min-width: 640px)`): `grid-template-rows: repeat(4, 115px); grid-auto-columns: calc((100% - 24px) / 3.2); gap: 12px;`
     - Desktop (`@media (min-width: 1024px)`): `grid-template-rows: repeat(4, 135px); grid-auto-columns: calc((100% - 45px) / 4.25); gap: 15px;`
     - Custom sleek horizontal scrollbar and `scroll-snap-type: x mandatory`.
   - Style desktop gallery arrow navigation buttons.
3. **`scripts/pages/index-page.js`**:
   - Add click event listeners to `#gallery-nav-prev` and `#gallery-nav-next` to smoothly scroll by one column offset.
   - Implement optional mouse drag-to-scroll handler for desktop mice.

---

## 5. Testing & Verification Plan

1. **Mobile Test (< 640px)**:
   - Verify exactly 4 rows vertically.
   - Verify exactly 2 columns are fully visible (8 photos) and the 3rd column is visibly cut off (peeking).
   - Test touch swipe horizontally to reveal subsequent columns.
2. **Desktop Test (>= 1024px)**:
   - Verify exactly 4 rows vertically.
   - Verify 4 columns are fully visible (16 photos) and the 5th column peeks.
   - Click Left and Right header arrow buttons to verify smooth horizontal advancement.
   - Test mouse drag-to-scroll.
3. **Lightbox Test**:
   - Click any photo to verify that `#img-modal` lightbox opens correctly with full-screen view and image navigation.
4. **Build & Lint Verification**:
   - Run `compile_applet` and `lint_applet` to confirm zero regression.

---

## 6. Implementation Status & Verification Results

- [x] **Fixed 4-Row Horizontal Swipe Architecture**: Implemented in `styles/main.css` (`grid-template-rows: repeat(4, ...); grid-auto-flow: column;`), strictly locking the gallery to 4 vertical rows on all screen sizes to eliminate vertical page stretching.
- [x] **Mobile 8-Photo Matrix & Peeking Column**: Configured `grid-auto-columns: calc((100% - 10px) / 2.2)`, displaying 2 full columns (8 photos) with the 3rd column peeking on the right to visually prompt horizontal swiping.
- [x] **Desktop 16-Photo Matrix & Peeking Column**: Configured `grid-auto-columns: calc((100% - 45px) / 4.25)`, displaying 4 full columns (16 photos) on desktop and laptop displays with the 5th column peeking.
- [x] **Desktop Header Navigation Arrow Buttons**: Added `#gallery-nav-prev` and `#gallery-nav-next` in `index.html` with click handlers in `scripts/pages/index-page.js` for 1-click smooth horizontal advancement.
- [x] **Mouse Drag-to-Scroll & Lightbox Protection**: Implemented smooth mouse dragging with threshold detection so dragging does not accidentally trigger photo lightbox clicks.
- [x] **Compilation & Lint Verification**: Checked with `compile_applet` and `lint_applet` (`tsc --noEmit`) - both succeeded with 0 errors.

