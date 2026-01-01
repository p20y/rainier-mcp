# Amazon Ads MCP Server Architecture

## Overview

The Amazon Ads MCP Server is a modular Python framework that bridges the Model Context Protocol (MCP) with the Amazon Advertising API. It uses a plugin-based architecture with dynamic OpenAPI resource loading, multiple authentication providers, and middleware-based request processing.

## High-Level Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    MCP Client (Claude)                       │
│                  /mcp/ endpoint (HTTP)                        │
└──────────────────────┬──────────────────────────────────────┘
                       │
                       ▼
┌─────────────────────────────────────────────────────────────┐
│              FastMCP Server (Main Entry Point)              │
│  ┌──────────────────────────────────────────────────────┐  │
│  │  Middleware Chain                                      │  │
│  │  - Sampling Middleware                                 │  │
│  │  - Authentication Middleware (OpenBridge/Direct)      │  │
│  │  - OAuth Middleware                                    │  │
│  └──────────────────────────────────────────────────────┘  │
│  ┌──────────────────────────────────────────────────────┐  │
│  │  Built-in Tools                                        │  │
│  │  - Profile Management                                  │  │
│  │  - Region Management                                   │  │
│  │  - OAuth Tools (Direct Auth)                           │  │
│  │  - Identity Tools (OpenBridge)                         │  │
│  │  - Download Tools                                      │  │
│  └──────────────────────────────────────────────────────┘  │
│  ┌──────────────────────────────────────────────────────┐  │
│  │  Dynamic Resource Servers (OpenAPI-based)              │  │
│  │  - Campaign Management                                 │  │
│  │  - Reporting                                           │  │
│  │  - DSP Operations                                      │  │
│  │  - AMC Workflows                                       │  │
│  │  - ... (50+ API resources)                             │  │
│  └──────────────────────────────────────────────────────┘  │
└──────────────────────┬──────────────────────────────────────┘
                       │
                       ▼
┌─────────────────────────────────────────────────────────────┐
│              AuthenticatedClient (HTTP Layer)                │
│  - Header Injection                                         │
│  - Regional Routing                                         │
│  - Media Type Negotiation                                   │
│  - Response Shaping                                         │
└──────────────────────┬──────────────────────────────────────┘
                       │
                       ▼
┌─────────────────────────────────────────────────────────────┐
│              AuthManager (Authentication Layer)              │
│  ┌──────────────────────────────────────────────────────┐  │
│  │  Provider Registry                                    │  │
│  │  - Direct Provider (BYOA)                             │  │
│  │  - OpenBridge Provider                                │  │
│  │  - Custom Providers (extensible)                      │  │
│  └──────────────────────────────────────────────────────┘  │
│  ┌──────────────────────────────────────────────────────┐  │
│  │  Token Store                                          │  │
│  │  - In-Memory (default)                               │  │
│  │  - Encrypted Persistence (optional)                   │  │
│  └──────────────────────────────────────────────────────┘  │
└──────────────────────┬──────────────────────────────────────┘
                       │
                       ▼
