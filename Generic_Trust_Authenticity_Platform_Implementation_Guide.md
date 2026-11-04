# Generic Trust, Authenticity and Fraud-Risk Platform

## Core MVP and Industry Profile-Detection Pack Implementation Guide

**Purpose:** Define one reusable platform for detecting suspicious people, organizations, profiles, listings and activities, then extend it safely for tutors, matrimonial users, job candidates, companies, jobs, properties, owners and other industries.

**Recommended stack:** Python FastAPI or .NET Core, MS SQL Server as the primary database, private object storage, OpenCV/image embeddings, spaCy/transformer NLP, background workers, optional Neo4j for relationship exploration, and a web admin portal.

**Guiding principle:** The system identifies and explains risk signals. It does not declare a person, company or listing fraudulent solely from an AI score. Consequential actions require an authorized decision, evidence, audit history and an appeal/correction path.

---

## 1. Product Vision

Many digital platforms must decide whether an entity is authentic:

- Is this person using their own identity and consistent claims?
- Is this organization genuine and is the user authorized to represent it?
- Is this job, service or property listing legitimate?
- Are reviews, ratings and relationships organic or coordinated?
- Has the same image, document, contact detail or payment identity been reused across unrelated accounts?

Building separate fraud systems for every industry duplicates infrastructure, security, scoring, audit and review functionality. The proposed solution separates reusable capabilities from domain-specific logic:

```text
Shared Trust Platform
    + Domain Pack
    + Approved Verification Adapters
    = Industry-Specific Detection Product
```

The first release builds the **Core MVP**. Later releases add separately versioned and tested **Domain Packs** without copying the core application.

---

## 2. What Is Generic and What Is Domain-Specific?

### 2.1 Shared core

The core provides:

- Generic entities and profile versions
- Claims and supporting evidence
- Secure media and document handling
- Scan-job orchestration and retries
- Standard detector interface
- Exact and near-duplicate detection
- Text similarity and claim extraction framework
- Relationship and activity feature framework
- Versioned risk scoring
- Configurable workflow and action policies
- User reports
- Admin review queue and case screen
- Decisions, appeals and audit events
- Security, privacy, retention and monitoring controls
- Model/rule evaluation framework

### 2.2 Domain pack

Each domain pack defines:

- Entity roles and required profile fields
- Claims to extract and compare
- Permitted evidence and verification levels
- Domain-specific detectors and features
- Score weights, caps, exclusions and thresholds
- Human-review questions
- Publication/hold/restriction workflow
- User-facing explanations
- Retention and access rules
- Domain-labelled evaluation dataset
- Domain acceptance tests

### 2.3 External verification adapter

Adapters connect approved external services without coupling the core to one provider. Examples include:

- Identity or liveness provider
- Education/certification verification
- Company registry
- Employer-controlled-email verification
- Address verification
- Property records or authorized listing source
- Phone/email reputation or ownership check

External verification is never assumed to exist. Each adapter must have explicit authorization, contracts, error handling, privacy review and a fallback state such as `UNAVAILABLE` or `NOT_VERIFIED`.

---

## 3. Supported Entity Model

Use a generic entity model instead of tables named only for tutors or candidates.

### Entity types

| Entity type | Examples |
|---|---|
| `PERSON` | Tutor, bride/groom, candidate, recruiter, property owner, agent |
| `ORGANIZATION` | Company, school, agency, employer |
| `LISTING` | Tutor service, job post, property advertisement |
| `ASSET` | Property, qualification document, licence |
| `TRANSACTION` | Lesson, application, booking, interview, property enquiry |
| `REVIEW` | Student rating, employer review, listing feedback |

### Entity roles

Roles provide business meaning without creating a new core entity type for every use case:

```text
PERSON + TUTOR
PERSON + MATRIMONIAL_MEMBER
PERSON + JOB_CANDIDATE
PERSON + RECRUITER
PERSON + PROPERTY_OWNER
PERSON + PROPERTY_AGENT
ORGANIZATION + EMPLOYER
LISTING + JOB_POSTING
ASSET + PROPERTY
```

One entity may have multiple roles, but each role must have its own policy and permissions.

### Core concepts

