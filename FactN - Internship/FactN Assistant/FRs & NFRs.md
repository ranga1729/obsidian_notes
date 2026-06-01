# Comprehensive Functional & Non-Functional Requirements for ChatN Platform

Based on my deep understanding of the platform architecture, here is a detailed breakdown of all functional and non-functional requirements.

---

## FUNCTIONAL REQUIREMENTS

### FR1: User Management & Authentication

#### FR1.1 User Registration
- **Req**: Platform users can register with email and password
- **Acceptance Criteria**:
  - User submits email, password, password confirmation
  - System validates email format (RFC 5322 compliant)
  - System validates password strength (minimum 12 characters, uppercase, lowercase, digit, special char)
  - System creates unverified user account in PostgreSQL
  - System sends verification email with 24-hour expiry link
  - User clicks link → account marked as verified
  - Verified users can login and create projects
  - Unverified users receive error on login attempt

#### FR1.2 User Login
- **Req**: Users can authenticate with email and password
- **Acceptance Criteria**:
  - User submits email and password
  - System validates credentials against password hash in PostgreSQL
  - On success: system issues RS256 JWT access token (15-minute TTL)
  - On success: system issues HttpOnly refresh token cookie (7-day TTL)
  - On failure: return 401, do not leak whether email exists
  - Rate limiting: max 10 failed attempts per IP per minute (brute-force mitigation)
  - Refresh token stored as HMAC-SHA256 hash in PostgreSQL

#### FR1.3 Token Refresh
- **Req**: Users can refresh their access token using the refresh token
- **Acceptance Criteria**:
  - User sends refresh token (from HttpOnly cookie)
  - System validates token hash against PostgreSQL
  - System checks token has not been revoked or expired
  - System issues new access token and new refresh token (rotation)
  - Old refresh token immediately invalidated
  - Return new tokens to client
  - Expired refresh tokens cannot be used to issue new tokens

#### FR1.4 User Logout
- **Req**: Users can logout and terminate their session
- **Acceptance Criteria**:
  - User sends logout request with valid access token
  - System invalidates the refresh token (set revoked_at)
  - System clears the HttpOnly cookie on client
  - User's access token remains valid for its remaining TTL (user can see it expired after 15 min)
  - Subsequent API calls with invalidated refresh token fail

#### FR1.5 Password Reset
- **Req**: Users can reset forgotten password
- **Acceptance Criteria**:
  - User submits email to /auth/forgot-password
  - System generates 1-hour reset link regardless of whether email exists (prevent email enumeration)
  - If email exists: send reset link
  - If email doesn't exist: silently succeed (return same message)
  - User clicks link: redirected to password reset page
  - User enters new password + confirmation
  - System validates password strength
  - System updates password hash in PostgreSQL
  - All existing refresh tokens for that user are invalidated
  - User can immediately login with new password

#### FR1.6 User Profile Management
- **Req**: Users can view and update their profile
- **Acceptance Criteria**:
  - GET /auth/profile returns current user's email, created_at, subscription_plan, is_verified
  - User can update email (must re-verify new email)
  - User can update display name (if applicable)
  - User cannot directly change subscription plan (plan changes via admin or billing)
  - All profile updates logged to audit_logs table

---

### FR2: Subscription Plans & Billing

#### FR2.1 Subscription Plan Management
- **Req**: Platform operators can create and manage subscription plans
- **Acceptance Criteria**:
  - Admin can create plan with: name, display_name, price (monthly/yearly), feature flags, resource limits
  - Plan defines: max_projects, max_concurrent_sessions_per_project, max_system_prompts, max_mcp_servers, max_documents, rag_enabled, voice_enabled, webhooks_enabled, etc.
  - Plans can be made public (visible to all users) or hidden (enterprise only)
  - Plans can be activated/deactivated
  - Pricing displayed in dashboard for public plans
  - Admin can update plan details (but changes apply to new subscriptions, not retroactively)

#### FR2.2 User Subscription Assignment
- **Req**: Platform users are assigned to a subscription plan
- **Acceptance Criteria**:
  - Free tier assigned by default on registration
  - Users can manually upgrade via dashboard (links to billing page — placeholder for v1.0)
  - Admin can manually assign plans to users
  - Current plan visible in user profile and dashboard
  - Plan limits enforced throughout the platform (via PlanValidator)

#### FR2.3 Plan Limit Enforcement
- **Req**: User actions are bounded by their subscription plan
- **Acceptance Criteria**:
  - Creating project when max_projects reached: rejected with 429 + error message
  - Issuing ephemeral token when session limit reached: rejected with 429 + error message
  - Uploading document when max_documents reached: rejected with 429 + error message
  - Registering MCP server when max_mcp_servers reached: rejected with 429 + error message
  - Attempting to enable RAG when rag_enabled = false for plan: rejected with 403
  - All rejections include current usage and plan limits in response

#### FR2.4 Usage Tracking (Billing Foundation)
- **Req**: Platform tracks usage for future billing integration
- **Acceptance Criteria**:
  - Each session records: input_tokens, output_tokens, audio_in_secs, audio_out_secs
  - Sessions table stores estimated_cost_usd calculated from model pricing
  - Usage records aggregated by project and user (daily, monthly)
  - Cost calculated as: (input_tokens * input_cost) + (output_tokens * output_cost) + (audio_secs * audio_cost)
  - Usage data queryable via GET /projects/:id/usage and GET /users/:id/usage
  - Dashboard shows usage breakdown, cost estimates, and current billing period usage

---

### FR3: Project Management

#### FR3.1 Project Creation
- **Req**: Users can create new projects to configure AI applications
- **Acceptance Criteria**:
  - User submits: project_name, optional description
  - System checks user's project count against plan limit
  - System creates project record with: id (UUID), user_id, name, description, is_active=true, created_at
  - System automatically creates project_configs record with default values (no LLM config yet)
  - System returns project_id to user
  - Project is immediately ready for configuration

#### FR3.2 Project Listing & Details
- **Req**: Users can list and view details of their projects
- **Acceptance Criteria**:
  - GET /projects returns paginated list of user's projects (name, description, is_active, created_at, model selected, provider selected)
  - GET /projects/:id returns full project detail including config, active system prompt, document count, session count
  - Only projects owned by the authenticated user are returned (IDOR protection)
  - Pagination: default 20 per page, max 100

#### FR3.3 Project Update
- **Req**: Users can update project metadata
- **Acceptance Criteria**:
  - PATCH /projects/:id allows updating: name, description
  - Does NOT allow direct mutation of configuration (use dedicated config endpoints)
  - Updates logged to audit_logs table
  - Returns updated project object

#### FR3.4 Project Deactivation
- **Req**: Users can deactivate projects to stop creating new sessions
- **Acceptance Criteria**:
  - PATCH /projects/:id/deactivate sets is_active = false
  - When project is inactive: session creation requests are rejected with HTTP 400 + "project_inactive" error
  - Active sessions continue running until natural completion or timeout
  - Admin can force-close all sessions for an inactive project via admin API
  - Deactivated projects can be reactivated by user

#### FR3.5 Project Deletion
- **Req**: Users can permanently delete projects
- **Acceptance Criteria**:
  - DELETE /projects/:id hard-deletes the project record
  - Cascade behavior: all child records deleted (configs, prompts, keys, documents, sessions, mcp_servers, tools)
  - RAG cleanup: all document blobs deleted from Azure Blob Storage, all vectors deleted from Azure AI Search (via async job)
  - Active sessions are force-closed (4004 close code, project_inactive error)
  - Audit log records the deletion
  - Deletion is permanent and cannot be undone

---

### FR4: LLM Configuration per Project

