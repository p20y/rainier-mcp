# Amazon Ads MCP Server - Sequence Diagrams

This document contains sequence diagrams illustrating the key call flows in the Amazon Ads MCP Server architecture.

## 1. Tool Call Flow (Direct Auth)

This diagram shows the complete flow when an MCP client calls a tool that requires Amazon Ads API access.

```mermaid
sequenceDiagram
    participant Client as MCP Client<br/>(Claude)
    participant Server as FastMCP Server
    participant MW as Middleware Chain
    participant Tool as Tool Handler
    participant Client as AuthenticatedClient
    participant AuthMgr as AuthManager
    participant Provider as Direct Provider
    participant TokenStore as Token Store
    participant API as Amazon Ads API

    Client->>Server: POST /mcp/ (tool call)
    Server->>MW: Process request
    
    Note over MW: Sampling Middleware<br/>(if enabled)
    MW->>MW: Sample request/response
    
    Note over MW: Authentication Middleware
    MW->>AuthMgr: Get auth headers
    AuthMgr->>Provider: Get access token
    Provider->>TokenStore: Check cached token
    
    alt Token exists and valid
        TokenStore-->>Provider: Return cached token
    else Token expired or missing
        Provider->>API: Refresh token
        API-->>Provider: New access token
        Provider->>TokenStore: Cache new token
    end
    
    Provider-->>AuthMgr: Access token + client_id
    AuthMgr->>AuthMgr: Build auth headers
    AuthMgr-->>MW: Auth headers
    
    MW->>Tool: Execute tool handler
    Tool->>Client: Make API request
    
    Note over Client: Header Injection
    Client->>Client: Add Authorization header
    Client->>Client: Add Client ID header
    Client->>Client: Add Profile Scope header
    Client->>Client: Route to regional endpoint
    
    Client->>API: HTTP Request (with auth headers)
    API-->>Client: HTTP Response
    
    Client->>Client: Process response<br/>(shaping, error handling)
    Client-->>Tool: Response data
    Tool-->>MW: Tool result
    MW-->>Server: Processed result
    Server-->>Client: MCP Response
```

## 2. OAuth Authentication Flow (Direct Auth)

This diagram shows the OAuth 2.0 authorization flow for direct authentication.

```mermaid
sequenceDiagram
    participant User as User
    participant Client as MCP Client
    participant Server as FastMCP Server
    participant OAuthTool as OAuth Tools
    participant StateStore as OAuth State Store
    participant Amazon as Amazon OAuth<br/>Server
    participant Callback as OAuth Callback<br/>Handler
    participant TokenStore as Token Store

    User->>Client: "Start OAuth flow"
    Client->>Server: start_oauth_flow tool call
    Server->>OAuthTool: start_oauth_flow()
    
    OAuthTool->>StateStore: Generate secure state
    StateStore->>StateStore: Create HMAC signature
    StateStore->>StateStore: Store state (10min TTL)
    StateStore-->>OAuthTool: State token
    
    OAuthTool->>OAuthTool: Build auth URL<br/>(client_id, scope, redirect_uri, state)
    OAuthTool-->>Server: Return auth URL
    Server-->>Client: Auth URL + instructions
    Client-->>User: Display auth URL
    
    User->>Amazon: Visit auth URL
    Amazon->>User: Login & authorize
    Amazon->>Callback: Redirect with code + state
    
    Callback->>StateStore: Validate state
    StateStore->>StateStore: Check HMAC signature
    StateStore->>StateStore: Check expiration
    StateStore->>StateStore: Check replay (single-use)
    
    alt State valid
        StateStore-->>Callback: State valid
        Callback->>Amazon: Exchange code for tokens
        Amazon-->>Callback: Access token + Refresh token
        
        Callback->>TokenStore: Store tokens
        TokenStore-->>Callback: Tokens stored
        
        Callback->>Callback: Generate success page
        Callback-->>User: Success page
    else State invalid
        StateStore-->>Callback: State invalid
        Callback-->>User: Error page
    end
    
    User->>Client: "Check OAuth status"
    Client->>Server: check_oauth_status tool call
    Server->>OAuthTool: check_oauth_status()
    OAuthTool->>TokenStore: Get stored tokens
    TokenStore-->>OAuthTool: Tokens
    OAuthTool-->>Server: OAuth status
    Server-->>Client: Authentication confirmed
```

## 3. OpenBridge Authentication Flow

