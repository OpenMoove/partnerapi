# OpenMoove Partner API Developer Guide

## Table of Contents

1. [Introduction](#introduction)
2. [Getting Started](#getting-started)
3. [Authentication](#authentication)
4. [API Endpoints](#api-endpoints)
5. [Request & Response Format](#request--response-format)
6. [Error Handling](#error-handling)
7. [Webhooks](#webhooks)
8. [Rate Limiting](#rate-limiting)
9. [Code Examples](#code-examples)
10. [Best Practices](#best-practices)
11. [Testing](#testing)
12. [Troubleshooting](#troubleshooting)

## Introduction

The OpenMoove Partner API provides a RESTful interface for professional service providers to integrate with our property transaction platform. This guide covers everything you need to build a robust integration.

### Base URLs

| Environment | URL |
|-------------|-----|
| Production | `https://api.openmoove.com/api/partners/` |
| Staging | `https://staging.openmoove.com/api/partners/` |

## Getting Started

### Prerequisites

- An OpenMoove professional account (Estate Agent, Conveyancer, Mortgage Broker, or Auctioneer)
- Basic understanding of RESTful APIs
- Ability to make HTTP requests from your application

### Obtaining API Credentials

1. Log in to your OpenMoove account
2. Navigate to **Profile → Integrations**
3. Click **Generate API Key** in the Partner API section
4. Securely store your API key - you won't be able to see it again

### Making Your First Request

Test your API key with a simple GET request:

```bash
curl -H "X-API-KEY: your_api_key_here" \
     https://api.openmoove.com/api/partners/properties/
```

## Authentication

All API requests require authentication using an API key in the request header.

### Header Format

```http
X-API-KEY: your_api_key_here
```

### Security Best Practices

- **Never expose API keys** in client-side code or public repositories
- **Use environment variables** to store API keys
- **Rotate keys regularly** for enhanced security
- **Use HTTPS** for all API requests

### Example Implementation

```python
import os
import requests

class OpenMooveAPI:
    def __init__(self):
        self.api_key = os.environ.get('OPENMOOVE_API_KEY')
        self.base_url = 'https://api.openmoove.com/api/partners'
        self.session = requests.Session()
        self.session.headers.update({
            'X-API-KEY': self.api_key,
            'Content-Type': 'application/json'
        })
```

## API Endpoints

### 1. Create Client and Property

Create a new client (buyer/seller) and their property in a single request.

**Endpoint:** `POST /api/partners/clients/`

**Request Body:**
```json
{
  "first_name": "John",
  "last_name": "Smith",
  "email": "john.smith@example.com",
  "phone_number": "+447700900123",
  "property_address": "123 Example Street, London",
  "property_postcode": "SW1A 1AA",
  "sale_method": "standard",
  "estate_agent_email": "agent@agency.com",
  "conveyancer_email": "lawyer@lawfirm.com"
}
```

**Response (201 Created):**
```json
{
  "property_id": 456,
  "mover_id": 123,
  "message": "Client and property created successfully",
  "estate_agent_id": 789,
  "conveyancer_id": 101
}
```

#### Required Fields

| Field | Type | Description |
|-------|------|-------------|
| `first_name` | String | Client's first name (max 150 chars) |
| `last_name` | String | Client's last name (max 150 chars) |
| `email` | Email | Valid email address |
| `phone_number` | String | Phone in E164 format |
| `property_address` | String | Full property address (max 255 chars) |
| `property_postcode` | String | UK postcode (max 10 chars) |

#### Professional Emails

At least one professional email is required:
- `estate_agent_email`
- `conveyancer_email`
- `mortgage_broker_email`
- `auctioneer_email`

#### Sale Methods

| Value | Description | Requirements |
|-------|-------------|--------------|
| `standard` | Standard property sale | Default |
| `modern_auction` | Modern auction method | Requires `auctioneer_email` |
| `secure_sale` | Secure sale method | Requires `auctioneer_email` |

### 2. List Properties

Get a paginated list of properties where you're a team member.

**Endpoint:** `GET /api/partners/properties/`

**Query Parameters:**
- `page` - Page number (default: 1)
- `page_size` - Results per page (max: 100, default: 20)

**Response:**
```json
{
  "count": 25,
  "next": "https://api.openmoove.com/api/partners/properties/?page=2",
  "previous": null,
  "results": [
    {
      "id": 123,
      "address": "123 Example Street, London",
      "postcode": "SW1A 1AA",
      "uprn": 12345678901,
      "sale_method": "standard",
      "created_at": "2024-01-15T10:00:00Z"
    }
  ]
}
```

### 3. Get Property Details

Retrieve comprehensive information about a specific property.

**Endpoint:** `GET /api/partners/properties/{property_id}/detail/`

**Response:**
```json
{
  "id": 123,
  "address": "123 Example Street, London",
  "postcode": "SW1A 1AA",
  "uprn": 12345678901,
  "title_number": "LN12345",
  "sale_method": "standard",
  "seller": {
    "id": 456,
    "first_name": "John",
    "last_name": "Smith",
    "email": "john.smith@example.com",
    "phone_number": "+447700900123"
  },
  "team": {
    "estate_agent": {
      "id": 789,
      "name": "Sarah Johnson",
      "email": "sarah@premieragents.com",
      "firm": "Premier Agents Ltd"
    },
    "conveyancer": {
      "id": 101,
      "name": "James Wilson",
      "email": "james@lawfirm.com",
      "firm": "Wilson & Associates"
    }
  },
  "milestones_count": 8,
  "completed_milestones": 3
}
```

### 4. Get Property Milestones

Track the progress of a property transaction through milestones.

**Endpoint:** `GET /api/partners/properties/{property_id}/milestones/`

**Response:**
```json
[
  {
    "id": 1,
    "name": "Instruction",
    "description": "Property instruction received",
    "order": 1,
    "completed": true,
    "completed_at": "2024-01-15T10:00:00Z",
    "due_date": "2024-01-15T23:59:59Z",
    "is_overdue": false
  },
  {
    "id": 2,
    "name": "Legal Pack Preparation",
    "description": "Preparing legal documentation",
    "order": 2,
    "completed": false,
    "completed_at": null,
    "due_date": "2024-01-20T23:59:59Z",
    "is_overdue": false
  }
]
```

### 5. Get Current Milestone

Get the currently active milestone for a property.

**Endpoint:** `GET /api/partners/properties/{property_id}/milestones/current/`

## Request & Response Format

### Request Headers

All requests should include:
```http
Content-Type: application/json
X-API-KEY: your_api_key_here
```

### Response Format

All responses are JSON formatted with appropriate HTTP status codes.

### Pagination

List endpoints support pagination:
```json
{
  "count": 100,
  "next": "https://api.openmoove.com/api/partners/properties/?page=3",
  "previous": "https://api.openmoove.com/api/partners/properties/?page=1",
  "results": [...]
}
```

## Error Handling

### HTTP Status Codes

| Code | Description | Action |
|------|-------------|--------|
| 200 | Success | Process response |
| 201 | Created | Resource created successfully |
| 400 | Bad Request | Check validation errors |
| 401 | Unauthorized | Verify API key |
| 403 | Forbidden | Check permissions |
| 404 | Not Found | Resource doesn't exist |
| 429 | Rate Limited | Implement backoff |
| 500 | Server Error | Contact support |

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

#### Missing Required Fields
```json
{
  "first_name": ["This field is required."],
  "property_address": ["This field is required."]
}
```

#### Invalid Phone Number
```json
{
  "phone_number": ["Invalid phone number format"]
}
```

#### Missing Professional Email
```json
{
  "non_field_errors": [
    "At least one professional email must be provided"
  ]
}
```

## Webhooks

Webhooks provide real-time notifications about property and milestone changes.

### Configuration

1. Log in to OpenMoove
2. Navigate to **Profile → Integrations**
3. Add your webhook URL in the Partner API section

### Supported Events

| Event | Description |
|-------|-------------|
| `property.created` | New property added |
| `property.updated` | Property details changed |
| `property.milestone.created` | New milestone added |
| `property.milestone.updated` | Milestone status changed |

### Webhook Payload

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

### Webhook Security

All webhooks include an HMAC-SHA256 signature in the `X-OpenMoove-Signature` header.

#### Verifying Signatures (Python)

```python
import hmac
import hashlib

def verify_webhook(payload_body, signature_header, webhook_secret):
    """Verify webhook signature"""
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

#### Verifying Signatures (Node.js)

```javascript
const crypto = require('crypto');

function verifyWebhook(payloadBody, signatureHeader, webhookSecret) {
    if (!signatureHeader.startsWith('sha256=')) {
        return false;
    }
    
    const expected = crypto
        .createHmac('sha256', webhookSecret)
        .update(payloadBody)
        .digest('hex');
    
    const received = signatureHeader.slice(7); // Remove 'sha256='
    return crypto.timingSafeEqual(
        Buffer.from(expected),
        Buffer.from(received)
    );
}
```

### Retry Policy

Failed webhooks are retried with exponential backoff:
- Attempt 1: Immediate
- Attempt 2: 10 seconds
- Attempt 3: 100 seconds
- Attempt 4: 1000 seconds

Return HTTP status < 400 to acknowledge receipt.

## Rate Limiting

### Limits

- **Rate**: 1000 requests per hour per API key
- **Burst**: Up to 100 requests per minute

### Rate Limit Headers

```http
X-RateLimit-Limit: 1000
X-RateLimit-Remaining: 999
X-RateLimit-Reset: 1642680000
```

### Handling Rate Limits

Implement exponential backoff when rate limited:

```python
import time
import random

def retry_with_backoff(func, max_retries=3):
    for attempt in range(max_retries):
        try:
            return func()
        except RateLimitError:
            if attempt == max_retries - 1:
                raise
            delay = (2 ** attempt) + random.uniform(0, 1)
            time.sleep(delay)
```

## Code Examples

### Python Implementation

```python
import requests
import os

class OpenMooveAPI:
    def __init__(self, api_key=None):
        self.api_key = api_key or os.environ.get('OPENMOOVE_API_KEY')
        self.base_url = 'https://api.openmoove.com/api/partners'
        self.session = requests.Session()
        self.session.headers.update({
            'X-API-KEY': self.api_key,
            'Content-Type': 'application/json'
        })
    
    def create_client(self, client_data):
        """Create a new client and property"""
        response = self.session.post(
            f'{self.base_url}/clients/',
            json=client_data
        )
        response.raise_for_status()
        return response.json()
    
    def list_properties(self, page=1, page_size=20):
        """List properties with pagination"""
        params = {'page': page, 'page_size': page_size}
        response = self.session.get(
            f'{self.base_url}/properties/',
            params=params
        )
        response.raise_for_status()
        return response.json()
    
    def get_property_details(self, property_id):
        """Get detailed property information"""
        response = self.session.get(
            f'{self.base_url}/properties/{property_id}/detail/'
        )
        response.raise_for_status()
        return response.json()

# Usage example
api = OpenMooveAPI()

# Create a client
client_data = {
    'first_name': 'John',
    'last_name': 'Smith',
    'email': 'john.smith@example.com',
    'phone_number': '+447700900123',
    'property_address': '123 Example Street, London',
    'property_postcode': 'SW1A 1AA',
    'estate_agent_email': 'agent@agency.com'
}

result = api.create_client(client_data)
print(f"Created property {result['property_id']}")
```

### JavaScript/Node.js Implementation

```javascript
const axios = require('axios');

class OpenMooveAPI {
    constructor(apiKey) {
        this.apiKey = apiKey || process.env.OPENMOOVE_API_KEY;
        this.client = axios.create({
            baseURL: 'https://api.openmoove.com/api/partners',
            headers: {
                'X-API-KEY': this.apiKey,
                'Content-Type': 'application/json'
            }
        });
    }
    
    async createClient(clientData) {
        try {
            const response = await this.client.post('/clients/', clientData);
            return response.data;
        } catch (error) {
            if (error.response) {
                throw new Error(`API Error: ${error.response.status} - ${JSON.stringify(error.response.data)}`);
            }
            throw error;
        }
    }
    
    async listProperties(page = 1, pageSize = 20) {
        const response = await this.client.get('/properties/', {
            params: { page, page_size: pageSize }
        });
        return response.data;
    }
    
    async getPropertyDetails(propertyId) {
        const response = await this.client.get(`/properties/${propertyId}/detail/`);
        return response.data;
    }
}

// Usage example
const api = new OpenMooveAPI('your_api_key_here');

const clientData = {
    first_name: 'John',
    last_name: 'Smith',
    email: 'john.smith@example.com',
    phone_number: '+447700900123',
    property_address: '123 Example Street, London',
    property_postcode: 'SW1A 1AA',
    estate_agent_email: 'agent@agency.com'
};

api.createClient(clientData)
    .then(result => console.log(`Created property ${result.property_id}`))
    .catch(error => console.error('Error:', error.message));
```

### cURL Examples

**Create Client:**
```bash
curl -X POST \
  -H "Content-Type: application/json" \
  -H "X-API-KEY: your_api_key_here" \
  -d '{
    "first_name": "John",
    "last_name": "Smith",
    "email": "john.smith@example.com",
    "phone_number": "+447700900123",
    "property_address": "123 Example Street, London",
    "property_postcode": "SW1A 1AA",
    "estate_agent_email": "agent@agency.com"
  }' \
  https://api.openmoove.com/api/partners/clients/
```

**List Properties:**
```bash
curl -H "X-API-KEY: your_api_key_here" \
     "https://api.openmoove.com/api/partners/properties/?page=1&page_size=20"
```

## Best Practices

### 1. Error Handling

Always implement comprehensive error handling:

```python
try:
    result = api.create_client(client_data)
except requests.exceptions.HTTPError as e:
    if e.response.status_code == 400:
        errors = e.response.json()
        # Handle validation errors
    elif e.response.status_code == 401:
        # Handle authentication error
    elif e.response.status_code == 429:
        # Handle rate limiting
    else:
        # Handle other errors
```

### 2. Data Validation

Validate data before sending requests:

```javascript
function validatePhoneNumber(phone) {
    // E164 format validation
    const e164Regex = /^\+[1-9]\d{1,14}$/;
    return e164Regex.test(phone);
}

function validateEmail(email) {
    const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
    return emailRegex.test(email);
}
```

### 3. Idempotency

Make operations idempotent where possible:

```python
def create_or_update_client(api, client_data):
    """Create client if doesn't exist, update if does"""
    try:
        # Try to find existing client
        properties = api.list_properties()
        existing = next(
            (p for p in properties['results'] 
             if p['seller']['email'] == client_data['email']),
            None
        )
        
        if existing:
            # Update existing
            return api.update_property(existing['id'], client_data)
        else:
            # Create new
            return api.create_client(client_data)
    except Exception as e:
        # Handle errors
        raise
```

### 4. Logging

Implement comprehensive logging:

```python
import logging

logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)

class OpenMooveAPI:
    def create_client(self, client_data):
        logger.info(f"Creating client: {client_data['email']}")
        try:
            response = self.session.post(...)
            logger.info(f"Client created successfully: {response.json()}")
            return response.json()
        except Exception as e:
            logger.error(f"Failed to create client: {str(e)}")
            raise
```

### 5. Security

- Store API keys in environment variables
- Use HTTPS for all requests
- Validate webhook signatures
- Implement request timeouts
- Log security events

## Testing

### Test Environment

Use the staging environment for development:
```
https://staging.openmoove.com/api/partners/
```

### Test Data

#### Test Phone Numbers
Use these numbers for testing (bypass SMS verification):
- `+447700900001` through `+447700900010`

#### Test Professionals
```json
{
  "estate_agent_email": "test-agent@example.com",
  "conveyancer_email": "test-conveyancer@example.com",
  "auctioneer_email": "test-auctioneer@example.com"
}
```

### Integration Testing

1. **Authentication Testing**
   ```python
   def test_authentication():
       # Test with invalid key
       api = OpenMooveAPI('invalid_key')
       with pytest.raises(AuthenticationError):
           api.list_properties()
       
       # Test with valid key
       api = OpenMooveAPI(VALID_TEST_KEY)
       properties = api.list_properties()
       assert 'results' in properties
   ```

2. **Validation Testing**
   ```python
   def test_validation():
       api = OpenMooveAPI(VALID_TEST_KEY)
       
       # Test missing required fields
       invalid_data = {'first_name': 'John'}
       with pytest.raises(ValidationError) as exc:
           api.create_client(invalid_data)
       assert 'last_name' in str(exc.value)
   ```

3. **End-to-End Testing**
   ```python
   def test_full_workflow():
       api = OpenMooveAPI(VALID_TEST_KEY)
       
       # Create client
       client_data = generate_test_client_data()
       result = api.create_client(client_data)
       property_id = result['property_id']
       
       # Get property details
       details = api.get_property_details(property_id)
       assert details['id'] == property_id
       
       # Check milestones
       milestones = api.get_milestones(property_id)
       assert len(milestones) > 0
   ```

## Troubleshooting

### Common Issues

#### 401 Unauthorized
- **Check**: API key is included in `X-API-KEY` header
- **Verify**: API key is correct and active
- **Ensure**: Using HTTPS, not HTTP

#### 400 Bad Request
- **Check**: All required fields are provided
- **Verify**: Phone number is in E164 format
- **Ensure**: At least one professional email provided
- **For auctions**: Include auctioneer email

#### 404 Not Found
- **Check**: Property ID exists
- **Verify**: You have access to the property
- **Ensure**: Using correct endpoint URL

#### 429 Rate Limited
- **Implement**: Exponential backoff
- **Check**: Rate limit headers
- **Reduce**: Request frequency
- **Consider**: Caching responses

### Debug Checklist

- [ ] API key included in headers
- [ ] Content-Type set to application/json
- [ ] All required fields provided
- [ ] Phone number in valid E164 format
- [ ] Professional emails valid
- [ ] Using correct endpoint URLs
- [ ] Handling error responses properly

### Getting Help

When contacting support, include:
- API endpoint used
- Request headers (exclude API key)
- Request payload
- Response received
- Expected behavior
- Timestamp of request

**Support Contact**: api-support@openmoove.com

---

*Last updated: January 2024*