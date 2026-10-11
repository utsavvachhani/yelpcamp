# 🤖 AI Coding Agent Instructions (`AGENTS.md`)

Welcome to the **YelpCamp** repository. This file serves as the primary entry point and operational contract for all AI coding agents (Antigravity, Cursor, GitHub Copilot, Claude Code, Windsurf, etc.) working on this codebase.

---

## 🚨 Golden Rule: Inspect Before Modifying

> **MANDATORY PRE-FLIGHT CHECK**: Before proposing or writing any code changes, you **MUST** inspect the existing codebase—specifically the relevant files in `pages/`, `components/`, `models/`, and `util/`—to understand established patterns, naming conventions, data structures, and state flows. Never guess imports, schema fields, or component props.

---

## 📚 Standardized Documentation Index

All architectural, design, error-handling, and testing rules are modularized inside the `docs/` directory and project root. **You must consult the relevant guide before executing tasks in that domain:**

| Document | Location | Core Responsibility |
|---|---|---|
| **[README.md](./README.md)** | `/README.md` | Project overview, feature list, environment variables, local setup, directory structure, and API endpoint reference. |
| **[TECH_STACK.md](./docs/TECH_STACK.md)** | `/docs/TECH_STACK.md` | System architecture, Next.js 13 Pages Router conventions, Mongoose 6 ODM schemas & hooks, NextAuth, Firebase, Mapbox, Geoapify, and Stripe integrations. |
| **[UI_UX_GUIDELINES.md](./docs/UI_UX_GUIDELINES.md)** | `/docs/UI_UX_GUIDELINES.md` | Tailwind CSS design tokens (`brand`, `dark`, `primaryBg`, `secondaryBg`), `Volkhov` & `Poppins` typography, reusable UI components, responsiveness, and accessibility. |
| **[ERROR_HANDLING.md](./docs/ERROR_HANDLING.md)** | `/docs/ERROR_HANDLING.md` | Client-side Formik/Yup validation, `react-toastify` error feedback, API route HTTP status conventions, Stripe webhook verification, and security practices. |
| **[TESTING_GUIDELINES.md](./docs/TESTING_GUIDELINES.md)** | `/docs/TESTING_GUIDELINES.md` | Testing pyramid, unit/integration/E2E testing patterns, mocking third-party services (Stripe, Firebase, NextAuth, Mapbox), and pre-commit verification. |

---

## 🧭 Standard Agent Workflow

When assigned a feature, bug fix, or refactoring task, follow this 5-step execution workflow:

### 1. Context Discovery & Codebase Inspection
- Read this `AGENTS.md` file and the domain-specific documents in `docs/`.
- Inspect existing implementations before creating new files:
  - **Database Models**: Check `models/Campground.js`, `models/User.js`, and `models/Review.js`.
  - **Database Helpers**: Check `util/mongo.js`, `util/campgrounds.js`, `util/user.js`, and `util/firebase.js`.
  - **UI Components**: Check `components/` (`Button.js`, `ButtonOutline.js`, `Input.js`, `Modal.js`, `Form.js`, `Layout.js`, `Header.js`) to reuse existing primitives instead of duplicating markup.
  - **Pages & API Routes**: Check `pages/` and `pages/api/` to match existing SSR (`getServerSideProps`) and API handler patterns.

### 2. Architectural Alignment
- **Next.js Pages Router Only**: This project uses the Next.js 13 **Pages Router** (`pages/`). Do **not** introduce the `app/` directory or React Server Components unless explicitly requested.
- **Database Connection Pooling**: Always import and invoke `connectDB()` from `util/mongo.js` inside API routes and server-side data utilities. Never call `mongoose.connect()` directly in route handlers.
- **Model Export Pattern**: Always use `export default models.ModelName || model('ModelName', schema)` to prevent Mongoose model overwrite errors during Next.js hot module replacement (HMR).

### 3. Implementation Standards
- **Code Style**: Follow `.eslintrc.json` and Prettier (`prettier-plugin-tailwindcss`) conventions—single quotes, no semicolons, 2-space indentation, and automatically sorted Tailwind utility classes.
- **UI Consistency**: Use custom theme tokens defined in `tailwind.config.js` (e.g., `bg-brand`, `text-dark`, `text-paragraph`, `bg-primaryBg`, `bg-secondaryBg`, `font-volkhov`, `font-poppins`) rather than arbitrary hex values.
- **Form Validation**: Use **Formik** (`useFormik`) + **Yup** schemas for all client-side forms, paired with server-side validation in API handlers.
- **Image Handling**: Upload user-generated campground images to Firebase Storage (`util/firebase.js`), store `{ url, id }` in MongoDB, and render via `next/image` with configured remote patterns in `next.config.js`.

### 4. Security & Error Handling
- Never expose secret keys (`STRIPE_PRIVATE_KEY`, `STRIPE_WEBHOOK_SECRET`, `NEXTAUTH_SECRET`, `GOOGLE_SECRET`, `MONGODB_URI`) to client-side code (`NEXT_PUBLIC_*` is strictly reserved for Mapbox and Geoapify public tokens).
- Wrap async client operations and API calls in `try/catch` blocks and surface user-friendly messages via `toast.error()` from `react-toastify`.
- Return structured JSON error responses (`{ message: '...' }`) with appropriate HTTP status codes (`400`, `401`, `403`, `404`, `422`, `500`) from API routes.

### 5. Verification & Quality Assurance
- Run `npm run lint` to verify there are no ESLint or Next.js warnings/errors.
- Verify that builds succeed via `npm run build` when making structural, routing, or dependency changes.
- Follow the test specifications in [`docs/TESTING_GUIDELINES.md`](./docs/TESTING_GUIDELINES.md) to validate new endpoints, models, and UI workflows.

---

## 🚫 Prohibited Actions for AI Agents

1. **Do NOT** migrate routes to the Next.js App Router (`app/`) without explicit user instruction.
2. **Do NOT** hardcode API keys, credentials, or environment-specific URLs in source files.
3. **Do NOT** bypass Mongoose middleware hooks (e.g., `findOneAndDelete` and `findOneAndUpdate` hooks in `models/Campground.js` that clean up orphaned Firebase Storage images and user/review references).
4. **Do NOT** enable Next.js `bodyParser` on `pages/api/webhook.js`—Stripe signature verification requires raw request buffers via `micro`.
5. **Do NOT** introduce competing CSS frameworks, icon libraries, or state management libraries when Tailwind CSS, Headless UI, React Icons, and React hooks already cover the use case.
