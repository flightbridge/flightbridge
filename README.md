# FlightBridge

**Sync Flight Crew View to LogTen Pro with Automatic Timezone Corrections**

🚀 **Status:** Dashboard UI Complete with Real Flight Data! 🎉  
🌐 **Live:** https://flightbridge.app/  
📊 **Dashboard:** https://flightbridge.app/webhook/dashboard

---

## ✅ What We've Built So Far

### Phase 1: Foundation ✅ **COMPLETE**
- [x] Domain & DNS setup
- [x] Cloudflare Worker proxy ✅ **FIXED & WORKING**
- [x] Landing page live (with Login button)
- [x] Privacy & Terms pages
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
- [x] Flight data API call - **WORKING** ✅ (returns ~100KB JSON with 60+ flights)

### Phase 4: User Authentication ✅ **COMPLETE** 
- [x] Login/Signup page created (`/webhook/login`)
- [x] User registration workflow built
- [x] Password hashing (SHA-256)
- [x] User login workflow built
- [x] FCView connection status check
- [x] Connect FCView page (`/webhook/connect-fcview?user_id=xxx`)
- [x] **Password Reset System** ✅

### Phase 5: Flight Data Sync & Dashboard ✅ **COMPLETE**
- [x] OAuth Callback retrieves flight data from FCView API
- [x] Flight data parsing code node added
- [x] Filters out deadhead flights
- [x] Database stores 414 flights for test user
- [x] **Dashboard displays real flight data** ✅
- [x] **Fixed "Get Flights" Supabase query bug** ✅
- [x] **Built complete dashboard UI** ✅
- [x] Flight cards show: trip number, block, scheduled/actual times, duration, aircraft
- [x] Expandable "More Details" section with UTC times and tail number
- [x] Side-by-side scheduled vs actual times layout
- [x] Pending/Imported status badges
- [x] Import to LogTen Pro buttons (ready for Phase 6)

---

## 🎨 Dashboard Features (Just Built!)

### **Flight Display:**
- ✅ Shows all user flights from database (414 flights working!)
- ✅ Clean card layout with flight number and status
- ✅ Route display (ATL → MLU)
- ✅ Trip number from FCView
- ✅ Block time
- ✅ Scheduled departure & arrival (side-by-side)
- ✅ Actual departure & arrival (side-by-side)
- ✅ Flight duration
- ✅ Expandable "More Details" section with:
  - Tail number
  - Actual departure UTC
  - Actual arrival UTC

### **Header:**
- ✅ Green "Connected to FCView" status box
- ✅ Last synced timestamp
- ✅ Sync Now button (refreshes page)
- ✅ Settings button (placeholder)

---

## 📋 All Working URLs

### **Public Pages:**
- **Landing Page:** `https://flightbridge.app/` ✅
- **Login/Signup:** `https://flightbridge.app/webhook/login` ✅
- **Terms of Service:** `https://flightbridge.app/webhook/terms` ✅
- **Privacy Policy:** `https://flightbridge.app/webhook/privacy` ✅

### **Authentication Flow:**
- **Forgot Password Form:** `https://flightbridge.app/webhook/forgot-password` ✅
- **Reset Password Form:** `https://flightbridge.app/webhook/reset-password?token=xxx` ✅
- **Login Auth Handler:** `POST https://flightbridge.app/webhook/login-auth` ✅

### **User Dashboard:**
- **Dashboard:** `https://flightbridge.app/webhook/dashboard?user_id=XXX` ✅ **REAL DATA!**
- **Connect FCView:** `https://flightbridge.app/webhook/connect-fcview?user_id=XXX` ✅

### **OAuth Flow:**
- **OAuth Start:** `https://flightbridge.app/webhook/oauth/start?user_id=XXX` ✅
- **OAuth Callback:** `https://flightbridge.app/webhook/oauth/callback` ✅

---

## 🗄️ Database Schema

### **users table**
```sql
CREATE TABLE users (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  email VARCHAR(255) UNIQUE NOT NULL,
  password_hash TEXT NOT NULL,
  first_name VARCHAR(100),
  last_name VARCHAR(100),
  
  -- FCView OAuth
  fcview_access_token TEXT,
  fcview_refresh_token TEXT,
  fcview_token_expires_at TIMESTAMP,
  
  -- Password reset
  reset_token VARCHAR(255),
  reset_token_expires_at TIMESTAMP,
  
  -- Subscription
  subscription_status VARCHAR(50) DEFAULT 'trial',
  stripe_customer_id VARCHAR(255),
  
  -- Settings
  auto_sync_enabled BOOLEAN DEFAULT true,
  fix_timezones BOOLEAN DEFAULT true,
  
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW()
);
```

### **flights table**
```sql
CREATE TABLE flights (
  user_id UUID REFERENCES users(id),
  fcview_flight_id VARCHAR(255) UNIQUE,
  flight_number VARCHAR(50),
  departure_airport VARCHAR(10),
  arrival_airport VARCHAR(10),
  
  -- Times
  scheduled_out_local TIMESTAMP,
  scheduled_in_local TIMESTAMP,
  actual_out_local TIMESTAMP,
  actual_in_local TIMESTAMP,
  actual_out_local_utc TIMESTAMP,
  actual_in_local_utc TIMESTAMP,
  
  -- Flight details
  Block VARCHAR(10),
  flight_duration VARCHAR(10),
  fcview_trip_number VARCHAR(50),
  tail_number VARCHAR(20),
  
  -- Sync status
  imported_to_logten BOOLEAN DEFAULT false,
  imported_at TIMESTAMP,
  created_at TIMESTAMP DEFAULT NOW()
);

CREATE INDEX idx_user_flights ON flights(user_id, actual_out_local DESC);
```

