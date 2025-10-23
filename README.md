# FlightBridge

**Sync Flight Crew View to LogTen Pro with Automatic Timezone Corrections**

🚀 **Status:** Password Reset Complete! OAuth & Authentication System Ready! 🎉  
🌐 **Live:** https://flightbridge.app/  
📊 **Dashboard:** https://flightbridge.app/webhook/dashboard

---

## ✅ What We've Built So Far

### Phase 1: Foundation ✅ **COMPLETE**
- [x] Domain & DNS setup
- [x] Cloudflare Worker proxy ✅ **FIXED & WORKING**
- [x] Landing page live (with Login button)
- [x] Privacy & Terms pages
- [x] Dashboard UI (ready for real data)
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
- [x] Cloudflare Worker proxy - **FIXED & WORKING** ✅
- [x] Authorization header error - **FIXED** ✅
- [x] Flight data API call - **WORKING** ✅ (returns ~100KB JSON with 60+ flights)

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
- [x] Connect FCView page (`/webhook/connect-fcview?user_id=xxx`)
- [x] User ID flow through OAuth (via state parameter)
- [x] **Password Reset System** ✅ **NEW**

### Phase 4.5: Password Reset ✅ **COMPLETE** 
- [x] Forgot password link on login page
- [x] Forgot password form (`/webhook/forgot-password`)
- [x] Password reset email with token
- [x] Reset password form (`/webhook/reset-password`)
- [x] Token validation & expiration (1 hour)
- [x] Password update workflow
- [x] Security: Same message for existing/non-existing emails
- [x] Token cleanup after successful reset

### Phase 5: Flight Data Sync 🚧 **IN PROGRESS**
- [x] OAuth Callback retrieves flight data from FCView API
- [x] Flight data parsing code node added
- [x] Filters out deadhead flights
- [x] Extracts: flight_number, airports, times, aircraft, runways
- [ ] **CURRENT:** Configure Supabase upsert for mass flight insertion
- [ ] Test complete flow with real user data
- [ ] Background sync workflow (daily at 6am)

---

## 🎯 Current Status: Authentication System Complete!

### ✅ **Latest Progress (Oct 22, 2025, 8:07 PM):**

#### **Password Reset System - Complete:**

**Workflows Created:**
1. **FlightBridge Forgot Password** - GET form at `/webhook/forgot-password`
2. **FlightBridge Forgot Password Handler** - POST to `/webhook/forgot-password-reset`
   - Validates email
   - Generates secure reset token (64-char hex)
   - Sets 1-hour expiration
   - Sends email via Gmail with reset link
3. **FlightBridge Reset Password** - GET form at `/webhook/reset-password?token=xxx`
4. **FlightBridge Reset Password Submit** - POST to `/webhook/reset-password-submit`
   - Validates token & expiration
   - Hashes new password (SHA-256)
   - Updates user password
   - Clears reset token

**Database Changes:**
```sql
-- Added to users table
ALTER TABLE users 
ADD COLUMN reset_token VARCHAR(255),
ADD COLUMN reset_token_expires_at TIMESTAMP;
```

**Security Features:**
- ✅ Tokens expire in 1 hour
- ✅ Tokens cleared after successful reset
- ✅ Same response for existing/non-existing emails (prevents user enumeration)
- ✅ Password validation (minimum 8 characters)
- ✅ Token stored with user, no separate table needed
- ✅ Email sent from support@flightbridge.app

**User Flow:**
1. User clicks "Forgot Password?" on login page
2. Enters email → Receives reset link
3. Clicks link → Opens password reset form
4. Enters new password → Password updated
5. Redirected to login → Can login with new password

---

## 📋 Working URLs

### **Production URLs (ALL WORKING):**
- Landing Page: `https://flightbridge.app/` ✅
- Login/Signup: `https://flightbridge.app/webhook/login` ✅
- **Forgot Password: `https://flightbridge.app/webhook/forgot-password` ✅ NEW**
- **Reset Password: `https://flightbridge.app/webhook/reset-password?token=xxx` ✅ NEW**
- Connect FCView: `https://flightbridge.app/webhook/connect-fcview?user_id=XXX` ✅
- OAuth Start: `https://flightbridge.app/webhook/oauth/start?user_id=XXX` ✅
- OAuth Callback: `https://flightbridge.app/webhook/oauth/callback` ✅
- Dashboard: `https://flightbridge.app/webhook/dashboard?user_id=XXX` ✅
- Terms: `https://flightbridge.app/webhook/terms` ✅
- Privacy: `https://flightbridge.app/webhook/privacy` ✅

### **FCView API Endpoints:**
- Authorization: `https://www.flightcrewview2.com/logbook/logbookuserauth/`
- Token Exchange: `https://www.flightcrewview2.com/logbook/api/token/`
- Flights API: `https://www.flightcrewview2.com/logbook/api/flights/` ✅ **WORKING**

---

## 🗄️ Database Schema

### **users table ✅ UPDATED**
```sql
CREATE TABLE users (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  email VARCHAR(255) UNIQUE NOT NULL,
  password_hash TEXT NOT NULL,
  first_name VARCHAR(100),
  last_name VARCHAR(100),
  fcview_access_token TEXT,
  fcview_refresh_token TEXT,
  fcview_token_expires_at TIMESTAMP,
  
  -- Password reset (NEW)
  reset_token VARCHAR(255),
  reset_token_expires_at TIMESTAMP,
  
  subscription_status VARCHAR(50) DEFAULT 'trial',
  stripe_customer_id VARCHAR(255),
  auto_sync_enabled BOOLEAN DEFAULT true,
  fix_timezones BOOLEAN DEFAULT true,
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW()
);
```

