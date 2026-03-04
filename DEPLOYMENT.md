# Artlor Website - Deployment Guide

## Overview

This guide covers the complete deployment process for the Artlor custom artwork commission platform. The site is built with React, TypeScript, and Vite, and is optimized for production deployment.

## Environment Setup

### Development Environment
```bash
# Install dependencies
npm install

# Start development server
npm run dev

# Build for production
npm run build

# Preview production build
npm run preview
```

### Production Environment Variables

Create a `.env.production` file with:

```env
VITE_APP_NAME=Artlor
VITE_API_BASE_URL=https://api.artlor.com
VITE_ANALYTICS_ID=G-XXXXXXXXXX
```

## Build Configuration

The project uses Vite with the following configuration:

- **Entry Point:** `src/main.tsx`
- **Output Directory:** `dist/`
- **Build Target:** Modern browsers (ES2020)
- **Code Splitting:** Enabled for optimal bundle size
- **Source Maps:** Disabled in production

## Deployment Options

### Option 1: Vercel (Recommended)

#### Setup
```bash
# Install Vercel CLI
npm install -g vercel

# Deploy
vercel
```

#### Configuration (`vercel.json`)
```json
{
  "buildCommand": "npm run build",
  "outputDirectory": "dist",
  "env": {
    "NODE_ENV": "production"
  },
  "headers": [
    {
      "source": "/assets/(.*)",
      "headers": [
        {
          "key": "Cache-Control",
          "value": "public, max-age=31536000, immutable"
        }
      ]
    },
    {
      "source": "/(.*)",
      "headers": [
        {
          "key": "X-Content-Type-Options",
          "value": "nosniff"
        },
        {
          "key": "X-Frame-Options",
          "value": "SAMEORIGIN"
        },
        {
          "key": "X-XSS-Protection",
          "value": "1; mode=block"
        }
      ]
    }
  ],
  "rewrites": [
    {
      "source": "/(.*)",
      "destination": "/index.html"
    }
  ]
}
```

### Option 2: Netlify

#### Setup
```bash
# Install Netlify CLI
npm install -g netlify-cli

# Deploy
netlify deploy --prod
```

#### Configuration (`netlify.toml`)
```toml
[build]
  command = "npm run build"
  publish = "dist"

[[redirects]]
  from = "/*"
  to = "/index.html"
  status = 200

[[headers]]
  for = "/*"
  [headers.values]
    X-Content-Type-Options = "nosniff"
    X-Frame-Options = "SAMEORIGIN"
    X-XSS-Protection = "1; mode=block"
    Referrer-Policy = "strict-origin-when-cross-origin"

[[headers]]
  for = "/assets/*"
  [headers.values]
    Cache-Control = "public, max-age=31536000, immutable"

[[headers]]
  for = "/*.css"
  [headers.values]
    Cache-Control = "public, max-age=604800"

[[headers]]
  for = "/*.js"
  [headers.values]
    Cache-Control = "public, max-age=604800"
```

### Option 3: AWS S3 + CloudFront

#### Setup
```bash
# Build the project
npm run build

# Sync to S3
aws s3 sync dist/ s3://your-bucket-name --delete

# Invalidate CloudFront
aws cloudfront create-invalidation --distribution-id YOUR_DIST_ID --paths "/*"
```

#### Configuration
- **S3 Bucket:** Static website hosting enabled
- **CloudFront Distribution:** Cache optimized
- **SSL Certificate:** AWS Certificate Manager
- **Distribution Domain:** Use custom domain with Route53

### Option 4: Traditional Shared Hosting

#### Via FTP/SFTP
```bash
# Build project
npm run build

# Upload dist folder contents to public_html/
# Keep .htaccess in root for URL rewriting
```

#### Required `.htaccess` (Already in `public/.htaccess`)
```apache
<IfModule mod_rewrite.c>
  RewriteEngine On
  RewriteCond %{REQUEST_FILENAME} !-f
  RewriteCond %{REQUEST_FILENAME} !-d
  RewriteRule ^ index.html [L]
</IfModule>
```

## Domain & DNS Configuration

### 1. Register Domain
- Registrar: GoDaddy, Namecheap, Route53, etc.
- Use www and non-www variants

### 2. Configure DNS Records

For Vercel:
```
A Record:
  Name: @
  Value: 76.76.19.20
  
CNAME Record:
  Name: www
  Value: cname.vercel-dns.com
```

For Netlify:
```
CNAME Records:
  Name: @ 
  Value: (provided by Netlify)
  
  Name: www
  Value: (provided by Netlify)
```

### 3. SSL/TLS Certificate
- Automatically handled by Vercel/Netlify
- For traditional hosting: Let's Encrypt (free) or premium provider

## Performance Optimization

### 1. Asset Optimization
- Images already optimized (lazy loading, responsive)
- CSS minified by Vite
- JavaScript minified and tree-shaken
- Gzip compression enabled in .htaccess

