# API & File Security Skill

## Purpose

This skill teaches the agent how to analyze APIs and file-handling functionality for common security weaknesses in authorized applications, labs, and CTF environments.

The skill focuses on:

- REST API security
- GraphQL security
- API authentication and authorization
- Input validation
- Mass assignment
- HTTP method abuse
- File upload security
- File download/path traversal
- Content-type validation
- API information exposure
- Rate-limit observations

The goal is:

> **Understand how an API or file operation is supposed to behave, then verify whether security controls actually enforce that behavior.**

---

# 1. Scope and Safety

Use dedicated test accounts and test files.

Never:

- Download private user files
- Upload malware
- Upload destructive payloads
- Access unauthorized tenant data
- Modify production data
- Attempt destructive file operations
- Exfiltrate secrets
- Bypass authentication on real systems

---

# 2. API Security Workflow

```text id="9v7h2k"
Discover API
    ↓
Map Endpoints
    ↓
Identify Authentication
    ↓
Identify Roles
    ↓
Map Request/Response
    ↓
Test Authorization
    ↓
Test Input Validation
    ↓
Test HTTP Methods
    ↓
Test Mass Assignment
    ↓
Test Error Handling
    ↓
Test Rate Limiting
    ↓
Analyze Sensitive Data
    ↓
Document Findings
```

---

# 3. API Discovery

Look for common API locations:

```text id="3d8xqk"
/api
/api/v1
/api/v2
/api/users
/api/auth
/api/admin
/api/upload
/api/download
```

Also inspect:

```text id="2u5f7p"
JavaScript files
OpenAPI specifications
Swagger documentation
GraphQL endpoints
Network requests
Mobile application traffic
```

Common documentation paths:

```text id="c7n2pz"
/swagger
/swagger-ui
/openapi.json
/swagger.json
/api-docs
```

Do not assume an endpoint exists just because the path is common.

---

# 4. Build an API Inventory

Create a table:

| Method | Endpoint | Auth | Role | Purpose |
|---|---|---|---|---|
| GET | `/api/profile` | Yes | User | Profile |
| POST | `/api/orders` | Yes | User | Create order |
| GET | `/api/admin/users` | Yes | Admin | User management |
| DELETE | `/api/order/:id` | Yes | Owner | Delete order |

This becomes the API attack-surface map.

---

# 5. Analyze Authentication

For every API endpoint determine:

```text id="4w8nqv"
Does it require authentication?
What token/cookie is used?
Does logout invalidate access?
Does the token expire?
Is authorization checked server-side?
```

Test using dedicated accounts.

Example:

```text id="8b2z5m"
User A token → own resource
User B token → User B resource
User A token → User B test resource
```

Do not use real users' resources.

---

# 6. Authorization Testing

API authentication does not automatically mean authorization is correct.

Check:

```text id="9f7x1a"
User → own object
User → another test user's object
User → admin-only endpoint
Admin → normal endpoint
```

Potential issues:

```text id="5x3w2m"
BOLA
IDOR
Broken Function Level Authorization
Privilege escalation
Tenant isolation failure
```

For detailed access-control testing, use the dedicated Access Control skill.

---

# 7. HTTP Method Testing

An endpoint may behave differently depending on the HTTP method.

For an authorized application, compare supported methods:

```text id="r2d7qp"
GET
POST
PUT
PATCH
DELETE
```

Example:

```text id="a8xkq3"
GET /api/profile
```

Determine whether unsupported methods are properly rejected.

Expected behavior might be:

```text id="6s8q1z"
405 Method Not Allowed
```

Do not perform destructive methods against production data.

Use test resources.

---

# 8. Input Validation

Identify every API-controlled field:

```json id="1x7f3p"
{
  "username": "aryan",
  "age": 21,
  "role": "user"
}
```

Test:

```text id="k3z8qm"
missing fields
empty values
unexpected types
very long values
unexpected JSON fields
null values
arrays instead of strings
strings instead of numbers
```

Example:

```json id="h2r6wb"
{
  "age": "not-a-number"
}
```

Observe whether the API:

```text id="z5w4rc"
rejects input
normalizes input
silently accepts it
causes an error
stores invalid data
```

---

# 9. Mass Assignment

