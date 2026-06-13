# Quick Reference Card

## 🎯 Your Setup Checklist

### Phase 1: Dual Storage (Do This Now ✅)

- [ ] **Step 1**: Go to Google Sheet → Extensions → Apps Script
- [ ] **Step 2**: Copy code from `GoogleAppsScript.gs` → Paste in Apps Script
- [ ] **Step 3**: Click Deploy → New Deployment → Web app → Deploy
- [ ] **Step 4**: Copy deployment URL
- [ ] **Step 5**: Open app → Click ⚙️ Settings → Paste URL → Save
- [ ] **Step 6**: Test: Mark a day → Check Google Sheet appears

### Commands You'll Need

```bash
# If using command line (Git Bash, Terminal, PowerShell):
git add .
git commit -m "Add dual storage: Google Sheets + LocalStorage"
git push
```

---

## 🔗 Important Links

| Item | Link |
|------|------|
| Your Google Sheet | https://docs.google.com/spreadsheets/d/1xRWs999mHz51IGToz8gA-WjMuxAt9TB8fJXDqKv38-U/edit |
| Full Setup Guide | See `SETUP.md` |
| Firebase Guide | See `FIREBASE_SETUP.md` |

---

## 📱 User IDs

### Current (Phase 1)
- **Format**: `user_1702345678_abc123xyz`
- **Storage**: Browser localStorage
- **Scope**: Per device/browser
- **View**: Settings → Current User ID

### Future (Phase 2 - Firebase)
- **Format**: Phone number `+1234567890` or email `user@gmail.com`
- **Storage**: Firebase Authentication
- **Scope**: Multi-device same user
- **View**: Login page

---

## 💾 Data Flow

```
┌─────────────┐
│ User marks  │
│ a day       │
└──────┬──────┘
       │
       ▼
┌──────────────────────┐
│ Save to localStorage │ ✅ Instant
│ (offline works)      │
└──────┬───────────────┘
       │
       ▼
┌─────────────────────────────┐
│ Send to Google Sheet        │ ⏳ Background
│ (via Apps Script)           │
└─────────────────────────────┘
       │
       ▼
┌──────────────────────────────┐
│ Sync back on next load       │
│ (if localStorage empty)      │
└──────────────────────────────┘
```

---

## 🛠️ Troubleshooting Quick Fix

| Problem | Solution |
|---------|----------|
| "Connection failed" | Check Settings → URL starts with `https://script.google.com/` |
| Data not in Sheet | Open DevTools (F12) → Console → Look for errors |
| Empty after cache clear | Wait 2 sec on reload, check console for "synced from Google Sheet" |
| Multiple Users can't share | This is Phase 2 (Firebase) - use different device for now |

---

## 📞 What You Can Do Now

✅ Mark office/home/leave/holiday/no-show days  
✅ Sync to Google Sheet  
✅ Clear cache and reload from Sheet  
✅ Share Google Sheet with team (read data)  
✅ Export data from Google Sheet to CSV  

---

## ⏳ What Comes Next

🔜 Phase 2: Firebase Login  
🔜 Multi-user on same device  
🔜 Mobile OTP authentication  
🔜 Cross-device sync for same user  
🔜 Admin dashboard  
🔜 Export/reporting features  

---

## 📋 File Structure

```
/workspaces/officegaurab.github.io/
├── index.html                    ← Main app (open in browser)
├── GoogleAppsScript.gs           ← Deploy to Google Apps Script
├── README.md                     ← Overview & features
├── SETUP.md                      ← Step-by-step setup guide
├── FIREBASE_SETUP.md             ← Phase 2 guide
└── QUICK_REFERENCE.md            ← This file!
```

---

## 🎨 UI Changes

### New Settings Button (⚙️)
- Location: Top-right, next to month navigation
- Click to open Settings panel
- Enter Google Apps Script URL here
- View your User ID

### Settings Panel
- Configure integration
- View current User ID
- Test connection

---

## 🔒 Security Notes

- **LocalStorage**: Browser can access (same origin only)
- **Google Sheet**: Protected by Google Account permissions
- **Apps Script**: Public endpoint, but filters by User ID
- **Phase 2**: Firebase adds phone verification + auth
- **Recommendation**: Don't share personal Google Sheet link publicly

---

## 💡 Pro Tips

1. **Bookmark your app**: Pin the GitHub Pages URL
2. **Share Google Sheet**: Use Share button to invite team
3. **Export data**: In Google Sheet, use File → Download
4. **Clear data fresh start**: Delete rows in Sheet manually
5. **Check sync**: Press F12 → Console → Look for sync messages

---

## ❓ FAQ

**Q: My data is offline - is it safe?**  
A: Yes! It's stored locally until synced to Google Sheet.

**Q: Can I use on multiple devices?**  
A: Yes, but each device is separate user (Phase 2 fixes this).

**Q: Is Google Sheet required?**  
A: No - app works 100% offline. Sheet is optional backup.

**Q: Can I edit Google Sheet directly?**  
A: Not recommended - app won't see those changes. Use app only.

**Q: How do I delete a day's entry?**  
A: Click day → Click "Reset" button → Confirms delete

---

## 🚀 Launch Commands

```bash
# View files
ls -la

# View index.html content
cat index.html

# Git status
git status

# Push to GitHub Pages
git add -A && git commit -m "Update dual storage" && git push
```

---

**Version**: Phase 1 - Dual Storage Ready ✅  
**Last Updated**: Dec 2024  
**Next Phase**: Firebase OTP Auth (Phase 2)
