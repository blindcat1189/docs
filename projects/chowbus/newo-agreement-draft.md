# Chowbus - Newo.ai Agreement (Draft)

> Refined contract terms from Chowbus legal perspective

---

## 1.3 Newo.ai Obligations

Newo.ai agrees to:

### 1.3.1 API Access and Documentation
- Provide secure API access to AI Agent platform for Chowbus integration
- Provide comprehensive API documentation, including:
  - OpenAPI/Swagger specification
  - Authentication and authorization protocols
  - Error codes and handling procedures
  - Rate limits and usage guidelines
- Maintain API versioning with minimum 12-month deprecation notice for breaking changes

### 1.3.2 Integration Support
- Provide dedicated integration support during implementation phase
- Provide ongoing technical support with the following response times:
  - **Critical (service down)**: 1-hour response, 4-hour resolution
  - **High (degraded service)**: 4-hour response, 24-hour resolution
  - **Medium (non-critical)**: 24-hour response, 72-hour resolution
  - **Low (general inquiry)**: 48-hour response
- Assign dedicated technical account manager for Chowbus

### 1.3.3 One-Click Creator
- Provide One-Click Creator interface for rapid restaurant onboarding
- Maintain 99.9% availability for One-Click Creator platform
- Support bulk onboarding for multi-location restaurant chains

### 1.3.4 Escalation Protocol
- Follow the escalation protocol as described in Section 3.2
- Provide 24/7 escalation path for critical issues
- Conduct quarterly escalation procedure reviews with Chowbus

### 1.3.5 White Label Solution
- Permit Chowbus to white label Newo.ai's Technology, allowing Chowbus to:
  - Rebrand and customize the user interface
  - Present Newo.ai Technology capabilities as Chowbus's own offering
  - Remove all Newo.ai attribution and branding
- White label rights shall survive termination for existing restaurant deployments for a period of 12 months
- Provide source code escrow with release conditions including Newo.ai bankruptcy, material breach, or discontinuation of service

### 1.3.6 Reporting
- Provide real-time dashboard access for:
  - Active Locations count
  - Call volume and duration metrics
  - Order success/failure rates
  - AI accuracy metrics
- Provide daily automated usage reports via API or email
- Provide monthly billing reconciliation reports within 5 business days of month end
- Retain reporting data for minimum 24 months

### 1.3.7 Multilingual Support
- Ensure AI Agents support the following languages at launch:
  - English (US)
  - Chinese (Mandarin and Cantonese)
  - Spanish
- Additional languages may be added by mutual written agreement
- Maintain equivalent accuracy rates across all supported languages

---

## 1.4 Integration Services

### 1.4.1 Core Integrations

Newo.ai shall provide and maintain the following integrations:

| Integration | Description | SLA |
|-------------|-------------|-----|
| **One-Click Creator** | Rapid AI agent deployment for new restaurants | < 5 minutes per location |
| **Menu Synchronization** | Bi-directional sync with Chowbus catalog | < 30 seconds sync latency |
| **Real-time Pricing** | Price updates from Chowbus to AI Agent | < 5 seconds propagation |
| **Order Submission** | Order placement to Chowbus POS | < 2 seconds response time |
| **Order Status** | Status updates from Chowbus to Newo.ai | Webhook delivery < 5 seconds |
| **Restaurant Info** | Location, hours, capabilities sync | < 1 minute sync latency |

### 1.4.2 Menu Synchronization
- Pull full menu from Chowbus on initial setup and daily reconciliation
- Receive real-time webhook updates for:
  - Item availability changes
  - Price changes
  - Menu structure changes
- Handle menu sync failures with automatic retry (3 attempts, exponential backoff)
- Notify Chowbus of persistent sync failures within 15 minutes

### 1.4.3 Order Submission
- Submit orders to Chowbus POS API in real-time
- Include idempotency key with each order to prevent duplicates
- Handle order submission failures:
  - Retry up to 3 times with exponential backoff
  - Notify customer of failure if all retries exhausted
  - Log all failures for reconciliation
- Provide order confirmation callback to customer within 30 seconds

### 1.4.4 Customer Data Management
- Collect only minimum necessary customer data:
  - Name
  - Phone number
  - Order details
  - Special instructions
- All customer data shall be:
  - Transmitted to Chowbus immediately upon collection
  - Not retained by Newo.ai beyond 30 days except as required for dispute resolution
  - Subject to the Data Protection terms in Section 4
