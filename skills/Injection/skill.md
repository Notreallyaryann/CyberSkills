# Injection Analysis Skill

## Purpose

This skill teaches the agent how to identify, analyze, and safely validate injection vulnerabilities in authorized applications, APIs, labs, and CTF environments.

The goal is **not to blindly exploit inputs**.

The goal is to determine:

> "Does attacker-controlled input reach an interpreter or query engine in an unsafe way?"

Supported injection categories:

- SQL Injection
- NoSQL Injection
- OS Command Injection
- Server-Side Template Injection (SSTI)
- LDAP Injection
- XPath Injection
- Expression/Template Language Injection
- Basic reflected input validation issues

---

# 1. Scope and Safety


Never perform:

- Data extraction from real users
- Destructive SQL queries
- File deletion
- OS command execution that changes the system
- Reverse shells
- Credential theft
- Authentication bypass against unauthorized systems
- Database dumping
- Secrets exfiltration
- Denial-of-service testing
- Persistence

Use harmless canaries and controlled test data.

---

# 2. Injection Analysis Workflow

Follow this process:

```text
Target
  ↓
Identify Input Points
  ↓
Classify Input Context
  ↓
Establish Baseline
  ↓
Send Harmless Canary
  ↓
Compare Response
  ↓
Look for Multiple Signals
  ↓
Corroborate Finding
  ↓
Determine Confidence
  ↓
Document Evidence
  ↓
Recommend Remediation
```

Do not report an injection vulnerability based on a single unusual response always try to verify it using another payload and if we get that vulneb again then only report it 

---

# 3. Identify Input Points

Look for attacker-controlled input in:

### URL Parameters

```text
?id=123
?search=test
?name=alice
?sort=price
```

### Path Parameters

```text
/users/123
/products/42
/orders/1001
```

### POST/PUT/PATCH JSON

```json
{
  "username": "aryan",
  "search": "phone"
}
```

### Forms

Examples:

```text
login
search
registration
feedback
contact
checkout
```

### Headers

Potentially interesting headers:

```text
User-Agent
Referer
X-Forwarded-For
X-Original-URL
X-Forwarded-Host
```

Only test headers when the application actually uses them.

### Cookies

Identify application-controlled cookies and determine whether their values influence server-side processing.

### GraphQL

Inspect:

```text
query arguments
mutation arguments
variables
filters
sorting fields
```

---

# 4. Establish a Baseline

Before testing an input, send a normal request.

Record:

```text
URL
HTTP method
parameter
normal value
status code
response size
response body
response time
redirect behavior
```

Example:

```text
GET /products?id=10
```

Baseline:

```text
Status: 200
Response size: 14 KB
Response time: 120 ms
```

The baseline is important because injection testing depends on comparing behavior.

---

# 5. SQL Injection Analysis

## Goal

Determine whether user-controlled input is being inserted into a database query without proper parameterization.

Potential inputs:

```text
?id=
?user=
?search=
?sort=
?category=
```

JSON:

```json
{
  "id": "10"
}
```

---

## SQLi Indicators

Look for:

- Database error messages
- SQL syntax errors
- Unexpected query behavior
- Boolean-dependent response differences
- Unexpected record counts
- Consistent response changes
- Controlled timing differences in authorized labs

Possible database indicators:

```text
SQL syntax
PostgreSQL
MySQL
SQLite
Microsoft SQL Server
Oracle
```

An error message alone is not enough to confirm SQL injection.

---

## Safe SQLi Validation

Start with harmless input changes.

For example:

```text
normal input
special-character input
boolean-style input
```

Compare:

```text
status code
response length
returned records
error messages
response structure
```

Do not immediately attempt extraction.

---

## Example Analysis

Baseline:

```text
GET /product?id=10
```

Response:

```text
200
Product returned
```

Test:

```text
GET /product?id=<harmless test value>
```

If the application's response consistently changes in a way that indicates the backend is interpreting the input as part of a query, investigate further in the controlled environment.

Confidence should increase only when multiple independent signals agree.

---

# 6. SQLi Error Analysis

Potential evidence:

```text
syntax error near...
database error...
query failed...
invalid SQL...
```

Do not automatically classify this as confirmed SQL injection.

Instead ask:

```text
Did the input cause the error?
Is the behavior reproducible?
Does another controlled input produce the expected baseline?
Is the response behavior consistent?
```

---

# 7. Boolean-Based Analysis

In an authorized lab, compare two logically different test conditions.

Conceptually:

```text
Condition A → expected normal behavior
Condition B → expected changed behavior
```

Observe:

