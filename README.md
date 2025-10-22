# FlightBridge

**Sync Flight Crew View to LogTen Pro with Automatic Timezone Corrections**

🚀 **Status:** OAuth Complete! Flight Data Sync In Progress! 🎉  
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

### Phase 5: Flight Data Sync 🚧 **IN PROGRESS**
- [x] OAuth Callback retrieves flight data from FCView API
- [x] Flight data parsing code node added
- [x] Filters out deadhead flights
- [x] Extracts: flight_number, airports, times, aircraft, runways
- [ ] **CURRENT:** Configure Supabase upsert for mass flight insertion
- [ ] Test complete flow with real user data
- [ ] Background sync workflow (daily at 6am)

---

## 🎯 Current Status: Flight Data Extraction Ready!

### ✅ **Latest Progress (Oct 22, 2025, 2:43 PM):**

#### **OAuth Callback Workflow - Enhanced:**
1. **Webhook** - Receives OAuth callback with code & state (user_id)
2. **Code: Exchange Code for Tokens** - Gets access_token & refresh_token
3. **Supabase: Store Tokens** - Saves tokens to users table
4. **HTTP Request: Get Flights** - Fetches all flights from FCView API
   - Authorization: `Bearer {access_token}` ✅ **WORKING**
   - Returns ~100KB JSON with 60+ flights
5. **Code: Parse Flight Data** ✅ **NEW** - Extracts flight details:
   ```javascript
   - user_id (from state parameter)
   - fcview_flight_id (unique identifier)
   - flight_number
   - departure_airport, arrival_airport
   - departure_time, arrival_time (UTC)
   - flight_date
   - aircraft_type, tail_number
   - departure_runway, arrival_runway
   - Filters: !is_deadhead (skip deadhead flights)
   ```
6. **Supabase: Insert Flights** ⏳ **NEXT** - Batch upsert to flights table

#### **Database Schema Updates:**
```sql
-- flights table structure (COMPLETE)
CREATE TABLE flights (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID REFERENCES users(id) ON DELETE CASCADE,
  fcview_flight_id VARCHAR(255) UNIQUE NOT NULL,
  
  -- Flight details
  flight_number VARCHAR(50),
  departure_airport VARCHAR(10),
  arrival_airport VARCHAR(10),
  departure_time TIMESTAMP,
  arrival_time TIMESTAMP,
  flight_date DATE,
  
  -- Aircraft info (NEW)
  aircraft_type VARCHAR(50),
  tail_number VARCHAR(20),
  departure_runway VARCHAR(10),
  arrival_runway VARCHAR(10),
  duration_minutes INTEGER,
  
  -- LogTen sync tracking
  imported_to_logten BOOLEAN DEFAULT false,
  imported_at TIMESTAMP,
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW()
);

-- Index for fast queries
CREATE INDEX idx_user_flights ON flights(user_id, flight_date DESC);
```

**Note:** ✅ Removed `flight_data` JSONB column to save storage (using parsed fields only)

---

## 🚧 **IMMEDIATE NEXT STEPS:**

### **Task #1: Configure Supabase Upsert (Priority)**
**Goal:** Mass insert flights with automatic updates for existing records

**Steps:**
1. Add Supabase node after "Parse Flight Data" code node
2. **Operation:** Insert
3. **Enable:** On Conflict → Do Update
4. **Conflict Column:** `fcview_flight_id`
5. **Columns to Insert:**
   - user_id
   - fcview_flight_id
   - flight_number
   - departure_airport
   - arrival_airport
   - departure_time
   - arrival_time
   - flight_date
   - aircraft_type
   - tail_number
   - departure_runway
   - arrival_runway

**Why Upsert?**
- Handles duplicate flights automatically
- Updates existing records if FCView data changes
- Processes all 60+ flights in one batch operation

### **Task #2: Test Complete OAuth → Flight Sync Flow**
1. User signs up/logs in
2. Connects FCView account
3. OAuth callback runs
4. Flights automatically sync to database
5. Verify flights appear in Supabase

