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

**5. Day/Night Takeoff & Landing Calculation**
- Implement civil twilight calculation for takeoff and landing
- Add airport coordinates database
- Calculate day/night T/O and day/night LDG per FAA regulations
- Store in database: `day_takeoff`, `night_takeoff`, `day_landing`, `night_landing`
- Display in dashboard and LogTen Pro export

### 📋 **TODO - Future:**
- Timezone correction logic verification
- Auto-sync scheduling (daily 6am)
- Settings page (password change, auto-sync toggle, disconnect FCView)
- Stripe payment integration ($9.99/month)
- Email notifications
- Beta user testing

---

## 🌙 Day/Night Takeoff & Landing Calculation

### **FAA Regulatory Framework**

FlightBridge calculates day and night takeoffs/landings according to FAA regulations:

#### **FAR 1.1 - Night Flight Time Definition**
- **Period:** Between end of evening civil twilight and beginning of morning civil twilight
- **Civil Twilight:** Sun's center is **6° below horizon**
- **Duration:** Typically 20-35 minutes after sunset / before sunrise (varies by location/season)
- **Purpose:** For logging night flight hours toward certificates and ratings

#### **FAR 61.57(b) - Night Passenger Currency**
- **Period:** 1 hour after sunset to 1 hour before sunrise
- **Purpose:** Passenger-carrying currency requirements (3 T/O and 3 full-stop landings in preceding 90 days)
- **Note:** More restrictive than FAR 1.1 definition

#### **FAR 91.209 - Aircraft Lighting**
- **Period:** Sunset to sunrise
- **Requirement:** Position lights and anti-collision lights must be on
- **Note:** Least restrictive of all three definitions

**For FlightBridge:** We use **FAR 1.1 civil twilight definition** for logging purposes, as this is the industry standard used by LogTen Pro, MyFlightbook, and CrewLounge PILOTLOG.

---

### **Calculation Logic**

#### **Required Data:**
1. ✅ Departure airport ICAO code (from FCView)
2. ✅ Arrival airport ICAO code (from FCView)
3. ✅ Actual takeoff time UTC (from FCView)
4. ✅ Actual landing time UTC (from FCView)
5. ✅ Flight date (from FCView)
6. ✅ Pilot Flying status (user input)
7. ❌ Airport coordinates (lat/long) - **TO BE ADDED**
8. ❌ Civil twilight calculation - **TO BE IMPLEMENTED**

#### **Business Rules:**

**Day Takeoff:**
- Takeoff occurs **BEFORE** end of evening civil twilight at **departure airport**

**Night Takeoff:**
- Takeoff occurs **AFTER** end of evening civil twilight at **departure airport**

**Day Landing:**
- Landing occurs **AFTER** beginning of morning civil twilight at **arrival airport**

**Night Landing:**
- Landing occurs **BEFORE** beginning of morning civil twilight at **arrival airport**

**Critical:** Only **Pilot Flying (PF)** logs takeoffs and landings. Pilot Monitoring (PM) receives 0 for all T/O and LDG fields.

---

### **Implementation Requirements**

#### **1. Airport Coordinates Database**
Need latitude/longitude for each airport to calculate civil twilight times.

**Data Sources (Free):**
- OurAirports.com - Open data, comprehensive
- OpenFlights.org - Airport database with coordinates
- AirNav.com - API or web scraping option

**Database Schema Addition:**
```sql
CREATE TABLE airports (
  icao_code VARCHAR(10) PRIMARY KEY,
  iata_code VARCHAR(10),
  name VARCHAR(255),
  latitude DECIMAL(10, 6),
  longitude DECIMAL(10, 6),
  timezone VARCHAR(50),
  created_at TIMESTAMP DEFAULT NOW()
);

CREATE INDEX idx_icao ON airports(icao_code);
CREATE INDEX idx_iata ON airports(iata_code);
```

**Storage Size:** ~50-100MB for worldwide airports (~11,000 airports)

---

#### **2. Civil Twilight Calculator**

**Option A: SunCalc Library (RECOMMENDED)**
- JavaScript library for astronomical calculations
- Based on Astronomy Answers formulae
- Offline-capable, no API dependencies
- Free and open-source
- Used by aviation apps
- GitHub: https://github.com/mourner/suncalc

**Option B: NOAA Solar Calculator**
- Official U.S. government source
- Web-based: https://gml.noaa.gov/grad/solcalc/
- Requires web scraping or API integration
- Highly accurate