┌─────────────────────────────────────────────────────────────┐
│              Amazon Advertising API                           │
│  - Regional Endpoints (NA/EU/FE)                            │
│  - OAuth 2.0 Authentication                                  │
│  - RESTful API Operations                                    │
└─────────────────────────────────────────────────────────────┘
```

## Core Components

### 1. Server Layer (`server/`)

#### `mcp_server.py` - Main Entry Point
- **Purpose**: Entry point for the MCP server
- **Responsibilities**:
  - Parse command-line arguments (transport, host, port)
  - Initialize logging and security
  - Create server instance via `ServerBuilder`
  - Handle cleanup on shutdown
  - Support multiple transports (stdio, http, streamable-http)

#### `server_builder.py` - Server Construction
- **Purpose**: Builder pattern for server initialization
- **Key Methods**:
  - `build()`: Orchestrates server setup
  - `_create_main_server()`: Creates FastMCP instance
  - `_setup_middleware()`: Configures middleware chain
  - `_setup_http_client()`: Creates authenticated HTTP client
  - `_mount_resource_servers()`: Dynamically loads OpenAPI resources
  - `_setup_builtin_tools()`: Registers built-in MCP tools
  - `_setup_oauth_callback()`: Sets up OAuth callback route

#### `builtin_tools.py` - Built-in MCP Tools
- **Purpose**: Registers core MCP tools
- **Tool Categories**:
  - **Profile Management**: `set_active_profile`, `get_active_profile`, `clear_active_profile`
  - **Region Management**: `set_region`, `get_region`, `list_regions`, `get_routing_state`
  - **OAuth Tools** (Direct Auth): `start_oauth_flow`, `check_oauth_status`, `refresh_oauth_token`, `clear_oauth_tokens`
  - **Identity Tools** (OpenBridge): `set_active_identity`, `get_active_identity`, `list_identities`
  - **Download Tools**: `download_export`, `list_downloads`
  - **Sampling Tools**: `test_sampling` (if enabled)

#### `openapi_utils.py` - OpenAPI Processing
- **Purpose**: Processes OpenAPI specifications for tool generation
- **Functions**:
  - `slim_openapi_for_tools()`: Optimizes OpenAPI specs for MCP context limits
  - Removes unnecessary metadata
  - Truncates long descriptions

#### `sidecar_loader.py` - Resource Loading
- **Purpose**: Loads OpenAPI resources with sidecar files
- **Features**:
  - Loads `.json` (OpenAPI spec)
  - Loads `.transform.json` (response transformations)
  - Loads `.media.json` (media type configurations)
  - Loads `.manifest.json` (metadata)

### 2. Authentication Layer (`auth/`)

#### `manager.py` - AuthManager (Singleton)
- **Purpose**: Central authentication coordinator
- **Key Features**:
  - Singleton pattern for consistent state
  - Provider registry integration
  - Identity management
  - Token caching and refresh
  - Profile scope management
  - Region-specific endpoint handling

#### `base.py` - Provider Base Classes
- **Purpose**: Abstract base classes for providers
- **Classes**:
  - `BaseAmazonAdsProvider`: Base for Amazon Ads providers
  - `BaseIdentityProvider`: Base for identity management
  - `ProviderConfig`: Configuration data structure

#### `providers/direct.py` - Direct Authentication
- **Purpose**: "Bring Your Own App" (BYOA) authentication
- **Features**:
  - OAuth 2.0 flow support
  - Token refresh management
  - Region-specific endpoints
  - Profile ID management

#### `providers/openbridge.py` - OpenBridge Provider
- **Purpose**: Partner application authentication
- **Features**:
  - JWT token conversion
  - Identity listing and management
  - Token caching
  - Custom endpoint support

#### `registry.py` - Provider Registry
- **Purpose**: Plugin system for authentication providers
- **Features**:
  - Dynamic provider registration
  - Provider discovery
  - Factory pattern for provider creation

#### `token_store.py` - Token Storage
- **Purpose**: Secure token management
- **Features**:
  - In-memory storage (default)
  - Encrypted persistence (optional)
  - Token expiration tracking
  - Multi-identity support

#### `oauth_state_store.py` - OAuth State Management
- **Purpose**: Secure OAuth state validation
- **Features**:
  - HMAC-signed state tokens
  - Replay attack prevention
  - Expiration handling
  - IP/User-Agent validation

### 3. Middleware Layer (`middleware/`)

#### `authentication.py` - Auth Middleware
- **Purpose**: Request authentication processing
- **Components**:
  - `RefreshTokenMiddleware`: Converts refresh tokens to JWTs
  - `JWTAuthenticationMiddleware`: Validates JWT tokens
  - `create_auth_middleware()`: Factory for middleware chain

#### `oauth.py` - OAuth Middleware
- **Purpose**: OAuth callback handling
- **Features**:
  - Callback route registration
  - State validation
  - Token exchange
  - Error handling

#### `sampling.py` - Sampling Middleware
- **Purpose**: Request/response sampling for testing
- **Features**:
  - Conditional sampling based on config
  - Fallback mechanisms
  - Context attachment

### 4. HTTP Client Layer (`utils/http_client.py`)

#### `AuthenticatedClient` - Enhanced HTTP Client
- **Purpose**: Authenticated HTTP client for Amazon Ads API
- **Key Features**:
  - **Header Injection**: Automatically adds auth headers
  - **Header Scrubbing**: Removes conflicting headers from FastMCP
  - **Regional Routing**: Routes to correct regional endpoint
  - **Media Type Negotiation**: Handles content type negotiation
  - **Response Shaping**: Truncates large AMC responses
  - **Profile Scope Management**: Adds profile ID to scope headers

#### `utils/http/` - HTTP Utilities
- **resilient_client.py**: Circuit breaker and retry logic
- **circuit_breaker.py**: Circuit breaker pattern implementation
- **retry.py**: Exponential backoff retry logic
- **client_manager.py**: HTTP client connection pooling

### 5. Configuration Layer (`config/`)

#### `settings.py` - Application Settings
- **Purpose**: Centralized configuration management
- **Features**:
  - Pydantic-based settings
  - Environment variable loading
  - `.env` file support
  - Type validation
  - Default values

#### `sampling.py` - Sampling Configuration
- **Purpose**: Sampling feature configuration
- **Features**:
  - Sampling rate configuration
  - Handler configuration
  - Fallback settings

### 6. Tools Layer (`tools/`)

#### `profile.py` - Profile Management
- **Purpose**: Profile ID management tools
- **Functions**:
  - `set_active_profile()`: Set active profile
  - `get_active_profile()`: Get current profile
  - `clear_active_profile()`: Clear profile

#### `region.py` - Region Management
- **Purpose**: Region/marketplace management
- **Functions**:
  - `set_region()`: Set active region
  - `get_region()`: Get current region
  - `list_regions()`: List available regions

#### `identity.py` - Identity Management
- **Purpose**: Identity management (OpenBridge)
- **Functions**:
  - `set_active_identity()`: Set active identity
  - `get_active_identity()`: Get current identity
  - `list_identities()`: List available identities

#### `oauth.py` - OAuth Tools
- **Purpose**: OAuth flow management
- **Classes**:
  - `OAuthTools`: OAuth operation handler
  - Methods: `start_oauth_flow()`, `check_oauth_status()`, `refresh_access_token()`, `clear_oauth_tokens()`

### 7. Models Layer (`models/`)

#### `base_models.py` - Base Data Models
- **Purpose**: Pydantic models for API data
- **Models**:
  - `Token`: Access token representation
  - `Identity`: Identity information
  - `AuthCredentials`: Authentication credentials

#### `api_responses.py` - API Response Models
- **Purpose**: Typed response models
- **Features**:
  - Type-safe API responses
  - Validation
  - Serialization

### 8. Utilities Layer (`utils/`)

#### `region_config.py` - Region Configuration
- **Purpose**: Regional endpoint management
- **Features**:
  - Region-to-endpoint mapping
  - OAuth endpoint configuration
  - Region validation

#### `header_resolver.py` - Header Name Resolution
- **Purpose**: Handles header name variations
- **Features**:
  - Case-insensitive header matching
  - Header name normalization
  - Scope header detection

#### `media/` - Media Type Handling
- **Purpose**: Content type negotiation
- **Components**:
  - `negotiator.py`: Media type negotiation
  - `types.py`: Media type definitions

#### `openapi/` - OpenAPI Utilities
- **Purpose**: OpenAPI specification handling
- **Components**:
  - `loader.py`: OpenAPI spec loading
  - `refs.py`: Reference resolution
  - `json.py`: JSON schema handling

## Data Flow

### 1. Tool Call Flow

```
MCP Client Request
    ↓
