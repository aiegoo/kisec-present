
# 🛡️ Security Assessment Report  
## GraphQL Reconnaissance & Attack Surface Enumeration  
**Project:** KISEC Project 1  
**Author:** @aiegoo  이병남 

**Date:** April9  2025  

---

## 1. Summary

This report summarizes the findings of a reconnaissance and enumeration assessment conducted on a GraphQL endpoint deployed on port `5013`. Through the use of open-source security tools and custom payloads, we discovered several security risks associated with an improperly secured GraphQL API and WebSocket-based subscription endpoint.

---

## 2. Objectives

- Identify exposed GraphQL endpoints
- Perform schema introspection to enumerate data types and operations
- Detect WebSocket upgrade mechanisms tied to GraphQL subscriptions
- Determine if access control or query restrictions are enforced
- Map the attack surface based on response behavior

---

## 3. Methodology

### 3.1 Tools Used

- `nmap` with `http-grep` and `-sV` service detection  
- `curl` for manual GraphQL query injection  
- `ffuf` for endpoint discovery  
- Manual inspection of WebSocket headers and upgrade mechanisms

### 3.2 Test Environment

- Local instance exposed via `ngrok` for remote enumeration  
- GraphQL served via Werkzeug on TCP `5013`  
- Endpoint `/graphql` and WebSocket `/subscriptions`

---

## 4. Findings

### 4.1 GraphQL Introspection Enabled ✅

The following introspection query returned schema metadata:

```graphql
{
  __schema {
    types {
      name
    }
  }
}
```

Response confirms the presence of:
- Custom object types: `PasteObject`, `UserObject`, `AuditObject`, etc.
- Core types: `Query`, `Mutation`, `Subscription`

### 4.2 WebSocket Protocol Exposure ✅

Using the following headers, the server initiated a WebSocket handshake:

```
GET /subscriptions HTTP/1.1
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Version: 13
```

Response:
```
HTTP/1.1 101 Switching Protocols
Sec-WebSocket-Accept: <key>
```

This exposes real-time subscription mechanics without authentication barriers, potentially leaking live data to unauthorized clients.

### 4.3 Weak Input Validation ❌

Endpoint `/graphql` returns:

```json
{
  "errors": [
    {
      "message": "Must provide query string"
    }
  ]
}
```

This suggests:
- Open POST access without rate-limiting
- No protection from introspection or fuzz queries
- Verbose error responses (useful for attackers)

---

## 5. Risk Evaluation

| Risk                               | Impact   | Likelihood | Severity |
|------------------------------------|----------|------------|----------|
| Introspection Enabled              | Medium   | High       | **High** |
| Unauthenticated WebSocket Access  | High     | Medium     | **High** |
| Error Disclosure (Verbose Output) | Medium   | High       | **Medium** |
| Schema Enumeration via Payloads   | Medium   | High       | **High** |

---

## 6. Recommendations

1. **Disable Introspection in Production**
   - Configure GraphQL server to block `__schema` queries

2. **Authenticate WebSocket Connections**
   - Require session tokens or JWTs before upgrade

3. **Rate-Limit and Monitor API Access**
   - Protect `/graphql` with WAF rules and logging

4. **Suppress Detailed Error Messages**
   - Show generic error codes instead of stack trace-level feedback

5. **Validate Input at API Layer**
   - Implement strict schemas and deny invalid queries early

---

## 7. Conclusion

The GraphQL server exposed in this test is susceptible to enumeration, live subscription hijacking, and schema leakage. This reconnaissance phase demonstrates that without proper hardening, GraphQL can become a significant attack surface in modern applications.

---

## 8. Appendix

- 🔍 Payloads used: See `https://gist.github.com/aiegoo/2ac371f792996dccd321f9fff93a52b9` as a part of secret gist; request for access.  
- 📊 Slides: See `Gamma Pitch Deck` in `/slides/`  
- 📎 Screenshots: See `/images/` folder for visual proof of tests

---

## 9. References

- OWASP GraphQL Security Cheat Sheet: https://cheatsheetseries.owasp.org/cheatsheets/GraphQL_Security_Cheat_Sheet.html  
- WebSocket Security: https://portswigger.net/web-security/websockets  
- Nmap Scripts: https://nmap.org/nsedoc/scripts/http-grep.html



---
