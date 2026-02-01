# VAPI Booking Workflow Documentation

## Overview

This n8n workflow integrates with VAPI (Voice AI) to handle appointment booking via voice calls. It provides two main functions:
1. **Get Available Slots** - Returns available appointment times for a given date
2. **Book Appointment** - Creates a calendar event and sends confirmations

**Workflow ID:** `DuYsnUndTKZJlVak`  
**Workflow Name:** Dev -vapi-booking  
**n8n Instance:** https://jcr656.app.n8n.cloud

---

## Webhooks

### 1. Get Available Slots
- **URL:** `https://jcr656.app.n8n.cloud/webhook/vapi-slots`
- **Method:** POST
- **Purpose:** Returns available 30-minute appointment slots for a requested date

**VAPI Tool Call Format:**
```json
{
  "message": {
    "toolCalls": [{
      "id": "unique-tool-call-id",
      "function": {
        "name": "getslot_tool",
        "arguments": "{\"date\":\"2026-02-07\"}"
      }
    }]
  }
}
```

**Response Format:**
```json
{
  "results": [{
    "toolCallId": "unique-tool-call-id",
    "result": "Available slots on 2026-02-07: 09:00, 09:30, 10:00, ..."
  }]
}
```

### 2. Book Appointment
- **URL:** `https://jcr656.app.n8n.cloud/webhook/book_appointment`
- **Method:** POST
- **Purpose:** Books an appointment and sends confirmations

**VAPI Tool Call Format:**
```json
{
  "message": {
    "toolCalls": [{
      "id": "unique-tool-call-id",
      "function": {
        "name": "book_appointment",
        "arguments": "{\"name\":\"John Doe\",\"email\":\"john@example.com\",\"phone\":\"5551234567\",\"date\":\"2026-02-07\",\"time\":\"10:00\",\"title\":\"Consultation\",\"notes\":\"Optional notes\"}"
      }
    }]
  }
}
```

**Response Format:**
```json
{
  "results": [{
    "toolCallId": "unique-tool-call-id",
    "result": "Appointment booked for John Doe on 2026-02-07 at 10:00. Your Google Meet link is https://meet.google.com/xxx-xxxx-xxx. Confirmation email sent to john@example.com"
  }]
}
```

---

## Workflow Architecture

### Get Slots Flow
```
Get Slots Webhook → Parse Slots Request → Get Calendar Events → Build Available Slots → Respond Slots
```

### Booking Flow
```
Book Webhook → Parse Booking → Check Conflicts → Evaluate Conflicts → Has Conflict?
                                                                          ↓ (FALSE - No conflict)
                                                                    Create Calendar Event
                                                                          ↓
                                                                    Create GHL Contact
                                                                          ↓
                                                                    Create GHL Opportunity
                                                                          ↓
                                                                    SMS to Client
                                                                          ↓
                                                                    Prepare Emails
                                                                       ↓     ↓
                                                          Send Client Email  Send Owner Email
                                                                       ↓     ↓
                                                                    Respond Booked
```

---

## Node Details

### Critical Settings

| Node | Setting | Value | Purpose |
|------|---------|-------|---------|
| Get Calendar Events | Always Output Data | ✅ Enabled | Prevents workflow stop when calendar is empty |
| Check Conflicts | Always Output Data | ✅ Enabled | Prevents workflow stop when no conflicts exist |
| Has Conflict? | FALSE output | → Create Calendar Event | Books only when NO conflict |

### Integrations

1. **Google Calendar** (OAuth2)
   - Calendar: jcr656@gmail.com
   - Creates events with Google Meet links
   - Credential ID: `dz0wCOBvDc3HNrtl`

2. **GoHighLevel (GHL)**
   - Creates/finds contacts
   - Creates opportunities in pipeline
   - Sends SMS confirmations
   - Location ID: `Od0jTeQPHdqsdWqxxTRk`
   - Pipeline ID: `mdN8g8tNX04Urfe3GFMO`

3. **SMTP Email**
   - Sends client confirmation email
   - Sends owner notification email
   - From: jcr656@gmail.com

---

## Email Templates

### Client Email
```
Subject: Your Appointment Confirmation - [DATE] at [TIME]

Hi [NAME]!

Your appointment has been confirmed!

Date: [DATE]
Time: [TIME]

Join via Google Meet:
[MEET_LINK]

If you need to reschedule, please reply to this email or call us.

Thank you!

- JR Cloud Technologies
```

### Owner Email
```
Subject: NEW BOOKING: [NAME] - [DATE] [TIME]

NEW APPOINTMENT BOOKED!

Name: [NAME]
Phone: [PHONE]
Email: [EMAIL]
Date: [DATE]
Time: [TIME]
Notes: [NOTES]

Google Meet:
[MEET_LINK]

GHL: [CRM_LINK]
```

---

## Business Hours

- **Available Hours:** 9:00 AM - 5:00 PM (EST/UTC-5)
- **Slot Duration:** 30 minutes
- **Days:** All days (no weekend exclusion currently)

---

## Troubleshooting

### Common Issues

1. **404 Error on webhook**
   - Ensure workflow is activated (toggle ON in n8n)
   - Deactivate and reactivate to re-register webhooks

2. **Workflow stops at Check Conflicts**
   - Enable "Always Output Data" on the node
   - This allows empty results to pass through

3. **No emails sent**
   - Check SMTP credentials are configured
   - Verify email addresses in node parameters

4. **IF node routing wrong**
   - "Has Conflict?" FALSE output should connect to Create Calendar Event
   - TRUE output should be empty (or handle conflict response)

5. **VAPI arguments not parsing**
   - Arguments come as JSON string, must be parsed:
   ```javascript
   const argsRaw = toolCalls[0]?.function?.arguments || '{}';
   const args = typeof argsRaw === 'string' ? JSON.parse(argsRaw) : argsRaw;
   ```

---

## Testing

### PowerShell Test Command
```powershell
$body = @{
    message = @{
        toolCalls = @(
            @{
                id = "test-booking"
                function = @{
                    arguments = @{
                        name = "Test User"
                        email = "test@example.com"
                        phone = "5551234567"
                        date = "2026-02-07"
                        time = "10:00"
                        title = "Test Appointment"
                        notes = "Testing"
                    }
                }
            }
        )
    }
} | ConvertTo-Json -Depth 10

Invoke-WebRequest -Uri "https://jcr656.app.n8n.cloud/webhook/book_appointment" -Method POST -Body $body -ContentType "application/json"
```

### cURL Test Command
```bash
curl -X POST "https://jcr656.app.n8n.cloud/webhook/book_appointment" \
  -H "Content-Type: application/json" \
  -d '{"message":{"toolCalls":[{"id":"test","function":{"name":"book_appointment","arguments":"{\"name\":\"Test User\",\"email\":\"test@example.com\",\"phone\":\"5551234567\",\"date\":\"2026-02-07\",\"time\":\"10:00\"}"}}]}}'
```

---

## Version History

| Date | Change |
|------|--------|
| 2026-02-01 | Initial workflow creation |
| 2026-02-01 | Fixed VAPI argument parsing (JSON string) |
| 2026-02-01 | Added "Always Output Data" to calendar nodes |
| 2026-02-01 | Fixed IF node connection (FALSE → Create Calendar Event) |
| 2026-02-01 | Added client email node |

---

## Author

**JR Cloud Technologies**  
Created with assistance from Claude AI
