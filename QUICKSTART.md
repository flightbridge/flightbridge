# 🚀 Quick Start Guide

Get FlightBridge landing page & dashboard live in 10 minutes!

## Prerequisites

✅ n8n Cloud account (https://app.n8n.cloud)  
✅ Cloudflare account with Worker configured  
✅ Domain: flightbridge.app

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

### Update Worker to Route Root Domain

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
    
    return await fetch(modifiedRequest);
  }
}
```

**Test URL**: https://flightbridge.app/

---

## Step 4: Verify Everything Works (2 minutes)

### Check These URLs:

✅ **Landing Page**: https://flightbridge.app/  
✅ **Dashboard**: https://flightbridge.app/webhook/dashboard ✅ **NEW!**  
✅ **Privacy Policy**: https://flightbridge.app/webhook/privacy  
✅ **Terms of Service**: https://flightbridge.app/webhook/terms  

### Test Checklist:

- [ ] Landing page loads with styling
- [ ] Dashboard displays flight cards with sample data
- [ ] Mobile responsive (test on phone)
- [ ] All links work (Privacy, Terms, Contact)
- [ ] "Join Waitlist" button opens email client
- [ ] Filter buttons work on dashboard
- [ ] Page loads fast (< 2 seconds)

---

## Step 5: Next Actions

### Immediate (Today):

1. **Email FCView for API Approval**
   ```
   To: support@flightcrewview.com
   Subject: Indie Tier API Access - Privacy Policy & Terms Completed
   
   Hi FCView Team,
   
   I've completed the required Privacy Policy and Terms of Service:
   - Privacy: https://flightbridge.app/webhook/privacy
   - Terms: https://flightbridge.app/webhook/terms
   
   FlightBridge syncs Flight Crew View to LogTen Pro with timezone corrections.
   
   Ready for Indie tier approval.
   
   Thank you!
   Byungchan Lim
   ```

2. **Share Landing Page & Dashboard**
   - Test with friends/colleagues
   - Get feedback on design
   - Validate value proposition

### This Week:

- [ ] Monitor FCView API approval (2-5 business days)
- [ ] Set up Google Analytics (optional)
- [ ] Create social media accounts (optional)
- [ ] Prepare OAuth workflows (after API approval)

### This Month:

- [ ] Build OAuth integration
- [x] ~~Develop user dashboard~~ ✅ **COMPLETE**
- [ ] Connect dashboard to live data
- [ ] Set up Stripe payments
- [ ] Launch beta to first users

---

## Troubleshooting

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

### Email Link Not Working?

**Update in landing-page.html:**
```html
<a href="mailto:support@flightbridge.app?subject=Waitlist">Join Waitlist</a>
```

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

### Pending Features (After API Approval):
- ⏳ Live flight data from FCView API
- ⏳ User authentication
- ⏳ Working import to LogTen Pro functionality
- ⏳ Auto-sync capability
- ⏳ Settings page

---

## Pro Tips

### Performance:
- Landing page is static HTML = ultra fast
- Dashboard uses minimal JavaScript = fast loading
- Served via CDN = global speed

### SEO:
- Meta tags already included
- Submit to Google Search Console
- Add structured data (optional)

### Analytics:
Add before `</body>`:
```html
<!-- Google Analytics (optional) -->
<script async src="https://www.googletagmanager.com/gtag/js?id=G-XXXXXXXXXX"></script>
<script>
  window.dataLayer = window.dataLayer || [];
  function gtag(){dataLayer.push(arguments);}
  gtag('js', new Date());
  gtag('config', 'G-XXXXXXXXXX');
</script>
```

### A/B Testing:
- Test different headlines
- Try different CTAs
- Experiment with pricing display

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
- Dashboard: https://flightbridge.app/webhook/dashboard ✅ **NEW!**
- Privacy: https://flightbridge.app/webhook/privacy
- Terms: https://flightbridge.app/webhook/terms

**Development:**
- GitHub Repo: https://github.com/flightbridge/flightbridge
- n8n Cloud: https://app.n8n.cloud
- Cloudflare Dashboard: https://dash.cloudflare.com

**API Resources:**
- FCView API Docs: https://help.flightcrewview.com/support/solutions/articles/16000189645
- FCView API Portal: https://flightcrewview2.com/logbook/logbookapiclientportal

---

**🎉 Congratulations! Your landing page & dashboard are live!**

**Current Status:** 
- ✅ Landing page deployed
- ✅ Dashboard UI deployed
- ⏳ Awaiting FCView API approval

**Next Step**: Email FCView for API approval, then connect dashboard to live data.

**Questions?** Check DEPLOYMENT.md for detailed instructions.
