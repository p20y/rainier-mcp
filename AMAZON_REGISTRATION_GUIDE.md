# Amazon Ads API Application Registration Guide

## Step-by-Step Instructions

### Step 1: Access Amazon Developer Console

1. **Go to the Amazon Developer Console**
   - Open your browser and navigate to: https://developer.amazon.com/
   - Sign in with your Amazon account (the one associated with your Amazon Ads account)

### Step 2: Create or Select Your Application

1. **Navigate to Login with Amazon**
   - In the Amazon Developer Console, find "Login with Amazon" section
   - Click on "Create a New Security Profile" or select an existing one

2. **If Creating New Application:**
   - Click "Create a New Security Profile"
   - Fill in the required information:
     - **Security Profile Name**: Choose a descriptive name (e.g., "Amazon Ads MCP Server")
     - **Security Profile Description**: Describe your use case
     - **Privacy Notice URL**: (Optional for development, required for production)

### Step 3: Configure OAuth Settings

1. **Set Allowed Return URLs**
   - In your Security Profile settings, find the "Allowed Return URLs" section
   - **IMPORTANT**: Add this exact URL for local development:
     ```
     http://localhost:9080/auth/callback
     ```
   - Click "Save"

2. **Note Your Credentials**
   - After saving, you'll see:
     - **Client ID**: A long string (starts with `amzn1.application-oa2-client...`)
     - **Client Secret**: Another long string
   - **Copy both of these** - you'll need them in the next step

### Step 4: Request API Access

1. **Request Amazon Ads API Access**
   - Your application needs to be approved for Amazon Ads API access
   - Go to: https://advertising.amazon.com/API/docs/en-us/get-started/overview
   - Follow Amazon's process to request API access
   - This may take several days for approval

### Step 5: Update Your .env File

Once you have your Client ID and Client Secret, update your `.env` file:

```bash
# Authentication Method
AUTH_METHOD=direct

# Amazon Ads API Credentials (from Amazon Developer Console)
AMAZON_AD_API_CLIENT_ID="your-actual-client-id-here"
AMAZON_AD_API_CLIENT_SECRET="your-actual-client-secret-here"

# Region Configuration
AMAZON_ADS_REGION=na

# Server Configuration
TRANSPORT=http
HOST=0.0.0.0
PORT=9080
LOG_LEVEL=INFO

# Amazon Ads API Packages
AMAZON_AD_API_PACKAGES=profiles,accounts-ads-accounts,exports-snapshots
```

### Step 6: Restart Your Server

After updating the `.env` file, restart your MCP server:

```bash
# Stop the current server
pkill -f "amazon_ads_mcp.server"

# Start it again (it will load the new credentials)
cd /Users/pradeepsrini/projects/rainier-mcp/rainier-mcp
export PATH="$HOME/.local/bin:$PATH"
uv run python -m amazon_ads_mcp.server --transport http --port 9080
```

## Important Notes

### Callback URL Configuration

- **Local Development**: `http://localhost:9080/auth/callback`
- **Production**: `https://your-domain.com/auth/callback`

The callback URL **must match exactly** what you configure in Amazon Developer Console.

### Region Selection

The server defaults to `na` (North America) region. If you need a different region:
- `na` - North America
- `eu` - Europe  
- `fe` - Far East

Update `AMAZON_ADS_REGION` in your `.env` file accordingly.

### Security Best Practices

1. **Never commit your `.env` file** to version control
2. **Keep your Client Secret secure** - treat it like a password
3. **Use environment variables** in production instead of `.env` files
4. **Rotate credentials** if they are ever exposed

## Next Steps

After completing registration and updating credentials:

1. **Start OAuth Flow**: Use the MCP tool `start_oauth_flow` to begin authentication
2. **Complete Authorization**: Follow the OAuth flow in your browser
3. **Verify Connection**: Use `check_oauth_status` to confirm successful authentication
4. **Start Using API**: You can now use all Amazon Ads API tools through the MCP server

## Troubleshooting

### "Invalid redirect_uri" Error
- Ensure the callback URL in Amazon Developer Console **exactly matches** `http://localhost:9080/auth/callback`
- Check that there are no trailing slashes or extra characters

### "Client authentication failed" Error
- Verify your Client ID and Client Secret are correct
- Ensure there are no extra spaces or quotes in your `.env` file
- Make sure `AUTH_METHOD=direct` is set

### Application Not Approved
- Amazon Ads API access requires approval
- Check your application status in the Amazon Developer Console
- Contact Amazon support if approval is taking longer than expected

## Additional Resources

- [Amazon Ads API Documentation](https://advertising.amazon.com/API/docs/en-us/get-started/overview)
- [Login with Amazon Documentation](https://developer.amazon.com/docs/login-with-amazon/overview.html)
- [OAuth 2.0 Flow Explanation](https://oauth.net/2/)

