# API Reference

Complete reference for all OpenMoove Partner API v1 endpoints with detailed examples and response schemas.

---

## Base URLs

| Environment | URL |
|-------------|-----|
| **Staging** | `https://staging.openmoove.com/api/v1/partners/` |
| **Production** | `https://api.openmoove.com/api/v1/partners/` |

## Authentication

All endpoints require the `X-API-KEY` header:
```http
X-API-KEY: om_your_api_key_here
Content-Type: application/json
```

---

## 📦 Products

### List Available Products

Get all products available for purchase with properties.

**Endpoint:** `GET /products/`

**Headers:**
```http
X-API-KEY: om_your_api_key_here
```

**Response (200 OK):**
```json
{
  "count": 1,
  "results": [
    {
      "name": "Moove Ready Pack",
      "slug": "moove-ready-pack",
      "description": "Complete property transaction package including legal support and milestone tracking",
      "price": {
        "amount": 0,
        "currency": "GBP"
      }
    }
  ]
}
```

**cURL Example:**
```bash
curl -H "X-API-KEY: om_your_key_here" \
  "https://staging.openmoove.com/api/v1/partners/products/"
```

---

## 👥 Clients & Properties

### Create Client and Property

Create a new client (mover) and their property in a single request. Automatically sets up professional teams and milestone tracking.

**Endpoint:** `POST /clients/`

**Headers:**
```http
X-API-KEY: om_your_api_key_here
Content-Type: application/json
```

**Request Body:**

#### Required Fields
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

#### Professional Team (At least one required)
```json
{
  "estate_agent_email": "agent@agency.com",
  "conveyancer_email": "lawyer@lawfirm.com",
  "mortgage_broker_email": "broker@mortgages.com",
  "auctioneer_email": "auctioneer@auctions.com"
}
```

#### Optional Fields
```json
{
  "sale_method": "standard",  // "standard", "modern_auction", "secure_sale"
  "products": ["moove-ready-pack"]  // Product slugs to purchase
}
```

#### Complete Example Request:
```json
{
  "first_name": "Jane",
  "last_name": "Doe",
  "email": "jane.doe@example.com",
  "phone_number": "+447973138902",
  "property_address": "456 Auction Avenue, Manchester",
  "property_postcode": "M1 2AB",
  "sale_method": "modern_auction",
  "estate_agent_email": "auction-agent@example.com",
  "auctioneer_email": "integration@biduk.com",
  "products": ["moove-ready-pack"]
}
```

**Response (201 Created):**
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
          "firstName": "Jane",
          "lastName": "Doe"
        },
        "email": "jane.doe@example.com",
        "phone": "+447973138902",
        "role": "Seller"
      }
    ],
    "propertyPack": {
      "priceInformation": {
        "disposal": "Modern Method of Auction"
      },
      "address": {
        "buildingNumber": "1",
        "street": "456 Auction Avenue",
        "line1": "456 Auction Avenue, Manchester",
        "postcode": "M1 2AB",
        "countryCode": "GBR"
      },
      "ownership": {
        "ownershipsToBeTransferred": [
          {
            "titlePropertyDescription": "456 Auction Avenue, Manchester",
            "ownershipType": "Freehold",
            "wholeFreeholdForSale": {
              "yesNo": "Yes"
            }
          }
        ]
      }
    },
    "externalIds": {
      "openmoove": "d693c0f9-ee27-4744-b490-0a3ccf4e1c48"
    }
  },
  "estate_agent_id": 100,
  "auctioneer_id": 67,
  "purchases": [
    {
      "id": 34,
      "product_name": "Moove Ready Pack",
      "amount": "0",
      "payment_terms": "net_30",
      "payment_status": "pending",
      "purchased_at": "2025-07-22T14:12:37.914557+00:00"
    }
  ],
  "total_amount": "0"
}
```

**cURL Example:**
```bash
curl -X POST \
  -H "Content-Type: application/json" \
  -H "X-API-KEY: om_your_key_here" \
  -d '{
    "first_name": "Jane",
    "last_name": "Doe",
    "email": "jane.doe@example.com",
    "phone_number": "+447973138902",
    "property_address": "456 Auction Avenue, Manchester",
    "property_postcode": "M1 2AB",
    "sale_method": "modern_auction",
    "estate_agent_email": "auction-agent@example.com",
    "auctioneer_email": "integration@biduk.com",
    "products": ["moove-ready-pack"]
  }' \
  "https://staging.openmoove.com/api/v1/partners/clients/"
