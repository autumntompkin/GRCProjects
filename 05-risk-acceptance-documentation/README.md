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
│   Shared service account                If data is altered or deleted, the company   │
│                                         cannot determine who made the change;        |
|                                         Non-compliant with PCI DSS, risking fines    |
|                                         and potential loss of payment processing     |
│                                         capability.                                  |
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
│   No database query logging             Data breach may go undetected;               |
│                                         forensics impossible.                        │
│                                                                                      |
|                                                                                      │
│   Excessive DB admin privileges         Too many users have full access;             │
│                                         Attackers who gain this access can alter or  │
│                                         delete $2B in financial records.             │
└──────────────────────────────────────────────────────────────────────────────────────┘
```

### Phase 2: Assess Likelihood and Impact

Providing specific, evidence-based assessment.

┌──────────────────────────────────────────────────────────────────────────────────────┐
│                              LIKELIHOOD ASSESSMENT                                   │
├──────────────────────────────────────────────────────────────────────────────────────┤
│                                                                                      │
│   FACTOR                                               ASSESSMENT                    │
│   ──────                                               ──────────                    │
│                                                                                      │
│   Threat Actor Interest                                  HIGH                        │
│   - Financial data is high-value target                                              │
│   - Payment processors frequently targeted                                           │
│   - 3 competitors breached in past 24 months                                         │
│                                                                                      │
│   Attack Complexity                                      LOW (Easy to exploit)       │
│   - Password hardcoded in configuration files                                        │
│   - 15 users know credentials                                                        |
│     (large social engineering surface)                                               |
│   - No MFA, no IP restrictions                                                       │
│                                                                                      │
│   Detection Capability                                   LOW (Poor visibility)       │
│   - No database-level query logging                                                  │
│   - Legitimate and malicious access look identical                                   │
│   - Data can be exfiltrated without triggering alerts                                │
│                                                                                      │
│   Historical Incidents                                   NONE KNOWN                  │
│   - No known compromise of this system                                               │
│   - However, the company would not detect                                            │
│     a sophisticated breach                                                           |    │                                                                                      |
│   OVERALL LIKELIHOOD: MODERATE-HIGH                                                  │
│   (Conditions favor eventual compromise despite no confirmed incidents)              │
│                                                                                      │
└──────────────────────────────────────────────────────────────────────────────────────┘

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
│                             (B2B customers leave)                           |
│                                                                             |
│                                                                             │
│   Business Disruption       System taken offline for         $100K/day      │
│                             forensics                                       │
│                                                                             │
│   TOTAL POTENTIAL IMPACT: $8-50M (depending on severity)                    │
│   MOST LIKELY SCENARIO: $10-15M  (assuming delayed detection and            |
|                                   moderate data exposure)                   │
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
│   OVERALL: Compensating controls provide LIMITED protection                 │
│   They reduce some risk but do not address the core vulnerability           │
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
│   Likely Potential Cost: $10-15M if breach occurs                           │
│   Residual Risk: HIGH                                                       │
│                                                                             │
│   OPTION D: ACCEPT TEMPORARILY WITH MINIMAL IMPROVEMENTS                    │
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

## Risk ID: RA-2026-0506  
## System: PaymentReconciler v3.2 
## Risk Owner: Robert Hayden, CFO
## Prepared By: Autumn Tompkin, GRC
## Date: May 5th, 2026

---

## 1. EXECUTIVE SUMMARY

This document requests formal acceptance of the security risks associated with the
PaymentReconciler system's authentication architecture. The system does not have database-level logging and 
uses a shared service account with database administrator privileges known to 15 employees,
with a password unchanged for 18 months.

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

OPTION A: FULL REMEDIATION
OPTION B: PARTIAL REMEDIATION
OPTION C: ACCEPT RISK
OPTION D: ACCEPT TEMPORARILY WITH MINIMAL IMPROVEMENTS
       

---

## 5. SELECTED OPTION

**Option D: Accept Temporarily with Minimal Improvements**

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
- No Database query-level audit trail
- 5 people still know the password

**If a breach occurs, we will likely NOT be able to:**
- Identify which individual led to compromised credentials
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