Mass assignment occurs when an API accepts fields that the client should not control.

Example legitimate request:

```json id="4c7k2n"
{
  "name": "Aryan"
}
```

But the backend may accidentally accept:

```json id="q6y9pa"
{
  "name": "Aryan",
  "role": "admin"
}
```

or:

```json id="7d2m8x"
{
  "name": "Aryan",
  "isVerified": true
}
```

In an authorized test environment:

1. Identify server-managed fields.
2. Determine whether they are accepted from the client.
3. Use a dedicated test account.
4. Check whether the value actually changes.
5. Restore the test account if necessary.

Do not attempt privilege escalation against real accounts.

---

# 10. API Response Analysis

Look for unnecessary sensitive information:

```text id="f4m9vc"
password hashes
API keys
internal IDs
database IDs
internal URLs
debug information
stack traces
tokens
private metadata
```

Example:

```json id="8p1m6v"
{
  "id": 123,
  "name": "Aryan",
  "email": "test@example.com",
  "internalDatabaseId": 93281
}
```

Determine whether each field is actually required by the client.

---

# 11. Error Handling

Send controlled invalid input and inspect the response.

Look for:

```text id="j4k2sn"
stack traces
framework versions
database errors
file paths
internal hostnames
debug information
source-code fragments
```

Example:

```text id="q8x3vm"
500 Internal Server Error

Error:
PostgreSQL connection failed at
/internal/database/connection.js
```

This may represent information disclosure.

Do not automatically classify every `500` as a vulnerability.

---

# 12. GraphQL Analysis

If GraphQL is present, identify:

```text id="8v4m2q"
queries
mutations
variables
object types
fields
relationships
```

Analyze:

```text id="z2q7xm"
authentication
authorization
excessive data exposure
field-level authorization
input validation
```

Pay special attention to whether a normal user can access fields intended only for privileged users.

---

# 13. GraphQL Introspection

If introspection is enabled, determine whether that is intentional.

Potentially exposed information:

```text id="q9f5wx"
types
queries
mutations
fields
arguments
relationships
```

Do not automatically report introspection as a vulnerability.

Consider:

```text id="b6m3qa"
Is this production?
Is the schema sensitive?
Does it reveal privileged functionality?
Is introspection intentionally enabled?
```

---

# 14. File Upload Security

Identify upload functionality:

```text id="7p2nvc"
/upload
/profile/photo
/document
/attachments
/import
```

Determine:

```text id="4m8x6r"
Allowed extensions
Maximum file size
Content-Type validation
Filename handling
Storage location
Authentication requirements
Authorization
Whether uploaded files are publicly accessible
```

---

# 15. Safe File Upload Testing

Use harmless files.

Examples:

```text id="x7k3qp"
test.txt
test.jpg
test.pdf
```

Test mismatches such as:

```text id="w5n2zr"
filename extension
Content-Type
actual file format
```

Example:

```text id="0a6m8v"
filename: test.jpg
Content-Type: image/jpeg
actual content: harmless text
```

Determine whether the server validates the actual file type rather than trusting only the extension.

---

# 16. File Upload Controls

Check whether the application enforces:

```text id="6q9v2m"
Extension allowlist
MIME validation
File signature validation
Maximum size
Filename sanitization
Randomized storage names
Authentication
Authorization
Non-executable storage
```

A secure application should not rely only on:

```text id="r7x1kc"
.jpg
.png
.pdf
```

because extensions can be manipulated.

---

# 17. File Download Security

Look for endpoints such as:

```text id="v3k9qa"
/download?id=123
/file/123
/uploads/document.pdf
```

Check:

```text id="2p7m4x"
Does authentication exist?
Does authorization exist?
Can User A access User B's test file?
Are filenames predictable?
Are tenant boundaries enforced?
```

This overlaps with the Access Control skill.

---

# 18. Path Traversal Analysis

If an application accepts a filename or path:

```text id="8k4m1v"
file=
filename=
path=
document=
template=
```

Determine whether the application safely restricts the requested file to its intended directory.

In a controlled lab, test traversal handling with benign files.

The goal is to determine:

```text id="c5x9qp"
Can user input escape the intended directory?
```

Do not read system secrets such as:

```text id="r2v7mn"
/etc/passwd
SSH keys
environment secrets
application credentials
```

