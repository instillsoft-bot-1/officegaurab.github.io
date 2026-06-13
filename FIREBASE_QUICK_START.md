# Firebase Configuration Guide - V2

## Quick Firebase Setup (15 minutes)

### 1️⃣ Create Firebase Project (3 min)

```
1. Open: https://console.firebase.google.com/
2. Click: "+ Add project"
3. Enter: "attendance-tracker-v2"
4. Select region: Your closest region
5. Click: "Create project"
6. Wait: Project creation (1-2 min)
```

### 2️⃣ Get Firebase Config (3 min)

```
1. Click: ⚙️ Project Settings (top-right)
2. Go to: "General" tab
3. Scroll: "Your apps" section
4. Click: "</>" (Web icon)
5. Register: "Attendance Tracker"
6. Copy: Entire firebaseConfig object
```

Your config will look like:
```javascript
const firebaseConfig = {
  apiKey: "AIzaSyD...",
  authDomain: "attendance-tracker-v2.firebaseapp.com",
  projectId: "attendance-tracker-v2",
  storageBucket: "attendance-tracker-v2.appspot.com",
  messagingSenderId: "123456789",
  appId: "1:123456789:web:abc123..."
};
```

### 3️⃣ Enable Phone Auth (5 min)

```
1. In Firebase: Click "Authentication" (left menu)
2. Click: "Sign-in method" tab
3. Find: "Phone" provider
4. Click: Phone provider
5. Toggle: "Enable" switch
6. Enter: YOUR test phone number (optional)
7. Click: "Save"
```

### 4️⃣ Configure reCAPTCHA (3 min)

```
1. In Firebase: Authentication → Settings tab
2. Find: "reCAPTCHA Enterprise" section
3. If not present: Skip (uses default)
4. If present: Click "Create Key" if needed
5. Note: Site key (usually auto-configured)
```

---

## Update Your App

### Step 1: Open index.html

Find around **line 480**:

```javascript
const firebaseConfig = {
  apiKey: "YOUR_API_KEY",
  authDomain: "YOUR_PROJECT.firebaseapp.com",
  projectId: "YOUR_PROJECT",
  storageBucket: "YOUR_PROJECT.appspot.com",
  messagingSenderId: "YOUR_ID",
  appId: "YOUR_APP_ID"
};
```

### Step 2: Replace with Your Config

```javascript
const firebaseConfig = {
  apiKey: "AIzaSyD...YOUR_ACTUAL_KEY",
  authDomain: "attendance-tracker-v2.firebaseapp.com",
  projectId: "attendance-tracker-v2",
  storageBucket: "attendance-tracker-v2.appspot.com",
  messagingSenderId: "123456789",
  appId: "1:123456789:web:abc123..."
};
```

### Step 3: Save

```
Ctrl+S (or Cmd+S on Mac)
```

---

## Test Firebase Auth

### 1. Open App
```
https://officegaurab.github.io/
```

### 2. On Login Screen
- Enter: **+91 9876543210** (your test number)
- Click: "Send OTP"
- Complete: reCAPTCHA puzzle

### 3. Get OTP
Option A: Check your phone (real SMS)
Option B: Firebase Console → Users tab (test OTP)

### 4. Enter OTP
- In app: Enter 6-digit OTP
- Click: "Verify & Login"

### 5. Registration Form Appears
✅ Firebase is working!

---

## Where to Find Test OTP

### Method 1: Real Phone (Recommended)
- Send OTP to your real number
- Check SMS inbox
- Enter in app

### Method 2: Firebase Console
```
1. Firebase Console → Authentication
2. Click "Users" tab
3. Look for your phone number
4. Click it to expand
5. Find "Sign-in methods" section
6. Look for SMS OTP (if available)
```

### Method 3: During Development
- Check browser console (F12)
- Look for "reCAPTCHA verified" message
- OTP usually appears in Firebase logs

---

## Common Firebase Issues

### ❌ "reCAPTCHA error"
**Solution:**
```
1. Go: Firebase Console → Authentication
2. Check: reCAPTCHA setting
3. If empty: Let it use default
4. Refresh: Browser page
5. Try: Again
```

### ❌ "Phone number format invalid"
**Solution:**
```
Format: +CC1234567890 where:
- CC = Country code (91 for India)
- Rest = Your number

✅ Correct: +91 9876543210
❌ Wrong: 9876543210 (missing +91)
❌ Wrong: +91-9876543210 (has dash)
```

### ❌ "Firebase not initialized"
**Solution:**
```
1. Check: Firebase config is correct
2. Verify: All values are filled (no "YOUR_...")
3. Reload: Browser page
4. Check: Console (F12) for errors
```

### ❌ "Authentication is not enabled"
**Solution:**
```
1. Firebase Console → Authentication
2. Click: "Sign-in method"
3. Look for: "Phone" provider
4. If grayed out: Click to enable
5. Save: Changes
```

---

## Firebase API Keys Explained

| Key | Purpose | Safe to Expose |
|-----|---------|---|
| **apiKey** | Client-side authentication | ✅ Yes |
| **authDomain** | Firebase auth domain | ✅ Yes |
| **projectId** | Your Firebase project | ✅ Yes |
| **storageBucket** | Cloud storage | ✅ Yes |
| **messagingSenderId** | Cloud messaging | ✅ Yes |
| **appId** | App identifier | ✅ Yes |

**✅ Safe**: These keys are meant to be public in client-side code.
**❌ Unsafe**: Service account keys (never expose these!)

---

## Security Rules for Google Sheet

The current setup doesn't require rules since Firebase auth handles user verification. Each user only sees their own data via the `userId` check in Google Apps Script.

**Future Enhancement** (when you add Firestore):
```javascript
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /users/{uid} {
      allow read, write: if request.auth.uid == uid;
    }
    match /attendance/{document=**} {
      allow read, write: if request.auth != null;
    }
  }
}
```

---

## Firebase Pricing (Free Tier)

- **Authentication**: ✅ Free (10k SMS/month)
- **Storage**: ✅ 1 GB free
- **Realtime DB**: ✅ 1 GB free
- **Firestore**: ✅ 50k reads/day free

**Your usage**: Minimal (likely free forever)

---

## Next: Deploy Google Apps Script

See [SETUP_V2.md](SETUP_V2.md) for Google Apps Script deployment.

---

## Checklist

- [ ] Firebase project created
- [ ] Phone authentication enabled
- [ ] Firebase config copied
- [ ] reCAPTCHA configured
- [ ] Config pasted into index.html
- [ ] File saved
- [ ] Test login successful
- [ ] Received OTP on phone
- [ ] Registration form appeared

---

**Need Help?**
- Check: index.html line 480 for correct config
- Check: Firebase Console for errors
- Check: Browser console (F12) for logs
