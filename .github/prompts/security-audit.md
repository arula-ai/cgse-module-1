# Security Audit

Perform a comprehensive security audit on the specified service file.

## Instructions

Act as a senior application security engineer. Analyze the provided service for security vulnerabilities following OWASP Top 10 (2021) guidelines.

For each vulnerability found, document:
1. **Line number** - Exact location in the code
2. **OWASP Category** - e.g., A03:2021-Injection, A07:2021-Auth Failures
3. **Severity** - Critical, High, Medium, or Low
4. **Description** - What the vulnerability is and why it's dangerous
5. **Attack Scenario** - How an attacker could exploit this
6. **Remediation** - Specific code fix recommendation

## Output Format

Return findings as a markdown table:

| # | Line | OWASP Category | Severity | Description | Remediation |
|---|------|----------------|----------|-------------|-------------|
| 1 | ... | ... | ... | ... | ... |

After the table, provide:
- **Priority Order**: List vulnerabilities by remediation priority
- **Quick Wins**: Easy fixes that should be done immediately
- **Architecture Concerns**: Any systemic issues requiring larger changes

## Focus Areas

- Injection vulnerabilities (SQL, NoSQL, Command, LDAP)
- Authentication and session management flaws
- Sensitive data exposure (logging, storage, transmission)
- Access control issues (mass assignment, privilege escalation)
- Security misconfigurations
- Input validation gaps

## Constraints

- Do not modify any code - only analyze and document
- Focus on realistic, exploitable vulnerabilities
- Consider the technology stack context (Angular/Spring Boot)
- Prioritize findings by actual risk, not theoretical concerns
