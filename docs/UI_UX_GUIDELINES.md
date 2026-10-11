# 🎨 UI/UX & Design System Guidelines (`UI_UX_GUIDELINES.md`)

This document defines the visual language, Tailwind CSS design tokens, typography, layout structure, and reusable component standards for **YelpCamp**. AI coding agents must strictly adhere to these guidelines when creating or updating UI elements.

---

## 🌈 Color Palette & Theme Tokens

All colors are defined in [`tailwind.config.js`](../tailwind.config.js) under `theme.extend.colors`. **Never use arbitrary hex codes or default Tailwind greens/grays when a semantic project token exists.**

| Token Name | Hex Value | Tailwind Classes | Intended Usage |
|---|---|---|---|
| **`brand`** | `#00B25A` | `bg-brand`, `text-brand`, `border-brand` | Primary CTAs, active states, links, badges, success accents |
| **`dark`** | `#222222` | `text-dark`, `bg-dark` | Primary headings, body text default, high-contrast surfaces |
| **`text`** | `#444444` | `text-text` | Secondary headings, form labels, emphasized copy |
| **`paragraph`** | `#666666` | `text-paragraph` | Body paragraphs, descriptions, muted captions, metadata |
| **`primaryBg`** | `#FEFCFB` | `bg-primaryBg` | Warm off-white page surface background |
| **`secondaryBg`**| `#F7F8FC` | `bg-secondaryBg`, `text-secondaryBg` | Card backgrounds, section bands (`.bg-bleed`), button text on brand backgrounds |
| **`lightBlue`** | `#eff0f6` | `bg-lightBlue`, `border-lightBlue` | Input backgrounds, subtle dividers, secondary section bands (`.bg-bleed-lightBlue`) |
| **`blue`** | `#4086F4` | `text-blue`, `bg-blue` | Informational accents, links, secondary highlights |
| **`yellow`** | `#FFB60A` | `text-yellow`, `bg-yellow` | Star ratings, YelpCamp Plus highlights, warnings |
| **`red`** | `#ef476f` | `text-red`, `bg-red`, `border-red` | Danger actions (Delete, Cancel booking), form validation errors |
| **`lightRed`** | `#FFE7DB` | `bg-lightRed` | Soft error/alert background banners |
| **`white`** | `#ffffff` | `bg-white`, `text-white` | Elevated cards, modals, dropdown menus |

---

## 🔤 Typography System

Fonts are loaded via Google Fonts in [`pages/_document.js`](../pages/_document.js) and registered in [`tailwind.config.js`](../tailwind.config.js):

1. **Display & Section Headings (`font-volkhov`)**:
   - Font stack: `['Volkhov', 'Poppins', 'serif']`
   - Usage: Hero titles, major page headings (`h1`, `h2`), and featured callouts.
   - Example: `<h1 className="font-volkhov text-3xl font-bold text-dark md:text-5xl">`
2. **Body, UI Controls & Navigation (`font-poppins`)**:
   - Font stack: `['Poppins', 'serif']`
   - Applied globally on the main container in [`components/Layout.js`](../components/Layout.js).
   - Usage: Navigation links, buttons, form inputs, campground cards, reviews, and body copy.

---

## 📐 Layout, Spacing & Full-Bleed Sections

### Global Page Wrapper (`components/Layout.js`)
Every page is automatically wrapped by `<Layout>` in `pages/_app.js`. Do **not** re-import `<Header />`, `<Footer />`, or `<ToastContainer />` inside individual pages.
- **Container**: `mx-auto flex min-h-[calc(100vh-64px)] max-w-7xl flex-col px-8 pb-12 font-poppins text-dark sm:px-10 md:px-12 lg:min-h-[calc(100vh-96px)] lg:px-14`
- **Custom Width Tokens** (`tailwind.config.js`):
  - `w-sm` (`24rem`), `w-md` (`28rem`), `w-lg` (`32rem`), `w-xl` (`36rem`), `w-2xl` (`48rem`), `w-3xl` (`56rem`), `w-4xl` (`64rem`).

