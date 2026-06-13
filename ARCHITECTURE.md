# 🏗️ V2 Architecture & Data Flow

## System Architecture Diagram

```
┌─────────────────────────────────────────────────────────────────┐
│                        USER (Browser)                           │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │              Attendance Tracker App                     │    │
│  │  ┌────────────────────────────────────────────────┐    │    │
│  │  │  Header: Logo | Profile Button | Menu Button │    │    │
│  │  └────────────────────────────────────────────────┘    │    │
│  │  ┌────────────────┐          ┌──────────────────┐      │    │
│  │  │   Calendar     │          │  Right Drawer   │      │    │
│  │  │  (Attendance)  │          │  • Profile      │      │    │
│  │  │  - Mark days   │          │  • Settings     │      │    │
│  │  │  - View stats  │          │  • Logout       │      │    │
│  │  │  - Edit        │          │  • Help         │      │    │
│  │  └────────────────┘          └──────────────────┘      │    │
│  │                                                         │    │
│  └─────────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────────┘
           │                                    │
           │ (JavaScript fetch)                 │
           │                                    │
  ┌────────▼─────────────────────────────────────────────────┐
  │        Firebase Authentication                           │
  │  ┌──────────────────────────────────────────────────┐   │
  │  │  Phone OTP Login                                │   │
  │  │  1. User enters phone                           │   │
  │  │  2. Firebase sends OTP via SMS                  │   │
  │  │  3. User enters OTP                             │   │
  │  │  4. Firebase verifies                           │   │
  │  │  5. User authenticated                          │   │
  │  └──────────────────────────────────────────────────┘   │
  └────────┬─────────────────────────────────────────────────┘
           │
           │ (Authenticated User ID = Phone Number)
           │
  ┌────────▼──────────────────────────────────────────────┐
  │     Google Apps Script (Backend)                      │
  │  ┌──────────────────────────────────────────────┐   │
  │  │  Receives: POST request with action         │   │
  │  │  • saveAttendance()                          │   │
  │  │  • getAttendance()                           │   │
  │  │  • saveUser()                                │   │
  │  │  • checkSaveLimit()                          │   │
  │  │                                              │   │
  │  │  Validates: User ID, daily saves             │   │
  │  │  Processes: Data transformation              │   │
  │  │  Returns: JSON response                      │   │
  │  └──────────────────────────────────────────────┘   │
  └────────┬──────────────────────────────────────────────┘
           │
  ┌────────▼──────────────────────────────────────────────┐
  │    Google Sheet Database (3 Tabs)                    │
  │  ┌──────────────────────────────────────────────┐   │
  │  │ Tab 1: Attendance                            │   │
  │  │  Date | User | Status                        │   │
  │  │  ────────────────────────────────────────── │   │
  │  │  2024-06-13 | +91 98765 4321 | office       │   │
  │  │  2024-06-12 | +91 98765 4321 | home         │   │
  │  └──────────────────────────────────────────────┘   │
  │  ┌──────────────────────────────────────────────┐   │
  │  │ Tab 2: Users                                 │   │
  │  │  UserID | Mobile | Name | Email | Company   │   │
  │  │  ────────────────────────────────────────── │   │
  │  │  +919876... | +919876... | John | j@e.com  │   │
  │  └──────────────────────────────────────────────┘   │
  │  ┌──────────────────────────────────────────────┐   │
  │  │ Tab 3: SaveLogs                              │   │
  │  │  UserID | Date | Count                       │   │
  │  │  ────────────────────────────────────────── │   │
  │  │  +919876... | 06/13/2024 | 3                │   │
  │  └──────────────────────────────────────────────┘   │
  │                                                      │
  │  ✅ Real-time updates                               │
  │  ✅ Permanent storage                               │
  │  ✅ Multi-user support                              │
  │  ✅ Audit trail (SaveLogs)                          │
  └──────────────────────────────────────────────────────┘
```

---

## User Journey Flow

