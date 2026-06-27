# Enterprise Vendor Security Questionnaire — Aranthir

**Assessing organization:** SwissMedAI GmbH (Health Ledger · HL-Dentistry)
**Vendor under assessment:** Aranthir
**Assessment owner:** Jesus Gomez Rossi (CEO), gomez-rossi.j@swissmedai.com
**Date issued:** 2026-06-27
**Target return date:** _(to be agreed — recommend 15 business days)_
**Classification of this document:** Confidential — Vendor Risk Management
**Framework basis:** ISO/IEC 27001:2022 Annex A · GDPR (Art. 28 & Art. 32) · German BSI IT-Grundschutz · CSA CAIQ (cloud) · NIST CSF 2.0

---

## How to use this document

This questionnaire is **pre-scoped by SwissMedAI** (Section 1) and must be **completed by Aranthir** (Sections 3–15). It is part of SwissMedAI's third-party risk-management process for the Health Ledger HL-Dentistry platform, which processes **special-category health data (GDPR Art. 9)** belonging to dental patients in German nursing homes.

For each question, Aranthir provides:

| Column | Meaning |
|---|---|
| **Response** | `Yes` / `No` / `Partial` / `N/A` |
| **Detail** | Free-text explanation, control description, or scope of applicability |
| **Evidence** | Reference to attached/linked evidence (policy, certificate, report, screenshot). Mark `[E#]` and list in Section 16. |

> **Honesty over score.** A truthful `No` with a remediation date is worth more than an unsupported `Yes`. Unsupported `Yes` answers that contradict later evidence are treated as a finding.

### Scoring & risk rating (completed by SwissMedAI after return)

Each answer is scored 0–3 and weighted by domain criticality.

| Score | Meaning |
|---|---|
| 3 | Control fully implemented and evidenced |
| 2 | Implemented, minor gap or evidence pending |
| 1 | Partial / informal / compensating control only |
| 0 | Absent, or answer contradicts evidence |

**Domain weighting** (Critical domains gate the engagement regardless of total):

| Tier | Domains | Effect |
|---|---|---|
| **Critical (gating)** | Data Protection & GDPR (§4), Data Security & Encryption (§6), Incident Response (§9), Sub-processors (§13) | Any score of 0 in a gating domain → **engagement blocked** pending remediation |
| **High** | Access Control (§5), Infrastructure & Cloud (§7), Application Security & SDLC (§8), Business Continuity (§10) | |
| **Standard** | Governance (§3), Network (§7), HR Security (§11), Physical (§12), AI/ML Governance (§14), Compliance & Audit (§15) | |

**Overall risk rating bands** (weighted % of max achievable):

| Band | Score | Outcome |
|---|---|---|
| 🟢 Low | ≥ 85% and no gating 0 | Approve |
| 🟡 Moderate | 70–84% | Approve with conditions + remediation plan |
| 🟠 Elevated | 55–69% | Conditional / re-assess after remediation |
| 🔴 High | < 55% or any gating 0 | Do not engage / escalate |

---

## 1. Engagement scope — *pre-filled by SwissMedAI*

| Item | Value |
|---|---|
| Service Aranthir provides | _(confirm)_ Cloud/SaaS processing on behalf of SwissMedAI — i.e. a **sub-processor under GDPR Art. 28** |
| Data Aranthir will access/process | Patient master data, dental clinical records (odontogram, PA §22a findings), treatment & billing data, scheduling, staff identities |
| Data classification | **Special-category personal data (GDPR Art. 9 — health)** — highest sensitivity |
| Approximate data subjects | Nursing-home residents (patients) + dental practice staff across multiple German `Heime` |
| Data residency requirement | **EU/EEA only** (Germany preferred). No transfer to third countries without Art. 46 safeguards (SCCs + TIA) |
| Integration surface | API/data exchange with HL backend (Node.js + Fastify + Prisma + PostgreSQL), JWT-authenticated; potentially SendGrid email, Claude/Whisper AI pipelines |
| Regulatory context | GDPR / German DSGVO + BDSG, §203 StGB (professional secrecy), patient-data retention duties, BSI IT-Grundschutz expectations |
| Contractual prerequisites | Signed **AVV / DPA (Auftragsverarbeitungsvertrag)** under GDPR Art. 28 **before** any production data is shared |

