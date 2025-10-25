# 🚀 Quick Start Guide

Get FlightBridge landing page & dashboard live in 10 minutes!

## Prerequisites

✅ n8n Cloud account (https://app.n8n.cloud)  
✅ Cloudflare account with Worker configured  
✅ Domain: flightbridge.app

---

## Current Status (Oct 25, 2025)

### ✅ COMPLETED:
- **Authentication System**: Full OAuth flow working
  - OAuth Start & Callback workflows ✅
  - Login/Signup workflows ✅
  - Password reset workflows ✅
  - Tokens storing correctly in Supabase ✅
- **Landing Page**: Deployed and live
- **Dashboard**: Deployed with real flight data
- **Sync Workflow**: "FlightBridge Sync FCView" functional
  - Filters deadhead/BDM/AVL/DTY flights ✅
  - Filters incomplete past flights (no actual_out_local) ✅
  - Filters to last 3 months ✅
  - INSERT/UPDATE/SKIP logic implemented ✅
  - Merge node issue resolved (using Append mode) ✅
- **LogTen Pro Import**: Single-flight import button working ✅

### 🔄 IN PROGRESS:
- Sync workflow testing (waiting for new flights to test INSERT/UPDATE/SKIP)
- Sync button comprehensive testing

### 📋 TODO LIST:

#### High Priority:
1. **Test INSERT/UPDATE/SKIP Logic**
   - Need new flights from FCView to verify detection
   - Confirm UPDATE detection when flight data changes
   - Verify SKIP for unchanged flights

2. **Pilot Flying Field** (Required before Import)
   - Add dropdown: Yes/No/[blank default]
   - User must select before import
   - Show validation message if not selected: "Please select Pilot Flying (Yes/No) before importing"
   - Block import button until selected

3. **Dashboard Crew Information**
   - Add PIC (Captain) field to expandable "More Details" section
   - Add SIC (First Officer) field to expandable "More Details" section

#### Medium Priority:
4. **Date Filter Improvements**
   - Replace dropdown with calendar date picker
   - Add date range filter (From/To dates)
   - Apply filters to flight list display

5. **Mass Import Feature**
   - Add "Import All Filtered" button
   - Apply current date range filter
   - Batch import all visible flights
   - Show progress indicator during import

#### Future Enhancements:
- Timezone correction logic verification
- Auto-sync scheduling
- Stripe payment integration
- Email notifications for sync completion

---

## Step 1: Deploy Landing Page (5 minutes)

### Option A: Manual Setup (Recommended for learning)

1. **Log into n8n**: https://app.n8n.cloud

2. **Create New Workflow**: "FlightBridge Landing Page"

3. **Add 3 Nodes**:
   - Webhook (GET `/home`)
   - Code (copy HTML from `landing-page.html`)
   - Respond to Webhook (Content-Type: `text/html`)

4. **Configure Code Node**:
```javascript
const html = `
<!-- Paste entire landing-page.html content here -->
`;

return [{ json: { html } }];
```

5. **Activate Workflow** ✅

### Option B: Import Workflow (Faster)

1. Download: `n8n_workflows/landing_page_workflow.json`
2. Import into n8n
3. Update Code node with HTML content
4. Activate ✅

**Test URL**: https://kbarbershop.app.n8n.cloud/webhook/home

---

## Step 2: Deploy Dashboard ✅ (5 minutes)

### Same Process as Landing Page

1. **Create New Workflow**: "FlightBridge Dashboard"

2. **Add 3 Nodes**:
   - Webhook (GET `/dashboard`)
   - Code (copy HTML from `dashboard.html`)
   - Respond to Webhook (Content-Type: `text/html`)

3. **Configure Code Node**:
```javascript
const html = `
<!-- Paste entire dashboard.html content here -->
`;

return [{ json: { html } }];
```

4. **Activate Workflow** ✅

**Test URL**: https://kbarbershop.app.n8n.cloud/webhook/dashboard

**Live URL**: https://flightbridge.app/webhook/dashboard

---

## Step 3: Configure Cloudflare Worker (3 minutes)

### Cloudflare Worker Code

**Deploy this in Cloudflare Dashboard → Workers & Pages:**

```javascript
export default {
  async fetch(request) {
    const url = new URL(request.url);
    
    // Proxy ALL /webhook/* paths to n8n
    if (url.pathname.startsWith('/webhook/')) {
      const n8nUrl = `https://kbarbershop.app.n8n.cloud${url.pathname}${url.search}`;
      
      return fetch(n8nUrl, {
        method: request.method,
        headers: request.headers,
        body: request.method !== 'GET' && request.method !== 'HEAD' 
          ? await request.text() 
          : undefined
      });
    }
    
    // Route root to landing page
    if (url.pathname === '/' || url.pathname === '') {
      return fetch('https://kbarbershop.app.n8n.cloud/webhook/home');
    }
    
    // Serve static pages
    if (url.pathname === '/login') {
      return fetch('https://raw.githubusercontent.com/flightbridge/flightbridge/main/login-page.html');
    }
    
    if (url.pathname === '/dashboard') {
      return fetch('https://kbarbershop.app.n8n.cloud/webhook/dashboard');
    }
    
    return new Response('Not Found', { status: 404 });
  }
};
```

### Deployment Steps:
1. Go to Cloudflare Dashboard → Workers & Pages
2. Edit existing worker or create new one
3. Paste code above
4. Deploy
5. Add route: `flightbridge.app/*` → Your worker

---

## Step 4: Authentication Setup ✅

### Completed Workflows:

1. **FlightBridge OAuth - Step 1 Start** ✅
   - Initiates FCView OAuth flow
   - Redirects to FCView authorization page

2. **FlightBridge OAuth - Step 2 Callback** ✅
   - Exchanges authorization code for tokens
   - Stores access_token & refresh_token in Supabase
   - Handles token expiration

3. **FlightBridge Login** ✅
   - Supabase authentication
   - Session management

4. **FlightBridge Password Reset** ✅
   - Password reset request
   - Email verification
   - Password update

**OAuth Configuration:**
- Client ID: `f0cf9180d491f06e`
- Redirect URI: `https://flightbridge.app/webhook/oauth/callback`
- Token expiration: 1 hour (access), 3 months (refresh)

---

## Step 5: Sync Workflow Setup ✅

### FlightBridge Sync FCView Workflow

**Purpose**: Background sync from FCView API to Supabase database

**Flow**:
1. Webhook trigger: `/sync-flights?user_id=xxx`
2. Find user in Supabase
3. Check token expiration → Refresh if needed
4. Fetch flights from FCView API (last 3 months)
5. Fetch existing flights from Supabase
6. Compare and identify: INSERT / UPDATE / SKIP
7. Execute database operations

**Filters Applied**:
- ✅ Deadhead flights excluded (`is_deadhead === 1`)
- ✅ Special flight types excluded (BDM, AVL, DTY)
- ✅ Incomplete past flights excluded (no `actual_out_local` if scheduled time passed)
- ✅ Date range: Last 3 months from `actual_out_local`

**Change Detection Fields**:
- `actual_out_local`
- `actual_in_local`
- `tail_number`
- `flight_number`

---

## Step 6: Verify Everything Works

### Check These URLs:

✅ **Landing Page**: https://flightbridge.app/  
✅ **Dashboard**: https://flightbridge.app/webhook/dashboard  
✅ **OAuth Start**: https://flightbridge.app/webhook/oauth/start  
✅ **Login**: https://flightbridge.app/webhook/login

### Test Checklist:

- [x] Landing page loads with styling
- [x] Dashboard displays flight cards with real data
- [x] OAuth flow complete (authorization → tokens → database)
- [x] Login/signup working
- [x] Password reset working
- [x] Single flight import to LogTen Pro working
- [x] Mobile responsive
- [ ] Sync button (pending comprehensive testing)
- [ ] INSERT/UPDATE/SKIP logic (pending new flights)
- [ ] Date range filter
- [ ] Mass import feature

---

## Dashboard Features

### Current Features:
- ✅ Real flight data from Supabase
- ✅ Flight list with status badges (Pending/Imported)
- ✅ Filter buttons (All/Pending/Imported)
- ✅ Flight details (route, date, time, aircraft, block time)
- ✅ "Import to LogTen Pro" button (working!)
- ✅ "Sync Now" button (needs testing)
- ✅ Expandable "More Details" section
- ✅ Mobile-responsive design

### Pending UI Improvements:

**1. Pilot Flying Field (High Priority)**
```html
<!-- Add to each flight card BEFORE import button -->
<div class="pilot-flying-selector">
  <label>Were you Pilot Flying?</label>
  <select id="pilot-flying-{flight_id}" required>
    <option value="">-- Select --</option>
    <option value="Y">Yes</option>
    <option value="N">No</option>
  </select>
</div>

<!-- Update import button logic -->
<script>
function importFlight(flightId) {
  const pilotFlying = document.getElementById(`pilot-flying-${flightId}`).value;
  
  if (!pilotFlying) {
    alert('Please select Pilot Flying (Yes/No) before importing');
    return;
  }
  
  // Proceed with import...
}
</script>
```

**2. Crew Information in More Details**
```html
<!-- Add to expandable section -->
<div class="detail-row">
  <span class="label">Captain (PIC):</span>
  <span class="value">{{PIC}}</span>
</div>
<div class="detail-row">
  <span class="label">First Officer (SIC):</span>
  <span class="value">{{SIC}}</span>
</div>
```

**3. Date Range Filter (Replace Current Dropdown)**
```html
<!-- Replace date dropdown with calendar date pickers -->
<div class="date-filter">
  <label>From:</label>
  <input type="date" id="date-from" onchange="filterFlights()">
  
  <label>To:</label>
  <input type="date" id="date-to" onchange="filterFlights()">
</div>
```

**4. Mass Import Feature**
```html
<!-- Add button next to Sync Now -->
<button class="mass-import-btn" onclick="massImport()">
  📦 Import All Filtered ({count} flights)
</button>

<script>
async function massImport() {
  // Get all filtered & visible flights
  const visibleFlights = getFilteredFlights();
  
  // Validate all have pilot_flying selected
  const missingPF = visibleFlights.filter(f => !f.pilot_flying);
  if (missingPF.length > 0) {
    alert(`Please select Pilot Flying for ${missingPF.length} flight(s)`);
    return;
  }
  
  // Show progress
  showProgressModal(`Importing ${visibleFlights.length} flights...`);
  
  // Import each flight
  for (const flight of visibleFlights) {
    await importFlight(flight.id);
  }
  
  hideProgressModal();
  alert('✅ All flights imported!');
}
</script>
```

---

## Database Schema

### Users Table:
```sql
CREATE TABLE users (
  id UUID PRIMARY KEY,
  email VARCHAR(255) UNIQUE,
  fcview_access_token TEXT,
  fcview_refresh_token TEXT,
  fcview_token_expires_at TIMESTAMP,
  subscription_status VARCHAR(50) DEFAULT 'trial',
  created_at TIMESTAMP DEFAULT NOW()
);
```

### Flights Table:
```sql
CREATE TABLE flights (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID REFERENCES users(id) ON DELETE CASCADE,
  fcview_flight_id VARCHAR(255) UNIQUE,
  
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
  PIC VARCHAR(255),
  SIC VARCHAR(255),
  pilot_flying VARCHAR(1), -- 'Y' or 'N', default NULL (requires user input)
  
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

## Testing Notes

### What Needs Testing:

1. **Sync Workflow INSERT/UPDATE/SKIP**
   - ⏳ Waiting for new flights from FCView
   - Need to verify UPDATE detection when flight changes
   - Need to verify SKIP for unchanged flights
   - Current: All flights show as INSERT (expected for first sync)

2. **Sync Button Comprehensive Test**
   - Trigger manual sync
   - Verify new flights detected
   - Verify updated flights detected
   - Check execution logs in n8n

3. **Mass Import Feature** (After Implementation)
   - Select date range
   - Filter flights
   - Set pilot_flying for all
   - Trigger mass import
   - Verify all imported to LogTen Pro

---

## Troubleshooting

### Sync Workflow Showing All INSERT?

**Expected Behavior**: First sync after clean database will show all as INSERT

**Future Syncs Should**:
- INSERT: New flights from FCView not in database
- UPDATE: Flights in database with changed data
- SKIP: Flights in database with no changes

**To Test**: 
1. Run sync once (all INSERT)
2. Run sync again without changes (all SKIP expected)
3. Change a flight in FCView
4. Run sync again (should detect UPDATE)

### Import Button Not Working?

**Check**:
1. pilot_flying field selected?
2. LogTen Pro installed on device?
3. LogTen URL format correct?
4. Browser console for JavaScript errors

### Date Filter Not Working?

**After Calendar Implementation**:
1. Check date format (YYYY-MM-DD)
2. Verify filter function logic
3. Test edge cases (same day, empty dates)

---

## Cost Reminder

**Current Monthly Costs:**
- n8n Cloud: $20/month
- Supabase: $0 (free tier)
- Cloudflare: $0 (free tier)
- Domain: $0.60/month
- **Total: $20.60/month**

**Break-even: 3 paying users @ $9.99/month**

---

## Next Sprint Goals

### This Week:
- [ ] Test INSERT/UPDATE/SKIP with new flights
- [ ] Implement pilot_flying dropdown validation
- [ ] Add PIC/SIC to More Details section
- [ ] Replace date dropdown with calendar picker
- [ ] Add date range filter

### Next Week:
- [ ] Implement mass import feature
- [ ] Comprehensive sync button testing
- [ ] Performance optimization
- [ ] Error handling improvements

### This Month:
- [ ] Stripe payment integration
- [ ] Auto-sync scheduling (daily)
- [ ] Email notifications
- [ ] Settings page
- [ ] Beta user testing

---

## Support

**GitHub Issues**: https://github.com/flightbridge/flightbridge/issues  
**Email**: support@flightbridge.app  
**Documentation**: See README.md and DEPLOYMENT.md

---

## Resources

**Live URLs:**
- Landing Page: https://flightbridge.app/
- Dashboard: https://flightbridge.app/webhook/dashboard
- OAuth Start: https://flightbridge.app/webhook/oauth/start
- Login: https://flightbridge.app/webhook/login

**Development:**
- GitHub Repo: https://github.com/flightbridge/flightbridge
- n8n Cloud: https://app.n8n.cloud
- Cloudflare Dashboard: https://dash.cloudflare.com
- Supabase: https://supabase.com/dashboard

**API Resources:**
- FCView API Docs: https://help.flightcrewview.com/support/solutions/articles/16000189645
- FCView Client Portal: https://flightcrewview2.com/logbook/logbookapiclientportal
- LogTen Pro URL Scheme: See `logten pro api doc`

---

**Questions?** Check DEPLOYMENT.md for detailed instructions.