- **Profile:** Current presentation of an entity.
- **Profile version:** Immutable snapshot used to reproduce an assessment.
- **Claim:** A statement such as “PhD,” “10 years’ experience,” “authorized recruiter,” or “owner of property.”
- **Evidence:** Image, document, verified email, transaction or approved external result supporting a claim.
- **Signal:** A detector result indicating risk, consistency, absence, insufficiency or processing failure.
- **Assessment:** Aggregated risk result for one entity/profile version and policy version.
- **Review case:** Human investigation and decision workflow.
- **Relationship:** A connection between entities, identifiers, evidence, devices, listings or activities.

---

## 4. Core MVP Scope

### Included

1. Create/update generic entities, roles, profiles, claims and evidence.
2. Secure image/document upload and metadata storage.
3. Asynchronous scan-job pipeline.
4. Exact and near-duplicate image/document fingerprint checks.
5. Generic text normalization, entity extraction and similarity checks.
6. Configurable deterministic claim-consistency rules.
7. Generic relationship features for shared identifiers and evidence.
8. Generic activity anomaly features.
9. Explainable, versioned risk scoring.
10. Configurable action bands and review workflow.
11. User reporting, admin review, decision and appeal.
12. Audit, observability, security and evaluation foundation.
13. One simple reference domain pack used to prove extensibility.

### Excluded from the Core MVP

- Internet-wide reverse-image search
- Automatic legal determination of fraud
- Unrestricted web scraping
- Training a large language or vision model from scratch
- Industry registry integrations not required by the reference pack
- Fully autonomous permanent bans
- Biometric face comparison unless separately approved
- Domain-specific compatibility, desirability, creditworthiness or employability scoring
- Production Neo4j unless SQL-based relationship features are demonstrably insufficient

---

## 5. Safety and Decision Boundaries

The platform must distinguish these states:

| State | Meaning |
|---|---|
| `CONSISTENT` | Available evidence supports the tested condition |
| `RISK_SIGNAL` | A relevant suspicious pattern was detected |
| `NOT_VERIFIED` | Claim has not been independently verified |
| `INSUFFICIENT_DATA` | There is not enough information for a conclusion |
| `UNAVAILABLE` | Required service or detector could not run |
| `PROCESSING_ERROR` | Technical failure occurred |
| `HUMAN_CONFIRMED` | Authorized reviewer confirmed the stated outcome |

`NOT_VERIFIED`, `INSUFFICIENT_DATA`, `UNAVAILABLE` and `PROCESSING_ERROR` must never silently become fraud points.

The system must not infer or penalize protected or highly sensitive personal characteristics. It must not use appearance, caste, religion, ethnicity, gender, disability, health, sexuality or similar characteristics as fraud predictors. Domain packs must document prohibited features.

---

## 6. Reference Architecture

### Components

| Component | Responsibility |
|---|---|
| Client portal | Entity onboarding, evidence upload, reports, corrections and appeals |
| Admin portal | Review queue, evidence inspection, decision, escalation and audit history |
| Core API | Authorization, validation, entity/profile management and workflow APIs |
| Scan orchestrator | Creates idempotent jobs and invokes enabled detectors |
| Worker service | Runs image, text, document, relationship and activity analysis |
| Policy service | Resolves domain configuration, rules, thresholds and workflow version |
| Scoring service | Aggregates standardized signals with caps and exclusions |
| MS SQL Server | Primary records, state, signals, assessments, cases and audits |
| Private object storage | Encrypted original and sanitized evidence files |
| Optional graph store | Multi-hop investigation when SQL no longer meets requirements |
| Monitoring platform | Metrics, logs, traces, alerts, queue health and model/rule drift |

### End-to-end flow

1. User submits or updates an entity profile.
2. API validates authorization, fields, evidence types, signatures and size.
3. System creates an immutable profile version.
4. Files are encrypted in private object storage; SQL stores references and hashes.
5. API creates one scan job for the profile version and selected domain pack.
6. Orchestrator loads the versioned policy and enabled detectors.
7. Workers emit standardized signals independently.
8. Scoring service applies domain weights, caps and exclusions.
9. Workflow service selects a recommended action.
10. Policy may allow normal processing, request information or create a review case.
11. Authorized reviewer records a final decision and reason.
12. User may correct data or appeal according to the domain policy.
13. Every material action and version is auditable.

---

## 7. Standard Detector Contract

Every detector—shared or domain-specific—must implement the same conceptual contract.

### Detector input

- Entity ID and role
- Immutable profile version ID
- Domain pack and policy version
- Approved evidence references
- Relevant relationships and activity window
- Correlation/scan job ID
- Feature flags and privacy restrictions

### Detector output