#### FR4.1 Select Provider, API Type, and Model
- **Req**: Users configure which LLM provider and API the project will use
- **Acceptance Criteria**:
  - PATCH /projects/:id/config accepts: provider_api_id, model_id
  - provider_api_id references a row in llm_provider_apis (e.g., "openai_realtime" or "gemini_live")
  - model_id references a row in llm_models (e.g., "gpt-4-realtime-preview", "gemini-2.5-flash")
  - System validates the model belongs to the selected API type
  - System validates the API type is not deprecated
  - All valid options come from the provider catalog (users cannot enter arbitrary values)
  - Response includes supported transports (websocket, sse), capabilities (audio, tool_calling, vad, etc.)

#### FR4.2 Model Parameters
- **Req**: Users configure per-model parameters like temperature, max_output_tokens
- **Acceptance Criteria**:
  - PATCH /projects/:id/config accepts: temperature (0.0–2.0), max_output_tokens (1–4096)
  - System validates values are within model-specific constraints
  - Invalid values rejected with 400 + validation error
  - Parameters stored in project_configs table
  - These parameters injected into every LLM request made by adapters

#### FR4.3 Voice & Audio Settings
- **Req**: Users configure voice settings if the project uses a voice API
- **Acceptance Criteria**:
  - PATCH /projects/:id/config accepts: voice_id (string), vad_enabled (bool), transcription_enabled (bool)
  - voice_id must be valid for the selected provider (OpenAI voice list, Google voice list)
  - vad_enabled only settable if API supports VAD (supports_vad = true in provider catalog)
  - transcription_enabled only settable if API supports transcription (supports_transcription = true)
  - Settings stored in project_configs
  - These settings passed to adapter at session initialization

#### FR4.4 Conversation History Limit
- **Req**: Users control how many conversation turns are kept in Redis history
- **Acceptance Criteria**:
  - PATCH /projects/:id/config accepts: history_turn_limit (1–50, default 10)
  - This only applies to HTTP Orchestrator adapters (Chat Completions, Gemini Generate)
  - WS Bridge adapters (Realtime, Gemini Live) ignore this (provider holds state)
  - Stored in project_configs
  - Protocol Handler enforces: when turn_count > history_turn_limit, trim oldest pair

#### FR4.5 Inactivity Timeout
- **Req**: Users control session timeout
- **Acceptance Criteria**:
  - PATCH /projects/:id/config accepts: inactivity_timeout_seconds (30–3600, default 120)
  - If no messages in either direction for this duration: gateway closes session
  - Session Gateway sends {"type":"error","code":"session_timeout"} before closing
  - Timeout is per-project, not global

#### FR4.6 Transport Selection (Conditional)
- **Req**: System shows which transports are available based on API type
- **Acceptance Criteria**:
  - WebSocket always available for all API types
  - SSE only available if llm_provider_apis.supports_sse = true
  - Dashboard displays badge: "✅ WebSocket • ✅ SSE" or "✅ WebSocket • ❌ SSE (audio required)"
  - Integration guide shows connection examples for both transports where applicable
  - Session Gateway rejects SSE connection attempts on non-sse-supporting APIs with HTTP 400

---

### FR5: System Prompt Management

#### FR5.1 Create System Prompt
- **Req**: Users can create system prompts to define agent behavior
- **Acceptance Criteria**:
  - POST /projects/:id/prompts accepts: label, content
  - System checks project hasn't exceeded max_system_prompts_per_project (from plan)
  - label must be unique within the project (case-sensitive)
  - content is the actual prompt text (immutable after creation)
  - System creates system_prompts record with is_active=false, is_archived=false
  - Returns created prompt object with id, label, content, is_active, created_at

#### FR5.2 List Prompts
- **Req**: Users can see all system prompts for a project
- **Acceptance Criteria**:
  - GET /projects/:id/prompts returns list of all non-archived prompts
  - Returns: id, label, content, is_active, created_at
  - Active prompt marked clearly
  - Pagination: default 20 per page

#### FR5.3 Activate Prompt
- **Req**: Users can set a prompt as active
- **Acceptance Criteria**:
  - PATCH /projects/:id/prompts/:promptId/activate atomically:
    - Sets target prompt is_active = true
    - Sets all other prompts for this project is_active = false
    - Database-level uniqueness enforced by partial index (impossible for two to be active)
  - Response includes the newly active prompt
  - Next session opened after this call loads the new prompt
  - Cannot activate an archived prompt (error)

#### FR5.4 Archive Prompt
- **Req**: Users can archive prompts (soft delete)
- **Acceptance Criteria**:
  - PATCH /projects/:id/prompts/:promptId/archive sets is_archived = true
  - Cannot archive the currently active prompt (must activate another first)
  - Archived prompts do not appear in list view unless explicitly requested
  - Active sessions that loaded this prompt continue using it (stored in session record)
  - Archived prompts cannot be un-archived (create new version instead)

#### FR5.5 Prompt Immutability
- **Req**: System prompt text cannot be edited after creation
- **Acceptance Criteria**:
  - No PATCH endpoint exists for prompt content
  - Attempting to update content returns 403 "Prompts are immutable"
  - To change prompt: user creates new prompt, activates it
  - Old prompts remain archived for historical reference
  - This ensures A/B testing and audit trail integrity

#### FR5.6 Session Records Capture Active Prompt
- **Req**: For future A/B testing, each session records which prompt was used
- **Acceptance Criteria**:
  - sessions table includes system_prompt_id FK
  - At session initialization, Protocol Handler captures active prompt ID
  - Session record persisted to PostgreSQL includes the system_prompt_id
  - This enables future analytics: "sessions with prompt_a vs prompt_b"

---

### FR6: API Key Management

#### FR6.1 API Key Creation
- **Req**: Users can create long-lived API keys for server-side authentication
- **Acceptance Criteria**:
  - POST /projects/:id/keys accepts: optional label, optional expires_at
  - System generates cryptographically random key (256 bits entropy)
  - System computes HMAC-SHA256 hash using server secret
  - System stores: key_hash (unique), key_prefix (first 8 chars), label, project_id, expires_at, created_at, last_used_at=null
  - Raw key returned in response exactly once (marked "never shown again")
  - Raw key NOT stored in database
  - Response: {"key": "sk_live_...", "key_hash": "***...***", "created_at": "..."}

#### FR6.2 API Key Listing
- **Req**: Users can see all their API keys for a project
- **Acceptance Criteria**:
  - GET /projects/:id/keys returns list of keys
  - Returns: key_prefix, label, is_active, last_used_at, created_at, expires_at
  - Does NOT return the raw key or full hash
  - Shows count of tokens issued with each key (since last reset)
  - Pagination: default 20 per page

#### FR6.3 API Key Revocation
- **Req**: Users can revoke API keys
- **Acceptance Criteria**:
  - DELETE /projects/:id/keys/:keyId sets is_active = false
  - Revocation takes effect within 30 seconds (Redis cache invalidation)
  - API calls using revoked key fail immediately with 401
  - Active sessions authenticated with that key are NOT retroactively closed (they use ephemeral tokens)
  - Future token issuance requests using revoked key are rejected
  - Revocation is immediate and cannot be undone

#### FR6.4 API Key Expiry
- **Req**: API keys can have optional expiration dates
- **Acceptance Criteria**:
  - When creating key, user can set expires_at (default: never expires)
  - At token issuance time, system checks if key is expired (expires_at < now())
  - Expired key: reject token issuance with 401 "Key expired"
  - Background job (hourly): mark expired keys as inactive
  - Expired keys cannot be "used" even if is_active = true

---

### FR7: Ephemeral Token Issuance (Automated)

#### FR7.1 Token Issuance Endpoint
- **Req**: Users' server-side backends can request ephemeral tokens
- **Acceptance Criteria**:
  - POST /projects/:projectId/tokens
  - Authorization: Bearer <long_lived_api_key>
  - Body: {"scope": "session:connect"}
  - Endpoint must be rate-limited by plan (e.g., "100 tokens per minute for Starter plan")
  - HTTP 202 response on success
  - Return: {"token": "raw_token_here", "expires_in": 60}

