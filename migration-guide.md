# Migration Guide: Hardcoded Tokens → Centralized Credentials

## Overview
This guide helps you migrate from hardcoded Facebook API tokens in n8n workflows to a centralized credential management system.

---

## Current Problem

Your workflow has **hardcoded tokens** in multiple places:

```json
{
  "name": "Get Active Ads",
  "parameters": {
    "queryParameters": {
      "parameters": [
        {
          "name": "access_token",
          "value": "EAAcEdar4GN8BQB97IG33qWrrT7..." // ❌ HARDCODED!
        }
      ]
    }
  }
}
```

### Issues:
- ❌ Tokens expire → workflow breaks
- ❌ Multiple clients → multiple workflow copies
- ❌ Update token → update 5+ places
- ❌ Security risk (tokens in workflow JSON)

---

## Solution: n8n Credentials

### Step 1: Create n8n Facebook Credential

1. **Open n8n**
   - Go to Settings → Credentials
   - Click "Create New Credential"

2. **Choose Credential Type**
   - Search for "Generic Credential" or "HTTP Header Auth"
   - OR use "OAuth2 API" for Facebook

3. **Create Custom Facebook API Credential**

   **Option A: Using Generic Credential**
   - Name: `Facebook Marketing API - Corbel`
   - Add field: `access_token`
   - Value: `YOUR_SYSTEM_USER_TOKEN`

   **Option B: Using HTTP Header Auth**
   - Name: `Facebook Marketing API - Corbel`
   - Header Name: `Authorization`
   - Header Value: `Bearer YOUR_SYSTEM_USER_TOKEN`

---

### Step 2: Update Workflow to Use Credentials

#### BEFORE (Hardcoded):
```json
{
  "name": "Get Active Ads",
  "parameters": {
    "url": "https://graph.facebook.com/v24.0/act_3634319663373564/ads",
    "sendQuery": true,
    "queryParameters": {
      "parameters": [
        {
          "name": "access_token",
          "value": "EAAcEdar4GN8BQB97IG33qWrrT7..." // ❌
        }
      ]
    }
  }
}
```

#### AFTER (Using Credentials):

**Option 1: Use n8n Credentials** (Recommended)
```json
{
  "name": "Get Active Ads",
  "parameters": {
    "url": "https://graph.facebook.com/v24.0/act_3634319663373564/ads",
    "sendQuery": true,
    "queryParameters": {
      "parameters": [
        {
          "name": "access_token",
          "value": "={{ $credentials.access_token }}" // ✅ From credential
        }
      ]
    }
  },
  "credentials": {
    "genericCredentialType": {
      "id": "YOUR_CREDENTIAL_ID",
      "name": "Facebook Marketing API - Corbel"
    }
  }
}
```

**Option 2: Use Environment Variables** (Alternative)
```json
{
  "name": "Get Active Ads",
  "parameters": {
    "url": "https://graph.facebook.com/v24.0/act_3634319663373564/ads",
    "sendQuery": true,
    "queryParameters": {
      "parameters": [
        {
          "name": "access_token",
          "value": "={{ $env.FB_ACCESS_TOKEN_CORBEL }}" // ✅ From env
        }
      ]
    }
  }
}
```

---

### Step 3: Set Up Environment Variables (If Using Option 2)

1. **Edit your n8n .env file:**
   ```bash
   # Facebook Marketing API Tokens
   FB_ACCESS_TOKEN_CORBEL="YOUR_SYSTEM_USER_TOKEN_FOR_CORBEL"
   FB_ACCESS_TOKEN_CLIENT2="YOUR_SYSTEM_USER_TOKEN_FOR_CLIENT2"

   # Ad Account IDs
   FB_AD_ACCOUNT_CORBEL="act_3634319663373564"
   FB_AD_ACCOUNT_CLIENT2="act_XXXXXXXXXX"
   ```

2. **Restart n8n**
   ```bash
   # If using Docker
   docker-compose restart n8n

   # If using npm
   pm2 restart n8n
   ```

3. **Access in workflows:**
   ```javascript
   // In HTTP Request node
   $env.FB_ACCESS_TOKEN_CORBEL

   // In Code node
   const token = process.env.FB_ACCESS_TOKEN_CORBEL;
   ```

---

## Step 4: Create Workflow Template for New Clients

Instead of duplicating workflows, use **workflow variables**:

### Create Client Configuration Node

Add a "Set" node at the start of your workflow:

```json
{
  "name": "Client Configuration",
  "type": "n8n-nodes-base.set",
  "parameters": {
    "values": {
      "string": [
        {
          "name": "clientName",
          "value": "Corbel Group"
        },
        {
          "name": "adAccountId",
          "value": "act_3634319663373564"
        },
        {
          "name": "slackChannel",
          "value": "#notifications-corbel"
        },
        {
          "name": "googleSheetId",
          "value": "1TNBlbHauHOkcMNi16NzJt94Zy_qet7QhOVKIuiOmD0k"
        }
      ]
    }
  }
}
```

Then reference throughout workflow:
```javascript
// Get Active Ads URL
https://graph.facebook.com/v24.0/{{ $('Client Configuration').item.json.adAccountId }}/ads

// Slack channel
{{ $('Client Configuration').item.json.slackChannel }}
```

---

## Step 5: Update Existing Workflow

### Changes Needed:

1. **Get Active Ads Node**
   - Replace hardcoded `access_token` with `={{ $credentials.access_token }}`
   - Or `={{ $env.FB_ACCESS_TOKEN_CORBEL }}`

