# Meta App Setup Guide for Facebook Ad Comment Monitoring
## Complete Step-by-Step Guide for New Client Onboarding

**Client:** Corbel
**Date:** 2025-11-20
**Use Case:** Facebook Ad Comment Monitoring via n8n Workflow

---

## Table of Contents
1. [Prerequisites](#prerequisites)
2. [Part 1: Create Meta App](#part-1-create-meta-app)
3. [Part 2: Configure App Permissions](#part-2-configure-app-permissions)
4. [Part 3: Get Page Access Token](#part-3-get-page-access-token)
5. [Part 4: Get Ad Account Access Token](#part-4-get-ad-account-access-token)
6. [Part 5: Configure n8n Workflow](#part-5-configure-n8n-workflow)
7. [Part 6: Testing](#part-6-testing)
8. [Troubleshooting](#troubleshooting)

---

## Prerequisites

Before starting, ensure you have:
- [ ] Meta Business Manager account access
- [ ] Admin access to the client's Facebook Page
- [ ] Admin access to the client's Ad Account
- [ ] Client's Ad Account ID (format: `act_XXXXXXXXXXXXX`)
- [ ] Access to n8n instance
- [ ] Google Sheet prepared for comment logging (optional)

---

## Part 1: Create Meta App

### Step 1.1: Navigate to Meta Developers
1. Go to https://developers.facebook.com/
2. Click **My Apps** in the top right
3. Click **Create App** button

### Step 1.2: Select App Type
1. Choose **Business** as the app type
2. Click **Next**

### Step 1.3: Fill App Details
- **App Name:** `[Client Name] - Ad Comment Monitor` (e.g., "Corbel - Ad Comment Monitor")
- **App Contact Email:** Your business email
- **Business Account:** Select your Meta Business Manager account
- Click **Create App**

### Step 1.4: Navigate to App Dashboard
- After creation, you'll be redirected to the App Dashboard
- **Save your App ID** - you'll need this later
- **Save your App Secret** (found in Settings > Basic)

---

## Part 2: Configure App Permissions

### Step 2.1: Add Required Products
1. From the App Dashboard, scroll to **Add Products to Your App**
2. Click **Set Up** on the following:
   - **Facebook Login** (required for authentication)
   - **Marketing API** (required for ads data)

### Step 2.2: Configure Facebook Login
1. Go to **Facebook Login** > **Settings**
2. Set **Valid OAuth Redirect URIs:** (leave blank for Graph API Explorer testing)
3. Save changes

### Step 2.3: Request App Permissions
1. Go to **App Review** > **Permissions and Features**
2. Request the following permissions:

**Required Permissions:**
- [x] `pages_read_engagement` - Read comments on pages
- [x] `pages_manage_posts` - Access to page posts
- [x] `pages_read_user_content` - Read user-generated content
- [x] `ads_read` - Read ads data
- [x] `ads_management` - Manage ads

3. For each permission:
   - Click **Request**
   - Fill in the usage description
   - Submit for review (or use in Development Mode for testing)

### Step 2.4: Set App Mode
- During setup/testing, keep app in **Development Mode**
- Switch to **Live Mode** only after permissions are approved

---

## Part 3: Get Page Access Token
**This is the critical step for comment monitoring**

### Step 3.1: Navigate to Graph API Explorer
1. Go to https://developers.facebook.com/tools/explorer/
2. In the top right, select your newly created app from the **Meta App** dropdown
3. Keep **User Token** selected for now

### Step 3.2: Generate User Access Token
1. Click **Generate Access Token** button
2. A login dialog will appear - log in with an account that has:
   - Admin access to the Facebook Page
   - Admin access to the Ad Account
3. Grant all requested permissions

### Step 3.3: Get Available Pages
1. In the Graph API Explorer query field, enter: `/me/accounts`
2. Click **Submit**
3. You'll see a list of all pages the logged-in user manages

**Example Response:**
```json
{
  "data": [
    {
      "access_token": "EAAcEdar4GN8BP...[LONG_TOKEN]",
      "category": "Business Service",
      "name": "Corbel",
      "id": "1234567890",
      "tasks": ["ADVERTISE", "ANALYZE", "CREATE_CONTENT", "MODERATE", "MANAGE"]
    }
  ]
}
```

### Step 3.4: Save the Page Access Token
- **CRITICAL:** The `access_token` field in the response is your **Page Access Token**
- Copy this token and save it securely
- Label it: **"Corbel - Page Access Token (for comments)"**
- This token is used in the "Get Comments for Ad Post" node

### Step 3.5: Convert to Long-Lived Token (Optional but Recommended)
1. Go to https://developers.facebook.com/tools/debug/accesstoken/
2. Paste your Page Access Token
3. Click **Debug**
4. Click **Extend Access Token**
5. Copy the new long-lived token (valid for 60 days)

---

## Part 4: Get Ad Account Access Token

### Step 4.1: Navigate to Business Settings
1. Go to https://business.facebook.com/settings/
2. Click **Accounts** > **Ad Accounts**
3. Find the client's ad account
4. Note the Ad Account ID (format: `act_XXXXXXXXXXXXX`)

### Step 4.2: Generate Ad Account Access Token
1. Go back to Graph API Explorer: https://developers.facebook.com/tools/explorer/
2. Make sure your app is selected
3. Click **Generate Access Token** (or use the existing user token)
4. In the permissions dialog, ensure these are checked:
   - `ads_read`
   - `ads_management`

### Step 4.3: Test Ad Account Access
1. In the Graph API Explorer, enter this query:
   ```
   /act_XXXXXXXXXXXXX/ads?fields=id,name,status&limit=5
   ```
   (Replace `act_XXXXXXXXXXXXX` with your actual Ad Account ID)
2. Click **Submit**
3. If successful, you'll see a list of ads

**Example Response:**
```json
{
  "data": [
    {
      "id": "120212345678901234",
      "name": "Summer Campaign - Ad 1",
      "status": "ACTIVE"
    }
  ]
}
```

### Step 4.4: Save the Ad Account Access Token
- The token you used successfully is your **Ad Account Access Token**
- Copy and save it securely
- Label it: **"Corbel - Ad Account Access Token (for ads)"**
- This token is used in the "Get Active Ads" node

---

## Part 5: Configure n8n Workflow

### Step 5.1: Duplicate the Workflow
1. Open n8n
2. Find the workflow: **"Facebook AD Comment Monitor FINAL | HRAO"**
3. Click the three dots > **Duplicate**
4. Rename to: **"Facebook AD Comment Monitor | Corbel"**

### Step 5.2: Update "Get Active Ads" Node
1. Open the **"Get Active Ads"** node
2. Update the URL:
   ```
   https://graph.facebook.com/v24.0/act_XXXXXXXXXXXXX/ads
   ```
   (Replace with Corbel's Ad Account ID)
3. Update the `access_token` parameter with the **Ad Account Access Token**
4. Keep the fields parameter:
   ```
   id,name,status,creative{effective_object_story_id}
   ```

### Step 5.3: Update "Get Comments for Ad Post" Node
1. Open the **"Get Comments for Ad Post"** node
2. The URL stays dynamic: `https://graph.facebook.com/v24.0/{{ $json.postId }}/comments`
3. Update the `access_token` parameter with the **Page Access Token**
4. Keep the fields parameter:
   ```
   id,message,from,created_time,permalink_url
   ```

### Step 5.4: Update "Process Comment Data" Node
1. Open the **"Process Comment Data"** node (Code node)
2. Update the client configuration:

```javascript
const clients = {
  'act_XXXXXXXXXXXXX': {  // Replace with Corbel's Ad Account ID
    name: 'Corbel',
    channel: '#notifications-corbel',  // Update Slack channel
    priority: 'high'
  }
};
```

3. Update the `adAccountId` variable to match:
```javascript
const adAccountId = 'act_XXXXXXXXXXXXX';  // Corbel's Ad Account ID
```

### Step 5.5: Update Google Sheet (if using)
1. Create a new Google Sheet or use existing
2. Create a sheet named **"Comment Log"**
3. Add headers:
   - Date | Timestamp | Ad Name | Ad ID | Commenter | Comment | Comment URL
4. Update both Google Sheets nodes:
   - **"Log to Google Sheet1"**
   - **"Get Today's Comments"**
5. Update the `documentId` to your new sheet ID

### Step 5.6: Update Slack Notifications
1. Update all Slack nodes with the correct channel:
   - **"Send Slack Notification"** - change channel to `#notifications-corbel`
   - **"Send API Error to Slack"** - change channel to `#notifications-corbel`
   - **"Send Daily Summary to Slack"** - change channel to `#notifications-corbel`

---

## Part 6: Testing

### Step 6.1: Test "Get Active Ads" Node
1. Click on **"Get Active Ads"** node
2. Click **"Execute Node"** (test button)
3. Expected output: List of active ads from Corbel's ad account
4. Check that each ad has `creative.effective_object_story_id`

**Troubleshooting:**
- **Error 190:** Invalid token - regenerate Ad Account Access Token
- **Error 100:** Invalid ad account ID - verify the ID is correct
- **Empty data:** No active ads in the account

### Step 6.2: Test "Get Comments for Ad Post" Node
1. You'll need an ad with actual comments to test this
2. Or manually set a post ID in the test data
3. Click **"Execute Node"**
4. Expected output: List of comments with message, from, created_time

**Troubleshooting:**
- **Error 190:** Invalid token - regenerate Page Access Token
- **Error 100:** Invalid post ID - check the ad has a valid post
- **OAuthException:** Page Access Token doesn't have permission to read comments

### Step 6.3: Test Full Workflow
1. Click **"Execute Workflow"** from the Schedule trigger
2. Monitor each node's output
3. Verify:
   - Ads are retrieved
   - Comments are found (if any exist)
   - Slack notification is sent
   - Comment is logged to Google Sheet

### Step 6.4: Activate the Workflow
1. Once testing is successful, click **"Active"** toggle in top right
2. The workflow will now run every 15 minutes automatically
3. Daily summary will run at 5 PM weekdays

---

## Troubleshooting

### Issue: "Cannot add page access token to Meta App"
**Solution:**
- You don't manually "add" page access tokens to the app in the developer portal
- Page access tokens are generated via the Graph API Explorer
- The screenshot you showed is for **selecting which pages** your app can manage, not for entering tokens
- To add a page to your app:
  1. Go to **App Settings** > **Basic**
  2. Scroll to **App Domains** (leave blank for Graph API)
  3. The key is ensuring the user generating the token has page admin access

### Issue: "Invalid OAuth access token"
**Solution:**
- Token might have expired
- Regenerate token via Graph API Explorer
- For long-lived tokens, extend via Access Token Debugger
- Ensure the app has the required permissions

### Issue: "Error reading comments - OAuthException"
**Solution:**
- The Page Access Token doesn't have `pages_read_engagement` permission
- Regenerate token and ensure all permissions are granted
- Check that the app has been granted access to the page

### Issue: "No effective_object_story_id in ad creative"
**Solution:**
- Some ads don't have posts (e.g., dynamic ads)
- The workflow filters these out automatically
- Only ads with actual posts will be monitored for comments

### Issue: "App permissions stuck in review"
**Solution:**
- For testing, use **Development Mode** - no review needed
- Add your test accounts as **App Testers** or **Developers**
- Development mode allows full testing with limited users
- Submit for review only when ready to go live at scale

---

## Quick Reference: Token Types

| Token Type | Used For | Where to Get | Used In Node |
|------------|----------|-------------|--------------|
| **User Access Token** | Initial authentication | Graph API Explorer | Not stored (intermediate) |
| **Page Access Token** | Reading comments on page posts | `/me/accounts` API call | "Get Comments for Ad Post" |
| **Ad Account Access Token** | Reading ads from ad account | Graph API Explorer with ads_read | "Get Active Ads" |

---

## Checklist: New Client Setup

- [ ] Create Meta App for client
- [ ] Add required products (Facebook Login, Marketing API)
- [ ] Request required permissions
- [ ] Generate Page Access Token via `/me/accounts`
- [ ] Generate Ad Account Access Token via Graph API Explorer
- [ ] Test tokens in Graph API Explorer
- [ ] Duplicate n8n workflow
- [ ] Update "Get Active Ads" node (URL + token)
- [ ] Update "Get Comments for Ad Post" node (token)
- [ ] Update "Process Comment Data" node (client config)
- [ ] Update Slack channel references
- [ ] Update Google Sheet (if using)
- [ ] Test individual nodes
- [ ] Test full workflow execution
- [ ] Activate workflow
- [ ] Monitor first 24 hours for errors

---

## Support Notes

**Token Expiration:**
- Short-lived tokens: 1-2 hours
- Long-lived user tokens: 60 days
- Page access tokens: Don't expire (but can be invalidated)
- Best practice: Extend to long-lived tokens immediately

**App Permissions:**
- Development mode: No review needed, works for app testers
- Live mode: Requires Meta review (can take 3-7 days)
- For internal tools, development mode is sufficient

**Common Mistakes:**
- Using user token instead of page token for comments
- Forgetting to update client config in "Process Comment Data" node
- Not adding the Facebook Page to the app's allowed pages
- Testing with an account that doesn't have page admin access

---

**Document Version:** 1.0
**Last Updated:** 2025-11-20
**Created By:** Claude
**For Client:** Corbel
