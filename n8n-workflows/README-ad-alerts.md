# N8N Ad Alert Workflows

## Zero Leads Threshold Alert

**File:** `hrao-ad-alert-zero-leads.json`

### Purpose
Monitors active ads in real-time and sends Slack alerts when an ad spends above a threshold with 0 leads. This allows you to pause underperforming ads quickly.

### Features
- ✅ Runs every 3 hours during business hours (9am-5pm AEDT)
- ✅ Checks only ACTIVE ads
- ✅ Configurable spend threshold per client
- ✅ Direct link to Ads Manager for quick pausing
- ✅ Shows key metrics (spend, impressions, clicks, CPC)
- ✅ Prevents false alerts on new ads (min impressions filter)

### How to Use

#### 1. Import Workflow
- Copy the JSON from `hrao-ad-alert-zero-leads.json`
- In n8n, click **Import from File** or paste JSON
- Save the workflow

#### 2. Configure for Your Client
Edit the **"Load Client Config"** node:

```javascript
const config = {
  clientName: 'HR Advice Online',           // Change per client
  fbAdAccountId: 'act_1504596024142925',    // Change per client
  slackChannelId: 'C095KT33NNN',            // Change per client

  // ⚠️ MAIN SETTING - Adjust per client
  spendThresholdWithoutLead: 50,  // Alert at $50 spend with 0 leads

  // Optional: Prevent alerts on brand new ads
  minImpressionsBeforeAlert: 500  // Wait until ad has 500+ impressions
};
```

#### 3. Adjust Schedule (Optional)
The workflow runs every 3 hours from 9am-5pm AEDT by default.

To change, edit the **"Check Every 3 Hours"** node:
- `0 */3 9-17 * * *` = Every 3 hours, 9am-5pm AEDT
- `0 */2 9-17 * * *` = Every 2 hours, 9am-5pm AEDT
- `0 */4 9-17 * * *` = Every 4 hours, 9am-5pm AEDT

#### 4. Activate Workflow
- Click **Active** toggle in n8n
- The workflow will now monitor ads automatically

### Duplicating for Other Clients

1. **Duplicate the workflow** in n8n (right-click → Duplicate)
2. **Rename** it (e.g., "Client XYZ - Zero Leads Alert")
3. **Update the config** in "Load Client Config" node:
   - Change `clientName`
   - Change `fbAdAccountId`
   - Change `slackChannelId`
   - Adjust `spendThresholdWithoutLead` (e.g., $30, $75, $100)
4. **Activate** the new workflow

### Example Alert

```
⚠️ URGENT AD ALERT - Zero Leads!

Client: HR Advice Online
Time: Today at 2:30 PM

2 Active Ads Need Immediate Attention:

1. [Employment Law Services] Download Free Guide
🚨 Spent $52.30 with 0 leads (Threshold: $50)
📉 3,245 impressions | 87 clicks | CPC: $0.60
🔗 View & Pause Ad in Ads Manager

2. [HR Compliance] Free Consultation Offer
🚨 Spent $68.15 with 0 leads (Threshold: $50)
📉 4,512 impressions | 103 clicks | CPC: $0.66
🔗 View & Pause Ad in Ads Manager

✅ Recommended Action: Review and pause underperforming ads immediately.
```

### Settings by Client Type

**Conservative clients** (pause quickly):
- `spendThresholdWithoutLead: 30`
- Check every 2 hours

**Standard clients**:
- `spendThresholdWithoutLead: 50`
- Check every 3 hours

**High-budget clients** (more patience):
- `spendThresholdWithoutLead: 100`
- Check every 4 hours

### Notes
- Only monitors **ACTIVE** ads (paused/completed ads are ignored)
- Uses today's data only (lifetime stats would trigger false alerts)
- Includes minimum impressions filter to avoid alerting on brand new ads
- Direct links to Ads Manager for immediate action