2. **Get Comments for Ad Post Node**
   - Same as above

3. **Process Comment Data Node**
   - Update client configuration to read from "Client Configuration" node

### Updated Process Comment Data Code:

```javascript
// OLD: Hardcoded client config ❌
const clients = {
  'act_3634319663373564': {
    name: 'Corbel Group',
    channel: '#notifications-corbel',
    priority: 'high'
  }
};

// NEW: Dynamic from workflow variable ✅
const clientConfig = $('Client Configuration').item.json;
const adAccountId = clientConfig.adAccountId;

return {
  json: {
    clientName: clientConfig.clientName,
    clientChannel: clientConfig.slackChannel,
    // ... rest of the data
  }
};
```

---

## Step 6: Create One Workflow Per Client

### Recommended Structure:

```
workflows/
├── facebook-ad-monitor-corbel.json
├── facebook-ad-monitor-client2.json
└── facebook-ad-monitor-template.json (for new clients)
```

### Each workflow has:
1. Different credential selected (e.g., "Facebook API - Corbel")
2. Different "Client Configuration" values
3. Same logic/nodes (everything else identical)

---

## Step 7: Test Migration

### Checklist:

- [ ] Created Facebook app with System User
- [ ] Generated never-expiring access token
- [ ] Created n8n credential for each client
- [ ] Updated "Get Active Ads" node to use credential
- [ ] Updated "Get Comments" node to use credential
- [ ] Added "Client Configuration" node
- [ ] Updated "Process Comment Data" to read from config
- [ ] Tested workflow runs successfully
- [ ] Verified Slack notifications work
- [ ] Verified Google Sheets logging works
- [ ] Removed hardcoded tokens from workflow

---

## Rollback Plan

If something goes wrong:

1. **Keep backup of original workflow**
   ```bash
   # Export original workflow first
   workflows-backup/
   └── facebook-ad-monitor-corbel-ORIGINAL.json
   ```

2. **Test in duplicate workflow first**
   - Duplicate workflow
   - Make changes in duplicate
   - Test thoroughly
   - Only then delete original

---

## Multi-Client Setup Strategy

### Option A: One Workflow Per Client (Recommended)

**Pros:**
- Easy to manage
- Client-specific customizations
- Isolate failures
- Different schedules per client

**Cons:**
- More workflows to maintain
- Duplicate logic

**When to use:** 1-10 clients

---

### Option B: One Workflow for All Clients

Use a "Get Clients" node that loops through all clients:

```javascript
// Get Clients node
const clients = [
  {
    name: "Corbel Group",
    adAccountId: "act_3634319663373564",
    token: $env.FB_ACCESS_TOKEN_CORBEL,
    slackChannel: "#notifications-corbel"
  },
  {
    name: "Client 2",
    adAccountId: "act_XXXXXXXXXX",
    token: $env.FB_ACCESS_TOKEN_CLIENT2,
    slackChannel: "#notifications-client2"
  }
];

return clients.map(client => ({ json: client }));
```

Then loop through with "Split In Batches" node.

**Pros:**
- Single workflow to maintain
- Centralized logic
- Easy to add new clients

**Cons:**
- One failure affects all clients
- Complex error handling
- Harder to customize per client

**When to use:** 10+ clients with identical requirements

---

## Monitoring & Maintenance

### Set Up Alerts

1. **Token Expiry Alert** (if not using system user)
   - Add calendar reminder 50 days after token creation
   - Check token health weekly

2. **Workflow Failure Alert**
   - n8n has built-in error workflow
   - Set up email/Slack on workflow errors

3. **API Rate Limit Monitoring**
   - Check Facebook API usage in Business Manager
   - Adjust polling frequency if needed

### Monthly Checklist

- [ ] Review error logs for all clients
- [ ] Check token health (if not using system user tokens)
- [ ] Verify all clients still active
- [ ] Update API version if new version available
- [ ] Check Facebook API changelog for breaking changes

---

## Security Best Practices

### DO ✅
- Use System User tokens (never expire)
- Store tokens in n8n credentials or environment variables
- Use `.gitignore` to exclude credentials files
- Rotate tokens if compromised
- Use minimum required permissions
- Enable 2FA on Facebook accounts

### DON'T ❌
- Commit tokens to git
- Share tokens in Slack/email
- Use personal user tokens for automation
- Store tokens in plaintext files
- Give excessive permissions
- Use the same token across unrelated apps

---

## Next Steps

1. **Immediate** (Today)
   - [ ] Create Facebook app
   - [ ] Generate system user token for Corbel
   - [ ] Create n8n credential
   - [ ] Test on duplicate workflow

2. **This Week**
   - [ ] Migrate Corbel workflow completely
   - [ ] Test for 2-3 days
   - [ ] Document any issues
   - [ ] Update this guide if needed

3. **When Adding New Clients**
   - [ ] Generate new system user token
   - [ ] Create new n8n credential
   - [ ] Duplicate template workflow
   - [ ] Update "Client Configuration" node
   - [ ] Test before going live

---

## Support & Resources

- **Facebook Marketing API Docs**: https://developers.facebook.com/docs/marketing-api
- **n8n Credentials Guide**: https://docs.n8n.io/credentials/
- **n8n Environment Variables**: https://docs.n8n.io/hosting/environment-variables/

---

**Questions or Issues?**

Common issues and solutions are in `facebook-api-setup-guide.md`

**Last Updated**: November 2025
