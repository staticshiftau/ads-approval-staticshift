# Facebook Marketing API Setup Guide

## Overview
This guide will help you create a new Facebook Marketing API app for monitoring ad comments across multiple clients. This setup ensures workflows don't break when you add new clients.

## Prerequisites
- Facebook Business Manager account
- Admin access to client Facebook Ad Accounts
- Facebook Developer account

---

## Step 1: Create a Facebook App

1. **Go to Facebook Developers**
   - Visit https://developers.facebook.com/
   - Click "My Apps" → "Create App"

2. **Choose App Type**
   - Select **"Business"** as the app type
   - Click "Next"

3. **App Details**
   - **App Name**: `Static Shift - Ad Comment Monitor` (or your agency name)
   - **App Contact Email**: Your business email
   - **Business Account**: Select your Business Manager account
   - Click "Create App"

---

## Step 2: Add Marketing API Product

1. **Add Products**
   - In your app dashboard, find "Add Products"
   - Locate **"Marketing API"**
   - Click "Set Up"

2. **Configure Marketing API**
   - Accept the terms and conditions
   - Complete any required setup steps

---

## Step 3: Generate Access Token

### Option A: Using Graph API Explorer (Quick Test - 60 days)

1. **Go to Graph API Explorer**
   - Visit https://developers.facebook.com/tools/explorer/
   - Select your newly created app from the dropdown

2. **Set Permissions**
   Click "Add a Permission" and select:
   - `ads_read` - Read ad account data
   - `pages_read_engagement` - Read page posts and comments
   - `pages_read_user_content` - Read comments on page posts
   - `business_management` - Manage business assets

3. **Generate Token**
   - Click "Generate Access Token"
   - Authorize the app
   - **Copy the token immediately** (60-day expiry)

### Option B: Using System User (Recommended - Never Expires)

1. **Create System User**
   - Go to Business Settings → Users → System Users
   - Click "Add" → Create a name like "n8n-automation"
   - Assign role: "Admin" or "Employee" with appropriate permissions

2. **Assign Assets**
   - Add the system user to required Ad Accounts
   - Give "Manage Ad Accounts" permission

3. **Generate Token**
   - Click on the system user
   - Click "Generate New Token"
   - Select your app
   - Select permissions:
     - `ads_read`
     - `pages_read_engagement`
     - `pages_read_user_content`
     - `business_management`
   - Click "Generate Token"
   - **Save this token securely** - it never expires!

---

## Step 4: Get Ad Account IDs

1. **Find Ad Account ID**
   - Go to Facebook Ads Manager
   - Click "All Ad Accounts" dropdown
   - The ID is shown next to each account (format: `act_XXXXX`)

2. **Verify Access**
   - Test the token with this URL in your browser:
   ```
   https://graph.facebook.com/v24.0/act_YOUR_AD_ACCOUNT_ID/ads?access_token=YOUR_ACCESS_TOKEN&limit=1
   ```
   - You should see JSON data (not an error)

---

## Step 5: App Review (For Production - Optional)

If you need long-term access or access to pages you don't own:

1. **Business Verification**
   - Complete business verification in Business Manager
   - This is required for advanced permissions

2. **App Review**
   - Go to App Review in your Facebook App
   - Submit for review of required permissions
   - Provide use case explanation

---

## Step 6: Store Credentials Securely

**Create a credentials file for each client:**

```json
{
  "clientName": "Corbel Group",
  "adAccountId": "act_3634319663373564",
  "accessToken": "YOUR_NEVER_EXPIRING_TOKEN_HERE",
  "slackChannel": "#notifications-corbel",
  "googleSheetId": "1TNBlbHauHOkcMNi16NzJt94Zy_qet7QhOVKIuiOmD0k",
  "createdDate": "2025-11-26",
  "tokenType": "system-user",
  "tokenExpiry": "never"
}
```

---

## Best Practices

### Security
- ✅ **Use System User tokens** for automation (never expire)
- ✅ **Store tokens in environment variables** or encrypted credential store
- ✅ **Never commit tokens to git** - use .gitignore
- ❌ **Don't use personal user tokens** (expire after 60 days)
- ❌ **Don't hardcode tokens in workflows**

### Permissions
- Only request permissions you actually need
- Use `ads_read` not `ads_management` if you only need read access
- Request additional permissions only when required

### Token Management
- Document which token belongs to which client
- Set calendar reminders to check token health
- Monitor error logs for permission issues
- Keep backup tokens for each client

### Multiple Clients
- Create one System User per client (recommended)
- OR use one System User with access to all client ad accounts
- Document the mapping in your credentials file

---

## Troubleshooting

### "Invalid OAuth 2.0 Access Token"
- Token has expired (if using 60-day token)
- Token doesn't have required permissions
- Solution: Generate new token with correct permissions

### "Unsupported get request"
- Wrong API version
- Incorrect endpoint format
- Solution: Use v24.0 or latest version

### "User does not have permission to access this ad account"
- System user not assigned to ad account
- Solution: Add system user in Business Settings → Ad Accounts

### Token Expires Unexpectedly
- Using user token instead of system user token
- Solution: Use System User tokens (never expire)

---

## API Rate Limits

Facebook Marketing API has rate limits:
- **App-level**: 200 calls per hour per user
- **Ad Account-level**: Varies by ad spend

Best practices:
- Poll every 15 minutes (not every minute)
- Implement exponential backoff on errors
- Use batch requests when possible

---

## Resources

- [Facebook Marketing API Documentation](https://developers.facebook.com/docs/marketing-api)
- [Graph API Explorer](https://developers.facebook.com/tools/explorer/)
- [Business Manager](https://business.facebook.com/)
- [App Dashboard](https://developers.facebook.com/apps/)

---

## Quick Reference: Required Permissions

| Permission | Purpose |
|-----------|---------|
| `ads_read` | Read ad account data, ads, campaigns |
| `pages_read_engagement` | Read comments on page posts |
| `pages_read_user_content` | Read user-generated content |
| `business_management` | Access business-owned assets |

---

**Last Updated**: November 2025
**API Version Used**: v24.0