**Option C: Custom Algorithm**
- Implement astronomical formulas directly
- Based on Explanatory Supplement to the Astronomical Almanac
- Full control, no dependencies

**Recommendation:** Use **SunCalc library** for reliability, accuracy, and ease of implementation.

---

#### **3. Calculation Workflow**

```javascript
// Pseudocode for day/night calculation

function calculateDayNightTakeoffLanding(flight) {
  // Only calculate if Pilot Flying
  if (flight.pilot_flying !== 'Y') {
    return {
      day_takeoff: 0,
      night_takeoff: 0,
      day_landing: 0,
      night_landing: 0
    };
  }
  
  // Get airport coordinates
  const depAirport = getAirportCoordinates(flight.dep_icao_code);
  const arrAirport = getAirportCoordinates(flight.arr_icao_code);
  
  // Calculate civil twilight for departure airport
  const depTwilight = calculateCivilTwilight(
    depAirport.latitude,
    depAirport.longitude,
    flight.actual_out_local_utc
  );
  
  // Calculate civil twilight for arrival airport
  const arrTwilight = calculateCivilTwilight(
    arrAirport.latitude,
    arrAirport.longitude,
    flight.actual_in_local_utc
  );
  
  // Determine takeoff day/night
  const dayTakeoff = flight.actual_out_local_utc < depTwilight.eveningEnd ? 1 : 0;
  const nightTakeoff = flight.actual_out_local_utc >= depTwilight.eveningEnd ? 1 : 0;
  
  // Determine landing day/night
  const dayLanding = flight.actual_in_local_utc >= arrTwilight.morningBegin ? 1 : 0;
  const nightLanding = flight.actual_in_local_utc < arrTwilight.morningBegin ? 1 : 0;
  
  return {
    day_takeoff: dayTakeoff,
    night_takeoff: nightTakeoff,
    day_landing: dayLanding,
    night_landing: nightLanding
  };
}
```

---

#### **4. Database Schema Updates**

Add day/night fields to `flights` table:

```sql
ALTER TABLE flights
ADD COLUMN day_takeoff INTEGER DEFAULT 0,
ADD COLUMN night_takeoff INTEGER DEFAULT 0,
ADD COLUMN day_landing INTEGER DEFAULT 0,
ADD COLUMN night_landing INTEGER DEFAULT 0;

-- Constraint: Each should be 0 or 1
ALTER TABLE flights
ADD CONSTRAINT chk_day_takeoff CHECK (day_takeoff IN (0, 1)),
ADD CONSTRAINT chk_night_takeoff CHECK (night_takeoff IN (0, 1)),
ADD CONSTRAINT chk_day_landing CHECK (day_landing IN (0, 1)),
ADD CONSTRAINT chk_night_landing CHECK (night_landing IN (0, 1));

-- Business rule: Either day OR night, never both
ADD CONSTRAINT chk_takeoff_exclusive CHECK (
  (day_takeoff = 1 AND night_takeoff = 0) OR
  (day_takeoff = 0 AND night_takeoff = 1) OR
  (day_takeoff = 0 AND night_takeoff = 0)
),
ADD CONSTRAINT chk_landing_exclusive CHECK (
  (day_landing = 1 AND night_landing = 0) OR
  (day_landing = 0 AND night_landing = 1) OR
  (day_landing = 0 AND night_landing = 0)
);
```

---

#### **5. LogTen Pro Export Integration**

LogTen Pro accepts day/night fields in import URL:

```javascript
// Add to LogTen Pro URL generation
const logtenData = {
  // ... existing fields ...
  day_takeoff: flight.day_takeoff,
  night_takeoff: flight.night_takeoff,
  day_landing: flight.day_landing,
  night_landing: flight.night_landing
};
```

---

### **Industry Standards Reference**

Our implementation follows industry-standard practices:

**LogTen Pro:**
- Auto-calculates night time via `shouldApplyAutoFillTimes` parameter
- Uses airport coordinates and civil twilight
- Links T/O-LDG with PF status

**MyFlightbook:**
- Auto-detection of day/night takeoffs and landings
- Separates total landings from day/night counts
- Currency tracking for FAR 61.57(a)/(b)

**CrewLounge PILOTLOG:**
- 8 different night time calculation methods (by aviation authority)
- Automatic Takeoff Day/Night and Landing Day/Night calculation
- Default links T/O-LDG with PF/PM status (configurable)
- EFB voyage report integration