| Field | Purpose |
|---|---|
| `DetectorCode` | Stable detector identifier |
| `DetectorVersion` | Reproducibility and rollback |
| `SignalCode` | Stable explainable reason code |
| `SignalState` | Risk, consistent, unavailable, insufficient data or error |
| `RawValue` | Distance/count/rate or model probability where permitted |
| `NormalizedScore` | Standard 0–1 result before domain weighting |
| `EvidenceReferences` | Authorized references, not uncontrolled personal-data copies |
| `ExplanationParameters` | Safe values used to render reviewer/user explanation |
| `StartedOn/CompletedOn` | Operational traceability |

### Detector rules

- A detector must not update a final decision directly.
- Failures must be returned as failures, not zero risk.
- The same input and version must produce a reproducible result within documented tolerance.
- A detector must declare required data and supported entity roles.
- Domain packs may enable or disable detectors, but cannot change their meaning.
- Every detector requires unit tests, evaluation fixtures and an owner.

---

## 8. Shared Detector Library

### Image and media

- File-signature, MIME, byte-size and pixel-limit validation
- Malware/content scanning where supported
- Orientation and color normalization
- SHA-256 exact duplicate detection
- Perceptual hash near-duplicate detection
- Image embedding comparison for crops and transformations
- Face count/quality only where appropriate
- Optional face embedding only after biometric approval
- Weak manipulation/AI-generation signal, never standalone proof

### Document

- Exact and near-duplicate file fingerprint
- OCR extraction with source bounding boxes
- Metadata consistency
- Template/layout similarity
- Claim-to-document text comparison
- Expiry-date and issuer-field extraction
- Tamper indicator as a review signal, not a legal conclusion

### Text and claims

- Safe normalization while retaining original text
- Named-entity and date extraction
- Timeline consistency
- Arithmetic consistency, such as experience-years versus dates
- Cross-field contradictions
- Duplicate/copy similarity against the authorized corpus
- Prohibited or scam-language patterns configured by domain
- Model-assisted semantic classification only after baseline evaluation

### Relationship

- Shared normalized email/phone token
- Shared document or media fingerprint
- Shared payment/payout token
- Shared device/session token
- Shared network signal with low standalone weight
- Entity-to-organization authorization link
- Multi-hop coordinated account/review/listing patterns

### Activity and reputation

- Sudden activity burst
- New-account concentration
- Repeated text
- Activity without a prerequisite transaction
- Abnormal velocity relative to a suitable cohort
- Reciprocal/circular interactions
- Complaint/report rate with abuse resistance

---

## 9. Configuration and Extension Model

Configuration is appropriate for policy values. Code modules are appropriate for new behavior.

### Configuration should control

- Enabled detectors
- Required/optional profile fields
- Allowed evidence types
- Signal weights and group caps
- Exclusion and dependency rules
- Thresholds and action bands
- Review SLA and priority
- Reviewer roles
- Decision reason codes
- Publication/hold behavior
- Retention periods
- User-facing message templates
- Feature rollout percentage

### Code should implement

- New algorithms
- New document parsing
- Domain-specific feature computation
- External verification adapters
- Complex graph traversal
- Specialized model inference
- New workflow capability unavailable in the core

### Example domain-pack configuration

```yaml
domain_pack: property_listing
version: 1.0.0
entity_roles:
  - PROPERTY_OWNER
  - PROPERTY_AGENT
  - PROPERTY_LISTING

required_claims:
  - relationship_to_property
  - locality
  - property_type
  - asking_price

enabled_detectors:
  - IMAGE_EXACT_DUPLICATE
  - IMAGE_NEAR_DUPLICATE
  - TEXT_HIGH_SIMILARITY
  - SHARED_CONTACT
  - LISTING_PRICE_ANOMALY
  - OWNER_AUTHORIZATION_CONSISTENCY

group_caps:
  media: 25
  claims: 25
  relationships: 30
  activity: 20

action_bands:
  - max_score: 29
    action: NORMAL_PROCESSING
  - max_score: 59
    action: REVIEW
  - max_score: 79
    action: HOLD_AND_REVIEW
  - max_score: 100
    action: ESCALATE

prohibited_features:
  - religion
  - caste
  - ethnicity
  - gender
```

Configurations must be schema-validated, reviewed, versioned, tested and deployed like code. An invalid or missing configuration must fail safely.

---

## 10. Versioned Risk Scoring

### Core calculation