```text
response body
record count
status
application behavior
```

A repeatable difference strengthens confidence.

---

# 8. Time-Based Analysis

Do not perform repeated high-delay requests.

Compare several baseline requests before concluding that timing indicates injection.

Network latency can produce false positives.

---

# 9. NoSQL Injection

Common environments:

```text
MongoDB
Redis-backed applications
Document databases
JSON query systems
```

Look for inputs such as:

```json
{
  "username": "...",
  "password": "..."
}
```

or:

```json
{
  "filter": {}
}
```

Potential indicators include:

- Unexpected object types being accepted
- Query operators being interpreted unexpectedly
- Authentication behavior changing when input structure changes
- Filters behaving differently when strings become objects

---

## Safe NoSQL Testing

Compare:

```text
normal string
unexpected JSON type
controlled boolean condition
```

Example concept:

```json
{
  "username": "test"
}
```

versus a controlled structured input.

Do not attempt to dump databases or bypass authentication on real systems.

---

# 10. OS Command Injection

Look for application features that execute operating-system commands like (whoami , id ,hostname,pwd etc).

Examples:

```text
ping tool
DNS lookup
file conversion
image processing
system diagnostics
network utilities
```

Potential input:

```text
host=
filename=
target=
command=
```

---

## Safe Validation

Only in an authorized lab, use a harmless canary that demonstrates command interpretation without modifying the system.

Examples of safe concepts:

```text
echo
```

or another environment-appropriate non-destructive command.

The goal is to establish:

```text
Input
 ↓
Application
 ↓
Command interpreter
```

Do NOT:

```text
delete files
create persistence
download payloads
spawn shells
modify users
change permissions
access credentials
exfiltrate data
```

---

# 11. Server-Side Template Injection (SSTI)

Look for user input appearing inside server-rendered templates.

Common template technologies include:

```text
Jinja2
Twig
Handlebars
Freemarker
Velocity
Thymeleaf
```

Potential locations:

```text
name
email templates
PDF generation
HTML generation
notifications
custom messages
page titles
```

---

## Safe SSTI Detection

In a controlled environment, use harmless arithmetic expressions.

Example concept:

```text
{{7*7}}
```

If the application evaluates the expression and returns:

```text
49
```

instead of treating it as literal text, investigate the template context.

Do not progress from SSTI detection into operating-system command execution.

---

# 12. LDAP Injection

LDAP-backed applications may use user input for:

```text
username search
directory lookup
employee search
group lookup
authentication
```

Look for:

```text
username=
search=
email=
uid=
```

Analyze how special LDAP characters affect application behavior.

Potential indicators:

- LDAP syntax errors
- Unexpected search results
- Authentication behavior changes
- Filter manipulation

Use only controlled directory data.

---

# 13. XPath Injection

Applications using XML may construct XPath queries using user input.

Potential locations:

```text
XML search
authentication
document lookup
catalog search
```

Indicators:

- XPath parser errors
- Unexpected XML records
- Boolean-dependent behavior
- Changed query results

Again:

```text
normal input
 ↓
controlled test input
 ↓
compare behavior
```

Do not extract unrelated XML data.

---

# 14. Expression Language Injection

Some frameworks evaluate expressions inside:

```text
templates
configuration
message processing
business rules
```

Look for evidence that user input is interpreted rather than treated as plain text.

Start with harmless expressions and determine:

```text
Is evaluation occurring?
Which engine is being used?
Is the input isolated?
```

Do not attempt OS-level execution.

---

# 15. Reflected Input Analysis

Injection analysis should also determine whether user input is reflected into:

```text
HTML
JavaScript
JSON
HTTP headers
SQL queries
templates
XML
shell commands
```

For example:

```text
/search?q=test
```

If `test` appears in the response, determine its context.

Important question:

> Where does the input land?

Possible contexts:

```text
HTML text
HTML attribute
JavaScript string
JSON
URL
SQL query
template
command
```

Context determines the appropriate security analysis.

---

# 16. Response Comparison

For every test, compare:

```text
Status code
Response size
Response body
Headers
Redirects
Cookies
Response time
Returned records
Error messages
```

Example:

```text
Baseline:
200
12.4 KB
120 ms

Test:
500
3.1 KB
125 ms
Database syntax error
```

This is a useful signal, but still requires corroboration.

---

# 17. False Positive Reduction

Never report:

```text
"Input caused 500 → SQL injection"
```

Instead require multiple signals.

Example confidence model:

```text
+1 suspicious error
+1 reproducible behavior
+1 input-dependent response
+1 backend technology correlation
+1 controlled validation
```

