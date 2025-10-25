# FlightBridge

**Sync Flight Crew View to LogTen Pro with Automatic Timezone Corrections**

🚀 **Status:** Authentication & Sync Workflow Complete!  
🌐 **Live:** https://flightbridge.app/  
📊 **Dashboard:** https://flightbridge.app/webhook/dashboard

---

## 🎯 Current Status (October 25, 2025)

### ✅ **COMPLETED:**

**Foundation & Infrastructure:**
- ✅ Domain & DNS setup (flightbridge.app)
- ✅ Cloudflare Worker proxy configured
- ✅ Landing page live
- ✅ Privacy & Terms pages
- ✅ Supabase PostgreSQL database
- ✅ n8n Cloud workflows ($20/month)

**API Integration:**
- ✅ FCView API approval (Indie tier)
- ✅ Client ID: `f0cf9180d491f06e`
- ✅ OAuth configuration complete
- ✅ Redirect URI: `https://flightbridge.app/webhook/oauth/callback`

**Authentication System (100% Complete):**
- ✅ OAuth Start workflow (`/webhook/oauth/start`)
- ✅ OAuth Callback workflow (`/webhook/oauth/callback`)
- ✅ Login/Signup workflows
- ✅ Password reset workflows
- ✅ Token storage in Supabase (access + refresh tokens)
- ✅ Token refresh logic (1 hour access, 3 months refresh)
- ✅ Password hashing (SHA-256)

**Flight Data Sync:**
- ✅ "FlightBridge Sync FCView" workflow functional
- ✅ Fetches flights from FCView API (last 3 months)
- ✅ Filters deadhead flights (`is_deadhead === 1`)
- ✅ Filters special flight types (BDM, AVL, DTY)
- ✅ Filters incomplete past flights (no `actual_out_local`)
- ✅ INSERT/UPDATE/SKIP change detection logic
- ✅ Compares: `actual_out_local`, `actual_in_local`, `tail_number`, `flight_number`
- ✅ Stores flights in Supabase

**Dashboard:**
- ✅ Real flight data display from Supabase
- ✅ Flight cards with route, date, time, aircraft
- ✅ Status badges (Pending/Imported)
- ✅ Filter buttons (All/Pending/Imported)
- ✅ Expandable "More Details" section
- ✅ Single-flight "Import to LogTen Pro" button (working!)
- ✅ "Sync Now" button
- ✅ Mobile-responsive design

### 🔄 **IN TESTING:**
- ⏳ INSERT/UPDATE/SKIP logic (needs new flights to test detection)
- ⏳ Sync button comprehensive testing

### 📋 **TODO - High Priority:**

**1. Pilot Flying Field (Required Before Import)**
- Add dropdown to each flight card: Yes/No/[blank]
- User must select before importing
- Validation message: "Please select Pilot Flying (Yes/No) before importing"
- Block import until selected
- Store in `pilot_flying` VARCHAR(1) field ('Y' or 'N')

**2. Dashboard Crew Information**
- Add PIC (Captain) to expandable "More Details" section
- Add SIC (First Officer) to expandable "More Details" section
- Currently stored in database, not displayed

**3. Date Filter Improvements**
- Replace dropdown with calendar date picker
- Add date range filter (From/To dates)
- Apply to flight list display

**4. Mass Import Feature**
- "Import All Filtered" button
- Apply current date range filter
- Batch import all visible flights
- Progress indicator during import
- Validate pilot_flying for all flights before proceeding

### 📋 **TODO - Future:**
- Timezone correction logic verification
- Auto-sync scheduling (daily 6am)
- Settings page (password change, auto-sync toggle, disconnect FCView)
- Stripe payment integration ($9.99/month)
- Email notifications
- Beta user testing

---

## 📊 Working Features

### **Public Pages:**
- Landing Page: `https://flightbridge.app/` ✅
- Login/Signup: `https://flightbridge.app/webhook/login` ✅
- Terms: `https://flightbridge.app/webhook/terms` ✅
- Privacy: `https://flightbridge.app/webhook/privacy` ✅

