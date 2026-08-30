# ✅ KEPENG Deployment Checklist

Langkah-langkah untuk deploy KEPENG ke production (Vercel) dalam 30 menit!

---

## 📋 Pre-Deployment (5 menit)

- [ ] Buka terminal di folder `kepeng-backend`
- [ ] Run: `npm install` (pastikan semua dependencies installed)
- [ ] Run: `npm run dev` (test locally terlebih dahulu)
- [ ] Buka http://localhost:3000 - pastikan loading
- [ ] Click "Generate Test Key" - pastikan working

---

## 🔧 Setup GitHub (5 menit)

### Jika belum punya GitHub repo:

```bash
# Initialize git
git init

# Add all files
git add .

# Commit
git commit -m "Initial KEPENG MVP commit"

# Create repo di github.com
# Lalu set remote origin

git remote add origin https://github.com/<YOUR_USERNAME>/kepeng.git
git branch -M main
git push -u origin main
```

### Jika sudah punya repo:
```bash
git add .
git commit -m "Add KEPENG implementation"
git push origin main
```

---

## 🚀 Deploy ke Vercel (10 menit)

### Method 1: Via Vercel Dashboard (EASIEST)

1. **Buka** https://vercel.com
2. **Sign up** dengan GitHub (if belum ada account)
3. **Click** "New Project"
4. **Select** GitHub repo `kepeng` (atau search nama repo-nya)
5. **Configure**:
   - Project Name: `kepeng` (atau custom)
   - Framework: `Next.js` (auto-detect)
   - Root Directory: `./` (atau path ke kepeng-backend)
6. **Set Environment Variables**:
   - Click "Environment Variables"
   - Add ini 3 variables:
     ```
     Name: JWT_SECRET
     Value: your-secret-key-here
     
     Name: MASTER_KEY_FINGERPRINT
     Value: b3e926e194930d53
     
     Name: NODE_ENV
     Value: production
     ```