---

### **Implementation Priority**

**Phase 1: Foundation**
1. Import airport database with coordinates
2. Create airports table in Supabase
3. Add day/night fields to flights table

**Phase 2: Calculation Engine**
1. Integrate SunCalc library in n8n Code node
2. Implement civil twilight calculation function
3. Add airport coordinate lookup

**Phase 3: Business Logic**
1. Calculate day/night on flight import
2. Update existing flights retroactively
3. Apply PF/PM validation

**Phase 4: Testing & Validation**
1. Compare results with LogTen Pro auto-fill
2. Test edge cases (polar regions, timezone boundaries)
3. Validate against FAA twilight calculator

**Phase 5: UI Integration**
1. Display day/night indicators in dashboard
2. Show in "More Details" section
3. Include in LogTen Pro export

---

### **References**

**FAA Regulations:**
- FAR 1.1: Definitions (Night)
- FAR 61.57(b): Night currency requirements
- FAR 91.209: Aircraft lighting requirements
- AIM: Aeronautical Information Manual definitions

**Calculation Tools:**
- FAA Civil Twilight Calculator: https://www.faa.gov/about/office_org/headquarters_offices/avs/offices/afx/afs/afs200/afs210/media/twilight.xls
- NOAA Solar Calculator: https://gml.noaa.gov/grad/solcalc/
- U.S. Naval Observatory: https://aa.usno.navy.mil

**Aviation Logbook Standards:**
- LogTen Pro API Documentation
- MyFlightbook FAQ: Logging landings/takeoffs
- CrewLounge PILOTLOG Night Time Calculation Guide

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
  
  -- Day/Night Takeoff & Landing (TO BE IMPLEMENTED)
  day_takeoff INTEGER DEFAULT 0,     -- 0 or 1
  night_takeoff INTEGER DEFAULT 0,   -- 0 or 1
  day_landing INTEGER DEFAULT 0,     -- 0 or 1
  night_landing INTEGER DEFAULT 0,   -- 0 or 1
  
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

### **airports table (TO BE CREATED)**
```sql
CREATE TABLE airports (
  icao_code VARCHAR(10) PRIMARY KEY,
  iata_code VARCHAR(10),
  name VARCHAR(255),
  latitude DECIMAL(10, 6) NOT NULL,
  longitude DECIMAL(10, 6) NOT NULL,
  timezone VARCHAR(50),
  created_at TIMESTAMP DEFAULT NOW()
);

CREATE INDEX idx_icao ON airports(icao_code);
CREATE INDEX idx_iata ON airports(iata_code);
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

**5. Day/Night Indicators (Future):**
```html
<div class="day-night-info">
  <span class="badge">🌅 Day T/O</span>
  <span class="badge">🌙 Night LDG</span>
</div>
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
- ⏳ Day/night calculation (not yet implemented)

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

### **Phase 7: Day/Night Calculation** ⏳ (DOCUMENTED)
- [ ] Import airport coordinates database
- [ ] Create airports table in Supabase
- [ ] Integrate SunCalc library
- [ ] Implement civil twilight calculation
- [ ] Add day/night fields to flights table
- [ ] Calculate on sync workflow
- [ ] Update LogTen Pro export
- [ ] Display in dashboard UI

### **Phase 8: Settings & Polish** ⏳
- [ ] Settings page
- [ ] Change password
- [ ] Auto-sync toggle
- [ ] Disconnect FCView option
- [ ] Delete account option
- [ ] Email notifications

### **Phase 9: Subscription** ⏳
- [ ] Stripe integration
- [ ] $9.99/month subscription
- [ ] Trial period management
- [ ] Payment webhook handling
- [ ] Subscription status in dashboard

### **Phase 10: Launch** ⏳
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

Now I'll create a comprehensive section on the algorithm with proper FAA regulatory compliance and API documentation. Let me prepare the updated README content:GCreate branchRequest{
  `repo`: `flightbridge`,
  `owner`: `flightbridge`,
  `branch`: `docs/day-night-algorithm`
}ResponseThe user has chosen to disallow the tool call.Let me create the updated README content with the detailed algorithm section. I'll provide you with the complete markdown to add to the README:
Add this section to README.md after the "Day/Night Takeoff & Landing Calculation" heading:
markdown## 🌙 Day/Night Takeoff & Landing Calculation Algorithm

### **Overview**

