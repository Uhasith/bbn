# Hostinger VPS Deployment Guide

## After Git Commit & Pull - What You Need to Set

### 1. Backend .env Configuration (`server/.env`)

Create or update `/path/to/your/app/server/.env`:

```env
PORT=5001
NODE_ENV=production
JWT_SECRET=ChangeThisToARandomString32CharsLong

# EMAIL OPTION 1: Gmail (Good for testing/development)
EMAIL_USER=your-email@gmail.com
EMAIL_PASS=your-16-digit-app-password

# EMAIL OPTION 2: Hostinger SMTP (Better for production)
# EMAIL_HOST=smtp.hostinger.com
# EMAIL_PORT=465
# EMAIL_SECURE=true
# EMAIL_USER=noreply@yourdomain.com
# EMAIL_PASS=your-email-password
```

### 2. Frontend .env Configuration (`client/.env`)

**Before building**, update `/path/to/your/app/client/.env`:

```env
# If using domain
VITE_BACKEND_URL=https://yourdomain.com

# If using subdomain
# VITE_BACKEND_URL=https://api.yourdomain.com

# If using IP with Nginx
# VITE_BACKEND_URL=https://your-ip-address
```

---

## Email Setup Options

### Option A: Gmail (Quick Test Setup)

**Steps:**
1. Go to https://myaccount.google.com/apppasswords
2. Enable 2-Factor Authentication if not enabled
3. Create App Password for "Mail"
4. Use in `.env`:

```env
EMAIL_USER=youremail@gmail.com
EMAIL_PASS=abcd efgh ijkl mnop  # (remove spaces when pasting)
```

**Pros:** Quick to setup, free
**Cons:** Daily sending limits (500 emails/day), may be blocked

---

### Option B: Hostinger SMTP (Recommended for Production)

**Steps:**
1. Login to Hostinger hPanel
2. Go to "Email" → Create email account (e.g., noreply@yourdomain.com)
3. Set a strong password
4. Use these SMTP settings:

```env
# For Hostinger SMTP, update server/src/routes/responses.js and instructions.js
# Or add to .env and modify code:
EMAIL_HOST=smtp.hostinger.com
EMAIL_PORT=465
EMAIL_SECURE=true
EMAIL_USER=noreply@yourdomain.com
EMAIL_PASS=your-email-password
```

**Hostinger SMTP Settings:**
- **SMTP Host:** smtp.hostinger.com
- **SMTP Port:** 465 (SSL) or 587 (TLS)
- **Username:** Your full email address
- **Password:** Email account password

**Pros:** Professional, reliable, no daily limits
**Cons:** Requires domain email setup

---

## Deployment Steps on Hostinger VPS

### 1. SSH into your VPS
```bash
ssh root@your-vps-ip
# or
ssh username@your-vps-ip
```

### 2. Navigate to your app directory
```bash
cd /var/www/yourdomain.com
# or wherever your app is located
```

### 3. Pull latest changes
```bash
git pull origin main
# or: git pull origin master
```

### 4. Configure Backend .env
```bash
cd server
nano .env  # or vim .env
```

**Paste this and modify:**
```env
PORT=5001
NODE_ENV=production
JWT_SECRET=ChangeThisToARandom32CharacterString123

# Option 1: Gmail
EMAIL_USER=youremail@gmail.com
EMAIL_PASS=your-app-password

# Option 2: Hostinger SMTP (uncomment to use)
# EMAIL_HOST=smtp.hostinger.com
# EMAIL_PORT=465
# EMAIL_SECURE=true
# EMAIL_USER=noreply@yourdomain.com
# EMAIL_PASS=your-smtp-password
```

Save: `Ctrl+X`, then `Y`, then `Enter`

### 5. Configure Frontend .env
```bash
cd ../client
nano .env
```

**Paste:**
```env
VITE_BACKEND_URL=https://yourdomain.com
```

Save: `Ctrl+X`, then `Y`, then `Enter`

