# FlightBridge

**Sync Flight Crew View to LogTen Pro with Automatic Timezone Corrections**

🚀 **Status:** Legal pages live, awaiting FCView API approval

---

## Overview

FlightBridge is a web service that automatically syncs flight data from Flight Crew View to LogTen Pro, with intelligent timezone corrections for professional pilots.

**Live URLs:**
- Landing Page: https://flightbridge.app (Coming Soon)
- Privacy Policy: https://flightbridge.app/webhook/privacy
- Terms of Service: https://flightbridge.app/webhook/terms

---

## Features

- ✈️ **Automatic Sync** - Set it and forget it flight data synchronization
- 🌍 **Timezone Corrections** - Intelligent timezone detection and conversion
- 🔒 **Secure OAuth** - Official Flight Crew View API integration
- 📱 **One-Click Import** - Generate LogTen Pro import URLs instantly
- ⏰ **Real-Time Updates** - Stay current with schedule changes
- 💼 **Professional Grade** - Built by pilots, for pilots

---

## Tech Stack

### Current Infrastructure

- **Frontend:** HTML landing page (served via n8n)
- **Backend:** n8n Cloud workflows (kbarbershop.app.n8n.cloud)
- **Database:** Supabase (PostgreSQL)
- **Domain:** flightbridge.app (via Cloudflare Worker proxy)
- **Hosting:** Cloudflare Workers (serverless)

### Architecture

```
User Request → flightbridge.app
    ↓
Cloudflare Worker (Proxy)
    ↓
n8n Cloud Workflows
    ↓
Supabase Database
```

---

## Development Roadmap

### Phase 1: Foundation ✅ COMPLETE
- [x] Domain purchased (flightbridge.app)
- [x] Cloudflare Worker proxy configured
- [x] Privacy Policy page live
- [x] Terms of Service page live
- [x] Supabase database setup
- [x] Landing page created
- [x] GitHub repository

### Phase 2: API Integration ⏳ WAITING
- [ ] FCView Indie Tier API approval (2-5 business days)
- [ ] OAuth Start workflow
- [ ] OAuth Callback workflow
- [ ] Test authorization flow

### Phase 3: Core Features 📋 PLANNED
- [ ] Fetch flights from FCView API
- [ ] Store flights in Supabase
- [ ] Apply timezone corrections
- [ ] Generate LogTen import URLs

### Phase 4: User Dashboard 📋 PLANNED
- [ ] User authentication
- [ ] Dashboard UI
- [ ] Flight history view
- [ ] One-click import buttons
- [ ] Settings page

### Phase 5: Payments 💰 PLANNED
- [ ] Stripe integration
- [ ] Subscription management
- [ ] 14-day free trial
- [ ] Billing portal

---

## Deployment

### Landing Page Deployment

1. Create new n8n workflow
2. Add Webhook node: `GET /webhook/home`
3. Add HTML response with `landing-page.html` content
4. Activate workflow
5. Test at https://flightbridge.app/webhook/home

### n8n Workflow Import

1. Log into n8n Cloud: https://app.n8n.cloud
2. Go to Workflows → Import from File
3. Select JSON file from `n8n_workflows/`
4. Activate workflow

---

## Costs

**Monthly Operating Costs:**
- n8n Cloud: $20/month
- Supabase: $0 (free tier)
- Cloudflare: $0 (free tier)
- Domain: $0.60/month
- **Total: $20.60/month**

**Break-even:** 3 paying users at $9.99/month

---

## API Documentation

### FCView API Resources
- Documentation: https://help.flightcrewview.com/support/solutions/articles/16000189645
- API Portal: https://www.flightcrewview2.com/logbook/logbookapiclientportal
- Privacy Policy: https://flightcrewview2.com/logbook/privacypolicy/
- Terms: https://flightcrewview2.com/logbook/terms/

### LogTen Pro Import Format
LogTen Pro accepts CSV imports via custom URL schemes. Documentation TBD.

---

## Support

**Email:** support@flightbridge.app

**Issues:** Please open a GitHub issue for bug reports or feature requests

---

## License

Proprietary - All Rights Reserved

---

## Acknowledgments

- Flight Crew View for providing the API
- LogTen Pro for the excellent pilot logbook app
- The pilot community for feedback and support

---

**Last Updated:** October 16, 2025

**Status:** Legal pages live, waiting for FCView API approval