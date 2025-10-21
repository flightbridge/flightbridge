# FlightBridge

**Sync Flight Crew View to LogTen Pro with Automatic Timezone Corrections**

🚀 **Status:** Dashboard LIVE! Awaiting FCView API approval  
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

**Worker Code:**
```javascript
export default {
  async fetch(request) {
    const url = new URL(request.url);
    
    // Route root to landing page
    if (url.pathname === '/' || url.pathname === '') {
      url.pathname = '/webhook/home';
    }
    
    // Proxy to n8n
    url.hostname = 'kbarbershop.app.n8n.cloud';
    
    const modifiedRequest = new Request(url.toString(), {
      method: request.method,
      headers: request.headers,
      body: request.body,
      redirect: 'follow'
    });
    
    const response = await fetch(modifiedRequest);
    return response;
  }
}
```

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

#### 5. **User Dashboard** ✅ **NEW!**
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
- **Status:** UI complete, awaiting API integration for live data

#### 6. **n8n Workflows (Active)** ✅
All workflows hosted on n8n Cloud:

| Workflow | Path | Purpose | Status |
|----------|------|---------|--------|
| Landing Page | `/webhook/home` | Main website | ✅ Active |
| Privacy Policy | `/webhook/privacy` | Legal compliance | ✅ Active |
| Terms of Service | `/webhook/terms` | Legal compliance | ✅ Active |
| **Dashboard** | `/webhook/dashboard` | User flight management | ✅ **Active** |
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

---

## 🛠️ Tech Stack & Tools

### Services We're Using

#### 1. **Namecheap** (Domain Registrar)
- **What:** Domain registration
- **Domain:** flightbridge.app
- **Cost:** $7.20/year ($0.60/month)
- **Login:** Your Namecheap account
- **Purpose:** Own the domain name

#### 2. **Cloudflare** (DNS + Proxy)
- **What:** DNS management + serverless edge worker
- **Plan:** Free tier
- **Account:** chaaan123@gmail.com
- **Dashboard:** https://dash.cloudflare.com
- **Purpose:** 
  - DNS routing
  - SSL/HTTPS certificates (automatic)
  - Cloudflare Worker (proxy to n8n)
  - CDN/edge caching

#### 3. **n8n Cloud** (Backend/Automation)
- **What:** No-code workflow automation platform
- **Plan:** Creator ($20/month)
- **Account:** admin@k-barbershop.com
- **URL:** https://kbarbershop.app.n8n.cloud
- **Dashboard:** https://app.n8n.cloud
- **Purpose:**
  - Serve HTML pages (landing, privacy, terms, dashboard)
  - Handle webhooks
  - OAuth flow (future)
  - API integrations (future)
  - Business logic (future)

#### 4. **Supabase** (Database)
- **What:** PostgreSQL database with REST API
- **Plan:** Free tier
- **Account:** admin@k-barbershop.com (or Google OAuth)
- **Dashboard:** https://supabase.com/dashboard
- **Purpose:**
  - Store user accounts
  - Store OAuth tokens
  - Store flight data
  - Store sync history

#### 5. **GitHub** (Version Control)
- **What:** Code repository and documentation
- **Repo:** https://github.com/flightbridge/flightbridge
- **Account:** flightbridge
- **Purpose:**
  - Source code backup
  - Documentation
  - Collaboration
  - Version history

#### 6. **Flight Crew View API** (Data Source)
- **What:** Official FCView Logbook API
- **Status:** Awaiting Indie tier approval
- **Portal:** https://flightcrewview2.com/logbook/logbookapiclientportal
- **Purpose:** Fetch user flight schedules
- **Auth:** OAuth 2.0

#### 7. **LogTen Pro** (Data Destination)
- **What:** Pilot logbook iOS app
- **Integration:** CSV import via custom URL scheme
- **Purpose:** Import synced flights to user's logbook

---

## 🏗️ Infrastructure Setup

### Complete Architecture Diagram