#### FR7.2 Token Issuance Validation
- **Req**: System validates multiple conditions before issuing token
- **Acceptance Criteria**:
  - Hash inbound API key → look up in Redis (fast path) → PostgreSQL (fallback)
  - Validate: key is active, not expired, belongs to the project_id in URL
  - Validate: project is active (is_active = true)
  - Validate: user's subscription plan allows ephemeral tokens (ephemeral_tokens_enabled = true in plan)
  - Validate: project hasn't exceeded the token issuance rate limit for this minute
  - Validate: project hasn't exceeded the concurrent session limit (reject token if already at limit)
  - If any validation fails: return HTTP 429 or 401 (no token issued)

#### FR7.3 Token Storage & Single-Use Enforcement
- **Req**: Tokens are single-use and cannot be reused
- **Acceptance Criteria**:
  - Raw token is generated (256 bits entropy)
  - Token hash (HMAC-SHA256) computed and stored in Redis with TTL=60s
  - Redis entry: {"project_id": "...", "scope": "session:connect", "issued_to_ip": "...", "expires_at": "..."}
  - At Session Gateway connection, atomic Redis.GETDEL(token_hash) is called
  - Token either exists (delete and proceed) or doesn't (reject 401)
  - Second concurrent call with same token fails (already deleted)
  - Audit record inserted to ephemeral_tokens table (outcome=NULL initially)

#### FR7.4 Token Audit Trail
- **Req**: All token issuance events are logged
- **Acceptance Criteria**:
  - ephemeral_tokens table records: project_id, token_hash, scope, requesting_ip, created_at, expires_at, used_at, outcome
  - At issuance: INSERT with outcome=NULL
  - At connection: UPDATE used_at, outcome='used'
  - Background job (hourly): UPDATE outcome='expired' for tokens where outcome IS NULL and expires_at < now()
  - Unused tokens purged from table after 24 hours
  - Audit trail queryable via GET /projects/:id/tokens/audit

#### FR7.5 Token Scope Validation
- **Req**: Token scope is validated and enforced
- **Acceptance Criteria**:
  - Only valid scope in v1.0: "session:connect"
  - At token issuance: scope stored in Redis alongside token_hash
  - At connection: Session Gateway validates scope field from Redis
  - Mismatch returns 401 "Invalid token scope"
  - Future scopes (e.g., "admin:session", "session:readonly") can be added without changing token structure

---

### FR8: RAG (Retrieval-Augmented Generation)

#### FR8.1 Document Upload
- **Req**: Users can upload documents to a project for RAG
- **Acceptance Criteria**:
  - POST /projects/:id/documents multipart form upload
  - Supported formats: PDF, DOCX, TXT, Markdown
  - File size validated against plan limit (e.g., "max 25 MB per document for Pro plan")
  - Total project storage validated against plan limit (e.g., "max 5 GB for Pro plan")
  - RAG must be enabled on plan (rag_enabled = true)
  - System creates project_documents record: file_name, file_type, file_size_bytes, blob_url, index_status='pending'
  - File uploaded to Azure Blob Storage in project-specific container
  - System triggers Azure AI Search indexer run (or schedules it)
  - Returns document object with status='pending', id, upload_at

#### FR8.2 Document Listing
- **Req**: Users can see all uploaded documents for a project
- **Acceptance Criteria**:
  - GET /projects/:id/documents returns paginated list
  - Each document shows: id, file_name, file_type, file_size_bytes, index_status (pending/indexing/indexed/failed), chunk_count, uploaded_at, indexed_at
  - Soft-deleted documents (deleted_at IS NOT NULL) do not appear by default
  - Option to include deleted documents via ?include_deleted=true
  - Pagination: default 20 per page

#### FR8.3 Document Deletion
- **Req**: Users can delete documents from RAG
- **Acceptance Criteria**:
  - DELETE /projects/:id/documents/:docId soft-deletes (deleted_at = now())
  - Async job queued: delete blob from Azure Blob Storage
  - Async job queued: delete all vectors from Azure AI Search (filtered by document reference)
  - Audit logged
  - If no documents remain for project: set project_configs.rag_enabled = false
  - Returns 204 No Content on success

#### FR8.4 Automatic RAG Enablement
- **Req**: RAG is automatically enabled when first document is successfully indexed
- **Acceptance Criteria**:
  - When project_documents.index_status transitions to 'indexed' for the first time
  - System automatically sets project_configs.rag_enabled = true
  - If a user manually disables RAG: all documents remain in DB, but adapter doesn't inject search tool
  - User can re-enable RAG later without re-uploading documents

#### FR8.5 RAG Configuration
- **Req**: Users can configure RAG behavior at project level
- **Acceptance Criteria**:
  - PATCH /projects/:id/config accepts: rag_timeout_ms (default 5000), rag_top_k (default 3)
  - rag_timeout_ms: max milliseconds to wait for Azure AI Search to respond before returning empty result
  - rag_top_k: number of document chunks to retrieve per query
  - Stored in project_configs
  - Session Gateway Protocol Handler uses these values when intercepting search_internal_knowledge tool calls

#### FR8.6 Project Deletion Cascades to RAG
- **Req**: Deleting a project cleans up all RAG data
- **Acceptance Criteria**:
  - When DELETE /projects/:id is called
  - All project_documents rows soft-deleted (deleted_at = now())
  - Async jobs queue for: delete all blobs in project prefix, delete all vectors from AI Search
  - Cleanup jobs retried on failure, logged, tracked
  - Cannot proceed with project deletion until cleanup jobs complete (or all fail with logging)

---

### FR9: MCP (Model Context Protocol) Server Integration

#### FR9.1 Register MCP Server
- **Req**: Users can register external MCP servers to expose tools
- **Acceptance Criteria**:
  - POST /projects/:id/mcp-servers accepts: name, description, base_url, auth_type, auth_secret_ref
  - base_url validation: must start with https://, cannot be RFC1918 (10.0.0.0/8, 172.16.0.0/12, 192.168.0.0/16), cannot be loopback, cannot be metadata endpoint
  - auth_type: 'none' | 'api_key' | 'bearer'
  - If auth_type != 'none': auth_secret_ref must reference an Azure Key Vault secret name
  - System creates mcp_servers record: project_id, name, base_url, auth_type, auth_secret_ref, connection_status='pending'
  - Returns mcp_servers object with id

#### FR9.2 MCP Tool Discovery
- **Req**: System discovers tools exposed by MCP servers
- **Acceptance Criteria**:
  - Automatic: background job every 10 minutes per active MCP server
  - Manual: POST /projects/:id/mcp-servers/:serverId/rediscover (rate-limited: once per 30 sec)
  - Discovery protocol: send initialize → check capabilities → tools/list → receive tool definitions
  - System validates each tool schema (reject malformed tools, log)
  - Tool names are namespaced: mcp_{server_id}__{original_tool_name}
  - New tools: INSERT into tools table with source='mcp', mcp_server_id FK
  - Removed tools: UPDATE is_deprecated=true (never hard delete)
  - Changed schemas: UPDATE tools row
  - Tool list cached in Redis: project:{project_id}:mcp:{server_id}:tools (TTL 10 min)
  - Emit {"type":"rag_search",...} equivalent event to End Client (future: "mcp_discovery" event)

#### FR9.3 MCP Server Listing
- **Req**: Users can see all registered MCP servers and their tools
- **Acceptance Criteria**:
  - GET /projects/:id/mcp-servers returns list of servers
  - Each shows: id, name, base_url, connection_status, tool_count, last_discovered_at, last_error
  - GET /projects/:id/mcp-servers/:serverId returns detail including discovered tools
  - Tools show: name (namespaced), description, input_schema, output_schema, timeout_ms

