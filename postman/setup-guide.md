# OpenMoove Partner API - Postman Collection Setup

## Quick Start Guide

The OpenMoove Partner API Postman collection is completely self-contained and ready to use with minimal setup.

### Step 1: Import Collection

1. Open Postman
2. Click "Import" button
3. Import `OpenMoove_Partner_API.postman_collection.json`

### Step 2: Configure API Key

1. Right-click on the "OpenMoove Partner API" collection
2. Select "Edit"
3. Go to the "Variables" tab
4. Update the `api_key` value with your actual API key
5. Save the collection

### Step 3: Choose Environment (Optional)

By default, the collection uses production URLs. To test with staging:

1. In collection variables, change `current_base_url` from `{{base_url}}` to `{{staging_url}}`
2. Save the collection

### Step 4: Run Tests

1. Start with "Authentication Test" → "Verify API Key" to confirm your setup
2. Run "Client Management" requests to create test data
3. Use "Property Management" and "Milestone Tracking" to explore created data

## Collection Features

### 🔧 Self-Contained Setup
- All variables are built into the collection
- No separate environment files needed
- One-click import and setup

### 🔄 Smart Test Flow
- Variables automatically capture created IDs
- Later requests use previously created data
- Tests validate responses and set expectations

### 📊 Comprehensive Testing
- Success scenario testing
- Error handling validation
- Rate limiting verification
- Authentication testing

### 🌐 Environment Flexibility
- Easy switching between production and staging
- Clear variable descriptions and purposes
- Auto-managed test data variables

## Variable Reference

### User-Configurable Variables
| Variable | Purpose | Default Value |
|----------|---------|---------------|
| `api_key` | Your API key from OpenMoove | `your_api_key_here` |
| `current_base_url` | Active environment | `{{base_url}}` (production) |

### Auto-Managed Variables
| Variable | Purpose | Set By |
|----------|---------|--------|
| `test_property_id` | Created property ID | Client creation tests |
| `test_mover_id` | Created mover ID | Client creation tests |
| `sample_property_id` | Sample from property list | Property list test |
| `current_property_id` | Currently selected property | Pre-request scripts |

### Environment URLs
| Variable | Value | Purpose |
|----------|-------|---------|
| `base_url` | `https://api.openmoove.com` | Production API |
| `staging_url` | `https://staging.openmoove.com` | Testing/staging API |

## Request Organization

### 📋 Authentication Test
- Verify API key validity
- Check authentication setup

### 👥 Client Management
- Create standard sale clients
- Create auction sale clients  
- Test validation errors

### 🏠 Property Management
- List properties with pagination
- Get detailed property information

### 📊 Milestone Tracking
- View all property milestones
- Get current milestone status

### 💬 Chat Integration
- View chat history (mock)
- Get chat summaries (mock)

### ❌ Error Handling Tests
- Test 401 Unauthorized responses
- Test 404 Not Found responses

### ⚡ Rate Limiting Tests
- Check rate limit headers
- Monitor API usage

## Best Practices

### 🎯 Testing Workflow
1. **Start Fresh**: Run "Verify API Key" first
2. **Create Data**: Use "Create Client" requests to generate test data
3. **Explore**: Use property and milestone endpoints with created data
4. **Test Errors**: Run error handling tests to understand failure modes

### 🔧 Troubleshooting
- **401 Errors**: Check your `api_key` variable is set correctly
- **404 Errors**: Ensure you've created test data first
- **No Data**: Run client creation requests before property requests

### 🌍 Environment Switching
- **Development**: Use `{{staging_url}}` for safe testing
- **Production**: Use `{{base_url}}` for live integration
- **Testing**: Use test phone numbers like `+447700900001`

## Getting Your API Key

Contact OpenMoove support to obtain your Partner API key:
- **Email**: api-support@openmoove.com  
- **Include**: Your company name and integration requirements
- **Expect**: API key and any additional setup instructions

## Additional Resources

- **API Guide**: `PARTNER_API_GUIDE.md` - Comprehensive documentation
- **Quick Reference**: `PARTNER_API_QUICK_REFERENCE.md` - Fast lookup guide
- **Validation Guide**: `PARTNER_API_VALIDATION_GUIDE.md` - Field validation rules

## Support

For technical support with the Postman collection or API integration:
- Review the test console output for detailed error information
- Check the comprehensive guides for troubleshooting steps
- Contact api-support@openmoove.com with specific error details