---

# 19. File Access Authorization

Use two dedicated test accounts:

```text id="y8m3qa"
User A
User B
```

Create:

```text id="x2p7kc"
File A → owned by User A
File B → owned by User B
```

Test:

```text id="5q9v2m"
User A → File A → ALLOW
User A → File B → DENY
User B → File B → ALLOW
User B → File A → DENY
```

If unauthorized access succeeds, investigate as BOLA/IDOR.

---

# 20. Rate Limiting

Identify sensitive endpoints:

```text id="3m8q1x"
login
OTP
password reset
API key generation
expensive search
file upload
resource creation
```

Observe whether reasonable request limits exist.

Do not perform high-volume traffic or denial-of-service testing.

Look for responses such as:

```text id="5v7n2q"
429 Too Many Requests
Retry-After
```

Document observed controls rather than attempting to overwhelm the service.

---

# 21. API Security Headers

Inspect:

```text id="7x2m9p"
Content-Type
Cache-Control
CORS headers
Security headers
Set-Cookie
```

For CORS, determine:

```text id="m4q8vc"
Allowed origins
Credential handling
Wildcard usage
Origin reflection
```

Do not assume:

```text id="1z6k4p"
Access-Control-Allow-Origin: *
```

is automatically a vulnerability.

Consider whether sensitive authenticated data is exposed cross-origin.

---

# 22. API Versioning

Look for:

```text id="q3m7xb"
/api/v1
/api/v2
/api/internal
/api/legacy
```

Determine whether old versions remain accessible and whether they enforce the same security controls.

Potential concern:

```text id="c8v2mq"
v2 → authentication required
v1 → same sensitive endpoint without equivalent controls
```

---

# 23. Sensitive Endpoint Discovery

Search API documentation and frontend JavaScript for:

```text id="f5x8qm"
admin
internal
debug
export
backup
users
payments
keys
config
logs
```

Finding an endpoint is not equivalent to finding a vulnerability.

Verify:

```text id="8m3q7v"
authentication
authorization
data exposure
input validation
```

---

# 24. Evidence Collection

For each finding record:

```text id="2q8m5x"
Target:
Endpoint:
Method:
Authentication:
Role:
Request:
Expected:
Actual:
Response:
Evidence:
Impact:
Confidence:
Recommendation:
```

Example:

```text id="6v3p9m"
Finding:
Mass Assignment

Endpoint:
/api/profile

Method:
PATCH

Role:
Normal test user

Expected:
Only profile fields should be editable.

Observed:
A server-managed field was accepted and persisted.

Confidence:
High

Recommendation:
Use an explicit server-side allowlist of writable fields.
```

---

# 25. API Security Checklist

```text
[ ] API endpoints discovered
[ ] Authentication mapped
[ ] Authorization mapped
[ ] HTTP methods analyzed
[ ] Input validation tested
[ ] Mass assignment checked
[ ] Error responses analyzed
[ ] Sensitive data exposure checked
[ ] GraphQL analyzed if present
[ ] API versioning checked
[ ] CORS analyzed
[ ] Rate limiting observed
[ ] File uploads analyzed
[ ] File downloads analyzed
[ ] File authorization tested
[ ] Path handling analyzed
[ ] Evidence collected
[ ] Findings classified
```

---

# 26. Agent Decision Tree

```text
API discovered
      ↓
Does it require authentication?
      |
      ├── No → Determine whether public access is intended
      |
      └── Yes
            ↓
      Is authorization enforced?
            |
            ├── No → Investigate access control
            |
            └── Yes
                  ↓
          Analyze input/output
                  ↓
       ┌──────────┼───────────┐
       ↓          ↓           ↓
   Input       Files       Response
       ↓          ↓           ↓
 Validation   Upload/     Sensitive
 Mass assign  Download    Information
       ↓          ↓           ↓
       └──────────┼───────────┘
                  ↓
             Evidence
                  ↓
              Report
```

---

# 27. Core Principle

> **An API is secure only when authentication, authorization, input validation, data exposure, and resource handling are enforced server-side—not merely hidden by the frontend.**

For files:

> **Validate the file, isolate its storage, enforce authorization, and never trust user-controlled filenames, extensions, paths, or MIME types.**