### **Task #3: Dashboard Real Data Integration**
- Query flights from Supabase by user_id
- Display pending flights (not imported yet)
- Show flight details: route, date, aircraft
- "Import to LogTen Pro" button per flight

---

## 📋 Working URLs

### **Production URLs (ALL WORKING):**
- Landing Page: `https://flightbridge.app/` ✅
- Login/Signup: `https://flightbridge.app/webhook/login` ✅
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

### **users table ✅ COMPLETE**
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

### **n8n Workflows (5 Active):**
1. **FlightBridge Landing Page** - Serves homepage
2. **FlightBridge Login** - Shows login/signup form (GET)
3. **FlightBridge Login Auth** - Processes authentication (POST)
4. **OAuth Start** - Initiates FCView OAuth with user_id
5. **OAuth Callback** - Receives tokens, fetches & parses flights ✅ **ENHANCED**

### **OAuth Flow (Complete & Working):**
```
1. User signs up/logs in → gets user_id
2. Redirected to /webhook/connect-fcview?user_id=XXX
3. Clicks "Connect" → /webhook/oauth/start?user_id=XXX
4. FCView authorization → Callback with code & state=user_id
5. Token exchange → Store in Supabase users table
6. HTTP Request → Fetch all flights from FCView API
7. Parse flight data → Extract fields, filter deadheads
8. [NEXT] Upsert flights → Batch insert to Supabase
9. Redirect to dashboard with real flight data
```

### **Flight Data Extraction Logic:**
```javascript
const responseData = JSON.parse($json.data);
const flights = responseData.flights;
const userId = $('Webhook').first().json.query.state;

return flights
  .filter(flight => !flight.is_deadhead) // Skip deadhead flights
  .map(flight => ({
    json: {
      user_id: userId,
      fcview_flight_id: flight.fcv_flight_id,
      flight_number: flight.flight_number,
      departure_airport: flight.dep_airport,
      arrival_airport: flight.arr_airport,
      departure_time: flight.actual_out_utc || flight.scheduled_out_utc,
      arrival_time: flight.actual_in_utc || flight.scheduled_in_utc,
      flight_date: (flight.scheduled_out_local || '').split(' ')[0],
      aircraft_type: flight.fcv_aircraft_type,
      tail_number: flight.fcv_tail_number,
      departure_runway: flight.dep_runway,
      arrival_runway: flight.arr_runway
    }
  }));
```

---

## 📝 Implementation Roadmap

### ✅ **Phase 1-4: COMPLETE**
- Foundation, API integration, OAuth, User auth

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

## 📊 Technical Achievements

### **FCView API Integration:**
- ✅ OAuth 2.0 flow implemented correctly
- ✅ Token exchange working
- ✅ Authorization header format fixed (`Bearer {token}`)
- ✅ Flights endpoint returning data (~100KB JSON)
- ✅ Response parsing handling 60+ flights
- ✅ Deadhead flight filtering

### **Database Design:**
- ✅ User-flight relationship (foreign key)
- ✅ Unique constraint on fcview_flight_id (prevents duplicates)
- ✅ Index for fast queries by user and date
- ✅ Optimized schema (removed redundant JSONB column)

### **User Experience:**
- ✅ Seamless OAuth flow through custom domain
- ✅ User ID passed through entire flow via state parameter
- ✅ Automatic token storage linked to user account
- ✅ Ready for automatic flight sync on connect

---

## 🐛 Known Issues & Future Improvements

### **Current Issues:**
- None! OAuth and data extraction working smoothly ✅

### **Future Enhancements:**
- [ ] Token refresh logic (when access_token expires)
- [ ] Error handling for API failures
- [ ] Rate limiting for FCView API calls
- [ ] Pagination for users with 100+ flights
- [ ] Background job queue for sync operations

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

**Last Updated:** October 22, 2025, 2:43 PM PST  
**Current Status:** ✅ OAuth complete! Flight data extraction ready! Next: Configure Supabase upsert for mass sync.  
**Next Focus:** Configure batch upsert, test complete flow, display real flights on dashboard.