#### FR9.4 MCP Server Deactivation
- **Req**: Users can temporarily disable an MCP server
- **Acceptance Criteria**:
  - PATCH /projects/:id/mcp-servers/:serverId with is_active=false
  - Session Gateway stops discovering tools from this server
  - Existing discovered tools remain in database but is_active=false
  - New sessions don't inject these tools
  - Old sessions continue using them (reference is by name, not by active status)

#### FR9.5 MCP Server Deletion
- **Req**: Users can permanently delete an MCP server registration
- **Acceptance Criteria**:
  - DELETE /projects/:id/mcp-servers/:serverId
  - Cascade: all tools with this mcp_server_id are hard-deleted
  - If tools are referenced in active sessions: sessions continue (reference by name still works)
  - Returns 204 No Content

#### FR9.6 SSRF Prevention for MCP Endpoints
- **Req**: MCP server URLs and tool endpoint URLs are validated to prevent SSRF
- **Acceptance Criteria**:
  - At registration: validate base_url and all tool_endpoints have URL validation
  - Validation rules: HTTPS only, no RFC1918, no loopback, no metadata endpoints
  - Invalid URLs rejected with 400 + clear error message
  - Validation runs at registration AND at every tool endpoint creation
  - List of blocked IP ranges: 10.0.0.0/8, 172.16.0.0/12, 192.168.0.0/16, 127.0.0.0/8, ::1, 169.254.169.254

---

### FR10: Session Management

#### FR10.1 Session Initiation (WebSocket)
- **Req**: End Clients can open WebSocket connections to the Session Gateway
- **Acceptance Criteria**:
  - Client sends: wss://host/ws?project_id={uuid}&token={ephemeral_token}
  - Session Gateway extracts token, project_id
  - Session Gateway validates token (atomic Redis.GETDEL)
  - Session Gateway validates transport capability (supports_websocket = true)
  - Session Gateway loads project config from Redis or PostgreSQL
  - Session Gateway loads active system prompt
  - Session Gateway retrieves LLM provider credentials from Key Vault
  - Session Gateway instantiates adapter and connects to LLM provider
  - On success: sends {"type":"session_started","`session_id`":"<`uuid`>"}
  - On failure: returns HTTP 401 with {"error":"code","message":"..."}

#### FR10.2 Session Initiation (SSE)
- **Req**: End Clients can open SSE connections for text-only APIs
- **Acceptance Criteria**:
  - Client sends: GET https://host/sse?project_id={uuid}, Header: Authorization: Bearer {token}
  - Session Gateway validates token (same as WebSocket)
  - Session Gateway validates transport capability (supports_sse = true)
  - On failure: HTTP 400 with {"error":"sse_not_supported","message":"..."}
  - On success: HTTP 200, Content-Type: text/event-stream
  - Returns X-Session-Id header (client uses this for subsequent POST /sse/messages)
  - First SSE event: data: {"type":"session_started","session_id":"<`uuid`>"}
  - Server sends ": heartbeat" every 15 seconds to keep connection alive

#### FR10.3 Session Registry
- **Req**: Platform tracks all active sessions
- **Acceptance Criteria**:
  - Each session stored in Redis with TTL = inactivity_timeout + buffer
  - Redis key: session:{session_id}
  - Redis value: {project_id, user_id, adapter_type, model_id, node_id, started_at, transport, system_prompt_id}
  - Session also recorded in PostgreSQL sessions table (at teardown for historical records)
  - Active sessions queryable: GET /projects/:id/sessions (shows current live sessions)

#### FR10.4 Session State Machine
- **Req**: Sessions progress through defined state transitions
- **Acceptance Criteria**:
  - States: VALIDATING → INITIALIZING → ACTIVE ↔ RESPONDING → CLOSING → CLOSED
  - Also: REJECTED (auth failure), FAILED (provider connection failure)
  - Transitions enforced by state machine logic
  - Invalid transitions return error
  - State persisted in Redis atomically

#### FR10.5 Active Session Monitoring
- **Req**: Users can see real-time statistics of active sessions
- **Acceptance Criteria**:
  - GET /projects/:id/sessions returns paginated list of currently active sessions
  - Each session shows: session_id, start_time, adapter_type, model, transport, system_prompt label, live token count
  - Refreshes every 5 seconds (client polls or uses WebSocket)

#### FR10.6 Historical Session Querying
- **Req**: Users can view past session records with filtering
- **Acceptance Criteria**:
  - GET /projects/:id/sessions/history with query params: date_from, date_to, status (success/timeout/error), limit, offset
  - Returns: session_id, started_at, ended_at, duration, status, final_token_count, estimated_cost, system_prompt label
  - Pagination: default 20 per page, max 100
  - Cannot see sessions from other projects (IDOR protection)

#### FR10.7 Session Detail Drill-Down
- **Req**: Users can view detailed breakdown of a specific session
- **Acceptance Criteria**:
  - GET /sessions/:id returns full session detail
  - Includes: session_id, project_id, user_id, model_id, adapter_type, transport, system_prompt_id, input_tokens, output_tokens, audio_in_secs, audio_out_secs, estimated_cost_usd, started_at, ended_at, close_reason
  - Includes usage breakdown chart data (tokens per turn, latency per turn — if available in logs)
  - Only accessible by user who owns the project (IDOR protection)

#### FR10.8 Session Inactivity Timeout
- **Req**: Sessions with no activity are closed automatically
- **Acceptance Criteria**:
  - If no messages in either direction for inactivity_timeout_seconds (project config, default 120)
  - Session Gateway sends error event: {"type":"error","code":"session_timeout","message":"...","retryable":false}
  - WebSocket close code 4005 sent
  - Session moved to CLOSED state
  - Session record persisted to PostgreSQL with close_reason="inactivity_timeout"

#### FR10.9 Admin Force-Close
- **Req**: Admins can forcibly close any active session
- **Acceptance Criteria**:
  - DELETE /admin/sessions/{sessionId}
  - System sends command to owning Session Gateway node via Redis pub/sub
  - Node receives command and closes session within 5 seconds
  - End Client receives error event: {"type":"error","code":"admin_force_close","message":"...","retryable":false}
  - Session record persisted with close_reason="admin_force_close"

---

### FR11: Usage Tracking & Reporting

#### FR11.1 Session Usage Accumulation
- **Req**: Platform tracks token and audio usage per session
- **Acceptance Criteria**:
  - Each adapter emits usage data in TextDoneEvent and AudioDoneEvent
  - Session Gateway accumulates: input_tokens, output_tokens, audio_in_secs, audio_out_secs
  - Counters maintained in memory during session
  - Flushed to Redis every 30 seconds for durability
  - Final flush at session teardown before PostgreSQL persistence

#### FR11.2 Cost Estimation
- **Req**: Platform calculates estimated USD cost per session
- **Acceptance Criteria**:
  - llm_models table includes: input_token_cost_usd, output_token_cost_usd, audio_in_cost_usd, audio_out_cost_usd
  - Cost formula: (input_tokens * input_cost) + (output_tokens * output_cost) + (audio_in_secs * audio_in_cost) + (audio_out_secs * audio_out_cost)
  - Calculated at session teardown
  - Stored in sessions.estimated_cost_usd
  - Rounded to 6 decimal places

#### FR11.3 Project-Level Usage Report
- **Req**: Users can query usage aggregated by project
- **Acceptance Criteria**:
  - GET /projects/:id/usage?date_from=YYYY-MM-DD&date_to=YYYY-MM-DD
  - Returns: total_sessions, total_input_tokens, total_output_tokens, total_audio_in_secs, total_audio_out_secs, estimated_total_cost_usd
  - Includes breakdown by date (daily granularity)
  - Includes breakdown by provider (if multi-provider is ever used)
  - Includes breakdown by model
  - Date range default: last 30 days
  - Cached in Redis for 1 hour

