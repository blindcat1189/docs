# Chowbus - Newo.ai Agreement: Technical Requirements

> Contract language for technical obligations (Revised based on industry standards)

---

## 1.3 Newo.ai Technical Obligations

Newo.ai shall:

### 1.3.1 API Access
- Provide REST API access to AI Agent platform
- Provide API documentation (OpenAPI/Swagger specification)
- Maintain API versioning with minimum 12-month deprecation notice for breaking changes
- Maintain backward compatibility for non-breaking changes
- Support minimum 1,000 API requests per minute per restaurant

### 1.3.2 Integration Support
- Provide technical support with the following response and resolution times:

| Priority | Definition | Response | Resolution Target |
|----------|------------|----------|-------------------|
| Critical | Service unavailable | 1 hour | 4 hours |
| High | Service degraded | 4 hours | 8 hours |
| Medium | Feature impaired | 8 hours | 48 hours |
| Low | General inquiry | 24 hours | 5 business days |

### 1.3.3 Agent Provisioning
- Provision new AI agent within 5 minutes of API request
- Support bulk provisioning for multi-location restaurants
- Provide rollback capability within 24 hours of provisioning

### 1.3.4 White Label
- Enable full UI rebranding without Newo.ai attribution
- Provide customizable scripts (greeting, closing, error handling)
- Support custom voice/persona configuration

### 1.3.5 Reporting and Analytics
- Provide real-time dashboard for call metrics and performance
- Provide API access to usage and analytics data
- Retain reporting data for minimum 24 months

### 1.3.6 Multilingual Support
- Support English, Chinese (Mandarin), and Spanish at launch
- Maintain equivalent accuracy (within 5%) across all supported languages
- Additional languages by mutual written agreement

---

## 1.4 Integration Requirements

### 1.4.1 Data Synchronization

| Data Type | Sync Method | Max Latency | Retry Policy |
|-----------|-------------|-------------|--------------|
| Restaurant info | Pull + Webhook | 5 minutes | 3 retries, exponential backoff |
| Menu structure | Pull + Webhook | 5 minutes | 3 retries, exponential backoff |
| Item availability | Webhook | 30 seconds | 3 retries, exponential backoff |
| Price changes | Webhook | 60 seconds | 3 retries, exponential backoff |
| Order status | Webhook | 5 seconds | 3 retries, exponential backoff |

### 1.4.2 Order Submission
- Submit orders to Chowbus API within 2 seconds of customer confirmation
- Include idempotency key (UUID v4) with each order request
- Implement retry logic: 3 attempts with exponential backoff (1s, 2s, 4s)
- Log all order attempts and responses for minimum 90 days

### 1.4.3 Webhook Implementation
- Process incoming webhooks within 5 seconds
- Return HTTP 2xx for successful receipt
- Verify webhook signatures using HMAC-SHA256
- Implement idempotency handling for duplicate webhooks

### 1.4.4 Phone Service
- Provision dedicated phone number per restaurant location
- Phone numbers owned by Chowbus, managed by Newo.ai
- Support number porting within 30 days of termination request
- Implement call forwarding to designated backup number during outages

### 1.4.5 Customer Notifications
- Send order confirmation within 30 seconds of successful submission
- Send status update within 60 seconds of webhook receipt
- Support SMS and/or voice callback as configured per restaurant

---

## 1.5 Service Level Agreement

### 1.5.1 Availability Targets

| Service | Monthly Uptime | Max Downtime/Month |
|---------|----------------|-------------------|
| AI Agent Platform | 99.9% | 43 minutes |
| Management Portal | 99.5% | 3.6 hours |
| Phone/Telephony | 99.99% | 4.3 minutes |

**Measurement**: Uptime measured by Newo.ai monitoring systems. Chowbus may request third-party verification.

### 1.5.2 SLA Exclusions

