# FlightBridge

**Sync Flight Crew View to LogTen Pro with Automatic Timezone Corrections**

🚀 **Status:** OAuth Testing in Progress 🔄  
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
- [x] Client ID & Client Secret obtained
- [x] Redirect URI configured: `https://flightbridge.app/webhook/oauth/callback`

### Phase 3: OAuth & Authentication ✅ **MOSTLY COMPLETE**
- [x] OAuth Start workflow built (`/webhook/oauth/start`)
- [x] OAuth Callback workflow built (`/webhook/oauth/callback`)
- [x] Login page created and deployed (`/webhook/login`)
- [x] Landing page updated with Login button
- [x] OAuth flow tested - user sees FCView authorization page ✅
- [x] User can enter passkey successfully ✅
- [ ] OAuth Callback debugging - **IN PROGRESS** (Internal Server Error after passkey)
- [ ] Token storage in Supabase - **BLOCKED** (waiting for callback fix)

---

## 🎯 Current Status: OAuth Callback Debugging

### ✅ **What's Working:**
1. Landing page at flightbridge.app
2. Login page at flightbridge.app/webhook/login
3. OAuth Start flow - redirects to FCView correctly
4. FCView authorization page displays
5. User can enter passkey
6. FCView accepts passkey

### 🔴 **Current Issue:**
- After user submits passkey, FCView redirects to callback
- Callback workflow returns "Internal Server Error"
- Need to check n8n Executions tab for exact error

### **Next Debug Steps:**
1. Check OAuth Callback workflow execution history
2. Identify which node is failing (HTTP Request? Supabase?)
3. Check error message details
4. Fix the failing node
5. Test complete flow end-to-end

---

## 📋 Workflow URLs

### **User-Facing URLs:**
- Landing Page: `https://flightbridge.app/`
- Login Page: `https://flightbridge.app/webhook/login`
- Dashboard: `https://flightbridge.app/webhook/dashboard`

### **OAuth Endpoints:**
- OAuth Start: `https://flightbridge.app/webhook/oauth/start`
- OAuth Callback: `https://flightbridge.app/webhook/oauth/callback`

### **FCView OAuth URLs:**
- Authorization: `https://logbook.flightcrewview.com/oauth/authorize`
- Token Exchange: `https://logbook.flightcrewview.com/oauth/token`
- API Base: `https://logbook.flightcrewview.com/api/v1`

---

## 🔧 Technical Configuration

### **OAuth Flow:**
1. User clicks "Login" → `/webhook/login`
2. Clicks "Connect with Flight Crew View" → `/webhook/oauth/start`
3. OAuth Start redirects to FCView with:
   - `client_id`: `f0cf918bd491f06e`
   - `redirect_uri`: `https://flightbridge.app/webhook/oauth/callback`
   - `response_type`: `code`
   - `state`: random security token
4. User enters passkey on FCView page
5. FCView redirects back to callback with `code` parameter
6. Callback exchanges code for access_token
7. Tokens stored in Supabase
8. User redirected to dashboard

### **Key Files:**
- Landing page: `landing-page.html` (440 lines, Login button in header)
- Login page: `login-page.html` (OAuth connection page)
- OAuth workflows: `n8n_workflows/` directory

---

## 🟡 **NEXT: Fix OAuth Callback**

After callback is fixed:
1. ✅ Test complete OAuth flow with real user
2. ✅ Verify tokens stored in Supabase
3. ✅ Test token refresh mechanism
4. 🔄 Fetch real flight data from FCView API
5. 🔄 Parse and store flights in Supabase
6. 🔄 Display flights on dashboard

---

## 🟢 **THEN: Flight Data & LogTen Export**
- Timezone correction logic
- Generate LogTen Pro import URLs
- Automatic sync scheduling

---

## 📝 Important Notes

### **Redirect URI Configuration:**
- **MUST match** between FCView settings and n8n workflow
- Currently set to: `https://flightbridge.app/webhook/oauth/callback`
- Uses Cloudflare proxy (not direct n8n URL)

### **Passkey Authentication:**
- Users generate passkey in FCView app: Settings → Logbook Sync → Generate Passkey
- Passkey is NOT the same as FCView password
- Passkeys can be regenerated (invalidates old one)

### **Files Removed:**
- "Now Live" status badge removed from landing page (cleaner design)

---

**Last Updated:** October 21, 2025, 4:15 PM PST  
**Current Status:** Debugging OAuth callback workflow - Internal Server Error after passkey submission