FastMCP Server
    ↓
Middleware Chain
    ├─ Sampling Middleware (if enabled)
    ├─ Authentication Middleware
    │   ├─ Extract Authorization header
    │   ├─ Convert refresh token → JWT (if needed)
    │   └─ Validate JWT
    └─ OAuth Middleware (if applicable)
    ↓
Tool Handler (Built-in or OpenAPI-generated)
    ↓
AuthenticatedClient
    ├─ Header Injection
    │   ├─ Get auth headers from AuthManager
    │   ├─ Add Client ID
    │   ├─ Add Profile Scope
    │   └─ Add Region-specific headers
    ├─ Regional Routing
    │   └─ Route to correct endpoint (NA/EU/FE)
    ├─ Media Type Negotiation
    │   └─ Set Accept headers based on OpenAPI spec
    └─ Request Execution
    ↓
Amazon Ads API
    ↓
Response Processing
    ├─ Response Shaping (for large AMC responses)
    ├─ Error Handling
    └─ Token Refresh (if needed)
    ↓
MCP Client Response
```

### 2. Authentication Flow

#### Direct Auth (BYOA)
```
1. User calls start_oauth_flow
   ↓
2. Server generates OAuth URL with state
   ↓
3. User visits URL, authorizes
   ↓
4. Amazon redirects to /auth/callback
   ↓
