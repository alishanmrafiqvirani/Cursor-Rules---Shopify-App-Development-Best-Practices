---
description: Shopify App Security & Development Best Practices
globs:
  [
    "**/*.js",
    "**/*.ts",
    "**/*.jsx",
    "**/*.tsx",
    "**/*.py",
    "**/*.rb",
    "**/*.php",
    "**/*.go",
    "**/*.java",
    "**/*.cs",
    "**/*.cpp",
    "**/*.c",
    "**/*.sql",
    "**/*.yml",
    "**/*.yaml",
    "**/*.json",
    "**/*.env.example",
    "**/.gitignore",
  ]
alwaysApply: false
---

# Shopify App Security & Development Best Practices

## Source Control & Infrastructure

### 1. Source Control Requirements

- **ALWAYS** use source control (Git) for all application code
- **IMPLEMENT** automated deployment and build processes
- **ELIMINATE** manual deployment steps that introduce human error
- **USE** infrastructure as code (CloudFormation, Terraform, buildpacks)
- **PREFER** low-config deployment environments like Vercel for built-in best practices
- **WHY**: Manual processes increase likelihood of security vulnerabilities and deployment failures

### 2. Environment Variable Security

- **NEVER** store secrets (API keys, passwords, tokens) in codebase
- **USE** `.env` files for local development (add to `.gitignore`)
- **CREATE** `.env.example` to document required variables
- **SEPARATE** development and production service instances completely
- **CONFIGURE** Shopify CLI for automatic `.env` setup
- **WHY**: Prevents credential exposure and accidental data contamination

#### Environment Variable Best Practices

```bash
# .env (local development only - never commit)
SHOPIFY_API_KEY=your_dev_key
DATABASE_URL=postgresql://localhost/myapp_dev

# .env.example (commit this)
SHOPIFY_API_KEY=your_shopify_api_key
DATABASE_URL=your_database_connection_string
```

## Input Validation & XSS Prevention

### 3. User Input Security

- **NEVER** trust user-generated content without validation
- **AVOID** direct DOM manipulation with user input
- **ENCODE** all user input before rendering to prevent XSS attacks
- **REMOVE** HTML tags in backend before database storage
- **VALIDATE** input on both client and server sides
- **WHY**: Prevents Cross-Site Scripting and injection attacks

#### XSS Prevention Patterns

```javascript
// React - NEVER use dangerouslySetInnerHTML with user content
function UserContent({ userText }) {
  return <div>{userText}</div>; // Automatically escaped
}

// jQuery - Use .text() for safe content insertion
$("#content").text(userInput); // Safe
$("#content").html(userInput); // DANGEROUS - avoid

// Backend validation before storage
function sanitizeInput(input) {
  return input.replace(/<[^>]*>/g, ""); // Strip HTML tags
}
```

### 4. Multi-Tenant Security Requirements

- **SCOPE** all database queries to tenant ID
- **VALIDATE** tenant ownership before any data operation
- **IMPLEMENT** tenant isolation at database level
- **USE** ORM helpers for automatic tenant scoping
- **TEST** cross-tenant data leaks with automated tests
- **WHY**: Prevents data breaches between different merchant accounts

#### Multi-Tenant Query Pattern

```javascript
// Always include tenant scoping
async function getOrders(tenantId, userId) {
  return await Order.findAll({
    where: {
      tenant_id: tenantId, // MANDATORY: Always scope by tenant
      user_id: userId,
    },
  });
}

// Validate tenant ownership before operations
async function deleteResource(resourceId, tenantId, userId) {
  const resource = await Resource.findOne({
    where: { id: resourceId, tenant_id: tenantId },
  });

  if (!resource) {
    throw new Error("Resource not found or access denied");
  }

  return await resource.destroy();
}
```

## Dependency & Code Security

### 5. Dependency Management

- **AUDIT** dependencies regularly with `npm audit` or equivalent
- **USE** enhanced tools like Snyk for comprehensive vulnerability scanning
- **IMPLEMENT** dependency checking in CI/CD pipeline
- **UPDATE** dependencies promptly when security patches are available
- **AUTOMATE** dependency updates with tools like Dependabot
- **WHY**: Third-party vulnerabilities are a common attack vector

#### Dependency Audit Commands

```bash
# JavaScript/Node.js
npm audit
npm audit fix

# Ruby
bundle audit

# Python
safety check

# Automated CI/CD integration
# .github/workflows/security.yml
- name: Run security audit
  run: npm audit --audit-level moderate
```