> Aranthir: if any assumption above is inaccurate, correct it in Section 2 — it changes the assessment.

---

## 2. Vendor profile — *completed by Aranthir*

| Field | Response |
|---|---|
| Legal entity name & registration no. | |
| Headquarters / country of incorporation | |
| Year founded · employee count | |
| Primary security contact (name, role, email) | |
| Data Protection Officer (name, email) | |
| Service/product being supplied to SwissMedAI | |
| Where will SwissMedAI data be **stored** (countries/regions)? | |
| Where will SwissMedAI data be **processed/accessed** from? | |
| Names of any sub-processors involved | |
| Public trust/security page or status page URL | |

---

## 3. Security governance & risk management *(Standard)*

| # | Question | Response | Detail | Evidence |
|---|---|---|---|---|
| 3.1 | Do you have a documented, management-approved information security policy reviewed at least annually? | | | |
| 3.2 | Is there a named individual accountable for information security (CISO or equivalent)? | | | |
| 3.3 | Do you hold **ISO/IEC 27001** certification (or equivalent)? State scope & certificate number. | | | |
| 3.4 | Do you have a current **SOC 2 Type II** report (or BSI C5, ISO 27017/27018)? | | | |
| 3.5 | Do you run a formal risk-assessment process with a maintained risk register? | | | |
| 3.6 | Is security awareness training mandatory for all staff at onboarding and annually? | | | |
| 3.7 | Do you carry cyber-liability / professional-indemnity insurance? State coverage. | | | |
| 3.8 | Do you maintain an asset inventory classifying systems that store/process customer data? | | | |

## 4. Data protection & GDPR compliance *(CRITICAL — gating)*

| # | Question | Response | Detail | Evidence |
|---|---|---|---|---|
| 4.1 | Will you sign SwissMedAI's **AVV/DPA (GDPR Art. 28)** before processing any data? | | | |
| 4.2 | Is **all** processing of SwissMedAI data performed **within the EU/EEA**? List storage & backup regions. | | | |
| 4.3 | If any processing/support occurs outside the EEA, what Art. 46 safeguards apply (SCCs, TIA, supplementary measures)? | | | |
| 4.4 | Do you maintain a Record of Processing Activities (RoPA, Art. 30)? | | | |
| 4.5 | Can you support data-subject rights requests (access, rectification, erasure, portability) within statutory timelines? | | | |
| 4.6 | Do you enforce **data-minimisation** — only the fields strictly required are processed? | | | |
| 4.7 | Do you have documented retention & secure-deletion schedules? State retention period for health data. | | | |
| 4.8 | On contract termination, will you return and/or **certifiably destroy** all SwissMedAI data? State method (e.g. crypto-erase) and timeframe. | | | |
| 4.9 | Do you commit to **not** using SwissMedAI patient data for your own purposes, analytics, or model training? | | | |
| 4.10 | Have you completed (or will you support) a **DPIA** for this health-data processing (Art. 35)? | | | |
| 4.11 | Are you aware of and able to honour German professional-secrecy obligations (§203 StGB) for health data? | | | |
| 4.12 | Do you maintain an up-to-date list of sub-processors and notify customers before adding new ones? | | | |

## 5. Identity & access management *(High)*