### **flights table ✅ COMPLETE**
```sql
CREATE TABLE flights (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID REFERENCES users(id) ON DELETE CASCADE,
  fcview_flight_id VARCHAR(255) UNIQUE NOT NULL,
  
  flight_number VARCHAR(50),
  departure_airport VARCHAR(10),
  arrival_airport VARCHAR(10),
  departure_time TIMESTAMP,
  arrival_time TIMESTAMP,
  flight_date DATE,
  
  aircraft_type VARCHAR(50),
  tail_number VARCHAR(20),
  departure_runway VARCHAR(10),
  arrival_runway VARCHAR(10),
  duration_minutes INTEGER,
  
  imported_to_logten BOOLEAN DEFAULT false,
  imported_at TIMESTAMP,
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW()
);

CREATE INDEX idx_user_flights ON flights(user_id, flight_date DESC);
```

---

## 🔧 Technical Configuration

### **n8n Workflows (9 Active):**
1. **FlightBridge Landing Page** - Serves homepage
2. **FlightBridge Login** - Shows login/signup form (GET)
3. **FlightBridge Login Auth** - Processes authentication (POST)
4. **FlightBridge Forgot Password** - Shows forgot password form (GET) ✅ **NEW**
5. **FlightBridge Forgot Password Handler** - Sends reset email (POST) ✅ **NEW**
6. **FlightBridge Reset Password** - Shows reset form (GET) ✅ **NEW**
7. **FlightBridge Reset Password Submit** - Updates password (POST) ✅ **NEW**
8. **OAuth Start** - Initiates FCView OAuth with user_id
9. **OAuth Callback** - Receives tokens, fetches & parses flights

### **Complete Authentication Flow:**
```
SIGNUP/LOGIN:
1. User visits /webhook/login
2. Creates account or logs in
3. Password hashed with SHA-256
4. Redirected to dashboard or connect FCView

PASSWORD RESET:
1. User clicks "Forgot Password?"
2. Enters email → /webhook/forgot-password-reset (POST)
3. Generates token, saves to DB, sends email
4. User clicks email link → /webhook/reset-password?token=xxx (GET)
5. Enters new password → /webhook/reset-password-submit (POST)
6. Validates token, hashes password, updates DB, clears token
7. Redirected to login with success message

OAUTH:
1. User connects FCView account
2. OAuth flow exchanges code for tokens
3. Tokens stored in users table
4. Flights fetched and parsed
5. Ready for dashboard display
```

---

## 🚧 **IMMEDIATE NEXT STEPS:**

### **Task #1: Configure Supabase Upsert for Flights (Priority)**
**Goal:** Mass insert flights with automatic updates for existing records

**Steps:**
1. Add Supabase node after "Parse Flight Data" code node
2. **Operation:** Insert
3. **Enable:** On Conflict → Do Update
4. **Conflict Column:** `fcview_flight_id`
5. **Columns to Insert:** All parsed flight fields

### **Task #2: Dashboard Real Data Integration**
- Query flights from Supabase by user_id
- Display pending flights (not imported yet)
- Show flight details: route, date, aircraft
- "Import to LogTen Pro" button per flight

### **Task #3: Background Sync Workflow**
- Scheduled trigger (daily 6am)
- Fetch all users with FCView tokens
- Update flight data for each user
- Handle token refresh if needed

---

## 📝 Implementation Roadmap

### ✅ **Phase 1-4.5: COMPLETE**
- Foundation, API integration, OAuth, User auth, Password reset

### 🚧 **Phase 5: Flight Data Sync (IN PROGRESS)**
- [x] OAuth callback fetches flights
- [x] Flight data parsing
- [ ] **CURRENT:** Supabase batch upsert
- [ ] Dashboard displays real flights
- [ ] Background sync workflow (daily)
- [ ] Timezone correction logic
- [ ] Flight duration calculation

### ⏳ **Phase 6: LogTen Pro Integration (NOT STARTED)**
- [ ] Generate `logten://` URL workflow
- [ ] Format flight data for LogTen Pro
- [ ] Handle timezone conversions
- [ ] Mark flights as imported
- [ ] Test complete sync: FCView → Dashboard → LogTen Pro

### ⏳ **Phase 7: Subscription & Payments (NOT STARTED)**
- [ ] Stripe integration
- [ ] $9.99/month subscription
- [ ] Trial period logic
- [ ] Payment webhook handling
- [ ] Subscription status checking

### ⏳ **Phase 8: Polish & Launch (NOT STARTED)**
- [ ] Email notifications
- [ ] Auto-sync scheduling
- [ ] Error logging & monitoring
- [ ] User settings page
- [ ] Mobile responsiveness testing
- [ ] Beta user testing

---

## 💰 Cost Structure

**Current Monthly Costs:**
- n8n Cloud: $20/month
- Supabase: $0 (free tier, <500MB)
- Cloudflare: $0 (free tier)
- Domain: $0.60/month
- **Total: $20.60/month**

**Break-even: 3 paying users @ $9.99/month**

---

## 📞 Support & Resources

**GitHub:** https://github.com/flightbridge/flightbridge  
**Documentation:** See QUICKSTART.md and DEPLOYMENT.md  
**n8n Cloud:** https://app.n8n.cloud  
**Supabase:** https://supabase.com/dashboard

---

**Last Updated:** October 22, 2025, 8:07 PM PST  
**Current Status:** ✅ Complete authentication system with password reset! Ready for flight data sync.  
**Next Focus:** Configure Supabase batch upsert for flight data, display real flights on dashboard.