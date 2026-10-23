# 3-Column Responsive Notification Grid for Desktop & Laptop

Transform the public notifications section on the index page from a stretched single-column layout into a clean, 3-column responsive grid on desktop and laptop displays, a 2-column layout on tablets, and a single column on mobile, with smooth internal container scrolling.

---

### User Review & Critical Decisions

> [!IMPORTANT]
> The following parameters were confirmed by the user in Phase 1 clarification:
> - **Desktop & Laptop Layout**: Exactly 3 columns on desktop (`>= 1024px`), 2 columns on tablet (`640px - 1023px`), and 1 column on mobile (`< 640px`).
> - **Height & Scrolling Behavior**: 3-column grid housed within a dedicated scrollable container with custom smooth scrollbar when multiple notices are posted.
> - **Language Requirement**: All user communication remains in Malayalam; all application UI labels, dates, and buttons remain in English only.

---

## 1. Overview & Core Concept

### What It Does
Re-architects `#notices-list` on the public landing page (`index.html`) so notifications appear in a structured **3-column card grid** on desktop and laptop displays instead of stretching horizontally across the full screen width.

### Target Audience & Persona
- Parents, students, and institutional visitors viewing circulars, exam announcements, and notices on wide screens (desktop monitors, laptops, large tablets).

### Key Value
- Eliminates excessive horizontal eye-travel caused by full-width single-line notice cards on wide screens.
- Enhances visual density, allowing users to scan 6 notices in two rows at a glance without extensive scrolling.

---

## 2. User Experience & Visual Design

### Responsive Grid Breakpoints & Spatial Composition

```
[ DESKTOP / LAPTOP (>= 1024px) ]
┌─────────────────────────────────────────────────────────────────────────────┐
│ NOTIFICATIONS                                                               │
│ ┌──────────────────────┐ ┌──────────────────────┐ ┌──────────────────────┐ │
│ │ Notice Card 1        │ │ Notice Card 2        │ │ Notice Card 3        │ │
│ │ [Pinned] Annual Day  │ │ Fee Revision 2026    │ │ Vacation Notice      │ │
│ │ [Image Preview]      │ │ Text description...  │ │ Text description...  │ │
│ │ Oct 24, 2026  [View] │ │ Oct 22, 2026  [View] │ │ Oct 20, 2026  [View] │ │
│ └──────────────────────┘ └──────────────────────┘ └──────────────────────┘ │
│ ┌──────────────────────┐ ┌──────────────────────┐ ┌──────────────────────┐ │
│ │ Notice Card 4        │ │ Notice Card 5        │ │ Notice Card 6        │ │
│ │ Result Announcement  │ │ Admission Open       │ │ Sports Meet Circular │ │
│ │ Text description...  │ │ [Image Preview]      │ │ Text description...  │ │
│ │ Oct 18, 2026  [View] │ │ Oct 15, 2026  [View] │ │ Oct 12, 2026  [View] │ │
│ └──────────────────────┘ └──────────────────────┘ └──────────────────────┘ │
│ (Scrollable container if > 6 notices exist)                                 │
└─────────────────────────────────────────────────────────────────────────────┘

[ TABLET (640px - 1023px) ]
┌─────────────────────────────────────────────────────────┐
│ ┌───────────────────────────┐ ┌───────────────────────┐ │
│ │ Notice Card 1             │ │ Notice Card 2         │ │
│ └───────────────────────────┘ └───────────────────────┘ │
└─────────────────────────────────────────────────────────┘

[ MOBILE (< 640px) ]
┌──────────────────────────────────────┐
│ ┌──────────────────────────────────┐ │
│ │ Notice Card 1                    │ │
│ └──────────────────────────────────┘ │
└──────────────────────────────────────┘
```

### Card Layout & Aesthetic Polish
- **Card Anatomy**:
  - `Header`: Clean typography, pinned icon indicator (`fa-thumbtack`), notice title.
  - `Image Box`: Fixed aspect-ratio image container with rounded corners and subtle border.
  - `Body`: Line-clamped preview (3-4 lines) with link resolution.
  - `Footer`: `margin-top: auto` so date and action button (`Expand`/`View`) are anchored uniformly at the bottom regardless of varying card text heights.
