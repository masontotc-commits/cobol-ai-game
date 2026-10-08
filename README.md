# cobol-ai-game
A modern AI-powered gaming platform combining COBOL backend engineering with modern web technologies.

COBOL AI Game Engine

A modern software engineering project that combines COBOL, AI, APIs, and a modern web-based gaming frontend.

The goal is to demonstrate how traditional enterprise/mainframe technologies can be integrated with modern software architecture, artificial intelligence, and interactive applications.

🚀 Project Overview

┌───────────────────────────────┐
│       MODERN FRONTEND         │
│      React / Phaser / Web     │
└───────────────┬───────────────┘
                │
               API
                │
                ▼
┌───────────────────────────────┐
│         API / SERVICES         │
│       Integration Layer        │
└───────────────┬───────────────┘
                │
       ┌────────┴────────┐
       ▼                 ▼
┌──────────────┐  ┌──────────────┐
│ COBOL ENGINE │  │  AI MODULE   │
│ Game Logic   │  │ Intelligence │
└──────────────┘  └──────────────┘

🎯 Objectives

* Demonstrate modern use of COBOL beyond traditional batch processing.
* Integrate COBOL business logic with modern APIs.
* Explore AI integration with enterprise systems.
* Build an interactive gaming application.
* Demonstrate clean software architecture.
* Practice Git and GitHub-based development.
* Create a portfolio project combining legacy and modern technologies.

🧩 Technology Stack

Backend

* COBOL
* REST APIs
* JSON
* API integration

Frontend

* React
* Phaser
* JavaScript / TypeScript
* HTML5
* CSS

AI

* AI models
* AI modules
* Game intelligence
* Decision-making logic

Development

* Git
* GitHub
* VS Code
* Automated testing
* Documentation

📁 Project Structure

cobol-ai-game/
│
├── frontend/
│   ├── src/
│   ├── public/
│   └── README.md
│
├── backend/
│   ├── cobol/
│   ├── api/
│   └── docs/
│       └── architecture.md
│
├── ai/
│   ├── models/
│   ├── modules/
│   └── README.md
│
├── tests/
│
├── .gitignore
├── README.md
└── LICENSE

🏗️ Architecture

The application is divided into four logical components: the frontend, API layer, COBOL engine, and AI module. Each component has a defined responsibility so that game rules, integration concerns, presentation, and intelligent behaviour remain independently testable and replaceable.

Frontend

The React and Phaser frontend provides the interactive game experience. It renders the game state, captures player actions, sends requests to the API, and updates the interface from the API response.

The frontend does not implement authoritative game rules. It may perform basic input validation and presentation logic, but the backend remains responsible for validating actions and determining the resulting game state.

API Layer

The API layer is the integration boundary between the frontend and backend services. Its responsibilities include:

* Exposing REST endpoints for game operations.
* Validating request structure, authentication, and basic input constraints.
* Translating JSON requests into internal service commands.
* Coordinating calls to the COBOL engine and AI module.
* Normalising COBOL and AI responses into stable API responses.
* Handling errors, timeouts, retries, and correlation identifiers.
* Preventing implementation details from leaking into the frontend.
* Applying rate limits and other service-level controls where required.

Example endpoints may include:

POST /api/v1/games
GET  /api/v1/games/{gameId}
POST /api/v1/games/{gameId}/actions
POST /api/v1/games/{gameId}/ai-turn

The API should remain stateless where possible. Game state may be stored in a persistence layer in future versions, while the API coordinates access to that state.

API Contract

The API uses JSON over HTTPS. Unless otherwise stated, clients should send Content-Type: application/json and accept application/json.

Authentication

Protected endpoints expect a bearer token:

Authorization: Bearer <access-token>

POST /api/v1/games and all game-specific endpoints require authentication. The token identifies the player and determines whether the player may access the requested game. Authentication and authorisation failures must not reveal whether another user’s game exists.

Versioning

The initial contract is versioned through the URL prefix:

/api/v1/...

Future breaking changes will use a new major version, such as /api/v2/.... Backward-compatible additions, including optional response fields, may be introduced within the same major version. Clients should ignore unknown response fields.

Common Headers

Clients may send a request correlation identifier:

X-Correlation-Id: 7f3c2d1a-8f4e-4f2a-9f1d-123456789abc

If supplied, the API returns the same value. Otherwise, the API generates one and includes it in the response.

Example Request Schemas

Create a game:

POST /api/v1/games
{
  "mode": "SINGLE_PLAYER",
  "difficulty": "NORMAL"
}