### **Authentication:**
- Forgot Password: `https://flightbridge.app/webhook/forgot-password` ✅
- Reset Password: `https://flightbridge.app/webhook/reset-password?token=xxx` ✅
- Login Handler: `POST https://flightbridge.app/webhook/login-auth` ✅

### **User Dashboard:**
- Dashboard: `https://flightbridge.app/webhook/dashboard?user_id=xxx` ✅
- Connect FCView: `https://flightbridge.app/webhook/connect-fcview?user_id=xxx` ✅
- Sync Flights: `POST https://flightbridge.app/webhook/sync-flights?user_id=xxx` ✅

### **OAuth Flow:**
- OAuth Start: `https://flightbridge.app/webhook/oauth/start?user_id=xxx` ✅
- OAuth Callback: `https://flightbridge.app/webhook/oauth/callback` ✅

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
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID REFERENCES users(id) ON DELETE CASCADE,
  fcview_flight_id VARCHAR(255) UNIQUE NOT NULL,
  
  -- Flight Details
  flight_number VARCHAR(50),
  departure_airport VARCHAR(10),
  arrival_airport VARCHAR(10),
  dep_icao_code VARCHAR(10),
  arr_icao_code VARCHAR(10),
  
  -- Times
  scheduled_out_local TIMESTAMP,
  scheduled_in_local TIMESTAMP,
  actual_out_local TIMESTAMP,
  actual_in_local TIMESTAMP,
  actual_out_local_utc TIMESTAMP,
  actual_in_local_utc TIMESTAMP,
  
  -- Aircraft
  tail_number VARCHAR(50),
  aircraft_type VARCHAR(50),
  
  -- Crew
  PIC VARCHAR(255),              -- Captain name
  SIC VARCHAR(255),              -- First Officer name
  pilot_flying VARCHAR(1),       -- 'Y' or 'N' (NULL = not set, requires user input)
  
  -- Duration
  Block VARCHAR(10),
  flight_duration VARCHAR(10),
  
  -- Trip Info
  fcview_trip_number VARCHAR(255),
  
  -- Import Status
  imported_to_logten BOOLEAN DEFAULT false,
  imported_at TIMESTAMP,
  created_at TIMESTAMP DEFAULT NOW()
);

CREATE INDEX idx_user_flights ON flights(user_id, actual_out_local DESC);
CREATE UNIQUE INDEX idx_fcview_flight_id ON flights(fcview_flight_id);
```

---

## 🔐 Authentication Flow

```
SIGNUP:
1. User visits /webhook/login
2. Fills signup form → POST /webhook/login-auth (action: signup)
3. Password hashed (SHA-256), stored in users table
4. Redirected to /webhook/connect-fcview?user_id=xxx

LOGIN:
1. User visits /webhook/login
2. Enters credentials → POST /webhook/login-auth (action: login)
3. Password verified, FCView connection status checked
4. Redirect to dashboard (if connected) or connect page (if not)

PASSWORD RESET:
1. User clicks "Forgot Password?"
2. Enters email → POST /webhook/forgot-password-handler
3. Reset token generated, email sent
4. User clicks link → GET /webhook/reset-password?token=xxx
5. Enters new password → POST /webhook/password-reset-handler
6. Password updated → Redirect to login

OAUTH (FCView Connection):
1. User clicks "Connect FCView" → GET /webhook/oauth/start?user_id=xxx
2. Redirected to FCView authorization page
3. User enters 8-character passkey from FCView mobile app
4. FCView validates → Callback with authorization code
5. n8n exchanges code for access_token + refresh_token
6. Tokens stored in Supabase users table
7. Initial flight sync triggered
8. Redirected to dashboard

SYNC WORKFLOW:
1. Triggered via "Sync Now" button or scheduled (future)
2. POST /webhook/sync-flights?user_id=xxx
3. Check token expiration → Refresh if needed
4. Fetch flights from FCView API (last 3 months)
5. Fetch existing flights from Supabase
6. Compare and categorize:
   - INSERT: New flights not in database
   - UPDATE: Existing flights with changed data
   - SKIP: Existing flights with no changes
