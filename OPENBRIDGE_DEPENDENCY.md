# OpenBridge Dependency Explanation

## Overview

**OpenBridge is NOT a Python package dependency** - it is an **optional third-party authentication service** that acts as a partner application for Amazon Ads API access.

## What is OpenBridge?

OpenBridge is a **partner application provider** that offers a ready-to-go gateway to the Amazon Ads API. Instead of creating your own Amazon Ads API application (BYOA - Bring Your Own App), you can use OpenBridge's pre-approved application.

### Key Points

1. **No Python Package Required**: OpenBridge is not listed in `pyproject.toml` or `requirements.txt`
2. **HTTP API Service**: The code makes HTTP requests to OpenBridge's REST API
3. **Optional Authentication Method**: One of two supported authentication methods:
   - **Direct Auth** (BYOA): Use your own Amazon Ads API application
   - **OpenBridge Auth**: Use OpenBridge's partner application

## How OpenBridge is Used

### Architecture

```
┌─────────────────┐
│  MCP Client     │
│  (Claude)       │
└────────┬────────┘
         │ Authorization: Bearer <openbridge_refresh_token>
         ▼
┌─────────────────┐
│  MCP Server     │
│  (This Code)    │
└────────┬────────┘
         │
         ▼
┌─────────────────┐     HTTP API Calls     ┌─────────────────┐
│ OpenBridge      │◀───────────────────────│ OpenBridge      │
│ Provider        │                        │ API Service     │
│ (Code Module)   │───────────────────────▶│ (External)      │
└────────┬────────┘     JWT Token          └─────────────────┘
         │
         ▼
┌─────────────────┐
│ Amazon Ads API  │
└─────────────────┘
```

### Code Implementation

The OpenBridge integration is implemented in:
- **`src/amazon_ads_mcp/auth/providers/openbridge.py`**: OpenBridge provider implementation
- **`src/amazon_ads_mcp/middleware/authentication.py`**: Middleware for JWT conversion

### HTTP API Endpoints Used

The code makes HTTP requests to OpenBridge's API endpoints:

1. **Authentication Endpoint**:
   ```
   POST https://authentication.api.openbridge.io/auth/api/refresh
   ```
   - Converts refresh token to JWT
   - Default: `https://authentication.api.openbridge.io`
   - Configurable via `OPENBRIDGE_AUTH_BASE_URL`

2. **Identity Endpoint**:
   ```
   GET https://remote-identity.api.openbridge.io/v1/identities
   ```
   - Lists available Amazon Ads identities
   - Default: `https://remote-identity.api.openbridge.io`
   - Configurable via `OPENBRIDGE_IDENTITY_BASE_URL`

3. **Service Endpoint**:
   ```
   POST https://service.api.openbridge.io/v1/identities/{id}/auth/token
   ```
   - Gets Amazon Ads access token for an identity
   - Default: `https://service.api.openbridge.io`
   - Configurable via `OPENBRIDGE_SERVICE_BASE_URL`

## Dependency Analysis

### Python Package Dependencies

**OpenBridge is NOT a Python package dependency**. The codebase uses standard HTTP libraries to communicate with OpenBridge's REST API:

- `httpx`: For HTTP requests (already a dependency)
- `pyjwt`: For JWT token parsing (already a dependency)
- No OpenBridge-specific Python SDK required

### External Service Dependency

**OpenBridge IS an external service dependency** when using `AUTH_METHOD=openbridge`:

- **Required**: OpenBridge account and API key
- **Required**: Network access to OpenBridge API endpoints
- **Optional**: Can be replaced with Direct Auth (BYOA)

## Configuration

### Environment Variables

To use OpenBridge authentication, set:

```bash
# Required
AUTH_METHOD=openbridge
OPENBRIDGE_REFRESH_TOKEN=your-openbridge-api-key
# OR
OPENBRIDGE_API_KEY=your-openbridge-api-key

# Optional - Custom endpoints
OPENBRIDGE_AUTH_BASE_URL=https://authentication.api.openbridge.io
OPENBRIDGE_IDENTITY_BASE_URL=https://remote-identity.api.openbridge.io
OPENBRIDGE_SERVICE_BASE_URL=https://service.api.openbridge.io
```