Get a game:

GET /api/v1/games/{gameId}

No request body is required.

Submit a player action:

POST /api/v1/games/{gameId}/actions
{
  "action": "MOVE",
  "target": "NORTH",
  "clientRequestId": "move-0001"
}

Request fields:

Field	Type	Required	Description
action	string	Yes	Action requested by the player.
target	string	Conditional	Action target, direction, or entity.
clientRequestId	string	No	Client-generated identifier used for safe retries and deduplication.

Request validation is case-sensitive unless the endpoint explicitly documents otherwise. Unknown fields may be ignored for forward compatibility, but required fields and allowed values must be validated by the API.

Example Response Schemas

Create or retrieve a game:

{
  "gameId": "123",
  "status": "ACTIVE",
  "state": {
    "playerPosition": "START",
    "score": 0
  },
  "availableActions": [
    "MOVE",
    "WAIT"
  ],
  "createdAt": "2025-01-15T10:30:00Z",
  "updatedAt": "2025-01-15T10:30:00Z"
}

Successful action:

{
  "gameId": "123",
  "status": "ACTIVE",
  "state": {
    "playerPosition": "NORTH",
    "score": 100
  },
  "events": [
    {
      "type": "PLAYER_MOVED",
      "message": "Player moved north."
    }
  ],
  "nextAction": "OPPONENT_TURN",
  "correlationId": "7f3c2d1a-8f4e-4f2a-9f1d-123456789abc"
}

The state object is authoritative for the returned transition. Clients must not assume that an action succeeded unless the API returns a successful status code and the response contains the resulting state.

Common HTTP Status Codes

Status	Meaning	Typical use
200 OK	Request succeeded	Retrieve a game or apply an action.
201 Created	Resource created	Create a new game.
400 Bad Request	Invalid request	Malformed JSON, missing fields, or invalid values.
401 Unauthorized	Authentication required or invalid	Missing, expired, or invalid bearer token.
403 Forbidden	Access denied	Authenticated user cannot access the game.
404 Not Found	Resource not found	Game does not exist or is not visible to the caller.
409 Conflict	State conflict	Action is invalid for the current game state or duplicate request.
422 Unprocessable Entity	Semantically invalid input	Well-formed request that violates game rules.
429 Too Many Requests	Rate limit exceeded	Client must slow down and may use Retry-After.
500 Internal Server Error	Unexpected server failure	Unhandled API or integration error.
502 Bad Gateway	Upstream failure	COBOL or AI service returned an invalid or unavailable response.
503 Service Unavailable	Temporary unavailability	Service is unavailable or undergoing maintenance.

Error Format

All error responses use a consistent structure:

{
  "error": {
    "code": "INVALID_ACTION",
    "message": "The requested move is not valid from the current position.",
    "details": [
      {
        "field": "target",
        "reason": "Target NORTH is unavailable."
      }
    ],
    "retryable": false
  },
  "correlationId": "7f3c2d1a-8f4e-4f2a-9f1d-123456789abc"
}

Error fields:

Field	Type	Description
error.code	string	Stable machine-readable error code.
error.message	string	Safe, human-readable summary.
error.details	array	Optional validation or diagnostic details.
error.retryable	boolean	Indicates whether retrying may succeed.
correlationId	string	Identifier for tracing and support.

Error messages must not expose credentials, internal stack traces, sensitive game data, or implementation-specific COBOL details. Internal return codes may be logged and mapped to stable public error codes.

COBOL Engine

The COBOL engine is the authoritative source for deterministic game and business rules. It is responsible for operations such as:

* Validating player actions against the current game state.
* Calculating scores, rewards, movement, or outcomes.
* Applying deterministic rules consistently.
* Enforcing limits and state transitions.
* Returning structured results and error codes.

The COBOL engine should not depend directly on frontend concerns or AI-provider-specific logic. It should receive a well-defined input structure and return a predictable output structure.

A typical integration boundary may use JSON over an API adapter, although other approaches may be introduced later:

API request
    │
    ▼
Integration adapter
    │
    ├── Maps JSON to COBOL input fields
    ├── Invokes the COBOL program or service
    ├── Captures return codes and output fields
    └── Maps COBOL output to an API response

The adapter isolates differences between modern API formats and COBOL data structures, including field names, data types, numeric formats, fixed-width records, return codes, and error handling. This allows the COBOL program to remain focused on game logic while the integration layer manages transport and translation.