---

## 🎯 Current Status: Dashboard Complete!

### ✅ **Latest Progress (Oct 23, 2025, 9:40 AM):**

**Dashboard UI Build - Complete:**
1. ✅ Fixed Supabase "Get Flights" node filter bug (`.item` → `.first()`)
2. ✅ All 414 flights now load correctly
3. ✅ Built complete dashboard with real data
4. ✅ Data transformation node processes all flight fields
5. ✅ HTML generation with side-by-side scheduled/actual times
6. ✅ Expandable details section for UTC times and tail number
7. ✅ Clean, professional UI matching user requirements
8. ✅ Import buttons ready (awaiting LogTen URL generation)

**Key Technical Fixes:**
- Fixed expression: `$('Get User').first().json.id` (was `.item`)
- Removed Handlebars dependency (not allowed in n8n)
- Built pure JavaScript template generation
- Side-by-side time display layout
- Moved tail number to expandable section
- Green status box stays within bounds

---

## 🚧 **IMMEDIATE NEXT STEPS:**

### **Task #1: Generate LogTen Pro Import URLs (Priority)**
**Goal:** Create `logten://` URLs that open LogTen Pro with pre-filled flight data

**Steps:**
1. Create new workflow: "Generate LogTen URL"
2. Trigger: Webhook `/webhook/generate-logten?user_id=xxx&flight_id=yyy`
3. Get flight from database
4. Transform data to LogTen Pro format:
   - Convert timestamps to LogTen format
   - Calculate durations in HH:MM
   - Apply timezone corrections
   - Format aircraft type
5. Build URL: `logten://import?data={encoded_json}`
6. Return URL to dashboard
7. Dashboard opens URL → LogTen Pro imports flight

### **Task #2: Background Sync Workflow**
- Scheduled trigger (daily 6am)
- Get all users with valid FCView tokens
- Fetch new flights from FCView API
- Upsert to database
- Handle token refresh if needed

### **Task #3: Settings Page**
- Create settings workflow (`/webhook/settings`)
- Display user email, connection status
- Change password option
- Auto-sync toggle
- Disconnect FCView button
- Delete account option

---

## 🔐 Complete Authentication Flow

```
SIGNUP:
1. User visits /webhook/login
2. Fills signup form → POST /webhook/login-auth (action: signup)
3. Password hashed, stored in users table
4. Redirected to /webhook/connect-fcview?user_id=xxx

LOGIN:
1. User visits /webhook/login
2. Enters credentials → POST /webhook/login-auth (action: login)
3. Check FCView connection status
4. Redirect to dashboard (if connected) or connect page (if not)

PASSWORD RESET:
1. User clicks "Forgot Password?"
2. Enters email → Reset link sent
3. Clicks link → Enters new password
4. Password updated → Redirect to login

OAUTH:
1. User clicks "Connect FCView"
2. GET /webhook/oauth/start?user_id=xxx
3. FCView authorization → Callback with code
4. Exchange code for tokens → Fetch flights
5. Store flights in database → Redirect to dashboard

DASHBOARD:
1. GET /webhook/dashboard?user_id=xxx
2. Query user from database
3. Query flights from database (414 flights)
4. Transform data for display
5. Build HTML with flight cards
6. Show pending flights with import buttons
```

---

## 📝 Implementation Roadmap

### ✅ **Phase 1-5: COMPLETE**
- Foundation, API integration, OAuth, User auth, Password reset, Dashboard UI

### 🚧 **Phase 6: LogTen Pro Integration (NEXT)**
- [ ] **CURRENT:** Generate `logten://` URL workflow
- [ ] Format flight data for LogTen Pro
- [ ] Handle timezone conversions
- [ ] Mark flights as imported
- [ ] Test complete sync: FCView → Dashboard → LogTen Pro
- [ ] Update flight status badges after import

### ⏳ **Phase 7: Background Sync & Settings**
- [ ] Daily sync workflow
- [ ] Token refresh handling
- [ ] Settings page
- [ ] Change password
- [ ] Auto-sync toggle
- [ ] Disconnect FCView

### ⏳ **Phase 8: Subscription & Payments**
- [ ] Stripe integration
- [ ] $9.99/month subscription
- [ ] Trial period logic
- [ ] Payment webhook handling

### ⏳ **Phase 9: Polish & Launch**
- [ ] Email notifications
- [ ] Error logging
- [ ] Mobile testing
- [ ] Beta user testing
- [ ] SEO optimization

---

## 💰 Cost Structure

**Current Monthly Costs:**
- n8n Cloud: $20/month
- Supabase: $0 (free tier, <500MB, 414 flights stored)
- Cloudflare: $0 (free tier)
- Domain: $0.60/month
- **Total: $20.60/month**

**Break-even: 3 paying users @ $9.99/month**

---

## 📞 Support & Resources

**GitHub:** https://github.com/flightbridge/flightbridge  
**n8n Cloud:** https://app.n8n.cloud  
**Supabase:** https://supabase.com/dashboard

---

**Last Updated:** October 23, 2025, 9:40 AM PST  
**Current Status:** ✅ Dashboard UI complete with real flight data! 414 flights displaying correctly!  
**Next Focus:** Build LogTen Pro import URL generation workflow.