```
┌─────────────────────────────────────────────────────────────────┐
│                           USER'S BROWSER                         │
└────────────────────────────┬────────────────────────────────────┘
                             │
                             │ HTTPS Request
                             │
                             ▼
┌─────────────────────────────────────────────────────────────────┐
│                    NAMECHEAP DNS                                 │
│  Domain: flightbridge.app                                        │
│  Nameservers: → Cloudflare                                       │
└────────────────────────────┬────────────────────────────────────┘
                             │
                             ▼
┌─────────────────────────────────────────────────────────────────┐
│                    CLOUDFLARE                                    │
│  • DNS Management                                                │
│  • SSL/TLS Certificates (automatic)                              │
│  • Cloudflare Worker: flightbridge-proxy                         │
│    - Routes / → /webhook/home                                    │
│    - Proxies all requests to n8n                                 │
└────────────────────────────┬────────────────────────────────────┘
                             │
                             │ Proxied Request
                             │
                             ▼
┌─────────────────────────────────────────────────────────────────┐
│                    N8N CLOUD                                     │
│  URL: kbarbershop.app.n8n.cloud                                  │
│                                                                  │
│  Active Workflows:                                               │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │ 1. Landing Page Workflow                                  │  │
│  │    Webhook → Code (HTML) → Respond                        │  │
│  │    Path: /webhook/home                                    │  │
│  └──────────────────────────────────────────────────────────┘  │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │ 2. Privacy Policy Workflow                                │  │
│  │    Webhook → Code (HTML) → Respond                        │  │
│  │    Path: /webhook/privacy                                 │  │
│  └──────────────────────────────────────────────────────────┘  │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │ 3. Terms of Service Workflow                              │  │
│  │    Webhook → Code (HTML) → Respond                        │  │
│  │    Path: /webhook/terms                                   │  │
│  └──────────────────────────────────────────────────────────┘  │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │ 4. Dashboard Workflow ✅ NEW!                             │  │
│  │    Webhook → Code (HTML) → Respond                        │  │
│  │    Path: /webhook/dashboard                               │  │
│  │    Features: Flight list, import buttons, filters         │  │
│  └──────────────────────────────────────────────────────────┘  │
│                                                                  │
│  Future Workflows (After API Approval):                          │
│  • OAuth Start                                                   │
│  • OAuth Callback                                                │
│  • Fetch Flights                                                 │
│  • Sync to Database                                              │
└────────────────────────────┬────────────────────────────────────┘
                             │
                             │ Database Queries
                             │
                             ▼
┌─────────────────────────────────────────────────────────────────┐
│                    SUPABASE (PostgreSQL)                         │
│                                                                  │
│  Tables:                                                         │
│  • users (id, email, fcview_tokens, created_at)                  │
│  • flights (id, user_id, flight_data, timezone_corrected)       │
└─────────────────────────────────────────────────────────────────┘
```

### Current Data Flow (Static Pages + Dashboard)

```
User visits flightbridge.app/webhook/dashboard
  ↓
Cloudflare DNS resolves to Worker
  ↓
Worker proxies to n8n Cloud
  ↓
n8n Webhook receives request
  ↓
Code node generates Dashboard HTML
  ↓
Respond node sends HTML back
  ↓
User sees flight dashboard with sample data
```

---

## 🔄 Backend Logic Flow

### Current Implementation (Phase 1 + Phase 4)

**Dashboard Workflow:**
```javascript
// n8n Workflow: FlightBridge User Dashboard
// Path: /webhook/dashboard

1. Webhook Node (Trigger)
   - Type: GET request
   - Path: /webhook/dashboard
   - Response mode: Using 'Respond to Webhook' node
   
2. Code Node (JavaScript)
   const html = `...full dashboard HTML...`;
   // Includes:
   // - Flight list display
   // - Filter buttons (All/Pending/Imported)
   // - Import to LogTen buttons
   // - Sync Now functionality
   // - Mobile responsive design
   return [{ json: { html } }];
   
3. Respond to Webhook Node
   - Respond with: text
   - Response body: ={{ $json.html }}
   - Headers: Content-Type: text/html; charset=utf-8
```

**Current Features:**
- ✅ Professional UI matching landing page theme
- ✅ Flight card layout with all flight details
- ✅ Status badges (Pending/Imported)
- ✅ Filter functionality (client-side)
- ✅ "Import to LogTen Pro" button placeholders
- ✅ "Sync Now" button
- ✅ Empty state messaging
- ⏳ Awaiting live data integration (after API approval)

---

### Future Implementation (After API Approval)

#### OAuth Flow Logic

**Workflow: OAuth Start**
```
User clicks "Connect FCView"
  ↓
n8n generates OAuth URL with:
  - client_id
  - redirect_uri (flightbridge.app/webhook/oauth-callback)
  - scope (flight read permissions)
  - state (random token for security)
  ↓
Redirect user to FCView authorization page
  ↓
User approves access
  ↓
FCView redirects to callback URL with authorization code
```

**Workflow: OAuth Callback**
```
Receive authorization code
  ↓
Exchange code for access token + refresh token
  ↓
Store tokens in Supabase users table
  ↓
Create user session
  ↓
Redirect to dashboard
```

#### Flight Sync Logic

**Workflow: Fetch & Sync Flights**
```
Trigger: Manual or scheduled (daily)
  ↓
For each user in database:
  ↓
  Get user's FCView access token
  ↓
  Call FCView API: GET /api/v1/flights
  ↓
  Parse flight data:
    - Date
    - Origin airport
    - Destination airport
    - Departure time (UTC)
    - Arrival time (UTC)
    - Aircraft type
    - Flight number
  ↓
  Apply timezone corrections:
    - Look up airport timezone
    - Convert UTC times to local times
    - Calculate block time in local timezone
  ↓
  Store corrected flight data in Supabase flights table
  ↓
  Generate LogTen Pro import URL
  ↓
End loop
```

---

## 🗺️ Development Roadmap