The COBOL integration may initially run as a local executable or service for development. Future deployments may connect to a mainframe transaction, batch process, containerised COBOL runtime, or another enterprise execution environment.

AI Module

The AI module provides optional intelligent behaviour, such as opponent decisions, recommendations, dialogue, difficulty adjustment, or procedural game content.

AI responsibilities are intentionally bounded:

* Generate suggestions or decisions from approved game-state data.
* Select actions for AI-controlled players.
* Provide non-authoritative narrative or interaction content.
* Return structured outputs that can be validated by the API and COBOL engine.

The AI module must not be the final authority for core game rules, scoring, security, player balances, or state transitions. AI-generated actions are treated as proposals. The API and COBOL engine validate those proposals before they affect the game.

The AI module should also avoid receiving unnecessary sensitive data. Requests should contain only the game context required for the specific decision, and model failures, timeouts, invalid outputs, and unavailable providers should have deterministic fallback behaviour.

Data Flows

The primary data flow is request-driven:

Player action
    │
    ▼
Frontend
    │  JSON request
    ▼
API layer
    │
    ├── Validates request
    ├── Loads or receives current game state
    ├── Calls COBOL for deterministic rules
    └── Optionally calls AI for an intelligent decision
            │
            ▼
       Validated result
            │
            ▼
API response
    │
    ▼
Frontend updates the game view

A typical state transition follows this sequence:

1. The frontend sends a player action and game identifier.
2. The API validates the request and checks that the game is available.
3. The API passes the relevant state and action to the COBOL engine.
4. The COBOL engine validates the action and calculates the deterministic result.
5. If an AI decision is required, the API sends a limited game context to the AI module.
6. Any AI-generated action is validated by the COBOL engine before being applied.
7. The API returns the updated state, events, messages, and any error information to the frontend.
8. The frontend renders the new state.

Request Lifecycle Example

For a player choosing an action during a game:

1. Frontend
   POST /api/v1/games/123/actions
   { "action": "MOVE", "target": "NORTH" }
2. API Layer
   Validates the payload and forwards the command.
3. COBOL Engine
   Checks whether the move is legal, updates the game state,
   calculates the result, and returns a status code.
4. AI Module
   If the move triggers an opponent turn, proposes the opponent's action.
5. COBOL Engine
   Validates and applies the AI-proposed action.
6. API Layer
   Combines the authoritative state and events into a response.
7. Frontend
   Displays the updated board, score, messages, and next available actions.

Example response:

{
  "gameId": "123",
  "status": "ACTIVE",
  "state": {
    "playerPosition": "NORTH",
    "score": 100
  },
  "events": [
    {
      "type": "PLAYER_MOVED",
      "message": "Player moved north."
    }
  ],
  "nextAction": "OPPONENT_TURN"
}

Error and Reliability Boundaries

Each component should return clear, structured errors:

* The frontend displays user-friendly messages.
* The API returns appropriate HTTP status codes and correlation identifiers.
* The COBOL engine returns documented status codes and validation results.
* The AI module reports unavailable, invalid, or timed-out responses without blocking core deterministic processing unnecessarily.

If the AI module is unavailable, the game should continue using a deterministic fallback where possible. If the COBOL engine is unavailable, the API must not claim that a state transition succeeded.

🔐 Environment Variables

Environment-specific configuration must be supplied through environment variables or a managed configuration service. Secrets must not be committed to the repository.

Create local configuration from the example files provided by each component, such as .env.example, and keep actual .env files untracked.

Typical API variables include:

API_PORT=8080
API_BASE_PATH=/api/v1
LOG_LEVEL=info
CORS_ALLOWED_ORIGINS=http://localhost:3000
COBOL_ENGINE_URL=http://localhost:8081
COBOL_ENGINE_TIMEOUT_MS=5000
AI_SERVICE_URL=http://localhost:8000
AI_SERVICE_TIMEOUT_MS=10000
AI_PROVIDER=
AI_MODEL=
DATABASE_URL=
AUTH_ISSUER=
AUTH_AUDIENCE=
JWT_PUBLIC_KEY=

Typical frontend variables include:

VITE_API_BASE_URL=http://localhost:8080/api/v1
NEXT_PUBLIC_API_BASE_URL=http://localhost:8080/api/v1

Only variables intended for browser exposure should use a frontend public prefix such as VITE_ or NEXT_PUBLIC_. Never expose private keys, database credentials, signing secrets, or provider tokens through frontend configuration.

Recommended practices:

* Provide .env.example files containing names and safe placeholder values.
* Validate required variables during application startup.
* Fail fast when mandatory configuration is missing or malformed.
* Use separate configuration for development, testing, staging, and production.
* Store production secrets in a secret manager or deployment platform.
* Rotate credentials regularly and after suspected exposure.
* Do not log secret values or complete connection strings.

🛠️ Local Development Dependencies

The exact versions should be documented in component-specific manifests, but local development generally requires:

* Git
* A code editor such as VS Code
* Node.js and npm, pnpm, or Yarn for the frontend and API tooling
* A supported COBOL compiler or runtime, such as GnuCOBOL or an approved enterprise runtime
* Python 3.x if the AI module uses Python
* Docker and Docker Compose, if services are containerised
* A local database, if persistence is enabled
* An HTTP client such as curl, Postman, or HTTPie
* Optional: OpenAPI tooling for contract validation
* Optional: a local observability stack for logs, metrics, and traces

Before starting the application, verify the installed tools:

git --version
node --version
npm --version
cobc --version
python --version
docker --version

Not every component is required for every development task. For example, frontend-only work may require only Node.js, while COBOL engine work requires the compiler or runtime and its supporting libraries.

A typical local startup sequence is:

# Install frontend and API dependencies
npm install
# Install AI dependencies when applicable
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
# Build the COBOL component according to its local instructions
make -C backend/cobol
# Start supporting services
docker compose up -d
# Start the API
npm run dev
# Start the frontend in a separate terminal
npm run dev --workspace frontend

The commands may differ depending on the selected framework and build tooling. Each component should provide a local README.md with authoritative commands.

📊 Observability

Observability should make it possible to understand request flow, performance, failures, and dependency health without exposing sensitive information.

Logging

Services should emit structured logs, preferably as JSON, with fields such as:

* Timestamp
* Log level
* Service name
* Environment
* Correlation ID
* Request ID
* Route or operation
* Duration
* HTTP status
* Upstream dependency
* Sanitised error code

Logs must not contain access tokens, passwords, API keys, private user data, full request bodies, or sensitive game state unless explicitly approved and protected.

Use appropriate log levels:

* DEBUG for local diagnostic details.
* INFO for normal lifecycle and business events.
* WARN for recoverable failures, retries, and degraded dependencies.
* ERROR for failed requests, unavailable dependencies, and unexpected exceptions.

Metrics

Recommended metrics include:

* Request count by route and status.
* Request latency by route.
* COBOL invocation count, latency, and failure rate.
* AI request count, latency, timeout rate, and fallback count.
* Authentication failures.
* Rate-limit responses.
* Active games and state-transition failures.
* Database connection and query failures.
* Process health and resource usage.

Metrics should use bounded labels. Do not use raw user IDs, game IDs, correlation IDs, or arbitrary request values as metric labels.

Tracing and Correlation

The API should propagate a correlation identifier to downstream services. Where supported, use distributed tracing standards such as W3C Trace Context.

At minimum:

* Accept or generate X-Correlation-Id.
* Include it in responses and logs.
* Pass it to the COBOL adapter and AI module.
* Record it with integration failures.
* Return it to support personnel without exposing internal diagnostics.

Health Checks

Provide separate health checks where practical:

GET /health/live
GET /health/ready
GET /health/dependencies

* Liveness indicates that the process is running.
* Readiness indicates that the service can accept traffic.
* Dependency health reports the status of required integrations without exposing credentials or internal topology.

Health checks should have short timeouts and should not trigger expensive game operations.

🔒 Security Practices

Security is a cross-cutting requirement across the frontend, API, COBOL engine, AI module, and deployment environment.

Authentication and Authorisation

* Require authentication for protected endpoints.
* Validate token signature, issuer, audience, expiry, and required claims.
* Authorise access to each game resource server-side.
* Do not trust player or game ownership values supplied by the frontend.
* Return consistent responses for inaccessible resources where appropriate.
* Use short-lived access tokens and secure refresh-token handling.

Input and Output Validation

* Validate JSON structure, field types, lengths, and allowed values.
* Reject malformed or unexpectedly large requests.
* Validate all AI-generated actions before applying them.
* Use allowlists for actions, modes, difficulty values, and targets.
* Escape or sanitise user-generated content before rendering it.
* Avoid dynamic command construction and unsafe deserialisation.
* Validate COBOL numeric and fixed-width fields at the adapter boundary.

Secrets and Sensitive Data