- **Scroll Container**:
  - Sleek custom scrollbar (`::-webkit-scrollbar` with subtle track).
  - Max height adjusted to display ~2 complete rows (approx. 560px–600px) comfortably on desktop before scrolling engages.

---

## 3. Key Technical Decisions & Trade-Offs

- **CSS Grid over Flexbox**:
  - *Chosen Approach*: `display: grid; grid-template-columns: repeat(3, 1fr);` on desktop.
  - *Why*: Guarantees strictly uniform 3 equal-width columns without awkward wrapping or uneven spacing.
- **Dynamic Viewport Height Adaptation**:
  - *Chosen Approach*: Update `fitNoticeViewportForFourCards` in `scripts/pages/index-page.js` to account for multi-column grid math (measuring row heights rather than simply summing single cards).
  - *Why*: Ensures the scroll container height adapts correctly whether on mobile (1 col), tablet (2 cols), or desktop (3 cols).

---

## 4. Technical Architecture & File Modifications

### Files to Modify
1. **`styles/main.css`**:
   - Update `.notice-grid` to implement responsive media queries:
     - Base: 1 column (`grid-template-columns: 1fr; gap: 14px;`)
     - `@media (min-width: 640px)`: 2 columns (`grid-template-columns: repeat(2, 1fr); gap: 16px;`)
     - `@media (min-width: 1024px)`: 3 columns (`grid-template-columns: repeat(3, 1fr); gap: 20px;`)
   - Update `.notice-card`:
     - Set `display: flex; flex-direction: column; justify-content: space-between; height: 100%;`
     - Card footer anchored to bottom (`margin-top: auto;`)
   - Refine `.notice-grid-scroll`:
     - Responsive max-height limits and sleek scrollbar styling.
2. **`scripts/pages/index-page.js`**:
   - Refine `fitNoticeViewportForFourCards` helper to compute row-based height for responsive multi-column states instead of a naive vertical sum of 4 items.

---

## 5. Testing & Verification Plan

1. **Desktop Test (>= 1024px)**:
   - Verify notices appear in exactly 3 columns side-by-side.
   - Verify card footers and buttons align evenly across rows.
2. **Tablet Test (768px - 1023px)**:
   - Verify notices transition smoothly to 2 columns.
3. **Mobile Test (< 640px)**:
   - Verify notices remain single column without horizontal overflows.
4. **Scrolling Test**:
   - Verify that when > 6 notices are present, vertical scrolling operates smoothly within the container.
5. **Notice Modal Interaction**:
   - Verify that clicking "Expand", "View", or notice images continues to open `#notice-modal` properly.
6. **Build & Lint Verification**:
   - Run `compile_applet` and `lint_applet` to confirm zero regression.

---

## 6. Implementation Status & Verification Results

- [x] **Responsive 3-Column Grid**: Configured in `styles/main.css` (`@media (min-width: 1024px) { grid-template-columns: repeat(3, 1fr); }`, 2 columns on tablet, 1 column on mobile).
- [x] **Card Anatomy & Unified Bottom Alignment**: Updated `.notice-card` with flexbox column layout and `margin-top: auto` for `.notice-footer`, ensuring all cards in each row align their dates and "View" / "Expand" buttons at the same baseline.
- [x] **Dynamic Scroll Container & Height Adaptation**: Updated `fitNoticeViewportForFourCards` in `scripts/pages/index-page.js` to compute grid row heights dynamically, providing smooth scrolling for large sets of circulars.
- [x] **Custom Scrollbar Aesthetics**: Styled `.notice-grid-scroll::-webkit-scrollbar` with rounded thumb and unobtrusive track.
- [x] **Compilation & Lint Verification**: Checked with `compile_applet` and `lint_applet` (`tsc --noEmit`) - both succeeded with 0 errors.