This diagram shows the authentication flow when using OpenBridge as a partner application.

```mermaid
sequenceDiagram
    participant Client as MCP Client<br/>(with Bearer token)
    participant Server as FastMCP Server
    participant RefreshMW as RefreshToken<br/>Middleware
    participant JWTMW as JWT Auth<br/>Middleware
    participant OpenBridge as OpenBridge API
    participant AuthMgr as AuthManager
    participant Provider as OpenBridge Provider
    participant TokenStore as Token Store
    participant API as Amazon Ads API

    Client->>Server: POST /mcp/ (with Authorization: Bearer <refresh_token>)
    Server->>RefreshMW: Process request
    
    RefreshMW->>RefreshMW: Extract Bearer token
    RefreshMW->>OpenBridge: POST /identities/{id}/auth/token<br/>(refresh_token)
    
    OpenBridge->>OpenBridge: Fetch/refresh Amazon token
    OpenBridge->>API: Get Amazon access token (if needed)
    API-->>OpenBridge: Amazon access token
    OpenBridge->>OpenBridge: Create JWT with credentials
    OpenBridge-->>RefreshMW: JWT token
    
    RefreshMW->>RefreshMW: Store JWT in context
    RefreshMW->>JWTMW: Pass request
    
    JWTMW->>JWTMW: Extract JWT from context
    JWTMW->>JWTMW: Decode JWT (verify signature)
    JWTMW->>JWTMW: Extract payload:<br/>- access_token<br/>- client_id<br/>- scope/profile_id
    
    JWTMW->>AuthMgr: Set credentials from JWT
    AuthMgr->>Provider: Update provider with credentials
    Provider->>TokenStore: Cache credentials
    TokenStore-->>Provider: Cached
    
    JWTMW->>JWTMW: Clear JWT from context
    JWTMW->>Server: Request authenticated
    
    Server->>Server: Execute tool handler
    Server->>API: API call with extracted credentials
    API-->>Server: Response
    Server-->>Client: MCP Response
```

## 4. Resource Loading Flow (Server Startup)

This diagram shows how OpenAPI resources are dynamically loaded during server startup.

```mermaid
sequenceDiagram
    participant Main as Main Entry Point
    participant Builder as ServerBuilder
    participant Loader as Resource Loader
    participant Packages as packages.json
    participant Resources as OpenAPI Resources
    participant FastMCP as FastMCP Server

    Main->>Builder: build()
    Builder->>Builder: _mount_resource_servers()
    
    Builder->>Loader: Load packages.json
    Loader->>Packages: Read packages.json
    Packages-->>Loader: Namespace mapping<br/>Aliases<br/>Defaults
    
    Builder->>Builder: Load package allowlist<br/>(from AMAZON_AD_API_PACKAGES)
    
    loop For each .json in resources/
        Builder->>Builder: Check if in allowlist
        
        alt In allowlist or no allowlist
            Builder->>Resources: Load OpenAPI spec
            Resources-->>Builder: OpenAPI JSON
            
            Builder->>Resources: Load .transform.json (if exists)
            Resources-->>Builder: Transform rules
            
            Builder->>Resources: Load .media.json (if exists)
            Resources-->>Builder: Media type config
            
            Builder->>Builder: Resolve namespace prefix<br/>(from packages.json)
            Builder->>Builder: Slim OpenAPI spec<br/>(remove unnecessary metadata)
            
            Builder->>FastMCP: Mount resource server<br/>(with prefix)
            FastMCP->>FastMCP: Generate tools from OpenAPI
            FastMCP-->>Builder: Resource mounted
        else Not in allowlist
            Builder->>Builder: Skip resource
        end
    end
    
    Builder-->>Main: Server with mounted resources
```

## 5. Token Refresh Flow

This diagram shows the automatic token refresh mechanism when a token expires.

```mermaid
sequenceDiagram
    participant Tool as Tool Handler
    participant Client as AuthenticatedClient
    participant AuthMgr as AuthManager
    participant Provider as Auth Provider
    participant TokenStore as Token Store
    participant API as Amazon Ads API<br/>(Token Endpoint)

    Tool->>Client: Make API request
    Client->>AuthMgr: Get auth headers
    AuthMgr->>Provider: Get access token
    
    Provider->>TokenStore: Get cached token
    TokenStore-->>Provider: Cached token
    
    Provider->>Provider: Check token expiration
    
    alt Token expired
        Provider->>API: POST /auth/o2/token<br/>(grant_type=refresh_token)
        API-->>Provider: New access_token + refresh_token
        
        Provider->>TokenStore: Update tokens
        TokenStore-->>Provider: Tokens updated
        
        Provider-->>AuthMgr: New access token
    else Token valid
        Provider-->>AuthMgr: Cached access token
    end
    
    AuthMgr->>AuthMgr: Build auth headers
    AuthMgr-->>Client: Auth headers
    Client->>API: API request with headers
    API-->>Client: API response
    Client-->>Tool: Response data
```

