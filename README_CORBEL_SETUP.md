# Corbel - Facebook Ad Comment Monitoring Setup
## Complete Documentation Package

**Client:** Corbel
**Setup Date:** 2025-11-20
**Created By:** Claude (Static Shift)

---

## What This Is

This documentation package contains everything you need to set up Facebook Ad Comment Monitoring for Corbel (or any new client). The system monitors Facebook ads for new comments and sends real-time Slack notifications.

---

## The Problem You're Solving

You need to duplicate the existing HR Advice Online ad comment monitoring workflow for a new client (Corbel), but encountered issues understanding how to get Page Access Tokens from Meta Developer Portal.

**The confusion:** The Meta Developer Portal doesn't have a field where you manually enter page access tokens. Instead, tokens are generated via API calls using the Graph API Explorer.

---

## Documentation Overview

### Start Here

**1. QUICK_START_CHECKLIST.md** ⭐ START HERE
- 30-minute setup checklist
- Step-by-step with checkboxes
- Covers entire process from app creation to activation
- **Use this if:** You want to get started immediately

### Deep Dive Guides

**2. META_APP_SETUP_GUIDE.md**
- Comprehensive setup guide (full detail)
- All steps explained with context
- Includes prerequisites, testing, troubleshooting
- **Use this if:** You want to understand every step thoroughly

**3. PAGE_ACCESS_TOKEN_GUIDE.md** ⭐ KEY DOCUMENT
- Solves the "can't add page access token" problem
- Explains the confusion around token generation
- Visual flow diagrams
- Common mistakes explained
- **Use this if:** You're stuck on getting page access tokens

### Templates & Reference

**4. CORBEL_SETUP_TEMPLATE.md**
- Fill-in-the-blank template
- Record all your tokens, IDs, and settings
- Testing checklist
- Maintenance schedule
- **Use this as:** A reference document you fill out during setup

**5. TROUBLESHOOTING_FAQ.md**
- Common errors and solutions
- API error codes explained
- n8n workflow debugging
- Testing tips
- **Use this when:** Something goes wrong

---

## Quick Reference

### What You Need

**Before starting, gather:**
- [ ] Meta Business Manager account access
- [ ] Corbel Facebook Page (you must be admin)
- [ ] Corbel Ad Account ID: `act_________________`
- [ ] Access to n8n workflow instance
- [ ] Slack workspace access

### The Two Critical Tokens

**1. Page Access Token** (for reading comments)
- Generated via Graph API Explorer
- Call `/me/accounts` to get it
- Used in: "Get Comments for Ad Post" node

**2. Ad Account Access Token** (for reading ads)
- Generated via Graph API Explorer
- Same as your user token with `ads_read` permission
- Used in: "Get Active Ads" node

---

## Setup Process (High Level)

```
1. Create Meta App for Corbel
          ↓
2. Get Page Access Token (via /me/accounts)
          ↓
3. Get Ad Account Access Token (via Graph API Explorer)
          ↓
4. Duplicate n8n workflow
          ↓
5. Update tokens in workflow nodes
          ↓
6. Update client config (ad account ID, Slack channel)
          ↓
7. Test each node
          ↓
8. Activate workflow
          ↓
9. Monitor for 24 hours
```

**Total time:** 30-45 minutes

---

## The Key Insight (What You Were Missing)

### The Confusion

Looking at the Meta Developer Portal, you expected to find:
- A "Tokens" section
- An input field to paste or generate page tokens
- A button that says "Add Page Access Token"

**But this doesn't exist!**

### The Reality

Page Access Tokens are NOT configured in the Meta App settings. They are:
1. Generated dynamically via API call (`/me/accounts`)
2. Returned in the API response
3. Copied by you and stored in your n8n workflow
4. Never entered into the Meta Developer Portal

### What That Screenshot Shows

The screenshot you shared shows the **Page selection interface** - this is for:
- Choosing which pages your app CAN access
- Managing page permissions
- NOT for entering or generating tokens

---

## Step-by-Step Token Generation (Critical Section)

### For Page Access Token:

1. Open Graph API Explorer: https://developers.facebook.com/tools/explorer/
2. Select your Corbel app from dropdown
3. Click "Generate Access Token"
4. Log in as Corbel Page Admin
5. Grant permissions (especially `pages_read_engagement`)
6. In query field, type: `me/accounts`
7. Click Submit
8. Find "Corbel" in the response
9. Copy the `access_token` value (this is your Page token!)
10. Paste into n8n "Get Comments for Ad Post" node

### For Ad Account Access Token:

1. Same Graph API Explorer
2. Same app selected
3. Click "Generate Access Token" (or reuse existing)
4. Ensure `ads_read` permission granted
5. Test with: `act_XXXXX/ads?limit=1`
6. If successful, copy the token shown in the "Access Token" field
7. Paste into n8n "Get Active Ads" node

---

## Workflow Nodes to Update

When you duplicate the workflow, update these nodes:

### Node: "Get Active Ads"
- **URL:** `https://graph.facebook.com/v24.0/act_XXXXX/ads`
  - Replace `act_XXXXX` with Corbel's Ad Account ID
- **access_token:** Paste Ad Account Access Token

### Node: "Get Comments for Ad Post"
- **access_token:** Paste Page Access Token
- URL stays dynamic (no change needed)

