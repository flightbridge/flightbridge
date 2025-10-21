# FlightBridge

**Sync Flight Crew View to LogTen Pro with Automatic Timezone Corrections**

🚀 **Status:** Dashboard LIVE! FCView API APPROVED! ✅  
🌐 **Live:** https://flightbridge.app/  
📊 **Dashboard:** https://flightbridge.app/webhook/dashboard

---

## 📖 Table of Contents

- [What We're Building](#what-were-building)
- [What We've Built So Far](#what-weve-built-so-far)
- [Tech Stack & Tools](#tech-stack--tools)
- [Infrastructure Setup](#infrastructure-setup)
- [Backend Logic Flow](#backend-logic-flow)
- [Development Roadmap](#development-roadmap)
- [Costs & Business Model](#costs--business-model)

---

## 🎯 What We're Building

### The Problem
Professional pilots use **Flight Crew View** for their schedules and **LogTen Pro** for their digital logbooks. Currently, they must manually transfer flight data between these apps, which is:
- Time-consuming (manual data entry)
- Error-prone (especially timezone conversions)
- Tedious (repetitive work)

### The Solution: FlightBridge
An automated sync service that:
1. Connects to Flight Crew View via official OAuth API
2. Automatically fetches flight data
3. Applies intelligent timezone corrections
4. Generates one-click import URLs for LogTen Pro
5. Keeps logbooks up-to-date in real-time

### Target Market
- Professional airline pilots (US domestic airlines)
- Airlines using FLICA scheduling system
- Pilots using LogTen Pro for their logbook
- Estimated 10,000+ potential users

### Business Model
- **Price:** $9.99/month subscription
- **Trial:** 14-day free trial
- **Value Prop:** Save hours per month, ensure accuracy
- **Break-even:** 3 users = $29.97 > $20.60 costs

---

## ✅ What We've Built So Far

### Phase 1: Foundation ✅ **COMPLETE**

#### 1. **Domain & DNS** ✅
- Purchased `flightbridge.app` from Namecheap ($0.60/month)
- DNS pointed to Cloudflare nameservers
- CNAME record routing to Cloudflare Worker
- SSL/HTTPS working automatically

#### 2. **Cloudflare Worker Proxy** ✅
- Created `flightbridge-proxy` worker
- Routes all traffic to n8n backend
- Root path (`/`) redirects to landing page
- Zero-downtime serverless hosting

#### 3. **Landing Page** ✅
- Professional gradient design (purple theme)
- Mobile-responsive layout
- Hero section with CTA buttons
- 6 feature cards explaining value
- "How It Works" 3-step process
- Pricing card ($9.99/month)
- Footer with legal links
- **Live at:** https://flightbridge.app/

#### 4. **Legal Pages** ✅
- Privacy Policy: https://flightbridge.app/webhook/privacy
- Terms of Service: https://flightbridge.app/webhook/terms
- Both pages are FCView API compliance requirements
- Mobile-responsive, professionally formatted

#### 5. **User Dashboard** ✅
- **Live at:** https://flightbridge.app/webhook/dashboard
- Clean, modern interface matching landing page design
- Flight list display with:
  - Flight number and route (JFK → MIA)
  - Date, departure time, duration
  - Aircraft type and tail number
  - Status badges (Pending/Imported)
- Filter buttons (All/Pending/Imported)
- "Import to LogTen Pro" button for each flight
- "Sync Now" functionality
- Mobile-responsive design
- Empty state messaging
- **Note:** Currently showing test data (5 sample flights)
- **Status:** UI complete, ready for API integration

#### 6. **n8n Workflows (Active)** ✅
All workflows hosted on n8n Cloud:

| Workflow | Path | Purpose | Status |
|----------|------|---------|--------|
| Landing Page | `/webhook/home` | Main website | ✅ Active |
| Privacy Policy | `/webhook/privacy` | Legal compliance | ✅ Active |
| Terms of Service | `/webhook/terms` | Legal compliance | ✅ Active |
| Dashboard | `/webhook/dashboard` | User flight management | ✅ Active (test data) |
| Test Webhook | `/webhook/test` | Development/debugging | ✅ Active |

#### 7. **Database Setup** ✅
Supabase PostgreSQL database with tables:
- `users` - User accounts, OAuth tokens
- `flights` - Synced flight data

#### 8. **GitHub Repository** ✅
- Complete source code
- Documentation (README, DEPLOYMENT, QUICKSTART)
- Version control
- Backup of all workflows

### Phase 2: API Integration ✅ **COMPLETE**
- [x] Send FCView API approval email - DONE
- [x] FCView Indie Tier approval received ✅
- [x] Received Client ID & Client Secret ✅
- [ ] Build OAuth Start workflow - **NEXT**
- [ ] Build OAuth Callback workflow - **NEXT**
- [ ] Test authorization flow with your own FCView account
- [ ] Store tokens securely in Supabase

---

## 🗺️ Development Roadmap

### Phase 1: Foundation ✅ **COMPLETE**
- [x] Domain purchased (flightbridge.app)
- [x] Cloudflare Worker proxy configured
- [x] Privacy Policy page live
- [x] Terms of Service page live
- [x] Supabase database setup
- [x] Landing page created and deployed
- [x] GitHub repository with full documentation

### Phase 2: API Integration ✅ **COMPLETE**
- [x] Send FCView API approval email
- [x] FCView Indie Tier approval received ✅
- [x] Received Client ID & Client Secret ✅

### Phase 3: OAuth & Authentication 🔴 **IN PROGRESS - THIS IS NEXT**

**Priority Tasks:**
1. **Build OAuth Start Workflow** (2-3 hours)
   - Create `/webhook/oauth-start` endpoint
   - Generate FCView authorization URL with:
     - Client ID
     - Redirect URI: `flightbridge.app/webhook/oauth-callback`
     - Scope: flight read permissions
     - State parameter (security token)
   - Redirect user to FCView login

2. **Build OAuth Callback Workflow** (2-3 hours)
   - Create `/webhook/oauth-callback` endpoint
   - Receive authorization code from FCView
   - Exchange code for access token + refresh token
   - Store tokens securely in Supabase `users` table
   - Create user session
   - Redirect to dashboard

3. **Test OAuth Flow** (1 hour)
   - Test with your FCView account
   - Verify tokens are stored correctly
   - Verify redirect to dashboard works

**Estimated Timeline:** 1 day of focused work

### Phase 4: Fetch Real Flight Data 📋 **AFTER OAUTH**
- [ ] Build "Fetch Flights" workflow
  - Call FCView API endpoint with access token
  - Parse JSON response
  - Error handling (token expired, API errors)
- [ ] Store flights in Supabase
  - Insert new flights
  - Update existing flights
  - De-duplication logic
- [ ] Update dashboard to display real data
  - Replace test data with Supabase query
  - Show user's actual flights

**Estimated Timeline:** 2-3 days

### Phase 5: Timezone Correction & LogTen Export 📋 **PLANNED**
- [ ] Implement timezone correction logic
  - Airport timezone lookup (using timezone API)
  - UTC to local time conversion
  - Validation
- [ ] Generate LogTen Pro import URLs
  - CSV format conversion
  - Custom URL scheme: `logten://import?csv=...`
  - Testing with LogTen Pro app on iPhone

**Estimated Timeline:** 2-3 days

### Phase 6: User Dashboard Enhancement 📋 **PLANNED**
- [ ] User authentication system
  - Login page
  - Session management
  - Protected routes
- [ ] Settings page
  - Auto-sync toggle
  - Timezone preferences
  - Account management
- [ ] User onboarding flow
  - Welcome screen
  - FCView connection wizard
  - First sync

**Estimated Timeline:** 1-2 weeks

### Phase 7: Payments 💰 **PLANNED**
- [ ] Stripe account setup
- [ ] Stripe integration in n8n
  - Checkout flow
  - Webhook handlers
- [ ] Subscription management
  - 14-day free trial
  - Monthly billing
  - Cancel/resume subscription

**Estimated Timeline:** 1-2 weeks

### Phase 8: Launch & Growth 🚀
- [ ] Beta testing with 5-10 pilots
- [ ] Fix bugs and gather feedback
- [ ] Public launch
- [ ] Marketing (aviation forums, social media)

---

## 🎯 Next Immediate Actions (Priority Order)

### 1. **OAuth Start Workflow** 🔴 **DO THIS FIRST**
**Goal:** Allow users to connect their FCView account

**Steps:**
1. Get your FCView Client ID & Client Secret (you have this)
2. Create new n8n workflow: "OAuth Start"
3. Add webhook node: `/webhook/oauth-start`
4. Add Code node to generate FCView OAuth URL:
```javascript
const clientId = 'YOUR_CLIENT_ID';
const redirectUri = 'https://flightbridge.app/webhook/oauth-callback';
const scope = 'flights:read'; // Check FCView docs for exact scope
const state = Math.random().toString(36).substring(7); // Random security token

const authUrl = `https://flightcrewview2.com/logbook/oauth/authorize?client_id=${clientId}&redirect_uri=${redirectUri}&scope=${scope}&state=${state}&response_type=code`;

return [{
  json: {
    redirectUrl: authUrl,
    state: state // Store this to verify later
  }
}];
```
5. Store state in Supabase (for verification)
6. Redirect user to FCView

### 2. **OAuth Callback Workflow** 🔴 **DO THIS SECOND**
**Goal:** Receive tokens from FCView after user approves

**Steps:**
1. Create new n8n workflow: "OAuth Callback"
2. Add webhook node: `/webhook/oauth-callback`
3. Receive authorization code from FCView
4. Exchange code for access token:
```javascript
// POST to FCView token endpoint
const response = await fetch('https://flightcrewview2.com/logbook/oauth/token', {
  method: 'POST',
  body: JSON.stringify({
    grant_type: 'authorization_code',
    code: $json.code,
    client_id: 'YOUR_CLIENT_ID',
    client_secret: 'YOUR_CLIENT_SECRET',
    redirect_uri: 'https://flightbridge.app/webhook/oauth-callback'
  })
});

const tokens = await response.json();
// tokens.access_token
// tokens.refresh_token
```
5. Store tokens in Supabase `users` table
6. Create user session
7. Redirect to dashboard

### 3. **Test OAuth Flow** 🔴 **DO THIS THIRD**
- Visit `/webhook/oauth-start`
- Log in with your FCView account
- Verify redirect to callback works
- Verify tokens are stored in Supabase
- Verify redirect to dashboard

### 4. **Fetch Real Flights** 🟡 **AFTER OAUTH WORKS**
- Build workflow to call FCView API
- Display real flights on dashboard
- Replace test data

---

## 💰 Costs & Business Model

### Monthly Operating Costs
| Service | Plan | Cost | Purpose |
|---------|------|------|------------|
| n8n Cloud | Creator | $20.00 | Backend workflows & hosting |
| Supabase | Free | $0.00 | Database (up to 500MB) |
| Cloudflare | Free | $0.00 | DNS, SSL, Worker |
| Namecheap | Domain | $0.60 | flightbridge.app domain |
| **TOTAL** | | **$20.60/month** | |

### Revenue Model
- **Price:** $9.99/month per user
- **Trial:** 14-day free trial (no credit card required)
- **Billing:** Monthly subscription via Stripe

---

## 🔗 Important Links

### Live URLs
- **Landing Page:** https://flightbridge.app/
- **Dashboard:** https://flightbridge.app/webhook/dashboard (test data)
- **Privacy Policy:** https://flightbridge.app/webhook/privacy
- **Terms of Service:** https://flightbridge.app/webhook/terms

### Admin Dashboards
- **n8n Cloud:** https://app.n8n.cloud (admin@k-barbershop.com)
- **Cloudflare:** https://dash.cloudflare.com (chaaan123@gmail.com)
- **Supabase:** https://supabase.com/dashboard
- **GitHub:** https://github.com/flightbridge/flightbridge
- **Namecheap:** https://www.namecheap.com

### API Documentation
- **FCView API Docs:** https://help.flightcrewview.com/support/solutions/articles/16000189645
- **FCView API Portal:** https://flightcrewview2.com/logbook/logbookapiclientportal

---

**Last Updated:** October 21, 2025, 3:35 PM PST  
**Current Status:** FCView API approved ✅, OAuth implementation NEXT  
**Next Task:** Build OAuth Start workflow