The following do not count toward downtime:
- Scheduled maintenance (with 72-hour notice)
- Chowbus system unavailability
- Force majeure events
- Issues caused by Chowbus's misuse of the service
- Internet connectivity issues outside Newo.ai's network

### 1.5.3 Performance Targets

| Metric | Target | Measurement |
|--------|--------|-------------|
| AI voice response latency | < 1.5 seconds (p95) | Time to first AI response |
| Order API response | < 2 seconds (p95) | API round-trip time |
| Webhook processing | < 5 seconds (p95) | Receipt to acknowledgment |
| Speech recognition accuracy | > 95% | Monthly sample audit |
| Order accuracy | > 99% | Orders without errors / total |

### 1.5.4 Service Credits

| Monthly Uptime | Credit (% of Monthly Fee) |
|----------------|---------------------------|
| 99.5% - 99.9% | 5% |
| 99.0% - 99.5% | 10% |
| 95.0% - 99.0% | 25% |
| < 95.0% | 50% |

**Credit Cap**: Maximum service credits shall not exceed 50% of monthly fees in any calendar month.

**Claim Process**: Chowbus must submit credit requests within 30 days of the incident. Newo.ai shall respond within 14 days.

**Sole Remedy**: Service credits are Chowbus's sole and exclusive remedy for failure to meet SLA targets, except in cases of gross negligence or willful misconduct.

### 1.5.5 Scheduled Maintenance

- Minimum 72-hour advance notice (email to designated contact)
- Preferred window: 2:00 AM - 6:00 AM Pacific Time
- Maximum 4 hours scheduled maintenance per calendar month
- Emergency maintenance permitted without notice; notification within 1 hour

---

## 1.6 Security Requirements

### 1.6.1 Data Transmission
- TLS 1.2 or higher for all API and webhook communication
- Certificate pinning recommended for mobile/embedded clients
- HMAC-SHA256 minimum for webhook signature verification

### 1.6.2 Data Storage
- Encryption at rest using AES-256 or equivalent
- Encryption keys managed via hardware security module (HSM) or equivalent
- Database access restricted to authorized personnel only

### 1.6.3 Access Control
- Role-based access control (RBAC) for all systems
- Multi-factor authentication (MFA) required for:
  - Administrative access
  - Production system access
  - Customer data access
- Unique user accounts (no shared credentials)
- Access reviews conducted quarterly

### 1.6.4 Audit Logging
- Log all access to customer data
- Log all administrative actions
- Log retention: minimum 12 months
- Logs tamper-evident and centrally stored

### 1.6.5 Security Certifications
- **SOC 2 Type II**: Maintain current certification; provide report annually
- **Penetration Testing**: Conduct annually by qualified third party; share executive summary
- **Vulnerability Scanning**: Conduct at least quarterly; remediate critical findings within 7 days

### 1.6.6 Incident Response
- Maintain documented incident response plan
- Security incident notification to Chowbus within 24 hours of discovery
- Post-incident report within 7 days of resolution

---

## 1.7 Data Protection

### 1.7.1 Data Ownership
- **Customer Data**: All data collected from end customers (name, phone, orders) is owned by Chowbus
- **Usage Data**: Aggregated, anonymized usage statistics may be retained by Newo.ai
- **AI Models**: Newo.ai owns AI models; Chowbus data shall not be used to train models for other customers without anonymization

### 1.7.2 Data Processing
- Newo.ai acts as **Data Processor**; Chowbus is **Data Controller**
- Processing only as instructed by Chowbus and as necessary to provide the service
- Data Processing Agreement (DPA) attached as Exhibit A

### 1.7.3 Subprocessors
- Newo.ai shall maintain list of subprocessors and provide upon request
- 30-day advance notice before engaging new subprocessors
- Chowbus may object to new subprocessors with reasonable grounds

### 1.7.4 Data Retention

