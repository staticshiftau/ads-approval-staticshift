# Corbel - Setup Information Template

## Fill this out as you complete the setup process

**Setup Date:** _______________
**Completed By:** _______________

---

## 1. Meta App Details

- **App Name:** Corbel - Ad Comment Monitor
- **App ID:** _______________
- **App Secret:** _______________ (Store securely)
- **App Mode:** [ ] Development [ ] Live

---

## 2. Facebook Page Information

- **Page Name:** _______________
- **Page ID:** _______________
- **Page URL:** https://facebook.com/_______________
- **Page Admin Email:** _______________

---

## 3. Ad Account Information

- **Ad Account Name:** _______________
- **Ad Account ID:** act_______________
  > Find this in Facebook Business Manager > Ad Accounts
- **Business Manager ID:** _______________

---

## 4. Access Tokens

### Page Access Token (for reading comments)
```
Token: EAAcEdar4GN8BP___________________________________
```
- **Generated Date:** _______________
- **Expires:** [ ] Never [ ] Long-lived (60 days) [ ] Short-lived (1-2 hours)
- **Used In:** "Get Comments for Ad Post" node

### Ad Account Access Token (for reading ads)
```
Token: EAAcEdar4GN8BP___________________________________
```
- **Generated Date:** _______________
- **Expires:** [ ] Never [ ] Long-lived (60 days) [ ] Short-lived (1-2 hours)
- **Used In:** "Get Active Ads" node

---

## 5. Testing Results

### Test 1: Get Active Ads
- **Date Tested:** _______________
- **Result:** [ ] Success [ ] Failed
- **Number of ads returned:** _______________
- **Notes:** _______________________________________________

### Test 2: Get Comments
- **Date Tested:** _______________
- **Result:** [ ] Success [ ] Failed
- **Test Post ID:** _______________
- **Number of comments returned:** _______________
- **Notes:** _______________________________________________

### Test 3: Full Workflow
- **Date Tested:** _______________
- **Result:** [ ] Success [ ] Failed
- **Slack notification received:** [ ] Yes [ ] No
- **Google Sheet updated:** [ ] Yes [ ] No [ ] N/A
- **Notes:** _______________________________________________

---

## 6. n8n Workflow Configuration

- **Workflow ID:** _______________
- **Workflow Name:** Facebook AD Comment Monitor | Corbel
- **Status:** [ ] Active [ ] Inactive
- **Schedule:** Every 15 minutes
- **Daily Summary Time:** 5:00 PM AEST (Weekdays)

---

## 7. Slack Integration

- **Slack Workspace:** Static Shift
- **Notification Channel:** #notifications-corbel
- **Channel ID:** _______________
- **Test Message Sent:** [ ] Yes [ ] No

---

## 8. Google Sheets (Optional)

- **Sheet Name:** _______________
- **Sheet ID:** _______________
- **Sheet URL:** https://docs.google.com/spreadsheets/d/_______________
- **Tab Name:** Comment Log
- **Access Granted To:** n8n service account

---

## 9. Permissions Granted

Check off each permission as it's granted in the Meta App:

- [ ] `pages_read_engagement` - Read comments on pages
- [ ] `pages_manage_posts` - Access to page posts
- [ ] `pages_read_user_content` - Read user-generated content
- [ ] `ads_read` - Read ads data
- [ ] `ads_management` - Manage ads

**Review Status:** [ ] Not submitted [ ] Pending [ ] Approved

---

## 10. Go-Live Checklist

- [ ] All tokens tested and working
- [ ] Workflow tested end-to-end
- [ ] Slack notifications working
- [ ] Google Sheet logging working (if applicable)
- [ ] Error notifications tested
- [ ] Daily summary tested
- [ ] Workflow activated
- [ ] Client notified
- [ ] Documentation updated
- [ ] Tokens stored in password manager

---

## 11. Support Information

**Primary Contact:** _______________
**Email:** _______________
**Phone:** _______________

**Escalation Contact:** _______________
**Email:** _______________

---

## 12. Notes / Issues Encountered

_______________________________________________
_______________________________________________
_______________________________________________
_______________________________________________

---

## 13. Maintenance Schedule

**Token Renewal Due:** _______________ (60 days from generation)
**Next Review Date:** _______________
**Last Updated:** _______________

---

**Template Version:** 1.0
**For Client:** Corbel
