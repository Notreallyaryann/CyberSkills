# Secrets & Information Disclosure Skill

## Purpose

This skill teaches the agent how to identify accidentally exposed sensitive information in authorized applications, APIs, repositories, cloud configurations, and web assets.

The agent should identify:

- API keys
- Access tokens
- JWTs
- Credentials
- Database connection strings
- Private configuration
- Environment files
- Source maps
- Debug information
- Backup files
- Internal endpoints
- Internal hostnames
- Sensitive API responses
- Secrets inside JavaScript bundles
- Exposed metadata

The goal is:

> **Discover whether sensitive information is unintentionally exposed, determine its likely impact, and report it without using or abusing the secret.**

---

# 1. Scope and Safety

**Do not:**

- Use the credential
- Log into an account
- Access private systems
- Exfiltrate data
- Modify cloud resources
- Test whether the credential works against third-party services
- Share the secret publicly

Instead:

```text
Detect
  ↓
Redact
  ↓
Identify Type
  ↓
Assess Exposure
  ↓
Assess Potential Impact
  ↓
Recommend Rotation
```

---

# 2. Discovery Workflow

```text
Target
  ↓
Web Assets
  ↓
JavaScript
  ↓
API Responses
  ↓
Configuration Files
  ↓
Backup Files
  ↓
Source Maps
  ↓
Error Messages
  ↓
Repository/Build Artifacts
  ↓
Cloud/Infrastructure Metadata
  ↓
Secret Detection
  ↓
Redaction
  ↓
Risk Classification
  ↓
Report
```

---

# 3. What Is Sensitive?

Classify discovered information.

## Critical

Examples:

```text
Cloud access credentials
Private API keys
Database credentials
Private signing keys
Production authentication secrets
```

## High

Examples:

```text
Service tokens
Webhook secrets
Internal API credentials
Privileged API keys
JWT signing material
```

## Medium

Examples:

```text
Internal hostnames
Private API endpoints
Debug information
Database names
Infrastructure metadata
```

## Low / Informational

Examples:

```text
Public API identifiers
Build versions
Public documentation URLs
Non-sensitive configuration
```

Do not classify something as a vulnerability simply because it looks technical

---

# 4. Search Web Pages

Inspect:

```text
HTML
JavaScript
CSS
JSON
API responses
comments
meta tags
```

Look for patterns such as:

```text
api_key
apikey
api-key
token
secret
password
passwd
authorization
bearer
client_secret
private_key
database_url
connection_string
uri
```

These are indicators, not automatic proof of a secret.

---

# 5. JavaScript Analysis

Modern applications often expose large JavaScript bundles.

Discover:

```text
/static/
/assets/
/js/
/_next/static/
```

Download only authorized assets.

Search for:

```text
API endpoints
environment variables
configuration
tokens
service URLs
internal domains
feature flags
debug settings
```

Example:

```bash
curl -s https://authorized-target.example/assets/app.js
```

Then inspect locally.

Never paste discovered secrets into chat, logs, GitHub, or reports.

---

# 6. Source Maps

Look for:

```text
.js.map
```

Example:

```text
app.js
app.js.map
```

Source maps may reveal:

```text
original source files
internal routes
comments
function names
development configuration
API endpoints
```

Potentially sensitive information should be reported without reproducing the secret itself.

---

# 7. Environment Files

Common files:

```text
.env
.env.local
.env.production
.env.development
.env.backup
```

Also investigate accidental exposure through:

```text
/.env
/config
/config.json
/settings.json
```

If a file is accessible:

```text
GET /.env
```

Do not copy the entire contents into the report.

Record:

```text
File:
Exposure:
Secret type:
Redacted evidence:
Potential impact:
```

Example:

```text
/.env

Contains:
[REDACTED] database credential

Risk:
High
```

---

# 8. Backup Files

Look for accidentally exposed artifacts such as:

```text
backup.zip
backup.tar
database.sql
site.zip
config.bak
config.old
index.php~
```

Also look for:

```text
.old
.bak
.backup
.tmp
.save
```