| # | Question | Response | Detail | Evidence |
|---|---|---|---|---|
| 5.1 | Is **MFA** enforced for all employee access to systems handling customer data? | | | |
| 5.2 | Is access granted on **least-privilege / need-to-know** with documented approval? | | | |
| 5.3 | Are privileged/admin accounts separated from standard accounts and individually attributable (no shared creds)? | | | |
| 5.4 | Do you review user access rights at least quarterly and revoke promptly on role change/offboarding? | | | |
| 5.5 | Is SSO/SAML/OIDC supported for any interface SwissMedAI users access? | | | |
| 5.6 | How are secrets/API keys/JWT signing keys stored and rotated? (e.g. HSM, vault, rotation period) | | | |
| 5.7 | Are session timeouts, lockout, and password policies enforced to a defined standard (e.g. NIST 800-63B)? | | | |
| 5.8 | Is all privileged access logged and monitored? | | | |

## 6. Data security & encryption *(CRITICAL — gating)*

| # | Question | Response | Detail | Evidence |
|---|---|---|---|---|
| 6.1 | Is customer data **encrypted at rest** (state algorithm, e.g. AES-256)? | | | |
| 6.2 | Is data **encrypted in transit** with TLS 1.2+ (1.3 preferred), HSTS, modern ciphers only? | | | |
| 6.3 | How are encryption keys managed (KMS/HSM), and who can access them? Is customer-managed/BYOK supported? | | | |
| 6.4 | Is SwissMedAI data **logically segregated** from other tenants? Describe the multi-tenancy isolation model. | | | |
| 6.5 | Is sensitive data masked/tokenised/pseudonymised where full values aren't required? | | | |
| 6.6 | Are non-production environments **free of real patient data** (or fully anonymised)? | | | |
| 6.7 | Are removable media and endpoints with data access encrypted (full-disk)? | | | |
| 6.8 | Do you have DLP controls to prevent exfiltration of customer data? | | | |

## 7. Infrastructure, cloud & network security *(High / Standard)*

| # | Question | Response | Detail | Evidence |
|---|---|---|---|---|
| 7.1 | Which cloud provider(s)/data centres host the service? Are they ISO 27001 / C5 certified? | | | |
| 7.2 | Is infrastructure hardened to a recognised baseline (CIS Benchmarks, BSI Grundschutz)? | | | |
| 7.3 | Are networks segmented, with firewalls/security groups restricting traffic to least-necessary? | | | |
| 7.4 | Do you run IDS/IPS and centralised log monitoring (SIEM)? | | | |
| 7.5 | Is a documented **vulnerability-management** process in place with defined remediation SLAs by severity? | | | |
| 7.6 | How frequently do you patch? State SLA for critical vulnerabilities. | | | |
| 7.7 | Do you conduct independent **penetration tests** at least annually? Will you share a summary/attestation? | | | |
| 7.8 | Are backups encrypted, access-controlled, and **restore-tested** on a defined schedule? | | | |
| 7.9 | Is anti-malware/EDR deployed on relevant systems and endpoints? | | | |
| 7.10 | Are administrative interfaces restricted (bastion/VPN, IP allow-listing)? | | | |

## 8. Application security & SDLC *(High)*

| # | Question | Response | Detail | Evidence |
|---|---|---|---|---|
| 8.1 | Do you follow a secure SDLC with security requirements and design review? | | | |
| 8.2 | Are code reviews and branch protections mandatory before merge to production? | | | |
| 8.3 | Do you run SAST, DAST, and **SCA / dependency scanning** in CI? | | | |
| 8.4 | Do you maintain an **SBOM** and track/remediate vulnerable third-party components? | | | |
| 8.5 | Are secrets prevented from entering source control (secret scanning, pre-commit hooks)? | | | |
| 8.6 | Do you defend against OWASP Top 10 (injection, auth, SSRF, etc.) with documented controls? | | | |
| 8.7 | Are environments (dev/test/prod) separated with controlled, audited promotion? | | | |
| 8.8 | Do you have a published API security standard (authn/authz, rate limiting, input validation)? | | | |
| 8.9 | Do you operate a vulnerability-disclosure or bug-bounty program? | | | |

