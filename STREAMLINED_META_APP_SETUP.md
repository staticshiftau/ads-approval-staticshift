# Streamlined Meta App Setup for Facebook Ad Comment Monitoring
## What ACTUALLY Works - Lessons from Corbel Setup

**Last Updated:** 2025-11-21
**Based on:** Real setup experience with Corbel client

---

## The Problem We Discovered

**Assumption:** If you have Business Manager access to a page, `/me/accounts` will show that page.

**Reality:** `/me/accounts` ONLY shows pages where you're a **direct Page Admin**. Business Manager access ≠ Page Admin role.

**Solution:** Use **System User tokens** instead of trying to become a direct Page Admin!

---

## Quick Decision Tree

```
Do you have Business Manager access to the client's page/ad account?
│
├─ YES → Skip trying to add yourself as admin
│         Go straight to System User token (faster!)
│
└─ NO → Ask client to add you to Business Manager first
```

---

## The ACTUAL Working Process

### Part 1: Get Ad Account Access Token (5 minutes)

**This one is easy!**

1. **Go to:** https://developers.facebook.com/tools/explorer/
2. **Select your Meta App:** "Static Shift Marketing API" (or your app)
3. **Click:** "Generate Access Token"
4. **Log in** as someone with Ad Account access
5. **Grant permissions:** Make sure `ads_read` is checked
6. **Test with this query:**
   ```
   act_XXXXXXXXXXXXX/ads?fields=id,name,status&limit=3
   ```
   (Replace `act_XXX` with client's Ad Account ID)
7. **If it works:** Copy the token from "Access Token" field
8. **Save it as:** "Client Name - Ad Account Token"

✅ **Done! That's Token 1 of 2.**

---

### Part 2: Get Page Access Token (10 minutes)

**⚠️ KEY INSIGHT:** Don't waste time trying `/me/accounts` if you access the page through Business Manager!

#### The Fast Path: System User Token

**Step 1: Create/Use System User**

1. **Go to:** https://business.facebook.com/settings/
2. **Navigate to:** Users > System Users
3. **Check if you can create one:**
   - If YES: Click "Add" → Name: "Client API Access" → Role: "Employee" → Create
   - If NO (hit limit): Use existing System User

**Step 2: Assign Page to System User**

1. **Click on the System User** you're using
2. **Click:** "Add Assets" or "Assign Assets"
3. **Select:** Facebook Pages
4. **Check:** Client's Facebook Page
5. **Select permissions:**
   - ✅ Manage Page (or Full Control)
6. **Save**

**Step 3: Generate Token**

1. **Still on System User page**
2. **Click:** "Generate New Token"
3. **Select app:** Your Meta App (e.g., "Static Shift Marketing API")
4. **Set expiration:** 60 days or Never (if available)
5. **Assign permissions:**
   - ✅ pages_read_engagement
   - ✅ pages_read_user_content
   - ✅ pages_manage_posts
6. **Generate Token**
7. **Copy the token**

**Step 4: Test the Token**

1. **Go to:** https://developers.facebook.com/tools/explorer/
2. **Paste the System User token** into "Access Token" field
3. **Get a test post ID** by running:
   ```
   act_XXXXX/ads?fields=creative{effective_object_story_id}&limit=1
   ```
4. **Copy the `effective_object_story_id`**
5. **Test comments access:**
   ```
   POST_ID/comments?fields=id,message,from&limit=5
   ```
6. **If you get `{"data": []}` or actual comments:** ✅ IT WORKS!
7. **If you get an error:** Go back and check permissions

✅ **Done! That's Token 2 of 2.**

---

## What We Learned (Don't Do This!)

### ❌ Time Wasters:

1. **Trying to add yourself as direct Page Admin when you have Business Manager access**
   - Doesn't help with token generation
   - Complicated permission flows
   - Can hit role limits

2. **Trying `/me/accounts` over and over**
   - Only works if you're direct Page Admin
   - Business Manager access won't show up here
   - Use System User instead!

3. **Looking for a "tokens" section in Meta App settings**
   - Doesn't exist
   - Tokens are generated via API/Graph Explorer

4. **Trying to "add" a page access token to the app**
   - Not how it works
   - Tokens are generated, not added

---

## The 15-Minute Setup (What Actually Works)

### Phase 1: Get Both Tokens (10 min)

1. ✅ **Ad Account Token:** Graph API Explorer → Test query → Copy token
2. ✅ **Page Token:** System User → Assign page → Generate token → Test

### Phase 2: Update n8n Workflow (5 min)

1. **Duplicate HRAO workflow**
2. **Update "Get Active Ads" node:**
   - URL: `https://graph.facebook.com/v24.0/act_CLIENTID/ads`
   - access_token: Ad Account Token
3. **Update "Get Comments for Ad Post" node:**
   - access_token: Page Access Token
4. **Update "Process Comment Data" node:**
   - Change client config
   - Change ad account ID
5. **Update all Slack nodes:** Change channel
6. **Test workflow**
7. **Activate!**

---

## Token Reference Card

| Token Type | Where to Get It | Test Query | Used In Node |
|------------|----------------|------------|--------------|
| **Ad Account Token** | Graph API Explorer with `ads_read` | `act_XXX/ads?limit=1` | "Get Active Ads" |
| **Page Access Token** | System User + Page assignment | `POST_ID/comments?limit=1` | "Get Comments for Ad Post" |

---

## Troubleshooting

### "Can't create System User - limit reached"

**Solution:** Use the existing System User. Most businesses only need one.

### "No permissions available when generating token"

**Solution:** You forgot to **assign the page to the System User first**!
- Go back to System User
- Click "Add Assets" or "Assign Assets"
- Assign the client's Facebook Page
- THEN generate token

### "Invalid OAuth 2.0 Access Token" when testing comments

**Solution:** You're using a User token, not a Page token.
- Make sure you generated the token FROM the System User
- Make sure the page was assigned BEFORE generating

### "A Page access token is required"

**Solution:** Same as above - you need a page-scoped token, not a user token.

---

## Quick Setup Checklist

### Before You Start:
- [ ] Client's Ad Account ID: `act_________________`
- [ ] Client's Facebook Page name: _______________
- [ ] You have Business Manager access
- [ ] You have access to n8n

### Token Generation:
- [ ] Ad Account Token generated and tested
- [ ] System User has client's page assigned
- [ ] Page Access Token generated from System User
- [ ] Page Access Token tested with comments query
- [ ] Both tokens saved securely

### n8n Configuration:
- [ ] Workflow duplicated
- [ ] "Get Active Ads" node updated (URL + token)
- [ ] "Get Comments for Ad Post" node updated (token)
- [ ] "Process Comment Data" node updated (client config)
- [ ] All Slack nodes updated (channel)
- [ ] Workflow tested manually
- [ ] Workflow activated

---

## The "Aha!" Moments

### 1. Business Manager Access ≠ Page Admin

If you manage a page through Business Manager:
- ✅ You CAN generate tokens via System User
- ❌ You WON'T see it in `/me/accounts`

**Don't fight it - just use System User tokens!**

### 2. System Users Are Made for This

System Users are designed for:
- API access
- Automation
- Long-lived tokens
- Business Manager scenarios

**This is the RIGHT tool for the job!**

### 3. Test Early, Test Often

After EACH token generation:
- ✅ Test with Graph API Explorer
- ✅ Confirm it works BEFORE updating n8n
- ✅ Don't assume it worked

**5 minutes of testing saves hours of debugging!**

---

## Time Comparison

### What We Did (Trial & Error): ~2 hours
- Tried `/me/accounts`: 20 min
- Tried adding as Page Admin: 30 min
- Tried multiple approaches: 40 min
- Finally System User: 10 min
- Testing & setup: 20 min

### What You Should Do (This Guide): ~15 minutes
- Get Ad Account token: 5 min
- Get System User Page token: 5 min
- Update n8n workflow: 5 min

**60x faster when you know the right path!**

---

## Future Clients - The Script

**For every new client:**

1. "What's your Ad Account ID?" → Test in Graph Explorer → Get token ✅
2. "I'll use our System User for page access" → Assign page → Generate token ✅
3. Update workflow → Test → Activate ✅

**Done in 15 minutes. Every time.**

---

## Common Questions

**Q: Can I use the same System User for multiple clients?**
A: Yes! Just assign multiple pages to the same System User.

**Q: Do System User tokens expire?**
A: You can set them to 60 days or "Never" (depending on app settings). Long-lived is fine for automation.

**Q: What if the client doesn't have Business Manager?**
A: Then you need to be added as a direct Page Admin, and `/me/accounts` will work. But most businesses have Business Manager.

**Q: Can I use the same token for both ads and comments?**
A: No. You need:
- Ad Account token (with `ads_read` permission) for ads
- Page token (with `pages_read_engagement` permission) for comments

---

## Success Criteria

You know it's working when:
- ✅ Graph Explorer test queries return data (not errors)
- ✅ n8n "Get Active Ads" node returns Corbel's ads
- ✅ n8n "Get Comments" node returns comments (or empty array if none)
- ✅ Slack notifications appear when comments are posted
- ✅ Workflow runs automatically every 15 minutes

---

## Files in This Package

```
STREAMLINED_META_APP_SETUP.md     ← You are here (the REAL guide)
META_APP_SETUP_GUIDE.md           ← Full detailed version
PAGE_ACCESS_TOKEN_GUIDE.md        ← Deep dive on token confusion
QUICK_START_CHECKLIST.md          ← Fast checklist
CORBEL_SETUP_TEMPLATE.md          ← Fill-in template
TROUBLESHOOTING_FAQ.md            ← Error solutions
```

---

**Document Version:** 2.0 (The "What Actually Worked" Edition)
**Last Updated:** 2025-11-21
**Time Saved:** ~1 hour 45 minutes per client setup
**Created By:** Claude & Pauline (after much trial and error! 😄)

---

## TL;DR (The Absolute Fastest Path)

```
1. Graph API Explorer → Test ad account → Copy token
2. Business Manager → System User → Assign page → Generate token
3. n8n → Duplicate workflow → Update tokens → Test → Activate
4. Done in 15 minutes ✅
```

**Skip everything else. This is all you need.**
