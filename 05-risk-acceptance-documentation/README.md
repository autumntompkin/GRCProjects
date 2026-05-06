## Risk Acceptance for $2B Payment System Security Gap

**Company:** FinanceFlow Inc.
**Industry:** B2B Payment Processing
**Size:** 400 employees, $50M ARR
**Compliance:** SOC 2 Type II, PCI DSS Level 2

### The Security Gap

The company's legacy payment reconciliation system:

1. Processes $2B in transactions annually
2. Uses a shared service account with database administrator privileges
3. Is accessible to 15 team members who know the password for the shared service account
4. Has not had the password rotated in 18 months
5. Would require 12–18 months to replace

**Technical Details:**
- System: PaymentReconciler v3.2 (internal application)
- Database: Oracle 12c (stores cardholder data subject to PCI DSS requirements)  
- Authentication: Uses a static username and password stored in configuration files  
- Monitoring: Application activity is logged, but database-level activity is not tracked
- Environment: Operates within the cardholder data environment (CDE), 
               where sensitive payment data is stored and processed

**Business Context:**
- The finance team relies on this system for daily operations
- No budget is allocated for replacement in current fiscal year
- A previous attempt to modernize the system failed 2 years ago
- The system is owned by the CFO  

---

### Phase 1: Understanding the Risk

Translating technical findings into business risk.

```
┌──────────────────────────────────────────────────────────────────────────────────────┐
│                              VULNERABILITY VS RISK                                   │
├──────────────────────────────────────────────────────────────────────────────────────┤
│                                                                                      │    
│   VULNERABILITY (Technical)             RISK (Business)                              │   
│   ─────────────────────────             ───────────────                              |
│                                                                                      │
│   Shared service account                Company cannot determine who alters or       │
│                                         deletes data; Non-compliant with PCI DSS,    |
|                                         creating risk of fines and potential         |
|                                         loss of payment processing capability.       |
│                                                                                      |
|                                                                                      │
│   Broad password exposure               Increases likelihood of compromise;          |
|   (15 users know credentials)           A single phishing attack could grant         |    |                                         access to $2B in transaction data.           |
|                                                                                      |    
│                                                                                      │
│   No password rotation                  If password is compromised,                  |
|                                         attacker has 18+ months of access.           |
|                                         Also non-compliant with PCI DSS.             |
|                                                                                      |
│                                                                                      │
│   No database query logging             Data breach may go undetected;               │
│                                         forensics impossible.                        │
│                                                                                      |
|                                                                                      │
│   Excessive DB admin privileges         Too many users have full access;             │
│                                         Attackers who gain entry will also have      │
│                                         full access to alter or delete $2B in financial records. │
└──────────────────────────────────────────────────────────────────────────────────────┘
```

### Phase 2: Assess Likelihood and Impact

Providing specific, evidence-based assessment.


