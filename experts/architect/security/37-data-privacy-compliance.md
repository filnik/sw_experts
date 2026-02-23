# Data Privacy and Compliance

## Profile

### What It Is
Data Privacy and Compliance encompass the technical and organizational measures to protect personal data, meet regulatory requirements (GDPR, CCPA, HIPAA), and build systems that respect user privacy by design. This includes data classification, consent management, data minimization, encryption, access controls, and audit trails.

### Origin and Evolution
- EU Data Protection Directive (1995) — first major data privacy regulation
- GDPR (2018) — global gold standard for data privacy, with significant penalties
- CCPA/CPRA (California, 2020) — US privacy regulation
- HIPAA (healthcare), PCI-DSS (payments), SOC 2 (service organizations)
- Current: global privacy law proliferation, privacy engineering as a discipline

### Key Authors and References
- **Ann Cavoukian** — Privacy by Design principles
- **European Commission** — GDPR text and guidelines
- **NIST** — Privacy Framework (2020)
- **Daniel Solove** — *Understanding Privacy*

### Key Concepts at a Glance
| Concept | Description |
|---------|-------------|
| Privacy by Design | Build privacy into systems from the start, not as an afterthought |
| Data Minimization | Collect and retain only data that's necessary |
| Purpose Limitation | Use data only for the purpose it was collected for |
| Consent Management | Explicit, informed, revocable user consent |
| Right to be Forgotten | Users can request deletion of their personal data |
| Data Protection Impact Assessment | Formal risk assessment for high-risk processing |

---

## Core Philosophy

### The 5 Fundamental Principles
1. **Privacy by design, not by policy** — Privacy controls must be implemented in the system architecture, not just documented in a privacy policy.
2. **Data minimization** — Collect the minimum data necessary. If you don't have the data, you can't leak it.
3. **Purpose limitation** — Data collected for purpose A should not be used for purpose B without explicit consent.
4. **User control** — Users should be able to access, correct, export, and delete their personal data.
5. **Accountability** — Maintain audit trails, data processing records, and impact assessments demonstrating compliance.

### When to Use vs. When to Avoid

**Privacy engineering is always relevant when:**
- Processing personal data of any kind
- Serving EU users (GDPR applies regardless of company location)
- Handling health, financial, or children's data
- Sharing data with third parties

**Investment scales with:**
- Volume and sensitivity of personal data
- Number of jurisdictions served
- Regulatory requirements of the industry
- Risk appetite and penalty exposure

---

## Frameworks and Methodologies

### 1. Privacy by Design Implementation — step-by-step
1. Classify all data: personal, sensitive, public, internal
2. Map data flows: collection → storage → processing → sharing → deletion
3. Apply data minimization: remove unnecessary data collection
4. Implement consent management: collection, storage, enforcement
5. Add data subject rights: access, rectification, deletion, portability
6. Set up retention policies and automated deletion
7. Conduct Data Protection Impact Assessment (DPIA) for high-risk processing

### 2. Data Subject Rights Implementation — step-by-step
1. Build a data subject request (DSR) API/portal
2. Implement data access: gather all data for a user across systems
3. Implement data portability: export in machine-readable format (JSON, CSV)
4. Implement right to erasure: delete or anonymize all personal data
5. Implement consent withdrawal: stop processing and propagate to third parties
6. Set SLAs for DSR fulfillment (GDPR: 30 days)

---

## How to Apply

### Decision Checklist
- [ ] Is all personal data classified and documented in a data inventory?
- [ ] Is consent collected before data processing (where required)?
- [ ] Can users access, export, and delete their data?
- [ ] Are retention policies defined and automated?
- [ ] Is data encrypted at rest and in transit?
- [ ] Are audit logs maintained for data access?

### Implementation Patterns

**Data Classification and Handling:**
```python
from enum import Enum

class DataClassification(Enum):
    PUBLIC = "public"
    INTERNAL = "internal"
    PERSONAL = "personal"         # PII — name, email, address
    SENSITIVE = "sensitive"       # Health, financial, biometric

# Field-level classification
class UserSchema:
    id: str                                    # INTERNAL
    email: Annotated[str, Personal]            # PERSONAL
    name: Annotated[str, Personal]             # PERSONAL
    health_data: Annotated[dict, Sensitive]    # SENSITIVE
    preferences: Annotated[dict, Internal]     # INTERNAL
```

**Right to Erasure:**
```python
class DataErasureService:
    def __init__(self, services: list[DataService]):
        self.services = services  # All services with user data

    async def erase_user_data(self, user_id: str) -> ErasureReport:
        report = ErasureReport(user_id=user_id)
        for service in self.services:
            try:
                result = await service.delete_user_data(user_id)
                report.add_success(service.name, result)
            except Exception as e:
                report.add_failure(service.name, str(e))

        # Anonymize data that can't be deleted (audit logs, aggregates)
        await self.anonymize_retained_data(user_id)

        # Log the erasure for compliance
        await self.audit_log.record(
            action="data_erasure",
            user_id=user_id,
            report=report.to_dict(),
        )
        return report
```

**Consent Management:**
```python
class ConsentManager:
    def record_consent(self, user_id: str, purpose: str,
                       version: str) -> ConsentRecord:
        return ConsentRecord(
            user_id=user_id,
            purpose=purpose,
            consent_version=version,
            granted_at=datetime.utcnow(),
            ip_address=request.client.host,
        )

    def check_consent(self, user_id: str, purpose: str) -> bool:
        consent = self.repo.find_latest(user_id, purpose)
        return consent is not None and not consent.withdrawn

    def withdraw_consent(self, user_id: str, purpose: str) -> None:
        consent = self.repo.find_latest(user_id, purpose)
        consent.withdrawn_at = datetime.utcnow()
        self.repo.save(consent)
        # Trigger: stop processing, notify third parties
        self.event_bus.publish(ConsentWithdrawn(user_id, purpose))
```

### Common Mistakes
1. **Privacy policy without implementation** — Having a privacy policy but no technical enforcement
2. **No data inventory** — Not knowing what personal data exists in the system
3. **Indefinite retention** — Keeping data forever "just in case" instead of defined retention periods
4. **Hard deletion only** — Forgetting to check backups, logs, caches, and third-party systems
5. **Consent as checkbox** — Consent must be informed, specific, and freely given, not a pre-checked box

### Integration with Other Topics
- **Application Security** — Privacy and security are complementary
- **Authentication and Authorization** — Access control protects personal data
- **Multi-Tenancy** — Tenant data isolation is a privacy requirement
- **Database Design** — Data classification affects schema design and encryption
- **Observability** — Logging must not expose personal data
- **API Design** — APIs should support data subject rights (export, delete)

---

## Resources

### Essential Reading
- GDPR text (gdpr-info.eu)
- NIST Privacy Framework
- *Privacy by Design* — Ann Cavoukian (original principles)

### Online Resources
- ICO (UK) GDPR guidance
- IAPP (International Association of Privacy Professionals)
- OWASP Privacy Risks Project

### Practice Exercises
1. Create a data inventory for a web application
2. Implement data subject access request (DSAR) endpoint
3. Build automated data retention and deletion pipeline
4. Implement consent collection and enforcement
5. Conduct a Data Protection Impact Assessment