### Full-Bleed Background Utilities (`styles/globals.css`)
When a section needs to break out of the `max-w-7xl` container to span 100% of the viewport width while keeping its inner content aligned:
- Use `.bg-bleed` for a full-viewport `#f7f8fc` (`secondaryBg`) background band.
- Use `.bg-bleed-lightBlue` for a full-viewport `#eff0f6` (`lightBlue`) background band.

---

## 🧩 Reusable Component Catalog (`components/`)

AI agents must inspect and reuse existing components in `components/` before building new UI primitives:

### 1. Buttons & Links
- **[`components/Button.js`](../components/Button.js)**:
  - Primary solid action button (`bg-brand text-secondaryBg`).
  - Automatically renders a spinning `<FaSpinner className="animate-spin" /> Please wait` state when `disabled={true}` is passed.
  - Hover/Focus transition: `hover:border-brand hover:bg-transparent hover:text-brand`.
- **[`components/ButtonOutline.js`](../components/ButtonOutline.js)**:
  - Secondary or destructive outlined button (`danger` prop switches border/text to `red`).
- **[`components/LinkButton.js`](../components/LinkButton.js)**:
  - Next.js `<Link>` styled to match primary CTA buttons.

### 2. Form Controls
- **[`components/Input.js`](../components/Input.js)**:
  - Standardized input & textarea component supporting labels, icons, Formik bindings (`value`, `onChange`, `onBlur`), and inline Yup validation error messages with `<FaExclamationCircle />`.
- **[`components/ComboBox.js`](../components/ComboBox.js)**:
  - Accessible autocomplete dropdown built with `@headlessui/react` `<Combobox>`.
- **[`components/Form.js`](../components/Form.js)**:
  - Comprehensive campground creation/edit form featuring drag-and-drop image upload, amenity tag selector (`amenitiesList`), interactive Mapbox pin placement, and Geoapify reverse geocoding.

### 3. Cards & Listings
- **[`components/CampgroundCard.js`](../components/CampgroundCard.js)**:
  - Vertical card with image carousel/thumbnail, star rating badge, location, and adult price—used on the landing page and grid views.
- **[`components/CampgroundWideCard.js`](../components/CampgroundWideCard.js)**:
  - Horizontal split card used on `/campgrounds` search results alongside the Mapbox map.
- **[`components/CampgroundListItem.js`](../components/CampgroundListItem.js)**:
  - Compact row card used on user dashboards for managing hosted campgrounds or booked trips.

### 4. Feedback, Modals & Navigation
- **[`components/Header.js`](../components/Header.js)**:
  - Responsive top navigation bar with mobile menu drawer, notification bell popover (with unread badge count), and authenticated user profile dropdown.
- **[`components/Modal.js`](../components/Modal.js)** & **[`components/ModalInput.js`](../components/ModalInput.js)**:
  - Accessible dialog overlays powered by `@headlessui/react` `<Dialog>` and `<Transition>`.
- **[`components/Loader.js`](../components/Loader.js)** & **NProgress (`styles/nprogress.css`)**:
  - Displayed automatically during Next.js page transitions.

---

## 📱 Responsiveness & Accessibility Standards

1. **Mobile-First Breakpoints**:
   - Design for mobile (`default`) first, then scale up using `sm:` (`640px`), `md:` (`768px`), `lg:` (`1024px`), and `xl:` (`1280px`).
   - Maps and split-screen layouts should stack vertically on mobile (`flex-col`) and switch to side-by-side (` lg:flex-row` or CSS Grid) on desktop viewports.
2. **Next.js Image Optimization**:
   - Always use `next/image` (`<Image />`) with explicit `width`/`height` or `fill` (with a `relative` parent container) and descriptive `alt` attributes.
3. **Interactive States & Loading Feedback**:
   - Every asynchronous button action (form submit, checkout redirect, review submission, image upload) must disable the trigger button (`disabled={loading}`) to display the spinner and prevent duplicate submissions.
4. **Keyboard & Screen Reader Accessibility**:
   - Use `@headlessui/react` primitives (`Dialog`, `Menu`, `Popover`, `Combobox`) for overlays and menus to ensure focus trapping, `ESC` dismissal, and ARIA compliance.