### Node: "Process Comment Data"
```javascript
const clients = {
  'act_XXXXX': {  // Corbel's ad account ID
    name: 'Corbel',
    channel: '#notifications-corbel',
    priority: 'high'
  }
};

const adAccountId = 'act_XXXXX';  // Same ad account ID
```

### Slack Nodes (all 3)
- Update channel to: `#notifications-corbel`

---

## Testing Checklist

- [ ] Test "Get Active Ads" - should return ad list
- [ ] Check ads have `creative.effective_object_story_id`
- [ ] Test "Get Comments" with actual post (if available)
- [ ] Make a test comment on an ad
- [ ] Run workflow manually
- [ ] Verify Slack notification appears
- [ ] Check Google Sheet updated (if using)
- [ ] Activate workflow
- [ ] Wait 15 minutes, verify automatic run

---

## Common Mistakes to Avoid

1. ❌ Using User token for comments (wrong!)
   - ✅ Use Page token from `/me/accounts` response

2. ❌ Looking for token input in Meta Developer Portal
   - ✅ Generate tokens via Graph API Explorer

3. ❌ Not being logged in as Page Admin when generating tokens
   - ✅ Must be Page Admin for the specific page

4. ❌ Forgetting to update Ad Account ID in multiple places
   - ✅ Update in: URL, client config, and adAccountId variable

5. ❌ Using wrong token in wrong node
   - ✅ Page token for comments, Ad Account token for ads

---

## When to Use Each Document

| Situation | Document to Use |
|-----------|----------------|
| First time setup | QUICK_START_CHECKLIST.md |
| Can't get page token | PAGE_ACCESS_TOKEN_GUIDE.md |
| Want full context | META_APP_SETUP_GUIDE.md |
| Need to record info | CORBEL_SETUP_TEMPLATE.md |
| Something broke | TROUBLESHOOTING_FAQ.md |
| Future client setup | All documents (reusable!) |

---

## Success Criteria

You'll know it's working when:
- [x] Workflow runs every 15 minutes without errors
- [x] New comments trigger Slack notifications within 15 minutes
- [x] Comments are logged to Google Sheet (if configured)
- [x] Daily summary arrives at 5 PM on weekdays
- [x] No error messages in #notifications-corbel

---

## Support & Resources

**Your Documentation:**
- `META_APP_SETUP_GUIDE.md` - Complete setup guide
- `PAGE_ACCESS_TOKEN_GUIDE.md` - Token generation help
- `QUICK_START_CHECKLIST.md` - Quick setup checklist
- `CORBEL_SETUP_TEMPLATE.md` - Template to fill in
- `TROUBLESHOOTING_FAQ.md` - Problem solving

**Meta Resources:**
- Graph API Explorer: https://developers.facebook.com/tools/explorer/
- Access Token Debugger: https://developers.facebook.com/tools/debug/accesstoken/
- Meta Developers Docs: https://developers.facebook.com/docs/

**n8n Workflow:**
- Original: "Facebook AD Comment Monitor FINAL | HRAO"
- New: "Facebook AD Comment Monitor | Corbel"

---

## Next Steps

1. **Read:** QUICK_START_CHECKLIST.md
2. **Have open:** PAGE_ACCESS_TOKEN_GUIDE.md (for token generation)
3. **Fill out:** CORBEL_SETUP_TEMPLATE.md (as you go)
4. **Reference:** TROUBLESHOOTING_FAQ.md (if issues arise)

---

## Reusability

This entire documentation package can be reused for future clients!

**To set up another client:**
1. Follow QUICK_START_CHECKLIST.md again
2. Replace "Corbel" with new client name
3. Get new tokens for new client's pages/ad accounts
4. Duplicate workflow again
5. Update configurations

**Time for subsequent setups:** ~20-30 minutes (faster than first time)

---

## Files in This Package

```
README_CORBEL_SETUP.md          ← You are here (start)
QUICK_START_CHECKLIST.md        ← 30-min setup guide
META_APP_SETUP_GUIDE.md         ← Complete detailed guide
PAGE_ACCESS_TOKEN_GUIDE.md      ← Solve token confusion
CORBEL_SETUP_TEMPLATE.md        ← Fill-in template
TROUBLESHOOTING_FAQ.md          ← Error solutions
```

---

## Questions?

**"I still can't find where to enter the page access token in Meta portal"**
→ You don't enter it there! See PAGE_ACCESS_TOKEN_GUIDE.md

**"What's the difference between page token and ad account token?"**
→ Page token = read comments. Ad account token = read ads. Both needed!

**"How do I know if my tokens are working?"**
→ Test them in Graph API Explorer. See TROUBLESHOOTING_FAQ.md

**"Can I use the same app for multiple clients?"**
→ Yes, but better to create separate apps for isolation and easier management

**"Do I need Meta to review/approve my app?"**
→ Not if you keep it in Development Mode and only use it for your own accounts

---

## Final Checklist

Before you start, ensure you have:
- [ ] Read this README
- [ ] Printed/opened QUICK_START_CHECKLIST.md
- [ ] Have PAGE_ACCESS_TOKEN_GUIDE.md handy
- [ ] Ready to fill out CORBEL_SETUP_TEMPLATE.md
- [ ] Access to Meta Business Manager
- [ ] Access to n8n
- [ ] 30-45 minutes of uninterrupted time

**Ready? Go to QUICK_START_CHECKLIST.md to begin!**

---

**Created:** 2025-11-20
**For:** Corbel Setup
**Prepared By:** Claude
**Version:** 1.0

Good luck! 🚀