7. **Click "Deploy"**
8. **Wait** 3-5 menit untuk build & deploy
9. **Done!** ✅ Vercel kasih URL (e.g., https://kepeng.vercel.app)

### Method 2: Via Vercel CLI

```bash
# Install Vercel CLI
npm i -g vercel

# Deploy
vercel

# Follow prompts:
# - Link to GitHub repo? (Y/n) - Y
# - Set project name? - kepeng
# - Deploy to production? (Y/n) - Y

# Vercel kasih URL - save it!
```

---

## 🧪 Post-Deployment Testing (5 menit)

### 1. Test Landing Page
```bash
curl https://kepeng.vercel.app/
# Should return HTML
```

### 2. Test API Key Generation
```bash
curl -X POST https://kepeng.vercel.app/api/v1/auth/keys \
  -H "Content-Type: application/json" \
  -d '{"userID": "test-user"}'

# Response: { "success": true, "data": {...} }
```

### 3. Test Transfer Creation
```bash
# Gunakan apiKey & secret dari step sebelumnya
curl -X POST https://kepeng.vercel.app/api/v1/transfer \
  -H "Authorization: Bearer <API_KEY>:<SECRET>" \
  -H "Content-Type: application/json" \
  -d '{
    "fromAccount": "123456",
    "fromBank": "BCA",
    "toAccount": "654321",
    "toBank": "GoPay",
    "amount": 100000,
    "currencyFrom": "IDR",
    "currencyTo": "IDR"
  }'

# Response: { "success": true, "data": {...} }
```

### 4. Test Interface Setup
```bash
curl -X POST https://kepeng.vercel.app/api/v1/interfaces \
  -H "Authorization: Bearer <API_KEY>:<SECRET>" \
  -H "Content-Type: application/json" \
  -d '{
    "interfaceID": "test-qr",
    "type": "qr"
  }'

# Response: { "success": true, "data": {...} }
```

---

## 📊 Monitoring (After Deploy)

### Check Vercel Dashboard
1. Buka https://vercel.com/dashboard
2. Click project `kepeng`
3. Check:
   - ✅ **Deployments**: Latest deployment should be "Ready"
   - ✅ **Status**: Green checkmark
   - ✅ **Logs**: No errors

### Monitor API Health
```bash
# Every 5 menit, test endpoint ini:
curl https://kepeng.vercel.app/
```

### View Logs (if error)
```bash
# Via Vercel CLI
vercel logs kepeng

# Atau buka Vercel Dashboard → Deployments → Logs
```

---

## 🔐 Security Hardening (After Deploy)

### 1. Change JWT Secret
Edit di Vercel Dashboard → Settings → Environment Variables:
```
JWT_SECRET=<GENERATE_RANDOM_STRING>
```

Generate random:
```bash
node -e "console.log(require('crypto').randomBytes(32).toString('hex'))"
```

### 2. Enable HTTPS Enforcement
Vercel auto-enable HTTPS - no action needed ✅

### 3. Setup Custom Domain (Optional)
Vercel Dashboard → Domains → Add domain
```
kepeng.io → points ke kepeng.vercel.app
```

### 4. Enable Vercel Protection (Optional)
Vercel Dashboard → Settings → Protection
- Add allowed email untuk akses dashboard

---

## 📧 Announce Launch

Setelah deploy sukses, announce ke users:

```markdown
🚀 KEPENG is now LIVE!

Try it out: https://kepeng.vercel.app

Features:
- Encrypted transfer API
- Multi-currency support
- Universal interface builder
- Barcode, QR, Link, Webhook, Embed

Get started in 5 minutes!
```

---

## 🔄 Continuous Deployment

Setelah setup, GitHub integration auto-deploy setiap push:

```bash
git add .
git commit -m "Fix: improve encryption"
git push origin main
# → Vercel auto-deploy dalam 2-3 menit!
```

---

## 🐛 Troubleshooting Deployment

### Problem: "Build failed"
```
Solution: Check logs di Vercel Dashboard
Likely: Missing env variables atau syntax error
Fix: Add correct env vars atau fix code & push again
```

### Problem: "Deployment stuck at building"
```
Solution: Click "Redeploy" di Vercel Dashboard
Or: Push new commit untuk trigger rebuild
```

### Problem: "API returns 404"
```
Solution: Check URL format
- Wrong: https://kepeng.vercel.app/transfer
- Right: https://kepeng.vercel.app/api/v1/transfer
```

### Problem: "Authentication error"
```
Solution: Check API key format
- Format: Authorization: Bearer <apiKey>:<secret>
- No spaces around colon!
```

### Problem: "Encryption error"
```
Solution: Check MASTER_KEY_FINGERPRINT di env vars
Should be: b3e926e194930d53
```

---

## 🎯 After Deployment

### 1. Create Test Transfers (10 minutes)
- [ ] Test create transfer (IDR → IDR)
- [ ] Test create transfer (USD → SGD)
- [ ] Test barcode generation
- [ ] Test QR generation
- [ ] Test payment link generation

### 2. Setup Analytics (Optional)
- [ ] Integrate Google Analytics
- [ ] Setup error tracking (Sentry)
- [ ] Setup uptime monitoring (UptimeRobot)

### 3. Documentation Update
- [ ] Update README dengan live URL
- [ ] Add API documentation link
- [ ] Share dengan early users

### 4. Gather Feedback
- [ ] Share dengan 10-20 beta users
- [ ] Collect feedback
- [ ] Plan improvements

---

## 📈 Scale Checklist (When Traffic Grows)

As kamu get more users:

- [ ] Setup Firebase untuk data storage
- [ ] Add real-time currency rates API
- [ ] Setup webhook retry system
- [ ] Add request logging & analytics
- [ ] Upgrade Vercel to Pro tier ($20/month)
- [ ] Add database backups
- [ ] Setup CDN (Cloudflare)
- [ ] Performance monitoring
- [ ] Security audit

---

## ✨ Deployment Success Indicators

Project berhasil di-deploy kalau:

✅ Landing page loads (https://kepeng.vercel.app)
✅ Can generate API key
✅ Can create transfer
✅ Can execute transfer
✅ Can check status
✅ Encryption working
✅ Interface generator working
✅ No 500 errors di logs

---

## 🎉 Congratulations!

Kamu sekarang punya **production-ready payment infrastructure**!

**KEPENG Live URL**: https://kepeng.vercel.app

**Next steps**:
1. Share dengan users
2. Gather feedback
3. Improve based on feedback
4. Scale & monetize

---

## 📞 Emergency Contacts

Jika ada issue:

1. **Check Vercel Logs**: https://vercel.com/dashboard
2. **Check API**: Via curl atau Postman
3. **Check GitHub**: Push latest code
4. **Restart**: Vercel auto-restart

---

**CONGRATS! KEPENG is LIVE! 🚀🔌**

Now go get users! 💪
