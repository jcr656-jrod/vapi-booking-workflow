# Configuration Reference

## Client Configuration Object

Each client deployment uses a configuration object at the start of the workflow. Update these values for each new client.

```javascript
const CLIENT_CONFIG = {
  // ===== BUSINESS INFO =====
  businessName: "Acme Dental",
  businessPhone: "555-123-4567",
  businessEmail: "info@acmedental.com",
  timezone: "America/New_York",
  
  // ===== BOOKING SETTINGS =====
  slotDuration: 30,              // minutes per appointment
  businessHoursStart: 9,         // 9 AM
  businessHoursEnd: 17,          // 5 PM
  workDays: [1, 2, 3, 4, 5],     // Mon-Fri (0=Sun, 6=Sat)
  
  // ===== GOOGLE CALENDAR =====
  calendarEmail: "calendar@acmedental.com",
  
  // ===== GOHIGHLEVEL (Optional) =====
  ghlEnabled: true,
  ghlApiKey: "pit-xxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
  ghlLocationId: "xxxxxxxxxxxxxxxx",
  ghlPipelineId: "xxxxxxxxxxxxxxxx",
  ghlStageId: "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
  
  // ===== SMS SETTINGS =====
  smsEnabled: true,
  
  // ===== EMAIL SETTINGS =====
  ownerEmail: "owner@acmedental.com",
  fromEmail: "bookings@acmedental.com",
  
  // ===== BRANDING =====
  emailSignature: "- Acme Dental Team",
  smsSignature: "- Acme Dental"
};
```

---

## Configuration Fields

### Business Info

| Field | Type | Description | Example |
|-------|------|-------------|---------|
| `businessName` | string | Display name for branding | "Acme Dental" |
| `businessPhone` | string | Business phone number | "555-123-4567" |
| `businessEmail` | string | Primary contact email | "info@acme.com" |
| `timezone` | string | IANA timezone | "America/New_York" |

### Booking Settings

| Field | Type | Description | Example |
|-------|------|-------------|---------|
| `slotDuration` | number | Minutes per appointment | 30 |
| `businessHoursStart` | number | Opening hour (24h) | 9 |
| `businessHoursEnd` | number | Closing hour (24h) | 17 |
| `workDays` | array | Days of week (0=Sun) | [1,2,3,4,5] |

### GoHighLevel (Optional)

| Field | Type | Description |
|-------|------|-------------|
| `ghlEnabled` | boolean | Enable GHL integration |
| `ghlApiKey` | string | Private Integration Token |
| `ghlLocationId` | string | GHL Location ID |
| `ghlPipelineId` | string | Pipeline for opportunities |
| `ghlStageId` | string | Initial stage ID |

### Email & SMS

| Field | Type | Description |
|-------|------|-------------|
| `ownerEmail` | string | Where owner notifications go |
| `fromEmail` | string | Sender address for emails |
| `smsEnabled` | boolean | Enable SMS confirmations |
| `emailSignature` | string | Sign-off for emails |
| `smsSignature` | string | Sign-off for SMS |

---

## Credential Setup

### 1. Google Calendar OAuth

1. Go to n8n Credentials
2. Add new "Google Calendar OAuth2"
3. Client must authorize their Google account
4. Update `calendarEmail` in config

### 2. SMTP Email

1. Go to n8n Credentials
2. Add new "SMTP"
3. Configure:
   - Host: smtp.gmail.com (or client's provider)
   - Port: 465 (SSL) or 587 (TLS)
   - User: Client's email
   - Password: App password

### 3. GoHighLevel API

1. In GHL: Settings → Integrations → API Keys
2. Create "Private Integration" token
3. Copy token to `ghlApiKey`
4. Get Location ID from URL: `app.gohighlevel.com/location/{LOCATION_ID}`

---

## Environment-Specific URLs

After deploying workflow, update VAPI with these webhook URLs:

```
Get Slots:  https://[n8n-instance]/webhook/[client-slug]-slots
Book:       https://[n8n-instance]/webhook/[client-slug]-book
```

Example for "Acme Dental":
```
https://n8n.yourcompany.com/webhook/acme-dental-slots
https://n8n.yourcompany.com/webhook/acme-dental-book
```