1. Accept only valid, completed signal outputs.
2. Apply the domain-specific signal weight.
3. Apply dependency and exclusion rules.
4. Deduplicate correlated evidence.
5. Cap each signal group.
6. Calculate total score and risk band.
7. Record the exact score-policy version.
8. Generate a recommended action, not a final factual declaration.

### Why caps matter

One reused image could trigger an exact hash, perceptual hash and embedding match. Without deduplication and caps, one piece of evidence could be counted three times and create an unjustified high score.

### Required assessment output

- Total score and risk band
- Recommended action
- Signal contributions by group
- Consistent checks
- Unavailable/insufficient/error checks
- Evidence references
- Detector and scoring versions
- Profile and domain-pack versions
- Creation time and correlation ID
- Human decision and appeal state, when present

Do not expose internal thresholds, raw embeddings, another user's private data or abuse-enabling detail to general users.

---

## 11. Core MS SQL Data Model

| Table | Purpose | Important fields |
|---|---|---|
| `TrustEntity` | Generic person, organization, listing, asset or transaction | EntityId, EntityType, Status |
| `EntityRole` | Domain role assigned to entity | EntityRoleId, EntityId, RoleCode, DomainPackCode |
| `EntityProfileVersion` | Immutable assessed snapshot | ProfileVersionId, EntityId, VersionNumber, DataJson, SubmittedOn |
| `EntityClaim` | Structured claim | ClaimId, ProfileVersionId, ClaimType, ClaimValueJson, VerificationState |
| `EvidenceItem` | File/external evidence metadata | EvidenceId, EntityId, EvidenceType, StorageKey, Hash, RetentionUntil |
| `ClaimEvidenceLink` | Claim-to-evidence connection | ClaimId, EvidenceId, RelationshipType |
| `EntityRelationship` | Versioned relationship edge | RelationshipId, SourceEntityId, TargetEntityId, Type, Strength |
| `ActivityEvent` | Domain activity input | ActivityId, EntityId, ActivityType, OccurredOn, MetadataJson |
| `ScanJob` | Scan lifecycle | ScanJobId, ProfileVersionId, DomainPackVersion, Status, Attempts |
| `DetectorRun` | Individual detector execution | DetectorRunId, ScanJobId, DetectorCode, Version, Status, DurationMs |
| `RiskSignal` | Standard detector output | SignalId, DetectorRunId, SignalCode, State, NormalizedScore, EvidenceJson |
| `RiskAssessment` | Aggregated score | AssessmentId, ScanJobId, TotalScore, RiskBand, RecommendedAction, PolicyVersion |
| `UserReport` | User-submitted concern | ReportId, TargetEntityId, ReporterEntityId, ReasonCode, Status |
| `ReviewCase` | Human review lifecycle | CaseId, AssessmentId, Priority, AssignedTo, Status, DueOn |
| `ReviewDecision` | Authorized decision | DecisionId, CaseId, DecisionCode, ReasonCode, DecidedBy, DecidedOn |
| `AppealCase` | Correction/appeal workflow | AppealId, DecisionId, SubmittedBy, Status, Outcome |
| `AuditEvent` | Append-only material action record | AuditEventId, ActorId, ActionCode, EntityId, CreatedOn, MetadataJson |
| `PolicyVersion` | Active configuration identity | PolicyVersionId, DomainPackCode, Version, Checksum, Status |

### Data requirements

- Use primary/foreign keys, useful indexes, UTC timestamps and row concurrency controls.
- Apply `CreatedBy`, `CreatedOn`, `ModifiedBy`, `ModifiedOn` and `IsActive` where mutable records require them.
- Preserve immutable profile and policy versions used for each assessment.
- Keep binary evidence in encrypted private storage unless an approved requirement says otherwise.
- Tokenize or salt/hash high-risk identifiers; do not store plain payment/device/government identifiers unnecessarily.
- Validate all JSON against an owned schema.
- Design retention/deletion so expired evidence can be removed without destroying required decision audit history.

---

## 12. Core API Surface