## 9. Incident response & breach notification *(CRITICAL — gating)*

| # | Question | Response | Detail | Evidence |
|---|---|---|---|---|
| 9.1 | Do you have a documented, tested incident-response plan with defined roles? | | | |
| 9.2 | Will you notify SwissMedAI of a confirmed/suspected breach affecting its data **without undue delay and within 24 hours**? | | | |
| 9.3 | Do you support GDPR Art. 33 timelines (controller's 72-hour authority notification) by providing required details promptly? | | | |
| 9.4 | Do you maintain forensic logging sufficient to investigate an incident? State log retention period. | | | |
| 9.5 | Do you conduct post-incident reviews and share root-cause/remediation with affected customers? | | | |
| 9.6 | Have you experienced a reportable breach in the last 24 months? If yes, summarise and remediation. | | | |
| 9.7 | Is there a 24/7 security on-call / contact channel for incidents? | | | |

## 10. Business continuity & resilience *(High)*

| # | Question | Response | Detail | Evidence |
|---|---|---|---|---|
| 10.1 | Do you have a documented, tested BCP/DR plan? State test frequency. | | | |
| 10.2 | What are your **RTO** and **RPO** for the service? | | | |
| 10.3 | What uptime SLA do you commit to? Provide trailing-12-month availability. | | | |
| 10.4 | Is the service deployed with redundancy/failover across availability zones? | | | |
| 10.5 | Are backups geographically redundant **within the EEA**? | | | |
| 10.6 | Do you provide a public status page and proactive outage communication? | | | |

## 11. Human-resources security *(Standard)*

| # | Question | Response | Detail | Evidence |
|---|---|---|---|---|
| 11.1 | Are background/reference checks performed (where legally permitted) before granting data access? | | | |
| 11.2 | Are staff bound by confidentiality/NDA covering customer data? | | | |
| 11.3 | Is there a documented offboarding process that revokes all access and recovers assets? | | | |
| 11.4 | Are contractors/temps held to the same security obligations as employees? | | | |
| 11.5 | Is there an enforced acceptable-use and remote-work/BYOD policy? | | | |

## 12. Physical & environmental security *(Standard)*

| # | Question | Response | Detail | Evidence |
|---|---|---|---|---|
| 12.1 | Are data centres certified (ISO 27001 / C5 / SOC 2) with documented physical controls? | | | |
| 12.2 | Is physical access restricted, logged, and reviewed (badges, biometrics, CCTV)? | | | |
| 12.3 | Are environmental controls (power, fire, cooling) and redundancy in place? | | | |
| 12.4 | Is media securely sanitised/destroyed at end of life with certificates? | | | |
| 12.5 | For any office handling customer data, are clean-desk and visitor controls enforced? | | | |

## 13. Sub-processor & supply-chain management *(CRITICAL — gating)*

| # | Question | Response | Detail | Evidence |
|---|---|---|---|---|
| 13.1 | Provide a complete list of sub-processors touching SwissMedAI data (name, purpose, location). | | | |
| 13.2 | Do you flow down equivalent security & GDPR obligations to all sub-processors via contract? | | | |
| 13.3 | Do you perform security due diligence on sub-processors before onboarding and periodically? | | | |
| 13.4 | Will you give SwissMedAI **advance notice and a right to object** before adding/changing sub-processors? | | | |
| 13.5 | Are all sub-processors processing health data located within the EEA (or covered by Art. 46 safeguards)? | | | |

## 14. AI / ML governance *(Standard — applies if AI is used)*

> Relevant because HL-Dentistry uses AI (Claude API assistant, Whisper dictation). Complete if Aranthir provides or uses any AI/ML on SwissMedAI data.

| # | Question | Response | Detail | Evidence |
|---|---|---|---|---|
| 14.1 | Do any AI/ML features process SwissMedAI patient data? Describe the data flow. | | | |
| 14.2 | Do you guarantee SwissMedAI data is **never used to train or fine-tune** your or any provider's models? | | | |
| 14.3 | Which AI sub-providers/model APIs are involved, and where are they hosted (EEA)? | | | |
| 14.4 | Is AI input/output covered by the same encryption, logging, and retention controls as other data? | | | |
| 14.5 | Are prompts/outputs retained by any AI provider? State retention and opt-out (e.g. zero-retention agreements). | | | |
| 14.6 | Do you have human-oversight, bias, and accuracy controls appropriate to clinical context (and EU AI Act readiness)? | | | |
| 14.7 | Are AI features isolated so a prompt-injection or model failure cannot expose other tenants' data? | | | |

## 15. Compliance, audit & right to audit *(Standard)*

| # | Question | Response | Detail | Evidence |
|---|---|---|---|---|
| 15.1 | Will you provide annual evidence of compliance (certs, pen-test summary, SOC 2) for the duration of the contract? | | | |
| 15.2 | Do you grant SwissMedAI (or an independent auditor) a contractual **right to audit** per GDPR Art. 28(3)(h)? | | | |
| 15.3 | Will you complete this questionnaire (or an updated risk review) at least annually? | | | |
| 15.4 | Do you notify customers of material changes to your security posture or certifications? | | | |
| 15.5 | Are you subject to any regulatory action, investigation, or material litigation relevant to security/privacy? | | | |

---

## 16. Evidence index — *completed by Aranthir*

| Ref | Document / evidence | Date | Notes |
|---|---|---|---|
| E1 | | | |
| E2 | | | |
| E3 | | | |

---

## 17. Scoring summary — *completed by SwissMedAI*

| Domain | Tier | Max | Score | % | Notes |
|---|---|---|---|---|---|
| 3 Governance | Standard | | | | |
| 4 Data Protection & GDPR | **Critical** | | | | |
| 5 Access Control | High | | | | |
| 6 Data Security & Encryption | **Critical** | | | | |
| 7 Infrastructure / Cloud / Network | High | | | | |
| 8 AppSec & SDLC | High | | | | |
| 9 Incident Response | **Critical** | | | | |
| 10 Business Continuity | High | | | | |
| 11 HR Security | Standard | | | | |
| 12 Physical | Standard | | | | |
| 13 Sub-processors | **Critical** | | | | |
| 14 AI/ML Governance | Standard | | | | |
| 15 Compliance & Audit | Standard | | | | |
| **TOTAL** | | | | | |

**Gating check (any score of 0 in §4, §6, §9, §13?):** ☐ None ☐ Found → block & remediate
**Overall risk rating:** ☐ 🟢 Low ☐ 🟡 Moderate ☐ 🟠 Elevated ☐ 🔴 High
**Decision:** ☐ Approve ☐ Approve with conditions ☐ Re-assess after remediation ☐ Do not engage

### Findings & required remediations

| # | Finding | Severity | Required action | Owner | Due |
|---|---|---|---|---|---|
| 1 | | | | | |
| 2 | | | | | |

---

## 18. Attestation & sign-off

**Vendor (Aranthir).** I confirm the responses above are accurate and complete to the best of my knowledge, and that supporting evidence is genuine.

| Name | Title | Signature | Date |
|---|---|---|---|
| | | | |

**Assessor (SwissMedAI GmbH).**

| Name | Title | Decision | Signature | Date |
|---|---|---|---|---|
| Jesus Gomez Rossi | CEO | | | |

---

*Prepared for SwissMedAI GmbH / Health Ledger HL-Dentistry vendor-risk process. References: ISO/IEC 27001:2022, GDPR Art. 28/30/32/33/35, German BDSG & §203 StGB, BSI IT-Grundschutz / C5, CSA CAIQ v4, NIST CSF 2.0. This assessment does not by itself constitute a Data Processing Agreement; a signed AVV/DPA is a separate contractual prerequisite.*
