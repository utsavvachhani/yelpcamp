# 🏕️ YelpCamp

A full-stack campground discovery, hosting, and booking platform built with **Next.js 13 (Pages Router)**, **React 18**, **Tailwind CSS 3**, and **MongoDB (Mongoose 6)**. Users can discover campsites worldwide, explore interactive Mapbox cluster maps, host and manage their own campgrounds with Firebase Storage photo uploads, write community reviews, subscribe to **YelpCamp Plus**, and book stays via **Stripe Checkout**.

---

## 📋 Table of Contents
- [AI Agent & Developer Documentation](#-ai-agent--developer-documentation)
- [How YelpCamp Works (End-to-End Architecture)](#-how-yelpcamp-works-end-to-end-architecture)
- [Features](#-features)
- [Tech Stack](#-tech-stack)
- [Prerequisites](#-prerequisites)
- [Environment Variables & Credentials](#-environment-variables--credentials)
- [Step-by-Step Setup Guide](#-step-by-step-setup-guide)
- [Directory Structure](#-directory-structure)
- [API Routes Reference](#-api-routes-reference)
- [Available Scripts](#-available-scripts)

---

## 🤖 AI Agent & Developer Documentation

This repository uses a standardized documentation system for developers and AI coding agents. Start with [`AGENTS.md`](./AGENTS.md) before making changes:

| Document | Purpose |
|---|---|
| **[`AGENTS.md`](./AGENTS.md)** | Primary operational guide and rules for AI coding agents. |
| **[`docs/TECH_STACK.md`](./docs/TECH_STACK.md)** | Detailed architecture, data models, libraries, and third-party service specifications. |
| **[`docs/UI_UX_GUIDELINES.md`](./docs/UI_UX_GUIDELINES.md)** | Design system tokens, typography (`Volkhov` & `Poppins`), component patterns, and layout rules. |
| **[`docs/ERROR_HANDLING.md`](./docs/ERROR_HANDLING.md)** | Client/server validation, toast notifications, HTTP error codes, and security policies. |
| **[`docs/TESTING_GUIDELINES.md`](./docs/TESTING_GUIDELINES.md)** | Testing strategies, mocking external integrations, and quality assurance checklists. |

---

## 🔄 How YelpCamp Works (End-to-End Architecture)

1. **Discovery & SSR Data Fetching**:
   - Landing (`pages/index.js`) and Explore (`pages/campgrounds/index.js`) pages fetch campground listings server-side via `getServerSideProps` using helper utilities (`util/campgrounds.js`, `util/mongo.js`).
   - Users filter campsites by search query, location, price range, and **YelpCamp Plus** exclusivity, while viewing interactive markers on a **Mapbox GL** map.
2. **Authentication & User Profiles**:
   - Powered by **NextAuth.js** (`pages/api/auth/[...nextauth].js`) using JWT sessions.
   - Supports both **Credentials** (email + `bcryptjs` hashed password registered via `/api/signup` with auto-generated DiceBear avatars) and **Google OAuth 2.0**.
3. **Hosting & Managing Campgrounds**:
   - Authenticated users create/edit campgrounds via `components/Form.js` (validated with **Formik** & **Yup**).
   - Dropping a pin or using the Mapbox Geocoder triggers reverse geocoding via the **Geoapify API** to auto-fill `city`, `state`, and `country`.
   - Photos are uploaded directly to **Firebase Storage** (`images/{uuid}`) and saved as `{ url, id }` references in MongoDB. Updating or deleting a campground triggers Mongoose post-hooks in `models/Campground.js` to automatically purge removed images from Firebase Storage.
4. **Bookings, Subscriptions & Webhooks**:
   - **Campsite Reservations**: Selecting check-in/check-out dates and guest counts calls `/api/campgrounds/[id]/checkout`, creating a **Stripe Checkout** session with booking metadata.
   - **YelpCamp Plus**: Subscribing calls `/api/subscribe` to launch a recurring Stripe subscription session.
   - **Webhook Fulfillment**: Stripe sends signed events (`checkout.session.completed`, `customer.subscription.created`) to `/api/webhook`, which verifies the signature using `micro` buffer, records the trip on the camper's `User` document, credits 65% of the booking charge to the host's campground earnings, and pushes an unread host notification.

---

## ✨ Features

- **Campground Discovery & Filtering**: Search campgrounds by name, location, price range, and YelpCamp Plus exclusive tier.
- **Interactive Mapbox Maps**: View campsite locations, geolocate user position, and search addresses with Mapbox GL & Geocoder.
- **Dual Authentication**: NextAuth.js session management supporting Credentials (Email & Password) and Google OAuth.
- **Campground CRUD & Cloud Media**: Create, edit, and delete campgrounds with drag-and-drop multi-image upload and automated cleanup via Firebase Storage.
- **Reverse Geocoding**: Automatic reverse geocoding via Geoapify API to resolve campsite city, state, and country from map coordinates.
- **Reviews & Dynamic Ratings**: Camper community 1–5 star ratings and reviews with automatic average rating recalculation (`Campground.findAndAddReview`).
- **Stripe Payments, Host Earnings & Notifications**: Integrated Stripe Checkout for campsite bookings, booking cancellations (`/api/cancelBooking`), host earnings tracking, notifications, and YelpCamp Plus memberships.
- **Responsive UI**: Built with Tailwind CSS 3, Headless UI, NProgress route indicators, and React Toastify alerts.

---

## 🛠️ Tech Stack

| Layer | Technology |
|---|---|
| **Frontend Framework** | [Next.js 13.0.2](https://nextjs.org/) (Pages Router), [React 18.2.0](https://reactjs.org/) |
| **Styling & UI** | [Tailwind CSS 3.2.2](https://tailwindcss.com/), [Headless UI 1.7](https://headlessui.com/), [React Icons 4.6](https://react-icons.github.io/react-icons/), [React Toastify 9.1](https://fkhadra.github.io/react-toastify/), [NProgress](https://ricostacruz.com/nprogress/) |
| **Forms & Validation** | [Formik 2.2.9](https://formik.org/), [Yup 0.32.11](https://github.com/jquense/yup), [card-validator](https://github.com/braintree/card-validator) |
| **Database & ODM** | [MongoDB](https://www.mongodb.com/), [Mongoose 6.7.2](https://mongoosejs.com/) |
| **Authentication** | [NextAuth.js 4.17.0](https://next-auth.js.org/), [bcryptjs 2.4.3](https://github.com/dcodeIO/bcrypt.js) |
| **Maps & Location** | [Mapbox GL 2.13](https://docs.mapbox.com/mapbox-gl-js/), [React Map GL 7.0](https://visgl.github.io/react-map-gl/), [@mapbox/mapbox-gl-geocoder](https://github.com/mapbox/mapbox-gl-geocoder), [Geoapify API](https://www.geoapify.com/) |
| **Cloud Storage** | [Firebase Storage 9.14.0](https://firebase.google.com/products/storage) |
| **Payment Processing** | [Stripe 11.11.0](https://stripe.com/), [micro 10.0.1](https://github.com/vercel/micro) |

> See [`docs/TECH_STACK.md`](./docs/TECH_STACK.md) for full architectural diagrams and implementation details.

---

## 📦 Prerequisites

Ensure you have the following installed on your system:
- **Node.js** `v18.0.0` or higher (v20+ / v22+ LTS supported)
- **npm** `v9.0.0` or higher
- **MongoDB** running locally on port `27017` or a cloud cluster on [MongoDB Atlas](https://www.mongodb.com/atlas)

---

## 🔐 Environment Variables & Credentials

Create a `.env` file in the root directory (copy from `.env.example`):

```bash
cp .env.example .env
```

### Complete Environment Variables Reference

| Variable | Required | Description | Where to Obtain |
|---|---|---|---|
| `MONGODB_URI` | **Yes** | MongoDB connection string (e.g. `mongodb://127.0.0.1:27017/yelpcamp` or Atlas URI) | Local MongoDB service or [MongoDB Atlas](https://cloud.mongodb.com) |
| `NEXTAUTH_URL` | **Yes** | Canonical base URL for your application (default `http://localhost:3000`) | Self-hosted or deployment URL |
| `NEXTAUTH_URL_INTERNAL` | **Yes** | Internal base URL for NextAuth (default `http://localhost:3000`) | Self-hosted or deployment URL |
| `NEXTAUTH_SECRET` | **Yes** | Secret used to encrypt NextAuth JWT tokens and cookies | Generate using `openssl rand -base64 32` |
| `GOOGLE_ID` | Optional | Google OAuth 2.0 Client ID | [Google Cloud Console](https://console.cloud.google.com/) -> APIs & Services -> Credentials |
| `GOOGLE_SECRET` | Optional | Google OAuth 2.0 Client Secret | [Google Cloud Console](https://console.cloud.google.com/) -> APIs & Services -> Credentials |
| `FIREBASE_KEY` | Optional* | Firebase Web API Key for uploading campground photos (*needed to create/edit campgrounds) | [Firebase Console](https://console.firebase.google.com/) -> Project Settings -> General -> Web API Key |
| `STRIPE_PUBLIC_KEY` | Optional* | Stripe publishable key (*needed for checkout flow) | [Stripe Dashboard](https://dashboard.stripe.com/test/apikeys) -> API Keys (`pk_test_...`) |
| `STRIPE_PRIVATE_KEY` | Optional* | Stripe secret key (*needed for checkout sessions) | [Stripe Dashboard](https://dashboard.stripe.com/test/apikeys) -> API Keys (`sk_test_...`) |
| `STRIPE_WEBHOOK_SECRET` | Optional* | Stripe webhook signing secret (*needed for payment events) | Stripe CLI (`stripe listen --forward-to localhost:3000/api/webhook`) or Stripe Dashboard |
| `NEXT_PUBLIC_MAPBOX_ACCESS_KEY` | Optional* | Mapbox public access token (*needed to render maps) | [Mapbox Account Dashboard](https://account.mapbox.com/access-tokens/) |
| `NEXT_PUBLIC_GEOAPIFY_KEY` | Optional* | Geoapify API key (*needed for reverse geocoding pin coordinates) | [Geoapify Projects Portal](https://myprojects.geoapify.com/) |

> **Note**: For local development and testing, only `MONGODB_URI`, `NEXTAUTH_URL`, and `NEXTAUTH_SECRET` are strictly required to run the application, browse campgrounds, and test credentials-based user signup/login.

---

## 🚀 Step-by-Step Setup Guide

### 1. Clone & Navigate to Project
```bash
cd c:\utsav\personal\yelpcamp
```

### 2. Install Dependencies
```bash
npm install
```

### 3. Configure `.env`
Set up your `.env` file with your local MongoDB connection:
```env
MONGODB_URI=mongodb://127.0.0.1:27017/yelpcamp
NEXTAUTH_URL=http://localhost:3000
NEXTAUTH_URL_INTERNAL=http://localhost:3000
NEXTAUTH_SECRET=secret
```

### 4. Ensure MongoDB is Running
- **Windows (PowerShell)**:
  ```powershell
  Get-Service -Name MongoDB
  # If not running:
  Start-Service -Name MongoDB
  ```
- **macOS / Linux**:
  ```bash
  sudo systemctl start mongod # or brew services start mongodb/brew/mongodb-community
  ```

### 5. Start the Development Server
```bash
npm run dev
```

The application will start on **[http://localhost:3000](http://localhost:3000)**.

---

## 📁 Directory Structure

```plaintext
yelpcamp/
├── components/              # Reusable UI components
│   ├── Button.js            # Primary brand button with loading spinner state
│   ├── ButtonOutline.js     # Outlined secondary/danger button
│   ├── CampgroundCard.js    # Vertical card for featured/grid listings
│   ├── CampgroundListItem.js# Compact list item for user dashboard
│   ├── CampgroundWideCard.js# Horizontal card for search/explore results
│   ├── ComboBox.js          # Headless UI autocomplete select
│   ├── Footer.js            # Site footer
│   ├── Form.js              # Campground creation/edit form (Formik, Mapbox, Firebase)
│   ├── Geocoder.js          # Mapbox GL Geocoder control wrapper
│   ├── Header.js            # Responsive navbar, notifications popover & user menu
│   ├── Input.js             # Standardized form input/textarea with validation error display
│   ├── Layout.js            # Global page wrapper with Header, ToastContainer & Footer
│   ├── LinkButton.js        # Next.js Link styled as a brand button
│   ├── Loader.js            # Route transition loading indicator
│   ├── Modal.js             # Headless UI Dialog modal
│   ├── ModalInput.js        # Modal-specific input control
│   ├── ReviewForm.js        # Star rating & review submission form
│   └── Reviews.js           # Campground review list display
├── docs/                    # Standardized AI Agent & engineering documentation
│   ├── TECH_STACK.md        # Architecture, database schemas & integrations
│   ├── UI_UX_GUIDELINES.md  # Design system, colors, typography & UI rules
│   ├── ERROR_HANDLING.md    # Validation, error handling & security standards
│   └── TESTING_GUIDELINES.md# Testing pyramid, mocking & QA checklists
├── models/                  # Mongoose 6 data schemas & middleware
│   ├── Campground.js        # Campground schema + Firebase cleanup post-hooks
│   ├── Review.js            # Review schema (rating, comment, user, campground)
│   └── User.js              # User schema (auth, premium, campgrounds, trips, notifications)
├── pages/                   # Next.js Pages Router views & API endpoints
│   ├── _app.js              # SessionProvider, Layout wrapper & NProgress router events
│   ├── _document.js         # Custom HTML document & Google Fonts (Volkhov, Poppins)
│   ├── index.js             # Landing page with hero search & top-rated campgrounds
│   ├── plus.js              # YelpCamp Plus membership showcase & checkout trigger
│   ├── user.js              # User dashboard (trips, hosted campgrounds, earnings, bank info)
│   ├── auth/                # Authentication pages (login.js, signup.js)
│   ├── campgrounds/         # Campground explore, new.js, [id]/index.js, [id]/edit.js
│   └── api/                 # Next.js serverless API routes
│       ├── auth/            # NextAuth configuration ([...nextauth].js)
│       ├── campgrounds/     # Campground CRUD, [id]/review.js & [id]/checkout.js
│       ├── notifications/   # Notification state updates (markAllRead.js)
│       ├── cancelBooking.js # Cancels a trip & deducts host earnings
│       ├── signup.js        # Credentials user registration with bcryptjs & DiceBear avatar
│       ├── subscribe.js     # Stripe Checkout session for YelpCamp Plus
│       ├── user.js          # User profile & bank account update endpoint
│       └── webhook.js       # Stripe webhook handler (raw buffer signature verification)
├── public/                  # Static images, logos, and SVG illustrations
├── styles/                  # Global stylesheets
│   ├── globals.css          # Tailwind directives, Toastify CSS variables & bleed utilities
│   └── nprogress.css        # Custom NProgress top bar styling
├── util/                    # Backend & client utility modules
│   ├── campgrounds.js       # Server-side campground query helpers
│   ├── firebase.js          # Firebase app & Cloud Storage initialization
│   ├── mongo.js             # Cached global Mongoose connection manager
│   └── user.js              # Server-side user profile & populated data queries
├── AGENTS.md                # Root AI coding agent instructions & workflow
├── .env.example             # Template environment configuration
├── next.config.js           # Next.js configuration & allowed image domains
├── package.json             # Dependencies and npm scripts
└── tailwind.config.js       # Custom Tailwind theme tokens (colors, fonts, widths)
```

---

## 🔌 API Routes Reference

| Endpoint | Method | Description |
|---|---|---|
| `/api/signup` | `POST` | Registers a new user with username, email, `bcryptjs` hashed password, and DiceBear avatar |
| `/api/auth/[...nextauth]` | `GET`, `POST` | NextAuth endpoint handling JWT sessions, Credentials login, and Google OAuth |
| `/api/campgrounds` | `POST` | Creates a new campground and appends `{ earnings: 0, campground }` to the owner's `User` document |
| `/api/campgrounds/[id]` | `PATCH`, `DELETE` | Updates or deletes an existing campground (triggers Firebase Storage image cleanup hooks) |
| `/api/campgrounds/[id]/review` | `POST`, `DELETE` | Adds or deletes a campground review and recalculates average star rating |
| `/api/campgrounds/[id]/checkout` | `GET` | Creates a Stripe Checkout session for booking a campsite with date/guest metadata |
| `/api/subscribe` | `GET` | Creates a Stripe Checkout session for YelpCamp Plus recurring subscription |
| `/api/webhook` | `POST` | Verifies Stripe webhook signatures (`checkout.session.completed`, `customer.subscription.created`) |
| `/api/cancelBooking` | `POST` | Cancels an existing trip reservation and adjusts host earnings |
| `/api/user` | `POST`, `PATCH` | Updates user profile details, password, or payout bank account details |
| `/api/notifications/markAllRead` | `POST` | Marks all unread host booking notifications as read |

---

## 📜 Available Scripts

- `npm run dev` — Starts the Next.js development server on `http://localhost:3000` with hot-reloading.
- `npm run build` — Compiles and optimizes the application for production.
- `npm run start` — Starts the production Next.js server.
- `npm run lint` — Runs ESLint (`eslint-config-next`) to check code quality.