## 6. Profile Management Flow

This diagram shows how profile ID is managed and used in API calls.

```mermaid
sequenceDiagram
    participant Client as MCP Client
    participant Server as FastMCP Server
    participant Tool as Profile Tool
    participant AuthMgr as AuthManager
    participant Client as AuthenticatedClient
    participant API as Amazon Ads API

    Client->>Server: set_active_profile(profile_id)
    Server->>Tool: set_active_profile()
    Tool->>AuthMgr: Set active profile
    AuthMgr->>AuthMgr: Store profile_id<br/>(per identity)
    AuthMgr-->>Tool: Profile set
    Tool-->>Server: Success
    Server-->>Client: Profile activated
    
    Note over Client,API: Later API call
    
    Client->>Server: API tool call
    Server->>Client: Make API request
    Client->>AuthMgr: Get auth headers
    AuthMgr->>AuthMgr: Get active profile_id
    AuthMgr->>AuthMgr: Add to scope header
    AuthMgr-->>Client: Headers with profile scope
    Client->>API: Request with<br/>Amazon-Advertising-API-Scope header
    API-->>Client: Profile-specific response
    Client-->>Server: Response
    Server-->>Client: MCP response
```

## 7. Regional Routing Flow

This diagram shows how requests are routed to the correct regional endpoint.

```mermaid
sequenceDiagram
    participant Tool as Tool Handler
    participant Client as AuthenticatedClient
    participant AuthMgr as AuthManager
    participant RegionConfig as Region Config
    participant API_NA as Amazon Ads API<br/>(North America)
    participant API_EU as Amazon Ads API<br/>(Europe)
    participant API_FE as Amazon Ads API<br/>(Far East)

    Tool->>Client: Make API request
    Client->>AuthMgr: Get region
    AuthMgr->>AuthMgr: Get region from:<br/>- Active identity<br/>- Active profile<br/>- Settings default
    
    AuthMgr-->>Client: Region (na/eu/fe)
    Client->>RegionConfig: Get endpoint for region
    RegionConfig-->>Client: Regional endpoint URL
    
    alt Region = "na"
        Client->>API_NA: Request to<br/>advertising-api.amazon.com
        API_NA-->>Client: Response
    else Region = "eu"
        Client->>API_EU: Request to<br/>advertising-api-eu.amazon.com
        API_EU-->>Client: Response
    else Region = "fe"
        Client->>API_FE: Request to<br/>advertising-api-fe.amazon.com
        API_FE-->>Client: Response
    end
    
    Client-->>Tool: Response
```

## 8. Error Handling and Retry Flow

This diagram shows how errors are handled and retries are performed.

```mermaid
sequenceDiagram
    participant Client as AuthenticatedClient
    participant Retry as Retry Logic
    participant CircuitBreaker as Circuit Breaker
    participant API as Amazon Ads API

    Client->>Retry: Execute request with retry
    Retry->>CircuitBreaker: Check circuit state
    
    alt Circuit Open
        CircuitBreaker-->>Retry: Circuit open (fail fast)
        Retry-->>Client: Error (circuit open)
    else Circuit Closed or Half-Open
        Retry->>API: HTTP request
        
        alt Success
            API-->>Retry: 200 OK
            Retry->>CircuitBreaker: Record success
            CircuitBreaker->>CircuitBreaker: Reset failure count
            Retry-->>Client: Success response
        else Rate Limit (429)
            API-->>Retry: 429 Too Many Requests
            Retry->>Retry: Parse Retry-After header
            Retry->>Retry: Calculate backoff delay<br/>(exponential + jitter)
            Retry->>Retry: Wait for delay
            Retry->>CircuitBreaker: Record throttle
            Retry->>API: Retry request
        else Server Error (5xx)
            API-->>Retry: 5xx Error
            Retry->>CircuitBreaker: Record failure
            CircuitBreaker->>CircuitBreaker: Increment failure count
            
            alt Failure count < threshold
                Retry->>Retry: Calculate backoff
                Retry->>Retry: Wait and retry
                Retry->>API: Retry request
            else Failure count >= threshold
                CircuitBreaker->>CircuitBreaker: Open circuit
                CircuitBreaker-->>Retry: Circuit opened
                Retry-->>Client: Error (circuit opened)
            end
        else Client Error (4xx)
            API-->>Retry: 4xx Error (no retry)
            Retry-->>Client: Error response
        end
    end
```