```
START
  │
  ├─► First Time User?
  │   ├─► NO  → Authenticated (Firebase)
  │   │          ├─► Existing User Found?
  │   │          │   ├─► YES → Load Profile & Attendance
  │   │          │   └─► NO → Show Registration Form
  │   │          │
  │   │          └─► User Logs In
  │   │
  │   └─► YES → Login Page
  │       ├─► Enter Phone
  │       ├─► Receive OTP
  │       ├─► Enter OTP
  │       ├─► Verify (Firebase)
  │       └─► Show Registration Form
  │
  ├─► Registration Form
  │   ├─► Fill: Name, Email, Company, Hours
  │   ├─► Upload: Profile Photo (optional)
  │   ├─► Save: To Google Sheet (Users tab)
  │   └─► Continue
  │
  ├─► Main App
  │   ├─► Load: Attendance Data (from Google Sheet)
  │   ├─► Display: Calendar with Statistics
  │   └─► Ready: For user input
  │
  ├─► User Clicks Day
  │   ├─► Check: Daily Save Limit
  │   │   ├─► Limit Reached? → Show Warning
  │   │   └─► Under Limit? → Show Options
  │   │
  │   ├─► User Selects Status
  │   │   (Office/Home/Holiday/Leave/No-Show)
  │   │
  │   ├─► Save: To Google Sheet (Attendance tab)
  │   ├─► Update: SaveLogs (increment counter)
  │   ├─► Refresh: Calendar UI
  │   └─► Show: Updated Stats
  │
  ├─► User Profile Button
  │   ├─► Display: User Info + Photo
  │   ├─► Allow: Edit All Fields
  │   ├─► Allow: Upload New Photo
  │   ├─► Save: Changes to Google Sheet
  │   └─► Continue: Using app
  │
  ├─► User Menu
  │   ├─► Profile → Edit Profile
  │   ├─► Settings → Configure App
  │   ├─► Logout → Firebase Sign Out
  │   └─► Help → Show Info
  │
  └─► END
```

---

## Data Flow: Marking Attendance

```
┌──────────────────┐
│ User Clicks Day  │
└────────┬─────────┘
         │
         ▼
┌──────────────────────────────────────┐
│ Check Daily Save Limit               │
│ Query: SaveLogs sheet                │
│ Filter: UserID = CurrentUser         │
│ Filter: Date = Today                 │
└────────┬─────────────────────────────┘
         │
         ├─► Count < 3? ───┐
         │                 │
         │                 ▼
         │         ┌──────────────────┐
         │         │ Show Status Menu │
         │         │ (Office/Home/...)│
         │         └────────┬─────────┘
         │                  │
         │                  ▼
         │         ┌──────────────────────────────────┐
         │         │ User Selects Status              │
         │         │ (e.g., "office")                 │
         │         └────────┬─────────────────────────┘
         │                  │
         │                  ▼
         │         ┌─────────────────────────────────────────┐
         │         │ Apps Script: saveAttendance()           │
         │         │ Params:                                 │
         │         │  - userId: "+91 98765 4321"             │
         │         │  - date: "2024-06-13"                   │
         │         │  - status: "office"                     │
         │         └────────┬────────────────────────────────┘
         │                  │
         │                  ▼
         │         ┌──────────────────────────────────────┐
         │         │ Write to Attendance Sheet            │
         │         │ Row: [2024-06-13|+919876...|office] │
         │         └────────┬─────────────────────────────┘
         │                  │
         │                  ▼
         │         ┌──────────────────────────────────────┐
         │         │ Update SaveLogs Sheet                │
         │         │ Increment count for today            │
         │         └────────┬─────────────────────────────┘
         │                  │
         │                  ▼
         │         ┌──────────────────────────────────────┐
         │         │ Return Success Response              │
         │         │ {success: true}                      │
         │         └────────┬─────────────────────────────┘
         │                  │
         │                  ▼
         │         ┌──────────────────────────────────────┐
         │         │ Browser: Refresh Calendar             │
         │         │ Reload data from Google Sheet          │
         │         │ Update UI & Statistics                │
         │         └──────────────────────────────────────┘
         │
         └─► Count >= 3? ────┐
                             │
                             ▼
                   ┌──────────────────────────┐
                   │ Show Error: Limit Reached│
                   │ Message: "3/3 saves used"│
                   │ "Try again tomorrow"     │
                   └──────────────────────────┘
```