### 6. Static Analysis Integration

- **IMPLEMENT** static analysis tools in development workflow
- **SCAN** for SQL injection, XSS, and other common vulnerabilities
- **REVIEW** flagged code areas manually for complex security issues
- **INTEGRATE** analysis tools into CI/CD pipeline
- **ADDRESS** code smells that indicate security risks
- **WHY**: Automated scanning catches common security flaws early

### 7. Automated Testing for Security

- **WRITE** automated tests for permission checks
- **TEST** multi-tenant data isolation
- **VERIFY** input validation and sanitization
- **IMPLEMENT** security regression tests
- **USE** testing frameworks like Jest for comprehensive coverage
- **WHY**: Security bugs are difficult to test manually and high-risk

#### Security Test Examples

```javascript
// Test tenant isolation
describe("Tenant Security", () => {
  test("should not access other tenant data", async () => {
    const tenant1Data = await getOrders(tenant1Id, userId);
    const tenant2Data = await getOrders(tenant2Id, userId);

    expect(tenant1Data).not.toContain(tenant2Data[0]);
  });
});

// Test input validation
describe("Input Validation", () => {
  test("should sanitize malicious script tags", () => {
    const maliciousInput = '<script>alert("xss")</script>Hello';
    const sanitized = sanitizeInput(maliciousInput);

    expect(sanitized).toBe("Hello");
    expect(sanitized).not.toContain("<script>");
  });
});
```

## Authentication & Encryption

### 8. Password Security

- **NEVER** store passwords in plaintext
- **USE** latest salted hashing algorithms (bcrypt, Argon2)
- **MIGRATE** from older algorithms (MD5, SHA1) to modern alternatives
- **CHECK** logs and error tracking for accidental password exposure
- **FOLLOW** OWASP password hashing recommendations
- **WHY**: Password breaches can compromise entire user accounts

#### Secure Password Hashing

```javascript
// Node.js with bcrypt
const bcrypt = require("bcrypt");

async function hashPassword(password) {
  const saltRounds = 12; // OWASP recommended minimum
  return await bcrypt.hash(password, saltRounds);
}

async function verifyPassword(password, hash) {
  return await bcrypt.compare(password, hash);
}
```

### 9. TLS/SSL Implementation

- **ENFORCE** HTTPS for all connections
- **USE** TLS certificates for all data transmission
- **IMPLEMENT** TLS between application servers and CDNs
- **LEVERAGE** free certificates from Let's Encrypt, AWS Certificate Manager, Cloudflare
- **VERIFY** TLS configuration is up-to-date and secure
- **WHY**: Prevents data interception and tampering

### 10. Key Rotation Strategy

- **ROTATE** API keys and secrets regularly
- **IMPLEMENT** automated key rotation processes
- **MONITOR** key usage and access patterns
- **REVOKE** compromised keys immediately
- **DOCUMENT** key rotation procedures for team members
- **WHY**: Prevents security breaches from compromised credentials

## Development Process Security

### 11. Code Review Requirements

- **MANDATE** peer code reviews for all changes
- **FOCUS** security considerations in review process
- **USE** pull request workflows in GitHub/GitLab/Bitbucket
- **EDUCATE** team members on security best practices
- **DOCUMENT** security review checklist
- **WHY**: Multiple eyes catch security issues human analysis misses

#### Security Code Review Checklist

```markdown
## Security Review Checklist

- [ ] Are user inputs properly validated and sanitized?
- [ ] Are database queries scoped to correct tenant?
- [ ] Are secrets removed from code and logs?
- [ ] Are new dependencies security audited?
- [ ] Are authentication checks properly implemented?
- [ ] Are error messages not exposing sensitive information?
- [ ] Are file uploads restricted and validated?
- [ ] Are API endpoints properly authorized?
```

## Infrastructure Security

### 12. Production Security Monitoring

- **MONITOR** application logs for security events
- **IMPLEMENT** intrusion detection systems
- **SET UP** automated alerting for suspicious activities
- **CONDUCT** regular security assessments
- **MAINTAIN** incident response procedures
- **WHY**: Early detection prevents major security breaches

### 13. Data Protection Standards

- **ENCRYPT** sensitive data at rest and in transit
- **IMPLEMENT** proper backup security measures
- **COMPLY** with data protection regulations (GDPR, CCPA)
- **LIMIT** data retention to necessary periods
- **AUDIT** data access and processing activities
- **WHY**: Legal compliance and customer trust protection
