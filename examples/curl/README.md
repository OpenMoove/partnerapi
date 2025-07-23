# cURL Examples

Copy-paste ready cURL commands for all OpenMoove Partner API endpoints. Perfect for testing, debugging, and quick integrations.

## 🔧 Setup

Replace `om_your_key_here` with your actual API key from [staging.openmoove.com/profile/integrations/](https://staging.openmoove.com/profile/integrations/).

**Environment Variables (Recommended):**
```bash
export OPENMOOVE_API_KEY="om_your_key_here"
export OPENMOOVE_BASE_URL="https://staging.openmoove.com/api/v1/partners"
```

Then use in commands:
```bash
curl -H "X-API-KEY: $OPENMOOVE_API_KEY" "$OPENMOOVE_BASE_URL/properties/"
```

---

## 📦 Products

### List Available Products
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

---

## 👥 Clients & Properties

### Create Standard Property Sale
```bash
curl -X POST \
  -H "Content-Type: application/json" \
  -H "X-API-KEY: om_your_key_here" \
  -d '{
    "first_name": "John",
    "last_name": "Smith",
    "email": "john.smith@example.com",
    "phone_number": "+447700900001",
    "property_address": "123 Example Street, London",
    "property_postcode": "SW1A 1AA",
    "sale_method": "standard",
    "estate_agent_email": "agent@youragency.com",
    "conveyancer_email": "lawyer@lawfirm.com",
    "products": ["moove-ready-pack"]
  }' \
  "https://staging.openmoove.com/api/v1/partners/clients/"
```

### Create Modern Auction Property
```bash
curl -X POST \
  -H "Content-Type: application/json" \
  -H "X-API-KEY: om_your_key_here" \
  -d '{
    "first_name": "Jane",
    "last_name": "Auction",
    "email": "jane.auction@example.com",
    "phone_number": "+447700900002",
    "property_address": "456 Auction Avenue, Manchester",
    "property_postcode": "M1 2AB",
    "sale_method": "modern_auction",
    "estate_agent_email": "auction-agent@example.com",
    "auctioneer_email": "auctioneer@bidhouse.com",
    "products": ["moove-ready-pack"]
  }' \
  "https://staging.openmoove.com/api/v1/partners/clients/"
```

### Create Property with Full Professional Team
```bash
curl -X POST \
  -H "Content-Type: application/json" \
  -H "X-API-KEY: om_your_key_here" \
  -d '{
    "first_name": "Sarah",
    "last_name": "Complete",
    "email": "sarah.complete@example.com",
    "phone_number": "+447700900003",
    "property_address": "789 Full Service Road, Birmingham",
    "property_postcode": "B1 1AA",
    "sale_method": "standard",
    "estate_agent_email": "agent@fullservice.com",
    "conveyancer_email": "solicitor@legalfirm.com",
    "mortgage_broker_email": "broker@mortgages.com",
    "products": ["moove-ready-pack"]
  }' \
  "https://staging.openmoove.com/api/v1/partners/clients/"
```

### Minimal Property Creation (Required Fields Only)
```bash
curl -X POST \
  -H "Content-Type: application/json" \
  -H "X-API-KEY: om_your_key_here" \
  -d '{
    "first_name": "Min",
    "last_name": "Required",
    "email": "min.required@example.com",
    "phone_number": "+447700900004",
    "property_address": "101 Minimal Street",
    "property_postcode": "SW1A 1AA",
    "estate_agent_email": "agent@minimal.com"
  }' \
  "https://staging.openmoove.com/api/v1/partners/clients/"
```

---

## 🏠 Properties

### List All Properties (Default Pagination)
```bash
curl -H "X-API-KEY: om_your_key_here" \
  "https://staging.openmoove.com/api/v1/partners/properties/"
```

### List Properties (Custom Pagination)
```bash
curl -H "X-API-KEY: om_your_key_here" \
  "https://staging.openmoove.com/api/v1/partners/properties/?page=2&page_size=10"
```

### Get Property Details
Replace `TRANSACTION_ID` with actual transaction ID from property creation:
```bash
curl -H "X-API-KEY: om_your_key_here" \
  "https://staging.openmoove.com/api/v1/partners/properties/TRANSACTION_ID/"
```

**Example with Real Transaction ID:**
```bash
curl -H "X-API-KEY: om_your_key_here" \
  "https://staging.openmoove.com/api/v1/partners/properties/4fa18bac-3c62-4d92-a271-7364b0f0e479/"
```

---

## 📊 Milestones

### Get All Property Milestones
```bash
curl -H "X-API-KEY: om_your_key_here" \
  "https://staging.openmoove.com/api/v1/partners/properties/TRANSACTION_ID/milestones/"
```

### Get Current Milestone Only
```bash
curl -H "X-API-KEY: om_your_key_here" \
  "https://staging.openmoove.com/api/v1/partners/properties/TRANSACTION_ID/milestones/current/"
```

**Example with Real Transaction ID:**
```bash
curl -H "X-API-KEY: om_your_key_here" \
  "https://staging.openmoove.com/api/v1/partners/properties/4fa18bac-3c62-4d92-a271-7364b0f0e479/milestones/"
```

---

## 💬 Chat (Coming Soon)

### Get Chat History
```bash
curl -H "X-API-KEY: om_your_key_here" \
  "https://staging.openmoove.com/api/v1/partners/properties/TRANSACTION_ID/chat/"
```

### Get Chat Summary
```bash
curl -H "X-API-KEY: om_your_key_here" \
  "https://staging.openmoove.com/api/v1/partners/properties/TRANSACTION_ID/chat/summary/"
```

---

## 🧪 Testing Workflows

### Complete Property Creation & Monitoring Workflow

**Step 1: List Products**
```bash
echo "=== Step 1: List Products ==="
curl -H "X-API-KEY: om_your_key_here" \
  "https://staging.openmoove.com/api/v1/partners/products/" | jq '.'
```

**Step 2: Create Property**
```bash
echo "=== Step 2: Create Property ==="
RESPONSE=$(curl -X POST \
  -H "Content-Type: application/json" \
  -H "X-API-KEY: om_your_key_here" \
  -d '{
    "first_name": "Test",
    "last_name": "Workflow",
    "email": "test.workflow@example.com",
    "phone_number": "+447700900005",
    "property_address": "Test Workflow Street",
    "property_postcode": "SW1A 1AA",
    "estate_agent_email": "agent@test.com",
    "products": ["moove-ready-pack"]
  }' \
  "https://staging.openmoove.com/api/v1/partners/clients/")

echo $RESPONSE | jq '.'

# Extract transaction ID for next steps
TRANSACTION_ID=$(echo $RESPONSE | jq -r '.property.transactionId')
echo "Transaction ID: $TRANSACTION_ID"
```

**Step 3: Get Property Details**
```bash
echo "=== Step 3: Get Property Details ==="
curl -H "X-API-KEY: om_your_key_here" \
  "https://staging.openmoove.com/api/v1/partners/properties/$TRANSACTION_ID/" | jq '.'
```

**Step 4: Check Milestones**
```bash
echo "=== Step 4: Check Milestones ==="
curl -H "X-API-KEY: om_your_key_here" \
  "https://staging.openmoove.com/api/v1/partners/properties/$TRANSACTION_ID/milestones/" | jq '.'
```

**Step 5: List All Properties**
```bash
echo "=== Step 5: List All Properties ==="
curl -H "X-API-KEY: om_your_key_here" \
  "https://staging.openmoove.com/api/v1/partners/properties/" | jq '.'
```

---

## 🔍 Debugging Commands

### Test API Key Validity
```bash
curl -v -H "X-API-KEY: om_your_key_here" \
  "https://staging.openmoove.com/api/v1/partners/properties/" 2>&1 | head -20
```

### Check Rate Limit Headers
```bash
curl -I -H "X-API-KEY: om_your_key_here" \
  "https://staging.openmoove.com/api/v1/partners/properties/"
```

### Test Invalid API Key (Should Return 401)
```bash
curl -H "X-API-KEY: invalid_key" \
  "https://staging.openmoove.com/api/v1/partners/properties/"
```

### Test Missing Required Fields (Should Return 400)
```bash
curl -X POST \
  -H "Content-Type: application/json" \
  -H "X-API-KEY: om_your_key_here" \
  -d '{
    "first_name": "Missing",
    "last_name": "Fields"
  }' \
  "https://staging.openmoove.com/api/v1/partners/clients/"
```

---

## 📋 Error Examples

### 400 Bad Request (Missing Required Fields)
```bash
curl -X POST \
  -H "Content-Type: application/json" \
  -H "X-API-KEY: om_your_key_here" \
  -d '{
    "first_name": "John"
  }' \
  "https://staging.openmoove.com/api/v1/partners/clients/"
```

**Response:**
```json
{
  "last_name": ["This field is required."],
  "email": ["This field is required."],
  "phone_number": ["This field is required."],
  "property_address": ["This field is required."],
  "property_postcode": ["This field is required."],
  "non_field_errors": ["At least one professional email must be provided"]
}
```

### 400 Bad Request (Invalid Phone Format)
```bash
curl -X POST \
  -H "Content-Type: application/json" \
  -H "X-API-KEY: om_your_key_here" \
  -d '{
    "first_name": "John",
    "last_name": "Smith",
    "email": "john@example.com",
    "phone_number": "invalid-phone",
    "property_address": "123 Street",
    "property_postcode": "SW1A 1AA",
    "estate_agent_email": "agent@test.com"
  }' \
  "https://staging.openmoove.com/api/v1/partners/clients/"
```

**Response:**
```json
{
  "phone_number": ["Invalid phone number format"]
}
```

### 401 Unauthorized (Invalid API Key)
```bash
curl -H "X-API-KEY: invalid_key" \
  "https://staging.openmoove.com/api/v1/partners/properties/"
```

**Response:**
```json
{
  "detail": "Invalid API key provided"
}
```

### 404 Not Found (Property Doesn't Exist)
```bash
curl -H "X-API-KEY: om_your_key_here" \
  "https://staging.openmoove.com/api/v1/partners/properties/non-existent-id/"
```

**Response:**
```json
{
  "detail": "Property not found or you don't have access"
}
```

---

## 🛠️ Advanced Usage

### Save Response to File
```bash
curl -H "X-API-KEY: om_your_key_here" \
  "https://staging.openmoove.com/api/v1/partners/properties/" \
  -o properties.json

cat properties.json | jq '.'
```

### Include Timing Information
```bash
curl -w "Time: %{time_total}s\n" \
  -H "X-API-KEY: om_your_key_here" \
  "https://staging.openmoove.com/api/v1/partners/properties/"
```

### Follow Redirects (If Any)
```bash
curl -L -H "X-API-KEY: om_your_key_here" \
  "https://staging.openmoove.com/api/v1/partners/properties/"
```

### Silent Mode (No Progress)
```bash
curl -s -H "X-API-KEY: om_your_key_here" \
  "https://staging.openmoove.com/api/v1/partners/properties/" | jq '.'
```

---

## 📊 Bulk Testing Script

Save as `test-api.sh`:
```bash
#!/bin/bash

API_KEY="om_your_key_here"
BASE_URL="https://staging.openmoove.com/api/v1/partners"

echo "Testing OpenMoove Partner API..."

# Test 1: List products
echo "1. Testing products endpoint..."
curl -s -H "X-API-KEY: $API_KEY" "$BASE_URL/products/" | jq -r '.results[0].name'

# Test 2: Create property
echo "2. Creating test property..."
RESPONSE=$(curl -s -X POST \
  -H "Content-Type: application/json" \
  -H "X-API-KEY: $API_KEY" \
  -d '{
    "first_name": "Bulk",
    "last_name": "Test",
    "email": "bulk.test@example.com",
    "phone_number": "+447700900006",
    "property_address": "Bulk Test Street",
    "property_postcode": "SW1A 1AA",
    "estate_agent_email": "agent@bulktest.com"
  }' \
  "$BASE_URL/clients/")

TRANSACTION_ID=$(echo $RESPONSE | jq -r '.property.transactionId')
echo "Created property: $TRANSACTION_ID"

# Test 3: Get property details
echo "3. Getting property details..."
curl -s -H "X-API-KEY: $API_KEY" "$BASE_URL/properties/$TRANSACTION_ID/" | jq -r '.propertyPack.address.line1'

# Test 4: Get milestones
echo "4. Getting milestones..."
MILESTONE_COUNT=$(curl -s -H "X-API-KEY: $API_KEY" "$BASE_URL/properties/$TRANSACTION_ID/milestones/" | jq 'length')
echo "Property has $MILESTONE_COUNT milestones"

echo "All tests completed!"
```

Make executable and run:
```bash
chmod +x test-api.sh
./test-api.sh
```

---

## 🔗 Next Steps

- **[Python Examples](../python/)** - Complete Python SDK implementation
- **[JavaScript Examples](../javascript/)** - Node.js and browser implementations
- **[Postman Collection](../postman/)** - Interactive API testing
- **[Integration Patterns](../../guides/integration-patterns.md)** - Real-world workflows
- **[API Reference](../../api-reference/endpoints.md)** - Complete endpoint documentation