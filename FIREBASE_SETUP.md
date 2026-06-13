# Phase 2: Firebase Mobile OTP Authentication

This guide outlines how to integrate Firebase Phone Authentication for multi-user support.

---

## Architecture Overview

### Current (Phase 1)
```
Browser → LocalStorage + Google Sheet
  └─ Device-based ID per browser
```

### Next (Phase 2)
```
Mobile/Web → Firebase Auth (Phone OTP)
  └─ Cloud: User Profile (email, phone, role)
  └─ Google Sheet (synced by email)
  └─ LocalStorage (fallback for offline)
```

---

## Step 1: Firebase Project Setup

### 1.1 Create Firebase Project
1. Go to [Firebase Console](https://console.firebase.google.com/)
2. Click **Add Project**
3. Enter project name (e.g., "attendance-tracker")
4. Select region, accept terms
5. Create project

### 1.2 Enable Phone Authentication
1. In Firebase Console: **Authentication**
2. Click **Sign-in method** tab
3. Enable **Phone**
4. Add your own phone number for testing
5. Save

### 1.3 Get Firebase Config
1. In Firebase Console: **Project Settings** (gear icon)
2. Copy Web config:
```javascript
const firebaseConfig = {
  apiKey: "YOUR_API_KEY",
  authDomain: "your-project.firebaseapp.com",
  projectId: "your-project-id",
  storageBucket: "your-project.appspot.com",
  messagingSenderId: "123456789",
  appId: "1:123456789:web:abcdef1234567890"
};
```

---

## Step 2: Update App for Phone Auth

### 2.1 Add Firebase SDK

Replace the script section in index.html:

```html
<!-- Add after existing <script> tags, before custom script -->

<!-- Firebase SDK -->
<script src="https://www.gstatic.com/firebasejs/10.7.0/firebase-app.js"></script>
<script src="https://www.gstatic.com/firebasejs/10.7.0/firebase-auth.js"></script>

<script>
  // Initialize Firebase (add your config)
  const firebaseConfig = {
    // Paste your config here
  };
  
  firebase.initializeApp(firebaseConfig);
  const auth = firebase.auth();
</script>
```

### 2.2 Add Login UI

Add this HTML (replace the settings sheet):

```html
<div id="loginSheet" class="sheet show">
  <div class="sheet-title">Login</div>
  
  <div id="phoneInputSection">
    <div style="margin:10px 0;font-size:13px">
      <label style="display:block;margin-bottom:5px">Phone Number:</label>
      <input type="tel" id="phoneInput" placeholder="+1234567890" 
             style="width:100%;padding:8px;border:1px solid #ccc;border-radius:6px">
    </div>
    
    <button class="action" style="background:#2196f3;color:white" 
            onclick="sendOTP()">
      Send OTP
    </button>
  </div>
  
  <div id="otpInputSection" style="display:none">
    <div style="margin:10px 0;font-size:13px">
      <label style="display:block;margin-bottom:5px">Enter OTP:</label>
      <input type="text" id="otpInput" placeholder="123456" 
             style="width:100%;padding:8px;border:1px solid #ccc;border-radius:6px">
    </div>
    
    <div id="recaptcha"></div>
    
    <button class="action" style="background:#4caf50;color:white" 
            onclick="verifyOTP()">
      Verify & Login
    </button>
    
    <button class="action reset-btn" onclick="resetLogin()">
      Change Phone
    </button>
  </div>
</div>
```

### 2.3 Add Auth Functions

```javascript
let confirmationResult = null;

// Hide calendar until logged in
function checkAuth() {
  firebase.auth().onAuthStateChanged(user => {
    if (user) {
      userId = user.email || user.phoneNumber;
      localStorage.setItem('userId', userId);
      document.getElementById('loginSheet').classList.remove('show');
      document.getElementById('backdrop').classList.remove('show');
      render();
    } else {
      document.getElementById('loginSheet').classList.add('show');
    }
  });
}

function sendOTP() {
  const phone = document.getElementById('phoneInput').value;
  
  if (!phone) {
    alert('Enter phone number');
    return;
  }
  
  // Setup reCAPTCHA
  window.recaptchaVerifier = new firebase.auth.RecaptchaVerifier('recaptcha', {
    'size': 'normal',
    'callback': onReCaptchaSuccess
  });
  
  auth.signInWithPhoneNumber(phone, window.recaptchaVerifier)
    .then(result => {
      confirmationResult = result;
      document.getElementById('phoneInputSection').style.display = 'none';
      document.getElementById('otpInputSection').style.display = 'block';
    })
    .catch(error => {
      alert('Error: ' + error.message);
      window.recaptchaVerifier.clear();
    });
}

function verifyOTP() {
  const otp = document.getElementById('otpInput').value;
  
  confirmationResult.confirm(otp)
    .then(result => {
      // User successfully signed in
      userId = result.user.phoneNumber;
      syncFromGoogleSheet(); // Sync data for this user
    })
    .catch(error => {
      alert('Invalid OTP: ' + error.message);
    });
}

function resetLogin() {
  confirmationResult = null;
  document.getElementById('phoneInputSection').style.display = 'block';
  document.getElementById('otpInputSection').style.display = 'none';
  document.getElementById('phoneInput').value = '';
  document.getElementById('otpInput').value = '';
  if (window.recaptchaVerifier) {
    window.recaptchaVerifier.clear();
  }
}

// Call on page load
window.addEventListener('load', () => {
  checkAuth();
});
```

---

## Step 3: Update Google Apps Script

Update the `getDataFromSheet()` function to support multi-user:

```javascript
function getDataFromSheet(userId) {
  try {
    const sheet = getSheet();
    const values = sheet.getRange('A:C').getValues();
    
    // Convert to object, filtering by userId
    const data = {};
    
    for (let i = 1; i < values.length; i++) {
      const row = values[i];
      
      // Match by phone number or email
      if ((row[1] === userId || row[1].includes(userId.replace('+', ''))) 
          && row[0]) {
        data[row[0]] = row[2];
      }
    }
    
    Logger.log('Data retrieved for: ' + userId);
    return formatResponse(true, data);
  } catch (error) {
    return formatResponse(false, error.toString());
  }
}
```

---

## Step 4: Test Multi-User Scenario

### Device 1:
1. Login with Phone A (+1234567890)
2. Mark days as "office"
3. Check Google Sheet

### Device 2:
1. Login with Phone B (+9876543210)
2. Should see empty calendar
3. Mark different days
4. Check Google Sheet (both users' data visible)

### Back to Device 1:
1. Logout and login with Phone B
2. Should see Device 2's data

---

## Data Structure After Phase 2

### Google Sheet (updated)
```
Date           | User                  | Status
2024-12-25     | +1234567890          | office
2024-12-26     | +1234567890          | home
2024-12-25     | +9876543210          | leave
2024-12-27     | +9876543210          | office
```

### Firebase Auth
```
User: +1234567890
├─ email: user1@gmail.com (optional)
├─ phone: +1234567890
└─ createdAt: 2024-12-01

User: +9876543210
├─ email: user2@gmail.com
├─ phone: +9876543210
└─ createdAt: 2024-12-05
```

---

## Security Considerations

1. **reCAPTCHA**: Prevents OTP spam
2. **Phone verification**: Firebase handles automatically
3. **Rate limits**: Firebase limits OTP attempts
4. **Rules**: Add Firestore rules if storing user profiles

### Example Firestore Rules (for user profiles):
```javascript
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /users/{uid} {
      allow read, write: if request.auth.uid == uid;
    }
  }
}
```

---

## Optional: Add User Profiles to Firestore

For storing user metadata (name, role, etc.):

```javascript
// After successful login
firebase.firestore().collection('users').doc(user.phoneNumber).set({
  phone: user.phoneNumber,
  email: user.email,
  name: user.displayName,
  createdAt: firebase.firestore.FieldValue.serverTimestamp(),
  role: 'employee' // admin, manager, employee
}, { merge: true });
```

---

## Testing Checklist

- [ ] Firebase project created
- [ ] Phone auth enabled
- [ ] Firebase config added to app
- [ ] Login form works
- [ ] OTP sent successfully
- [ ] OTP verification works
- [ ] Data syncs to Google Sheet by phone
- [ ] Multi-user data isolation working
- [ ] Logout functionality works
- [ ] Offline access works (localStorage)

---

## Timeline Estimate

- **Setup Firebase**: 15-20 minutes
- **Add Auth UI**: 30-45 minutes
- **Test & Debug**: 30 minutes
- **Total**: ~2 hours

---

## Support

When ready to implement Phase 2, ask for:
1. Help integrating Firebase code
2. Debugging auth issues
3. Adding Firestore for user profiles
4. Setting up admin dashboard