7. Execute database operations
8. Return sync summary
```

---

## 🔧 n8n Workflows

### **Active Workflows:**

1. **FlightBridge Landing Page** - Serves homepage
2. **FlightBridge Login** - Login/signup form
3. **FlightBridge Login Auth** - Handles authentication
4. **FlightBridge OAuth - Step 1 Start** - Initiates FCView OAuth
5. **FlightBridge OAuth - Step 2 Callback** - Handles OAuth callback
6. **FlightBridge Dashboard** - Displays user flights
7. **FlightBridge Connect FCView** - Connect FCView page
8. **FlightBridge Sync FCView** - Background sync workflow
9. **FlightBridge Generate LogTen URL** - Creates LogTen Pro import URLs
10. **FlightBridge Forgot Password** - Password reset form
11. **FlightBridge Forgot Password Handler** - Sends reset email
12. **FlightBridge Password Reset** - Reset password form
13. **FlightBridge Password Reset Handler** - Updates password
14. **FlightBridge Terms of Service** - Terms page
15. **FlightBridge Privacy Policy** - Privacy page

### **Sync Workflow Details:**

**Filters Applied:**
- Deadhead flights (`is_deadhead === 1`)
- Special types: BDM, AVL, DTY
- Incomplete past flights (no `actual_out_local` if `scheduled_out_local` passed)
- Date range: Last 3 months from `actual_out_local`

**Change Detection:**
Compares these fields between FCView and Supabase:
- `actual_out_local`
- `actual_in_local`
- `tail_number`
- `flight_number`

**Actions:**
- INSERT: Flight exists in FCView but not in Supabase
- UPDATE: Flight exists in both but data changed
- SKIP: Flight exists in both with identical data

---

## 🎨 Dashboard Features

### **Current Features:**
- ✅ Real flight data from Supabase
- ✅ Flight list with route (ATL → MLU)
- ✅ Status badges (Pending in yellow, Imported in green)
- ✅ Filter buttons (All/Pending/Imported)
- ✅ Flight details: Trip number, block time, scheduled/actual times, duration
- ✅ Expandable "More Details" with UTC times and tail number
- ✅ "Import to LogTen Pro" button (opens LogTen Pro app)
- ✅ "Sync Now" button (triggers sync workflow)
- ✅ Connected status indicator (green box)
- ✅ Last synced timestamp
- ✅ Mobile-responsive design

### **Pending UI Improvements:**

**1. Pilot Flying Dropdown:**
```html
<div class="pilot-flying-selector">
  <label>Were you Pilot Flying?</label>
  <select id="pilot-flying-{flight_id}" required>
    <option value="">-- Select --</option>
    <option value="Y">Yes</option>
    <option value="N">No</option>
  </select>
</div>
```

**2. PIC/SIC in More Details:**
```html
<div class="detail-row">
  <span class="label">Captain (PIC):</span>
  <span class="value">{{PIC}}</span>
</div>
<div class="detail-row">
  <span class="label">First Officer (SIC):</span>
  <span class="value">{{SIC}}</span>
</div>
```

**3. Calendar Date Picker:**
```html
<div class="date-filter">
  <label>From:</label>
  <input type="date" id="date-from" onchange="filterFlights()">
  
  <label>To:</label>
  <input type="date" id="date-to" onchange="filterFlights()">
</div>
```

**4. Mass Import Button:**
```html
<button class="mass-import-btn" onclick="massImport()">
  📦 Import All Filtered ({count} flights)
