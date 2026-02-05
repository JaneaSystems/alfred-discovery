# AX3 Digital Transformation: Additional Discovery Questions

> **Supplement to**: AX3_Discovery_Questions.docx  
> **Date**: February 5, 2026  
> **Topics**: Reporting & Analytics, Security & Compliance

---

## Group 9: Reporting & Analytics

These questions help us understand current reporting needs and how the new digital product platform should support business intelligence.

### Current State

**9.1 What reports are currently generated for digital product sales and inventory?**

- [ ] Sales reports by product type
- [ ] Sales reports by channel (Alfred.com vs. dealers)
- [ ] Royalty reports for rights holders
- [ ] Inventory/catalog reports
- [ ] Upload/processing status reports
- [ ] Error/failure reports
- [ ] Other: _________________

**9.2 What tools are currently used for reporting?**

- [ ] SQL Server Reporting Services (SSRS)
- [ ] Excel spreadsheets (manual)
- [ ] Power BI
- [ ] FileMaker reports
- [ ] AS400 native reports
- [ ] Custom application dashboards
- [ ] Other: _________________

**9.3 Who are the primary consumers of these reports?**

| Report Type | Primary Audience |
|-------------|------------------|
| Sales reports | _________________ |
| Royalty reports | _________________ |
| Operations/status reports | _________________ |
| Error/exception reports | _________________ |

**9.4 How frequently are reports needed?**

- [ ] Real-time / on-demand
- [ ] Daily
- [ ] Weekly
- [ ] Monthly
- [ ] Quarterly
- [ ] Ad-hoc only

### Pain Points

**9.5 What are the biggest challenges with current reporting?**

- [ ] Data is spread across multiple systems
- [ ] Reports require manual data consolidation
- [ ] Reports are too slow to generate
- [ ] Data accuracy/consistency issues
- [ ] Lack of self-service reporting capability
- [ ] Missing metrics we need
- [ ] Other: _________________

**9.6 Are there reports you need but cannot currently produce?**

Please describe:
________________________________________________________________________
________________________________________________________________________

### Future State

**9.7 For the new Digital Product Platform, what dashboard/reporting capabilities are most important?**

Rate 1-5 (1=Not needed, 5=Critical):

| Capability | Priority (1-5) |
|------------|----------------|
| Real-time pipeline status (products in progress) | ___ |
| Upload success/failure rates | ___ |
| Processing time metrics | ___ |
| Sales by product type and channel | ___ |
| Error trend analysis | ___ |
| Audit trail / change history | ___ |
| Royalty calculation summaries | ___ |

**9.8 Should the new system integrate with existing reporting tools or provide its own dashboards?**

- [ ] Integrate with existing tools (specify): _________________
- [ ] Provide standalone dashboards
- [ ] Both - standalone with export capability
- [ ] No preference

**9.9 Who should have access to the new platform's dashboards?**

- [ ] Digital operations team only
- [ ] All internal staff
- [ ] Management/executives
- [ ] External partners (dealers, rights holders)
- [ ] Other: _________________

---

## Group 10: Security & Compliance

These questions ensure the new platform meets security requirements and regulatory obligations.

### Access Control

**10.1 What authentication system is used for internal applications?**

- [ ] Active Directory / Windows authentication
- [ ] Azure AD / Microsoft 365
- [ ] Custom username/password per application
- [ ] SSO solution (specify): _________________
- [ ] Other: _________________

**10.2 Should the new Digital Product Platform integrate with existing authentication?**

- [ ] Yes - must use existing SSO/AD
- [ ] Yes - preferred but not required
- [ ] No - standalone authentication acceptable
- [ ] No preference

**10.3 What role-based access levels are needed?**

| Role | Access Level Needed |
|------|---------------------|
| Digital Operations Staff | _________________ |
| Managers/Supervisors | _________________ |
| IT/Administrators | _________________ |
| Finance/Royalties | _________________ |
| External Partners | _________________ |

**10.4 Are there specific actions that should require elevated permissions?**

- [ ] Deleting products
- [ ] Overriding validation errors
- [ ] Modifying pricing
- [ ] Accessing royalty data
- [ ] Bulk operations
- [ ] Configuration changes
- [ ] Other: _________________

### Data Security

**10.5 Where should data for the new platform be stored?**

- [ ] On-premises servers only
- [ ] Company-managed cloud (Azure/AWS)
- [ ] Either on-premises or cloud acceptable
- [ ] Specific requirement: _________________

**10.6 Are there data residency requirements (data must stay in specific geography)?**

- [ ] Yes - US only
- [ ] Yes - specific region: _________________
- [ ] No geographic restrictions
- [ ] Unknown

**10.7 What is the data classification of digital product information?**

- [ ] Public / non-sensitive
- [ ] Internal use only
- [ ] Confidential (business sensitive)
- [ ] Highly confidential (regulated)
- [ ] Mixed - depends on data type

**10.8 Are there encryption requirements?**

| Data State | Encryption Required? |
|------------|---------------------|
| Data at rest (stored) | [ ] Yes [ ] No [ ] Unknown |
| Data in transit (network) | [ ] Yes [ ] No [ ] Unknown |
| Backups | [ ] Yes [ ] No [ ] Unknown |

### Compliance & Audit

**10.9 What compliance frameworks apply to your business?**

- [ ] SOX (Sarbanes-Oxley)
- [ ] PCI-DSS (payment card data)
- [ ] GDPR (EU data privacy)
- [ ] CCPA (California privacy)
- [ ] SOC 2
- [ ] None specifically
- [ ] Other: _________________

**10.10 What audit trail requirements exist?**

- [ ] Log all user actions
- [ ] Log data changes only
- [ ] Log access to sensitive data
- [ ] Retain logs for ___ months/years
- [ ] No specific requirements

**10.11 Are there regular security audits or penetration testing requirements?**

- [ ] Yes - annual audits
- [ ] Yes - more frequent than annual
- [ ] No formal requirement
- [ ] Unknown

**10.12 Who is the appropriate contact for security and compliance questions?**

Name: _________________  
Role: _________________  
Email: _________________

### Third-Party Integrations

**10.13 Are there security requirements for third-party integrations (AWS, Dropbox, etc.)?**

- [ ] Must use company-approved vendors only
- [ ] Security review required for new integrations
- [ ] Data sharing agreements required
- [ ] No specific requirements
- [ ] Other: _________________

**10.14 Are API credentials and secrets managed through a central system?**

- [ ] Yes - secret management tool (specify): _________________
- [ ] Yes - secure configuration files
- [ ] No - managed per application
- [ ] Unknown

---

## Additional Notes

Please provide any additional context about reporting needs or security requirements:

________________________________________________________________________
________________________________________________________________________
________________________________________________________________________
________________________________________________________________________

---

*End of Additional Questions*
