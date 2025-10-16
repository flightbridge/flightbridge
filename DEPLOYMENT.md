# FlightBridge Landing Page Deployment Guide

## Quick Deploy to n8n

### Step 1: Prepare the HTML Content

The landing page HTML is in `landing-page.html`. You'll need to copy this content into the n8n workflow.

### Step 2: Create the n8n Workflow

1. **Log into n8n Cloud**
   - Go to: https://app.n8n.cloud
   - Sign in with your credentials

2. **Create New Workflow**
   - Click "New Workflow" button
   - Name it: "FlightBridge Landing Page"

3. **Add Webhook Node**
   - Drag "Webhook" node onto canvas
   - Configure:
     - **HTTP Method:** GET
     - **Path:** `home`
     - **Response Mode:** "Last Node"
   - Save

4. **Add Code Node**
   - Drag "Code" node onto canvas
   - Connect Webhook to Code node
   - In the code editor, paste:

```javascript
// Read the landing-page.html content
const html = `
<!-- PASTE ENTIRE CONTENT OF landing-page.html HERE -->
`;

return [{
  json: {
    html: html
  }
}];
```

5. **Add Respond to Webhook Node**
   - Drag "Respond to Webhook" node
   - Connect Code node to Respond node
   - Configure:
     - **Respond With:** text
     - **Response Body:** `={{ $json.html }}`
     - Under **Options** → **Response Headers**:
       - Name: `Content-Type`
       - Value: `text/html; charset=utf-8`

6. **Activate Workflow**
   - Click "Active" toggle in top right
   - Your webhook URL will be: `https://kbarbershop.app.n8n.cloud/webhook/home`

### Step 3: Test the Landing Page

**Via n8n directly:**
```
https://kbarbershop.app.n8n.cloud/webhook/home
```

**Via your custom domain (after Cloudflare Worker is routing):**
```
https://flightbridge.app/webhook/home
```

---

## Alternative: Import JSON Workflow

1. **Download the workflow template**
   - Get `n8n_workflows/landing_page_workflow.json` from this repo

2. **Import into n8n**
   - In n8n, click the menu (three dots) → Import from File
   - Select the JSON file
   - The workflow structure will be created

3. **Update the HTML Content**
   - Open the "Load HTML" Code node
   - Replace the placeholder comment with the actual HTML from `landing-page.html`

4. **Activate**
   - Toggle workflow to Active

---

## Setting Up Root Domain (Optional)

If you want the landing page at `https://flightbridge.app/` instead of `/webhook/home`:

### Option 1: Update Cloudflare Worker

Modify your Cloudflare Worker to route root to `/webhook/home`:

```javascript
export default {
  async fetch(request) {
    const url = new URL(request.url);
    
    // Route root to landing page
    if (url.pathname === '/' || url.pathname === '') {
      url.pathname = '/webhook/home';
    }
    
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

### Option 2: Change Webhook Path

In the n8n Webhook node, change the path from `home` to empty string or `/`:
- This makes it accessible at: `https://kbarbershop.app.n8n.cloud/webhook`

---

## Troubleshooting

### Landing Page Not Loading
- Check workflow is Active
- Verify webhook path matches URL
- Check Cloudflare Worker is routing correctly
- Look at n8n execution logs

### CSS/Styling Issues
- Ensure Content-Type header is set to `text/html; charset=utf-8`
- Check for any HTML escaping in the Code node

### Mobile Responsiveness
- The landing page is already mobile-responsive
- Test on different screen sizes

---

## Next Steps After Deployment

1. **Update Links**
   - Make sure Privacy/Terms links work
   - Test "Join Waitlist" button email link

2. **SEO Optimization**
   - Add meta tags (already included)
   - Submit sitemap to search engines
   - Set up Google Analytics (optional)

3. **A/B Testing**
   - Test different headlines
   - Optimize conversion rates
   - Gather user feedback

4. **Connect to Backend**
   - Once FCView API approved
   - Build OAuth flows
   - Add real signup functionality

---

## Maintenance

- **Update Content:** Edit the HTML in the Code node
- **Analytics:** Add Google Analytics or Plausible
- **Performance:** Landing page is static, very fast
- **Backups:** Workflow JSON is saved in this GitHub repo

---

**Questions?** Email: support@flightbridge.app