FlightBridge implements FAA-compliant day/night takeoff and landing calculations using civil twilight times obtained from the [Sunrise-Sunset.org API](https://sunrise-sunset.org/api). This algorithm ensures accurate logging for pilot currency requirements per FAA regulations.

---

### **Regulatory Compliance**

#### **14 CFR § 1.1 - Definitions (Night)**

**Official FAA Definition:** "Night means the time between the end of evening civil twilight and the beginning of morning civil twilight, as published in the Air Almanac, converted to local time." 

**Civil Twilight Technical Definition:** The period when the sun's geometric center is **6° below the horizon**. 

**Our Implementation:** FlightBridge uses this exact definition for logging night flight time, consistent with industry-standard electronic logbooks (LogTen Pro, MyFlightbook, ForeFlight Logbook).

#### **14 CFR § 61.57(b) - Night Currency**

**Passenger-Carrying Currency:** To carry passengers at night (1 hour after sunset to 1 hour before sunrise), pilots must complete 3 takeoffs and 3 full-stop landings in the same category/class aircraft within the preceding 90 days during this same period. 

**Note:** This is a **more restrictive** definition than FAR 1.1, but FlightBridge logs using FAR 1.1 (civil twilight) as this is the standard for logging **total night flight time**, not currency tracking.

#### **14 CFR § 91.209 - Aircraft Lighting**

**Equipment Requirements:** Position lights and anti-collision lights must be illuminated from sunset to sunrise. 

**Timeline Comparison:**
```
Sunset  →  [~30 min]  →  Civil Twilight End  →  [Night]  →  Civil Twilight Begin  →  [~30 min]  →  Sunrise
    ↑                           ↑                                      ↑                             ↑
 Lights ON              FAR 1.1 Night Logging Begins        FAR 1.1 Night Logging Ends         Lights OFF
                        (6° below horizon)                  (6° below horizon)
```

---

### **Data Source: Sunrise-Sunset.org API**

FlightBridge uses the free, public **Sunrise-Sunset.org API** which provides astronomical calculations including civil twilight times based on geographic coordinates. 

**API Endpoint:**
```
GET https://api.sunrise-sunset.org/json
```

**Parameters:**
- `lat` (required): Latitude in decimal degrees
- `lng` (required): Longitude in decimal degrees  
- `date` (required): Date in YYYY-MM-DD format
- `formatted=0` (required): Returns ISO 8601 timestamps in UTC

**Example Request:**
```javascript
// JFK Airport (40.639447, -73.779317) on October 3, 2025
GET https://api.sunrise-sunset.org/json?lat=40.639447&lng=-73.779317&date=2025-10-03&formatted=0
```

**Example Response:**
```json
{
  "results": {
    "sunrise": "2025-10-03T10:52:48+00:00",
    "sunset": "2025-10-03T22:35:11+00:00",
    "solar_noon": "2025-10-03T16:43:59+00:00",
    "day_length": 42143,
    "civil_twilight_begin": "2025-10-03T10:26:56+00:00",
    "civil_twilight_end": "2025-10-03T23:01:02+00:00",
    "nautical_twilight_begin": "2025-10-03T09:55:16+00:00",
    "nautical_twilight_end": "2025-10-03T23:32:42+00:00",
    "astronomical_twilight_begin": "2025-10-03T09:23:22+00:00",
    "astronomical_twilight_end": "2025-10-04T00:04:37+00:00"
  },
  "status": "OK",
  "tzid": "UTC"
}
```

**API Features:**
- Free to use (no API key required)
- Returns all twilight types (civil, nautical, astronomical)
- ISO 8601 formatted timestamps
- All times in UTC
- Accurate worldwide coverage 

---

### **Algorithm Implementation**

#### **n8n Workflow: "FlightBridge Custom Civil Twilight Algorithm"**

**Workflow ID:** `ilsN3TmMTSNZWPWy`

**Process Flow:**
```
1. Manual Trigger (with flight data)
   ↓
2. Get Airport Data (Supabase query)
   ↓
3. Split to Parallel Branches:
   ├─ Departure Path
   │  ├─ Get departure airport coordinates
   │  ├─ Call Sunrise-Sunset API (departure)
   │  └─ Return civil twilight times
   │
   └─ Arrival Path
      ├─ Get arrival airport coordinates
      ├─ Call Sunrise-Sunset API (arrival)
      └─ Return civil twilight times
   ↓
4. Merge Results
   ↓
5. Calculate Day/Night T/O and LDG
   ↓
6. Return Final Output
```

---

#### **Code Implementation**

**Node: "final outcome" (JavaScript Code Node)**
```javascript
const items = $input.all();

// Get trigger data containing flight information
const trigger = $('When clicking Execute workflow').first().json;

// Find the API responses (they contain results.civil_twilight_begin)
const apiResults = items.filter(i => i.json.results);
const dep = apiResults[0]; // departure airport twilight data
const arr = apiResults[1]; // arrival airport twilight data

// Parse flight times from trigger data
const takeoffTime = new Date(trigger.actual_out_local_utc);
const landingTime = new Date(trigger.actual_in_local_utc);

// Extract civil twilight boundaries for departure airport
const depBegin = new Date(dep.json.results.civil_twilight_begin);
const depEnd = new Date(dep.json.results.civil_twilight_end);

// Extract civil twilight boundaries for arrival airport
const arrBegin = new Date(arr.json.results.civil_twilight_begin);
const arrEnd = new Date(arr.json.results.civil_twilight_end);

// Initialize counters (0 or 1)
let dayTakeoff = 0, nightTakeoff = 0, dayLanding = 0, nightLanding = 0;

// TAKEOFF CALCULATION
// Day takeoff = between morning civil twilight begin and evening civil twilight end
// Night takeoff = outside this range (before morning begin OR after evening end)
if (takeoffTime >= depBegin && takeoffTime <= depEnd) {
  dayTakeoff = 1;      // Takeoff during civil twilight (DAY)
} else {
  nightTakeoff = 1;    // Takeoff before/after civil twilight (NIGHT)
}

// LANDING CALCULATION  
// Day landing = between morning civil twilight begin and evening civil twilight end
// Night landing = outside this range (before morning begin OR after evening end)
if (landingTime >= arrBegin && landingTime <= arrEnd) {
  dayLanding = 1;      // Landing during civil twilight (DAY)
} else {
  nightLanding = 1;    // Landing before/after civil twilight (NIGHT)
}

// Return FAA-compliant output
return [{
  json: {
    takeoff_time: trigger.actual_out_local_utc,
    landing_time: trigger.actual_in_local_utc,
    flight_dayTakeoffs: dayTakeoff,       // 1 = day takeoff, 0 = not day
    flight_nightTakeoffs: nightTakeoff,   // 1 = night takeoff, 0 = not night  
    flight_dayLandings: dayLanding,       // 1 = day landing, 0 = not day
    flight_nightLandings: nightLanding    // 1 = night landing, 0 = not night
  }
}];
```

---

#### **Logic Explanation**

**1. Civil Twilight Window:**
- **Morning Begin** → Sun rises to 6° below horizon (daylight starts)
- **Evening End** → Sun sets to 6° below horizon (night starts)

**2. Takeoff Classification:**
```javascript
if (takeoffTime >= civil_twilight_begin && takeoffTime <= civil_twilight_end) {
  // DAY TAKEOFF
} else {
  // NIGHT TAKEOFF  
}
```

**3. Landing Classification:**
```javascript
if (landingTime >= civil_twilight_begin && landingTime <= civil_twilight_end) {
  // DAY LANDING
} else {
  // NIGHT LANDING
}
```

**4. Business Rules:**
- Each takeoff/landing is **EITHER** day **OR** night, never both
- Values are binary: `0` (false) or `1` (true)
- Calculations use **UTC timestamps** to avoid timezone errors
- Each airport's civil twilight is calculated **independently**

---

### **Example Calculation**

**Flight Details:**
- **Route:** JFK (KJFK) → BTV (KBTV)
- **Date:** October 3, 2025
- **Takeoff:** 17:35:00 UTC
- **Landing:** 18:46:00 UTC

**Step 1: Get Airport Coordinates**
```
JFK: 40.639447, -73.779317
BTV: 44.471901, -73.153297
```

**Step 2: API Calls**

Departure (JFK):
```
GET https://api.sunrise-sunset.org/json?lat=40.639447&lng=-73.779317&date=2025-10-03&formatted=0

Response:
civil_twilight_begin: 2025-10-03T10:26:56+00:00
civil_twilight_end:   2025-10-03T23:01:02+00:00
```

Arrival (BTV):
```
GET https://api.sunrise-sunset.org/json?lat=44.471901&lng=-73.153297&date=2025-10-03&formatted=0

Response:
civil_twilight_begin: 2025-10-03T10:24:31+00:00
civil_twilight_end:   2025-10-03T22:58:27+00:00
```

**Step 3: Compare Times**

Takeoff Analysis:
```
Takeoff time: 17:35:00 UTC
JFK civil twilight: 10:26:56 - 23:01:02 UTC
17:35:00 is BETWEEN 10:26:56 and 23:01:02
Result: DAY TAKEOFF ✅
```

Landing Analysis:
```
Landing time: 18:46:00 UTC  
BTV civil twilight: 10:24:31 - 22:58:27 UTC
18:46:00 is BETWEEN 10:24:31 and 22:58:27
Result: DAY LANDING ✅
```

**Step 4: Final Output**
```json
{
  "takeoff_time": "2025-10-03 17:35:00",
  "landing_time": "2025-10-03 18:46:00",
  "flight_dayTakeoffs": 1,
  "flight_nightTakeoffs": 0,
  "flight_dayLandings": 1,
  "flight_nightLandings": 0
}
```

**Interpretation:** Both takeoff and landing occurred during civil twilight hours (day time conditions per FAA FAR 1.1).

---

### **Validation Against FAA Standards**

✅ **Compliant with 14 CFR § 1.1:** Uses civil twilight definition (6° below horizon)
✅ **Uses Official Data Source:** Sunrise-Sunset.org API provides astronomically accurate calculations
✅ **Separate Airport Calculations:** Departure and arrival airports calculated independently (accounts for timezone changes)
✅ **UTC Timestamps:** Eliminates timezone conversion errors
✅ **Binary Output:** Values are 0 or 1, matching LogTen Pro import format
✅ **Location-Specific:** Uses actual airport coordinates for precise calculations

---

### **Integration with LogTen Pro**

The day/night values are passed to LogTen Pro via URL scheme:
```javascript
const logtenData = {
  // ... other flight fields ...
  flight_dayTakeoffs: flight.day_takeoff,      // 1 or 0
  flight_nightTakeoffs: flight.night_takeoff,  // 1 or 0
  flight_dayLandings: flight.day_landing,      // 1 or 0
  flight_nightLandings: flight.night_landing   // 1 or 0
};

const logtenURL = `logten://import?data=${btoa(JSON.stringify(logtenData))}`;
```

---

### **References**

**FAA Regulations:**
- [14 CFR § 1.1 - Definitions](https://www.ecfr.gov/current/title-14/chapter-I/subchapter-A/part-1)
- [14 CFR § 61.57 - Recent flight experience](https://www.ecfr.gov/current/title-14/chapter-I/subchapter-D/part-61/subpart-A/section-61.57)
- [14 CFR § 91.209 - Aircraft lights](https://www.ecfr.gov/current/title-14/chapter-I/subchapter-F/part-91/subpart-C/section-91.209)
- [FAA Airplane Flying Handbook Chapter 11: Night Operations](https://www.faa.gov/sites/faa.gov/files/regulations_policies/handbooks_manuals/aviation/airplane_handbook/12_afh_ch11.pdf)

**API Documentation:**
- [Sunrise-Sunset.org API Docs](https://sunrise-sunset.org/api)
- [U.S. Naval Observatory - Astronomical Applications](https://aa.usno.navy.mil)

**Industry Standards:**
- [AOPA: Night Definitions and Operations](https://pilot-protection-services.aopa.org/news/2022/march/01/night-definitions-and-operations)
- [Boldmethod: When Can You Log Night Flight](https://www.boldmethod.com/learn-to-fly/regulations/logging-night-flight-time-and-night-landings-explained-2023/)

---

**Last Updated:** October 26, 2025  
**Algorithm Status:** ✅ Implemented and tested  
**Regulatory Compliance:** ✅ FAA 14 CFR § 1.1 compliant

## 📈 Recent Updates

**October 25, 2025:**
- ✅ Authentication system 100% complete
- ✅ Sync workflow functional with proper filtering
- ✅ Dashboard displaying real flight data
- ✅ LogTen Pro single-flight import working
- 📋 Documented day/night takeoff and landing calculation
- 📋 Added FAA regulatory framework reference
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
**Current Focus:** Day/night calculation documentation complete; ready for implementation after dashboard enhancements  
**Next Milestone:** Complete Phase 6 dashboard improvements, then implement Phase 7 day/night calculation