### Middleware Configuration

When using OpenBridge, additional middleware is required:

```bash
# Required for OpenBridge
REFRESH_TOKEN_ENABLED=true
AUTH_ENABLED=true
JWT_VALIDATION_ENABLED=false  # OpenBridge handles validation
```

## Comparison: Direct vs OpenBridge

| Aspect | Direct Auth (BYOA) | OpenBridge Auth |
|--------|-------------------|-----------------|
| **Python Package** | No | No |
| **External Service** | No (direct to Amazon) | Yes (OpenBridge API) |
| **Setup Complexity** | High (app approval needed) | Low (just API key) |
| **Approval Time** | Days to weeks | Instant (pre-approved) |
| **Cost** | Free | OpenBridge subscription |
| **Identity Management** | Manual | Managed by OpenBridge |
| **Token Management** | OAuth flow required | Refresh token only |

## When to Use OpenBridge

### Use OpenBridge When:
- ✅ You need immediate access (no approval wait)
- ✅ You want simplified identity management
- ✅ You're building a service/product that needs reliable auth
- ✅ You have multiple Amazon Ads accounts to manage
- ✅ You prefer managed authentication over DIY

### Use Direct Auth When:
- ✅ You want full control over authentication
- ✅ You don't want external service dependencies
- ✅ You're okay waiting for Amazon approval
- ✅ You want to avoid third-party costs
- ✅ You're building internal tools

## Code Dependencies

### Direct Code Dependencies

The OpenBridge provider code depends on:

```python
# Standard libraries (no external packages)
import httpx          # Already in dependencies
import jwt            # Already in dependencies (PyJWT)
import os             # Standard library
from datetime import datetime, timedelta, timezone  # Standard library
```

### No OpenBridge SDK

The code does **NOT** use:
- ❌ `openbridge-python` (doesn't exist)
- ❌ `openbridge-sdk` (doesn't exist)
- ❌ Any OpenBridge-specific Python package

Instead, it makes direct HTTP requests to OpenBridge's REST API.

## Removing OpenBridge Dependency

If you want to remove OpenBridge support entirely:

1. **Remove Provider Code**:
   - Delete `src/amazon_ads_mcp/auth/providers/openbridge.py`
   - Remove from `src/amazon_ads_mcp/auth/providers/__init__.py`

2. **Remove Middleware**:
   - Remove OpenBridge-specific middleware configuration
   - Remove `create_openbridge_config()` function

3. **Remove Tools**:
   - Remove identity management tools (if only used for OpenBridge)
   - Update `builtin_tools.py` to not register identity tools

4. **Update Documentation**:
   - Remove OpenBridge references from README
   - Update configuration examples

**Note**: The codebase is designed to work without OpenBridge - it's completely optional.

## Network Dependencies

When using OpenBridge, the server requires:

1. **Outbound HTTPS Access** to:
   - `authentication.api.openbridge.io`
   - `remote-identity.api.openbridge.io`
   - `service.api.openbridge.io`

2. **DNS Resolution** for OpenBridge domains

3. **No Firewall Blocking** of OpenBridge endpoints

## Security Considerations

1. **Token Storage**: OpenBridge refresh tokens should be stored securely
2. **HTTPS Only**: All OpenBridge API calls use HTTPS
3. **No Credentials in Code**: Tokens come from environment variables or headers
4. **JWT Validation**: OpenBridge JWTs are trusted (signature verification disabled by design)

## Summary

- **Python Package**: ❌ Not required
- **External Service**: ✅ Required only when `AUTH_METHOD=openbridge`
- **Network Access**: ✅ Required to OpenBridge API endpoints
- **Optional**: ✅ Can use Direct Auth instead
- **Code Dependency**: ✅ HTTP client libraries only (already in dependencies)

OpenBridge is an **optional authentication service**, not a required code dependency. The server can function completely without OpenBridge by using Direct Auth (BYOA) instead.