</button>
```

---

## 🧪 Testing Status

### **What's Working:**
- ✅ OAuth complete flow (authorization → tokens → storage)
- ✅ Login/signup with password hashing
- ✅ Password reset with email tokens
- ✅ Flight sync from FCView API
- ✅ Database storage and retrieval
- ✅ Dashboard display with real data
- ✅ Single-flight import to LogTen Pro

### **Needs Testing:**
- ⏳ INSERT/UPDATE/SKIP detection (waiting for new flights)
- ⏳ Sync button with multiple changes
- ⏳ Token refresh after 1 hour expiration
- ⏳ Mass import feature (not yet built)

### **Known Issues:**
- None currently

---

## 📝 Implementation Roadmap

### **Phase 1-5: COMPLETE** ✅
- Foundation setup
- API integration
- OAuth flow
- User authentication
- Password reset
- Dashboard with real data
- Flight sync workflow

### **Phase 6: Dashboard Enhancements** 🔄 (IN PROGRESS)
- [ ] Pilot flying dropdown validation
- [ ] PIC/SIC display in More Details
- [ ] Calendar date picker
- [ ] Date range filter
- [ ] Mass import feature
- [ ] Test INSERT/UPDATE/SKIP logic

### **Phase 7: Settings & Polish** ⏳
- [ ] Settings page
- [ ] Change password
- [ ] Auto-sync toggle
- [ ] Disconnect FCView option
- [ ] Delete account option
- [ ] Email notifications

### **Phase 8: Subscription** ⏳
- [ ] Stripe integration
- [ ] $9.99/month subscription
- [ ] Trial period management
- [ ] Payment webhook handling
- [ ] Subscription status in dashboard

### **Phase 9: Launch** ⏳
- [ ] Auto-sync scheduling (daily 6am)
- [ ] Performance optimization
- [ ] Error logging and monitoring
- [ ] Mobile app testing
- [ ] Beta user testing
- [ ] SEO optimization
- [ ] Marketing site updates

---

## 💰 Cost Structure

**Current Monthly Costs:**
- n8n Cloud: $20/month
- Supabase: $0 (free tier)
- Cloudflare: $0 (free tier)
- Domain (flightbridge.app): $0.60/month
- **Total: $20.60/month**

**Revenue Target:**
- Subscription: $9.99/month per user
- **Break-even: 3 paying users**
- **Profitable: 4+ users**

**Projected Costs at Scale:**
- n8n Cloud stays at $20/month (5,000 executions included)
- Supabase free tier: Up to 500MB (hundreds of users)
- Cloudflare free tier: Unlimited bandwidth

---

## 🚀 Quick Start

**For deployment instructions, see:** [QUICKSTART.md](./QUICKSTART.md)

**Key Setup Steps:**
1. Create n8n Cloud account
2. Set up Supabase database
3. Configure Cloudflare Worker
4. Import n8n workflows
5. Configure environment variables
6. Test authentication flow
7. Test sync workflow

---

## 🔗 API Resources

**FCView API:**
- Documentation: https://help.flightcrewview.com/support/solutions/articles/16000189645
- Client Portal: https://flightcrewview2.com/logbook/logbookapiclientportal
- API Docs PDF: See `Flight Crew View - Client Settings.pdf`

**LogTen Pro:**
- URL Scheme Documentation: See `logten pro api doc`
- Import Format: `logten://import?data={base64_json}`

**n8n:**
- Cloud Dashboard: https://app.n8n.cloud
- Documentation: https://docs.n8n.io

**Supabase:**
- Dashboard: https://supabase.com/dashboard
- PostgreSQL connection string in n8n credentials

---

## 📞 Support

**GitHub Issues:** https://github.com/flightbridge/flightbridge/issues  
**Email:** support@flightbridge.app  
**Documentation:** See QUICKSTART.md and DEPLOYMENT.md

---

## 📈 Recent Updates

**October 25, 2025:**
- ✅ Authentication system 100% complete
- ✅ Sync workflow functional with proper filtering
- ✅ Dashboard displaying real flight data
- ✅ LogTen Pro single-flight import working
- 📋 Added TODO list for dashboard improvements
- 📋 Documented pilot_flying field requirement
- 📋 Planned mass import feature

**October 23, 2025:**
- ✅ Dashboard UI complete with 414 flights
- ✅ Fixed Supabase query bugs
- ✅ Side-by-side time display
- ✅ Expandable details section

**October 21, 2025:**
- ✅ OAuth flow fully working
- ✅ Token storage in Supabase
- ✅ Password reset system complete

---

**Last Updated:** October 25, 2025  
**Current Focus:** Dashboard enhancements (pilot flying field, crew info, date filters, mass import)  
**Next Milestone:** Complete Phase 6 dashboard improvements