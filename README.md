# OpenMoove Partner API 🏠

**The fastest way to integrate property transactions into your platform.**

Perfect for auctioneers, estate agents, conveyancers, and mortgage brokers who want to streamline property transactions with automated milestone tracking, professional collaboration, and PDTF-compliant data exchange.

---

## 🚀 30-Second Quick Start

**1. Get your API key:**
Visit [staging.openmoove.com/profile/integrations/](https://staging.openmoove.com/profile/integrations/) and copy your key (format: `om_xxxx`)

**2. Test your connection:**
```bash
curl -H "X-API-KEY: om_your_key_here" \
  "https://staging.openmoove.com/api/v1/partners/properties/"
```

**3. Create your first property:**
```bash
curl -X POST \
  -H "Content-Type: application/json" \
  -H "X-API-KEY: om_your_key_here" \
  -d '{
    "first_name": "John",
    "last_name": "Smith",
    "email": "john.smith@example.com",
    "phone_number": "+447700900123",
    "property_address": "123 Example Street, London",
    "property_postcode": "SW1A 1AA",
    "estate_agent_email": "agent@youragency.com",
    "products": ["moove-ready-pack"]
  }' \
  "https://staging.openmoove.com/api/v1/partners/clients/"
```

**That's it!** ✅ You just created a property transaction with automatic professional team setup and milestone tracking.

---

## 📚 What's Next?

### 🎯 For Immediate Integration
- **[5-Minute Integration Guide](guides/quick-start.md)** - Complete step-by-step tutorial
- **[Copy-Paste Examples](examples/)** - Ready-to-use code in Python, JavaScript, cURL
- **[Postman Collection](examples/postman/)** - Interactive API testing

### 📖 For Comprehensive Understanding
- **[Complete API Reference](api-reference/endpoints.md)** - Every endpoint documented
- **[Authentication Guide](api-reference/authentication.md)** - Security best practices
- **[Integration Patterns](guides/integration-patterns.md)** - Real-world workflows

### 🔧 For Advanced Features
- **[Webhook Implementation](api-reference/webhooks.md)** - Real-time notifications
- **[PDTF Compliance](technical/pdtf-compliance.md)** - Property data standards
- **[Error Handling](api-reference/errors.md)** - Troubleshooting guide

---

## ⚡ Key Features

### 🎯 **One-Call Property Creation**
Create clients, properties, and professional teams in a single API call. Automatic invitation system for new professionals.

### 📊 **Automatic Milestone Tracking**
Properties automatically get milestone templates based on sale method (standard sale vs. modern auction).

### 👥 **Professional Team Management**
Seamlessly invite and assign estate agents, conveyancers, auctioneers, and mortgage brokers to property transactions.

### 📦 **Product Integration**
Built-in support for add-on products like "Moove Ready Pack" with flexible payment terms.

### 🔔 **Real-Time Updates**
Webhook notifications keep your system synchronized with property and milestone changes.

### 📋 **PDTF Compliant**
All property data follows Property Data Trust Framework standards for industry compatibility.

---

## 🌍 Environments

| Environment | Base URL | Purpose |
|-------------|----------|---------|
| **Staging** | `https://staging.openmoove.com/api/v1/partners/` | Development & testing |
| **Production** | `https://api.openmoove.com/api/v1/partners/` | Live transactions |

**Note:** The staging database is serverless and may take a few seconds to spin up, causing slightly longer initial response times.

---

## 🔧 Quick Reference

### Core Endpoints
- `GET /products/` - List available products
- `POST /clients/` - Create client and property
- `GET /properties/` - List your properties
- `GET /properties/{id}/` - Get property details
- `GET /properties/{id}/milestones/` - Get milestone progress

### Authentication
All requests require the `X-API-KEY` header with your API key.

### Rate Limits
- 1000 requests per hour
- Burst up to 100 requests per minute

---

## 💬 Support

### 📧 Get Help
- **Email:** api-support@openmoove.com
- **Response Time:** Within 24 hours

### 📝 Include in Support Requests
- API endpoint used
- Request payload (remove sensitive data)
- Response received
- Expected behavior
- Timestamp of issue

---

## 🚀 Ready to Build?

Choose your preferred starting point:

### For Speed Runners 🏃‍♂️
→ **[Postman Collection](examples/postman/)** - Import and start testing immediately

### For Methodical Builders 👩‍💻
→ **[Complete Integration Guide](guides/quick-start.md)** - Step-by-step walkthrough

### For Code-First Developers 💻
→ **[Code Examples](examples/)** - Python, JavaScript, and cURL implementations

---

*Built with ❤️ by the OpenMoove team. Making property transactions simple for everyone.*