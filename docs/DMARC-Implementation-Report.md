
# Project Closure Report: Enterprise Email Hardening (DMARC p=reject)
**Project Lead:** Graeme Edwards  
**Status:** Completed | Production Verified
**Date:** December 2025

## 1. Executive Summary
Following a third-party penetration test, our organization’s email domain was identified as a high-risk vector for exact-domain spoofing. This project successfully migrated the domain policy from `p=none` to `p=reject`. This move has effectively neutralized unauthorized use of the domain while maintaining 100% deliverability for legitimate school services.

## 2. Technical Scope
* **Environment:** Microsoft 365 / Exchange Online / Arbor MIS.
* **Objective:** Implement DMARC enforcement to block unauthorized mail.
* **Protocols:** SPF (Sender Policy Framework), DKIM (DomainKeys Identified Mail), and DMARC.

## 3. Implementation Workflow
1. **Audit Phase:** Used **MXToolbox** and DMARC reports to map sending sources, identifying **Arbor MIS** as a primary third-party sender for parent communications.
2. **Alignment Phase:** Initiated vendor-specific authentication protocols to ensure mail signed by Arbor would align with the organizational domain.
3. **Enforcement Phase:** Successfully moved the DMARC policy to `p=reject`.

## 4. Post-Enforcement Incident & Resolution
**The Issue:** Soon after enforcing `p=reject`, school office staff reported that weekly parent newsletters sent via **Arbor MIS** were being bounced by recipient providers (Gmail/Outlook).

**The Diagnosis:** * Conducted an **Exchange Message Trace** to identify the specific failure reason.
* Analyzed bounce headers which showed `dmarc=fail (p=reject)`.
* Confirmed that while Arbor was authorized in the SPF record, the emails lacked **DKIM signing**, causing a "DMARC Alignment" failure. Under `p=reject`, a simple SPF pass is insufficient if the identifier is not aligned.

**The Resolution:** * Coordinated with Arbor support to generate 3x domain-specific **CNAME records** for DKIM delegation.
* Published these records to the school's public DNS.
* Verified successful "DMARC Pass" and "Alignment: OK" using **MXToolbox Header Analyzer**.
* Confirmed successful delivery through the Exchange Mail Flow logs.
* Confirmed deliverability with staff, restoring full newsletter functionality.

## 5. Final Results & KPI
* **Spoofing Prevention:** 100% of unauthenticated emails from the primary domain are now rejected.
* **Business Continuity:** Zero ongoing deliverability issues for authorized school services.
* **Security Posture:** Successfully remediated the critical finding from the 2025 Penetration Test.

