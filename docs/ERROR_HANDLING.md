# 🛡️ Error Handling, Validation & Security Guidelines (`ERROR_HANDLING.md`)

This document establishes the error handling, input validation, user feedback, and security practices across client and server layers in **YelpCamp**. AI coding agents must enforce these patterns whenever writing or modifying forms, API routes, database hooks, or third-party integrations.

---

## 🧭 Core Principles

1. **Validate Early, Validate Twice**: Enforce schema validation on the client using **Formik + Yup** for immediate UX feedback, and re-validate data integrity, ownership, and session state on the server before mutating MongoDB.
2. **Actionable User Feedback**: Never fail silently on the client. Surface clear, human-readable error notifications using `react-toastify` (`toast.error()`).
3. **Fail Safely & Clean Up**: Ensure partial failures (e.g., failed image uploads or canceled bookings) do not leave orphaned files in Firebase Storage or inconsistent references in MongoDB.
4. **Never Leak Internals**: Do not expose raw stack traces, database connection strings, or third-party secret keys in API responses or client logs.

---

## 🖥️ Client-Side Validation & Error Handling

### 1. Form Validation (`Formik` + `Yup`)
All interactive forms must use `useFormik` paired with a `Yup.object()` schema (see [`components/Form.js`](../components/Form.js) and `pages/auth/`):
- Render inline field-level errors only after a field has been touched (`formik.touched.fieldName && formik.errors.fieldName`).
- Use the standardized [`components/Input.js`](../components/Input.js) component, which styles invalid fields with `border-red` and displays the error message alongside `<FaExclamationCircle />`.
- Example pattern from `components/Form.js`:
  ```javascript
  validationSchema: Yup.object({
    name: Yup.string().required('Please enter a name!'),
    desc: Yup.string()
      .max(750, 'Description cannot be more than 750')
      .required('Please enter a description!'),
    images: Yup.array().min(1, 'Please select atleast 1 image!').required(),
    adultPrice: Yup.number()
      .min(0, 'Price cannot be less than 0!')
      .required('Please enter a price!'),
  })
  ```

### 2. Toast Notifications (`react-toastify`)
A global `<ToastContainer />` is mounted in [`components/Layout.js`](../components/Layout.js) with `position="top-center"`, `theme="colored"`, and custom brand colors in [`styles/globals.css`](../styles/globals.css) (`--toastify-color-success: #00b25a`, `--toastify-color-error: #ef476f`).
- **Success Feedback**: `toast.success('Campground created successfully!')`
- **Error Feedback**:
  ```javascript
  try {
    setLoading(true)
    await axios.post('/api/campgrounds', payload)
    toast.success('Campground created!')
  } catch (err) {
    toast.error(err.response?.data?.message || 'Something went wrong. Please try again.')
  } finally {
    setLoading(false)
  }
  ```

### 3. Third-Party Client Fallbacks (Mapbox, Geoapify & Geolocation)
- **Browser Geolocation**: When requesting `navigator.geolocation.getCurrentPosition`, always supply an error callback that falls back to default coordinates or displays a non-blocking toast warning if the user denies location permissions.
- **Geoapify Reverse Geocoding**: Wrap `axios.get` calls to `api.geoapify.com` in `.catch()` handlers so network failures or missing API keys do not crash the form; allow users to manually type `city`, `state`, and `country` if reverse geocoding fails.

---

## ⚙️ Server-Side & API Route Error Handling (`pages/api/`)

### 1. Standard HTTP Status Codes
API route handlers in `pages/api/` must return consistent HTTP status codes and JSON error payloads (`{ message: '...' }`):