* Store secrets in environment variables only for local development and in a secret manager for deployed environments.
* Never commit credentials, private keys, tokens, or production configuration.
* Redact secrets from logs, error messages, traces, and support bundles.
* Minimise the game and player data sent to external AI providers.
* Review provider retention, training, and data-processing policies before sending data externally.

Transport and Network Security

* Use HTTPS outside isolated local development.
* Validate TLS certificates for upstream services.
* Restrict service-to-service network access.
* Configure CORS with explicit allowed origins.
* Apply rate limits to authentication, game creation, actions, and AI endpoints.
* Use secure headers and disable unnecessary server information disclosure.

Dependency and Supply-Chain Security

* Pin or constrain dependency versions.
* Review dependency updates before deployment.
* Run vulnerability scanning for application and container dependencies.
* Use trusted base images and rebuild them regularly.
* Generate software bills of materials where required.
* Protect CI/CD credentials and restrict deployment permissions.

Data Protection and Recovery

* Store only the data required by the application.
* Define retention and deletion rules for game and player data.
* Encrypt sensitive data at rest where applicable.
* Back up persistent game data and test restoration procedures.
* Avoid placing secrets or sensitive state in client-side storage.

🚢 Deployment Assumptions

The project assumes that the frontend, API, COBOL engine, and AI module may be deployed independently, although a local or demonstration deployment may run them on one machine.

Runtime Assumptions

* The frontend is served through a static hosting platform or web server.
* The API is deployed as a long-running service or container.
* The COBOL engine is available through a stable adapter, executable, service, transaction, or enterprise integration endpoint.
* The AI module is available through an internal service or approved external provider.
* Persistent game state, if enabled, is stored in a managed database or approved enterprise data store.
* Configuration and secrets are injected at deployment time.
* Services communicate over authenticated and encrypted channels in non-local environments.

Availability and Failure Assumptions

* The API must not report successful state changes when the COBOL engine has not confirmed them.
* AI availability is optional where deterministic fallback behaviour exists.
* COBOL engine availability is required for authoritative state transitions.
* Upstream calls use explicit connection and response timeouts.
* Retries are limited, use exponential backoff, and are applied only to safe or idempotent operations.
* Duplicate player actions are controlled with clientRequestId or an equivalent idempotency mechanism.
* Deployments should support health checks and graceful shutdown.
* Database migrations must be reviewed and applied in a controlled manner.

Deployment Environments

Recommended environments include:

development → test → staging → production

Each environment should have separate credentials, data, service endpoints, and access controls. Production data must not be copied into development without approved masking and handling procedures.

CI/CD Expectations

A deployment pipeline should:

1. Install dependencies from locked or approved versions.
2. Run formatting, linting, unit tests, and integration tests.
3. Compile and test the COBOL engine.
4. Validate API contracts and example payloads.
5. Scan dependencies and container images.
6. Build versioned artifacts.
7. Deploy to a non-production environment.
8. Run smoke tests and health checks.
9. Require appropriate approval before production deployment.
10. Record the deployed version and configuration reference.

🔧 Troubleshooting Common Integration Failures

Frontend Cannot Reach the API

Symptoms:

* Browser network requests fail.
* The frontend reports a connection or CORS error.
* The API appears healthy when accessed directly.

Checks:

1. Confirm the frontend API base URL is correct.
2. Confirm the API is running on the expected host and port.
3. Check browser developer tools for the exact request URL.
4. Verify CORS allows the frontend origin.
5. Confirm local proxies or reverse proxies are configured correctly.
6. Check whether HTTPS and HTTP are being mixed.
7. Review API access logs using the correlation identifier.

401 Unauthorized or 403 Forbidden

Checks:

* Confirm the bearer token is present and not expired.
* Verify issuer, audience, signature, and required claims.
* Confirm the authenticated user owns or may access the game.
* Check clock synchronisation between token issuer and API.
* Ensure the frontend is not accidentally stripping the Authorization header.
* Review authentication logs without logging the token itself.

400, 409, or 422 Responses

Checks:

* Compare the request with the documented API schema.
* Confirm required fields and allowed values.
* Verify the action is valid for the current game state.
* Check whether a clientRequestId was already processed.
* Inspect the returned stable error code and correlation ID.
* Do not retry validation or state-conflict errors without changing the request or state.

COBOL Engine Returns 502 or 503

Checks:

