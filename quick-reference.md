# OpenMoove Partner API Quick Reference

## 🚀 Quick Start

```bash
# Test your API key
curl -H "X-API-KEY: your_api_key" \
  https://api.openmoove.com/api/partners/properties/

# Create your first client
curl -X POST \
  -H "Content-Type: application/json" \
  -H "X-API-KEY: your_api_key" \
  -d '{
    "first_name":"John",
    "last_name":"Smith",
    "email":"john@example.com",
    "phone_number":"+447700900123",
    "property_address":"123 Example St",
    "property_postcode":"SW1A 1AA",
    "estate_agent_email":"agent@agency.com"
  }' \
  https://api.openmoove.com/api/partners/clients/
```

## 🔑 Authentication

All requests require API key in header:
```http
X-API-KEY: your_api_key_here
Content-Type: application/json
```

## 📋 API Endpoints

| Method | Endpoint | Purpose |
|--------|----------|---------|
| `POST` | `/api/partners/clients/` | Create client & property |
| `GET` | `/api/partners/properties/` | List your properties |
| `GET` | `/api/partners/properties/{id}/detail/` | Get property details |
| `GET` | `/api/partners/properties/{id}/milestones/` | Get all milestones |
| `GET` | `/api/partners/properties/{id}/milestones/current/` | Get current milestone |

## 🏠 Create Client & Property

**Endpoint:** `POST /api/partners/clients/`

### Required Fields
```json
{
  "first_name": "John",
  "last_name": "Smith",
  "email": "john.smith@example.com",
  "phone_number": "+447700900123",
  "property_address": "123 Example Street, London",
  "property_postcode": "SW1A 1AA"
}
```

### Professional Emails (at least one required)
```json
{
  "estate_agent_email": "agent@agency.com",
  "conveyancer_email": "lawyer@lawfirm.com",
  "mortgage_broker_email": "broker@mortgages.com",
  "auctioneer_email": "auctioneer@auctions.com"
}
```

### Sale Methods
```json
{
  "sale_method": "standard"        // Default
  "sale_method": "modern_auction"  // Requires auctioneer_email
  "sale_method": "secure_sale"     // Requires auctioneer_email
}
```

### Response
```json
{
  "property_id": 456,
  "mover_id": 123,
  "message": "Client and property created successfully",
  "estate_agent_id": 789,
  "conveyancer_id": 101
}
```

## 📍 List Properties

**Endpoint:** `GET /api/partners/properties/`

### Query Parameters
```bash
?page=1&page_size=20    # Pagination (max 100 per page)
```

### Response
```json
{
  "count": 25,
  "next": ".../?page=2",
  "previous": null,
  "results": [
    {
      "id": 123,
      "address": "123 Example Street, London",
      "postcode": "SW1A 1AA",
      "uprn": 12345678901,
      "sale_method": "modern_auction",
      "created_at": "2024-01-15T10:00:00Z"
    }
  ]
}
```

## 🔍 Property Details

**Endpoint:** `GET /api/partners/properties/{id}/detail/`

### Response
```json
{
  "id": 123,
  "address": "123 Example Street, London",
  "postcode": "SW1A 1AA",
  "seller": {
    "id": 456,
    "first_name": "John",
    "last_name": "Smith",
    "email": "john.smith@example.com",
    "phone_number": "+447700900123"
  },
  "team": {
    "estate_agent": {...},
    "conveyancer": {...},
    "auctioneer": {...}
  },
  "milestones_count": 8,
  "completed_milestones": 3
}
```

## 📊 Milestones

### All Milestones
**Endpoint:** `GET /api/partners/properties/{id}/milestones/`

### Current Milestone
**Endpoint:** `GET /api/partners/properties/{id}/milestones/current/`

### Response Format
```json
{
  "id": 2,
  "name": "Legal Pack Preparation",
  "description": "Legal documentation and property pack preparation",
  "order": 2,
  "completed": false,
  "completed_at": null,
  "due_date": "2024-01-20T23:59:59Z",
  "is_overdue": false
}
```

## 🔔 Webhooks

### Supported Events
| Event | Description |
|-------|-------------|
| `property.created` | New property added |
| `property.updated` | Property details changed |
| `property.milestone.created` | New milestone added |
| `property.milestone.updated` | Milestone status changed |

### Payload Format
```json
{
  "event": "property.milestone.updated",
  "timestamp": "2024-01-15T10:30:00Z",
  "data": {
    "property_id": 456,
    "milestone": {
      "id": 1001,
      "name": "Exchange of Contracts",
      "completed": true,
      "completed_date": "2024-01-15T10:30:00Z"
    }
  }
}
```

### Verify Webhook Signature
```python
import hmac, hashlib

def verify_webhook(payload_body, signature_header, webhook_secret):
    if not signature_header.startswith('sha256='):
        return False
    
    expected = hmac.new(
        webhook_secret.encode(),
        payload_body,
        hashlib.sha256
    ).hexdigest()
    
    received = signature_header[7:]  # Remove 'sha256='
    return hmac.compare_digest(expected, received)
```

## ⚡ Rate Limiting

- **Limit:** 1000 requests per hour per API key
- **Burst:** Up to 100 requests per minute

