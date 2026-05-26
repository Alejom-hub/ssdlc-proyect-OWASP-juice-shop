# Evidence Hardcoded Secrets Report

## Overview
This document summarizes the results of two Semgrep scans:

1. **Old insecure version**: `reports/insecurity_vieja.ts`
2. **Current modified version**: files scanned from the current working tree

---

## Old report: `reports/reporte_insecurity_vieja.json`

- **Target scanned**: `reports/insecurity_vieja.ts`
- **Findings**: 6

### Findings

1. **javascript.lang.security.audit.hardcoded-hmac-key.hardcoded-hmac-key**
   - File: `reports/insecurity_vieja.ts`
   - Line: 43
   - Issue: Hardcoded HMAC key detected. Secrets should not be stored in source code.

2. **javascript.express.security.audit.express-jwt-not-revoked.express-jwt-not-revoked**
   - File: `reports/insecurity_vieja.ts`
   - Line: 53
   - Issue: `express-jwt` is configured without token revocation. Revoked tokens could still be accepted.

3. **javascript.express.security.audit.express-jwt-not-revoked.express-jwt-not-revoked**
   - File: `reports/insecurity_vieja.ts`
   - Line: 54
   - Issue: `express-jwt` is configured without token revocation. Revoked tokens could still be accepted.

4. **javascript.jsonwebtoken.security.audit.jwt-exposed-data.jwt-exposed-data**
   - File: `reports/insecurity_vieja.ts`
   - Line: 55
   - Issue: Sensitive data may be exposed through JWT payload.

5. **javascript.jsonwebtoken.security.jwt-hardcode.hardcoded-jwt-secret**
   - File: `reports/insecurity_vieja.ts`
   - Line: 55
   - Issue: Hard-coded JWT secret detected. Secrets must be provided from a secure source such as environment variables.

6. **javascript.lang.security.audit.hardcoded-hmac-key.hardcoded-hmac-key**
   - File: `reports/insecurity_vieja.ts`
   - Line: 166
   - Issue: Hardcoded HMAC key detected in `deluxeToken()` generation.

### Interpretation
The old version of `insecurity.ts` contained embedded secrets and hardcoded credentials:
- A private key constant was stored directly in the source.
- An HMAC secret string was hardcoded in the source.
- JWT signing used a hardcoded private key instead of secure environment-sourced key material.
- `express-jwt` was not configured with token revocation.

---

## Current report: `reports/reporte_hardcoded_secrets.json`

- **Targets scanned**:
  - `.env.example`
  - `.gitignore`
  - `README.md`
  - `juice-shop/lib/insecurity.ts`
- **Findings**: 0

### Result
No hardcoded secrets were detected by Semgrep in the scanned files.

### Interpretation
The current modified version appears to have removed embedded secret literals and relies on environment variables or external key material instead of hardcoded credentials.

---

## Commands used to generate these reports

```bash
cd /workspaces/ssdlc-proyect-OWASP-juice-shop
semgrep --config=auto reports/insecurity_vieja.ts --json | python -m json.tool > reports/reporte_insecurity_vieja.json
semgrep --config=auto juice-shop/lib/insecurity.ts README.md .env.example .gitignore --json | python -m json.tool > reports/reporte_hardcoded_secrets.json
```
