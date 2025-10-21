# 🚀 Quick Start Guide

Get FlightBridge landing page & dashboard live in 10 minutes!

## Prerequisites

✅ n8n Cloud account (https://app.n8n.cloud)  
✅ Cloudflare account with Worker configured  
✅ Domain: flightbridge.app

---

## ⚠️ CRITICAL ISSUE FOUND (Oct 21, 2025)

### OAuth Flow Status: ✅ WORKING (via n8n direct) / ❌ BROKEN (via Cloudflare)

**Issue Discovered:**
- ✅ OAuth works perfectly: `https://kbarbershop.app.n8n.cloud/webhook/oauth/start`
- ❌ OAuth fails with 500 error: `https://flightbridge.app/webhook/oauth/start`

**Root Cause:** Cloudflare Worker at `flightbridge.app` is NOT properly proxying `/webhook/*` requests to n8n.

**Evidence:**
1. Test with real FCView passkey via n8n direct URL → ✅ Success
   - Authorization code received: `38113060c5785f61663785d5b9e3c8c5`
   - Access token obtained: `d2aa1e3124c91acbd34f9569d8c29e5daa6fbbb595b5b4a0d307dd680623c7b8`
   - Refresh token obtained: `46a0d766bbccf402c5131be751b38d356158a1d998d41e46b1959816f4f572dd`

2. Same passkey via Cloudflare domain → ❌ Internal Server Error (500)

**Next Action:** Fix Cloudflare Worker configuration in dashboard to properly proxy all `/webhook/*` paths to n8n.

**FCView OAuth Configuration:**
- Client ID: `f0cf9180d491f06e`
- Redirect URI (registered): `https://flightbridge.app/webhook/oauth/callback`
- OAuth endpoints working correctly
- Token exchange functioning properly

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

## Step 3: Configure Cloudflare Worker (3 minutes) ⚠️ NEEDS FIX

### Current Issue
The Cloudflare Worker is NOT properly proxying `/webhook/*` requests, causing OAuth to fail.

### Required Cloudflare Worker Code

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

**Critical:** This must properly proxy POST requests for OAuth to work.

### Deployment Steps:
1. Go to Cloudflare Dashboard → Workers & Pages
2. Edit existing worker or create new one
3. Paste code above
4. Deploy
5. Add route: `flightbridge.app/*` → Your worker

**Test URL After Fix**: https://flightbridge.app/webhook/oauth/start

---

## Step 4: Verify Everything Works (2 minutes)

### Check These URLs:

✅ **Landing Page**: https://flightbridge.app/  
✅ **Dashboard**: https://flightbridge.app/webhook/dashboard ✅ **NEW!**  
⚠️ **OAuth Start**: https://flightbridge.app/webhook/oauth/start (BROKEN - needs Cloudflare fix)
✅ **OAuth Start (direct)**: https://kbarbershop.app.n8n.cloud/webhook/oauth/start (WORKS!)

### Test Checklist:

- [ ] Landing page loads with styling
- [ ] Dashboard displays flight cards with sample data
- [ ] **OAuth flow works via Cloudflare (currently broken)**
- [ ] Mobile responsive (test on phone)
- [ ] All links work (Privacy, Terms, Contact)
- [ ] "Join Waitlist" button opens email client
- [ ] Filter buttons work on dashboard
- [ ] Page loads fast (< 2 seconds)

---

## OAuth Integration Status

### ✅ Working Components:
- FCView OAuth authorization page loads correctly
- Passkey validation working (tested with real user passkey)
- Authorization code exchange successful
- Access token and refresh token obtained
- Tokens stored in Supabase (with minor duplicate key issue to fix)

### ❌ Broken Components:
- Cloudflare Worker not proxying OAuth requests properly
- `flightbridge.app/webhook/oauth/start` returns 500 error
- POST requests to OAuth endpoints failing through Cloudflare

### 🔧 Current Workaround:
**Use n8n direct URLs for OAuth testing:**
- OAuth Start: `https://kbarbershop.app.n8n.cloud/webhook/oauth/start`
- OAuth Callback: `https://kbarbershop.app.n8n.cloud/webhook/oauth/callback`

### 📋 Next Steps:
1. **PRIORITY:** Fix Cloudflare Worker to proxy `/webhook/*` correctly
2. Minor: Update Supabase node to use UPSERT instead of INSERT (avoid duplicate key errors)
3. Test complete OAuth flow through `flightbridge.app` domain
4. Build dashboard to fetch and display user's real flights

---

## Step 5: Next Actions

### Immediate (Today):

1. **Fix Cloudflare Worker** (TOP PRIORITY)
   - Update worker code to properly proxy `/webhook/*` paths
   - Test OAuth flow through `flightbridge.app` domain
   - Verify POST requests work correctly

2. **Test OAuth Flow End-to-End**
   - Generate fresh passkey in FCView mobile app
   - Complete authorization via `flightbridge.app/webhook/oauth/start`
   - Verify tokens saved in Supabase
   - Confirm no duplicate key errors

### This Week:

- [ ] Complete OAuth integration through production domain
- [ ] Fix Supabase duplicate key constraint
- [ ] Build workflow to fetch flights from FCView API
- [ ] Connect dashboard to real flight data
- [ ] Test timezone correction logic

### This Month:

- [ ] Set up Stripe payments
- [ ] Build LogTen Pro URL generation
- [ ] Test complete sync flow: FCView → Dashboard → LogTen Pro
- [ ] Launch beta to first users

---

## Troubleshooting

### OAuth 500 Error at flightbridge.app?

**Cause:** Cloudflare Worker not proxying correctly

**Fix:** Update Cloudflare Worker code (see Step 3 above)

**Workaround:** Use n8n direct URL: `https://kbarbershop.app.n8n.cloud/webhook/oauth/start`

### Landing Page Not Loading?

**Check:**
1. n8n workflow is Active
2. Webhook path is `/home`
3. Cloudflare Worker routing is correct
4. DNS is pointing to Cloudflare

**Debug:**
- Test direct n8n URL first
- Check n8n execution logs
- Verify Cloudflare Worker logs

### Dashboard Not Loading?

**Check:**
1. n8n workflow is Active
2. Webhook path is `/dashboard`
3. Content-Type header is set correctly
4. HTML is not escaped in Code node

### Styling Issues?

**Check:**
- Content-Type header: `text/html; charset=utf-8`
- No HTML escaping in Code node
- Browser cache (hard refresh: Cmd+Shift+R)

---

## Dashboard Features ✅

### Current Features (UI Complete):
- ✅ Flight list with status badges (Pending/Imported)
- ✅ Filter buttons (All/Pending/Imported)
- ✅ Flight details (route, date, time, aircraft)
- ✅ "Import to LogTen Pro" buttons
- ✅ "Sync Now" button
- ✅ Mobile-responsive design
- ✅ Professional styling matching landing page

### Pending Features (After Cloudflare Fix):
- ⏳ OAuth flow through production domain
- ⏳ Live flight data from FCView API
- ⏳ User authentication
- ⏳ Working import to LogTen Pro functionality
- ⏳ Auto-sync capability
- ⏳ Settings page

---

## Current Status (Oct 21, 2025)

**✅ WORKING:**
- Landing page deployed
- Dashboard UI deployed
- OAuth flow functional (via n8n direct URL)
- FCView API integration tested and working
- Token exchange and storage working

**⚠️ ISSUES:**
- Cloudflare Worker NOT proxying `/webhook/*` correctly
- OAuth fails when accessed via `flightbridge.app`
- Minor Supabase duplicate key constraint

**🔧 NEXT ACTION:**
Fix Cloudflare Worker in dashboard to proxy all webhook requests to n8n.

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

## Support

**GitHub Issues**: https://github.com/flightbridge/flightbridge/issues  
**Email**: support@flightbridge.app  
**Documentation**: See README.md and DEPLOYMENT.md

---

## Resources

**Live URLs:**
- Landing Page: https://flightbridge.app/
- Dashboard: https://flightbridge.app/webhook/dashboard
- OAuth (broken via CF): https://flightbridge.app/webhook/oauth/start
- OAuth (working direct): https://kbarbershop.app.n8n.cloud/webhook/oauth/start

**Development:**
- GitHub Repo: https://github.com/flightbridge/flightbridge
- n8n Cloud: https://app.n8n.cloud
- Cloudflare Dashboard: https://dash.cloudflare.com

**API Resources:**
- FCView API Docs: https://help.flightcrewview.com/support/solutions/articles/16000189645
- FCView API Portal: https://flightcrewview2.com/logbook/logbookapiclientportal

---

**Questions?** Check DEPLOYMENT.md for detailed instructions.