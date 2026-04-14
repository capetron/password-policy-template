# Password Policy Template

An enterprise password policy template fully aligned with NIST SP 800-63b (Digital Identity Guidelines, 2024 revision). This policy replaces outdated password practices (forced rotation, arbitrary complexity) with evidence-based authentication requirements that actually improve security.

Use this template as a standalone password policy or integrate it into your broader information security policy framework. Suitable for compliance with NIST 800-171, CMMC, HIPAA, SOC 2, PCI DSS, and ISO 27001.

## Table of Contents

- [Why This Policy Matters](#why-this-policy-matters)
- [Policy Document](#policy-document)
- [NIST 800-63b Alignment](#nist-800-63b-alignment)
- [Password Requirements Summary](#password-requirements-summary)
- [Multi-Factor Authentication Requirements](#multi-factor-authentication-requirements)
- [Password Manager Requirements](#password-manager-requirements)
- [Banned Password List](#banned-password-list)
- [Service Account and Shared Credential Management](#service-account-and-shared-credential-management)
- [Technical Implementation Guide](#technical-implementation-guide)
- [Compliance Mapping](#compliance-mapping)
- [Migration Guide: Moving from Legacy Policies](#migration-guide-moving-from-legacy-policies)

---

## Why This Policy Matters

Compromised credentials are the leading cause of data breaches. According to industry research:

- 80% of hacking-related breaches involve brute force or stolen credentials
- The average cost of a credential-based breach exceeds $4.5 million
- Forced password rotation leads to weaker passwords (users choose predictable patterns)
- Password reuse across services amplifies the impact of any single breach

NIST SP 800-63b, updated in 2024, fundamentally changed password guidance based on decades of research. This policy template implements those evidence-based recommendations.

---

## Policy Document

### 1. Purpose

This policy establishes authentication requirements for all users, systems, and applications at `[ORGANIZATION NAME]` to prevent unauthorized access while maintaining usability. This policy is aligned with NIST SP 800-63b and replaces all prior password policies.

### 2. Scope

This policy applies to:
- All employees, contractors, and temporary workers
- All systems, applications, and services owned or managed by `[ORGANIZATION NAME]`
- All types of accounts: user, administrator, service, shared, and emergency
- Both on-premises and cloud-hosted systems

### 3. Password Requirements

#### 3.1 Standard User Accounts

| Requirement | Standard | NIST Reference |
|-------------|----------|---------------|
| **Minimum length** | 16 characters | 800-63b Sec. 5.1.1.1 |
| **Maximum length** | No maximum enforced; systems must support at least 64 characters | 800-63b Sec. 5.1.1.1 |
| **Character types** | All printable ASCII and Unicode characters must be accepted, including spaces | 800-63b Sec. 5.1.1.1 |
| **Complexity rules** | None. Do not require specific character types (uppercase, lowercase, numbers, symbols). Length is more effective than complexity. | 800-63b Sec. 5.1.1.1 |
| **Forced expiration** | None. Passwords are changed only when there is evidence of compromise. | 800-63b Sec. 5.1.1.2 |
| **Password history** | Cannot reuse last 10 passwords | Best practice |
| **Account lockout** | Lock after 5 consecutive failed attempts. Auto-unlock after 30 minutes or manual unlock by admin. | 800-63b Sec. 5.2.2 |
| **Truncation** | Passwords must not be truncated during processing | 800-63b Sec. 5.1.1.1 |
| **Password hints** | Prohibited | 800-63b Sec. 5.1.1.2 |

#### 3.2 Privileged/Administrator Accounts

| Requirement | Standard |
|-------------|----------|
| **Minimum length** | 20 characters |
| **MFA** | Required for all access, no exceptions |
| **Session timeout** | 15 minutes of inactivity |
| **Session logging** | All privileged sessions logged with full audit trail |
| **Just-in-time access** | Preferred; standing privileged access should be minimized |
| **Separate accounts** | Admins must use separate accounts for administrative and daily tasks |
| **Review frequency** | Quarterly access review |

#### 3.3 Service Accounts

| Requirement | Standard |
|-------------|----------|
| **Minimum length** | 25 characters (randomly generated) |
| **Rotation** | Every 90 days, upon personnel change, or upon suspected compromise |
| **Storage** | Secrets management vault only (HashiCorp Vault, Azure Key Vault, AWS Secrets Manager, CyberArk, etc.) |
| **Code/config files** | Passwords must never be stored in source code, configuration files, scripts, or documentation |
| **Ownership** | Each service account must have a documented human owner |
| **Review** | All service accounts reviewed quarterly |
| **Interactive login** | Prohibited for service accounts where technically possible |

#### 3.4 Emergency/Break-Glass Accounts

| Requirement | Standard |
|-------------|----------|
| **Minimum length** | 25 characters (randomly generated) |
| **Storage** | Sealed envelope in physical safe, or secrets vault with break-glass workflow |
| **Monitoring** | Any use triggers an immediate alert to the security team |
| **Rotation** | After every use, or quarterly if unused |
| **Documentation** | Existence, purpose, and location documented in DR plan |

### 4. Banned Passwords

Systems must check passwords against a banned list at creation and change. The banned list must include:

- **Breached passwords:** Passwords found in known breach databases (e.g., HIBP Pwned Passwords list)
- **Dictionary words:** Common English dictionary words used alone
- **Context-specific passwords:** Organization name, product names, service names, usernames, email addresses
- **Common patterns:** Sequential characters (123456, abcdef), repeated characters (aaaaaa), keyboard patterns (qwerty)
- **Seasonal patterns:** Season + year (Summer2026), month + year (April2026)

Passwords that match the banned list must be rejected with a clear error message explaining why and guiding the user to choose a stronger password.

### 5. Password Storage (for System Administrators and Developers)

| Requirement | Standard |
|-------------|----------|
| **Hashing algorithm** | bcrypt (cost factor 12+), scrypt, or Argon2id |
| **Never** | Store passwords in plaintext, reversible encryption, MD5, SHA-1, or unsalted SHA-256 |
| **Salting** | Unique, random salt per password (32+ bytes) |
| **Peppering** | Optional but recommended (application-level secret added before hashing) |
| **Transmission** | Passwords transmitted only over encrypted channels (TLS 1.2+) |
| **Logging** | Passwords must never appear in logs, error messages, or debug output |

### 6. Enforcement

- Systems must technically enforce all requirements where possible (minimum length, banned list, lockout)
- Violations discovered during audit or incident investigation may result in disciplinary action
- Repeated violations or intentional circumvention may result in access revocation and escalation to management

---

## Multi-Factor Authentication Requirements

### Where MFA Is Required

| Context | MFA Required | Approved Methods |
|---------|-------------|-----------------|
| Remote access (VPN) | Yes | FIDO2, TOTP, Push |
| Cloud application login | Yes | FIDO2, TOTP, Push |
| Email from non-managed device | Yes | FIDO2, TOTP, Push |
| Administrative/privileged access | Yes | FIDO2, TOTP (no SMS) |
| Access to Confidential/Restricted data | Yes | FIDO2, TOTP |
| On-premises login from managed device (internal network) | Recommended | FIDO2, TOTP, Push |
| Physical access (badge) | Separate system | Badge + PIN |

### MFA Method Hierarchy (Strongest to Weakest)

| Rank | Method | Phishing Resistant? | Notes |
|------|--------|-------------------|-------|
| 1 | **FIDO2/WebAuthn hardware key** (YubiKey, etc.) | Yes | Strongest option. Preferred for privileged accounts. |
| 2 | **Platform authenticator** (Windows Hello, Touch ID, passkeys) | Yes | Good balance of security and convenience. |
| 3 | **Authenticator app (TOTP)** (Google Authenticator, Microsoft Authenticator, Authy) | No | Acceptable for standard accounts. Can be phished with AiTM attacks. |
| 4 | **Push notification** (Microsoft Authenticator push, Duo push) | No | Susceptible to MFA fatigue attacks. Must implement number matching. |
| 5 | **SMS OTP** | No | Last resort only. Susceptible to SIM swapping, SS7 attacks. Requires documented risk acceptance. |

### MFA Enrollment

- New employees must enroll in MFA on day one
- Minimum two MFA methods must be registered (primary + backup)
- Recovery codes must be generated and stored securely
- Lost/stolen MFA devices must be reported to IT immediately

---

## Password Manager Requirements

### Policy

All employees must use the organization-approved password manager: `[TOOL NAME]`.

### Requirements

| Requirement | Standard |
|-------------|----------|
| **Master password** | Must meet privileged account requirements (20+ characters) |
| **MFA on vault** | Required (FIDO2 or TOTP) |
| **Password generation** | All non-memorized passwords must be generated by the manager (20+ characters, random) |
| **Sharing** | Password sharing must occur only through the vault's secure sharing feature |
| **Prohibited** | Browser "remember password" feature must be disabled; use the password manager browser extension instead |
| **Personal vaults** | Employees may use the manager for personal passwords (encouraged) but must keep personal and business vaults separate |

### Password Manager Selection Criteria

If selecting a password manager, evaluate:

- [ ] Zero-knowledge architecture (provider cannot access your passwords)
- [ ] SOC 2 Type II certified
- [ ] Supports FIDO2/WebAuthn for vault access
- [ ] Emergency access / account recovery workflow
- [ ] Admin console for policy enforcement (minimum length, MFA requirement)
- [ ] Breach monitoring integration
- [ ] Secure sharing with expiration
- [ ] Audit logging
- [ ] Cross-platform (Windows, macOS, Linux, iOS, Android, browser extensions)

---

## Technical Implementation Guide

### Active Directory / Entra ID

**Minimum password length:**
- AD default maximum is 14; use Fine-Grained Password Policies (FGPP) for 16+ characters
- Entra ID supports custom banned password lists and 16-character minimum

**Banned password list:**
- Entra ID: Custom banned passwords under Authentication Methods > Password Protection
- On-premises AD: Deploy Azure AD Password Protection proxy or third-party solution
- Open-source: Integrate with HIBP Pwned Passwords API or download the hash list

**Disable password expiration:**
```
# PowerShell - Set password to never expire (AD)
Set-ADUser -Identity username -PasswordNeverExpires $true

# For all users (bulk) - evaluate per-user first
Get-ADUser -Filter * | Set-ADUser -PasswordNeverExpires $true
```

**Account lockout policy:**
- Threshold: 5 invalid attempts
- Duration: 30 minutes
- Reset counter: 30 minutes

### Linux Systems

**PAM configuration for password quality (`/etc/security/pwquality.conf`):**
```
minlen = 16
minclass = 0
maxrepeat = 3
reject_username
gecoscheck = 1
dictcheck = 1
```

**Account lockout (`/etc/pam.d/common-auth`):**
```
auth required pam_faillock.so deny=5 unlock_time=1800
```

### Application Development

- Never implement custom password hashing -- use a well-tested library
- Use bcrypt with cost factor 12+ or Argon2id
- Implement account lockout with exponential backoff
- Check passwords against HIBP API (k-anonymity model preserves privacy)
- Set secure session timeouts
- Log failed authentication attempts (without logging the password itself)

---

## Compliance Mapping

| Framework | Requirement | How This Policy Addresses It |
|-----------|------------|----------------------------|
| NIST 800-171 | 3.5.1 -- Identify system users | Account management requirements |
| NIST 800-171 | 3.5.2 -- Authenticate users | Password + MFA requirements |
| NIST 800-171 | 3.5.3 -- Use MFA | MFA requirements section |
| NIST 800-171 | 3.5.7 -- Minimum password complexity | 16-character minimum, banned list |
| NIST 800-171 | 3.5.8 -- Prohibit password reuse | 10-password history |
| NIST 800-171 | 3.5.10 -- Store only cryptographically protected passwords | Hashing requirements (bcrypt/Argon2id) |
| CMMC L2 | IA.2.078 -- Enforce minimum password complexity | Length + banned list |
| CMMC L2 | IA.2.079 -- Prohibit password reuse | History requirement |
| CMMC L2 | IA.2.081 -- MFA for local and remote access | MFA requirements |
| PCI DSS 4.0 | 8.3.6 -- Minimum password length of 12 | Exceeds (16 minimum) |
| PCI DSS 4.0 | 8.3.7 -- Not same as previous 4 passwords | Exceeds (10 history) |
| PCI DSS 4.0 | 8.4.2 -- MFA for CDE access | MFA required for Confidential/Restricted |
| HIPAA | 164.312(d) -- Person or entity authentication | Password + MFA |
| SOC 2 | CC6.1 -- Logical access security | Full policy coverage |
| ISO 27001 | A.9.4.3 -- Password management system | Password manager + technical controls |

---

## Migration Guide: Moving from Legacy Policies

If your current policy requires forced 90-day rotation and complexity rules, follow this migration plan:

### Phase 1: Preparation (Weeks 1-4)
1. Deploy password manager to all employees
2. Implement banned password list checking
3. Configure MFA for all remote access
4. Communicate the upcoming policy change

### Phase 2: Transition (Weeks 5-8)
1. Increase minimum length from current to 12 characters
2. Remove complexity requirements (uppercase, number, symbol mandates)
3. Extend rotation to 180 days (intermediate step)
4. Require MFA enrollment for all employees

### Phase 3: Full Implementation (Weeks 9-12)
1. Increase minimum length to 16 characters
2. Disable forced password rotation
3. Require password manager for all non-memorized passwords
4. Enforce MFA for all contexts listed in policy
5. Begin quarterly access reviews

### Phase 4: Validation (Week 13+)
1. Audit compliance across all systems
2. Verify banned password list is active
3. Confirm MFA adoption rate (target: 100%)
4. Address remaining exceptions

---

## Revision History

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0 | `[DATE]` | `[AUTHOR]` | Initial release |

## Approval

| Role | Name | Signature | Date |
|------|------|-----------|------|
| CISO / IT Director | | | |
| HR Director | | | |
| CEO / Executive Sponsor | | | |

---

## Professional Policy Development

Need customized policies for your organization? **Petronella Technology Group** provides:

- [Cybersecurity Policy Development](https://petronellatech.com/cybersecurity/) - Tailored security policies
- [CMMC Compliance](https://petronellatech.com/compliance/cmmc-compliance-guide/) - Policy sets for CMMC Level 2
- [HIPAA Compliance](https://petronellatech.com/compliance/hipaa-compliance/) - Healthcare-specific policies
- [Risk Assessment Services](https://petronellatech.com/cybersecurity-assessment/) - Comprehensive risk evaluation

**Petronella Technology Group** is CMMC-RP certified. [Contact us](https://petronellatech.com/contact-us/) or call (919) 348-4912.

---

## About

Created and maintained by [Petronella Technology Group](https://petronellatech.com) - a cybersecurity and managed IT services firm based in Raleigh, NC. With 23+ years of experience and zero client breaches, we help businesses secure their infrastructure and achieve compliance.

- **Website:** [petronellatech.com](https://petronellatech.com)
- **Phone:** 919-348-4912
- **Free Assessment:** [Book a consultation](https://book.petronella.ai/blake)

## License

MIT License - See [LICENSE](LICENSE) for details.
