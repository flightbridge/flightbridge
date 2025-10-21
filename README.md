# FlightBridge

**Sync Flight Crew View to LogTen Pro with Automatic Timezone Corrections**

🚀 **Status:** OAuth Working! Cloudflare Proxy Issue 🔧  
🌐 **Live:** https://flightbridge.app/  
📊 **Dashboard:** https://flightbridge.app/webhook/dashboard

---

## ✅ What We've Built So Far

### Phase 1: Foundation ✅ **COMPLETE**
- [x] Domain & DNS setup
- [x] Cloudflare Worker proxy
- [x] Landing page live (with Login button)
- [x] Privacy & Terms pages
- [x] Dashboard UI (test data)
- [x] Supabase database
- [x] GitHub repository

### Phase 2: API Integration ✅ **COMPLETE**
- [x] FCView API approval received
- [x] Client ID: `f0cf9180d491f06e`
- [x] Client Secret obtained
- [x] Redirect URI configured: `https://flightbridge.app/webhook/oauth/callback`

### Phase 3: OAuth & Authentication ✅ **WORKING**
- [x] OAuth Start workflow built (`/webhook/oauth/start`)
- [x] OAuth Callback workflow built (`/webhook/oauth/callback`)
- [x] Login page created and deployed (`/webhook/login`)
- [x] Landing page updated with Login button
- [x] OAuth flow **FULLY WORKING** via direct n8n URL ✅
- [x] Token exchange working ✅
- [x] Access & refresh tokens retrieved successfully ✅
- [ ] Cloudflare Worker proxy - **BROKEN** 🔴
- [ ] Supabase user storage - needs UPSERT logic

---

## 🎯 Current Status: OAuth Working! Cloudflare Proxy Broken

### ✅ **MAJOR BREAKTHROUGH - OAuth Flow 100% Functional:**

**What We Discovered:**
1. OAuth flow works PERFECTLY when using direct n8n URL: `https://kbarbershop.app.n8n.cloud/webhook/oauth/start`
2. FCView successfully validates passkeys (both TEST1234 and real passkeys)
3. FCView redirects to callback with authorization code
4. Token exchange completes successfully
5. Access token & refresh token received from FCView API

**Proof of Success (Execution 1726 & 1740):**
```json
{
  "code": "4cc3d5b394c147a7d00581d3be8b7e7b",
  "state": "new_user",
  "access_token": "d2aa1e3124c91acbd34f9569d8c29e5daa6fbbb595b5b4a0d307dd680623c7b8",
  "refresh_token": "46a0d766bbccf402c5131be751b38d356158a1d998d41e46b1959816f4f572dd",
  "expires_in": 3600
}
```

### 🔴 **Current Issue: Cloudflare Worker Proxy**

**Problem:**
- When using production domain `https://flightbridge.app/webhook/oauth/start` → **Internal Server Error**
- When using direct n8n URL `https://kbarbershop.app.n8n.cloud/webhook/oauth/start` → **Works perfectly**

**Root Cause:**
The Cloudflare Worker at `flightbridge.app` is not properly proxying webhook requests to n8n.

**Error Symptoms:**
- OAuth Start page shows "Internal Server Error" with 500 status
- FCView passkey submission returns 302 redirect back to start page (instead of callback)
- No n8n execution history when using `flightbridge.app` domain

### ⚠️ **Minor Issue: Supabase Duplicate Users**

**Current Error:**
```
duplicate key value violates unique constraint "users_email_key"
Key (email)=(new_user@temp.flightbridge.app) already exists
```

**Why It Happens:**
- Testing with same `state` parameter (`new_user`)
- Callback workflow tries to INSERT instead of UPSERT

**Fix Needed:**
Change Supabase node from `create` operation to `upsert` operation to handle existing users.

---

## 🔧 **NEXT: Fix Cloudflare Worker Proxy**

### **Required Cloudflare Worker Fix:**

The Worker needs to properly proxy `/webhook/*` requests to n8n:

```javascript
export default {
  async fetch(request, env) {
    const url = new URL(request.url);
    
    // Proxy webhook requests to n8n (including query params and POST body)
    if (url.pathname.startsWith('/webhook/')) {
      const n8nUrl = `https://kbarbershop.app.n8n.cloud${url.pathname}${url.search}`;
      
      return fetch(n8nUrl, {
        method: request.method,
        headers: request.headers,
        body: request.method !== 'GET' && request.method !== 'HEAD' ? request.body : null
      });
    }
    
    // Serve your static pages for other routes
    return fetch(request);
  }
}
```

### **Where to Find Worker Code:**
1. Check Cloudflare Dashboard → Workers & Pages
2. Look for worker attached to `flightbridge.app` domain
3. Or check for `_worker.js` or `functions/` folder in Pages deployment

### **Testing Steps After Fix:**
1. Test OAuth Start: `https://flightbridge.app/webhook/oauth/start`
2. Enter TEST1234 passkey
3. Should redirect to callback with code parameter
4. Verify n8n execution appears in workflow history

---

## 📋 Working URLs

### **Direct n8n URLs (WORKING):**
- OAuth Start: `https://kbarbershop.app.n8n.cloud/webhook/oauth/start`
- OAuth Callback: `https://kbarbershop.app.n8n.cloud/webhook/oauth/callback`