## 9. Response Shaping Flow (AMC Large Responses)

This diagram shows how large AMC responses are automatically truncated.

```mermaid
sequenceDiagram
    participant API as Amazon Ads API
    participant Client as AuthenticatedClient
    participant Shaper as Response Shaper
    participant Tool as Tool Handler
    participant Client as MCP Client

    API->>Client: Large AMC response<br/>(e.g., 10,000+ items)
    Client->>Client: Detect AMC endpoint
    Client->>Shaper: Shape response
    
    Shaper->>Shaper: Check response size
    Shaper->>Shaper: Identify truncation rules<br/>(from transform.json)
    
    alt Response too large
        Shaper->>Shaper: Truncate arrays<br/>(e.g., keep first 10 items)
        Shaper->>Shaper: Add metadata:<br/>- truncated: true<br/>- original_count: 10000<br/>- returned_count: 10
        Shaper-->>Client: Shaped response
    else Response acceptable size
        Shaper-->>Client: Original response
    end
    
    Client-->>Tool: Response data
    Tool-->>Client: MCP response<br/>(with truncation metadata)
```

## 10. Download Export Flow

This diagram shows how exports are downloaded and stored locally.

```mermaid
sequenceDiagram
    participant Client as MCP Client
    participant Server as FastMCP Server
    participant Tool as Download Tool
    participant Handler as Download Handler
    participant API as Amazon Ads API<br/>(S3 Export URL)
    participant FS as File System

    Client->>Server: download_export(export_id, export_url)
    Server->>Tool: download_export()
    
    Tool->>Tool: Decode export_id<br/>(determine export type)
    Tool->>Handler: download_export(export_url, export_id, type)
    
    Handler->>Handler: Determine download directory<br/>(based on export type)
    Handler->>Handler: Generate file path
    
    Handler->>API: GET export_url<br/>(with auth headers)
    API-->>Handler: Export file (CSV/JSON)
    
    Handler->>FS: Write file to disk
    FS-->>Handler: File saved
    
    Handler->>Handler: Return file path
    Handler-->>Tool: File path
    Tool-->>Server: Download result
    Server-->>Client: File location
```

## ASCII Art Alternative (for non-Mermaid renderers)

### Tool Call Flow (ASCII)

```
┌──────────┐     ┌──────────┐     ┌──────────┐     ┌──────────┐     ┌──────────┐
│   MCP    │────▶│ FastMCP  │────▶│Middleware│────▶│  Tool    │────▶│Authentic.  │
│ Client   │     │  Server  │     │  Chain   │     │ Handler  │     │ Client   │
└──────────┘     └──────────┘     └──────────┘     └──────────┘     └──────────┘
                                                                         │
                                                                         ▼
┌──────────┐     ┌──────────┐     ┌──────────┐     ┌──────────┐     ┌──────────┐
│  Amazon  │◀────│AuthMgr   │◀────│ Provider │◀────│  Token   │     │          │
│ Ads API  │     │          │     │          │     │  Store   │     │          │
└──────────┘     └──────────┘     └──────────┘     └──────────┘     └──────────┘
```

## Notes on Sequence Diagrams

1. **Mermaid Syntax**: These diagrams use Mermaid syntax which can be rendered in:
   - GitHub/GitLab markdown
   - VS Code with Mermaid extension
   - Online Mermaid editors
   - Documentation tools (MkDocs, Docusaurus, etc.)

2. **Component Abbreviations**:
   - `AuthMgr` = AuthManager
   - `MW` = Middleware
   - `MW` = Middleware Chain

3. **Flow Patterns**:
   - **Solid arrows**: Synchronous calls
   - **Dashed arrows**: Async operations or data flow
   - **Alt blocks**: Conditional logic
   - **Notes**: Additional context

4. **Key Interactions**:
   - All flows start from MCP Client or User
   - Authentication is handled transparently by middleware
   - Token refresh happens automatically
   - Error handling includes retries and circuit breakers

