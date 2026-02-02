# Pizzeria Ferrante - Voice AI Order System

Voice AI phone ordering system for Pizzeria Ferrante, Saint Cloud, FL.

## Overview

- **AI Name:** Tony
- **Function:** Takes phone orders for pizza, entrees, subs, appetizers
- **Sends orders to:** Email (kitchen + owner)
- **Future:** POS integration

## Files

| File | Description |
|------|-------------|
| `vapi-assistant.json` | VAPI assistant configuration with full menu knowledge |
| `order-workflow.json` | n8n workflow for processing orders |
| `menu.md` | Complete menu reference |

## Setup Instructions

### 1. VAPI Setup

1. Go to [vapi.ai](https://vapi.ai) dashboard
2. Create new assistant
3. Import `vapi-assistant.json`
4. Add tool webhook URL (from n8n)

### 2. n8n Workflow Setup

1. Import `order-workflow.json` into n8n
2. Update SMTP credentials for email sending
3. Set email addresses:
   - Kitchen: `kitchen@pizzeriaferrante.com`
   - Owner: `owner@pizzeriaferrante.com`
4. Activate workflow
5. Copy webhook URL

### 3. Connect VAPI to n8n

In VAPI assistant settings, set the tool server URL to your n8n webhook:
```
https://your-n8n.app.n8n.cloud/webhook/ferrante-order
```

## Order Flow

```
Customer calls → VAPI answers as "Tony"
     ↓
Tony takes order, confirms items & total
     ↓
Tony collects: name, phone, pickup/delivery
     ↓
submit_order tool called → n8n webhook
     ↓
n8n formats order → sends email to kitchen & owner
     ↓
Tony confirms order # and estimated time
```

## Sample Order Email

```
═══════════════════════════════════════════
       PIZZERIA FERRANTE - NEW ORDER
═══════════════════════════════════════════

ORDER #: PF-M5X7K2
TIME: 2/2/2026, 6:45:32 PM

───────────────────────────────────────────
CUSTOMER INFO
───────────────────────────────────────────
Name: John Smith
Phone: +14075551234
Order Type: DELIVERY
Delivery Address: 123 Oak Street, Saint Cloud, FL 34772

───────────────────────────────────────────
ORDER ITEMS
───────────────────────────────────────────
1x Large Meat Lovers Pie - $28
1x Garlic Twists (12) - $8
1x 2-Liter Coke - $3

───────────────────────────────────────────
ORDER TOTAL: $39.00
───────────────────────────────────────────

ESTIMATED TIME: 35-45 minutes

═══════════════════════════════════════════
```

## Future Enhancements

- [ ] SMS confirmation to customer (via GHL or Twilio)
- [ ] POS integration (Square, Toast, etc.)
- [ ] Google Sheets order log
- [ ] Delivery zone validation
- [ ] Online payment integration

## Business Info

**Pizzeria Ferrante**  
2907 Canoe Creek Rd  
Saint Cloud, FL 34772  
Phone: 407-556-3305

---

*Built with VAPI + n8n*