---

## Profile Update Flow

```
┌─────────────────────────────┐
│ User Clicks Profile Button  │
└────────┬────────────────────┘
         │
         ▼
┌──────────────────────────────────────┐
│ Load Current User Data               │
│ Query: Users sheet                   │
│ Filter: UserID = CurrentUser         │
└────────┬─────────────────────────────┘
         │
         ▼
┌──────────────────────────────────────┐
│ Display Profile Sheet                │
│ Fields:                              │
│  - Name (editable)                   │
│  - Email (editable)                  │
│  - Company (editable)                │
│  - Office Hours In (dropdown)         │
│  - Office Hours Out (dropdown)        │
│  - Profile Photo (upload)             │
└────────┬─────────────────────────────┘
         │
         ├─► User Edit Fields ──┐
         │                      │
         │                      ▼
         │             ┌─────────────────────┐
         │             │ User Uploads Photo  │
         │             │ (FileReader API)    │
         │             │ Convert to Base64   │
         │             └────────┬────────────┘
         │                      │
         │   ┌──────────────────┘
         │   │
         ▼   ▼
┌────────────────────────────────────────┐
│ User Clicks: "Save Changes"            │
└────────┬─────────────────────────────────┘
         │
         ▼
┌──────────────────────────────────────────┐
│ Apps Script: saveUser()                  │
│ Params:                                  │
│  - userId: "+91 98765 4321"              │
│  - userData:                             │
│    {name, email, company, inTime,        │
│     outTime, profilePhoto}               │
└────────┬─────────────────────────────────┘
         │
         ▼
┌──────────────────────────────────────────┐
│ Update Users Sheet                       │
│ Find & Update Row for this UserID        │
│ All columns: Name, Email, Hours, Photo   │
└────────┬─────────────────────────────────┘
         │
         ▼
┌──────────────────────────────────────────┐
│ Return Success Response                  │
│ {success: true}                          │
└────────┬─────────────────────────────────┘
         │
         ▼
┌──────────────────────────────────────────┐
│ Browser: Update UI                       │
│ Show Alert: "Profile Updated!"           │
│ Close Profile Sheet                      │
│ Ready: For next action                   │
└──────────────────────────────────────────┘
```

---

## Save Limit: Daily Reset Logic

```
┌─────────────────────────────────┐
│ Each Day at Midnight (00:00)    │
└────────┬────────────────────────┘
         │
         ▼
┌────────────────────────────────────┐
│ SaveLogs Sheet                     │
│ All entries with today's date      │
│ get automatically archived         │
│ (Google Sheet handles via formula  │
│  or Apps Script scheduled trigger) │
└────────┬────────────────────────────┘
         │
         ▼
┌───────────────────────────────────┐
│ New day starts                    │
│ SaveLogs for today: [0]           │
│ User can save 3 times             │
└────────┬────────────────────────────┘
         │
         ├─► Save 1: count = 1 ──────┐
         │                           │
         ├─► Save 2: count = 2 ──────┤
         │                           │
         ├─► Save 3: count = 3 ──────┤─► User Limit Reached
         │                           │   "3/3 saves used"
         ├─► Save 4: BLOCKED ────────┘   "Try again tomorrow"
         │
         └─► Midnight Next Day: Reset to 0
```

---

## Technology Stack

```
Frontend:
├─ HTML5
├─ CSS3 (Grid, Flexbox)
├─ Vanilla JavaScript (ES6+)
└─ Firebase Auth SDK

Backend:
├─ Google Apps Script
├─ Google Sheets API
└─ JSON request/response

Database:
├─ Google Sheets (Cloud)
└─ 3 Tabs: Attendance | Users | SaveLogs

Services:
├─ Firebase Authentication
├─ Google Cloud Console
└─ GitHub Pages (hosting)

Security:
├─ Firebase Phone Verification
├─ HTTPS/TLS
└─ User-level data isolation
```

---

## API Endpoints (Google Apps Script)