| Method and route | Purpose |
|---|---|
| `POST /api/entities` | Create an entity and role |
| `POST /api/entities/{id}/profiles` | Submit a new immutable profile version |
| `POST /api/entities/{id}/evidence` | Upload approved evidence |
| `POST /api/profile-versions/{id}/scans` | Start or reuse an idempotent scan |
| `GET /api/scans/{id}` | Get scan progress without restricted evidence |
| `GET /api/entities/{id}/assessments/current` | Retrieve permitted current assessment |
| `POST /api/entities/{id}/reports` | Submit a concern/report |
| `GET /api/admin/trust/review-cases` | Filtered and paginated admin queue |
| `GET /api/admin/trust/review-cases/{id}` | Authorized case/evidence view |
| `POST /api/admin/trust/review-cases/{id}/decisions` | Record authorized decision |
| `POST /api/decisions/{id}/appeals` | Submit correction/appeal |
| `GET /api/admin/trust/policy-versions` | View policy versions and rollout state |

### API requirements

- Authentication and role/attribute-based authorization
- Object-level access checks
- File signature, decoder, size and field validation
- Idempotency for submission and scanning
- Rate limits for uploads, reports, rescans and decisions
- Pagination, filtering and stable sorting
- Correlation IDs and structured error responses
- Secure rendering and XSS protection for user text
- No secrets, evidence bytes, biometric templates or sensitive identifiers in logs
- Optimistic concurrency on decisions and assignments

---

## 13. Admin Review Experience

### Queue

Show:

- Entity type, domain role and safe display name
- Risk band and total score
- Top reason codes
- Trigger source
- Submission/queue age
- Scan completeness, including unavailable checks
- Assigned reviewer and SLA status

Provide filtering by domain, risk, signal, trigger, age, assignment and status.

### Case details

- Profile version and declared claims
- Permitted evidence preview
- Signals grouped by media, claims, relationships and activity
- Clear separation of risk, consistent, unavailable, insufficient and failed checks
- Related entities with reasons for the relationship
- Reports and prior assessments
- Policy/detector versions
- Audit timeline

### Decisions

The core should support configurable decisions such as:

- Approve/continue normal processing
- Request additional information
- Restrict visibility temporarily
- Hold pending review
- Escalate
- Suspend/reject, only for authorized roles
- Correct previous decision

Adverse decisions require confirmation and a structured reason. The application must prevent stale reviewers from overwriting a newer decision.

---

## 14. Core MVP Delivery Plan

The following is an indicative estimate for one small team or sequential member-led project where authentication, CI/CD, object storage and UI foundations already exist.

| Phase | Work | Estimate |
|---|---|---:|
| 0 | Discovery, policy boundaries and current-system mapping | 12 h |
| 1 | Generic schema, migrations and domain-pack contract | 24 h |
| 2 | Secure evidence service and profile versioning | 20 h |
| 3 | Scan orchestration, worker contract, retries and idempotency | 24 h |
| 4 | Shared image/document/text baseline detectors | 32 h |
| 5 | Relationship/activity feature framework | 20 h |
| 6 | Versioned scoring and workflow engine | 24 h |
| 7 | Core APIs and authorization | 24 h |
| 8 | Admin review, decisions, reports and appeals | 28 h |
| 9 | Reference domain pack and configuration validation | 20 h |
| 10 | Evaluation, security, monitoring, runbook and handover | 28 h |
| **Total** | **Indicative Core MVP** | **256 h** |

This is not equivalent to a production enterprise platform. External integrations, biometric processing, multilingual models, high scale, regulatory review or a new UI/authentication foundation require separate estimates.

### Core milestone delivery

1. **Foundation:** Entities, roles, versions, claims, evidence, scans and detector SDK.
2. **Detection:** Shared detectors, relationship/activity features and scoring.
3. **Operations:** Review dashboard, decisions, appeals, security and monitoring.
4. **Extensibility proof:** Reference pack delivered without changing core tables or copying core services.

---

## 15. How to Build a New Industry Domain Pack

### Step 1 — Define the business harm

Document exactly what harm is being prevented. Do not begin with a model.

Examples:

- Tutor impersonation or false qualification
- Matrimonial identity impersonation or duplicate deceptive accounts
- Candidate history inconsistency
- Recruiter/company impersonation
- Job fee/payment scam
- Property listing duplication or unauthorized representation

### Step 2 — Define entities and relationships

Map the domain to existing core types and roles. Add a new core type only if it cannot be represented correctly.

Example property map:

```text
PERSON(PROPERTY_OWNER) --claims--> ASSET(PROPERTY)
PERSON(PROPERTY_AGENT) --authorized_for--> ASSET(PROPERTY)
LISTING(PROPERTY_LISTING) --advertises--> ASSET(PROPERTY)
ORGANIZATION(AGENCY) --employs--> PERSON(PROPERTY_AGENT)
```

