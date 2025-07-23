# 5-Minute Integration Guide

Get up and running with the OpenMoove Partner API in just 5 minutes. This guide walks you through everything from API key setup to creating your first property transaction.

---

## ⏱️ What You'll Accomplish

By the end of this guide, you'll have:
- ✅ Generated and tested your API key
- ✅ Listed available products
- ✅ Created a complete property transaction
- ✅ Retrieved property details and milestones
- ✅ Understanding of next steps for full integration

**Time Required:** 5 minutes  
**Prerequisites:** None (we'll cover everything)

---

## 🎯 Step 1: Get Your API Key (1 minute)

### Generate Your Key
1. **Visit the staging environment:** [staging.openmoove.com](https://staging.openmoove.com)
2. **Sign up or log in** to your account
3. **Navigate to Profile → Integrations**
4. **Click "Generate API Key"** or "Create New Key"
5. **Copy your API key** (format: `om_xxxx`)

### Test Your Key
Verify your API key works:

```bash
curl -H "X-API-KEY: om_your_key_here" \
  "https://staging.openmoove.com/api/v1/partners/properties/"
```

**Expected Response:**
```json
{
  "count": 0,
  "next": null,
  "previous": null,
  "results": []
}
```

If you see this response, you're ready to proceed! 🎉

---

## 📦 Step 2: Explore Available Products (1 minute)

Before creating properties, let's see what products are available:

```bash
curl -H "X-API-KEY: om_your_key_here" \
  "https://staging.openmoove.com/api/v1/partners/products/"
```

**Response:**
```json
{
  "count": 1,
  "results": [
    {
      "name": "Moove Ready Pack",
      "slug": "moove-ready-pack",
      "description": "",
      "price": {
        "amount": 0,
        "currency": "GBP"
      }
    }
  ]
}
```

**Note the `slug`:** You'll use `"moove-ready-pack"` when creating properties.

---

## 🏠 Step 3: Create Your First Property (1 minute)

Now for the main event - create a complete property transaction:

```bash
curl -X POST \
  -H "Content-Type: application/json" \
  -H "X-API-KEY: om_your_key_here" \
  -d '{
    "first_name": "John",
    "last_name": "Smith",
    "email": "john.smith+test@example.com",
    "phone_number": "+447700900001",
    "property_address": "123 Integration Street, London",
    "property_postcode": "SW1A 1AA",
    "sale_method": "standard",
    "estate_agent_email": "agent@youragency.com",
    "products": ["moove-ready-pack"]
  }' \
  "https://staging.openmoove.com/api/v1/partners/clients/"
```

**Success Response:**
```json
{
  "property_id": 100,
  "mover_id": 68,
  "message": "Client and property created successfully",
  "property": {
    "transactionId": "4fa18bac-3c62-4d92-a271-7364b0f0e479",
    "status": "For sale",
    "participants": [
      {
        "name": {
          "firstName": "John",
          "lastName": "Smith"
        },
        "email": "john.smith+test@example.com",
        "phone": "+447700900001",
        "role": "Seller"
      }
    ],
    "propertyPack": {
      "priceInformation": {
        "disposal": "Standard Sale"
      },
      "address": {
        "line1": "123 Integration Street, London",
        "postcode": "SW1A 1AA",
        "countryCode": "GBR"
      }
    }
  },
  "estate_agent_id": 100,
  "purchases": [
    {
      "id": 34,
      "product_name": "Moove Ready Pack",
      "amount": "0",
      "payment_status": "pending"
    }
  ]
}
```

**🎉 Congratulations!** You just created:
- A new client (John Smith)
- A property transaction
- Professional team assignment (estate agent)
- Product purchase (Moove Ready Pack)
- Automatic milestone setup

**Copy the `transactionId`** - you'll need it for the next steps.

---

## 📊 Step 4: Explore Your Property (1 minute)

### View Property Details
Use the `transactionId` from Step 3:

```bash
curl -H "X-API-KEY: om_your_key_here" \
  "https://staging.openmoove.com/api/v1/partners/properties/4fa18bac-3c62-4d92-a271-7364b0f0e479/"
```

### Check Milestones
See the automated milestone tracking:

```bash
curl -H "X-API-KEY: om_your_key_here" \
  "https://staging.openmoove.com/api/v1/partners/properties/4fa18bac-3c62-4d92-a271-7364b0f0e479/milestones/"
```

**Milestone Response:**
```json
[
  {
    "id": 1,
    "name": "Instruction",
    "description": "Property instruction received from seller",
    "order": 1,
    "completed": true,
    "completed_at": "2024-01-15T10:00:00Z"
  },
  {
    "id": 2,
    "name": "Legal Pack Preparation",
    "description": "Legal documentation and property pack preparation",
    "order": 2,
    "completed": false,
    "due_date": "2024-01-20T23:59:59Z"
  }
]
```

### List All Your Properties
```bash
curl -H "X-API-KEY: om_your_key_here" \
  "https://staging.openmoove.com/api/v1/partners/properties/"
```

---

## 🚀 Step 5: Try Advanced Features (1 minute)

### Create an Auction Property
Modern auction properties require an auctioneer:

```bash
curl -X POST \
  -H "Content-Type: application/json" \
  -H "X-API-KEY: om_your_key_here" \
  -d '{
    "first_name": "Jane",
    "last_name": "Auction",
    "email": "jane.auction+test@example.com",
    "phone_number": "+447700900002",
    "property_address": "456 Auction Avenue, Manchester",
    "property_postcode": "M1 2AB",
    "sale_method": "modern_auction",
    "estate_agent_email": "agent@youragency.com",
    "auctioneer_email": "auctioneer@bidhouse.com",
    "products": ["moove-ready-pack"]
  }' \
  "https://staging.openmoove.com/api/v1/partners/clients/"
```

**Notice the differences:**
- `sale_method`: `"modern_auction"`
- `auctioneer_email`: Required for auction sales
- Property pack will show `"disposal": "Modern Method of Auction"`

---

## ✅ What You've Learned

In just 5 minutes, you've:

### 🔑 **API Basics**
- Generated and tested your API key
- Understanding staging vs production environments
- Basic request/response format

### 🏠 **Property Creation**
- Created complete property transactions
- Added professional teams automatically
- Integrated product purchases
- Different sale methods (standard vs auction)

### 📊 **Data Retrieval**
- Listed properties
- Retrieved detailed property information
- Accessed milestone tracking
- PDTF-compliant data structure

### 🎯 **Key Features**
- One-call property creation
- Automatic professional invitation
- Built-in milestone tracking
- Product integration

---

## 🔄 Common Variations

### Different Professional Teams
```json
{
  "conveyancer_email": "lawyer@lawfirm.com",
  "mortgage_broker_email": "broker@mortgages.com"
}
```

### Multiple Products
```json
{
  "products": ["moove-ready-pack", "additional-service"]
}
```

### International Phone Numbers
```json
{
  "phone_number": "+33123456789"  // French number
}
```

---

## 🚨 Common Issues & Solutions

### ❌ "Invalid API key provided"
**Solution:** 
- Check key format starts with `om_`
- Verify key was copied correctly
- Ensure you're using staging environment

### ❌ "At least one professional email must be provided"
**Solution:**
- Add `estate_agent_email`, `conveyancer_email`, or other professional email
- For auctions, `auctioneer_email` is required

### ❌ "Invalid phone number format"
**Solution:**
- Use international format: `+447700900123`
- For testing, use numbers ending in 001-010

### ❌ Property not found
**Solution:**
- Use the full `transactionId` UUID from the creation response
- Ensure you have access to the property (you created it or are on the team)

---

## 🎯 Next Steps

### 🔧 **For Integration**
Choose your development path:

**[Python Implementation](../examples/python/)** - Complete Python SDK
```python
from openmoove import OpenMooveAPI
api = OpenMooveAPI("om_your_key")
result = api.create_client({...})
```

**[JavaScript Implementation](../examples/javascript/)** - Node.js and browser
```javascript
const api = new OpenMooveAPI("om_your_key");
const result = await api.createClient({...});
```

**[cURL Examples](../examples/curl/)** - All endpoints with copy-paste commands

### 📚 **For Deep Understanding**
**[Integration Patterns](integration-patterns.md)** - Real-world workflows
- Standard property sales process
- Auction property workflows
- Professional team management
- Error handling strategies

### 🔧 **For Advanced Features**
**[Webhook Implementation](../api-reference/webhooks.md)** - Real-time updates
**[PDTF Compliance](../technical/pdtf-compliance.md)** - Property data standards

### 🧪 **For Testing**
**[Postman Collection](../examples/postman/)** - Interactive API testing

---

## 🆘 Need Help?

### 📧 Support
**Email:** api-support@openmoove.com  
**Response Time:** Within 24 hours

### 📋 Include in Support Requests
- API endpoint used
- Request payload (remove sensitive data)
- Response received
- Expected behavior
- Steps you tried

### 🔍 Self-Help Resources
- **[Complete API Reference](../api-reference/endpoints.md)** - Every endpoint documented
- **[Error Handling Guide](../api-reference/errors.md)** - Troubleshooting guide
- **[Authentication Guide](../api-reference/authentication.md)** - Security best practices

---

## 🎉 Congratulations!

You've successfully integrated with the OpenMoove Partner API! You're now ready to:
- Build property transaction workflows into your platform
- Automate client onboarding and team setup
- Track transaction progress with milestone data
- Integrate with the broader property ecosystem using PDTF standards

**Ready to go deeper? Choose your next step above and keep building!** 🚀