- Newo.ai shall NOT:
  - Use customer data for marketing without explicit consent
  - Share customer data with third parties
  - Train AI models on identifiable customer data without anonymization

### 1.4.5 Phone Number Provisioning
- Provision dedicated AI phone numbers for each restaurant location
- Phone numbers shall be:
  - Owned by Chowbus
  - Portable to another provider upon termination with 30-day notice
  - Maintained with 99.99% availability
- Provide call forwarding to restaurant backup number during outages
- Support SMS capabilities where available

### 1.4.6 Management Portal
- Provide web-based management portal for Chowbus administrators
- Portal shall include:
  - Real-time monitoring of all active AI agents
  - Call recordings and transcripts (retained for 90 days)
  - Performance analytics and reporting
  - Configuration management for AI behavior
  - User access control with role-based permissions
- Provide Management Portal API for programmatic access

### 1.4.7 Third-Party Integrations
- Google Maps integration for restaurant location data
  - Newo.ai shall bear Google Maps API costs
- Additional third-party integrations subject to:
  - Mutual written agreement
  - Cost allocation agreement
  - Security review by Chowbus

### 1.4.8 Multi-Location Support
- Support restaurant chains with multiple locations
- Provide chain-level reporting and management
- Support location-specific menu and pricing variations
- Enable bulk operations for chain-wide changes

---

## 1.5 Service Level Agreement (NEW)

### 1.5.1 Availability
| Service | Target | Measurement Period |
|---------|--------|-------------------|
| AI Agent Platform | 99.9% | Monthly |
| Order Submission API | 99.95% | Monthly |
| Management Portal | 99.5% | Monthly |
| One-Click Creator | 99.5% | Monthly |

### 1.5.2 Performance
| Metric | Target |
|--------|--------|
| AI response latency | < 2 seconds |
| Order submission latency | < 2 seconds |
| Menu sync latency | < 30 seconds |
| Webhook delivery | < 5 seconds |

### 1.5.3 Accuracy
| Metric | Target |
|--------|--------|
| Order accuracy rate | > 99% |
| Speech recognition accuracy | > 95% |
| Intent recognition accuracy | > 95% |

### 1.5.4 Service Credits
| Uptime | Service Credit |
|--------|----------------|
| 99.0% - 99.9% | 10% of monthly fee |
| 95.0% - 99.0% | 25% of monthly fee |
| < 95.0% | 50% of monthly fee |

Service credits shall be applied to the following month's invoice. Chowbus may terminate for cause if uptime falls below 95% for two consecutive months.

### 1.5.5 Scheduled Maintenance
- Newo.ai shall provide 72-hour notice for scheduled maintenance
- Scheduled maintenance shall occur during off-peak hours (2:00 AM - 6:00 AM local time)
- Scheduled maintenance not to exceed 4 hours per month
- Scheduled maintenance excluded from uptime calculations if notice provided

---

## 1.6 Data Protection (NEW)

### 1.6.1 Data Ownership
- **Chowbus Data**: All customer data, order data, restaurant data, and usage data collected through the integration remains the sole property of Chowbus
- **Newo.ai Data**: Newo.ai retains ownership of its AI models, algorithms, and platform technology (excluding Chowbus-specific customizations)

### 1.6.2 Data Processing
- Newo.ai acts as a **Data Processor** on behalf of Chowbus (Data Controller)
- Newo.ai shall process data only as instructed by Chowbus
- A Data Processing Agreement (DPA) shall be attached as Exhibit A

### 1.6.3 Data Security
Newo.ai shall implement and maintain:
- Encryption in transit: TLS 1.2 or higher
- Encryption at rest: AES-256 or equivalent
- Access controls: Role-based access, principle of least privilege
- Authentication: Multi-factor authentication for all administrative access
- Logging: Comprehensive audit logging retained for 12 months

### 1.6.4 Compliance
Newo.ai shall maintain compliance with:
- SOC 2 Type II (provide annual report)
- GDPR (for EU customers)
- CCPA (for California customers)
- PCI-DSS Level 1 (if handling payment data)

### 1.6.5 Audit Rights
- Chowbus may audit Newo.ai's data practices annually with 30-day notice
- Chowbus may request immediate audit in case of suspected breach
- Newo.ai shall cooperate fully with audits at no additional cost

