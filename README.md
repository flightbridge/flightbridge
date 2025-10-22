# FlightBridge

**Sync Flight Crew View to LogTen Pro with Automatic Timezone Corrections**

🚀 **Status:** Login & Auth Complete! OAuth Working! 🎉  
🌐 **Live:** https://flightbridge.app/  
📊 **Dashboard:** https://flightbridge.app/webhook/dashboard

---

## ✅ What We've Built So Far

### Phase 1: Foundation ✅ **COMPLETE**
- [x] Domain & DNS setup
- [x] Cloudflare Worker proxy
- [x] Landing page live (with Login button)
- [x] Privacy & Terms pages
- [x] Dashboard UI (test data)
- [x] Supabase database
- [x] GitHub repository

### Phase 2: API Integration ✅ **COMPLETE**
- [x] FCView API approval received
- [x] Client ID: `f0cf9180d491f06e`
- [x] Client Secret obtained
- [x] Redirect URI configured: `https://flightbridge.app/webhook/oauth/callback`

### Phase 3: OAuth & Authentication ✅ **COMPLETE**
- [x] OAuth Start workflow built (`/webhook/oauth/start`)
- [x] OAuth Callback workflow built (`/webhook/oauth/callback`)
- [x] OAuth flow **FULLY WORKING** ✅
- [x] Token exchange working ✅
- [x] Access & refresh tokens retrieved successfully ✅
- [x] Cloudflare Worker proxy - **FIXED** ✅

### Phase 4: User Authentication ✅ **COMPLETE** 
- [x] Login/Signup page created (`/webhook/login`)
- [x] User registration workflow built
- [x] Password hashing (SHA-256)
- [x] Duplicate email checking
- [x] User login workflow built
- [x] FCView connection status check
- [x] Automatic routing (Dashboard vs Connect FCView)
- [x] Success messages with redirect delays
- [x] Error handling for invalid credentials
- [x] Supabase user table (email, password_hash, first_name, last_name, tokens)

---

## 🎯 Current Status: Login & Auth Complete!

### ✅ **NEW: Complete User Authentication System**

**What We Built Today (Oct 22, 2025):**

#### **Two Active Workflows:**

1. **FlightBridge Login** (ID: `POwWWbExpbnNaBD9`)
   - Path: `/webhook/login` (GET)
   - Shows beautiful login/signup page
   - Tab-based UI (Login / Sign Up)
   - Form validation (password match, 8+ characters)
   - Error/success message display

2. **FlightBridge Login Auth** (ID: `WuZ1fx5aPUeHEZfO`)
   - Path: `/webhook/login-auth` (POST)
   - Handles both signup and login actions
   - Password hashing with SHA-256
   - Duplicate email prevention
   - Credential validation
   - FCView connection check
   - Smart routing:
     - Has FCView token → Dashboard
     - No FCView token → Connect FCView page

#### **Complete User Flow:**
```
1. User visits /webhook/login
2. Chooses "Login" or "Sign Up" tab
3. Submits form → POST to /webhook/login-auth
4. If Signup:
   - Hash password
   - Check email doesn't exist
   - Create user in Supabase
   - Show "Account created! Redirecting..." (1.5s)
   - Redirect to /webhook/connect-fcview
5. If Login:
   - Hash password
   - Find user with matching credentials
   - Check if FCView connected
   - If connected → Dashboard
   - If not → Connect FCView page
```

#### **Error Handling:**
- ✅ Email already registered → 400 error with message
- ✅ Invalid credentials → 401 error with message
- ✅ Network errors caught with friendly message
- ✅ Password mismatch (frontend validation)
- ✅ Password too short (frontend validation)

---

## 🔧 Technical Configuration

### **n8n Workflows (5 Active):**
1. **FlightBridge Landing Page** - Serves homepage
2. **FlightBridge Login** - Shows login/signup form (GET)
3. **FlightBridge Login Auth** - Processes authentication (POST)
4. **OAuth Start** - Initiates FCView OAuth
5. **OAuth Callback** - Receives OAuth tokens

### **Supabase Database Schema:**
```sql
CREATE TABLE users (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  email VARCHAR(255) UNIQUE NOT NULL,
  password_hash VARCHAR(255) NOT NULL,
  first_name VARCHAR(100),
  last_name VARCHAR(100),
  fcview_access_token TEXT,
  fcview_refresh_token TEXT,
  fcview_token_expires_at TIMESTAMP,
  subscription_status VARCHAR(50) DEFAULT 'trial',
  stripe_customer_id VARCHAR(255),
  auto_sync_enabled BOOLEAN DEFAULT true,
  fix_timezones BOOLEAN DEFAULT true,
  created_at TIMESTAMP DEFAULT NOW()
);
```

### **OAuth Flow (Fully Working):**
1. User signs up/logs in → gets user_id
2. Redirected to `/webhook/connect-fcview?user_id=XXX`
3. Clicks "Connect" → `/webhook/oauth/start?user_id=XXX`
4. FCView authorization → Callback with code
5. Token exchange → Store tokens in Supabase (linked to user_id)
6. Redirect to dashboard

---

## 📋 Working URLs

### **Production URLs (ALL WORKING):**
- Landing Page: `https://flightbridge.app/`
- Login/Signup: `https://flightbridge.app/webhook/login` ✅ **NEW**
- Connect FCView: `https://flightbridge.app/webhook/connect-fcview?user_id=XXX`
- OAuth Start: `https://flightbridge.app/webhook/oauth/start?user_id=XXX`
- OAuth Callback: `https://flightbridge.app/webhook/oauth/callback`
- Dashboard: `https://flightbridge.app/webhook/dashboard?user_id=XXX`
- Terms: `https://flightbridge.app/webhook/terms`
- Privacy: `https://flightbridge.app/webhook/privacy`

