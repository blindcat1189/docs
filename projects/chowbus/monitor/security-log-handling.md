# Monitor System: Sensitive Log Security Assessment

**Document Type**: Security Analysis & Risk Assessment
**Classification**: Internal - Security Team
**Last Updated**: 2026-01-12
**Status**: Draft for Security Review

---

## Table of Contents

1. [Executive Summary](#1-executive-summary)
2. [System Overview](#2-system-overview)
3. [Log Classification Framework](#3-log-classification-framework)
4. [Access Control Assessment](#4-access-control-assessment)
5. [Data Protection Controls](#5-data-protection-controls)
6. [Data Isolation Analysis](#6-data-isolation-analysis)
7. [Risk Assessment](#7-risk-assessment)
8. [Recommendations](#8-recommendations)

---

## 1. Executive Summary

### Purpose

This document provides a security assessment of how the Monitor System handles sensitive log data, addressing:

- How logs are classified by sensitivity level
- Access controls protecting sensitive data
- Data masking and encryption mechanisms
- Whether physical data isolation is required

### Key Findings

| Control Area | Current Maturity | Risk Level | Action Required |
|-------------|------------------|------------|-----------------|
| Data Classification | Implemented | Low | Formalize criteria |
| Authentication | Strong | Low | Maintain |
| Authorization | Weak | **High** | Immediate action |
| Data Masking | Strong | Low | Extend to ingestion |
| Encryption at Rest | Not Implemented | **Medium** | Evaluate options |
| Audit Logging | Partial | **Medium** | Enhance |

### Primary Recommendation

**Physical separation (separate database instances) is NOT recommended.** The current logical separation model is adequate when combined with enhanced application-layer access controls. A separate instance adds operational complexity without proportional security benefit.

---

## 2. System Overview

### Data Flow Architecture

```
                              MONITOR SYSTEM DATA FLOW
  ┌─────────────────────────────────────────────────────────────────────────┐
  │                                                                         │
  │   DATA SOURCES              PROCESSING                 STORAGE          │
  │                                                                         │
  │   ┌─────────────┐      ┌──────────────────┐      ┌──────────────────┐  │
  │   │ Application │      │                  │      │                  │  │
  │   │ Logs        │─────▶│  Log Collector   │─────▶│   StarRocks      │  │
  │   │ (Kafka)     │      │  Service         │      │   (OLAP DB)      │  │
  │   └─────────────┘      │                  │      │                  │  │
  │                        │  • Parsing       │      │  ┌────────────┐  │  │
  │   Log Types:           │  • Classification│      │  │ General    │  │  │
  │   • API Access Logs    │  • Transformation│      │  │ Logs       │  │  │
  │   • Frontend Events    │                  │      │  │ (28 days)  │  │  │
  │   • POS Transactions   └──────────────────┘      │  └────────────┘  │  │
  │                                                  │                  │  │
  │                                                  │  ┌────────────┐  │  │
  │                                                  │  │ Sensitive  │  │  │
  │                                                  │  │ Logs       │  │  │
  │                                                  │  │ (180 days) │  │  │
  │                                                  │  └────────────┘  │  │
  │                                                  └──────────────────┘  │
  │                                                                         │
  └─────────────────────────────────────────────────────────────────────────┘
```

### Data Types Processed

| Category | Examples | Volume | Sensitivity |
|----------|----------|--------|-------------|
| API Request/Response | HTTP headers, bodies, status codes | High | Variable |
| User Identifiers | User ID, restaurant ID, device ID | High | Medium |
| Transaction Data | Order details, payment references | Medium | High |
| Device Information | IP address, user agent, app version | High | Medium |
| Business Events | Click events, page views, errors | High | Low |

---

## 3. Log Classification Framework

### 3.1 Current Classification Model

The system implements a **three-tier sensitivity model**:

| Level | Description | Retention | Storage Location |
|-------|-------------|-----------|------------------|
| **HIGH** | PII, financial data, authentication credentials | 180 days | Separate table |
| **MEDIUM** | Contact info, behavioral data, device fingerprints | 28 days | General table |
| **LOW** | Operational metrics, non-PII metadata | 28 days | General table |

### 3.2 Classification Criteria

#### HIGH Sensitivity (Requires Maximum Protection)

| Data Type | Examples | Regulatory Concern |
|-----------|----------|-------------------|
| Payment Card Data | Full PAN, CVV, expiry | PCI-DSS |
| Government IDs | SSN, driver's license, passport | Identity theft |
| Authentication Secrets | Passwords, tokens, API keys | Account compromise |
| Financial Account Numbers | Bank accounts, routing numbers | Financial fraud |
| Health Information | Medical records, prescriptions | HIPAA (if applicable) |

#### MEDIUM Sensitivity (Requires Protection)

| Data Type | Examples | Regulatory Concern |
|-----------|----------|-------------------|
| Contact Information | Email, phone, address | Privacy regulations |
| Behavioral Data | Purchase history, preferences | Profiling concerns |
| Location Data | GPS coordinates, IP-based location | Privacy regulations |
| Device Fingerprints | User agent, device ID combinations | Tracking concerns |

#### LOW Sensitivity (Standard Protection)

| Data Type | Examples |
|-----------|----------|
| Operational Metrics | Response times, error counts |
| Aggregated Statistics | Order volumes, traffic patterns |
| Technical Metadata | Request paths, status codes |

### 3.3 Classification Gaps

| Gap | Risk | Recommendation |
|-----|------|----------------|
| Manual classification only | Human error, inconsistency | Implement automated PII detection |
| No classification audit | Drift over time | Quarterly review process |
| Static assignment | Changing requirements | Dynamic reclassification capability |

---

## 4. Access Control Assessment

### 4.1 Authentication Controls

| Control | Implementation | Assessment |
|---------|---------------|------------|
| Authentication Method | JWT tokens via SSO | **Strong** |
| Identity Provider | Okta integration | **Strong** |
| Session Management | Token-based, 24hr expiry | **Adequate** |
| MFA | Enforced via Okta | **Strong** |

### 4.2 Authorization Controls

#### Current Role Model

| Role | Intended Purpose | Current Permissions |
|------|-----------------|---------------------|
| SUPER_ADMIN | Full system control | All operations |
| ADMIN | Administrative functions | All operations |
| USER | Standard access | All read operations |
| EXTERNAL | Partner access | All read operations |

#### Authorization Gap Analysis

| Finding | Severity | Description |
|---------|----------|-------------|
| **No data-level authorization** | **Critical** | All authenticated users can access all log data regardless of sensitivity |
| **No role differentiation** | **High** | USER and EXTERNAL have identical permissions |
| **No query restrictions** | **High** | No limits on data volume or scope of queries |
| **Missing audit trail** | **Medium** | Sensitive data access not logged |

### 4.3 Access Control Recommendations

| Priority | Recommendation | Risk Mitigated |
|----------|---------------|----------------|
| **P0** | Restrict HIGH sensitivity logs to ADMIN+ roles | Unauthorized access to PII |
| **P0** | Implement audit logging for all sensitive queries | Detection, compliance |
| **P1** | Restrict EXTERNAL role to LOW sensitivity only | Partner data exposure |
| **P1** | Add query scope limits (date range, record count) | Data exfiltration |
| **P2** | Implement attribute-based access (by business domain) | Lateral data access |

---

## 5. Data Protection Controls

### 5.1 Data Masking

#### Current Implementation

| Capability | Status | Assessment |
|------------|--------|------------|
| Field-level masking | Implemented | **Strong** |
| JSON path support | Implemented | **Strong** |
| Configurable rules | Implemented | **Strong** |
| Multiple masking types | 5 types available | **Strong** |

#### Masking Types Available

| Type | Use Case | Example |
|------|----------|---------|
| Middle Mask | Phone numbers | 138****5678 |
| Prefix Mask | Email addresses | ***an@email.com |
| Suffix Mask | Addresses | 123 Main **** |
| Full Mask | Passwords, tokens | ******** |
| Pattern-based | Credit cards | 4111-****-****-1111 |

#### Masking Gap Analysis

| Finding | Severity | Description |
|---------|----------|-------------|
| **Query-time masking only** | **High** | Raw data stored unmasked in database |
| **No ingestion-time masking** | **High** | PII persists in storage layer |
| **DB admin access bypasses masking** | **Medium** | Direct database access exposes raw data |

### 5.2 Encryption Assessment

| Layer | Current State | Risk |
|-------|--------------|------|
| **In Transit (External)** | TLS 1.2+ enforced | Low |
| **In Transit (Internal)** | HTTP between services | Medium |
| **At Rest (Database)** | Not encrypted | **Medium** |
| **Field-level Encryption** | Not implemented | Medium |

### 5.3 Data Protection Recommendations

| Priority | Recommendation | Risk Mitigated |
|----------|---------------|----------------|
| **P0** | Implement pre-masking for HIGH fields at ingestion | Storage exposure |
| **P1** | Enable database encryption at rest | Physical media theft |
| **P1** | Encrypt internal service communication | Network sniffing |
| **P2** | Implement field-level encryption for reversible cases | Targeted data theft |

---

## 6. Data Isolation Analysis

### 6.1 The Question

Should sensitive logs be stored in a physically separate database instance?

### 6.2 Options Evaluated

#### Option A: Physical Separation (Separate Database Instance)

```
  ┌─────────────────────┐          ┌─────────────────────┐
  │  General Instance   │          │  Sensitive Instance │
  │                     │          │                     │
  │  • LOW/MEDIUM logs  │          │  • HIGH logs only   │
  │  • Standard network │          │  • Isolated network │
  │  • All users        │          │  • Restricted users │
  └─────────────────────┘          └─────────────────────┘
```

| Factor | Assessment |
|--------|------------|
| Security Benefit | Physical network isolation |
| Operational Cost | 2x infrastructure, monitoring, maintenance |
| Query Complexity | Cross-instance analysis impossible |
| Incident Response | Correlation analysis hindered |
| Cost Impact | ~100% increase in database costs |

#### Option B: Logical Separation (Current + Enhanced)

```
  ┌───────────────────────────────────────────────────────┐
  │              Single Database Instance                  │
  │                                                        │
  │   ┌─────────────────┐    ┌─────────────────┐         │
  │   │ General Schema  │    │ Sensitive Schema│         │
  │   │ (public access) │    │ (restricted)    │         │
  │   └─────────────────┘    └─────────────────┘         │
  │                                                        │
  │   Access Control: Database-level RBAC                 │
  │   Audit: Query logging enabled                        │
  └───────────────────────────────────────────────────────┘
```

| Factor | Assessment |
|--------|------------|
| Security Benefit | Logical isolation + RBAC |
| Operational Cost | Single system to manage |
| Query Complexity | Cross-schema queries when authorized |
| Incident Response | Full correlation capability |
| Cost Impact | Minimal additional cost |

### 6.3 Recommendation

**Maintain logical separation with enhanced controls.**

#### Justification

1. **Adequate Security**: Database RBAC provides sufficient access control for current threat model
2. **Operational Efficiency**: Single system reduces management overhead and potential for misconfiguration
3. **Incident Response**: Ability to correlate sensitive and non-sensitive logs is valuable for security investigations
4. **Cost Efficiency**: Avoids doubling infrastructure costs
5. **Precedent**: Industry standard for similar data classification requirements

#### When to Reconsider

- Regulatory mandate for physical separation
- Data residency requirements (geographic separation)
- Significant increase in HIGH sensitivity data volume
- Security incident requiring enhanced isolation

---

## 7. Risk Assessment

### 7.1 Threat Model

| Threat | Likelihood | Impact | Current Controls | Residual Risk |
|--------|------------|--------|------------------|---------------|
| Unauthorized data access (internal) | Medium | High | Authentication only | **High** |
| Unauthorized data access (external) | Low | Critical | Perimeter security | Medium |
| Data exfiltration by insider | Medium | High | None | **High** |
| Database compromise | Low | Critical | Network isolation | Medium |
| Log injection attack | Low | Medium | Input validation | Low |

### 7.2 Risk Matrix

```
                         IMPACT
                    Low    Med    High   Crit
              ┌─────┬─────┬─────┬─────┬─────┐
         High │     │     │     │     │     │
              ├─────┼─────┼─────┼─────┼─────┤
  LIKELIHOOD  │     │     │  ●  │     │     │  ● Insider exfiltration
         Med  │     │     │  ●  │     │     │  ● Unauthorized internal access
              ├─────┼─────┼─────┼─────┼─────┤
         Low  │     │     │     │  ●  │  ●  │  ● DB compromise, External attack
              └─────┴─────┴─────┴─────┴─────┘
```

### 7.3 Compliance Considerations

| Framework | Relevance | Current Gaps |
|-----------|-----------|--------------|
| SOC 2 | Access controls, audit logging | Authorization controls, audit trail |
| GDPR | Data minimization, access rights | No data retention enforcement for EU data |
| PCI-DSS | Cardholder data protection | Pre-masking needed for payment data |
| CCPA | Consumer data rights | No data deletion workflow |

---

## 8. Recommendations

### 8.1 Immediate Actions (0-30 days)

| # | Action | Owner | Risk Addressed |
|---|--------|-------|----------------|
| 1 | Implement role-based access restrictions for HIGH sensitivity logs | Engineering | Unauthorized access |
| 2 | Enable audit logging for all sensitive data queries | Engineering | Detection, compliance |
| 3 | Document and formalize data classification criteria | Security | Inconsistent classification |
| 4 | Review and restrict EXTERNAL role permissions | Security | Partner data exposure |

### 8.2 Short-term Actions (30-90 days)

| # | Action | Owner | Risk Addressed |
|---|--------|-------|----------------|
| 5 | Implement pre-masking for HIGH sensitivity fields at data ingestion | Engineering | Storage exposure |
| 6 | Configure database-level RBAC for sensitive schema | DevOps | Direct DB access |
| 7 | Implement query scope limits (record count, date range) | Engineering | Data exfiltration |
| 8 | Enable database encryption at rest | DevOps | Physical media theft |

### 8.3 Medium-term Actions (90-180 days)

| # | Action | Owner | Risk Addressed |
|---|--------|-------|----------------|
| 9 | Implement automated PII detection in log parser | Engineering | Human error in classification |
| 10 | Deploy field-level encryption for reversible sensitive data | Engineering | Targeted data theft |
| 11 | Establish quarterly classification review process | Security | Classification drift |
| 12 | Implement data retention enforcement automation | Engineering | Compliance |

### 8.4 Success Metrics

| Metric | Target | Measurement |
|--------|--------|-------------|
| HIGH sensitivity data accessible only to authorized roles | 100% | Access audit review |
| Sensitive data queries logged | 100% | Audit log completeness |
| HIGH sensitivity fields pre-masked at ingestion | 100% | Data sampling verification |
| Unauthorized access attempts detected | <24hr detection | SIEM alerting |

---

## Document Approval

| Role | Name | Date | Signature |
|------|------|------|-----------|
| Security Lead | | | |
| Engineering Lead | | | |
| Compliance Officer | | | |

---

## Revision History

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0 | 2026-01-12 | Engineering | Initial draft for security review |

---

*This document should be reviewed quarterly and updated when significant changes are made to the Monitor System's data handling architecture.*
