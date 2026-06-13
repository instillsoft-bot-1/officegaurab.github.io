# Office Attendance Tracker - Setup Guide

## Overview
Your app now has **dual storage**: 
- **Local Storage** (primary) - Fast, offline access
- **Google Sheets** (backup) - Cloud sync, shareable data
- **Flow**: Save to both → Load from localStorage first → Fallback to Google Sheet

---

## Phase 1: Current Implementation (Dual Storage)

### Step 1: Set Up Google Apps Script

1. **Open your Google Sheet**: https://docs.google.com/spreadsheets/d/1xRWs999mHz51IGToz8gA-WjMuxAt9TB8fJXDqKv38-U/

2. **Create Apps Script**:
   - Click **Extensions** → **Apps Script**
   - Delete any default code
   - Copy entire code from `GoogleAppsScript.gs` file
   - Paste it into the editor
   - Save with Ctrl+S

3. **Deploy as Web App**:
   - Click **Deploy** button (top right)
   - Select **New Deployment**
   - Click gear icon → Choose **Web app**
   - Settings:
     - **Execute as**: Your account
     - **Who has access**: Anyone
   - Click **Deploy**

4. **Copy Deployment URL**:
   - You'll see a modal with the deployment URL
   - Example: `https://script.google.com/macros/s/ABC123xyz/usercontent`
   - Copy this URL

### Step 2: Configure Your App

1. **Open the tracker app** (index.html)
2. **Click Settings** (⚙️ button, top right)
3. **Paste the Google Apps Script URL** in the input field
4. **Click Save Settings**
5. You should see "Data synced from Google Sheet" in browser console

### Step 3: Test It

1. **Mark a day** in the calendar
2. **Check Google Sheet** - you should see the entry after a moment
3. **Clear localStorage** (DevTools → Application → Clear all)
4. **Refresh page** - data should reload from Google Sheet
5. ✅ Dual storage is working!

---

## Data Structure

### Local Storage (localStorage)
```javascript
{
  "2024-12-25": "office",
  "2024-12-26": "home",
  "2024-12-27": "leave",
  ...
}
```

### Google Sheet (Sheet1)
```
Date           | User                  | Status
2024-12-25     | user_1234567_abc123   | office
2024-12-26     | user_1234567_abc123   | home
2024-12-27     | user_1234567_abc123   | leave
```

---

## User Identification (Current)

- **Device-based ID**: Each browser gets a unique ID (e.g., `user_1702345678_xyz789`)
- **Stored in**: localStorage as `userId`
- **Can view**: Settings → Current User ID
- **Limitation**: Each device/browser has separate data

---

## Phase 2: Multi-User with Firebase (Next Phase)

### Implementation Plan:

1. **Firebase Authentication**:
   - Mobile login with OTP
   - Firebase Phone Authentication
   - Replace device ID with `user@email.com`

2. **Data Structure Update**:
   - Keep Google Sheet: Date | User (email) | Status
   - Add user profile: email, phone, name
   - App queries data by logged-in email

3. **Multi-Device Sync**:
   - All user's devices show same data
   - Cloud-first approach with localStorage fallback

4. **Integration Steps**:
   - Add Firebase SDK
   - Create login form with OTP
   - Store user email in localStorage (after login)
   - Use email instead of device ID in sync functions

### Firebase Setup (when ready):
```javascript
// Will replace userId generation
firebase.auth().signInWithPhoneNumber(phoneNumber, recaptchaVerifier)
  .then(confirmationResult => {
    // User enters OTP
    confirmationResult.confirm(otp).then(result => {
      userId = result.user.email; // Use email as user ID
    });
  });
```

---

## Troubleshooting

### "Connection Failed" or blank stats
- Check Google Apps Script URL in Settings
- Verify URL format: `https://script.google.com/macros/s/`
- Check browser console for errors

### Data not syncing to Sheet
- Open browser DevTools (F12)
- Go to Network tab
- Try changing a day
- Look for POST request to script URL
- Check response for errors

### Sheet shows wrong data
- Clear sheet manually and start fresh
- Or use new `user_ID` (clear localStorage `userId`)

---

## Files

- **index.html** - Main app with dual storage logic
- **GoogleAppsScript.gs** - Backend that handles Sheet I/O
- **SETUP.md** - This file

---

## Key Functions

### In App:
- `loadData()` - Loads from localStorage, syncs from Sheet if empty
- `saveData(data)` - Saves to both localStorage + Sheet
- `syncToGoogleSheet()` - Sends data to Apps Script
- `syncFromGoogleSheet()` - Retrieves data from Sheet

### In Apps Script:
- `doPost(e)` - Handles incoming requests
- `saveDataToSheet()` - Writes/updates Sheet rows
- `getDataFromSheet()` - Retrieves user's data

---

## Next Steps

1. ✅ Deploy Google Apps Script
2. ✅ Test dual storage
3. Plan Firebase integration (Phase 2)
4. Consider: User roles, permissions, sharing features
