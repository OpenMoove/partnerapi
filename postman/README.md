# OpenMoove Partner API - Postman Collection

## Overview

This directory contains a comprehensive Postman collection for testing the OpenMoove Partner API. The collection includes all endpoints, authentication setup, and automated tests.

## Files

- `OpenMoove_Partner_API.postman_collection.json` - Complete Postman collection
- `setup-guide.md` - Detailed setup instructions

## Quick Setup

1. **Import Collection**
   - Open Postman
   - Click "Import"
   - Select `OpenMoove_Partner_API.postman_collection.json`

2. **Configure API Key**
   - Right-click collection → "Edit"
   - Go to "Variables" tab
   - Set `api_key` value to your actual API key
   - Save

3. **Start Testing**
   - Run "Verify API Key" first
   - Create test data with "Create Client" requests
   - Explore with other endpoints

## Collection Features

### 🔧 Self-Contained
- No separate environment files needed
- All variables built into collection
- One-click import and setup

### 🔄 Smart Test Flow
- Variables automatically capture created IDs
- Later requests use previously created data
- Tests validate responses

### 📊 Comprehensive Testing
- Success scenarios
- Error handling validation
- Authentication testing
- Rate limiting checks

### 🌐 Environment Switching
- Production URLs by default
- Easy staging switch
- Auto-managed test variables

## Request Organization

### 📋 Authentication Test
- Verify API key validity

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
- Test 401 Unauthorized
- Test 404 Not Found

### ⚡ Rate Limiting Tests
- Check rate limit headers

## Variables Reference

### User-Configurable
| Variable | Purpose | Default |
|----------|---------|---------|
| `api_key` | Your API key | `your_api_key_here` |
| `current_base_url` | Active environment | Production |

### Auto-Managed
| Variable | Purpose |
|----------|---------|
| `test_property_id` | Created property ID |
| `test_mover_id` | Created mover ID |
| `sample_property_id` | Sample from list |
| `current_property_id` | Selected property |

## Environment URLs

| Variable | Value | Purpose |
|----------|-------|---------|
| `base_url` | `https://api.openmoove.com` | Production |
| `staging_url` | `https://staging.openmoove.com` | Testing |

## Best Practices

### 🎯 Testing Workflow
1. Start with "Verify API Key"
2. Create test data with client requests
3. Explore properties and milestones
4. Test error scenarios

### 🔧 Troubleshooting
- **401 Errors**: Check API key in variables
- **404 Errors**: Create test data first
- **No Data**: Run client creation first

### 🌍 Environment Switching
- **Development**: Use staging URL
- **Production**: Use production URL
- **Testing**: Use test phone numbers

## Getting Your API Key

1. Log in to OpenMoove
2. Go to Profile → Integrations
3. Find "Partner API Access" section
4. Click "Generate API Key"
5. Copy and store securely

## Support

For collection issues:
- Check setup guide
- Review console output
- Contact api-support@openmoove.com

---

*For detailed API documentation, see the [Developer Guide](../developer-guide.md)*