### **Production URLs (BROKEN - needs Cloudflare fix):**
- Landing Page: `https://flightbridge.app/`
- Login Page: `https://flightbridge.app/webhook/login`
- OAuth Start: `https://flightbridge.app/webhook/oauth/start` 🔴
- OAuth Callback: `https://flightbridge.app/webhook/oauth/callback` 🔴
- Dashboard: `https://flightbridge.app/webhook/dashboard`

### **FCView API Endpoints:**
- Authorization: `https://www.flightcrewview2.com/logbook/logbookuserauth/`
- Token Exchange: `https://www.flightcrewview2.com/logbook/api/token/`
- Flights API: `https://www.flightcrewview2.com/logbook/api/flights/`

---

## 🔧 Technical Configuration

### **OAuth Flow (Verified Working):**
1. User clicks "Login" → `/webhook/login`
2. Clicks "Connect with Flight Crew View" → `/webhook/oauth/start`
3. OAuth Start redirects to FCView with:
   - `client_id`: `f0cf9180d491f06e`
   - `redirect_uri`: `https://flightbridge.app/webhook/oauth/callback`
   - `state`: `new_user` (or unique user identifier)
4. User enters passkey on FCView authorization page
5. FCView validates passkey and redirects to callback: `https://flightbridge.app/webhook/oauth/callback?code=XXX&state=new_user`
6. Callback workflow exchanges authorization code for tokens via HTTP POST:
   ```
   POST https://www.flightcrewview2.com/logbook/api/token/
   Authorization: Basic {base64(client_id:client_secret)}
   Body: grant_type=authorization_code&code=XXX&redirect_uri=https://flightbridge.app/webhook/oauth/callback
   ```
7. Response contains `access_token`, `refresh_token`, `expires_in` (3600 seconds)
8. Tokens stored in Supabase users table
9. User redirected to dashboard

### **n8n Workflows:**
- **OAuth Start** (ID: `3XaAgDv6xmH8KXhI`): Active ✅
- **OAuth Callback** (ID: `DSZgk07w46MlGEY6`): Active ✅
- Both workflows tested and working via direct n8n URLs

### **Supabase Database Schema:**
```sql
CREATE TABLE users (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  email VARCHAR(255) UNIQUE NOT NULL,
  fcview_access_token TEXT,
  fcview_refresh_token TEXT,
  fcview_token_expires_at TIMESTAMP,
  subscription_status VARCHAR(50) DEFAULT 'trial',
  stripe_customer_id VARCHAR(255),
  auto_sync_enabled BOOLEAN DEFAULT true,
  fix_timezones BOOLEAN DEFAULT true,
  created_at TIMESTAMP DEFAULT NOW()
);
```

---

## 🟡 **AFTER Cloudflare Fix:**

### **Immediate Next Steps:**
1. Fix Cloudflare Worker proxy
2. Fix Supabase UPSERT logic (change `create` to `upsert`)
3. Test complete flow via production URLs
4. Add proper user state generation (not hardcoded "new_user")

### **Then: Flight Data Integration:**
1. Create "Sync Flights" workflow
2. Fetch flights from FCView API using access token
3. Parse flight data and apply timezone corrections
4. Store flights in Supabase
5. Display flights on dashboard
6. Generate LogTen Pro import URLs

---

## 📝 Important Notes

### **Test Passkey:**
- Passkey: `TEST1234`
- Always works for testing
- Returns static test flight data
- Authorization codes valid for 1 hour (vs 5 minutes for real passkeys)

### **Real Passkeys:**
- Generated in FCView mobile app: Menu → Settings → Logbook Sync → Generate Passkey
- Valid for 10 minutes after generation
- Can only be used once
- Must be entered immediately after generation

### **Redirect URI Requirements:**
- MUST be registered in FCView Client Portal: `https://flightbridge.app/webhook/oauth/callback`
- MUST match exactly in OAuth Start workflow
- Must use HTTPS (HTTP not allowed)
- Currently proxied through Cloudflare (which is broken)

### **Token Lifetimes:**
- Access Token: 1 hour (3600 seconds)
- Refresh Token: 3 months
- Authorization Code: 5 minutes (1 hour for TEST1234)

---

## 🐛 Debug Information

### **Successful Test Execution (1740):**
```
Start Time: 2025-10-21T23:25:02.512Z
Workflow: OAuth Callback (DSZgk07w46MlGEY6)
Status: Error (Supabase duplicate key)

Input:
- Code: 38113060c5785f61663785d5b9e3c8c5
- State: new_user
- Referer: https://www.flightcrewview2.com/

Token Exchange Response:
- Access Token: d2aa1e3124c91acbd34f9569d8c29e5daa6fbbb595b5b4a0d307dd680623c7b8
- Refresh Token: 46a0d766bbccf402c5131be751b38d356158a1d998d41e46b1959816f4f572dd
- Expires In: 3600 seconds

Error:
- Node: Save to Supabase
- Message: duplicate key value violates unique constraint "users_email_key"
- Key: (email)=(new_user@temp.flightbridge.app)
```

### **Current Blockers:**
1. **P0 - Cloudflare Worker:** Breaks all webhook requests via production domain
2. **P1 - Supabase UPSERT:** Prevents re-testing with same user (minor, workaround exists)

---

**Last Updated:** October 21, 2025, 7:40 PM PST  
**Current Status:** OAuth fully functional via n8n! Next: Fix Cloudflare Worker proxy to enable production domain.
