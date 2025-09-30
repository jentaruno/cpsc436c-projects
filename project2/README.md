# Financial Transaction Monitoring
Batch pipeline, daily 2 TB of payment logs from banks and fintech companies across Canada/India/UK. Overall, each layer of the pipeline keeps its operations to its jurisdiction, and data that crosses borders are anonymized and/or aggregated. Guardrails ensure complete, immutable accounts of logs compliant with privacy and security requirements. System is vigilant in checking failures in delivery and access as well as detecting anomalies.

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
