# 🏕️ YelpCamp

> Note: For the complete and detailed documentation, see [README.md](file:///c:/utsav/personal/yelpcamp/README.md).

## 🔐 Required Credentials & Environment Variables

Create or update your `.env` file in the project root:

```env
# ==============================================================================
# 1. Database (REQUIRED)
# ==============================================================================
MONGODB_URI=mongodb://127.0.0.1:27017/yelpcamp

# ==============================================================================
# 2. NextAuth Authentication (REQUIRED)
# ==============================================================================
NEXTAUTH_URL=http://localhost:3000
NEXTAUTH_URL_INTERNAL=http://localhost:3000
NEXTAUTH_SECRET=secret

# ==============================================================================
# 3. Google OAuth Provider (OPTIONAL - email/password works out of the box)
# ==============================================================================
GOOGLE_ID=
GOOGLE_SECRET=

# ==============================================================================
# 4. Firebase Storage (REQUIRED FOR PHOTO UPLOADS)
# ==============================================================================
# Web API Key from Firebase Console -> Project Settings -> General
FIREBASE_KEY=

# ==============================================================================
# 5. Stripe Payments (REQUIRED FOR BOOKING CHECKOUT & PLUS MEMBERSHIPS)
# ==============================================================================
STRIPE_PUBLIC_KEY=pk_test_...
STRIPE_PRIVATE_KEY=sk_test_...
STRIPE_WEBHOOK_SECRET=whsec_...

# ==============================================================================
# 6. Map & Geolocation Services (REQUIRED FOR MAPS & REVERSE GEOCODING)
# ==============================================================================
# Mapbox token from https://account.mapbox.com/
NEXT_PUBLIC_MAPBOX_ACCESS_KEY=
# Geoapify key from https://myprojects.geoapify.com/
NEXT_PUBLIC_GEOAPIFY_KEY=
```

---

## 🚀 Setup and Run Project

1. **Install dependencies**:
   ```bash
   npm install
   ```

2. **Ensure MongoDB is running**:
   ```powershell
   Get-Service -Name MongoDB
   # If not running:
   Start-Service -Name MongoDB
   ```

3. **Start the development server**:
   ```bash
   npm run dev
   ```

4. **Access the application**:
   Open [http://localhost:3000](http://localhost:3000) in your web browser.