### 2. Cache Strategy
```
Static Assets (1 year): /assets/*, /fonts/*
CSS/JS (1 month): *.css, *.js
HTML (2 days): index.html
API (Cache-Control from server): /api/*
```

### 3. CDN Configuration
- Use edge nodes for faster global delivery
- Cache static assets for 1 year
- Cache HTML for short duration
- Use compression (gzip/brotli)

## Monitoring & Analytics

### 1. Google Analytics
- Setup via Google Analytics 4
- Tag already in `index.html`
- Monitor user flows and conversions

### 2. Error Tracking
- Error Boundary catches React component errors
- Send to error tracking service:
  - **Sentry** (recommended)
  - **LogRocket**
  - **Rollbar**

Configuration for Sentry:
```typescript
// src/main.tsx
import * as Sentry from "@sentry/react";

Sentry.init({
  dsn: "YOUR_SENTRY_DSN",
  environment: import.meta.env.MODE,
  integrations: [
    new Sentry.Replay(),
  ],
  tracesSampleRate: 1.0,
});
```

### 3. Performance Monitoring
- Lighthouse CI for continuous monitoring
- Web Vitals tracking (LCP, FID, CLS)
- Error rate monitoring
- Uptime monitoring

### 4. Uptime Monitoring
- **UptimeRobot** (free tier available)
- **StatusPage.io** (customer-facing status)
- **DataDog** (comprehensive monitoring)

Configuration:
```
Check URL: https://artlor.com/
Interval: 5 minutes
Notifications: Email + Slack
```

## Security Checklist

### Before Deployment
- [ ] All API calls use HTTPS
- [ ] No sensitive data in frontend code
- [ ] Dependencies scanned for vulnerabilities
- [ ] Rate limiting configured on backend
- [ ] CORS properly configured
- [ ] Input validation on forms
- [ ] Output encoding for XSS prevention
- [ ] SQL injection protection (backend)

### After Deployment
- [ ] SSL certificate valid
- [ ] HSTS enabled
- [ ] CSP headers configured
- [ ] Security headers present
- [ ] Form submissions use CSRF tokens
- [ ] Regular security audits
- [ ] Dependency updates monitored

Content Security Policy (CSP):
```
default-src 'self';
script-src 'self' 'unsafe-inline' https://apis.google.com;
style-src 'self' 'unsafe-inline' https://fonts.googleapis.com;
img-src 'self' data: https:;
font-src 'self' https://fonts.gstatic.com;
connect-src 'self' https://api.artlor.com https://www.google-analytics.com;
```

## Rollback Plan

### If Issues Occur

1. **Quick Rollback (Vercel/Netlify)**
   ```bash
   # Revert to previous deployment
   vercel rollback
   # or use Netlify dashboard
   ```

2. **Manual Rollback**
   ```bash
   # Redeploy previous version
   git checkout <previous-commit>
   npm run build
   vercel deploy --prod
   ```

3. **DNS Fallback**
   - Have backup server ready
   - Update DNS to point to backup
   - Investigate issue on primary server

## Post-Deployment Steps

1. **Verification (30 minutes)**
   - [ ] Site loads successfully
   - [ ] All pages accessible
   - [ ] Animations work smoothly
   - [ ] Mobile responsiveness OK
   - [ ] Forms functional
   - [ ] No console errors
   - [ ] Analytics receiving data

2. **SEO Verification (1 day)**
   - [ ] Submit sitemap to Google Search Console
   - [ ] Submit to Bing Webmaster Tools
   - [ ] Check crawl status
   - [ ] Verify mobile-friendly report
   - [ ] Check Core Web Vitals

3. **Performance Verification (1 day)**
   - [ ] Run Lighthouse audit
   - [ ] Check page load times
   - [ ] Verify Core Web Vitals
   - [ ] Check error rate
   - [ ] Monitor user analytics

4. **Communication (Immediate)**
   - [ ] Notify team of successful deployment
   - [ ] Update status page if applicable
   - [ ] Document any changes
   - [ ] Create incident report if issues

## Maintenance

### Regular Tasks
- **Weekly:** Monitor error rates and performance
- **Monthly:** Review analytics and user feedback
- **Quarterly:** Security audit and dependency updates
- **Annually:** Full site audit and optimization review

### Update Process
```bash
# Update dependencies
npm update
npm audit fix

# Test thoroughly
npm run dev
npm run build
npm run preview

# Deploy when ready
vercel deploy --prod
```

## Contact & Support

- **Deployment Issues:** DevOps team
- **Code Issues:** Development team
- **Performance Issues:** Performance team
- **Security Issues:** Security team

## Additional Resources

- [Vite Deployment Guide](https://vitejs.dev/guide/static-deploy.html)
- [React Production Tips](https://react.dev/learn/deployment)
- [Web Vitals](https://web.dev/web-vitals/)
- [MDN Security Headers](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers)

---

**Last Updated:** 2025-01-24
**Version:** 1.0
**Status:** Ready for Deployment