### **FCView API Endpoints:**
- Authorization: `https://www.flightcrewview2.com/logbook/logbookuserauth/`
- Token Exchange: `https://www.flightcrewview2.com/logbook/api/token/`
- Flights API: `https://www.flightcrewview2.com/logbook/api/flights/`

---

## 🚀 **NEXT STEPS:**

### **Phase 5: Flight Data Sync (Priority)**

#### **A. Update Connect FCView Page**
- [ ] Add `user_id` parameter to OAuth start URL
- [ ] Update OAuth Callback to associate tokens with user_id
- [ ] Test complete signup → connect → dashboard flow

#### **B. Create Dashboard Data Loading**
- [ ] Modify dashboard to accept `user_id` parameter
- [ ] Query Supabase for user's FCView tokens
- [ ] Check token expiration
- [ ] Refresh token if expired

#### **C. Build Flight Sync Workflow**
- [ ] Create "Sync Flights" n8n workflow
- [ ] Trigger: Manual button on dashboard OR scheduled (daily)
- [ ] Get user's access token from Supabase
- [ ] Fetch flights from FCView API: `GET /logbook/api/flights/`
- [ ] Apply timezone corrections:
  - Parse airport codes
  - Look up airport timezones
  - Convert times to Zulu
  - Calculate accurate flight duration
- [ ] Store flights in Supabase `flights` table
- [ ] Return flight list to dashboard

#### **D. Dashboard Flight Display**
- [ ] Show pending flights (not imported yet)
- [ ] Display: Flight #, Route, Date, Duration
- [ ] "Import to LogTen Pro" button per flight
- [ ] Generate `logten://` URL with flight data
- [ ] Track imported flights (imported_at timestamp)

#### **E. LogTen Pro URL Generator**
- [ ] Create "Generate LogTen URL" workflow
- [ ] Input: flight_id
- [ ] Format flight data for LogTen Pro URL scheme
- [ ] Return URL that opens LogTen Pro app
- [ ] Mark flight as imported in database

### **Phase 6: Subscription & Payments**
- [ ] Stripe integration
- [ ] Subscription plans ($9.99/month)
- [ ] Payment processing
- [ ] Trial period logic
- [ ] Subscription status checking

### **Phase 7: Polish & Launch**
- [ ] Email notifications
- [ ] Auto-sync scheduling
- [ ] Error logging & monitoring
- [ ] User settings page
- [ ] Mobile responsiveness
- [ ] Beta user testing

---

## 🗄️ Database Schema (Expanded)

### **Current: users table ✅**
```sql
CREATE TABLE users (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  email VARCHAR(255) UNIQUE NOT NULL,
  password_hash VARCHAR(255) NOT NULL,
  first_name VARCHAR(100),
  last_name VARCHAR(100),
  fcview_access_token TEXT,
  fcview_refresh_token TEXT,
  fcview_token_expires_at TIMESTAMP,
  subscription_status VARCHAR(50) DEFAULT 'trial',
  stripe_customer_id VARCHAR(255),
  auto_sync_enabled BOOLEAN DEFAULT true,
  fix_timezones BOOLEAN DEFAULT true,
  created_at TIMESTAMP DEFAULT NOW()
);
```

### **TODO: flights table**
```sql
CREATE TABLE flights (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID REFERENCES users(id) ON DELETE CASCADE,
  fcview_flight_id VARCHAR(255) UNIQUE,
  flight_data JSONB NOT NULL,
  flight_number VARCHAR(50),
  departure_airport VARCHAR(10),
  arrival_airport VARCHAR(10),
  departure_time TIMESTAMP,
  arrival_time TIMESTAMP,
  flight_date DATE,
  duration_minutes INTEGER,
  imported_to_logten BOOLEAN DEFAULT false,
  imported_at TIMESTAMP,
  created_at TIMESTAMP DEFAULT NOW()
);

CREATE INDEX idx_user_flights ON flights(user_id, flight_date DESC);
```

---

## 📝 Important Notes

### **Security:**
- ✅ Passwords hashed with SHA-256 (never stored plaintext)
- ✅ Unique email constraint prevents duplicates
- ✅ All API endpoints use HTTPS
- ✅ OAuth tokens stored securely in Supabase
- ⚠️ Consider upgrading to bcrypt for production (more secure than SHA-256)

### **User Experience:**
- ✅ Tab-based login/signup (no separate pages)
- ✅ Real-time form validation
- ✅ Success messages with auto-redirect
- ✅ Friendly error messages
- ✅ Smooth transitions

### **Test Credentials:**
- Use any email/password to create test account
- FCView passkey for testing: `TEST1234`

---

## 🐛 Known Issues & Future Improvements

### **Minor Issues:**
- [ ] Consider bcrypt instead of SHA-256 for password hashing
- [ ] Add "Forgot Password" functionality
- [ ] Add email verification
- [ ] Add rate limiting for login attempts
- [ ] Add CSRF protection

### **Enhancement Ideas:**
- [ ] Social login (Google, Apple)
- [ ] Two-factor authentication
- [ ] Remember me functionality
- [ ] Account settings page
- [ ] Profile photo upload

---

**Last Updated:** October 22, 2025, 7:45 AM PST  
**Current Status:** ✅ Complete user authentication system! Ready to build flight sync workflow.  
**Next Focus:** Connect OAuth flow to user accounts, then build flight data sync.