### Step 3 — Define claims and evidence

For each claim, document:

- Claim code and meaning
- Required/optional state
- Data type and validation
- Permitted evidence
- Verification levels
- Who may view the evidence
- Retention period
- Failure/unknown behavior

### Step 4 — Perform abuse-case and fairness review

List:

- Likely attacker behavior
- Legitimate look-alike behavior
- False-positive harm
- Features that must never be used
- Groups/languages likely to have weaker data
- Required human-review protection

### Step 5 — Reuse shared detectors

Select the minimum shared detectors that directly address the documented abuse cases. Do not enable every detector just because it exists.

### Step 6 — Add domain-specific detectors

Implement only behavior that cannot be expressed through configuration. Follow the standard detector contract and add independent tests.

### Step 7 — Create scoring and workflow policy

Define weights, group caps, dependencies and action thresholds. For example, duplicate property images may be legitimate across authorized agents, while duplicate matrimonial images across unrelated identities may deserve higher review priority.

### Step 8 — Prepare a lawful labelled dataset

- Define labels and ambiguity state
- Record data source and permission
- De-identify where possible
- Split by person/group to prevent leakage
- Keep a locked acceptance set
- Document class balance and limitations

### Step 9 — Evaluate before activation

Measure precision, recall, false-positive rate, review volume and error patterns at proposed thresholds. Evaluate relevant language and business cohorts where sample sizes allow.

### Step 10 — Release safely

Use this progression:

1. Offline evaluation
2. Shadow mode with no user impact
3. Reviewer-visible recommendations
4. Limited rollout
5. Normal rollout with monitoring

Permanent autonomous adverse action is outside the initial domain-pack release.

### Step 11 — Operate and recalibrate

Monitor drift, reviewer overturns, appeals, abuse adaptation, queue growth and external-provider failures. Changes require a new policy/model version and regression evaluation.

---

## 16. Initial Domain-Pack Matrix

| Domain pack | Main entities | Reusable signals | Required specialized logic |
|---|---|---|---|
| Tutors/mentors | Person, service listing, qualification | Image reuse, bio similarity, claims, reviews, multi-account | Education/experience timeline, lesson eligibility |
| Matrimonial | Person, profile | Image reuse, duplicate account, claim consistency, reports | Consent-sensitive identity workflow and stricter privacy controls |
| Job candidates | Person, resume, qualification | Resume similarity, dates, document reuse, shared contact | Employment timeline and employer/education verification adapters |
| Companies | Organization, representative | Contact/domain/document reuse, relationships | Registration, director/address/domain-ownership validation |
| Job listings | Organization, recruiter, listing | Copied text, contact reuse, activity burst, reports | Employer authorization, fee/payment scam and vacancy consistency |
| Properties | Asset, listing, owner, agent | Image/text duplicate, contact reuse, listing activity | Address normalization, price anomaly and listing-to-asset resolution |
| Property owners/agents | Person/organization, asset | Identity/contact/document relationships | Ownership/authorization and agency relationship evidence |

---

## 17. Domain-Specific Warnings

### Matrimonial profiles

- Never use fraud detection to rank compatibility or desirability.
- Do not infer caste, religion, health, sexuality or personality.
- Restrict image matches and identity evidence to specially authorized reviewers.
- Provide correction, consent and appeal controls.

### Candidates and jobs

- “Not verified” does not mean false; employers may close or decline verification.
- Do not infer employability from sensitive characteristics.
- Verify the recruiter’s authorization to represent the employer.
- Payment requests and off-platform contact may be risk signals, not automatic proof.

### Companies

- A real company name can be copied by an impostor.
- Verify both the organization and the relationship of the representative to it.
- Registry unavailability must not become a negative result.

### Property

- Several agents may legitimately advertise the same property.
- Duplicate photos require asset resolution and authorization context.
- Price anomalies require local, time-relevant comparison data and should be review signals.
- Ownership documents require highly restricted handling and retention.

---

## 18. Acceptance Tests for the Core MVP

### Functional