1. Confirm the COBOL process, transaction, or service is running.
2. Verify COBOL_ENGINE_URL and related connection settings.
3. Test the integration endpoint directly with a known-safe request.
4. Check adapter logs for timeout, connection, encoding, or mapping errors.
5. Confirm the request matches the COBOL copybook or input contract.
6. Check numeric formats, field widths, padding, signs, and character encoding.
7. Verify the COBOL return code and map it to the documented API error.
8. Confirm the API does not retry non-idempotent operations unsafely.

Common causes include:

* Incorrect host or port.
* Service not started.
* Firewall or network policy restrictions.
* EBCDIC/ASCII or UTF-8 conversion problems.
* Fixed-width field truncation.
* Invalid packed-decimal or numeric data.
* Copybook and adapter version mismatch.
* Unexpected null, blank, or default values.
* COBOL runtime or shared-library configuration errors.

AI Module Times Out or Returns Invalid Output

Checks:

* Confirm the AI service URL, model, and credentials are configured.
* Check provider quotas, rate limits, and service status.
* Verify request and response timeouts.
* Inspect the sanitised prompt or structured request shape.
* Validate that the response matches the expected schema.
* Confirm fallback behaviour is enabled.
* Ensure AI output is treated as a proposal and validated by the COBOL engine.
* Check whether the request contains unsupported or excessive context.

The API should return a controlled response or deterministic fallback rather than exposing provider errors or blocking indefinitely.

JSON or Encoding Errors

Checks:

* Confirm the request uses Content-Type: application/json.
* Validate JSON with a parser before sending it to the adapter.
* Check character encoding and line-ending conversions.
* Verify COBOL field lengths and numeric representations.
* Confirm special characters are escaped correctly.
* Compare the adapter payload with the COBOL input contract.
* Check whether an upstream service returned HTML or plain text instead of JSON.

Duplicate or Out-of-Order Actions

Checks:

* Confirm the frontend sends a unique clientRequestId.
* Verify the API stores or checks idempotency keys where required.
* Check whether the user double-clicked or retried after a timeout.
* Confirm game-state version or optimistic-lock checks are working.
* Ensure the frontend refreshes authoritative state after a conflict.
* Do not assume a timeout means the action failed; query the game state before retrying.

Health Check Fails After Deployment

Checks:

* Review startup logs for missing environment variables.
* Confirm the service can resolve and reach required dependencies.
* Verify readiness checks do not require optional services.
* Check database migrations and credentials.
* Confirm the deployed artifact contains the expected configuration and build version.
* Inspect resource limits, port bindings, and network policies.
* Use the deployment correlation or release identifier when reviewing logs.

🔄 Development Approach

The project is being developed incrementally:

1. Define the architecture.
2. Build the core COBOL engine.
3. Create the API integration layer.
4. Develop the AI module.
5. Build the frontend.
6. Connect the components.
7. Add automated tests.
8. Add observability and security controls.
9. Validate deployment assumptions.
10. Deploy and document the application.

📚 Documentation

Architecture documentation:

backend/docs/architecture.md

Additional documentation should cover:

* Component-specific setup instructions.
* Environment variable references.
* API schemas and examples.
* COBOL copybooks and adapter mappings.
* AI provider configuration and fallback behaviour.
* Deployment manifests and operational runbooks.
* Security and data-handling decisions.
* Monitoring dashboards and alert definitions.

🧪 Testing

Testing will cover:

* COBOL business logic.
* API integration.
* AI modules.
* Frontend functionality.
* End-to-end application behaviour.
* Request and response validation.
* Error handling and fallback behaviour.
* Contract compatibility between the API adapter and COBOL engine.
* Authentication and authorisation.
* Idempotency and duplicate requests.
* Timeout, retry, and dependency-failure scenarios.
* Observability fields and health checks.
* Security scanning and dependency vulnerabilities.

🔮 Future Development

Potential future capabilities include:

* AI-controlled opponents.
* Multiplayer functionality.
* Player profiles.
* Game analytics.
* Persistent game data.
* Cloud deployment.
* Event-driven processing.
* Mainframe integration.
* AI-assisted game generation.
* Real-time APIs.
* Distributed tracing.
* Automated deployment rollbacks.
* Disaster recovery testing.
* Advanced operational dashboards.

👨🏽‍💻 Author

Tiyani Masonto

Senior Mainframe / COBOL Developer
Mainframe Specialist | Core Banking Technology | Software Engineering

South Africa

⸻

Exploring the intersection between enterprise COBOL systems, modern APIs, artificial intelligence, and software engineering.

📄 License

This project is intended primarily as a software engineering and portfolio project. License terms will be added as the project matures.