### Rate Limit Headers
```http
X-RateLimit-Limit: 1000
X-RateLimit-Remaining: 999
X-RateLimit-Reset: 1642680000
```

## 🛠️ Common Workflows

### 1. Auction Property Creation
```json
{
  "first_name": "John",
  "last_name": "Smith",
  "email": "john@example.com",
  "phone_number": "+447700900123",
  "property_address": "123 Main St",
  "property_postcode": "SW1A 1AA",
  "sale_method": "modern_auction",
  "estate_agent_email": "agent@agency.com",
  "auctioneer_email": "auctioneer@bid.com"
}
```

### 2. Check Property Progress
```bash
# Get current milestone
curl -H "X-API-KEY: your_key" \
  https://api.openmoove.com/api/partners/properties/123/milestones/current/

# Get all milestones
curl -H "X-API-KEY: your_key" \
  https://api.openmoove.com/api/partners/properties/123/milestones/
```

### 3. Monitor Multiple Properties
```bash
# List all your properties
curl -H "X-API-KEY: your_key" \
  "https://api.openmoove.com/api/partners/properties/?page_size=100"
```

## 📱 Code Examples

### Python
```python
import requests

class OpenMooveAPI:
    def __init__(self, api_key):
        self.session = requests.Session()
        self.session.headers.update({
            'X-API-KEY': api_key,
            'Content-Type': 'application/json'
        })
        self.base_url = 'https://api.openmoove.com/api/partners'
    
    def create_client(self, data):
        response = self.session.post(f'{self.base_url}/clients/', json=data)
        response.raise_for_status()
        return response.json()
    
    def list_properties(self):
        response = self.session.get(f'{self.base_url}/properties/')
        response.raise_for_status()
        return response.json()
```

### JavaScript
```javascript
const api = axios.create({
  baseURL: 'https://api.openmoove.com/api/partners',
  headers: { 'X-API-KEY': 'your_key' }
});

// Create client
const result = await api.post('/clients/', clientData);

// List properties  
const properties = await api.get('/properties/');
```

## ❌ Error Handling

### Status Codes
| Code | Meaning | Action |
|------|---------|--------|
| 200 | Success | Process response |
| 201 | Created | Resource created |
| 400 | Bad Request | Check validation errors |
| 401 | Unauthorized | Check API key |
| 403 | Forbidden | Check permissions |
| 404 | Not Found | Resource doesn't exist |
| 429 | Rate Limited | Implement backoff |
| 500 | Server Error | Retry with exponential backoff |

### Common Validation Errors
```json
// Missing required fields
{
  "first_name": ["This field is required."],
  "property_address": ["This field is required."]
}

// Invalid phone number
{
  "phone_number": ["Invalid phone number format"]
}

// Missing professional email
{
  "non_field_errors": [
    "At least one professional email must be provided"
  ]
}

// Auction without auctioneer
{
  "non_field_errors": [
    "auctioneer_email is required for auction sale methods"
  ]
}
```

## 🧪 Testing

### Staging Environment
```
https://staging.openmoove.com/api/partners/
```

### Test Phone Numbers
Use `+447700900XXX` format (001-010) for testing - these bypass SMS verification.

### Test Data Example
```json
{
  "first_name": "Test",
  "last_name": "User",
  "email": "test+timestamp@example.com",
  "phone_number": "+447700900001",
  "property_address": "Test Property Address",
  "property_postcode": "SW1A 1AA",
  "estate_agent_email": "test-agent@example.com"
}
```

## 🔧 Troubleshooting

### Quick Debug Checklist

**401 Unauthorized**
- ✅ API key included in `X-API-KEY` header
- ✅ API key is correct and active
- ✅ Using HTTPS, not HTTP

**400 Bad Request**
- ✅ All required fields provided
- ✅ Phone number in valid format (`+447700900123`)
- ✅ At least one professional email included
- ✅ For auctions: auctioneer_email provided

**404 Not Found**
- ✅ Property ID exists
- ✅ You're assigned to the property team
- ✅ Using correct endpoint URL

**429 Rate Limited**
- ✅ Implement exponential backoff
- ✅ Check rate limit headers
- ✅ Reduce request frequency

### Professional Invitation Flow

1. **Existing Professional:** Used immediately
2. **New Professional:** Invited via email with 12-character code
3. **Invitation States:**
   - **Invited:** Cannot access API yet
   - **Active:** Accepted invitation, can access API

### Validation Rules Summary

| Field | Format | Notes |
|-------|--------|-------|
| `phone_number` | E164 (`+447700900123`) | Auto-converted from various formats |
| `email` | Valid email | Standard email validation |
| `property_postcode` | UK postcode | 10 chars max |
| `sale_method` | `standard`/`modern_auction`/`secure_sale` | Auctions require auctioneer |

## 📞 Support

- **Email:** api-support@openmoove.com
- **Issues:** [GitHub Issues](https://github.com/openmoove/partner-api/issues)
- **Status:** [API Status](https://status.openmoove.com)

### Include in Support Requests
- API endpoint used
- Request payload (remove sensitive data)
- Response received
- Expected behavior
- Timestamp of issue

---

*For detailed information, see the [Developer Guide](./developer-guide.md)*