| ID | Scenario | Expected result |
|---|---|---|
| CORE-AT-01 | Create person, organization, listing and asset entities | Each valid type and role is stored without domain-specific core tables |
| CORE-AT-02 | Submit a profile | Immutable version created and scan queued |
| CORE-AT-03 | Submit same profile request twice | Idempotency prevents duplicate version/job where configured |
| CORE-AT-04 | Upload corrupt/oversized/spoofed file | Request safely rejected; invalid evidence is not assessed |
| CORE-AT-05 | Exact reused image | Standard exact-duplicate signal produced with authorized reference |
| CORE-AT-06 | Resized/cropped image | Near-duplicate detector returns evaluated result without failure |
| CORE-AT-07 | Duplicate document | Document reuse signal generated; file content remains private |
| CORE-AT-08 | Highly similar profile text | Text-similarity signal uses stable reason code and version |
| CORE-AT-09 | Generic contradictory dates | Claim rule emits exact conflicting fields and explanation |
| CORE-AT-10 | Detector timeout | State is unavailable/error, retry occurs and result is not treated as clean |
| CORE-AT-11 | Multiple related signals from one image | Deduplication/caps prevent triple counting |
| CORE-AT-12 | Domain configuration changes | Existing assessment retains old version; new scan uses new version |
| CORE-AT-13 | Invalid domain configuration | Activation fails safely with actionable validation result |
| CORE-AT-14 | Unauthorized evidence access | Access denied and auditable; no evidence leaked |
| CORE-AT-15 | Authorized review decision | State changes atomically and append-only audit event created |
| CORE-AT-16 | Concurrent decisions | Stale decision is rejected by concurrency control |
| CORE-AT-17 | User report abuse | Authentication, rate limit and duplicate-report controls apply |
| CORE-AT-18 | Appeal/correction | Case links to original decision and preserves history |
| CORE-AT-19 | Evidence retention expiry | Bytes removed/de-identified per policy without falsifying audit history |
| CORE-AT-20 | New reference domain pack | Pack runs using core interfaces without copying/modifying core services |

### Non-functional

| ID | Requirement | Acceptance expectation |
|---|---|---|
| CORE-NFT-01 | Submission latency | Asynchronous submission meets agreed percentile target under expected load |
| CORE-NFT-02 | Scan reliability | Retry, dead-letter/manual retry and partial-detector states are observable |
| CORE-NFT-03 | Reproducibility | Assessment trace contains profile, detector, domain and policy versions |
| CORE-NFT-04 | Security | Auth, object access, upload, XSS, injection, secret and dependency tests pass |
| CORE-NFT-05 | Privacy | Sensitive evidence is encrypted, access-controlled and retained by policy |
| CORE-NFT-06 | Accessibility | Critical admin flow is keyboard usable and does not rely on color alone |
| CORE-NFT-07 | Observability | Queue age/depth, latency, failures, retries and decision outcomes are monitored |
| CORE-NFT-08 | Extensibility | Detector/domain SDK contract and example pack are documented and tested |
| CORE-NFT-09 | Rollback | Policy/detector release can be disabled or rolled back without data loss |
| CORE-NFT-10 | Audit | Material changes and accesses are attributable, timestamped and tamper-evident |

---

## 19. Acceptance Template for Every Domain Pack

Every new pack must supply tests for:

1. One legitimate low-risk case.
2. Each major abuse case independently.
3. A combined high-risk case.
4. At least three legitimate look-alike cases that could create false positives.
5. Missing/insufficient data.
6. External verifier unavailable and timeout states.
7. Unauthorized evidence access.
8. Policy threshold boundaries.
9. Score deduplication and caps.
10. Human review, correction and appeal.
11. Retention/deletion.
12. Dataset evaluation and approved quality targets.
13. Shadow-mode comparison against reviewer decisions.
14. Rollback to the previous policy version.

Domain release approval must record unresolved limitations and the business owner accepting them.

---

## 20. Security, Privacy and Fairness Requirements

- Complete threat, privacy and abuse assessments before production.
- Obtain explicit approval before processing biometric templates.
- Encrypt transport and storage.
- Use private object storage and short-lived authorized access.
- Apply least privilege to users, support staff, reviewers and administrators.
- Isolate risky file parsing and scanning.
- Validate actual file signatures and decoding limits.
- Protect against injection, stored XSS, IDOR, malicious files and report spam.
- Keep raw evidence, identifiers, tokens and embeddings out of normal logs.
- Version and approve configuration/model changes.
- Define retention and deletion by domain and evidence type.
- Provide correction and appeal where decisions affect access or opportunity.
- Measure false positives and reviewer overturns.
- Prohibit sensitive/protected attributes and unjustified proxies.
- Require human review for material adverse decisions during initial releases.

---

## 21. Monitoring and Governance