```

#### Validation Rules

**Phone Number:**
- Must be valid international format
- Automatically converted to E164 format
- Examples: `+447700900123`, `07700900123`, `447700900123`

**Professional Emails:**
- At least one professional email is required
- For auction sales: `auctioneer_email` is mandatory
- All emails must be valid format

**Sale Methods:**
- `standard`: Standard property sale
- `modern_auction`: Modern method auction (requires auctioneer)
- `secure_sale`: Secure sale method (requires auctioneer)

---

## 🏠 Properties

### List Properties

Get a paginated list of properties where you're assigned as a professional team member.

**Endpoint:** `GET /properties/`

**Query Parameters:**
| Parameter | Type | Description | Default |
|-----------|------|-------------|---------|
| `page` | Integer | Page number | 1 |
| `page_size` | Integer | Results per page (max 100) | 20 |

**Response (200 OK):**
```json
{
  "count": 25,
  "next": "https://staging.openmoove.com/api/v1/partners/properties/?page=2",
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

**cURL Example:**
```bash
curl -H "X-API-KEY: om_your_key_here" \
  "https://staging.openmoove.com/api/v1/partners/properties/?page=1&page_size=20"
```

### Get Property Details

Retrieve comprehensive information about a specific property using its UUID.

**Endpoint:** `GET /properties/{property_uuid}/`

**Path Parameters:**
- `property_uuid`: The UUID of the property (from property creation or list response)

**Response (200 OK):**
```json
{
  "transactionId": "4fa18bac-3c62-4d92-a271-7364b0f0e479",
  "status": "For sale",
  "participants": [
    {
      "name": {
        "firstName": "Jane",
        "lastName": "Doe"
      },
      "email": "jane.doe@example.com",
      "phone": "+447973138902",
      "role": "Seller"
    }
  ],
  "propertyPack": {
    "priceInformation": {
      "disposal": "Modern Method of Auction"
    },
    "address": {
      "buildingNumber": "1",
      "street": "456 Auction Avenue",
      "line1": "456 Auction Avenue, Manchester",
      "postcode": "M1 2AB",
      "countryCode": "GBR"
    },
    "ownership": {
      "ownershipsToBeTransferred": [
        {
          "titlePropertyDescription": "456 Auction Avenue, Manchester",
          "ownershipType": "Freehold",
          "wholeFreeholdForSale": {
            "yesNo": "Yes"
          }
        }
      ]
    }
  },
  "externalIds": {
    "openmoove": "d693c0f9-ee27-4744-b490-0a3ccf4e1c48"
  }
}
```

**cURL Example:**
```bash
curl -H "X-API-KEY: om_your_key_here" \
  "https://staging.openmoove.com/api/v1/partners/properties/4fa18bac-3c62-4d92-a271-7364b0f0e479/"
```

---

## 📊 Milestones

### Get Property Milestones

Retrieve all milestones for a property transaction, showing progress through the conveyancing process.

**Endpoint:** `GET /properties/{property_uuid}/milestones/`

**Path Parameters:**
- `property_uuid`: The UUID of the property

**Response (200 OK):**
```json
[
  {
    "id": 1,
    "name": "Instruction",
    "description": "Property instruction received from seller",
    "order": 1,
    "completed": true,
    "completed_at": "2024-01-15T10:00:00Z",
    "due_date": "2024-01-15T23:59:59Z",
    "is_overdue": false
  },
  {
    "id": 2,
    "name": "Legal Pack Preparation",
    "description": "Legal documentation and property pack preparation",
    "order": 2,
    "completed": false,
    "completed_at": null,
    "due_date": "2024-01-20T23:59:59Z",
    "is_overdue": false
  },
  {
    "id": 3,
    "name": "Property Marketing",
    "description": "Property listed and marketed to potential buyers",
    "order": 3,
    "completed": false,
    "completed_at": null,
    "due_date": "2024-01-25T23:59:59Z",
    "is_overdue": false
  }
]
```

**cURL Example:**
```bash
curl -H "X-API-KEY: om_your_key_here" \
  "https://staging.openmoove.com/api/v1/partners/properties/4fa18bac-3c62-4d92-a271-7364b0f0e479/milestones/"
```

### Get Current Milestone

Retrieve the current active milestone for a property.

**Endpoint:** `GET /properties/{property_uuid}/milestones/current/`

**Response (200 OK):**
```json
{
  "id": 2,
  "name": "Legal Pack Preparation",
  "description": "Legal documentation and property pack preparation",
  "order": 2,
  "completed": false,
  "completed_at": null,
  "due_date": "2024-01-20T23:59:59Z",
  "is_overdue": false,
  "days_until_due": 5,
  "assigned_to": "Conveyancer",
  "priority": "high"
}
```

**cURL Example:**
```bash
curl -H "X-API-KEY: om_your_key_here" \
  "https://staging.openmoove.com/api/v1/partners/properties/4fa18bac-3c62-4d92-a271-7364b0f0e479/milestones/current/"
```

---

## 💬 Chat (Coming Soon)

### Get Chat History

Retrieve chat messages for a property (currently returns mock data).

**Endpoint:** `GET /properties/{property_uuid}/chat/`

**Query Parameters:**
| Parameter | Type | Description | Default |
|-----------|------|-------------|---------|
| `page` | Integer | Page number | 1 |
| `page_size` | Integer | Messages per page (max 100) | 20 |

**Response (200 OK):**
```json
{
  "count": 0,
  "next": null,
  "previous": null,
  "results": [],
  "message": "Chat integration not yet implemented"
}
```

### Get Chat Summary

Retrieve AI-generated chat summaries (currently returns mock data).

**Endpoint:** `GET /properties/{property_uuid}/chat/summary/`

**Response (200 OK):**
```json
{
  "summaries": [
    {
      "date": "2024-01-15",
      "summary": "Discussion about property viewing arrangements and auction timeline"
    }
  ],
  "message": "Mock chat summary - real implementation pending"
}
```

---

## 📋 PDTF Compliance

All property data returned by the API follows the [Property Data Trust Framework (PDTF)](https://pdtf.org.uk/) standards for industry compatibility.

### Key PDTF Elements

**Property Pack Structure:**
- `priceInformation`: Sale method and pricing details
- `address`: Standardized address format with country codes
- `ownership`: Property ownership and tenure information
- `participants`: All parties involved in the transaction

**Transaction ID:**
- Each property gets a unique `transactionId` for tracking
- Used for cross-system integration and compliance

**Standard Role Definitions:**
- `Seller`: Property owner
- `Buyer`: Property purchaser (added when identified)
- `Estate Agent`: Marketing and sales professional
- `Conveyancer`: Legal professional
- `Auctioneer`: Auction professional (for auction sales)

---

## ⚠️ Error Responses

### Common HTTP Status Codes

| Code | Description | Meaning |
|------|-------------|---------|
| 200 | OK | Request successful |
| 201 | Created | Resource created successfully |
| 400 | Bad Request | Invalid request data |
| 401 | Unauthorized | Invalid or missing API key |
| 403 | Forbidden | Insufficient permissions |
| 404 | Not Found | Resource not found |
| 429 | Too Many Requests | Rate limit exceeded |
| 500 | Internal Server Error | Server error |

### Error Response Format

```json
{
  "detail": "Error message",
  "field_errors": {
    "phone_number": ["Invalid phone number format"],
    "email": ["Enter a valid email address"]
  }
}
```

### Common Validation Errors

**Missing Required Fields:**
```json
{
  "first_name": ["This field is required."],
  "property_address": ["This field is required."]
}
```

**Invalid Phone Number:**
```json
{
  "phone_number": ["Invalid phone number format"]
}
```

**Missing Professional Email:**
```json
{
  "non_field_errors": [
    "At least one professional email must be provided"
  ]
}
```

**Auction Without Auctioneer:**
```json
{
  "non_field_errors": [
    "auctioneer_email is required for auction sale methods"
  ]
}
```

---

## 🔄 Rate Limiting

### Limits
- **Rate**: 1000 requests per hour per API key
- **Burst**: Up to 100 requests per minute

### Headers
Rate limit information included in response headers:
```http
X-RateLimit-Limit: 1000
X-RateLimit-Remaining: 999
X-RateLimit-Reset: 1642680000
```

### Rate Limit Exceeded (429)
```json
{
  "detail": "Request was throttled. Expected available in 3600 seconds."
}
```

---

## 🧪 Testing

### Test Data

**Staging Environment:**
- Use phone numbers: `+447700900001` to `+447700900010` (bypass SMS verification)
- Use test email format: `test+timestamp@example.com`
- Safe test postcodes: `SW1A 1AA`, `M1 2AB`, `B1 1AA`

**Professional Test Emails:**
```json
{
  "estate_agent_email": "test-agent@example.com",
  "conveyancer_email": "test-conveyancer@example.com",
  "auctioneer_email": "test-auctioneer@example.com"
}
```

### Complete Test Example

```bash
# 1. List products
curl -H "X-API-KEY: om_your_key" \
  "https://staging.openmoove.com/api/v1/partners/products/"

# 2. Create property with product
curl -X POST \
  -H "Content-Type: application/json" \
  -H "X-API-KEY: om_your_key" \
  -d '{
    "first_name": "Test",
    "last_name": "User",
    "email": "test+1642680000@example.com",
    "phone_number": "+447700900001",
    "property_address": "Test Property Address",
    "property_postcode": "SW1A 1AA",
    "estate_agent_email": "test-agent@example.com",
    "products": ["moove-ready-pack"]
  }' \
  "https://staging.openmoove.com/api/v1/partners/clients/"

# 3. List your properties
curl -H "X-API-KEY: om_your_key" \
  "https://staging.openmoove.com/api/v1/partners/properties/"

# 4. Get property details (use UUID from step 2)
curl -H "X-API-KEY: om_your_key" \
  "https://staging.openmoove.com/api/v1/partners/properties/{property_uuid}/"

# 5. Get milestones
curl -H "X-API-KEY: om_your_key" \
  "https://staging.openmoove.com/api/v1/partners/properties/{property_uuid}/milestones/"
```

---

## 🔗 Next Steps

- **[Authentication Guide](authentication.md)** - API key setup and security
- **[Webhook Implementation](webhooks.md)** - Real-time notifications
- **[Error Handling Guide](errors.md)** - Troubleshooting and best practices
- **[Integration Patterns](../guides/integration-patterns.md)** - Common workflows
- **[Code Examples](../examples/)** - Implementation examples in multiple languages