Do not download large or sensitive backups unnecessarily.

If exposure is confirmed, stop after obtaining enough evidence to establish the issue.

---

# 9. Configuration Files

Potentially exposed files include:

```text
config.json
settings.json
application.json
docker-compose.yml
```

Inspect for:

```text
database hosts
internal services
credentials
API endpoints
cloud configuration
debug settings
```

Separate:

```text
configuration
```

from:

```text
secret
```

Not every configuration value is sensitive.

---

# 10. API Response Disclosure

Inspect API responses for unnecessary fields.

Example:

```json
{
  "id": 42,
  "name": "Test User",
  "email": "test@example.com",
  "internalNotes": "...",
  "apiToken": "[REDACTED]"
}
```

Potential issues include:

```text
tokens
password hashes
private metadata
internal identifiers
debug fields
administrative information
```

Determine whether the frontend actually requires the field.

---

# 11. HTTP Headers

Inspect response headers:

```text
Server
X-Powered-By
Via
X-Debug
X-Request-ID
Set-Cookie
```

Information disclosure may occur when headers unnecessarily reveal:

```text
framework versions
server versions
internal infrastructure
debug state
internal hostnames
```

Version disclosure is usually lower risk unless it contributes to a larger attack path.

---

# 12. Error Messages

Send controlled invalid requests.

Look for:

```text
stack traces
absolute file paths
database errors
framework information
internal IP addresses
service names
source code
configuration
```

Example:

```text
500 Internal Server Error

Traceback:
...
/app/services/database.py
...
```

This may reveal internal implementation details.

Do not intentionally cause excessive errors.

---

# 13. Git and Repository Exposure

For authorized applications, check whether web-accessible repository artifacts exist:

```text
.git/
.gitignore
.git/config
.git/HEAD
```

Other common artifacts:

```text
.env
Dockerfile
docker-compose.yml
package-lock.json
requirements.txt
```

A dependency file is usually not a secret.

A repository containing historical credentials can be much more serious.

Do not clone or dump repositories that you are not authorized to access.

---

# 14. Git History

For your own repository, inspect history for accidentally committed secrets.

Useful local checks:

```bash
git log --all --oneline
git grep -n "api_key"
git grep -n "secret"
git grep -n "password"
```

For automated detection, use secret-scanning tools in your own repository.

The objective is:

```text
Find
  ↓
Identify
  ↓
Revoke/rotate
  ↓
Remove from history where appropriate
```

Simply deleting the secret from the latest commit does not necessarily remove it from Git history.

---

# 15. Common Secret Patterns

Search for conceptual patterns such as:

```text
API_KEY=
SECRET=
TOKEN=
PASSWORD=
DATABASE_URL=
PRIVATE_KEY=
CLIENT_SECRET=
```

Also identify structured credentials:

```text
Authorization: Bearer ...
```

```text
"access_token": "..."
```

```text
"client_secret": "..."
```

Treat pattern matches as candidates requiring validation.

---

# 16. JWT Detection

JWTs commonly have a structure resembling:

```text
xxxxx.yyyyy.zzzzz
```

If a JWT is exposed:

Do not use it to access accounts.

Instead record:

```text
Location:
Token type:
Where exposed:
Whether it appears sensitive:
Expiration if safely observable:
```

Redact it:

```text
eyJ...[REDACTED]
```

---

# 17. Database Connection Strings

Potential patterns:

```text
postgresql://...
mysql://...
mongodb://...
redis://...
```

If discovered:

```text
Do not connect to the database.
Do not validate the credentials.
Do not enumerate the database.
```

Record only:

```text
Database type
Exposure location
Credential presence
Redacted evidence
Potential impact
```

---

# 18. Cloud Credential Detection

Potential cloud credentials may appear in:

```text
JavaScript
configuration
environment files
repositories
CI/CD logs
build artifacts
```

If found:

```text
DO NOT attempt cloud login.
```

Instead:

```text
Identify provider
Identify credential type
Redact value
Assess likely privilege if known
Recommend immediate rotation
```

