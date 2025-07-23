# Authentication Guide

## Overview

The OpenMoove Partner API uses API key authentication for secure access. All requests must include your API key in the `X-API-KEY` header.

---

## 🔑 Getting Your API Key

### Step 1: Access Your Profile
1. Visit [staging.openmoove.com](https://staging.openmoove.com) (for development)
2. Log in to your account
3. Navigate to **Profile → Integrations**

### Step 2: Generate API Key
1. Click **"Generate API Key"** or **"Create New Key"**
2. Copy the generated key (format: `om_xxxx`)
3. Store it securely (see [Security Best Practices](#security-best-practices))

### Step 3: Test Your Key
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

---

## 🔐 Using Your API Key

### Header Format
All API requests must include your API key in the request headers:

```http
X-API-KEY: om_your_key_here
Content-Type: application/json
```

### Example Request
```bash
curl -X POST \
  -H "Content-Type: application/json" \
  -H "X-API-KEY: om_your_key_here" \
  -d '{"first_name": "John", "last_name": "Smith", ...}' \
  "https://staging.openmoove.com/api/v1/partners/clients/"
```

---

## 🌍 Environment Configuration

### Staging Environment (Development)
- **Base URL:** `https://staging.openmoove.com/api/v1/partners/`
- **Purpose:** Development, testing, and integration
- **Performance:** May have slight delays due to serverless database
- **Data:** Safe test environment with mock data

### Production Environment (Live)
- **Base URL:** `https://api.openmoove.com/api/v1/partners/`
- **Purpose:** Live transactions and real client data
- **Performance:** Optimized for production workloads
- **Data:** Real transactions and sensitive information

### Environment-Specific Configuration

**Python Example:**
```python
import os

# Environment-based configuration
if os.getenv('ENVIRONMENT') == 'production':
    BASE_URL = 'https://api.openmoove.com/api/v1/partners'
    API_KEY = os.getenv('OPENMOOVE_PROD_API_KEY')
else:
    BASE_URL = 'https://staging.openmoove.com/api/v1/partners'
    API_KEY = os.getenv('OPENMOOVE_STAGING_API_KEY')

headers = {
    'X-API-KEY': API_KEY,
    'Content-Type': 'application/json'
}
```

**JavaScript Example:**
```javascript
const config = {
  staging: {
    baseUrl: 'https://staging.openmoove.com/api/v1/partners',
    apiKey: process.env.OPENMOOVE_STAGING_API_KEY
  },
  production: {
    baseUrl: 'https://api.openmoove.com/api/v1/partners',
    apiKey: process.env.OPENMOOVE_PROD_API_KEY
  }
};

const env = process.env.NODE_ENV || 'staging';
const { baseUrl, apiKey } = config[env];

const headers = {
  'X-API-KEY': apiKey,
  'Content-Type': 'application/json'
};
```

---

## 🔒 Security Best Practices

### 1. Secure Storage
**✅ Do:**
- Store API keys in environment variables
- Use secure configuration management systems
- Rotate keys regularly (quarterly)

**❌ Don't:**
- Commit API keys to version control
- Include keys in client-side code
- Share keys in plain text communications

### 2. Environment Variables
Create a `.env` file for local development:
```bash
# .env file
OPENMOOVE_STAGING_API_KEY=om_your_staging_key
OPENMOOVE_PROD_API_KEY=om_your_production_key
ENVIRONMENT=staging
```

Add `.env` to your `.gitignore`:
```gitignore
.env
*.env.local
*.env.production
```

### 3. HTTPS Only
- **Always use HTTPS** for API requests
- Never send API keys over unencrypted connections
- Verify SSL certificates in production

### 4. Access Control
- **Principle of least privilege**: Only grant necessary permissions
- **Separate keys**: Use different keys for staging and production
- **Monitor usage**: Track API key usage and detect anomalies

---

## 🚨 Authentication Errors

### 401 Unauthorized
**Cause:** Invalid or missing API key

**Example Response:**
```json
{
  "detail": "Invalid API key provided"
}
```

**Solutions:**
- ✅ Verify API key is included in `X-API-KEY` header
- ✅ Check API key format (should start with `om_`)
- ✅ Ensure API key is active and not expired
- ✅ Verify you're using the correct environment

### 403 Forbidden
**Cause:** Valid API key but insufficient permissions

**Example Response:**
```json
{
  "detail": "You do not have permission to perform this action"
}
```

**Solutions:**
- ✅ Verify your account has the necessary permissions
- ✅ Check if you're assigned to the property/resource
- ✅ Contact support if permissions seem incorrect

---

## 🔄 API Key Management

### Key Rotation
Rotate your API keys regularly for security:

1. **Generate new key** in your profile
2. **Update environment variables** with new key
3. **Test new key** with a simple API call
4. **Deploy updated configuration**
5. **Revoke old key** after successful deployment

### Multiple Keys
You can maintain multiple API keys for different purposes:
- **Development key**: For local development and testing
- **Staging key**: For staging environment and CI/CD
- **Production key**: For live production systems

### Key Monitoring
Monitor your API key usage:
- **Request volume**: Track daily/monthly API calls
- **Error rates**: Monitor authentication failures
- **Usage patterns**: Detect unusual activity
- **Performance**: Monitor response times

---

## 🧪 Testing Authentication

### Quick Test Script
Save this as `test-auth.py`:
```python
import requests
import os

# Test authentication
api_key = os.getenv('OPENMOOVE_STAGING_API_KEY')
base_url = 'https://staging.openmoove.com/api/v1/partners'

headers = {
    'X-API-KEY': api_key,
    'Content-Type': 'application/json'
}

try:
    response = requests.get(f'{base_url}/properties/', headers=headers)
    if response.status_code == 200:
        print("✅ Authentication successful!")
        print(f"Response: {response.json()}")
    else:
        print(f"❌ Authentication failed: {response.status_code}")
        print(f"Error: {response.text}")
except Exception as e:
    print(f"❌ Request failed: {e}")
```

Run with:
```bash
OPENMOOVE_STAGING_API_KEY=om_your_key python test-auth.py
```

---

## 🆘 Troubleshooting

### Common Issues

**Problem:** "Invalid API key provided"
- **Check:** API key format (starts with `om_`)
- **Check:** Environment variables are loaded correctly
- **Check:** No extra spaces or characters in key

**Problem:** "Request timed out"
- **Check:** Network connectivity
- **Check:** Correct base URL for environment
- **Note:** Staging database may take time to warm up

**Problem:** "SSL certificate error"
- **Check:** Using HTTPS, not HTTP
- **Check:** System certificates are up to date
- **Check:** No proxy interference

### Debug Checklist
- [ ] API key included in `X-API-KEY` header
- [ ] API key format correct (`om_xxxx`)
- [ ] Using correct base URL for environment
- [ ] Content-Type header set to `application/json`
- [ ] Using HTTPS, not HTTP
- [ ] Environment variables loaded correctly

---

## 📞 Support

If you're experiencing authentication issues:

**Email:** api-support@openmoove.com

**Include in your request:**
- Environment (staging/production)
- Request URL (remove sensitive data)
- Response status code and message
- Steps you've already tried
- Timestamp of the issue

**Response time:** Within 24 hours

---

## 🔗 Next Steps

Once authentication is working:
1. **[Complete API Reference](endpoints.md)** - Explore all available endpoints
2. **[Quick Start Guide](../guides/quick-start.md)** - Create your first property
3. **[Code Examples](../examples/)** - Implementation examples in multiple languages
4. **[Integration Patterns](../guides/integration-patterns.md)** - Common workflows and best practices