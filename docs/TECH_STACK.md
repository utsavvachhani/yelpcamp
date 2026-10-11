# 🛠️ Complete Tech Stack & Architecture Specification (`TECH_STACK.md`)

This is the definitive technical specification and architectural manual for the final production build of **YelpCamp**. It details every layer of the application, including frameworks, databases, third-party APIs, security practices, and a dedicated breakdown of **how image saving and media management works**.

---

## 📋 Table of Contents
1. [System Architecture Overview](#-system-architecture-overview)
2. [Complete Production Tech Stack](#-complete-production-tech-stack)
3. [Deep Dive: Image Saving & Media Storage Architecture](#-deep-dive-image-saving--media-storage-architecture)
4. [Deep Dive: Maps & Geolocation Architecture](#-deep-dive-maps--geolocation-architecture)
5. [Deep Dive: Authentication & Security Architecture](#-deep-dive-authentication--security-architecture)
6. [Deep Dive: Payments, Bookings & Revenue Split](#-deep-dive-payments-bookings--revenue-split)
7. [Database Schema & Connection Pooling](#-database-schema--connection-pooling)
8. [Production Deployment & Infrastructure](#-production-deployment--infrastructure)
9. [Mandatory Rules for AI Agents & Developers](#-mandatory-rules-for-ai-agents--developers)

---

## 🏗️ System Architecture Overview

YelpCamp is designed as a full-stack web platform using the **Next.js 13 Pages Router** architecture. It pairs high-performance React client-side rendering and Server-Side Rendering (SSR) with serverless API route handlers and managed cloud services.

```
┌───────────────────────────────────────────────────────────────────────────────────┐
│                             Client Layer (React 18.2)                             │
│  - UI: Tailwind CSS 3.2, Headless UI 1.7, React Icons 4.6                         │
│  - Form & Validation: Formik 2.2, Yup 0.32, Card-Validator 8.1                    │
│  - Maps: Mapbox GL 2.13, React Map GL 7.0, Mapbox Geocoder                        │
│  - Media Upload: Direct client upload to Firebase Storage (util/uploadImage.js)   │
│  - Session: NextAuth SessionProvider, NProgress route loading bar                 │
└─────────────────────────────────────────┬─────────────────────────────────────────┘
                                          │ HTTP REST / JSON (Axios, Fetch)
┌─────────────────────────────────────────▼─────────────────────────────────────────┐
│                      Server Layer (Next.js 13.0.2 Pages API)                      │
│  - Edge Middleware: middleware.js (route guards via next-auth/middleware)         │
│  - SSR Hydration: getServerSideProps (util/campgrounds.js, util/user.js)          │
│  - Serverless Endpoints: pages/api/** (Auth, Campgrounds, Reviews, Payments)      │
│  - Mongoose Connection Pool: util/mongo.js (cached global promise)                │
└──────────────┬──────────────────────────┬──────────────────────────┬──────────────┘
               │                          │                          │
               ▼                          ▼                          ▼
     ┌──────────────────┐       ┌──────────────────┐       ┌──────────────────┐
     │ MongoDB Database │       │    Stripe API    │       │ Firebase Storage │
     │  (Mongoose 6.7)  │       │(Checkout/Webhook)│       │  & Mapbox APIs   │
     └──────────────────┘       └──────────────────┘       └──────────────────┘
```

---

## 💻 Complete Production Tech Stack

| Layer | Technology | Version | Purpose in Final Product |
|---|---|---|---|
| **Core Framework** | [Next.js](https://nextjs.org/) | `13.0.2` | Full-stack Pages Router framework, SSR, dynamic routing, and API endpoints. |
| **UI Library** | [React](https://reactjs.org/) | `18.2.0` | Declarative component UI and state management (`useState`, `useEffect`, `useRef`). |
| **Styling Engine** | [Tailwind CSS](https://tailwindcss.com/) | `3.2.2` | Utility-first styling with custom theme tokens (`brand`, `dark`, `paragraph`). |
| **UI Primitives** | [Headless UI](https://headlessui.com/) | `1.7.13` | Accessible modals, comboboxes, transition transitions, and dropdown menus. |
| **Icons** | [React Icons](https://react-icons.github.io/react-icons/) | `4.6.0` | Feather, FontAwesome, and Heroicons SVG icon library. |
| **Database** | [MongoDB](https://www.mongodb.com/) | `6.0+` / `8.0+` | NoSQL document database storing campgrounds, users, reviews, and bookings. |
| **ODM Layer** | [Mongoose](https://mongoosejs.com/) | `6.7.2` | Schema modeling, relationship population, validation, and lifecycle hooks. |
| **Authentication** | [NextAuth.js](https://next-auth.js.org/) | `4.17.0` | JWT-based session auth supporting Credentials (email/password) & Google OAuth. |
| **Password Hashing** | [bcryptjs](https://github.com/dcodeIO/bcrypt.js) | `2.4.3` | Salted password hashing before storing credentials in MongoDB. |
| **Media Storage** | [Firebase Storage](https://firebase.google.com/products/storage) | `9.14.0` | Cloud blob storage for multi-image campground photo uploads and lifecycle deletion. |
| **Interactive Maps** | [Mapbox GL](https://docs.mapbox.com/mapbox-gl-js/) & [React Map GL](https://visgl.github.io/react-map-gl/) | `2.13.0` / `7.0.21` | Interactive cluster maps, individual site pinpoints, and viewport controls. |
| **Address Geocoder** | [@mapbox/mapbox-gl-geocoder](https://github.com/mapbox/mapbox-gl-geocoder) | `5.0.1` | Location search input with autocomplete suggestions. |
| **Reverse Geocoding** | [Geoapify](https://www.geoapify.com/) / [Nominatim](https://nominatim.org/) | REST | Converts clicked map GPS coordinates (`lat`, `long`) to human-readable address names. |
| **Payments & Subscriptions** | [Stripe](https://stripe.com/) | `11.11.0` | Stripe Checkout sessions for booking stays, host payout earnings, and YelpCamp Plus. |
| **Webhook Parsing** | [micro](https://github.com/vercel/micro) | `10.0.1` | Reads raw request buffer for Stripe signature verification. |
| **Form Handling** | [Formik](https://formik.org/) | `2.2.9` | Declarative form state handling, change events, and submission flows. |
| **Input Validation** | [Yup](https://github.com/jquense/yup) | `0.32.11` | Client and server object schema validation for listings and reviews. |
| **Date & Calendar** | [react-tailwindcss-datepicker](https://github.com/onesine/react-tailwindcss-datepicker) / [Day.js](https://day.js.org/) | `1.5.0` / `1.11.7` | Check-in/check-out calendar picker, duration math, and formatted review dates. |
| **User Feedback** | [react-toastify](https://fkhadra.github.io/react-toastify/) | `9.1.1` | Toast alerts for operations (success, errors, booking confirmations). |
| **Avatars** | [DiceBear API](https://www.dicebear.com/) | `v5.x` | Automatic unique SVG initials avatars generated per user on registration. |

---

## 📸 Deep Dive: Image Saving & Media Storage Architecture

Campground listings rely heavily on high-resolution photography. Here is exactly how media saving and synchronization is implemented from the browser to cloud storage and database records:

### 1. The Image Upload Workflow

```
[User selects image files in Form]
                 │
                 ▼
[Formik state receives File objects & previews]
                 │
                 ▼
[util/uploadImage.js: uploadAndGetImgUrls()]
   ├── Assigns unique UUID (uuid v4) to each image: `images/${uuid}`
   ├── Direct upload to Firebase Storage Bucket: `uploadBytes(imgRef, file)`
   └── Retrieves public signed download URL: `getDownloadURL(imgRef)`
                 │
                 ▼
[Returns array of objects: [{ id: "uuid", url: "https://firebasestorage..." }]]
                 │
                 ▼
[Next.js API route (/api/campgrounds) saves { id, url } array into MongoDB]
```

### 2. Implementation in the Codebase

* **Client Initialization (`util/firebase.js`)**:
  Initializes Firebase Client SDK using the `FIREBASE_KEY` environment variable:
  ```javascript
  import { initializeApp } from 'firebase/app'
  import { getStorage } from 'firebase/storage'

  const firebaseConfig = {
    apiKey: process.env.FIREBASE_KEY,
    authDomain: 'yelpcamp2-0.firebaseapp.com',
    projectId: 'yelpcamp2-0',
    storageBucket: 'yelpcamp2-0.appspot.com',
  }
  const app = initializeApp(firebaseConfig)
  export const storage = getStorage(app)
  ```

* **Batch Uploader Helper (`util/uploadImage.js`)**:
  Uploads all newly selected image files concurrently using `Promise.all` and preserves any existing images that already have URLs:
  ```javascript
  import { storage } from './firebase'
  import { getDownloadURL, ref, uploadBytes } from 'firebase/storage'

  export const uploadAndGetImgUrls = async images => {
    return await Promise.all(
      images.map(async img => {
        if (img.url) return img // Already uploaded, keep original
        const imgRef = ref(storage, `images/${img.id}`)
        await uploadBytes(imgRef, img.file)
        const url = await getDownloadURL(imgRef)
        return { id: img.id, url }
      })
    )
  }
  ```

* **Database Storage Pattern (`models/Campground.js`)**:
  Images are stored in MongoDB as an array of subdocuments containing both the **URL** and the unique **ID** (needed for deletion):
  ```javascript
  images: {
    type: [{ url: String, id: String }],
    required: true,
  }
  ```

### 3. Automated Lifecycle Deletion (Mongoose Hooks)
To prevent orphaned images and rising storage costs when campgrounds are updated or removed:
* **Campground Deletion Hook (`findOneAndDelete`)**:
  When a campground is deleted, its post-hook deletes all its photos from Firebase Storage:
  ```javascript
  campgroundSchema.post('findOneAndDelete', async doc => {
    await Promise.all(
      doc.images.map(image => deleteObject(ref(storage, `images/${image.id}`)))
    )
    await Review.deleteMany({ _id: { $in: doc.reviews } })
    await User.findByIdAndUpdate(doc.owner, { $pull: { campgrounds: doc._id } })
  })
  ```
* **Campground Edit Hook (`findOneAndUpdate`)**:
  When editing a listing, any previously uploaded images that were removed by the user are automatically deleted from Firebase Storage.

### 4. Next.js Image Optimization Whitelist (`next.config.js`)
Next.js `<Image />` component requires whitelisting remote domains for security and automatic WebP/AVIF compression:
```javascript
images: {
  remotePatterns: [
    { hostname: 'firebasestorage.googleapis.com' },
    { hostname: 'api.dicebear.com' },
    { hostname: 'lh3.googleusercontent.com' },
    { hostname: 'external-content.duckduckgo.com' }
  ]
}
```

### 5. Production Alternatives Comparison for Media Storage

| Solution | Pros | Cons | Best Use Case |
|---|---|---|---|
| **Firebase Storage** *(Current)* | Direct client-to-cloud upload, built-in Google Cloud reliability, free Spark plan (5 GB). | Requires Firebase SDK; transform on the fly needs Firebase Extensions. | Fast setup, great default for Next.js full-stack apps. |
| **Cloudinary** | Automatic thumbnail generation, image cropping, AI compression, responsive sizing URLs. | Paid tiers can become expensive at scale. | Production apps requiring on-the-fly image transformations. |
| **AWS S3 + CloudFront** | Industry standard, cheapest storage cost per GB, global CDN edge caching. | Requires backend pre-signed URL generation and IAM setup. | Enterprise scale deployments with high throughput. |

---

## 🗺️ Deep Dive: Maps & Geolocation Architecture

1. **Map Display & Clustering**:
   * Powered by **Mapbox GL JS** wrapped in `react-map-gl`.
   * **Explore Page (`pages/campgrounds/index.js`)**: Renders a cluster map grouping campsites into count clusters that expand upon zooming.
   * **Campground Detail Page (`pages/campgrounds/[id]/index.js`)**: Renders a pinpoint marker displaying exact latitude and longitude.
2. **Interactive Location Picker (`components/Form.js`)**:
   * Users can click anywhere on an interactive map to set campsite coordinates.
   * **Reverse Geocoding**: The coordinates are dispatched to Geoapify API (or Nominatim / Mapbox Geocoding) which automatically extracts `{ city, state, country }` and fills the form fields.

---

## 🔐 Deep Dive: Authentication & Security Architecture

1. **Session Strategy**:
   * **JWT (JSON Web Tokens)** managed by NextAuth.js.
   * Token encrypted using `NEXTAUTH_SECRET`.
2. **Authentication Providers**:
   * **Local Credentials**: Validates user email and compares bcrypt hash with password stored in MongoDB.
   * **Google OAuth**: One-click Google sign-in using Google Cloud Console credentials (`GOOGLE_ID`, `GOOGLE_SECRET`).
   * **GitHub OAuth**: One-click GitHub sign-in using GitHub Developer App credentials (`GITHUB_ID`, `GITHUB_SECRET`).
   * **Avatar Whitelist**: Profile images from both Google (`lh3.googleusercontent.com`) and GitHub (`avatars.githubusercontent.com`) are whitelisted in `next.config.js`.
3. **Route Guards (`middleware.js`)**:
   * Uses NextAuth middleware to restrict sensitive routes (`/campgrounds/new`, `/api/subscribe`, `/campgrounds/:id/confirm`) to authenticated sessions only.

---

## 💳 Deep Dive: Payments, Bookings & Revenue Split

1. **Campsite Reservation Flow**:
   * Camper selects dates and guest counts (`adults`, `children`, `infants`).
   * `/api/campgrounds/[id]/checkout` generates a Stripe Checkout session with custom pricing:
     $$\text{Total} = (\text{Adults} \times \text{Price} + \text{Children} \times \text{Child Price}) \times \text{Nights}$$
2. **Host Revenue Payout Split (65% / 35%)**:
   * When Stripe fires `checkout.session.completed`, `/api/webhook.js`:
     * Credits the camper's profile with the new trip record.
     * Credits **65% of the total booking charge** to the campground host's `earnings` account:
       ```javascript
       $inc: { 'campgrounds.$.earnings': (amount_total / 100) * 0.65 }
       ```
     * Dispatches an in-app notification to the campground owner.
3. **YelpCamp Plus Subscription**:
   * Recurring membership giving access to exclusive campsites.
   * Handled via `/api/subscribe` and Stripe `customer.subscription.created` webhook.

---

## 🗄️ Database Schema & Connection Pooling

### Connection Pooling (`util/mongo.js`)
In serverless environments like Next.js API routes, database connections must be cached globally to avoid opening new connections on every request:
```javascript
let cached = global.mongoose || (global.mongoose = { conn: null, promise: null })

async function connectDB() {
  if (cached.conn) return cached.conn
  if (!cached.promise) {
    cached.promise = mongoose.connect(MONGODB_URI, { bufferCommands: false })
  }
  cached.conn = await cached.promise
  return cached.conn
}
```

### Models Summary:
* **`Campground`**: `name`, `desc`, `price`, `images [{ id, url }]`, `owner (User)`, `reviews [Review]`, `location`, `rating`, `amenities`, `plusExclusive`.
* **`User`**: `name`, `email`, `password`, `image`, `auth_type`, `premium`, `campgrounds [{ campground, earnings }]`, `trips`, `notifications`.
* **`Review`**: `rating`, `comment`, `user (User)`, `campground (Campground)`.

---

## 🚀 Production Deployment & Infrastructure

* **Frontend & Serverless Host**: [Vercel](https://vercel.com/) or containerized Node.js (Docker).
* **Database**: [MongoDB Atlas](https://www.mongodb.com/atlas) (M0 free cluster or M10 production cluster).
* **Media Bucket**: [Firebase Storage](https://firebase.google.com/) or Google Cloud Storage bucket.
* **Environment Configuration**: Managed via Vercel Project Settings or `.env.production`.

---

## 📌 Mandatory Rules for AI Agents & Developers

1. **Pages Router Standard**: Do not migrate to or create the `app/` directory. All routes must live in `pages/`.
2. **Always Call `connectDB()`**: Every API handler or server-side utility must call `await connectDB()` before any model queries.
3. **Image Lifecycle Integrity**: Always delete photos from Firebase Storage when updating or removing campgrounds to prevent orphaned files.
4. **Never Expose Private Keys**: Secrets (`STRIPE_PRIVATE_KEY`, `STRIPE_WEBHOOK_SECRET`, `NEXTAUTH_SECRET`, `FIREBASE_KEY`) must never have `NEXT_PUBLIC_` prefixes or be returned in client API responses.
