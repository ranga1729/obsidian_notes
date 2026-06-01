#### Public
- GET /v1/health
- GET /v1/plans
#### AUTH
  POST   /auth/register
  POST   /auth/login
  POST   /auth/refresh
  POST   /auth/logout
  POST   /auth/forgot-password
  POST   /auth/reset-password

#### PROJECTS
  GET    /projects  ← list user's projects
  POST   /projects  ← create project
  GET    /projects/{id}
  PUT    /projects/{id}
  DELETE /projects/{id}

#### PROJECT CONFIGURATION
  GET    /projects/{id}/config  ← get full config
  PUT    /projects/{id}/config  ← update config (provider, model, prompt...)
  PUT    /projects/{id}/config/system-prompt
  PUT    /projects/{id}/config/tools
  PUT    /projects/{id}/config/voice
  PUT    /projects/{id}/config/transcription

#### API KEYS
  GET    /projects/{id}/api-keys
  POST   /projects/{id}/api-keys  ← create key
  DELETE /projects/{id}/api-keys/{keyId}  ← revoke key

#### EPHEMERAL TOKENS
  POST   /projects/{id}/tokens  ← issue short-lived token (e.g. 60s TTL)
  called from user's app backend, not frontend. This is a public endpoint.

#### SESSIONS
  GET    /projects/{id}/sessions  ← active + historical sessions
  GET    /projects/{id}/sessions/{sid} ← session detail
  DELETE /projects/{id}/sessions/{sid} ← force close session

#### USAGE
  GET    /projects/{id}/usage  ← tokens, audio seconds, cost estimate
  GET    /usage  ← platform-wide usage (user level)
