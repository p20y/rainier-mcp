# OAuth Scope Error Troubleshooting Guide

## Error: "An unknown scope was requested" (400 Bad Request)

### Root Cause
Your Login with Amazon application has not been **approved for Amazon Ads API access** yet. The scope `cpc_advertising:campaign_management` is correct, but Amazon rejects it until your application is whitelisted.

### Solution Steps

#### Step 1: Verify Application Status

1. **Go to Amazon Developer Console**
   - Visit: https://developer.amazon.com/
   - Sign in with your Amazon account

2. **Check Your Application**
   - Navigate to "Login with Amazon" → Your Security Profile
   - Look for "API Access" or "Amazon Ads API" section
   - Check if there's a status indicator showing:
     - ✅ Approved
     - ⏳ Pending Approval
     - ❌ Not Applied / Rejected

#### Step 2: Apply for Amazon Ads API Access

If you haven't applied yet:

1. **Go to Amazon Ads API Portal**
   - Visit: https://advertising.amazon.com/API/docs/en-us/get-started/overview
   - Click "Get Started" or "Request Access"

2. **Complete the Application**
   - Fill out the required information
   - Select the scopes you need (typically `cpc_advertising:campaign_management`)
   - Submit the application

3. **Wait for Approval**
   - Approval can take **several days to weeks**
   - Amazon will review your use case
   - You'll receive email notifications about the status

#### Step 3: Verify Application Configuration

While waiting for approval, ensure your application is correctly configured:

1. **Security Profile Settings**
   - ✅ Client ID matches your `.env` file
   - ✅ Client Secret matches your `.env` file
   - ✅ Allowed Return URLs includes: `http://localhost:9080/auth/callback`

2. **Application Type**
   - ✅ Application is set up as "Login with Amazon" (not Alexa Skills or other types)
   - ✅ Security Profile is active and not suspended

#### Step 4: Alternative - Use Partner App (OpenBridge)

If you need to test immediately while waiting for approval, you can use a partner application:

1. **Switch to OpenBridge Auth**
   ```bash
   # In your .env file, change:
   AUTH_METHOD=openbridge
   ```

2. **Get OpenBridge Credentials**
   - Sign up at: https://www.openbridge.com/
   - Get your API key
   - Configure it in your `.env` or Claude Desktop config

3. **Restart Server**
   - Restart the MCP server after changing auth method

### Verification Checklist

Before trying OAuth again, verify:

- [ ] Application is approved for Amazon Ads API access
- [ ] Client ID in `.env` matches Developer Console
- [ ] Client Secret in `.env` matches Developer Console  
- [ ] Callback URL `http://localhost:9080/auth/callback` is in Allowed Return URLs
- [ ] Server is running on port 9080
- [ ] `AUTH_METHOD=direct` is set in `.env`

### Common Issues

#### Issue: "Application shows as approved but still getting scope error"
- **Solution**: Wait 24-48 hours for changes to propagate
- **Solution**: Try clearing browser cache and cookies
- **Solution**: Verify you're using the correct Client ID

#### Issue: "Can't find where to apply for API access"
- **Solution**: Go directly to: https://advertising.amazon.com/API/docs/en-us/get-started/overview
- **Solution**: Look for "Request API Access" or "Get Started" button
- **Solution**: You may need to have an active Amazon Ads account first

#### Issue: "Application was rejected"
- **Solution**: Review the rejection reason in Developer Console
- **Solution**: Update your application description to be more specific
- **Solution**: Contact Amazon Ads API support for clarification
- **Solution**: Consider using a partner app (OpenBridge) instead

### Testing Without Approval

Unfortunately, there's no way to test Amazon Ads API OAuth without approval. The scope validation happens on Amazon's servers before any tokens are issued.

### Next Steps

1. **If Not Applied**: Apply for Amazon Ads API access immediately
2. **If Pending**: Wait for approval (check email regularly)
3. **If Approved**: Wait 24-48 hours, then try OAuth flow again
4. **If Need Immediate Access**: Consider using OpenBridge partner app

### Support Resources

- **Amazon Ads API Docs**: https://advertising.amazon.com/API/docs/en-us/get-started/overview
- **Developer Console**: https://developer.amazon.com/
- **Login with Amazon Docs**: https://developer.amazon.com/docs/login-with-amazon/overview.html
- **Amazon Ads API Support**: Contact through the Developer Console