```
Endpoint: [Deployment URL]
Method: POST
Content-Type: application/json

ACTION: saveAttendance
Request:
{
  action: "saveAttendance",
  userId: "+91 98765 4321",
  date: "2024-06-13",
  status: "office"
}
Response:
{
  success: true,
  data: "Attendance saved",
  timestamp: "2024-06-13T10:30:00Z"
}

─────────────────────────────────

ACTION: getAttendance
Request:
{
  action: "getAttendance",
  userId: "+91 98765 4321"
}
Response:
{
  success: true,
  data: {
    "2024-06-13": "office",
    "2024-06-12": "home",
    "2024-06-11": "holiday"
  },
  timestamp: "2024-06-13T10:30:00Z"
}

─────────────────────────────────

ACTION: saveUser
Request:
{
  action: "saveUser",
  userId: "+91 98765 4321",
  userData: {
    mobile: "+91 98765 4321",
    name: "John",
    email: "john@email.com",
    company: "Acme",
    inTime: "9",
    outTime: "18",
    profilePhoto: "data:image/..."
  }
}
Response:
{
  success: true,
  data: "User profile saved",
  timestamp: "2024-06-13T10:30:00Z"
}

─────────────────────────────────

ACTION: checkSaveLimit
Request:
{
  action: "checkSaveLimit",
  userId: "+91 98765 4321"
}
Response:
{
  success: true,
  data: {
    allowed: true,
    count: 2,
    limit: 3,
    remaining: 1
  },
  timestamp: "2024-06-13T10:30:00Z"
}
```

---

## Deployment Architecture

```
GitHub Repository
└─ officegaurab.github.io
   ├─ index.html (Web App)
   ├─ GoogleAppsScript.gs (Backend Spec)
   ├─ Documentation (MD files)
   └─ .git/

                    │
                    │ (Hosted on)
                    ▼
         GitHub Pages HTTPS
         https://officegaurab.github.io/

                    │
         ┌──────────┼──────────┐
         │          │          │
         ▼          ▼          ▼
    Firebase    Google        User
    Auth        Sheet         Browser
    Service     Database      (Static)
```

---

## Scalability Matrix

```
Current Setup (V2):
├─ Users: ∞ (unlimited)
├─ Attendance Records: ∞ (unlimited)
├─ Google Sheet Rows: 5 million max (enough for 1000s of users for years)
├─ Firebase SMS: 10k/month free (enough for signup + recovery)
└─ Concurrent Users: Limited by Google Sheet API rate limits

Growth Path (V3+):
├─ Add: Firestore (scalable NoSQL)
├─ Add: Cloud Functions (serverless compute)
├─ Add: Cloud Storage (for photos/reports)
├─ Add: Admin Dashboard (React)
├─ Add: Mobile App (React Native/Flutter)
└─ Add: Analytics (BigQuery integration)
```

---

## Error Handling Flow

```
USER ACTION
     │
     ▼
┌─────────────────────┐
│ JavaScript Execute  │
└──────────┬──────────┘
           │
           ├─► Validation Error?
           │   ├─► YES: Show Alert
           │   └─► NO: Continue
           │
           ▼
┌─────────────────────────┐
│ Fetch Google Apps       │
│ Script Endpoint         │
└──────────┬──────────────┘
           │
           ├─► Network Error?
           │   ├─► YES: Show "Connection Failed"
           │   └─► NO: Continue
           │
           ▼
┌─────────────────────────┐
│ Apps Script Processes   │
│ Request                 │
└──────────┬──────────────┘
           │
           ├─► Server Error?
           │   ├─► YES: Return error in JSON
           │   └─► NO: Return success
           │
           ▼
┌─────────────────────────┐
│ Browser Receives        │
│ Response                │
└──────────┬──────────────┘
           │
           ├─► Success?
           │   ├─► YES: Update UI
           │   └─► NO: Show Error Alert
           │
           ▼
        DONE
```

---

This architecture is designed to be:
- **Scalable**: Handles 1000s of users
- **Reliable**: Uses proven Google services
- **Secure**: Firebase authentication
- **Simple**: Vanilla JavaScript, no frameworks
- **Maintainable**: Well-documented code

---

**Architecture Version**: V2  
**Last Updated**: June 2024  
**Status**: Production Ready ✅