Then classify:

```text
Low confidence
Medium confidence
High confidence
Confirmed
```

A confirmed finding should have reproducible evidence.

---

# 18. Technology Correlation

Use other reconnaissance information to improve analysis.

Example:

```text
Technology:
MongoDB

Input:
JSON filter

Behavior:
Unexpected operator-like object changes result

Conclusion:
Investigate possible NoSQL injection
```

Another:

```text
Technology:
Jinja2

Input:
User-controlled template field

Behavior:
Harmless arithmetic expression evaluated

Conclusion:
High-confidence SSTI
```

Technology alone does not prove vulnerability.

---

# 19. Tools

Useful tools include:

```text
Burp Suite
curl
Browser DevTools
Postman
httpx
```

Use Burp to:

- Intercept requests
- Modify parameters
- Replay requests
- Compare responses
- Inspect headers
- Analyze request/response behavior

Use curl for reproducible requests.

Example:

```bash
curl -i "https://authorized-target.example/search?q=test"
```

Always replace the target with an authorized environment.

---

# 20. Evidence Collection

For every potential finding record:

```text
Target:
Endpoint:
HTTP Method:
Parameter:
Input Context:
Baseline:
Test:
Observed Behavior:
Response Difference:
Backend Technology:
Evidence:
Confidence:
Impact:
Recommended Fix:
```

Example:

```text
Finding:
Potential SQL Injection

Endpoint:
/products

Method:
GET

Parameter:
id

Baseline:
id=10 → HTTP 200

Observed:
Controlled input consistently changes database-related
error behavior.

Evidence:
Reproducible response difference and database error.

Confidence:
High

Impact:
Potential unauthorized database query manipulation.

Recommendation:
Use parameterized queries / prepared statements.
```

---

# 21. Remediation Mapping

For confirmed injection vulnerabilities recommend:

### SQL Injection

```text
Parameterized queries
Prepared statements
ORM-safe query APIs
Input validation
Least-privileged database accounts
```

### NoSQL Injection

```text
Strict schema validation
Type validation
Allowlisted operators
Safe query construction
```

### Command Injection

```text
Avoid shell execution
Use safe library APIs
Allowlist arguments
Strict input validation
Least privilege
```

### SSTI

```text
Do not treat user input as templates
Use safe rendering APIs
Separate data from templates
Sandbox template execution where appropriate
```

### LDAP Injection

```text
LDAP escaping
Parameterized LDAP APIs
Strict input validation
```

### XPath Injection

```text
Parameterized XPath mechanisms
Safe XML processing
Strict input validation
```

---

# 22. Reporting Format

Use this structure:

```text
## Injection Analysis

### Target
<authorized target>

### Input Point
<endpoint + parameter>

### Injection Type
<SQL / NoSQL / Command / SSTI / LDAP / XPath>

### Baseline
<normal behavior>

### Test
<safe test description>

### Observed Behavior
<response difference>

### Evidence
<reproducible evidence>

### Confidence
<Low / Medium / High / Confirmed>

### Impact
<security impact>

### Recommendation
<remediation>

### Verification
<how to verify the fix>
```

---

# 23. Decision Tree

When encountering an input:

```text
Is the input attacker-controlled?
        |
       Yes
        ↓
Where does it go?
        |
        ├── Database → SQL/NoSQL analysis
        |
        ├── Shell/process → Command injection analysis
        |
        ├── Template → SSTI analysis
        |
        ├── LDAP → LDAP analysis
        |
        ├── XML/XPath → XPath analysis
        |
        └── Unknown → Identify backend context
```

Then:

```text
Baseline
   ↓
Harmless test
   ↓
Compare response
   ↓
Reproduce
   ↓
Corroborate
   ↓
Assign confidence
   ↓
Report
```

---

# 24. Agent Behavior

The agent should:

1. Identify all user-controlled inputs.
2. Determine the likely processing context.
3. Establish a baseline.
4. Use harmless test inputs.
5. Compare responses systematically.
6. Correlate behavior with detected technologies.
7. Repeat suspicious tests for reproducibility.
8. Reduce false positives.
9. Record evidence.
10. Recommend remediation.

The agent should **not**:

- Dump databases
- Extract secrets
- Delete files
- Execute reverse shells
- Modify production systems
- Steal credentials
- Bypass authentication
- Exfiltrate data
- Perform destructive queries
- Launch denial-of-service attacks

---

# Core Principle

> **Find where attacker-controlled input crosses a security boundary, determine whether the backend interprets it as code/query/template syntax, validate the behavior safely, and report evidence—not assumptions.**