### 6. Install Dependencies & Build
```bash
# Backend
cd /var/www/yourdomain.com/server
npm install --production

# Frontend
cd /var/www/yourdomain.com/client
npm install
npm run build
```

### 7. Restart Backend with PM2
```bash
pm2 restart bbn-backend
# or if not running yet:
# pm2 start /var/www/yourdomain.com/server/src/index.js --name bbn-backend
```

### 8. Check if it's running
```bash
pm2 status
pm2 logs bbn-backend  # Check for errors
```

---

## Quick Test: Which Email to Use?

### For Initial Testing (Quick & Easy):
✅ **Use Gmail with App Password**
- Takes 5 minutes to setup
- Works immediately
- Good for testing if everything works

### For Production (Professional):
✅ **Use Hostinger SMTP with your domain email**
- Takes 10-15 minutes to setup
- More reliable
- Looks professional (emails from noreply@yourdomain.com)

---

## If Using Hostinger SMTP, Update Backend Code

You need to modify the email transporter in two files:

### File 1: `server/src/routes/responses.js` (around line 335)
Find:
```javascript
const transporter = nodemailer.createTransport({
  service: "gmail",
  auth: {
    user: process.env.EMAIL_USER,
    pass: process.env.EMAIL_PASS,
  },
});
```

Replace with:
```javascript
const transporter = nodemailer.createTransport({
  host: process.env.EMAIL_HOST || "smtp.hostinger.com",
  port: process.env.EMAIL_PORT || 465,
  secure: process.env.EMAIL_SECURE || true,
  auth: {
    user: process.env.EMAIL_USER,
    pass: process.env.EMAIL_PASS,
  },
});
```

### File 2: `server/src/routes/instructions.js` (around line 69)
Same change as above.

---

## Testing After Deployment

1. **Access your site:** https://yourdomain.com
2. **Login as admin:** username: `admin`, password: `admin`
3. **Change admin password immediately!**
4. **Add a test instruction** with title and content
5. **Test as tester:**
   - Go to Tester panel
   - Complete a test
   - Submit questionnaire
   - Check if email arrives

---

## Troubleshooting

### Email not sending with Gmail:
```bash
# Check logs
pm2 logs bbn-backend

# Common issues:
# - App password has spaces (remove them)
# - 2FA not enabled
# - "Less secure apps" blocking (use App Password instead)
```

### Email not sending with Hostinger SMTP:
```bash
# Check:
# 1. Email account exists in hPanel
# 2. SMTP settings are correct
# 3. Port 465 or 587 is not blocked
# 4. Username is full email address

# Test SMTP connection:
telnet smtp.hostinger.com 465
```

### Frontend can't connect to backend:
```bash
# Check client/.env has correct URL
cat client/.env

# Rebuild frontend after changing .env
cd client
npm run build

# Copy build to server public folder (if needed)
```

### Port 5001 already in use:
```bash
lsof -ti:5001 | xargs kill -9
pm2 restart bbn-backend
```

---

## Recommended Setup for Hostinger VPS

**My recommendation:**

1. **Start with Gmail** for initial testing:
   - Quick to setup
   - Test if everything works
   - Good for development

2. **Switch to Hostinger SMTP** for production:
   - Professional emails
   - Better deliverability
   - No Gmail daily limits

---

## Security Checklist

- [ ] `.env` files are NOT committed to Git (check `.gitignore`)
- [ ] JWT_SECRET is a strong random string (32+ chars)
- [ ] Admin password changed from default
- [ ] HTTPS enabled (SSL certificate installed)
- [ ] Firewall configured (ufw)
- [ ] PM2 running with startup enabled

---

## Quick Commands Reference

```bash
# Check PM2 status
pm2 status

# View logs
pm2 logs bbn-backend

# Restart after changes
pm2 restart bbn-backend

# Check if backend is responding
curl http://localhost:5001/api/instructions

# Test email configuration
pm2 logs bbn-backend --lines 100 | grep -i email
```

---

That's it! Start with Gmail for quick testing, then switch to Hostinger SMTP for production use. 🚀