┌─────────────────────────────────────────────────────────────────────────────┐
│                         LIKELIHOOD ASSESSMENT                               │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│   FACTOR                                ASSESSMENT                          │
│   ──────                                ──────────                          │
│                                                                             │
│   Threat Actor Interest                 HIGH                                │
│   - Financial data is high-value target                                     │
│   - Payment processors frequently targeted                                  │
│   - 3 competitors breached in past 24 months                                │
│                                                                             │
│   Attack Complexity                     LOW (Easy to exploit)     <-- CHANGING           │
│   - Password may be in source control                                       │
│   - 15 people know it (social engineering surface)                          │
│   - No MFA, no IP restrictions                                              │
│                                                                             │
│   Detection Capability                  LOW                       <-- CHANGING           │
│   - No query logging                                                        │
│   - Legitimate and malicious access look identical                          │
│   - Could exfiltrate data without triggering alerts                         │
│                                                                             │
│   Historical Incidents                  NONE KNOWN                 <-- CHANGING          │
│   - No known compromise of this system                                      │
│   - However, we would not detect a sophisticated breach                     │
│                                                                             │
│   OVERALL LIKELIHOOD: MODERATE-HIGH                                         │
│   (Not certain, but conditions favor eventual compromise)                   │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────────────┐
│                         IMPACT ASSESSMENT                                   │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│   IMPACT CATEGORY           POTENTIAL CONSEQUENCE            ESTIMATED COST │
│   ───────────────           ──────────────────────           ─────────────  │
│                                                                             │
│   Data Breach               Exposure of cardholder data      $5-15M         │
│                             (PCI penalties + notification)                  │
│                                                                             │
│   Financial Fraud           Manipulation of transaction      $1-10M         │
│                             records                          (Depends on    │
│                                                              detection)     │
│                                                                             │
│   Regulatory Action         PCI DSS non-compliance finding   $500K-5M       │
│                             and SOC 2 qualified opinion      (Fines +       │
│                                                              remediation)   │
│                                                                             │
│   Reputation                Customer loss, media coverage    $2-20M         │
│                             (B2B customers leave)            (Hard to       │
│                                                              quantify)      │
│                                                                             │
│   Business Disruption       System taken offline for         $100K/day      │
│                             forensics                                       │
│                                                                             │
│   TOTAL POTENTIAL IMPACT: $8-50M (depending on severity)                    │
│   MOST LIKELY SCENARIO: $10-15M  (assuming <-- MUST FILL IN)                                            │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Phase 3: Evaluate Compensating Controls

Assessing what protection currently exists.

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    COMPENSATING CONTROLS ASSESSMENT                         │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│   CLAIMED CONTROL           EFFECTIVENESS              HONEST ASSESSMENT    │
│   ───────────────           ─────────────              ─────────────────    │
│                                                                             │
│   "Network is                PARTIAL                   True, but:           │
│   segmented"                                           - 15 people have     │
│                                                          legitimate access  │
│                                                        - Lateral movement   │
│                                                          from their         │
│                                                          workstations       │
│                                                          possible           │
│                                                                             │
│   "Application              WEAK                       - App logs exist     │
│   logging enabled"                                     - DB queries not     │
│                                                          logged             │
│                                                        - Could copy entire  │
│                                                          DB without trace   │
│                                                                             │
│   "Background checks        MINIMAL                    - Doesn't prevent    │
│   on employees"                                          compromise         │
│                                                        - Doesn't prevent    │
│                                                          insider threat     │
│                                                        - Doesn't detect     │
│                                                          abuse              │
│                                                                             │
│   "Encrypted at rest"       IRRELEVANT                 - Legitimate user    │
│                                                          sees decrypted     │
│                                                          data anyway        │
│                                                        - Encryption doesn't │
│                                                          prevent authorized │
│                                                          access abuse       │
│                                                                             │
│   OVERALL: Compensating controls provide LIMITED protection                │
│   They reduce some risk but do not address the core vulnerability          │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Phase 4: Present Options to Business

Giving decision-makers real choices.

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         DECISION OPTIONS                                    │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│   OPTION A: FULL REMEDIATION                                                │
│   ─────────────────────────                                                 │
│   Replace system with modern architecture                                   │
│   - Individual service accounts with MFA                                    │
│   - Full audit logging                                                      │
│   - Automated password rotation                                             │
│                                                                             │
│   Cost: $800K-1.2M (development + migration)                                │
│   Timeline: 12-18 months                                                    │
│   Residual Risk: LOW                                                        │
│                                                                             │
│   OPTION B: PARTIAL REMEDIATION                                             │
│   ────────────────────────────                                              │
│   Keep system, add controls:                                                │
│   - Implement database activity monitoring                                  │
│   - Reduce password knowledge to 3 people                                   │
│   - Add IP allow-listing                                                    │
│   - Quarterly password rotation                                             │
│                                                                             │
│   Cost: $150K (tooling + configuration)                                     │
│   Timeline: 3 months                                                        │
│   Residual Risk: MEDIUM (still shared account, still no MFA)                │
│                                                                             │
│   OPTION C: ACCEPT RISK                                                     │
│   ───────────────────────                                                   │
│   Continue current state with documented acceptance                         │
│                                                                             │
│   Cost: $0 direct cost                                                      │
│   Potential Cost: $10-15M if breach occurs                                  │
│   Residual Risk: HIGH                                                       │
│                                                                             │
│   OPTION D: ACCEPT WITH MINIMAL IMPROVEMENTS                                │
│   ──────────────────────────────────────────                                │
│   Accept risk but implement:                                                │
│   - Password rotation (now and quarterly)                                   │
│   - Reduce to 5 people with access                                          │
│   - Add to next fiscal year budget for full fix                             │
│                                                                             │
│   Cost: $20K + staff time                                                   │
│   Timeline: 2 weeks                                                         │
│   Residual Risk: HIGH (but slightly reduced)                                │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Phase 5: Write the Risk Acceptance Document

