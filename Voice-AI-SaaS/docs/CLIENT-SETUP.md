# Client Setup Guide

## Overview

This guide walks through deploying Voice-AI SaaS for a new client. Total setup time: ~30 minutes.

---

## Pre-Requisites

Collect from client before starting:

- [ ] Business name
- [ ] Business phone number
- [ ] Owner email address
- [ ] Google account for calendar
- [ ] GoHighLevel account (if using CRM)
- [ ] Business hours
- [ ] Appointment duration preference

---

## Step 1: Create Workflow Copy

1. Open n8n dashboard
2. Go to workflow template
3. Click **⋮ Menu** → **Duplicate**
4. Rename to: `[Client Name] - Voice AI Booking`

---

## Step 2: Configure Client Settings

1. Open the duplicated workflow
2. Find the **"Client Config"** code node
3. Update all values:

```javascript
const CLIENT_CONFIG = {
  businessName: "CLIENT BUSINESS NAME",
  businessPhone: "CLIENT PHONE",
  businessEmail: "client@email.com",
  timezone: "America/New_York",
  
  slotDuration: 30,
  businessHoursStart: 9,
  businessHoursEnd: 17,
  workDays: [1, 2, 3, 4, 5],
  
  calendarEmail: "client-calendar@gmail.com",
  
  ghlEnabled: true,  // or false
  ghlApiKey: "pit-xxxxx",
  ghlLocationId: "xxxxx",
  ghlPipelineId: "xxxxx",
  ghlStageId: "xxxxx",
  
  smsEnabled: true,
  
  ownerEmail: "owner@client.com",
  fromEmail: "bookings@client.com",
  
  emailSignature: "- Client Business Team",
  smsSignature: "- Client Business"
};
```

---

## Step 3: Set Up Credentials

### Google Calendar

1. Click the **Get Calendar Events** node
2. Click **Credential** dropdown → **Create New**
3. Name it: `[Client Name] - Google Calendar`
4. Click **Sign in with Google**
5. Have client authorize their Google account
6. Apply same credential to **Check Conflicts** and **Create Calendar Event** nodes

### SMTP Email

1. Click the **Send Client Email** node
2. Click **Credential** dropdown → **Create New**
3. Name it: `[Client Name] - SMTP`
4. Configure:
   - **Host:** smtp.gmail.com
   - **Port:** 465
   - **Secure:** ON
   - **User:** client's email
   - **Password:** app password (not regular password)
5. Apply same credential to **Send Owner Email** node

### GoHighLevel (If Using)

1. Have client create Private Integration in GHL
2. Copy API key to `ghlApiKey` in config
3. Get Location ID from GHL URL
4. Find Pipeline ID and Stage ID in GHL settings

---

## Step 4: Update Webhook Paths

1. Click **Get Slots Webhook** node
2. Change path to: `[client-slug]-slots`
   - Example: `acme-dental-slots`
3. Click **Book Webhook** node
4. Change path to: `[client-slug]-book`
   - Example: `acme-dental-book`

---

## Step 5: Activate Workflow

1. Click the **Active** toggle (top right)
2. Confirm webhooks are registered
3. Note the webhook URLs

---

## Step 6: Configure VAPI

1. Log into VAPI dashboard
2. Open client's assistant (or create new)
3. Add tools:

### Get Slots Tool
```json
{
  "name": "get_available_slots",
  "description": "Get available appointment slots for a specific date",
  "parameters": {
    "type": "object",
    "properties": {
      "date": {
        "type": "string",
        "description": "Date in YYYY-MM-DD format"
      }
    },
    "required": ["date"]
  },
  "server": {
    "url": "https://[n8n-instance]/webhook/[client-slug]-slots"
  }
}
```

### Book Appointment Tool
```json
{
  "name": "book_appointment",
  "description": "Book an appointment for the customer",
  "parameters": {
    "type": "object",
    "properties": {
      "name": {
        "type": "string",
        "description": "Customer's full name"
      },
      "email": {
        "type": "string",
        "description": "Customer's email address"
      },
      "phone": {
        "type": "string",
        "description": "Customer's phone number"
      },
      "date": {
        "type": "string",
        "description": "Appointment date (YYYY-MM-DD)"
      },
      "time": {
        "type": "string",
        "description": "Appointment time (HH:MM)"
      },
      "notes": {
        "type": "string",
        "description": "Any additional notes"
      }
    },
    "required": ["name", "phone", "date", "time"]
  },
  "server": {
    "url": "https://[n8n-instance]/webhook/[client-slug]-book"
  }
}
```

---

## Step 7: Test

1. Call the VAPI phone number
2. Request an appointment
3. Verify:
   - [ ] Calendar event created
   - [ ] Customer received SMS
   - [ ] Customer received email
   - [ ] Owner received notification
   - [ ] GHL contact created (if enabled)

---

## Step 8: Handoff to Client

Provide client with:
- [ ] VAPI phone number
- [ ] How to check appointments in Google Calendar
- [ ] How to view leads in GoHighLevel
- [ ] Support contact info
- [ ] Invoice/billing setup

---

## Client Onboarding Checklist

```
□ Workflow duplicated and renamed
□ Client config updated
□ Google Calendar credential created
□ SMTP credential created
□ GHL API configured (if using)
□ Webhook paths customized
□ Workflow activated
□ VAPI tools configured
□ Test call successful
□ Client trained on dashboard
□ Billing set up
```