#### FR11.4 Account-Level Usage Report
- **Req**: Users can see aggregated usage across all projects
- **Acceptance Criteria**:
  - GET /users/:id/usage?date_from=...&date_to=...
  - Returns same structure as project-level, but summed across all user's projects
  - Includes per-project breakdown
  - Only accessible by the user themselves or admin (IDOR protection)

#### FR11.5 Usage Dashboard Visualization
- **Req**: Dashboard displays usage analytics
- **Acceptance Criteria**:
  - Time-series charts: tokens/day, audio-seconds/day, estimated-cost/day
  - Date range picker (default: last 30 days, options: 7d, 30d, 90d, custom)
  - Current billing period usage vs plan limit (if applicable)
  - Cost forecast based on current usage trends
  - Model usage breakdown (pie chart)
  - API type usage breakdown (pie chart)

---

### FR12: Provider Catalog Management

#### FR12.1 Provider Records
- **Req**: Platform maintains catalog of supported LLM providers
- **Acceptance Criteria**:
  - llm_providers table with: id, name (unique), display_name, is_active
  - Providers: OpenAI, Google (v1.0), extensible for Anthropic, Mistral, etc. (v1.1+)
  - Admins can add providers: POST /admin/providers
  - Admins can update display_name, is_active
  - Deactivated providers hidden from user UI

#### FR12.2 API Type Records
- **Req**: Platform maintains catalog of API types per provider
- **Acceptance Criteria**:
  - llm_provider_apis table with: id, provider_id (FK), name, display_name, adapter_key, supports_websocket, supports_sse, supports_text_input, supports_text_output, supports_audio_input, supports_audio_output, supports_tool_calling, supports_vad, supports_transcription, is_stateful, is_active
  - API types: OpenAI Realtime API, Chat Completions, Responses API, Gemini Live API, Gemini Generate Content (v1.0)
  - adapter_key maps directly to ILLMSessionAdapter implementation class name
  - Capability flags determine what features are available to the user
  - Admins can create/update API types

#### FR12.3 Model Records
- **Req**: Platform maintains catalog of models per API type
- **Acceptance Criteria**:
  - llm_models table with: id, provider_api_id (FK), name, display_name, input_token_cost_usd, output_token_cost_usd, audio_in_cost_usd, audio_out_cost_usd, context_window_tokens, is_deprecated, is_active
  - Models: gpt-4o, gpt-4-realtime-preview, gemini-2.5-flash, etc.
  - Deprecated models hidden from new project configs (but existing projects can keep them)
  - Pricing fields used for cost estimation
  - Admins can create/update models
  - Context window informs max_output_tokens validation

