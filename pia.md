# Privacy Impact Assessment

### Overview
- Project name, purpose, and scope.
- Data processing summary (collection → processing → sharing → retention).

### Data Inventory
- Fields collected (raw and derived).
- For each field: purpose, lawful basis (if applicable), minimization, retention, access roles.

### Linkability & Identifiability
- Can events be linked across sessions/users? How? (e.g., referrer+time+token)
- Quasi-identifiers present (IP, UA, location) and re-identification risks.

### Purpose Limitation & Secondary Use
- Declared purposes. Process to approve new purposes.
- Protections against function creep.

### Minimization & Retention
- Aggregation/sampling strategies; bucketing/truncation; hashing/anonymization.
- Raw TTL (days); aggregate retention; secure deletion plan.

### Access Control & Governance
- Roles and least-privilege policies. Auditability of access.
- Third parties involved (if any) and contracts