Creating a formal document for CFO signature

**Document Structure:**

```markdown
# RISK ACCEPTANCE DOCUMENT

## Risk ID: RA-2026-017 <-- CHANGING 
## System: PaymentReconciler v3.2 <-- CHANGING 
## Risk Owner: [CFO Name] <-- CHANGING 
## Prepared By: Autumn Tompkin, GRC
## Date: May 5th, 2026

---

## 1. EXECUTIVE SUMMARY

This document requests formal acceptance of security risk associated with the
PaymentReconciler system's authentication architecture. The system uses a shared
service account with database administrator privileges, known to 15 employees,
with a password unchanged for 18 months. <-- CHANGING 

**Recommendation:** Option D (Accept with Minimal Improvements) while allocating
budget for full remediation in FY2027.

---

## 2. RISK DESCRIPTION

### What Could Happen
An unauthorized person could gain access to the PaymentReconciler database and:
- View all transaction records ($2B annually)
- Modify financial data without detection
- Exfiltrate cardholder data subject to PCI DSS

### How It Could Happen
- One of 15 employees could have their credentials phished
- A former employee may still have knowledge of the password
- The password may exist in backups, source control, or documentation
- An insider could abuse their access

### Why We Might Not Know
- No database query logging exists
- Legitimate and malicious access appear identical
- A sophisticated attacker could operate undetected

---

## 3. RISK ASSESSMENT

| Factor | Assessment | Basis |
|--------|------------|-------|
| Likelihood | Moderate-High | 15 people know password; no rotation in 18 months; high-value target |
| Impact | $10-15M most likely | PCI fines, breach notification, reputation damage |
| Current Detection | Poor | No DB-level logging |
| Existing Controls | Limited | Network segmentation only |

---

## 4. OPTIONS CONSIDERED

[Insert options table from Phase 4]

---

## 5. SELECTED OPTION

**Option D: Accept with Minimal Improvements**

### Rationale
- Full remediation cannot be completed in current fiscal year
- Partial improvements meaningfully reduce likelihood
- Risk is documented and owned
- Budget allocated for FY2027 resolution

### Immediate Actions (Within 2 Weeks)
1. Rotate password immediately
2. Reduce access to 5 named individuals
3. Document all individuals with access
4. Establish quarterly rotation schedule

### FY2027 Commitment
- Allocate $1M for system replacement
- Begin requirements gathering Q1 FY2027

---

## 6. RESIDUAL RISK ACKNOWLEDGMENT

After implementing Option D, the following risks remain:

- Shared account still in use (no individual accountability)
- No MFA on database access
- No query-level audit trail
- 5 people still know the password

**If a breach occurs, we will likely NOT be able to:**
- Identify which individual's credentials were used
- Determine what data was accessed
- Prove to regulators that we had adequate controls

---

## 7. CONDITIONS AND REVIEW

This acceptance is valid under the following conditions:
- Password is rotated within 2 weeks and quarterly thereafter
- Access is limited to 5 named individuals
- Budget for remediation is included in FY2027 planning
- This acceptance is reviewed in 6 months

If any condition is not met, this acceptance is void and re-evaluation is required.

---

## 8. SIGNATURES

### Risk Owner (Business)
By signing, I acknowledge that I understand the risk described above and accept
responsibility for this risk on behalf of the organization.

Name: _______________________
Title: CFO
Date: _______________________
Signature: ___________________

### Security Review
By signing, I confirm that the risk has been accurately described and the
assessment is complete.

Name: _______________________
Title: CISO
Date: _______________________
Signature: ___________________

### Compliance Review
By signing, I confirm that relevant compliance implications have been considered.

Name: _______________________
Title: Head of Compliance
Date: _______________________
Signature: ___________________
```

