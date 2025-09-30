# Financial Transaction Monitoring
Batch pipeline, daily 2 TB of payment logs from banks and fintech companies across Canada and India. Overall, each layer of the pipeline keeps its operations to its jurisdiction, and data that crosses borders are anonymized and/or aggregated. Guardrails ensure complete, immutable accounts of logs compliant with privacy and security requirements. System is vigilant in checking failures in delivery and access as well as detecting anomalies.

## Architecture Diagram
<img src="architecture.png">

### Ingest Layer

Goal: Securely collect 2 TB/day of payment logs from Canada, India, UK.  
- S3/Blob ingestion endpoints per region (Canada/India/UK)
- Kafka / Kinesis / PubSub for streaming as an option if some logs arrive continuously.
- TLS + mutual authentication for API endpoints
- Region-specific ingress to enforce data sovereignty

### Storage Layer

Goal: Store raw logs securely for batch processing, compliant with retention requirements.  
- S3 data lakes per jurisdiction
- Encryption at rest: KMS-managed keys, region-specific
- Access controls: Role-based access to prevent cross-jurisdiction leakage
- Retention rules: Apply per country (e.g., Canada 7 years, India 5 years, UK 6 years)

### Transform Layer

Goal: Normalize, enrich, anonymize, and generate AML insights from the raw logs.  
Compute: Cloud-managed Spark / Databricks / EMR / Dataflow jobs per region 

Steps:
1. Validation & cleaning
2. Jurisdiction-aware transformations  
   Mask PII for unbanked populations, currency conversion, flag suspicious patterns
3. Aggregation / analytics
4. Metadata tagging (source country, retention deadline)

### Publish / Reporting Layer

Goal: Deliver processed data to regulators, AML teams, and dashboards. 
- Country-specific AML submissions (Canada FINTRAC, India FIU-IND, UK FCA/NCA)
- Internal dashboards (aggregated analytics, alerts)
- Anonymized/aggregated data if global reporting needed
- Federated analytics to avoid raw data movement

### Observability / Monitoring & Guardrails (Cross-cutting)

Goal: Ensure reliability, compliance, and privacy. Raw data never leaves its country; only aggregated/anonymized insights can cross borders if global reporting is needed. 
- Ingest: Schema enforcement, TLS validation, failed delivery alerts
- Storage: Bucket access logs, object immutability, retention compliance
- Transform: Job success/fail metrics, data quality, PII masking verification, anomaly detection
- Publish: Delivery logs, dashboard access logs, AML submission audit trail
- Security / Privacy: KMS key rotation, IAM monitoring, differential privacy checks

## Clause -> Control -> Test
Ensures compliance with regulations from the countries involved.  
- Canada: Personal Information Protection and Electronic Documents Act (PIPEDA).  
- India: Digital Personal Data Protection (DPDP).  
- Global: Anti-Money Laundering (AML) regulations.

| Guardrail / Clause (Source)                                                                                                                      | AWS Service / Tool                                                   | Plan to Test                                                                                                                        |
| ------------------------------------------------------------------------------------------------------------------------------------------------ | -------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------- |
| **Data residency / localization** (Canada PIPEDA, India DPDP) – Personal and financial data must remain within jurisdiction.                     | S3 buckets in specific regions, Lake Formation row-level permissions | Attempt cross-region read/write; verify access denied. Audit CloudTrail logs.                                                       |
| **Encryption at rest & in transit** (PIPEDA Sec 4.7, DPDP Sec 11) – All data must be encrypted.                                                  | KMS CMKs, SSE-KMS on S3, gRPC + mTLS                                 | Upload test data, verify encryption keys; attempt access without key/role and confirm denial.                                       |
| **Access control / least privilege** (PIPEDA, DPDP) – Only authorized users/jobs can read/write data.                                            | Lake Formation, IAM policies                                         | Test with unauthorized user/job trying to read/write; ensure access denied.                                                         |
| **Consent enforcement** (DPDP Sec 8) – Process personal data only if consent exists.                                                             | ETL jobs, Glue catalog metadata                      | Load dataset with missing consent; ETL job should skip processing; audit logs show skipped rows.                                    |
| **Data minimization & masking** (PIPEDA, DPDP) – Sensitive fields must be masked or tokenized for dashboards and analytics.                      | Glue/EMR/Databricks ETL + Lake Formation column-level permissions    | Run ETL on test dataset; confirm sensitive columns masked; attempt unauthorized read.                                               |
| **Auditability & retention** (PIPEDA, DPDP, AML regulations) – Maintain logs of all processing and AML reporting for required retention periods. | CloudTrail + S3 lifecycle + Lake Formation lineage                   | Simulate retention lifecycle; verify logs accessible and cannot be deleted prematurely; check lineage metadata for transformations. |
| **Data lineage & transformation traceability** (AML reporting guidelines, DPDP Sec 9) – Track how raw data flows to curated outputs.             | Lake Formation lineage + Glue Data Catalog                           | Run test ETL; verify output columns have lineage metadata showing source columns/tables.                                            |
| **Monitoring & alerting** (PIPEDA Sec 4.7, DPDP Sec 11) – Detect anomalous access or processing errors.                                          | CloudWatch metrics, alarms, Macie for PII scanning                   | Inject simulated anomalies (e.g., large unauthorized read); verify alerts trigger and logs recorded.                                |
