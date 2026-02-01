# VAPI Integration Guide

## Overview

VAPI (Voice AI) handles the phone calls and conversations. This workflow provides the backend tools that VAPI calls to check availability and book appointments.

---

## How It Works

```
Customer calls → VAPI answers → Customer asks for appointment
                                         ↓
                              VAPI calls get_slots tool
                                         ↓
                              n8n returns available times
                                         ↓
                              VAPI reads times to customer
                                         ↓
                              Customer picks a time
                                         ↓
                              VAPI calls book_appointment tool
                                         ↓
                              n8n books & sends confirmations
                                         ↓
                              VAPI confirms to customer
```

---

## VAPI Tool Configuration

### Tool 1: Get Available Slots

**Purpose:** Returns available appointment times for a given date.

```json
{
  "type": "function",
  "function": {
    "name": "get_available_slots",
    "description": "Check available appointment slots for a specific date. Call this when the customer wants to know what times are available.",
    "parameters": {
      "type": "object",
      "properties": {
        "date": {
          "type": "string",
          "description": "The date to check availability for, in YYYY-MM-DD format. For example: 2026-02-15"
        }
      },
      "required": ["date"]
    }
  },
  "server": {
    "url": "https://your-n8n.com/webhook/client-slots",
    "method": "POST",
    "headers": {
      "Content-Type": "application/json"
    }
  }
}
```

### Tool 2: Book Appointment

**Purpose:** Creates the appointment and sends confirmations.

```json
{
  "type": "function",
  "function": {
    "name": "book_appointment",
    "description": "Book an appointment for the customer. Only call this after confirming the date, time, and collecting customer information.",
    "parameters": {
      "type": "object",
      "properties": {
        "name": {
          "type": "string",
          "description": "Customer's full name"
        },
        "email": {
          "type": "string",
          "description": "Customer's email address for confirmation"
        },
        "phone": {
          "type": "string",
          "description": "Customer's phone number"
        },
        "date": {
          "type": "string",
          "description": "Appointment date in YYYY-MM-DD format"
        },
        "time": {
          "type": "string",
          "description": "Appointment time in HH:MM format (24-hour)"
        },
        "title": {
          "type": "string",
          "description": "Type of appointment (e.g., Consultation, Checkup)"
        },
        "notes": {
          "type": "string",
          "description": "Any additional notes or special requests"
        }
      },
      "required": ["name", "phone", "date", "time"]
    }
  },
  "server": {
    "url": "https://your-n8n.com/webhook/client-book",
    "method": "POST",
    "headers": {
      "Content-Type": "application/json"
    }
  }
}
```

---

## VAPI Assistant Prompt Template

Use this prompt for the VAPI assistant:

```
You are a friendly receptionist for {{businessName}}. Your job is to help callers schedule appointments.

GUIDELINES:
- Be warm, professional, and conversational
- Ask for the customer's preferred date first
- Use the get_available_slots tool to check availability
- Present 3-4 time options at a time
- Once they choose a time, collect: name, email (optional), and phone number
- Use the book_appointment tool to finalize
- Confirm the appointment details before ending the call

BUSINESS HOURS:
{{businessHours}}

APPOINTMENT TYPES:
{{appointmentTypes}}

EXAMPLE CONVERSATION:
Caller: "Hi, I'd like to schedule an appointment."
You: "Of course! I'd be happy to help you schedule an appointment. What date works best for you?"
Caller: "How about next Tuesday?"
You: "Let me check our availability for Tuesday..." [calls get_available_slots]
You: "I have openings at 10:00 AM, 11:30 AM, and 2:00 PM. Which works best for you?"
...
```

---

## VAPI Request Format

When VAPI calls your webhooks, it sends this structure:

```json
{
  "message": {
    "toolCalls": [
      {
        "id": "call_abc123",
        "function": {
          "name": "book_appointment",
          "arguments": "{\"name\":\"John Doe\",\"phone\":\"5551234567\",\"date\":\"2026-02-07\",\"time\":\"10:00\"}"
        }
      }
    ],
    "call": {
      "customer": {
        "number": "+15551234567"
      }
    }
  }
}
```

**Important:** The `arguments` field is a JSON **string**, not an object. The workflow parses this automatically.

---

## Expected Response Format

Your webhook must return this exact format:

```json
{
  "results": [
    {
      "toolCallId": "call_abc123",
      "result": "Appointment booked for John Doe on 2026-02-07 at 10:00. Confirmation sent."
    }
  ]
}
```

The `toolCallId` must match the incoming `id` from the request.

---

## VAPI Phone Numbers

### Getting a Number

1. Go to VAPI Dashboard → Phone Numbers
2. Click "Buy Number" or "Import Number"
3. Choose area code matching client's location
4. Assign to the client's assistant

### Forwarding Fallback

Configure fallback to client's main number if:
- AI can't understand caller
- Caller says "operator" or "real person"
- Technical issues occur

---

## Testing VAPI Integration

### Test via VAPI Dashboard

1. Open assistant in VAPI
2. Click "Test Call" button
3. Go through booking flow
4. Check n8n executions

### Test via Phone

1. Call the VAPI number
2. Request an appointment
3. Complete the booking
4. Verify all confirmations sent

### Test Webhooks Directly

```bash
curl -X POST "https://your-n8n.com/webhook/client-book" \
  -H "Content-Type: application/json" \
  -d '{
    "message": {
      "toolCalls": [{
        "id": "test-123",
        "function": {
          "name": "book_appointment",
          "arguments": "{\"name\":\"Test\",\"email\":\"test@test.com\",\"phone\":\"5551234567\",\"date\":\"2026-02-07\",\"time\":\"10:00\"}"
        }
      }]
    }
  }'
```

---

## Common VAPI Issues

| Issue | Cause | Fix |
|-------|-------|-----|
| Tool returns 404 | Workflow not active | Activate workflow in n8n |
| "No result returned" | Webhook timeout | Check n8n execution logs |
| Arguments not parsed | Wrong JSON format | Ensure args are JSON string |
| Wrong times offered | Timezone mismatch | Set correct timezone in config |