| Data Type | Retention Period | Deletion Trigger |
|-----------|------------------|------------------|
| Call recordings | 90 days | Auto-delete after period |
| Call transcripts | 90 days | Auto-delete after period |
| Order data | 30 days | After transmission to Chowbus |
| Customer PII | 30 days | After transaction completion |
| Aggregated analytics | 24 months | N/A (anonymized) |

### 1.7.5 Data Subject Rights
- Support Chowbus in responding to data subject requests (access, deletion, portability)
- Response within 72 hours of Chowbus request
- No additional charge for reasonable volume of requests

### 1.7.6 Breach Notification
- Notify Chowbus of confirmed data breach within **24 hours** of discovery
- Notify Chowbus of suspected breach within **48 hours**
- Notification shall include: nature of breach, data affected, remediation steps, timeline

---

## 1.8 Business Continuity

### 1.8.1 Disaster Recovery

| Metric | Target |
|--------|--------|
| Recovery Time Objective (RTO) | 4 hours |
| Recovery Point Objective (RPO) | 1 hour |

### 1.8.2 Backup
- Daily backups of all configuration and customer data
- Backups stored in geographically separate location
- Backup restoration tested quarterly

### 1.8.3 Redundancy
- No single point of failure for critical services
- Multi-region or multi-availability-zone deployment for AI Agent platform

---

## 1.9 Audit Rights

### 1.9.1 Audit Scope
Chowbus may audit Newo.ai's compliance with:
- Security requirements (Section 1.6)
- Data protection obligations (Section 1.7)
- Service level commitments (Section 1.5)

### 1.9.2 Audit Process
- **Frequency**: Once per calendar year, or upon reasonable suspicion of breach
- **Notice**: 30 days advance written notice (except for suspected breach)
- **Conduct**: During normal business hours, minimizing disruption
- **Cost**: Each party bears its own costs
- **Confidentiality**: Audit findings treated as confidential information

### 1.9.3 Third-Party Audits
- Chowbus may rely on SOC 2 Type II report in lieu of direct audit
- Chowbus may engage qualified third-party auditor

---

## 1.10 Termination Obligations

Upon termination or expiration, Newo.ai shall:

| Obligation | Timeline |
|------------|----------|
| Continue service (wind-down period) | 90 days from notice |
| Provide data export (call logs, transcripts, analytics) | Within 14 days of request |
| Port phone numbers to Chowbus-designated carrier | Within 30 days of request |
| Delete all Chowbus customer data | Within 30 days of termination |
| Provide written certification of deletion | Within 30 days of deletion |
| Return or destroy confidential information | Within 30 days of termination |

### 1.10.1 Transition Assistance
- Newo.ai shall provide reasonable transition assistance during wind-down
- Assistance includes: data export, documentation, technical support
- Additional fees may apply for assistance exceeding 20 hours

---

## Definitions

| Term | Definition |
|------|------------|
| **Downtime** | Period when service is unavailable, excluding SLA Exclusions |
| **Monthly Uptime** | (Total minutes - Downtime minutes) / Total minutes × 100% |
| **p95** | 95th percentile of measurements |
| **Customer Data** | Data about end customers collected through the service |
| **PII** | Personally Identifiable Information |
| **RTO** | Recovery Time Objective - max time to restore service after disaster |
| **RPO** | Recovery Point Objective - max acceptable data loss measured in time |

---

## Exhibits

- **Exhibit A**: Data Processing Agreement (DPA)
- **Exhibit B**: List of Subprocessors
- **Exhibit C**: Security Whitepaper
- **Exhibit D**: API Documentation Reference

---

*Revised based on industry standards and best practices*
*Sources: [Spendflo SaaS SLA Guide](https://www.spendflo.com/blog/saas-service-level-agreements-sla), [Enterprise Ready SLA Guide](https://www.enterpriseready.io/features/sla-support/), [Secureframe SOC 2 Guide](https://secureframe.com/blog/soc-2-type-ii), [Promise Legal SaaS Agreements](https://www.promise.legal/startup-legal-guide/contracts/saas-agreements)*