---

## Success Criteria

Your risk acceptance demonstrates strong GRC judgment if:

| Criteria | Evidence |
|----------|----------|
| **Plain language** | Executive can understand without security background |
| **Specific, not vague** | Dollar amounts, timelines, named individuals |
| **Honest about controls** | Doesn't inflate effectiveness of compensating controls |
| **Options presented** | Business makes informed choice, not security dictating |
| **Residual risk clear** | Explicitly states what danger remains |
| **Conditions stated** | Acceptance has boundaries and review dates |

### Red Flags (Weak Project)

- Technical jargon without translation
- "Risk is low" without evidence
- "Controls are in place" without effectiveness assessment
- Only one option presented (no choice)
- No discussion of what happens if breach occurs
- Security "approving" instead of business "owning"

---

## Communication Comparison

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    BAD VS GOOD RISK COMMUNICATION                           │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│   BAD (Technical Jargon)                                                    │
│   ──────────────────────                                                    │
│   "The PaymentReconciler system presents a credential stuffing attack       │
│   surface with potential for lateral movement into the CDE. The shared      │
│   service account architecture enables privilege escalation scenarios       │
│   and creates accountability gaps in audit trails."                         │
│                                                                             │
│   GOOD (Plain Language)                                                     │
│   ─────────────────────                                                     │
│   "15 people know the password to our payment database. The password        │
│   hasn't been changed in 18 months. If any of those people are phished,     │
│   or if a former employee shared the password, someone could access         │
│   $2B worth of transaction data. We would probably not detect it."          │
│                                                                             │
│   BAD (Vague)                                                               │
│   ───────────                                                               │
│   "Risk: Medium. Impact: High. Likelihood: Low. Controls are in place."     │
│                                                                             │
│   GOOD (Specific)                                                           │
│   ───────────────                                                           │
│   "If this system is breached, we estimate $10-15M in costs from PCI        │
│   fines, customer notification, and likely loss of two major clients.       │
│   Three similar companies were breached last year with comparable           │
│   impacts."                                                                 │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## Deliverables Checklist

```
□ Risk description in plain language
□ Likelihood assessment with evidence
□ Impact assessment with dollar estimates
□ Compensating controls evaluation (honest)
□ Options analysis (minimum 3 options)
□ Formal risk acceptance document
□ Conditions and review schedule
□ 5-minute video presenting this to a hypothetical CFO
```

---

## Additional Scenarios (Practice)

Apply this framework to other common risk acceptance scenarios:

1. **Unpatched Legacy System** - Vendor no longer provides updates
2. **MFA Exception** - Executive refuses to use MFA
3. **Third-Party Risk** - Critical vendor fails security assessment
4. **Encryption Gap** - Database encryption "too expensive"
5. **Insider Threat** - No DLP on sensitive data

---

## Reflection Questions

After completing this project, answer:

1. What was hardest about writing in plain language?
2. How did you decide on the dollar impact estimates?
3. What would you do if the CFO refused all options and demanded acceptance?
4. How would you feel presenting this if the breach happened next month?
5. What's the difference between this document and a "cover your ass" exercise?

---

*Remember: Risk acceptance is a business decision, not a security permission slip. The goal is to present a clear, honest picture so the business can choose knowingly.*
