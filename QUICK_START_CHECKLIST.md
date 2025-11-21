# Quick Start Checklist - Corbel Setup
## 30-Minute Setup Guide

Print this and check off each item as you complete it.

---

## PHASE 1: Meta App Creation (10 minutes)

- [ ] **1.1** Go to https://developers.facebook.com/apps/
- [ ] **1.2** Click "Create App" > Select "Business"
- [ ] **1.3** Name: "Corbel - Ad Comment Monitor"
- [ ] **1.4** Save App ID: _______________
- [ ] **1.5** Save App Secret: _______________
- [ ] **1.6** Add Product: "Facebook Login"
- [ ] **1.7** Add Product: "Marketing API"

---

## PHASE 2: Get Tokens (15 minutes)

### Part A: Page Access Token

- [ ] **2.1** Go to https://developers.facebook.com/tools/explorer/
- [ ] **2.2** Select "Corbel - Ad Comment Monitor" from app dropdown
- [ ] **2.3** Click "Generate Access Token"
- [ ] **2.4** Log in as Corbel Page Admin
- [ ] **2.5** Grant all permissions (especially `pages_read_engagement`)
- [ ] **2.6** In query field, type: `me/accounts`
- [ ] **2.7** Click Submit
- [ ] **2.8** Find "Corbel" page in response
- [ ] **2.9** Copy the `access_token` value from that page object
- [ ] **2.10** Save as "Page Access Token": _______________

### Part B: Ad Account Access Token

- [ ] **2.11** Get Corbel's Ad Account ID from Business Manager: act_______________
- [ ] **2.12** Still in Graph API Explorer, click "Generate Access Token" (or reuse current)
- [ ] **2.13** Ensure `ads_read` permission is granted
- [ ] **2.14** Test with query: `act_XXXXX/ads?fields=id,name&limit=3`
- [ ] **2.15** If successful, copy the current access token
- [ ] **2.16** Save as "Ad Account Access Token": _______________

---

## PHASE 3: Configure n8n Workflow (5 minutes)

- [ ] **3.1** Open n8n
- [ ] **3.2** Find workflow: "Facebook AD Comment Monitor FINAL | HRAO"
- [ ] **3.3** Click ⋯ menu > Duplicate
- [ ] **3.4** Rename to: "Facebook AD Comment Monitor | Corbel"
- [ ] **3.5** Open "Get Active Ads" node
  - [ ] Update URL to: `https://graph.facebook.com/v24.0/act_XXXXX/ads`
  - [ ] Update `access_token` parameter to Ad Account Access Token
- [ ] **3.6** Open "Get Comments for Ad Post" node
  - [ ] Update `access_token` parameter to Page Access Token
- [ ] **3.7** Open "Process Comment Data" node
  - [ ] Update client config:
    ```javascript
    const clients = {
      'act_XXXXX': {
        name: 'Corbel',
        channel: '#notifications-corbel',
        priority: 'high'
      }
    };
    ```
  - [ ] Update `adAccountId` variable
- [ ] **3.8** Update all Slack nodes to channel: `#notifications-corbel`

---

## PHASE 4: Testing (5 minutes)

- [ ] **4.1** Click "Get Active Ads" node > Execute Node
- [ ] **4.2** Verify ads are returned
- [ ] **4.3** Check that `creative.effective_object_story_id` exists
- [ ] **4.4** Execute full workflow (if ads with comments exist)
- [ ] **4.5** Verify Slack notification received
- [ ] **4.6** Check Google Sheet updated (if configured)

---

## PHASE 5: Activation

- [ ] **5.1** Click "Active" toggle in top right of workflow
- [ ] **5.2** Verify schedule shows: "Every 15 minutes"
- [ ] **5.3** Wait 15 minutes and check for first automatic run
- [ ] **5.4** Monitor #notifications-corbel channel

---

## DONE!

**Setup completed by:** _______________
**Date:** _______________
**Time taken:** _______________ minutes

---

## If Something Goes Wrong

### Can't get ads (Error 190)
→ Regenerate Ad Account Access Token
→ Ensure `ads_read` permission is granted

### Can't get comments (Error 190 or OAuthException)
→ Regenerate Page Access Token via `/me/accounts`
→ Ensure `pages_read_engagement` permission is granted
→ Verify logged in as Page Admin

### No ads returned
→ Check Ad Account ID is correct (should start with `act_`)
→ Verify account has active ads
→ Check ads have status "ACTIVE"

### No post ID in ads
→ Some ads don't have posts (dynamic ads, etc.)
→ This is normal - workflow will skip these

---

## Support

**Detailed Guide:** See `META_APP_SETUP_GUIDE.md`
**Token Help:** See `PAGE_ACCESS_TOKEN_GUIDE.md`
**Template:** See `CORBEL_SETUP_TEMPLATE.md`

---

**Version:** 1.0
**Client:** Corbel
