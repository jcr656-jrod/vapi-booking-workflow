# Troubleshooting Guide

## Quick Diagnostics

### Check Workflow Status
1. Is the workflow **Active**? (toggle should be ON)
2. Check recent executions for errors
3. Look at execution details to see which node failed

---

## Common Issues

### 1. 404 Error on Webhook

**Symptoms:**
- VAPI shows "Tool call failed"
- curl returns 404 Not Found

**Causes & Fixes:**

| Cause | Fix |
|-------|-----|
| Workflow not active | Click Active toggle ON |
| Wrong webhook path | Check path in webhook node matches VAPI config |
| n8n instance down | Check n8n server status |

**Quick Fix:**
1. Deactivate workflow
2. Wait 5 seconds
3. Reactivate workflow

---

### 2. Workflow Stops at Calendar Node

**Symptoms:**
- Execution shows "success" but only 2-3 nodes ran
- No calendar event created
- No emails sent

**Cause:** Calendar node returns empty when no events exist, stopping the flow.

**Fix:**
1. Click the **Get Calendar Events** node
2. Go to **Settings**
3. Enable **"Always Output Data"**
4. Repeat for **Check Conflicts** node
5. Save and reactivate

---

### 3. IF Node Going Wrong Direction

**Symptoms:**
- Works when there ARE conflicts
- Fails when there are NO conflicts (normal booking)

**Cause:** "Create Calendar Event" connected to TRUE output instead of FALSE.

**Fix:**
1. Open workflow editor
2. Click the connection from "Has Conflict?" to "Create Calendar Event"
3. Delete it
4. Draw new connection from **bottom output** (FALSE) to "Create Calendar Event"
5. Save and reactivate

---

### 4. Emails Not Sending

**Symptoms:**
- Workflow completes
- Calendar event created
- No emails received

**Checks:**

| Check | How |
|-------|-----|
| SMTP credentials | Open email node → Test credential |
| Spam folder | Check spam/junk folder |
| From address | Must be authorized to send from that address |
| Gmail App Password | Can't use regular password; need App Password |

**Gmail App Password Setup:**
1. Go to Google Account → Security
2. Enable 2-Factor Authentication
3. Go to App Passwords
4. Generate password for "Mail"
5. Use this in n8n SMTP credential

---

### 5. VAPI Arguments Not Parsing

**Symptoms:**
- Booking fails with empty values
- Name, email, phone are blank

**Cause:** VAPI sends arguments as JSON string, not object.

**Fix in Parse Booking node:**
```javascript
// WRONG
const args = toolCalls[0]?.function?.arguments || {};

// CORRECT
const argsRaw = toolCalls[0]?.function?.arguments || '{}';
const args = typeof argsRaw === 'string' ? JSON.parse(argsRaw) : argsRaw;
```

---

### 6. Wrong Timezone / Times

**Symptoms:**
- Slots show wrong hours
- Calendar events at wrong time

**Cause:** Timezone mismatch between config and calendar.

**Fix:**
1. Check `timezone` in CLIENT_CONFIG
2. Use IANA format: `America/New_York`, not `EST`
3. Ensure Google Calendar timezone matches

---

### 7. GHL Contact Not Created

**Symptoms:**
- Appointment booked successfully
- No contact in GoHighLevel

**Checks:**

| Check | How |
|-------|-----|
| ghlEnabled | Set to `true` in config |
| API Key | Valid Private Integration token |
| Location ID | Correct location ID from URL |
| API Version | Must include `Version: 2021-07-28` header |

**Test GHL API:**
```bash
curl -X GET "https://services.leadconnectorhq.com/contacts/?locationId=YOUR_LOCATION_ID" \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -H "Version: 2021-07-28"
```

---

### 8. SMS Not Sending

**Symptoms:**
- Everything else works
- No SMS to customer

**Causes:**

| Cause | Fix |
|-------|-----|
| smsEnabled is false | Set to `true` in config |
| No phone number | Ensure phone is captured |
| No GHL contact | SMS requires GHL contactId |
| GHL SMS not configured | Set up Twilio in GHL settings |

---

### 9. Webhook Timeout

**Symptoms:**
- VAPI shows timeout error
- Workflow still running in background

**Cause:** Workflow takes too long (>30 seconds)

**Fixes:**
1. Check for slow API calls (GHL, Calendar)
2. Move non-critical tasks to after response
3. Use webhook "Respond Immediately" option

---

## Debugging Steps

### Step 1: Check Execution Log
1. Go to workflow → Executions
2. Click failed execution
3. Find red node (error)
4. Click to see error message

### Step 2: Test Individual Nodes
1. Open workflow editor
2. Click node
3. Click "Execute Node"
4. Check output/errors

### Step 3: Check Credentials
1. Go to n8n Settings → Credentials
2. Click credential
3. Click "Test" button

### Step 4: Test Webhook Manually
```bash
curl -X POST "https://your-n8n.com/webhook/path" \
  -H "Content-Type: application/json" \
  -d '{"test": "data"}'
```

---

## Error Messages Reference

| Error | Meaning | Fix |
|-------|---------|-----|
| `Cannot read property 'json' of undefined` | Previous node returned nothing | Enable "Always Output Data" |
| `401 Unauthorized` | Bad API key | Check GHL/Calendar credentials |
| `ECONNREFUSED` | Can't reach external service | Check internet/service status |
| `Workflow could not be activated` | Invalid workflow config | Check all nodes have valid settings |
| `No result returned` | Empty response to VAPI | Ensure Respond node is connected |

---

## Getting Help

1. Check n8n execution logs
2. Review this troubleshooting guide
3. Check n8n community forum
4. Contact Voice-AI SaaS support