### Operational metrics

- Scan queue depth and oldest job
- End-to-end and detector latency
- Completion, failure, retry and dead-letter rate
- External-verifier availability
- Review queue age and SLA breaches
- Evidence access failures and suspicious access

### Quality and business metrics

- Signal frequency by detector and version
- Precision, recall and false-positive rate on labelled samples
- Percentage routed to review
- Reviewer decision distribution
- Reviewer agreement and overturn rate
- Appeal rate and appeal outcome
- User correction rate
- Performance by supported language/cohort where valid and safe
- Input and concept drift

### Change governance

Every policy/model change must include:

- Owner and business justification
- Source/configuration diff
- Evaluation report
- False-positive impact
- Security/privacy review where applicable
- Rollout and rollback plan
- Effective version and date
- Post-release monitoring window

---

## 22. Repository Structure

```text
trust-authenticity-platform/
├── README.md
├── docs/
│   ├── architecture.md
│   ├── detector-contract.md
│   ├── domain-pack-contract.md
│   ├── data-dictionary.md
│   ├── security-and-privacy.md
│   ├── evaluation-standard.md
│   └── operations-runbook.md
├── core/
│   ├── api/
│   ├── entities/
│   ├── evidence/
│   ├── orchestration/
│   ├── scoring/
│   ├── workflow/
│   ├── review/
│   └── audit/
├── detectors/
│   ├── media/
│   ├── document/
│   ├── text/
│   ├── relationship/
│   └── activity/
├── domain-packs/
│   ├── reference/
│   ├── tutors/
│   ├── matrimonial/
│   ├── candidates/
│   ├── companies/
│   ├── jobs/
│   └── property/
├── adapters/
├── database/
│   ├── migrations/
│   └── seed/
├── admin-portal/
├── tests/
│   ├── unit/
│   ├── integration/
│   ├── contract/
│   ├── security/
│   └── acceptance/
├── config-schemas/
└── deployment/
```

---

## 23. Definition of Done

### Core MVP

- Generic entities/roles support the reference pack without tutor/property/job-specific core tables.
- Profile, evidence, scan, signal, assessment and decision versions are reproducible.
- Shared detectors implement the standard contract and pass tests.
- Detector failure cannot appear as a clean result.
- Scoring applies dependencies, deduplication and group caps.
- Domain configuration is validated and versioned.
- Admin review, decision, correction and appeal work end-to-end.
- Access restrictions and evidence privacy tests pass.
- Metrics, alerts, retry and rollback are demonstrated.
- Core acceptance tests pass or have approved exceptions.
- Documentation, deployment, runbook and handover are complete.

### Industry domain pack

- Business harms, entities, claims, evidence and prohibited features are documented.
- Shared detectors are reused where appropriate.
- Domain-only code follows the detector/adapter contracts.
- Lawful labelled data and locked acceptance data are documented.
- Thresholds are calibrated against domain results, not copied from another industry.
- Legitimate look-alike and insufficient-data tests pass.
- Reviewer UI shows domain-specific evidence and questions.
- Shadow/limited rollout is completed.
- Owner approves limitations, monitoring and rollback.

---

## 24. Final Demonstration

The team must demonstrate, without manual database edits:

1. Create different entity types and roles.
2. Submit and version a profile with claims and evidence.
3. Detect an exact and transformed duplicate.
4. Detect a text/claim inconsistency.
5. Show relationship/activity signals.
6. Show score deduplication, group caps and recommended action.
7. Force a detector failure and show unavailable/retry handling.
8. Review the case, record a decision and show audit history.
9. Submit a correction/appeal.
10. Activate a new reference domain-pack version safely.
11. Prove an earlier assessment retains its original policy version.
12. Roll back/disable a faulty detector or policy.
13. Show security denial for an unauthorized evidence request.
14. Show evaluation, monitoring, known limitations and ownership.

---

## 25. Recommended Implementation Decision

Build one reusable core, but do not attempt one universal “fake/not fake” model.

The appropriate product pattern is:

```text
Generic data and workflow foundation
        +
Reusable detector library
        +
Versioned industry domain pack
        +
Optional authorized verification adapters
        +
Human review and appeal
```

Begin with the Core MVP and one narrow reference pack. Treat each later industry as a separate product release with its own claims, evidence, dataset, thresholds, safeguards and acceptance tests. This preserves reuse while preventing a rule that is reasonable for one industry from incorrectly harming users in another.