### Phase 1: Foundation ✅ **COMPLETE**
- [x] Domain purchased (flightbridge.app) - October 16
- [x] Cloudflare Worker proxy configured - October 16
- [x] Privacy Policy page live - October 16
- [x] Terms of Service page live - October 16
- [x] Supabase database setup - Earlier
- [x] Landing page created and deployed - October 16
- [x] GitHub repository with full documentation - October 16

### Phase 2: API Integration ⏳ **IN PROGRESS**
- [ ] Send FCView API approval email - **DO THIS NOW!**
- [ ] Wait for FCView Indie Tier approval (2-5 business days)
- [ ] Receive Client ID & Client Secret
- [ ] Build OAuth Start workflow
- [ ] Build OAuth Callback workflow
- [ ] Test authorization flow with your own FCView account
- [ ] Store tokens securely in Supabase

**Estimated Timeline:** 1-2 weeks (waiting on FCView)

### Phase 3: Core Features 📋 **NEXT**
- [ ] Build "Fetch Flights" workflow
  - Call FCView API endpoint
  - Parse JSON response
  - Error handling
- [ ] Implement timezone correction logic
  - Airport timezone lookup
  - UTC to local time conversion
  - Validation
- [ ] Store flights in Supabase
  - Insert new flights
  - Update existing flights
  - De-duplication logic
- [ ] Generate LogTen Pro import URLs
  - CSV format conversion
  - Custom URL scheme
  - Testing with LogTen Pro app
- [ ] Connect dashboard to live flight data

**Estimated Timeline:** 1-2 weeks of development

### Phase 4: User Dashboard ✅ **COMPLETE**
- [x] Dashboard UI created (n8n HTML workflow) - October 21
  - [x] Flight history table with status badges
  - [x] Filter functionality (All/Pending/Imported)
  - [x] "Import to LogTen Pro" buttons
  - [x] "Sync Now" button
  - [x] Mobile-responsive design
  - [x] Empty state messaging
  - [x] Professional styling matching landing page
- [ ] Connect to live data (pending API approval)
- [ ] User authentication system
  - [ ] Login page
  - [ ] Session management
  - [ ] Protected routes
- [ ] Settings page
  - [ ] Auto-sync toggle
  - [ ] Timezone preferences
  - [ ] Account management
- [ ] User onboarding flow
  - [ ] Welcome screen
  - [ ] FCView connection wizard
  - [ ] First sync

**UI Status:** ✅ Complete  
**Data Integration:** ⏳ Pending API approval  
**Estimated Timeline for Auth:** 1-2 weeks after API approval

### Phase 5: Payments 💰 **PLANNED**
- [ ] Stripe account setup
- [ ] Stripe integration in n8n
  - Checkout flow
  - Webhook handlers
- [ ] Subscription management
  - 14-day free trial
  - Monthly billing
  - Cancel/resume subscription
- [ ] Billing portal
  - Payment history
  - Update payment method
  - Invoices

**Estimated Timeline:** 1-2 weeks

### Phase 6: Launch & Growth 🚀
- [ ] Beta testing with 5-10 pilots
- [ ] Fix bugs and gather feedback
- [ ] Public launch
- [ ] Marketing (aviation forums, social media)
- [ ] Content marketing (blog posts, guides)
- [ ] Paid ads (if needed)

**Estimated Timeline:** Ongoing

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

### Financial Projections

**Break-Even:**
- 3 users × $9.99 = $29.97/month
- Profit: $9.37/month
- **Break-even at 3 users!**

**Target (Year 1):**
- 10 users × $9.99 = $99.90/month
- Annual: $1,198.80
- Costs: $247.20/year
- **Profit: $951.60/year**

**Growth Scenario (Year 2):**
- 50 users × $9.99 = $499.50/month
- Annual: $5,994
- Costs: $247.20/year
- **Profit: $5,746.80/year**

**Realistic Target:**
- 0.1% of 10,000 pilots = 10 users
- Conservative, achievable goal

---

## 🔗 Important Links

### Live URLs
- **Landing Page:** https://flightbridge.app/
- **Dashboard:** https://flightbridge.app/webhook/dashboard ✅ **NEW!**
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
- **FCView Privacy:** https://flightcrewview2.com/logbook/privacypolicy/
- **FCView Terms:** https://flightcrewview2.com/logbook/terms/

---

## 📞 Support

**Email:** support@flightbridge.app

**GitHub Issues:** https://github.com/flightbridge/flightbridge/issues

---

## 🎯 Next Immediate Actions

1. ✅ Landing page deployed
2. ✅ Legal pages deployed
3. ✅ Cloudflare Worker configured
4. ✅ Documentation complete
5. ✅ **Dashboard UI complete** - October 21
6. **🔴 NEXT: Send FCView API approval email**
7. **🔴 THEN: Connect dashboard to live flight data**

---

**Last Updated:** October 21, 2025, 3:22 PM PST  
**Current Status:** Dashboard UI LIVE, awaiting FCView API approval  
**Latest Deployment:** User dashboard at https://flightbridge.app/webhook/dashboard
