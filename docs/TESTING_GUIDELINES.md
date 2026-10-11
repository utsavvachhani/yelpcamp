# 🧪 Testing & Quality Assurance Guidelines (`TESTING_GUIDELINES.md`)

This document defines the testing strategy, mocking standards, and verification workflows for **YelpCamp**. AI coding agents and developers must validate all changes against these guidelines before completing any task.

---

## 🏛️ Testing Pyramid & Strategy

YelpCamp's full-stack Next.js Pages Router architecture requires verification across four layers:

```
          ┌───────────────────────┐
          │   E2E / User Flows    │  <- Auth, Booking Checkout, Campground CRUD
          ├───────────────────────┤
          │   API & Integration   │  <- pages/api/* handlers & Mongoose hooks
          ├───────────────────────┤
          │   Component / UI      │  <- Formik validation, Cards, Modals, Buttons
          ├───────────────────────┤
          │   Unit & Utilities    │  <- Rating calculation, date/price math, helpers
          └───────────────────────┘
```

| Layer | Scope | Recommended Tools |
|---|---|---|
| **Static Analysis** | Code formatting, Next.js rules, React hooks rules | `eslint` (`npm run lint`), `prettier`, `next build` |
| **Unit & Model Tests** | Mongoose schemas (`models/`), rating math, utility functions (`util/`) | Jest / Vitest + `mongodb-memory-server` |
| **Component Tests** | Form validation (`components/Form.js`, `ReviewForm.js`), UI states (`Button.js`, `Modal.js`) | React Testing Library (`@testing-library/react`) + `@testing-library/user-event` |
| **API Route Tests** | Serverless handlers in `pages/api/**` | `node-mocks-http` or Supertest |
| **End-to-End (E2E)** | Full browser workflows (Signup -> Create Camp -> Review -> Checkout) | Playwright or Cypress |

---

## 🔬 Domain-Specific Testing Requirements

### 1. Database Models & Mongoose Hooks (`models/`)
When modifying schemas in `models/Campground.js`, `models/User.js`, or `models/Review.js`, verify:
- **Schema Validation**:
  - `Campground`: Rejects missing `name`, `desc` > 750 characters, missing `price`, missing `location`, or missing `owner`.
  - `User`: Enforces `auth_type` enum (`['credentials', 'google']`) and minimum password length (`8`).
- **Average Rating Calculation (`Campground.findAndAddReview`)**:
  - Adding the first review sets `campground.rating` equal to `review.rating`.
  - Adding subsequent reviews accurately computes the weighted average to 1 decimal place (`toFixed(1)`) and appends `review._id` to `campground.reviews`.
- **Post-Delete & Post-Update Cleanup Hooks**:
  - Deleting a campground via `findOneAndDelete` invokes Firebase `deleteObject` for all campground images, removes associated `Review` documents, and pulls the campground ID from the owner's `User.campgrounds` array.
  - Updating a campground via `findOneAndUpdate` deletes only the images removed from the updated `images` array while preserving retained images.

### 2. API Routes (`pages/api/`)
Every API endpoint must be tested for both happy paths and error edge cases:
- **`POST /api/signup`**:
  - Creates a new user with a `bcryptjs` hashed password (never plaintext) and a DiceBear avatar URL.
  - Returns `422` when `email` or `username` already exists.
- **`POST /api/campgrounds` & `PATCH / DELETE /api/campgrounds/[id]`**:
  - Creates/updates/deletes campground documents and keeps `User.campgrounds` in sync.
- **`POST /api/webhook` (Stripe Webhook)**:
  - Rejects requests missing a valid `stripe-signature` header with `400 Bad Request`.
  - On `checkout.session.completed`:
    1. Appends the booked trip to the camper's `User.trips` array with `charge = amount_total / 100`.
    2. Increments the host's campground earnings by `65%` of the booking charge (`(amount_total / 100) * 0.65`).
    3. Pushes an unread notification (`read: false`) with guest counts and dates to the owner's `User.notifications` array.
  - On `customer.subscription.created`:
    1. Updates `User.premium.subscribed` to `true` for the customer's email.

### 3. UI Components & Forms (`components/`)
- **`components/Form.js` (Campground Form)**:
  - Attempting to submit an empty form displays Yup validation errors (`Please enter a name!`, `Please enter a description!`, `Please select atleast 1 image!`, `Please select atleast 1 amenity`, `You must agree to terms and conditions to continue`).
  - Toggling `plusExclusive` is only enabled when `subscribed={true}` is passed.
- **`components/Button.js`**:
  - Renders `text` prop when enabled and switches to a disabled button with `<FaSpinner className="animate-spin" /> Please wait` when `disabled={true}`.

---

## 🎭 Mocking Third-Party Services in Tests

Automated tests must **never** make live network calls to production Stripe, Firebase Storage, Mapbox, or Geoapify APIs. Always mock external services:

1. **Firebase Storage (`firebase/storage` & `util/uploadImage.js`)**:
   - Mock `uploadBytes`, `getDownloadURL`, and `deleteObject` to resolve deterministically (e.g., returning `https://firebasestorage.googleapis.com/mock-image.jpg`).
2. **Stripe SDK (`stripe`)**:
   - Mock `stripe.checkout.sessions.create` to return `{ url: 'https://checkout.stripe.com/test-session' }`.
   - Mock `stripe.webhooks.constructEvent` in webhook unit tests to return synthetic `checkout.session.completed` and `customer.subscription.created` event objects.
3. **NextAuth (`next-auth/react` & `next-auth/next`)**:
   - Mock `useSession()` on the client to test both authenticated (`status: 'authenticated'`, `data: { user: ... }`) and unauthenticated (`status: 'unauthenticated'`) UI states.
4. **Mapbox GL (`mapbox-gl` & `react-map-gl`)**:
   - Mock `react-map-gl` components (`ReactMapGl`, `Marker`, `GeolocateControl`) in Jest/RTL environments since WebGL canvas context is not natively available in JSDOM.
5. **Geoapify Reverse Geocoding (`axios`)**:
   - Mock `axios.get` for `https://api.geoapify.com/v1/geocode/reverse` to return `{ data: { features: [{ properties: { city: 'Manali', state: 'Himachal Pradesh', country: 'India' } }] } }`.

---

## ✅ Pre-Completion Verification Checklist (Mandatory for AI Agents)

Before marking any coding task as complete, AI agents must run and verify the following checks:

1. **Linter Check**:
   ```bash
   npm run lint
   ```
   *Must exit with 0 errors and 0 warnings.*

2. **Production Build Verification**:
   ```bash
   npm run build
   ```
   *Must compile cleanly without Next.js SSR, static generation, or module resolution errors.*

3. **Functional Smoke Test Checklist**:
   - [ ] Existing routes and API endpoints continue to function without regressions.
   - [ ] All Mongoose queries call `await connectDB()` first.
   - [ ] All new images use `next/image` with whitelisted domains in `next.config.js`.
   - [ ] UI changes adhere to [`docs/UI_UX_GUIDELINES.md`](./UI_UX_GUIDELINES.md) tokens and responsive breakpoints.
   - [ ] Error states are handled gracefully according to [`docs/ERROR_HANDLING.md`](./ERROR_HANDLING.md).
