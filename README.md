# ClearMyCredit - Production Deployment Guide

> ⚡ Generate professional FCRA-compliant credit dispute letters in 30 seconds

## Overview

ClearMyCredit is a static HTML application that helps users generate credit dispute letters. This guide covers everything needed to deploy to production securely.

---

## Critical Pre-Deployment Checklist

### ⚠️ MUST DO BEFORE LAUNCH

- [ ] **Configure Stripe Live Mode**
  - Log into [Stripe Dashboard](https://dashboard.stripe.com)
  - Switch to LIVE mode (toggle in top-left)
  - Create a Payment Link for $5
  - Copy the live link URL
  - Update `preview.html` → `CMC_CONFIG.STRIPE_PAYMENT_LINK`

- [ ] **Update Domain References**
  - Replace all `clearmycredit.com` references with your actual domain
  - Update Open Graph meta tags in `index.html`

- [ ] **Configure Web Server Security Headers**
  - See "Security Headers" section below
  - These MUST be configured at the server level

- [ ] **Test Payment Flow**
  - Complete a test payment with a real card (small amount)
  - Verify redirect after payment works correctly
  - Confirm letter unlocks properly

- [ ] **Privacy Policy & Legal Pages**
  - Create `privacy.html`
  - Create `terms.html`
  - Create `disclaimer.html`
  - Ensure compliance with your jurisdiction

---

## File Structure

```
clearmycredit/
├── index.html          # Landing page
├── form.html           # Multi-step form
├── preview.html        # Letter preview + paywall
├── result.html         # Payment success page
├── privacy.html        # Privacy policy (YOU MUST CREATE)
├── terms.html          # Terms of use (YOU MUST CREATE)
├── disclaimer.html     # Legal disclaimer (YOU MUST CREATE)
└── README.md           # This file
```

---

## Environment Variables

While this is a static site, these values need to be configured in the code:

| Variable | Location | Description |
|----------|----------|-------------|
| `STRIPE_PAYMENT_LINK` | `preview.html` line 285 | Your live Stripe payment link |
| `BASE_URL` | All files | Your production domain |
| `PRICE` | `preview.html` line 288 | Price displayed to users |

### Updating Stripe Configuration

1. Open `preview.html`
2. Find the `CMC_CONFIG` object (around line 280)
3. Replace:
   ```javascript
   STRIPE_PAYMENT_LINK: 'https://buy.stripe.com/YOUR_LIVE_PAYMENT_LINK_HERE'
   ```
4. With your actual live payment link from Stripe Dashboard

---

## Security Headers (REQUIRED)

Your web server MUST send these headers. Examples for common servers:

### Nginx

```nginx
server {
    listen 443 ssl http2;
    server_name clearmycredit.com www.clearmycredit.com;
    
    root /var/www/clearmycredit;
    index index.html;
    
    # Security Headers
    add_header Content-Security-Policy "default-src 'self'; script-src 'self' 'unsafe-inline' https://js.stripe.com; style-src 'self' 'unsafe-inline'; frame-src https://buy.stripe.com https://checkout.stripe.com; connect-src 'self' https://api.stripe.com; img-src 'self' data: https:;" always;
    add_header X-Frame-Options "DENY" always;
    add_header X-Content-Type-Options "nosniff" always;
    add_header X-XSS-Protection "1; mode=block" always;
    add_header Referrer-Policy "strict-origin-when-cross-origin" always;
    add_header Permissions-Policy "geolocation=(), microphone=(), camera=()" always;
    
    # SSL
    ssl_certificate /path/to/cert.pem;
    ssl_certificate_key /path/to/key.pem;
    
    location / {
        try_files $uri $uri/ =404;
    }
}

# Redirect HTTP to HTTPS
server {
    listen 80;
    server_name clearmycredit.com www.clearmycredit.com;
    return 301 https://$server_name$request_uri;
}
```

### Apache (.htaccess)

```apache
# Security Headers
Header always set Content-Security-Policy "default-src 'self'; script-src 'self' 'unsafe-inline' https://js.stripe.com; style-src 'self' 'unsafe-inline'; frame-src https://buy.stripe.com https://checkout.stripe.com; connect-src 'self' https://api.stripe.com; img-src 'self' data: https:;"
Header always set X-Frame-Options "DENY"
Header always set X-Content-Type-Options "nosniff"
Header always set X-XSS-Protection "1; mode=block"
Header always set Referrer-Policy "strict-origin-when-cross-origin"
Header always set Permissions-Policy "geolocation=(), microphone=(), camera=()"

# Enable HTTPS redirect
RewriteEngine On
RewriteCond %{HTTPS} off
RewriteRule ^(.*)$ https://%{HTTP_HOST}%{REQUEST_URI} [L,R=301]
```

### Cloudflare

If using Cloudflare, enable:
- Always Use HTTPS
- Automatic HTTPS Rewrites
- Security Level: Medium or High
- Browser Integrity Check: ON

---

## Deployment Options

### Option 1: Static Hosting (Recommended for beginners)

#### Netlify
1. Drag & drop your files to Netlify
2. Add `_headers` file:
   ```
   /*
     Content-Security-Policy: default-src 'self'; script-src 'self' 'unsafe-inline' https://js.stripe.com; style-src 'self' 'unsafe-inline'; frame-src https://buy.stripe.com https://checkout.stripe.com; connect-src 'self' https://api.stripe.com; img-src 'self' data: https:;
     X-Frame-Options: DENY
     X-Content-Type-Options: nosniff
     Referrer-Policy: strict-origin-when-cross-origin
   ```

#### Vercel
1. Install Vercel CLI: `npm i -g vercel`
2. Run: `vercel`
3. Add `vercel.json`:
   ```json
   {
     "headers": [
       {
         "source": "/(.*)",
         "headers": [
           { "key": "Content-Security-Policy", "value": "default-src 'self'; script-src 'self' 'unsafe-inline' https://js.stripe.com; style-src 'self' 'unsafe-inline'; frame-src https://buy.stripe.com https://checkout.stripe.com; connect-src 'self' https://api.stripe.com; img-src 'self' data: https:;" },
           { "key": "X-Frame-Options", "value": "DENY" },
           { "key": "X-Content-Type-Options", "value": "nosniff" },
           { "key": "Referrer-Policy", "value": "strict-origin-when-cross-origin" }
         ]
       }
     ]
   }
   ```

#### GitHub Pages
1. Push files to GitHub repository
2. Enable GitHub Pages in settings
3. Note: Limited header customization - consider Cloudflare in front

### Option 2: VPS/Server

1. Upload files to `/var/www/clearmycredit/`
2. Configure Nginx/Apache with security headers (see above)
3. Set up SSL certificate (Let's Encrypt recommended)
4. Enable HTTPS redirect

---

## Stripe Configuration

### Setting Up Your Payment Link

1. Go to [Stripe Dashboard](https://dashboard.stripe.com)
2. Switch to **LIVE** mode
3. Navigate to **Payment Links** → **Create**
4. Configure:
   - **Product**: Create new product "Credit Dispute Letter"
   - **Price**: $5.00 (or your desired price)
   - **After payment**: Redirect to `https://yourdomain.com/result.html?payment=success`
5. Copy the Payment Link URL
6. Update `preview.html` with this URL

### Testing Payments

**Before going live:**
1. Use Stripe test mode to verify flow
2. Use test card: `4242 4242 4242 4242`
3. Any future date, any CVC, any ZIP

**After going live:**
1. Make a small test payment with real card
2. Verify funds appear in Stripe dashboard
3. Confirm letter unlocks correctly

---

## Security Considerations

### Data Storage

- **User data is stored in browser localStorage**
- Data is **NOT encrypted** (browser limitation)
- Data auto-expires after 24 hours of inactivity
- Data never leaves the user's device
- **No server-side storage** = No database to secure

### PCI Compliance

Since you're using Stripe Payment Links:
- ✅ You are NOT handling card data directly
- ✅ Stripe handles all PCI compliance
- ✅ You only need Stripe's payment confirmation

### Privacy (GDPR/CCPA)

You MUST:
1. Create a Privacy Policy explaining data handling
2. Disclose that data is stored locally in browser
3. Provide instructions for users to clear their data
4. Not track users without consent

---

## Post-Launch Monitoring

### Check Daily

- [ ] Stripe Dashboard for failed payments
- [ ] Any user complaints or support requests
- [ ] Error logs (if available)

### Weekly

- [ ] Review payment success rate
- [ ] Check for any security alerts
- [ ] Verify SSL certificate validity

### Monthly

- [ ] Review and update dependencies (if any)
- [ ] Check for browser compatibility issues
- [ ] Analyze user flow for improvements

---

## Troubleshooting

### Payment not unlocking letter

1. Check browser console for JavaScript errors
2. Verify `localStorage` is enabled in browser
3. Confirm Stripe redirect URL includes `?payment=success`
4. Check that `preview.html` has correct Stripe link

### Form data not saving

1. Check browser allows localStorage
2. Private/Incognito mode may block storage
3. Clear browser cache and retry

### Security header errors

1. Verify headers are set at server level
2. Check CSP doesn't block Stripe domains
3. Ensure `frame-src` includes Stripe checkout

---

## Support & Maintenance

### Updating the Site

1. Make changes in development
2. Test thoroughly in staging
3. Update production files
4. Clear CDN cache if applicable

### Backup Strategy

- Keep all files in version control (Git)
- Tag releases with version numbers
- Document all changes

---

## Legal Disclaimer Template

Create `disclaimer.html` with content similar to:

```html
<!DOCTYPE html>
<html>
<head><title>Disclaimer - ClearMyCredit</title></head>
<body>
  <h1>Disclaimer</h1>
  <p><strong>ClearMyCredit is not a law firm.</strong></p>
  <p>We do not provide legal advice, legal opinions, or legal representation. 
  The information and tools provided are for self-help purposes only.</p>
  <p>Credit repair takes time and no specific outcome is guaranteed. 
  Results may vary based on individual circumstances.</p>
  <p>If you need legal advice, please consult with a qualified attorney.</p>
</body>
</html>
```

---

## Contact & Resources

- **Stripe Docs**: https://stripe.com/docs/payment-links
- **CSP Reference**: https://developer.mozilla.org/en-US/docs/Web/HTTP/CSP
- **Security Headers**: https://securityheaders.com/

---

## License

This is proprietary software. All rights reserved.

---

**Last Updated**: 2026-03-26  
**Version**: 1.0.0-production
