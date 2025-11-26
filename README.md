# Facebook Ads Approval System

A complete system for managing Facebook ad approvals and monitoring ad comments for multiple clients.

## System Components

### 1. **Ad Approval Portal** (`index.html`)
   - React-based approval interface
   - Integrates with Google Sheets
   - Client-facing ad review system
   - Real-time status updates

### 2. **Comment Monitoring Workflow** (n8n)
   - Monitors Facebook ad comments
   - Sends Slack notifications
   - Logs to Google Sheets
   - Daily summary reports

---

## Quick Start

### For New Clients

1. **Set Up Facebook Marketing API**
   - Follow: `facebook-api-setup-guide.md`
   - Create system user token (never expires)
   - Get ad account ID

2. **Configure Credentials**
   - Copy `credentials-template.json` to `credentials.json`
   - Add client information
   - **Never commit credentials.json to git!**

3. **Set Up n8n Workflow**
   - Follow: `migration-guide.md`
   - Create n8n credential
   - Import workflow template
   - Configure client settings

4. **Set Up Approval Portal**
   - Update `index.html` CONFIG section
   - Set Google Sheet ID
   - Set API keys
   - Deploy to hosting

---

## File Structure

```
ads-approval-eddie/
├── index.html                      # Ad approval portal (React app)
├── facebook-api-setup-guide.md     # How to create Facebook app
├── credentials-template.json       # Template for client credentials
├── migration-guide.md              # How to migrate workflows
├── README.md                       # This file
├── .gitignore                      # Git ignore rules
└── workflows/                      # n8n workflow exports (optional)
    ├── facebook-ad-monitor-corbel.json
    └── facebook-ad-monitor-template.json
```

---

## Current Clients

### Corbel Group
- **Ad Account**: `act_3634319663373564`
- **Slack Channel**: `#notifications-corbel`
- **Google Sheet**: [Link](https://docs.google.com/spreadsheets/d/1TNBlbHauHOkcMNi16NzJt94Zy_qet7QhOVKIuiOmD0k)
- **Status**: ✅ Active

---

## Documentation

| File | Description |
|------|-------------|
| `facebook-api-setup-guide.md` | Complete guide to create Facebook Marketing API app |
| `credentials-template.json` | Template for storing client API credentials |
| `migration-guide.md` | How to migrate from hardcoded tokens to credentials |

---

## Features

### Ad Approval Portal
- ✅ Facebook-style ad preview
- ✅ Approve/Edit/Reject workflow
- ✅ Real-time Google Sheets sync
- ✅ Client notes and feedback
- ✅ Mobile responsive
- ✅ Image error handling
- ✅ Local caching

### Comment Monitor
- ✅ Checks every 15 minutes
- ✅ Filters new comments only
- ✅ Slack notifications with ad context
- ✅ Google Sheets logging
- ✅ Daily summary at 5 PM (weekdays)
- ✅ Error handling & API error alerts
- ✅ Multi-client support

---

## Setup Instructions

### Prerequisites
- Facebook Business Manager account
- Facebook Developer account
- n8n instance
- Google Sheets API access
- Slack workspace with webhook access

### 1. Facebook API Setup

Follow the detailed guide in `facebook-api-setup-guide.md`:

```bash
# Key steps:
1. Create Facebook App
2. Add Marketing API product
3. Create System User
4. Generate never-expiring token
5. Get ad account ID
```

### 2. Configure Credentials

```bash
# Copy template
cp credentials-template.json credentials.json

# Edit with your client details
{
  "clients": [
    {
      "clientName": "Your Client",
      "adAccountId": "act_XXXXX",
      "accessToken": "YOUR_TOKEN",
      "slackChannel": "#notifications",
      ...
    }
  ]
}
```

### 3. Set Up n8n Workflow

```bash
# 1. Create credential in n8n
# Settings → Credentials → New Credential → Generic Credential
# Name: "Facebook Marketing API - [Client Name]"
# Field: access_token = YOUR_SYSTEM_USER_TOKEN

# 2. Import workflow
# Import → Upload JSON → Select workflow file

# 3. Update nodes:
# - Client Configuration: Update client details
# - Get Active Ads: Select credential
# - Get Comments: Select credential
# - Slack nodes: Select Slack credential
# - Google Sheets: Select Google credential

# 4. Activate workflow
```

### 4. Deploy Approval Portal

```bash
# 1. Update CONFIG in index.html:
const CONFIG = {
  SHEET_ID: 'YOUR_GOOGLE_SHEET_ID',
  API_KEY: 'YOUR_GOOGLE_API_KEY',
  APPS_SCRIPT_URL: 'YOUR_APPS_SCRIPT_URL',
  ...
};

# 2. Deploy to hosting (GitHub Pages, Netlify, Vercel, etc.)
git add index.html
git commit -m "Update client config"
git push origin main
```

---

## Maintenance

### Weekly
- [ ] Check n8n workflow execution logs
- [ ] Review Slack notifications for errors
- [ ] Verify Google Sheets logging

### Monthly
- [ ] Review token health (should be "never expires")
- [ ] Check Facebook API changelog for updates
- [ ] Update API version if needed
- [ ] Audit client list (add/remove as needed)

### When Adding New Client
1. Generate new system user token
2. Create new n8n credential
3. Duplicate workflow template
4. Update client configuration
5. Test thoroughly before activation

---

## Troubleshooting

### "Invalid OAuth 2.0 Access Token"
- **Cause**: Token expired or invalid
- **Solution**: Generate new system user token (see guide)

### "No comments detected"
- **Cause**: Incorrect post ID or permissions
- **Solution**: Check ad has `effective_object_story_id` in creative

### "API Error" Slack alerts
- **Cause**: Rate limit, token issue, or network error
- **Solution**: Check error message, verify token permissions

### Workflow not triggering
- **Cause**: Schedule disabled or workflow inactive
- **Solution**: Check workflow is activated in n8n

---

## Security

### ✅ Best Practices
- Use System User tokens (never expire)
- Store credentials in n8n or environment variables
- Never commit `credentials.json` to git
- Use `.gitignore` for sensitive files
- Enable 2FA on all Facebook accounts
- Use minimum required API permissions

### ❌ Don't
- Commit access tokens to git
- Share tokens in Slack/email
- Use personal user tokens
- Store tokens in plaintext in public repos
- Give excessive permissions

---

## API Versions

- **Facebook Marketing API**: v24.0 (current)
- **Google Sheets API**: v4
- **n8n**: Latest stable
- **React**: 18

### Updating Facebook API Version

```javascript
// In n8n HTTP Request nodes:
// OLD: https://graph.facebook.com/v24.0/...
// NEW: https://graph.facebook.com/v25.0/...

// Update in both:
// - Get Active Ads node
// - Get Comments for Ad Post node
```

---

## Support

### Resources
- [Facebook Marketing API Docs](https://developers.facebook.com/docs/marketing-api)
- [n8n Documentation](https://docs.n8n.io/)
- [Google Sheets API](https://developers.google.com/sheets/api)

### Common Issues
See `migration-guide.md` for detailed troubleshooting

---

## License

Internal use only - Static Shift Agency

---

## Changelog

### 2025-11-26
- ✅ Created Facebook API setup guide
- ✅ Created credentials template
- ✅ Created migration guide
- ✅ Added .gitignore for security
- ✅ Documented complete system

---

**Last Updated**: November 26, 2025
**Maintained By**: Static Shift Team