### 1.6.6 Breach Notification
- Newo.ai shall notify Chowbus of any data breach within 24 hours of discovery
- Notification shall include: nature of breach, data affected, remediation steps
- Newo.ai shall cooperate with Chowbus's breach response procedures

### 1.6.7 Data Retention and Deletion
- Call recordings: 90 days (unless longer required by law)
- Order data: Transmit to Chowbus immediately, delete from Newo.ai systems within 30 days
- Customer PII: Delete within 30 days of order completion
- Upon termination: Delete all Chowbus data within 30 days, provide deletion certification

---

## 1.7 Security Requirements (NEW)

### 1.7.1 Security Certifications
Newo.ai shall maintain:
- SOC 2 Type II certification (provide annual report)
- ISO 27001 certification (if available)

### 1.7.2 Security Testing
- Annual third-party penetration testing (share executive summary with Chowbus)
- Quarterly vulnerability scanning
- Remediation of critical vulnerabilities within 24 hours
- Remediation of high vulnerabilities within 7 days

### 1.7.3 Secure Development
- Secure software development lifecycle (SDLC)
- Code review for all changes
- Automated security scanning in CI/CD pipeline
- Separation of development, staging, and production environments

### 1.7.4 Infrastructure Security
- Production infrastructure hosted in SOC 2 compliant data centers
- Network segmentation and firewalls
- DDoS protection
- Regular security patching (critical patches within 48 hours)

### 1.7.5 Personnel Security
- Background checks for employees with data access
- Security awareness training for all employees
- Confidentiality agreements for all employees and contractors

---

## 1.8 Insurance (NEW)

Newo.ai shall maintain the following insurance coverage:

| Coverage Type | Minimum Amount |
|---------------|----------------|
| Cyber Liability | $2,000,000 per occurrence |
| Errors & Omissions | $2,000,000 per occurrence |
| General Liability | $1,000,000 per occurrence |

- Chowbus shall be named as additional insured
- Newo.ai shall provide certificates of insurance upon request
- 30-day notice required before cancellation or material change

---

## 1.9 Termination (NEW)

### 1.9.1 Termination for Convenience
Either party may terminate with 90 days written notice.

### 1.9.2 Termination for Cause
Either party may terminate immediately upon:
- Material breach not cured within 30 days of written notice
- Bankruptcy or insolvency of the other party
- Failure to maintain required insurance or certifications

### 1.9.3 Effect of Termination
Upon termination, Newo.ai shall:

| Obligation | Timeline |
|------------|----------|
| Continue service during wind-down | 90 days |
| Export all Chowbus data in standard format | 14 days |
| Port phone numbers to Chowbus-designated provider | 30 days |
| Delete all Chowbus data | 30 days |
| Provide deletion certification | 30 days |
| Return or destroy confidential information | 30 days |

### 1.9.4 Survival
The following sections survive termination:
- Data Protection (Section 1.6)
- Confidentiality
- Indemnification
- Limitation of Liability
- White label rights for existing deployments (12 months)

---

## 1.10 Indemnification (NEW)

### 1.10.1 Newo.ai Indemnification
Newo.ai shall indemnify, defend, and hold harmless Chowbus from:
- Claims arising from AI Agent errors (wrong orders, incorrect pricing, etc.)
- Claims arising from data breaches caused by Newo.ai
- Claims of IP infringement related to Newo.ai technology
- Claims arising from Newo.ai's violation of applicable laws

### 1.10.2 Chowbus Indemnification
Chowbus shall indemnify, defend, and hold harmless Newo.ai from:
- Claims arising from Chowbus's menu data or pricing errors
- Claims arising from Chowbus's violation of applicable laws
- Claims of IP infringement related to Chowbus technology

### 1.10.3 Indemnification Procedures
- Prompt written notice of claim
- Control of defense by indemnifying party
- Cooperation by indemnified party
- No settlement without consent of indemnified party

---

## Exhibits

- **Exhibit A**: Data Processing Agreement (DPA)
- **Exhibit B**: Service Level Agreement Details
- **Exhibit C**: Security Requirements
- **Exhibit D**: API Documentation
- **Exhibit E**: Pricing Schedule

---

*Draft prepared by Chowbus Legal Team*
*Version: 1.0*
*Date: January 2026*
