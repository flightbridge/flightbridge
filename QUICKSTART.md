# 🚀 Quick Start Guide

Get FlightBridge landing page live in 10 minutes!

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

## Step 2: Configure Cloudflare Worker (3 minutes)

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

## Step 3: Verify Everything Works (2 minutes)

### Check These URLs:

✅ **Landing Page**: https://flightbridge.app/  
✅ **Privacy Policy**: https://flightbridge.app/webhook/privacy  
✅ **Terms of Service**: https://flightbridge.app/webhook/terms  

### Test Checklist:

- [ ] Landing page loads with styling
- [ ] Mobile responsive (test on phone)
- [ ] All links work (Privacy, Terms, Contact)
- [ ] "Join Waitlist" button opens email client
- [ ] Page loads fast (< 2 seconds)

---

## Step 4: Next Actions

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

2. **Share Landing Page**
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
- [ ] Develop user dashboard
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

## Pro Tips

### Performance:
- Landing page is static HTML = ultra fast
- No JavaScript = works everywhere
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

**🎉 Congratulations! Your landing page is live!**

**Next Step**: Email FCView for API approval, then wait 2-5 business days.

**Questions?** Check DEPLOYMENT.md for detailed instructions.