---

# 19. CI/CD Secret Exposure

For authorized projects inspect:

```text
GitHub Actions
GitLab CI
Jenkins
Docker builds
deployment logs
CI artifacts
```

Look for accidental logging of:

```text
API keys
tokens
environment variables
cloud credentials
deployment credentials
```

Important:

```text
Secret stored securely → good
Secret printed in build log → exposure
```

---

# 20. Docker and Container Artifacts

Inspect authorized container configurations for accidental secrets.

Examples:

```text
Dockerfile
docker-compose.yml
.env
environment variables
build arguments
```

Avoid putting secrets directly into Docker images.

Prefer:

```text
runtime secret injection
secret managers
environment-specific secret stores
```

---

# 21. Client-Side vs Server-Side Secrets

Important distinction:

### Public/client-side

```text
Frontend configuration
Public API identifier
Browser-safe configuration
```

May be intentionally exposed.

### Server-side

```text
Database passwords
Private API keys
Cloud credentials
Signing keys
Admin tokens
```

Should generally never be shipped to the browser.

---

# 22. Secret Validation Without Using the Secret

When a candidate secret is found:

```text
1. Identify the source.
2. Determine the secret type.
3. Redact the value.
4. Determine whether it is intended to be public.
5. Determine potential privilege from context.
6. Check whether the application exposes it to unauthorized users.
7. Stop.
```

Do not perform:

```text
credential validation
login attempts
API calls using the secret
cloud enumeration
database connection
```

unless separately authorized and explicitly required by the testing scope.

---

# 23. False Positive Reduction

Not every string resembling a secret is a secret.

Check:

```text
Is it a placeholder?
Is it example data?
Is it publicly documented?
Is it a test credential?
Is it intentionally public?
Does the application actually expose it?
```

Examples:

```text
YOUR_API_KEY_HERE
example-secret
test-token
```

are usually not vulnerabilities by themselves.

---

# 24. Risk Assessment

Consider:

```text
Secret type
Exposure location
Who can access it
Whether it is production
Privilege level
Expiration
Scope
Potential impact
```

Example:

```text
Public JavaScript
      ↓
Production database password
      ↓
Critical exposure
```

Versus:

```text
Public JavaScript
      ↓
Public API identifier
      ↓
Likely informational
```

---

# 25. Evidence Format

Never include the complete secret.

Use:

```text
Finding:
Exposed API Credential

Location:
/assets/app.js

Type:
API credential

Evidence:
Credential-like value present in publicly accessible
JavaScript.

Redacted value:
abc123...[REDACTED]

Exposure:
Unauthenticated

Potential Impact:
Unauthorized use of the associated service.

Confidence:
High

Recommendation:
Revoke/rotate the credential and remove it from
client-accessible assets.
```

---

# 26. Reporting Template

```text
## Information Disclosure

### Target
<authorized target>

### Location
<URL / file / endpoint>

### Information Type
<API key / token / config / debug data>

### Exposure
<public / authenticated / role-specific>

### Evidence
<redacted evidence>

### Potential Impact
<impact>

### Confidence
<Low / Medium / High / Confirmed>

### Recommendation
<remediation>

### Verification
<how to verify the fix>
```

---

# 27. Automated Agent Behavior

The agent should:

1. Crawl authorized web assets.
2. Identify JavaScript and configuration files.
3. Inspect API responses.
4. Search for secret-like patterns.
5. Detect source maps.
6. Identify backup/configuration artifacts.
7. Analyze error messages.
8. Detect credential-like values.
9. Redact secrets immediately.
10. Classify exposure.
11. Correlate with application context.
12. Assign confidence.
13. Produce a remediation report.

The agent should never:

- Use discovered credentials
- Log into accounts with discovered tokens
- Access databases using exposed connection strings
- Enumerate cloud accounts
- Exfiltrate private data
- Publish secrets
- Store raw secrets in logs
- Include raw secrets in reports

---

# 28. Core Principle

> **A discovered secret is evidence of exposure. Detect it, and report**