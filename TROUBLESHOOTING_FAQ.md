# Troubleshooting FAQ
## Common Issues & Solutions for Facebook Ad Comment Monitoring Setup

---

## Table of Contents
1. [Token Issues](#token-issues)
2. [API Errors](#api-errors)
3. [Permission Errors](#permission-errors)
4. [n8n Workflow Issues](#n8n-workflow-issues)
5. [Data Issues](#data-issues)
6. [Notification Issues](#notification-issues)

---

## Token Issues

### Q: My token expired - how do I get a new one?

**Answer:**
Page Access Tokens from `/me/accounts` typically don't expire, but if they do:

1. Go to Graph API Explorer: https://developers.facebook.com/tools/explorer/
2. Select your app
3. Click "Generate Access Token" (log in as page admin)
4. Call `/me/accounts` again
5. Copy the new `access_token` for your page
6. Update the token in your n8n workflow

**Prevention:** Use the Access Token Debugger to check expiration:
- https://developers.facebook.com/tools/debug/accesstoken/
- Paste your token
- Check "Expires" field
- If it shows a date, click "Extend Access Token"

---

### Q: How do I know if I'm using the right token?

**Answer:**
Test each token in Graph API Explorer:

**For Page Access Token (comments):**
```
Query: me?fields=id,name
Expected: Should show PAGE name (not your personal name)
```

**For Ad Account Access Token (ads):**
```
Query: act_XXXXX/ads?limit=1
Expected: Should return at least one ad (if account has ads)
```

**For User Access Token (only for generating page tokens):**
```
Query: me?fields=id,name
Expected: Should show YOUR personal name
```

---

### Q: Access Token Debugger shows "Invalid Token"

**Answer:**
Your token has been invalidated. Common reasons:
- Password was changed on the account that generated it
- The user was removed as page admin
- The app was disconnected from the page/ad account
- Token was manually revoked in Meta Business Suite

**Solution:** Generate a fresh token using the process in `PAGE_ACCESS_TOKEN_GUIDE.md`

---

## API Errors

### Error 190: "Invalid OAuth 2.0 Access Token"

**Cause:** Token is expired, invalid, or doesn't have required permissions

**Solution:**
1. Use Access Token Debugger to check token validity
2. If expired, generate new token
3. If valid but still failing, check permissions
4. Ensure token type matches use case:
   - Page token for comments
   - Ad account token for ads

---

### Error 100: "Invalid parameter"

**Cause:** Usually means the ID you're querying doesn't exist or token doesn't have access

**Common scenarios:**

**For ads query:**
- Ad Account ID is wrong (should be `act_` + numbers)
- Token doesn't have access to this ad account
- Ad account is disabled

**For comments query:**
- Post ID is wrong or doesn't exist
- Post was deleted
- Page access token doesn't have permission to read this post

**Solution:**
1. Verify the ID is correct
2. Test the ID directly in Graph API Explorer
3. Check permissions for that specific resource

---

### Error 200: "Permissions error"

**Cause:** Missing required permissions

**Solution:**
1. Go to Graph API Explorer
2. Click "Generate Access Token"
3. In the permissions dialog, manually check these:
   - ✓ `pages_read_engagement`
   - ✓ `pages_manage_posts`
   - ✓ `pages_read_user_content`
   - ✓ `ads_read`
   - ✓ `ads_management`
4. Grant permissions
5. Generate new tokens

---

### Error 17: "User request limit reached"

**Cause:** Too many API calls in a short time

**Solution:**
- This is normal if testing frequently
- Wait 1 hour for limit to reset
- For production, the 15-minute schedule should never hit limits
- If hitting limits in production, reduce polling frequency

---

## Permission Errors

### "This endpoint requires the 'pages_read_engagement' permission"

**Cause:** Page Access Token doesn't have permission to read comments

**Solution:**
1. Check if permission is requested in your app:
   - Go to App Dashboard > App Review > Permissions and Features
   - Ensure `pages_read_engagement` is requested (and approved if in Live mode)

2. Generate new token with this permission:
   - Graph API Explorer > Generate Access Token
   - **Manually check** `pages_read_engagement` in the popup
   - Call `/me/accounts` to get new page token

---

### "This endpoint requires the 'ads_read' permission"

**Cause:** Access Token doesn't have permission to read ads

**Solution:**
1. Generate new token in Graph API Explorer
2. Ensure `ads_read` is checked in permissions popup
3. Test with: `act_XXXXX/ads?limit=1`

---

### "(#10) Application does not have permission for this action"

**Cause:** Your app hasn't been granted access to perform this action

**Solution:**
1. Check app is in correct mode:
   - **Development mode:** Works for app testers/developers only
   - **Live mode:** Requires Meta review for advanced permissions

2. Add yourself as app tester:
   - App Dashboard > Roles > Testers
   - Add your Facebook account

3. For Live mode, submit app for review:
   - App Dashboard > App Review > Request
   - Provide use case documentation

---

## n8n Workflow Issues

### Workflow shows "No data" for "Get Active Ads"

**Possible causes:**

1. **No active ads in account**
   - Solution: Create at least one active ad, or test with a different account

2. **Wrong Ad Account ID**
   - Solution: Verify format is `act_1234567890` (starts with `act_`)
   - Find correct ID in Business Manager > Ad Accounts

3. **Token doesn't have access**
   - Solution: Ensure logged-in user is admin of ad account
   - Regenerate token

---

### Workflow shows "No effective_object_story_id"

**Cause:** Some ad types don't create Facebook posts (e.g., dynamic ads, some catalog ads)

**Solution:** This is normal and expected!
- The workflow automatically skips these ads
- Only ads with actual posts will be monitored
- No action needed - this is handled by the "Has Post ID?" filter node

---

### "Get Comments" node fails with Error 100

**Cause:** Post ID doesn't exist or is inaccessible

**Possible reasons:**
1. Post was deleted
2. Ad is paused/deleted
3. Post ID format is wrong

**Solution:**
1. Check if ad is still active
2. Verify post still exists on Facebook
3. Test post ID directly in Graph API Explorer:
   ```
   POST_ID?fields=id,message,created_time
   ```

---

### Workflow runs but no Slack notifications

**Possible causes:**

1. **No new comments**
   - Solution: Check "Process Comment Data" node - it filters for comments < 20 mins old
   - Test with a fresh comment

2. **Slack channel ID wrong**
   - Solution: Verify channel ID in all Slack nodes
   - Use channel name `#notifications-corbel` or ID `C096MFMFDHB`

3. **Filter removing comments**
   - Solution: Check "Filter New Comments" node
   - Ensure comments are passing through (check node output)

---

### Workflow runs but no comments found (but comments exist)

**Cause:** Comments are older than 20 minutes

**Solution:**
The workflow only processes comments created in the last 20 minutes to avoid duplicate notifications.

To test with old comments:
1. Open "Process Comment Data" node
2. Find this line:
   ```javascript
   if (minutesSinceComment > 20) {
   ```
3. Change to:
   ```javascript
   if (minutesSinceComment > 1440) {  // 24 hours
   ```
4. Test
5. Change back to 20 after testing

---

## Data Issues

### Comments appear multiple times in Slack

**Cause:** Workflow ran multiple times and caught the same comment

**Solution:**
This is expected behavior when:
- Manually testing the workflow multiple times
- Comment is still within the 20-minute window

**Prevention:**
- Once activated, don't manually execute (let schedule handle it)
- Alternatively, implement comment ID deduplication in the workflow

---

### Missing fields in Google Sheet

**Cause:** Column mapping in "Log to Google Sheet" node doesn't match sheet headers

**Solution:**
1. Open "Log to Google Sheet1" node
2. Check the "Columns" section mapping
3. Ensure these match your sheet headers exactly:
   - Date
   - Timestamp
   - Ad Name
   - Ad ID
   - Commenter
   - Comment
   - Comment URL

---

### Daily summary shows no comments (but comments were logged)

**Cause:** Date filter in "Get Today's Comments" node not matching

**Solution:**
1. Check the date format in your Google Sheet "Date" column
2. Should be: `YYYY-MM-DD` (e.g., `2025-11-20`)
3. Verify the filter in "Get Today's Comments" node:
   ```
   Date = {{ new Date().toISOString().split('T')[0] }}
   ```

---

## Notification Issues

### Slack notifications have wrong timestamp

**Cause:** Timezone mismatch

**Solution:**
1. Open "Send Slack Notification" node
2. Update the timestamp format to use Australian timezone:
   ```javascript
   new Date($json.commentTime).toLocaleString('en-AU', {
     timeZone: 'Australia/Sydney'
   })
   ```

---

### Daily summary sends at wrong time

**Cause:** n8n server timezone different from expected

**Solution:**
1. Open "Daily Summary - 5 PM Weekdays" trigger node
2. Check the cron expression: `0 17 * * 1-5`
   - This means: "At minute 0 of hour 17 (5 PM) on Monday through Friday"
3. Adjust for your timezone
4. For AEDT (UTC+11), you might need:
   - `0 6 * * 1-5` (6 AM UTC = 5 PM AEDT)

---

## Testing Tips

### How to test without waiting for real comments?

**Method 1: Use Test Data**
1. Click on "Split Comments" node
2. Click "Add test data"
3. Paste sample comment JSON:
```json
{
  "id": "12345_67890",
  "message": "Test comment",
  "from": {
    "name": "Test User",
    "id": "123456"
  },
  "created_time": "2025-11-20T10:00:00+0000",
  "permalink_url": "https://facebook.com/test"
}
```
4. Execute from this node onward

**Method 2: Make a real test comment**
1. Go to one of your ads' posts on Facebook
2. Comment on it (even just "test")
3. Immediately run the workflow
4. Comment should be detected (it's new)
5. Delete test comment after

---

## Quick Diagnostic Checklist

If workflow isn't working, check these in order:

- [ ] **1.** Can you get ads in Graph API Explorer?
  - Test: `act_XXXXX/ads?fields=id,name&limit=3`

- [ ] **2.** Do ads have `creative.effective_object_story_id`?
  - Test: `act_XXXXX/ads?fields=creative{effective_object_story_id}&limit=5`

- [ ] **3.** Can you get comments in Graph API Explorer?
  - Test: `POST_ID/comments?fields=id,message`

- [ ] **4.** Are tokens in correct nodes?
  - "Get Active Ads" = Ad Account token
  - "Get Comments" = Page Access token

- [ ] **5.** Is Ad Account ID correct in all places?
  - "Get Active Ads" URL
  - "Process Comment Data" client config

- [ ] **6.** Is Slack channel correct?
  - All Slack nodes should have same channel

---

## Still Not Working?

### Enable Debug Logging

1. Open "Process Comment Data" node
2. The `console.log` statements will show in n8n execution log
3. Check for:
   - "Processing comment: [ID] Age: [minutes]"
   - "Skipping old comment" (if comment too old)

### Check n8n Execution History

1. Click "Executions" in n8n sidebar
2. Find your workflow executions
3. Click on failed executions
4. Check which node failed and why

### Export and Review Workflow Data

1. Click any node
2. Check the "INPUT" and "OUTPUT" tabs
3. Verify data is flowing correctly through the workflow

---

## Contact Information

**Documentation:**
- Full Guide: `META_APP_SETUP_GUIDE.md`
- Token Help: `PAGE_ACCESS_TOKEN_GUIDE.md`
- Quick Start: `QUICK_START_CHECKLIST.md`

**External Resources:**
- Meta Developer Docs: https://developers.facebook.com/docs/
- Graph API Explorer: https://developers.facebook.com/tools/explorer/
- Access Token Debugger: https://developers.facebook.com/tools/debug/accesstoken/

---

**Version:** 1.0
**Last Updated:** 2025-11-20
**For:** Corbel Setup & General Troubleshooting