5. Server validates state, exchanges code for tokens
   ↓
6. Tokens stored in TokenStore
   ↓
7. Tokens used for API calls
```

#### OpenBridge Auth
```
1. Client sends Authorization header with refresh token
   ↓
2. RefreshTokenMiddleware intercepts
   ↓
3. Converts refresh token to JWT via OpenBridge API
   ↓
4. JWTAuthenticationMiddleware validates JWT
   ↓
5. Extracts credentials from JWT payload
   ↓
6. Credentials used for API calls
```

### 3. Resource Loading Flow

```
Server Startup
    ↓
ServerBuilder._mount_resource_servers()
    ↓
Load packages.json (namespace mapping, aliases)
    ↓
Load package allowlist from AMAZON_AD_API_PACKAGES
    ↓
For each .json file in resources/:
    ├─ Check if in allowlist
    ├─ Load OpenAPI spec
    ├─ Load sidecar files (.transform.json, .media.json)
    ├─ Slim OpenAPI spec for MCP
    ├─ Generate tool prefix from namespace
    └─ Mount as FastMCP resource server
    ↓
Tools available via MCP
```

## Key Design Patterns

### 1. Builder Pattern
- **ServerBuilder**: Encapsulates complex server initialization
- **Benefits**: Testable, maintainable, clear initialization order

### 2. Singleton Pattern
- **AuthManager**: Single instance across application
- **Benefits**: Consistent state, shared token store

### 3. Provider Pattern
- **Provider Registry**: Pluggable authentication providers
- **Benefits**: Extensible, testable, supports multiple auth methods

### 4. Middleware Pattern
- **Middleware Chain**: Request processing pipeline
- **Benefits**: Composable, testable, separation of concerns

### 5. Factory Pattern
- **Token Store Factory**: Creates appropriate token store
- **Provider Factory**: Creates providers based on config
- **Benefits**: Flexible, configurable

## Security Features

1. **Token Encryption**: Fernet symmetric encryption for persisted tokens
2. **OAuth State Validation**: HMAC-signed state tokens prevent CSRF
3. **Header Scrubbing**: Removes potentially malicious headers
4. **Secure Logging**: Masks sensitive data in logs
5. **Token Expiration**: Automatic token refresh
6. **Replay Prevention**: OAuth state tokens are single-use

## Performance Optimizations

1. **Connection Pooling**: HTTP client connection reuse
2. **Token Caching**: Reduces authentication overhead
3. **OpenAPI Slimming**: Reduces MCP context usage
4. **Response Shaping**: Truncates large AMC responses
5. **Circuit Breaker**: Prevents cascading failures
6. **Exponential Backoff**: Handles rate limiting

## Extension Points

1. **Custom Providers**: Register new auth providers via `@register_provider`
2. **Custom Middleware**: Add middleware to processing chain
3. **Custom Tools**: Add built-in tools via `@server.tool`
4. **Response Transformations**: Add `.transform.json` files
5. **Media Type Handlers**: Extend `MediaTypeRegistry`

## Testing Architecture

- **Unit Tests**: Test individual components in isolation
- **Integration Tests**: Test component interactions
- **Mock Providers**: Test with mock authentication
- **Fixture System**: Reusable test data and mocks

## Deployment Architecture

- **Docker**: Containerized deployment
- **Environment Variables**: Configuration via `.env` or env vars
- **Multiple Transports**: stdio, http, streamable-http
- **Health Checks**: Server health monitoring
- **Logging**: Structured logging with levels