#### FR12.4 Catalog API Endpoints
- **Req**: Users can query the catalog
- **Acceptance Criteria**:
  - GET /catalog/providers returns list of active providers with display names
  - GET /catalog/apis?provider_id=uuid returns API types for a provider with capability flags
  - GET /catalog/models?api_id=uuid returns models for an API type with pricing
  - All endpoints return is_active=true records only (users don't see deprecated/inactive)
  - No authentication required (catalog is public)

---

### FR13: User Administration

#### FR13.1 User Listing (Admin)
- **Req**: Platform operators can search and list user accounts
- **Acceptance Criteria**:
  - GET /admin/users with query params: email, created_after, created_before, is_verified, subscription_plan_id, limit, offset
  - Returns: id, email, created_at, is_verified, subscription_plan (name), account_status (active/deactivated), last_login_at
  - Pagination: default 20 per page, max 100
  - Admin role required

#### FR13.2 User Detail (Admin)
- **Req**: Operators can view detailed user information
- **Acceptance Criteria**:
  - GET /admin/users/:id returns: id, email, created_at, is_verified, subscription_plan, is_active, project_count, last_login_at, auth_method
  - Shows user's projects and session count
  - Shows user's usage this billing period
  - Admin role required

#### FR13.3 User Deactivation (Admin)
- **Req**: Operators can deactivate user accounts
- **Acceptance Criteria**:
  - PATCH /admin/users/:id with is_active=false
  - Deactivated users cannot login
  - All active sessions for that user are force-closed (close code 4007 "user_deactivated")
  - All refresh tokens are invalidated
  - API keys become inoperative
  - User can be reactivated by setting is_active=true
  - Audit logged

#### FR13.4 User Reactivation (Admin)
- **Req**: Operators can reactivate deactivated accounts
- **Acceptance Criteria**:
  - PATCH /admin/users/:id with is_active=true
  - User can immediately login again
  - Existing API keys and projects become usable again
  - No data loss

#### FR13.5 Subscription Plan Assignment (Admin)
- **Req**: Operators can change a user's subscription plan
- **Acceptance Criteria**:
  - PATCH /admin/users/:id with subscription_plan_id=uuid
  - Plan change effective immediately
  - If user now exceeds plan limits (e.g., 15 projects but plan allows 5): warning to operator, but change allowed
  - Audit logged with old and new plan

---

### FR14: Audit Logging

#### FR14.1 Audit Log Recording
- **Req**: All sensitive actions are logged
- **Acceptance Criteria**:
  - audit_logs table records: id, user_id, action (enum: user_created, login, token_issued, key_created, key_revoked, project_created, project_deleted, config_updated, prompt_created, rag_document_uploaded, mcp_server_registered, user_deactivated, etc.), resource_type, resource_id, old_values (JSON), new_values (JSON), requesting_ip, user_agent, timestamp
  - All management-plane operations logged to audit_logs
  - Data-plane operations (sessions, messages) not logged to avoid explosion of records
  - Audit log is append-only (immutable)
  - Soft deletions record the deletion in audit log

#### FR14.2 Audit Log Querying (Admin)
- **Req**: Operators can query audit logs
- **Acceptance Criteria**:
  - GET /admin/audit-logs with filters: user_id, action, resource_type, date_from, date_to, requesting_ip, limit, offset
  - Returns paginated list with all fields
  - Default order: descending by timestamp (newest first)
  - Admin role required
  - Queryable for at least 90 days of history

---

### FR15: Email Services

#### FR15.1 Email Verification on Registration
- **Req**: Users must verify their email address
- **Acceptance Criteria**:
  - On registration, system generates verification token (HMAC-SHA256, 24-hour expiry)
  - Verification link sent to user's email with token in URL
  - User clicks link → system verifies token, marks user as verified
  - User cannot login until verified
  - After 24 hours, verification link expires and user must request new one
  - Can resend verification email: POST /auth/resend-verification-email (rate-limited: 3x per hour)

#### FR15.2 Password Reset Email
- **Req**: Users can reset forgotten passwords
- **Acceptance Criteria**:
  - POST /auth/forgot-password accepts email (no error if not found)
  - If email exists: generate reset token (1-hour expiry), send reset link
  - If email not found: silently succeed (return same message) — prevent email enumeration
  - Reset link in email, user clicks → redirected to password reset form
  - User enters new password, system verifies token, updates password hash
  - All existing refresh tokens invalidated
  - User can login with new password immediately

#### FR15.3 Email Service Integration
- **Req**: Emails are sent via external service
- **Acceptance Criteria**:
  - Email provider: Azure Communication Service or SMTP (configurable)
  - Email templates for: verification, password reset, session notifications (future), billing alerts (future)
  - All emails include unsubscribe link (future compliance feature)
  - Email sending errors logged but do not block registration/reset (async retry)
  - Emails sent with From address: noreply@chatn.example.com

---

### FR16: Admin Dashboard

#### FR16.1 Admin System Overview
- **Req**: Admins see platform-wide statistics
- **Acceptance Criteria**:
  - Dashboard shows: total active users, total projects, active sessions (live count), total sessions (all-time), failed sessions (24h)
  - Shows provider status (green/red for each provider)
  - Shows error rate (% of sessions that failed, 24h, 7d)
  - Shows link to centralized logs (Application Insights, Datadog, etc.)
  - Refreshes every minute

#### FR16.2 Plan Management Interface
- **Req**: Admins can manage subscription plans
- **Acceptance Criteria**:
  - GET /admin/plans returns all plans (active + inactive)
  - POST /admin/plans creates new plan with all fields
  - PATCH /admin/plans/:id updates plan definition
  - PATCH /admin/plans/:id?activate=true activates plan
  - PATCH /admin/plans/:id?deactivate=true deactivates plan
  - Plan changes apply to new subscriptions immediately
  - Existing subscriptions on old plan can be migrated via user migration job (future)

#### FR16.3 Catalog Management Interface
- **Req**: Admins can manage providers, APIs, and models
- **Acceptance Criteria**:
  - Full CRUD for providers: POST, GET, PATCH (name, display_name, is_active)
  - Full CRUD for APIs: POST, GET, PATCH (all capability flags, transport flags)
  - Full CRUD for models: POST, GET, PATCH (all pricing fields, deprecation status)
  - Changes take effect immediately (no need to redeploy gateway)
  - Deprecating a model doesn't break existing projects (they can keep using it)

---

### FR17: Connection Gateway (Session Gateway Service)

#### FR17.1 WebSocket Connection Handling
- **Req**: End Clients connect via WebSocket
- **Acceptance Criteria**:
  - Already documented in Session Gateway diagram
  - Connection flows through Connection Manager → Transport Handler (WebSocketTransport) → Protocol Handler → Adapters

#### FR17.2 SSE Connection Handling
- **Req**: End Clients connect via SSE for text-only APIs
- **Acceptance Criteria**:
  - Already documented in Session Gateway diagram
  - GET /sse initiates stream
  - POST /sse/messages sends client messages
  - Routing via Redis pub/sub for multi-node setup

#### FR17.3 Message Routing to Adapters
- **Req**: Protocol Handler routes messages correctly
- **Acceptance Criteria**:
  - Already documented: dual async loops, tool interception, etc.

#### FR17.4 Tool Call Interception
- **Req**: Gateway intercepts RAG and MCP tool calls
- **Acceptance Criteria**:
  - Already documented: search_internal_knowledge intercepted and sent to Azure AI Search
  - MCP-prefixed tools intercepted and sent to MCP servers
  - Results returned to LLM without End Client seeing them

#### FR17.5 Conversation History Management
- **Req**: Already documented in FR17
- **Acceptance Criteria**:
  - HTTP Orchestrator adapters: history in Redis (assembled turns only, not deltas)
  - WS Bridge adapters: no history in Redis (provider holds state)
  - Responses API: only previous_response_id in Redis
  - History trimmed when turn_count exceeds limit

---

## NON-FUNCTIONAL REQUIREMENTS

### NFR1: Performance

#### NFR1.1 API Response Time
- **Requirement**: REST API endpoints respond within specified latencies
- **Specification**:
  - Read operations (GET): < 200ms (p99)
  - Write operations (POST, PATCH): < 500ms (p99), excluding file uploads
  - File upload (RAG documents): < 5 seconds (p99)
  - Catalog queries: < 100ms (p99), served from cache
  - Audit log queries: < 500ms (p99)
  - These latencies measured from load balancer to response

#### NFR1.2 Gateway Message Latency
- **Requirement**: WebSocket and SSE messages are delivered with minimal latency
- **Specification**:
  - Text delta events: < 100ms from adapter to End Client (p99)
  - Audio delta events: < 50ms from adapter to End Client (p99)
  - Tool call events: < 200ms from adapter to End Client (p99)
  - RAG search intercept: < 500ms (including Azure AI Search query + network)
  - Measured from moment event is produced by adapter to moment it reaches End Client

#### NFR1.3 Token Issuance Latency
- **Requirement**: Ephemeral tokens are issued quickly
- **Specification**:
  - Token issuance endpoint: < 100ms (p99)
  - Includes: API key lookup, plan validation, token generation, Redis write

#### NFR1.4 Session Initialization Latency
- **Requirement**: New sessions establish quickly
- **Specification**:
  - Session initialization (token validation to LLM connection): < 2 seconds (p99)
  - Includes: credential retrieval from Key Vault (with 5-min cache), config load, adapter instantiation, LLM connection
  - RAG-enabled projects with Azure AI Search: may take up to 4 seconds

#### NFR1.5 Throughput
- **Requirement**: Platform handles multiple concurrent requests
- **Specification**:
  - REST API: minimum 1000 requests/second sustained (horizontal scaling)
  - Session Gateway: minimum 500 concurrent WebSocket connections per node (add nodes to scale)
  - Redis: minimum 10,000 operations/second sustained
  - Database: minimum 100 concurrent connections, 500 queries/second sustained

---

### NFR2: Scalability

#### NFR2.1 Horizontal Scalability
- **Requirement**: All services scale horizontally
- **Specification**:
  - REST API Service: stateless, can add/remove instances freely
  - Session Gateway Service: stateless (state in Redis), can add/remove nodes freely
  - Redis: single instance for v1.0 (cluster in v1.1+)
  - PostgreSQL: single instance for v1.0 (read replicas + primary in v1.1+)
  - No sticky session requirements except for SSE GET connections (handled by load balancer)

#### NFR2.2 Connection Scaling
- **Requirement**: Session Gateway can handle growing connection count
- **Specification**:
  - Each gateway node handles 500 concurrent WebSocket connections (tunable)
  - Add additional nodes to handle more connections
  - Load balancer distributes new connections across nodes
  - Existing connections unaffected by node addition/removal (except on node failure)

#### NFR2.3 Data Scaling
- **Requirement**: Database and cache scale with data growth
- **Specification**:
  - PostgreSQL: minimum 500 GB storage for v1.0 growth (1 year)
  - Redis: minimum 100 GB for session data (ephemeral, grows with concurrent sessions)
  - Indexes on all foreign keys and frequently-queried columns
  - Archival strategy for old sessions/audit logs (move to cold storage after 1 year)

---

### NFR3: Reliability & Availability

#### NFR3.1 Uptime Target
- **Requirement**: Platform is highly available
- **Specification**:
  - Target: 99.9% uptime (9 hours downtime per year) for managed services
  - Excludes planned maintenance windows (announced 7 days in advance)
  - Measured per calendar month

#### NFR3.2 Session Continuity on Transient Failure
- **Requirement**: Sessions survive temporary infrastructure hiccups
- **Specification**:
  - If a gateway node crashes: End Clients must reconnect (new session, new token required) — this is acceptable for v1.0
  - Conversation history in Redis survives node crash (stored separately from node process)
  - Usage counters: max 30 seconds of data loss (flushed every 30 sec)
  - Session records persisted to PostgreSQL at teardown (never lost)

#### NFR3.3 Database Failover
- **Requirement**: Database failures do not cause prolonged outages
- **Specification**:
  - PostgreSQL: automated backup (daily) + point-in-time recovery (7-day retention)
  - For v1.1: add read replicas and automatic failover for HA
  - Redis: RDB snapshots (optional for v1.0; recommended for v1.1+)
  - Recovery Time Objective (RTO): 15 minutes for database recovery
  - Recovery Point Objective (RPO): 1 hour for database (daily backup interval)

#### NFR3.4 Credential Caching & Failover
- **Requirement**: Key Vault unavailability does not immediately break all sessions
- **Specification**:
  - LLM provider credentials cached in-process for up to 5 minutes
  - If Key Vault is down: existing sessions continue, new sessions fail
  - Retry logic: 3 retries with exponential backoff before failing a session
  - Monitoring: alerts fire if Key Vault unavailable for > 5 minutes

#### NFR3.5 Graceful Shutdown
- **Requirement**: Services can be stopped gracefully
- **Specification**:
  - On SIGTERM, services enter "drain mode" (stop accepting new connections, finish existing ones)
  - Drain timeout: 30 seconds for gateway nodes, 10 seconds for API nodes
  - After timeout: force-close remaining connections with close code 1001 (going away)
  - No requests are lost during graceful drain

---

### NFR4: Security

#### NFR4.1 Transport Security (TLS)
- **Requirement**: All network traffic is encrypted
- **Specification**:
  - TLS 1.2 minimum for all connections (client-to-platform, platform-to-providers, platform-to-infrastructure)
  - TLS 1.3 strongly recommended for client-facing connections
  - Cipher suites: exclude RC4, MD5, export-grade encryption
  - Certificate: valid, not self-signed (ACME-issued for production)
  - Load balancer terminates TLS; backend communication over private network

#### NFR4.2 Authentication & Authorization
- **Requirement**: Only authenticated users can access their resources
- **Specification**:
  - All API endpoints (except registration, login, catalog) require valid JWT or API key + ephemeral token
  - JWT RS256 signature verified on every request
  - Refresh tokens validated against PostgreSQL hash
  - API key fast-path lookup in Redis (hash comparison)
  - Ephemeral token single-use enforcement (atomic GETDEL)
  - Authorization: IDOR checks at repository layer (every query includes user_id or project_id predicate)

#### NFR4.3 Credential Management
- **Requirement**: Secrets are never exposed
- **Specification**:
  - LLM provider API keys: stored exclusively in Azure Key Vault
  - MCP server auth credentials: stored exclusively in Azure Key Vault
  - JWT signing private key: stored exclusively in Azure Key Vault
  - API keys: stored as HMAC-SHA256 hashes, raw key shown once
  - Refresh tokens: stored as HMAC-SHA256 hashes
  - Ephemeral tokens: stored as HMAC-SHA256 hashes
  - Zero credentials in plaintext in code, config files, or logs

#### NFR4.4 SSRF Prevention
- **Requirement**: Users cannot force the platform to make requests to internal/private systems
- **Specification**:
  - MCP server URLs: validated at registration (HTTPS only, non-RFC1918, non-loopback, non-metadata)
  - Tool endpoint URLs: validated at creation (same rules)
  - Blocked IP ranges: 10.0.0.0/8, 172.16.0.0/12, 192.168.0.0/16, 127.0.0.0/8, ::1, 169.254.169.254
  - Validation runs at registration AND at every endpoint update
  - Invalid URLs rejected with 400 + error message

#### NFR4.5 IDOR Prevention
- **Requirement**: Users cannot access other users' data
- **Specification**:
  - Every database query enforces user_id or project_id scope predicate at repository layer
  - Queries that omit scope are considered bugs
  - No reliance on application-layer filtering
  - Test coverage: minimum 95% of repository queries include explicit scope check
  - Static analysis tools configured to flag queries missing scope

#### NFR4.6 Rate Limiting & Brute-Force Mitigation
- **Requirement**: Platform resists brute-force and DoS attacks
- **Specification**:
  - Login endpoint: max 10 attempts per IP per minute
  - Token issuance: max 100 tokens per minute per API key (plan-dependent)
  - Connection rate: max 10 new WebSocket/SSE connections per second per API key
  - API endpoints (authenticated): max 300 requests per minute per user
  - Unauthenticated endpoints (registration, login, catalog): max 10 requests per minute per IP
  - Rate limit exceeded: HTTP 429 with Retry-After header

#### NFR4.7 Password Security
- **Requirement**: Passwords are securely hashed
- **Specification**:
  - Hash algorithm: bcrypt with cost factor ≥ 12
  - Password minimum length: 12 characters
  - Password requirements: uppercase, lowercase, digit, special character
  - Passwords never logged or exposed
  - Password reset tokens: single-use, 1-hour expiry
  - Verification tokens: single-use, 24-hour expiry

#### NFR4.8 Audit Logging & Compliance
- **Requirement**: All sensitive actions are logged for audit
- **Specification**:
  - audit_logs table: immutable append-only record
  - Fields: user_id, action, resource_type, resource_id, old_values, new_values, requesting_ip, user_agent, timestamp
  - Minimum 90 days retention
  - Admin access to audit logs logged itself (recursive audit)
  - No way to modify or delete audit logs (must be retained for compliance)

#### NFR4.9 Token Lifecycle Security
- **Requirement**: Tokens are properly managed throughout their lifecycle
- **Specification**:
  - Ephemeral tokens: 60-second TTL (maximum 300 seconds configurable)
  - Ephemeral tokens: single-use enforcement (atomic Redis.GETDEL)
  - Ephemeral tokens: audit logged at issuance and consumption
  - JWT access tokens: 15-minute TTL (non-extendable)
  - Refresh tokens: 7-day TTL, rotated on every use
  - Refresh tokens: all tokens revoked on password change or logout
  - No token reuse across different user accounts

---

### NFR5: Data Privacy & Protection

#### NFR5.1 Conversation Content Never Persisted
- **Requirement**: User messages and model responses are never stored in PostgreSQL
- **Specification**:
  - Constraint C-04 enforcement: zero conversation content in primary database
  - History in Redis: ephemeral only, explicit deletion at session teardown
  - Sessions table: stores only metadata (tokens, duration, cost), never message text
  - tool_executions table: stores only metadata (status, latency, error code), never arguments or responses
  - Compliance: ready for GDPR "right to be forgotten" (no long-term conversation storage)

#### NFR5.2 Personal Data Handling
- **Requirement**: User personal data is handled securely
- **Specification**:
  - Email addresses: required for registration, never shared
  - Passwords: never stored in plaintext, never logged
  - IP addresses: logged for audit trail, retained for 90 days
  - User agent strings: logged for audit trail, retained for 90 days
  - No profiling, behavioral tracking, or third-party sharing of user data

#### NFR5.3 Data Encryption at Rest
- **Requirement**: Sensitive data is encrypted on storage devices
- **Specification**:
  - PostgreSQL: encrypted at rest (Azure managed keys or customer-managed)
  - Blob Storage: encrypted at rest (Azure managed keys or customer-managed)
  - Key Vault: Azure-managed, keys never leave the vault
  - Backups: encrypted with same keys as live data

#### NFR5.4 Data Isolation
- **Requirement**: Tenant data is logically isolated
- **Specification**:
  - RAG vector DB: mandatory project_id metadata filter on every search
  - No query without project_id filter can return RAG vectors
  - MCP tools: namespaced by server_id to prevent collisions
  - Sessions: keyed by session_id, isolated in Redis
  - No tenant can query another tenant's projects, keys, sessions, or documents

---

### NFR6: Maintainability & Operations

#### NFR6.1 Code Quality & Testing
- **Requirement**: Code is high-quality and well-tested
- **Specification**:
  - Test coverage: minimum 80% for all services
  - Unit tests: every business logic function has tests
  - Integration tests: every repository method and critical flow tested
  - End-to-end tests: major user workflows (registration, token issuance, session creation)
  - Code review: all changes reviewed by 2 senior developers before merge
  - Static analysis: SonarQube or equivalent, no new issues introduced

#### NFR6.2 Logging & Observability
- **Requirement**: Platform provides visibility into its operation
- **Specification**:
  - All services log to centralized store (Application Insights, DataDog, ELK stack)
  - Log levels: DEBUG, INFO, WARN, ERROR
  - Debug logging disabled in production (performance)
  - Sensitive data (passwords, keys, tokens) never logged
  - Structured logging (JSON format) for easy parsing
  - Key metrics logged: request latency, error rate, session count, resource usage

#### NFR6.3 Monitoring & Alerting
- **Requirement**: Operations team is alerted to problems
- **Specification**:
  - Metrics: CPU usage, memory, disk, network I/O, database connections
  - Metrics: API response times, error rates, session counts
  - Alerts: fire when p99 latency exceeds 1 second, error rate exceeds 1%, availability drops below 99.5%
  - Alerts: fire when any service is down, database is unavailable, Redis disconnects
  - Alert channels: PagerDuty, email, Slack (configurable)
  - Dashboard: Grafana or equivalent, showing all key metrics in real-time

#### NFR6.4 Health Checks
- **Requirement**: Load balancer can detect unhealthy instances
- **Specification**:
  - Each service exposes /health endpoint returning HTTP 200 + JSON status
  - Health check includes: database connection status, Redis connection status, Key Vault accessibility
  - Health check runs every 10 seconds from load balancer
  - If health check fails 3 times in a row: instance marked unhealthy and removed from load balancer

#### NFR6.5 Deployment & Rollout
- **Requirement**: New code is deployed safely
- **Specification**:
  - CI/CD pipeline: automated tests run on every commit
  - Deployment: blue-green deployment (new version runs alongside old, traffic switched atomically)
  - Rollback: automatic rollback if error rate exceeds threshold after deployment
  - Zero-downtime deployment: existing connections not interrupted
  - Gradual rollout: option to send only 10% of traffic to new version for canary testing

#### NFR6.6 Database Migrations
- **Requirement**: Schema changes are applied safely
- **Specification**:
  - All migrations must be backwards-compatible (never breaking changes)
  - Migrations tested in staging environment before production
  - Rollback plan for every migration documented
  - Zero-downtime migrations: no locks held for > 5 seconds
  - Version tracking: all migrations tracked in schema_versions table

#### NFR6.7 Configuration Management
- **Requirement**: Configuration is external to code
- **Specification**:
  - All configuration from environment variables or Azure Key Vault
  - No secrets in code, config files, or version control
  - Configuration changes do not require code deployment
  - Feature flags for gradual rollout of new features
  - Rollback: feature flag can disable problematic feature immediately

---

### NFR7: Cost Optimization

#### NFR7.1 Resource Efficiency
- **Requirement**: Platform operates efficiently
- **Specification**:
  - API caching: Redis cache for project config, API keys, MCP tools (1-hour TTL)
  - Connection pooling: database connection pool, min 10, max 100
  - Lazy loading: LLM credentials loaded only at session initialization, cached for 5 minutes
  - Garbage collection: automatic cleanup of expired tokens, audit logs older than 90 days
  - Batch operations: MCP tool discovery, document indexing done asynchronously

#### NFR7.2 Storage Efficiency
- **Requirement**: Platform minimizes storage costs
- **Specification**:
  - Conversation history: kept only in ephemeral Redis, not persisted
  - Sessions table: stores only metadata, not messages
  - Audit logs: archived to cold storage after 1 year
  - Blob storage: documents retained for 1 year, then deleted (per plan)
  - Database indexes: aggressive indexing for query performance, trade-off with write performance

#### NFR7.3 Network Efficiency
- **Requirement**: Platform minimizes bandwidth usage
- **Specification**:
  - Message compression: gzip compression for SSE responses
  - Delta streaming: send only text deltas (not full text repeatedly)
  - Pagination: never return > 100 items in a response
  - ETags: HTTP caching for static catalog data

---

### NFR8: Documentation & Support

#### NFR8.1 API Documentation
- **Requirement**: API is well-documented
- **Specification**:
  - OpenAPI/Swagger spec: auto-generated from code
  - All endpoints documented: method, path, parameters, request body, response body, status codes
  - Code examples: cURL, Python, JavaScript, Go
  - Error codes documented: meaning, cause, remediation
  - Hosted on developer portal (docs.chatn.example.com)

#### NFR8.2 Integration Guide
- **Requirement**: Users can easily integrate with the platform
- **Specification**:
  - Step-by-step guide: register, create project, configure, generate token, connect
  - Code examples: WebSocket + SSE client implementations
  - Sample apps: Python, Node.js, Go applications
  - FAQ: common questions and solutions
  - Video tutorials: setup, configuration, debugging

#### NFR8.3 Troubleshooting Guides
- **Requirement**: Common issues are easy to resolve
- **Specification**:
  - Troubleshooting section: "Why can't I create a token?", "What's a session timeout?", etc.
  - Error message reference: what each error code means and how to fix it
  - Debug mode: way to enable verbose logging for debugging
  - Support contact: link to support team, SLA for response time

#### NFR8.4 Status Page
- **Requirement**: Users can see platform status
- **Specification**:
  - Public status page: shows uptime, active incidents, scheduled maintenance
  - Status page updated in real-time as incidents occur
  - Historical uptime: 30-day, 90-day, annual
  - Email notifications: users can subscribe to updates

---

### NFR9: Compliance & Legal

#### NFR9.1 GDPR Compliance
- **Requirement**: Platform complies with GDPR
- **Specification**:
  - Data Processing Agreement (DPA) available
  - Right to be forgotten: users can request data deletion via support
  - Data portability: users can export their projects and configurations
  - Consent: email consent for marketing communications
  - Privacy Policy: published and accessible
  - Data retention policy: explained to users

#### NFR9.2 Terms of Service
- **Requirement**: Legal terms are clear
- **Specification**:
  - Terms of Service: published, updated annually
  - Acceptable use policy: what users can/cannot do (no spam, abuse, illegal content)
  - Liability limitations: platform provided "as-is"
  - Warranties: minimal warranties, clear what is not covered
  - Changes to terms: 30-day notice before changes take effect

#### NFR9.3 Data Residency
- **Requirement**: Data can be kept in specified regions
- **Specification**:
  - Option to specify data residency (US, EU, APAC)
  - All data stored in selected region
  - Cross-region replication disabled for compliance
  - Backups stored in same region

---

### NFR10: Extensibility & Future-Proofing

#### NFR10.1 Adapter Extensibility
- **Requirement**: New LLM providers can be added without code changes
- **Specification**:
  - Constraint C-06: new provider = new adapter class + database catalog insert
  - Adapter interface (ILLMSessionAdapter) fixed and stable
  - No changes to Protocol Handler, Connection Manager, or Session State Machine
  - Admin can add provider to catalog via admin API

#### NFR10.2 Transport Extensibility
- **Requirement**: New transport protocols can be added
- **Specification**:
  - Transport abstraction (IClientTransport) allows new implementations
  - Example: WebTransport, gRPC streaming could be added
  - Protocol Handler unaware of transport type
  - Admin can enable transport in catalog (supports_websocket, supports_sse flags)

#### NFR10.3 Tool System Extensibility
- **Requirement**: New tool types can be added
- **Specification**:
  - Tool taxonomy (client, gateway, builtin, mcp) is extensible
  - New tool sources can be added: webhook tools (future), Lambda functions (future)
  - Protocol Handler routing logic handles new tool types

#### NFR10.4 A/B Testing Ready
- **Requirement**: Platform architecture supports future A/B testing
- **Specification**:
  - System prompts are immutable, versioned, archived
  - Session records capture which prompt was used
  - Column exists for future traffic_weight (A/B test weight allocation)
  - No code changes needed when A/B testing is implemented

---

## Summary Table

| Category | # Reqs | Key Areas |
|---|---|---|
| **Functional** | 110+ | Authentication, Projects, Configuration, Prompts, Keys, Tokens, RAG, MCP, Sessions, Usage, Catalog, Admin, Email |
| **Non-Functional** | 50+ | Performance, Scalability, Reliability, Security, Privacy, Maintainability, Cost, Documentation, Compliance, Extensibility |
| **Total** | 160+ | End-to-end platform requirements |

All requirements are SMART (Specific, Measurable, Achievable, Relevant, Time-bound) and aligned with your SRS document.