| Status Code | Meaning | When to Use in YelpCamp |
|---|---|---|
| `200 OK` / `201 Created` | Success | Successful resource fetch, creation, update, or deletion. |
| `400 Bad Request` | Malformed Request / Signature Failure | Missing required parameters, invalid Stripe webhook signature (`pages/api/webhook.js`). |
| `401 Unauthorized` | Unauthenticated | Missing or invalid NextAuth session on protected endpoints. |
| `403 Forbidden` | Unauthorized Action | Authenticated user attempting to edit/delete a campground or review they do not own. |
| `404 Not Found` | Resource Missing | Campground, User, or Review ID does not exist in MongoDB. |
| `405 Method Not Allowed` | Invalid HTTP Verb | Request method (`GET`, `POST`, `PATCH`, `DELETE`) is not supported by the handler. |
| `422 Unprocessable Entity`| Validation / Conflict Error | Duplicate email or username during registration (`pages/api/signup.js`), invalid password on login/update. |
| `500 Internal Server Error`| Unexpected Server Failure | Unhandled database or third-party SDK exception. |

### 2. Server-Side Route Template
When creating or updating API routes in `pages/api/`, follow this defensive structure:

```javascript
import connectDB from '../../util/mongo'

export default async function handler(req, res) {
  try {
    await connectDB()

    if (req.method !== 'POST') {
      return res.status(405).json({ message: 'Method not allowed' })
    }

    // 1. Validate inputs / session
    // 2. Perform Mongoose / SDK operation
    return res.status(200).json({ data: result })
  } catch (error) {
    console.error(`[API Error] ${req.url}:`, error.message)
    return res.status(500).json({ message: 'Internal server error' })
  }
}
```

### 3. SSR Error Handling (`getServerSideProps`)
- In dynamic routes like `pages/campgrounds/[id]/index.js`, `edit.js`, or `pages/users/[id]/index.js`, if a resource lookup returns `null` or an invalid MongoDB `ObjectId` is provided:
  - Return `{ notFound: true }` or redirect gracefully (`{ redirect: { destination: '/', permanent: false } }`) rather than allowing `getServerSideProps` to throw an unhandled exception.

---

## 🔐 Security Requirements

### 1. Environment Variable Isolation
- **Server-Only Secrets**: `MONGODB_URI`, `NEXTAUTH_SECRET`, `GOOGLE_SECRET`, `STRIPE_PRIVATE_KEY`, and `STRIPE_WEBHOOK_SECRET` must **never** be prefixed with `NEXT_PUBLIC_` or imported into client-rendered components.
- **Public Tokens**: Only `NEXT_PUBLIC_MAPBOX_ACCESS_KEY` and `NEXT_PUBLIC_GEOAPIFY_KEY` are exposed to the browser bundle.

### 2. Authentication & Ownership Enforcement
- **Middleware Protection**: Keep `/campgrounds/new`, `/api/subscribe`, `/api/campgrounds`, and `/campgrounds/:id/confirm` listed in [`middleware.js`](../middleware.js).
- **Ownership Checks**: Before modifying or deleting a campground (`pages/api/campgrounds/[id]/index.js`) or review (`pages/api/campgrounds/[id]/review.js`), verify that the authenticated user is the resource `owner` / `user`.
- **Password Hashing**: Always hash passwords using `bcryptjs` with a salt work factor of `14` (`await hash(password, 14)`) as implemented in [`pages/api/signup.js`](../pages/api/signup.js). Never store or log plaintext passwords.

### 3. Stripe Webhook Integrity (`pages/api/webhook.js`)
- Always keep `export const config = { api: { bodyParser: false } }` enabled on `pages/api/webhook.js`.
- Always verify the `stripe-signature` header against `process.env.STRIPE_WEBHOOK_SECRET` using `stripe.webhooks.constructEvent(buf, sign, webhookSecret)` before fulfilling bookings, updating user subscriptions, or incrementing host earnings.

### 4. Next.js Image & SVG Content Security Policy
- [`next.config.js`](../next.config.js) enables `dangerouslyAllowSVG: true` (required for DiceBear initial avatars) paired with a strict Content Security Policy:
  `contentSecurityPolicy: "default-src 'self'; script-src 'none'; sandbox;"`
- **Do not weaken or remove this CSP**, and only add trusted domains to `images.remotePatterns`.
