# Fake Profile Detection for Mentors and Tutors

## member Implementation and Delivery Guide

**Document purpose:** Convert the feature idea into a buildable, testable MVP that a new member can understand and deliver safely.

**Target stack:** Python FastAPI, MS SQL Server, OpenCV, NLP models, optional Neo4j, and a web-based admin dashboard.

**Estimated MVP effort:** 134 hours, subject to the assumptions and exclusions in this document.

---

## 1. The Real Problem

An online learning platform cannot safely assume that every mentor or tutor profile is genuine. A bad actor may:

- Upload a profile photo already used by another account.
- Use a stock, AI-generated, or stolen-looking image.
- Claim qualifications, employment, or teaching experience without supporting evidence.
- Copy another tutor's biography or use highly misleading promotional language.
- Create several accounts controlled by the same person.
- Create coordinated learner accounts that submit fake ratings or reviews.

The platform needs an automated system that finds **risk signals**, combines them into an explainable score, and sends suspicious profiles to an administrator for review.

### What the system must not do

- It must not declare a person fraudulent solely because an ML model returned a high score.
- It must not treat a face match as proof of identity theft.
- It must not attempt unrestricted internet surveillance or scrape external sites without authorization.
- It must not reject a profile based on protected characteristics, writing style, accent, nationality, age, gender, disability, or appearance.
- It must not store facial images or embeddings longer than the approved retention period.

The correct outcome is: **detect, explain, review, decide, record, and allow appeal**.

---

## 2. MVP Goal

When a tutor submits or edits a profile, the platform will:

1. Validate the submitted data and files.
2. compare the profile image with images already registered on the platform;
3. extract and analyze biography and qualification claims;
4. inspect account, device, contact, and rating relationships;
5. calculate an explainable fraud-risk score;
6. automatically approve low-risk profiles or place medium/high-risk profiles in an admin review queue according to policy;
7. let an administrator review evidence and record a decision; and
8. retain an audit trail of every automated and manual action.

### MVP success definition

The MVP is complete only when a new profile can travel end-to-end from submission to a recorded decision, with repeatable automated tests and no manual database changes.

---

## 3. Users and User Stories

### Tutor or mentor

- As a tutor, I can submit a profile photo, biography, experience, qualifications, and supporting documents.
- As a tutor, I can see that my profile is pending review without seeing internal fraud rules.
- As a tutor, I can correct missing information or appeal an adverse decision.

### Learner

- As a learner, I can report a suspicious profile and select a reason.
- As a learner, I receive confirmation without being shown private investigation details.

### Administrator or trust analyst

- As an administrator, I can see profiles ordered by risk and age.
- As an administrator, I can view each risk signal and its supporting evidence.
- As an administrator, I can approve, request information, restrict, suspend, or reject a profile.
- As an administrator, I must enter a reason for a decision.

### Platform or compliance owner

- As an owner, I can see false-positive rate, review turnaround time, model/rule version, and audit history.

---

## 4. Scope

### Included in the 134-hour MVP

- New-profile and profile-update scanning.
- Within-platform duplicate-image detection.
- Basic image quality and manipulation-risk signals.
- NLP extraction of qualification and experience claims.
- Biography duplication and suspicious-text signals.
- Rules for multiple-account and abnormal-rating patterns.
- Explainable weighted risk scoring.
- User report intake.
- Admin review dashboard.
- Admin decision and audit history.
- Unit, integration, security, and acceptance tests.
- Initial threshold tuning using an approved labelled dataset.

### Not included unless separately approved

- Searching the entire public internet for stolen images.
- Direct verification with universities, employers, government databases, LinkedIn, or paid identity providers.
- Production-grade custom training of BERT/RoBERTa from scratch.
- Live video identity verification or liveness detection.
- Legal adjudication of fraud.
- Fully automatic permanent bans.
- A production Neo4j cluster; the MVP can calculate graph features in SQL first.
- Native mobile application changes.
- Multilingual model training beyond languages represented in the approved dataset.

---

## 5. Recommended Architecture

### Components

| Component | Responsibility |
|---|---|
| Web portal | Tutor submission, learner report, and admin review screens |
| FastAPI service | Validation, orchestration, scoring, review APIs, and audit events |
| Background worker | Image/NLP analysis outside the request-response cycle |
| MS SQL Server | Primary transactional data, signals, scores, decisions, and audit logs |
| Object storage | Encrypted profile images and qualification documents |
| Model/rule package | Versioned image, text, graph, and rating analyzers |
| Optional Neo4j | Relationship exploration when SQL graph queries become insufficient |
| Monitoring | Job health, latency, failures, drift, and review outcomes |

### Processing flow

1. Portal submits a profile.
2. API validates file type, size, required fields, and authorization.
3. API stores metadata in SQL and file bytes in private object storage.
4. API creates a scan job and returns `202 Accepted` with a job identifier.
5. Worker loads the profile and runs enabled detectors.
6. Each detector writes a normalized signal with score, reason, evidence reference, and version.
7. Scoring service calculates overall risk and recommended action.
8. Low-risk profiles follow the configured approval policy; other profiles enter the review queue.
9. Admin records the final decision and reason.
10. The system writes an immutable audit event and updates metrics.

---

## 6. Detection Signals

### 6.1 Image signals

Implement these in order:

1. **Upload validation**
   - Permit only approved formats such as JPEG, PNG, and WebP.
   - Verify actual file signature; do not trust the extension.
   - Reject files above the configured size or pixel limit.
   - Remove unsafe metadata before normal use.

2. **Exact duplicate detection**
   - Calculate SHA-256 for the normalized image.
   - Find another active account with the same hash.
   - Exact reuse is a strong signal but may have legitimate explanations.

3. **Near-duplicate detection**
   - Normalize orientation, dimensions, and color space.
   - Calculate a perceptual hash such as pHash.
   - Compare Hamming distance against configured thresholds.
   - Use image embeddings for crops, compression, borders, or small edits.

4. **Face consistency, only if approved**
   - Detect whether the image contains zero, one, or multiple faces.
   - Compare face embeddings only against authorized platform records.
   - Encrypt embeddings and treat them as sensitive biometric data.
   - Never use emotion, ethnicity, gender, or attractiveness inference.

5. **Quality/manipulation indicators**
   - Very low resolution, heavy blur, unusual aspect ratio, or failed face detection.
   - Treat AI-generated/manipulation classifiers as weak signals because their error rates can be high.

**Important:** “No face detected” is not fraud. It may mean the image is a logo, poor quality, inaccessible to the detector, or intentionally privacy-preserving. Send it for correction or review.

### 6.2 Text and credential signals

1. Normalize text while preserving the original.
2. Use NLP to extract entities and claims:
   - degrees and certifications;
   - institutions;
   - employers and job titles;
   - subjects taught;
   - years and date ranges;
   - claimed years of experience.
3. Apply deterministic consistency checks first:
   - end date occurs before start date;
   - claimed years exceed the timeline;
   - “PhD” appears in biography but not in qualifications;
   - mutually inconsistent job and education dates;
   - unsupported superlatives such as “government certified” with no issuing body.
4. Detect biography duplication against other platform profiles using text similarity.
5. Use a pretrained language model only for classification or semantic similarity after a labelled evaluation set exists.
6. Store the exact reason for every text signal; do not store only a model probability.

**Credential verification levels:**

| Level | Meaning |
|---|---|
| Self-declared | Tutor entered the claim; no supporting document checked |
| Document supplied | A document was uploaded, but issuer is not independently verified |
| Manually reviewed | An authorized reviewer compared the claim and document |
| Issuer verified | An approved authoritative source confirmed it |

The UI must not display “verified” unless the relevant verification level and policy permit it.

### 6.3 Multiple-account and relationship signals

Potential relationships include shared:

- normalized email or phone number;
- payment account or payout beneficiary token;
- device identifier or strongly linked session pattern;
- IP/network pattern, used cautiously because homes, colleges, and VPNs may be shared;
- exact or near-duplicate image;
- biography or qualification document fingerprint;
- reviewer accounts and rating timing.

Start with SQL queries and scheduled feature calculation. Add Neo4j only when analysts need multi-hop relationship exploration, for example:

`Tutor A -> device -> Reviewer B -> reviews -> Tutor C -> image -> Tutor A`

Shared IP alone must never cause rejection.

### 6.4 Rating and review signals

Calculate explainable features such as:

- sudden burst of reviews shortly after profile creation;
- many reviews from newly created learner accounts;
- repeated review text or high text similarity;
- unusually high rating concentration compared with a relevant cohort;
- reciprocal or circular relationships;
- many reviewers sharing device/payment/network signals;
- rating velocity inconsistent with completed lessons;
- review created without an eligible completed transaction.

Require a minimum sample size before using statistical anomaly rules. New tutors with few reviews should be marked “insufficient data,” not suspicious solely because the sample is small.

---

## 7. Fraud-Risk Scoring

Use a transparent score for the MVP rather than a single opaque model.

### Example score design

| Signal group | Maximum points | Example evidence |
|---|---:|---|
| Image risk | 30 | Exact/near duplicate with another unrelated account |
| Credential/text risk | 25 | Contradictory or highly duplicated claims |
| Multiple-account risk | 25 | Strong combination of shared identifiers |
| Rating/review risk | 20 | Coordinated or ineligible review pattern |
| **Total** | **100** | Sum after caps and policy exclusions |

### Example action bands

| Score | Risk label | Recommended action |
|---:|---|---|
| 0–29 | Low | Continue according to normal onboarding policy |
| 30–59 | Medium | Manual review or request more information |
| 60–79 | High | Hold publication and prioritize review |
| 80–100 | Critical | Hold publication and escalate to senior reviewer |

These values are starting assumptions, not final production thresholds. Thresholds must be calibrated against labelled data and approved by the product/compliance owner.

### Required score output

Every result must contain:

- overall score and label;
- triggered signals and their point contribution;
- non-triggered or unavailable checks;
- evidence references, not raw sensitive data where unnecessary;
- detector/rule/model version;
- timestamp and scan job identifier;
- recommended action; and
- final human decision, when available.

---

## 8. Minimum Data Model in MS SQL Server

Use organization naming standards and audit columns. Do not store image bytes in normal SQL rows unless explicitly required.

| Table | Purpose | Important fields |
|---|---|---|
| `TutorProfile` | Current tutor profile | ProfileId, UserId, Bio, Status, SubmittedOn |
| `TutorQualification` | Qualification claims | QualificationId, ProfileId, Degree, Institution, StartDate, EndDate, VerificationLevel |
| `ProfileMedia` | Secure media reference and hashes | MediaId, ProfileId, StorageKey, Sha256, PerceptualHash, RetentionUntil |
| `ProfileScanJob` | Asynchronous scan lifecycle | ScanJobId, ProfileId, TriggerType, Status, Attempts, StartedOn, CompletedOn |
| `FraudSignal` | One explainable detector result | SignalId, ScanJobId, SignalType, RawScore, WeightedPoints, ReasonCode, EvidenceJson, DetectorVersion |
| `FraudAssessment` | Aggregated result | AssessmentId, ScanJobId, TotalScore, RiskLevel, RecommendedAction, ScoreVersion |
| `ProfileReport` | Learner/admin report | ReportId, ProfileId, ReporterUserId, ReasonCode, Description, Status |
| `AdminReview` | Human review and decision | ReviewId, AssessmentId, ReviewerUserId, Decision, ReasonCode, Notes, ReviewedOn |
| `RatingReview` | Rating data | RatingId, TutorId, LearnerId, LessonId, Score, ReviewText, CreatedOn |
| `RelationshipFeature` | Calculated account links | FeatureId, SourceUserId, TargetUserId, FeatureType, Strength, CalculatedOn |
| `AuditEvent` | Tamper-evident action history | EventId, ActorId, Action, EntityType, EntityId, MetadataJson, CreatedOn |

### Data rules

- Apply foreign keys, indexes, UTC timestamps, and least-privilege access.
- Add `CreatedBy`, `CreatedOn`, `ModifiedBy`, `ModifiedOn`, `IsActive`, and row-version/concurrency fields where appropriate.
- Keep user-entered text separate from normalized/model-derived fields.
- Do not store plain device, payment, or government identifiers; use scoped tokens or salted hashes approved by security.
- Evidence JSON must follow a schema and must not become a dumping ground for personal data.

---

## 9. API Delivery Contract

Exact routes may follow the existing platform convention.

| Method and route | Purpose | Expected result |
|---|---|---|
| `POST /api/tutor-profiles` | Submit profile and files | Profile created; scan queued |
| `POST /api/tutor-profiles/{id}/scan` | Authorized rescan | New idempotent scan job or existing active job |
| `GET /api/tutor-profiles/{id}/assessment` | View permitted assessment summary | Current score, signals, and status |
| `POST /api/tutor-profiles/{id}/reports` | Report suspicious profile | Validated report and reference ID |
| `GET /api/admin/fraud/review-queue` | Filtered/paginated admin queue | Profiles ordered by risk and queue age |
| `GET /api/admin/fraud/assessments/{id}` | Review evidence | Assessment and permitted evidence |
| `POST /api/admin/fraud/assessments/{id}/decision` | Record decision | State updated and audit event written |
| `POST /api/tutor-profiles/{id}/appeals` | Submit appeal | Appeal case created |

### API requirements

- Authenticate every endpoint and enforce role-based authorization.
- Validate request size, MIME signature, required fields, and identifiers.
- Use correlation IDs and structured error responses.
- Make scan creation idempotent for the same profile version.
- Never return embeddings, internal feature vectors, hidden thresholds, or another user's personal data.
- Rate-limit report submission and scan requests.
- Log security events without logging images, documents, access tokens, or unnecessary profile text.

---

## 10. Admin Dashboard Delivery Expectations

### Review queue

- Show profile ID/name, risk level, total score, top reasons, submitted time, queue age, and scan status.
- Filter by risk, date, trigger, decision status, and assigned reviewer.
- Sort by severity and queue age.
- Paginate results; do not load every case at once.

### Case details

- Show the submitted profile and approved document previews.
- Show signals grouped by image, credentials/text, accounts, and ratings.
- For image matches, show authorized thumbnails and similarity type without exposing unrelated private details.
- Show unavailable checks and processing errors separately from clean checks.
- Show scan version, prior decisions, reports, and audit history.

### Decision controls

- Approve.
- Request additional information.
- Restrict visibility pending evidence.
- Suspend or reject, only for authorized roles.
- Escalate.
- Record structured reason and optional notes.

The dashboard must require confirmation for adverse actions and must prevent an administrator from reviewing actions outside their assigned permissions.

---

## 11. Step-by-Step Delivery Plan for a New member

### Phase 0 — Understand the existing system (6 hours)

1. Run the portal, API, database, and tests locally.
2. Identify current authentication, profile, image upload, rating, and admin modules.
3. Draw a simple current-state request flow.
4. Confirm coding standards, branch/PR process, secret handling, and environments.
5. Write assumptions and unanswered questions in the project README.

**Deliverable:** Current-state notes, setup instructions, and confirmed API/database conventions.

### Phase 1 — Clarify requirements and prepare test data (10 hours)

1. Define profile states and allowed transitions.
2. Agree risk reason codes and initial score weights.
3. Define what administrators may see and do.
4. Create a consented, de-identified test dataset with positive, negative, and ambiguous examples.
5. Split labelled examples into development, validation, and locked test sets by person/account group to prevent leakage.

**Deliverable:** Requirements checklist, data dictionary, labelled dataset manifest, and acceptance-test mapping.

### Phase 2 — Database and scan pipeline (16 hours)

1. Create migration scripts for the minimum tables.
2. Add indexes for profile, scan status, risk level, created time, and hashes.
3. Implement scan job states: `Queued`, `Running`, `Completed`, `Failed`, `NeedsReview`.
4. Implement retry with a maximum-attempt policy and dead-letter/manual-retry state.
5. Ensure one profile version does not create duplicate concurrent jobs.

**Deliverable:** Versioned migrations, seed data, repository layer, and integration tests.

### Phase 3 — Image detector (22 hours)

1. Implement secure file validation and normalization.
2. Calculate SHA-256 and perceptual hash.
3. Compare against approved active platform images.
4. Add embedding comparison only after the simpler comparison works.
5. Return standardized signals with reason codes and confidence/ distance values.
6. Test rotated, resized, cropped, recompressed, blank, corrupt, and multi-face samples.

**Deliverable:** Image analyzer module, benchmark notebook/report, and automated tests.

### Phase 4 — Text and qualification detector (20 hours)

1. Normalize biography text safely.
2. Extract qualification, institution, role, subject, and date entities.
3. Implement timeline and claim consistency rules.
4. Implement biography similarity against existing profiles.
5. Evaluate a pretrained classifier only if it improves the approved baseline.
6. Return explainable reason codes and text spans visible to authorized reviewers.

**Deliverable:** NLP module, rule configuration, evaluation report, and tests.

### Phase 5 — Account and rating features (14 hours)

1. Implement eligible-review validation using completed lessons.
2. Calculate review velocity, account age, duplicate text, and concentration features.
3. Calculate strong and weak shared-account features.
4. Ensure shared IP and low review count cannot independently create high risk.
5. Save feature calculation date and version.

**Deliverable:** Feature queries/jobs, sample graph view or SQL report, and tests.

### Phase 6 — Scoring and APIs (18 hours)

1. Create versioned score configuration.
2. Cap each signal group to prevent duplicate evidence from inflating the score.
3. Implement orchestration, assessment, reporting, queue, and decision APIs.
4. Add authentication, authorization, validation, pagination, and rate limits.
5. Add idempotency and structured error responses.

**Deliverable:** Documented APIs, API collection/OpenAPI output, and integration tests.

### Phase 7 — Admin dashboard (14 hours)

1. Build queue, filters, sorting, pagination, and empty/error/loading states.
2. Build case detail with grouped evidence.
3. Add decision form, mandatory reason, and confirmation.
4. Ensure sensitive data is role-restricted and safely rendered.

**Deliverable:** Working dashboard with responsive basic layout and UI tests.

### Phase 8 — Evaluation, hardening, and handover (14 hours)

1. Run the locked acceptance dataset.
2. Measure precision, recall, false-positive rate, and review load by risk band.
3. Check performance, security, authorization, concurrency, and failure recovery.
4. Document deployment, configuration, model/rule versions, rollback, and monitoring.
5. Demonstrate the complete flow and hand over known limitations.

**Deliverable:** Test report, demo evidence, runbook, and signed acceptance checklist.

**Total: 134 hours**

---

## 12. Acceptance Test Cases

### Functional acceptance

| ID | Scenario | Steps / test data | Expected result |
|---|---|---|---|
| AT-01 | Valid unique profile | Submit valid image, consistent biography, qualifications, and no linked risk | Scan completes; explainable low-risk result; no processing error |
| AT-02 | Exact image reuse | Submit normalized image already attached to unrelated active profile | Exact-duplicate signal is created with evidence reference; case follows configured review policy |
| AT-03 | Resized image reuse | Resize/compress an existing profile image and submit it | Near-duplicate detector identifies it within validated threshold |
| AT-04 | Cropped image | Submit a moderately cropped approved duplicate | Embedding/perceptual comparison returns evaluated result; no crash; outcome recorded |
| AT-05 | Corrupt upload | Submit truncated or invalid image bytes with `.jpg` extension | API rejects safely with supported 4xx response; no scan or unsafe processing occurs |
| AT-06 | Oversized image | Upload above configured byte/pixel limit | Request rejected with clear validation error; file not retained as valid media |
| AT-07 | No face detected | Submit valid non-face photo | Quality signal or information request is produced; system does not call it proven fraud |
| AT-08 | Contradictory dates | Claim experience before education/working-age policy or end before start | Relevant consistency reason code appears with exact conflicting fields |
| AT-09 | Unsupported PhD claim | Bio claims PhD but qualification list contains no PhD | Claim mismatch signal created; human review can see the claim and absence |
| AT-10 | Copied biography | Submit bio substantially matching another unrelated profile | Text-similarity signal created with match score and permitted evidence |
| AT-11 | Shared household IP only | Two legitimate profiles share an IP but have distinct identity/contact/payment/image data | Shared IP alone does not generate high-risk/rejection outcome |
| AT-12 | Strong multi-account pattern | Two profiles share image plus payout token and device signal | Multiple-account group produces strong explainable risk and review queue entry |
| AT-13 | Review burst | Many new accounts submit top ratings in a short period without eligible lessons | Rating anomaly signals created and case placed in review per policy |
| AT-14 | Legitimate popularity | Established tutor receives a promotion-driven burst from completed lessons | Eligibility and cohort features prevent automatic fraud conclusion; result is explainable |
| AT-15 | Insufficient rating data | Tutor has one legitimate review | Rating module returns insufficient-data state, not a high-risk anomaly |
| AT-16 | Learner report | Authenticated learner reports a profile with valid reason | Report ID returned; duplicate/rate-limit policy applied; scan/review triggered as configured |
| AT-17 | Duplicate scan request | Same profile version is submitted twice concurrently | One active scan is used; no duplicated assessment or inflated signals |
| AT-18 | Worker failure | Force detector timeout | Job retries up to policy limit; final failure is visible and does not appear as a clean result |
| AT-19 | Admin approval | Authorized reviewer approves after review and enters reason | Status changes correctly and immutable audit event records actor, time, reason, and prior state |
| AT-20 | Unauthorized decision | Ordinary tutor calls admin decision endpoint | Access denied; no state change; security event logged appropriately |
| AT-21 | Missing decision reason | Reviewer submits adverse decision without reason | Validation fails and no decision is committed |
| AT-22 | Appeal | Restricted tutor submits permitted appeal | Appeal case created, linked to original assessment, and visible to authorized reviewer |
| AT-23 | Profile update | Tutor changes a scored field | New profile version triggers new assessment; previous assessment remains auditable |
| AT-24 | Evidence privacy | Reviewer with insufficient role opens restricted evidence | Sensitive evidence is hidden/denied and access is auditable |

### Non-functional acceptance

| ID | Requirement | Acceptance expectation |
|---|---|---|
| NFT-01 | API response | Submission endpoint responds within the agreed percentile target because ML runs asynchronously |
| NFT-02 | Scan completion | Agreed percentage of ordinary scans completes within the defined processing SLA under test load |
| NFT-03 | Reliability | Failed detector does not corrupt assessment; unavailable status is visible and retryable |
| NFT-04 | Security | Authentication, role authorization, upload validation, secret scanning, and dependency scanning pass |
| NFT-05 | Privacy | Images, documents, embeddings, and identifiers are encrypted and subject to retention/deletion policy |
| NFT-06 | Auditability | Every score and decision can be reproduced from stored input version, rule/model version, and signals |
| NFT-07 | Accessibility | Admin critical workflow is keyboard usable, labelled, and does not rely on color alone |
| NFT-08 | Observability | Dashboard/alerts expose queue depth, completion latency, failures, retry count, and signal distribution |
| NFT-09 | Maintainability | New detector can be added behind a common interface without modifying unrelated detectors |
| NFT-10 | Data deletion | Approved deletion request removes or de-identifies retained artifacts according to policy and records completion |

### Model-quality acceptance

The product owner must supply target values after reviewing the dataset and risk tolerance. At minimum, the report must include:

- precision and recall for each major signal category;
- false-positive and false-negative counts;
- confusion matrix at each proposed action threshold;
- results by relevant language/profile cohorts where sample size permits;
- percentage sent to manual review;
- reviewer agreement and overturn rate;
- performance on the locked test set only once thresholds are frozen; and
- known failure modes and examples.

Do not claim “90% accuracy” without defining the dataset, class balance, threshold, and metric.

---

## 13. Definition of Done

A task is not done merely because code has been written. The feature is accepted when:

- database migrations apply and roll back safely in a test environment;
- APIs are documented and pass unit/integration tests;
- image, text, account, and rating analyzers return standardized explainable signals;
- the same input and version produce a reproducible score;
- admin queue and decision flow work end-to-end;
- authorization and sensitive-data restrictions have been tested;
- all agreed acceptance tests pass or have approved exceptions;
- evaluation results and thresholds are documented;
- logs, metrics, alerts, retry, and failure states are visible;
- deployment and rollback instructions are tested;
- no secrets or real unapproved personal data exist in source control or test fixtures;
- code has peer review and the branch/PR checklist is complete; and
- demo, runbook, known limitations, and handover notes are delivered.

---

## 14. Required Repository Deliverables

Suggested structure:

```text
fake-profile-detection/
├── README.md
├── docs/
│   ├── architecture.md
│   ├── data-dictionary.md
│   ├── scoring-policy.md
│   ├── privacy-and-security.md
│   ├── evaluation-report.md
│   └── runbook.md
├── src/
│   ├── api/
│   ├── detectors/
│   │   ├── image/
│   │   ├── text/
│   │   ├── account/
│   │   └── rating/
│   ├── scoring/
│   ├── persistence/
│   └── workers/
├── database/
│   ├── migrations/
│   └── seed/
├── dashboard/
├── tests/
│   ├── unit/
│   ├── integration/
│   ├── acceptance/
│   └── fixtures/
├── config/
│   └── scoring.example.yaml
└── deployment/
```

Each pull request must include:

- linked task and acceptance-test IDs;
- summary of behavior changed;
- screenshots or API evidence where relevant;
- tests added and their result;
- database/configuration impact;
- privacy/security impact;
- rollback approach; and
- known limitations.

---

## 15. Dataset Requirements

The dataset is the largest project dependency. Before model work begins, confirm:

- lawful source and permission to use every image, document, biography, rating, and label;
- no production personal data is copied into developer machines without approval;
- labels have written definitions, for example `confirmed_duplicate`, `suspicious`, `legitimate`, and `unknown`;
- ambiguous cases are not forced into “fake” or “real”;
- train/validation/test splits prevent the same person, image family, or coordinated group appearing across splits;
- class imbalance is recorded;
- dataset version and checksum are recorded;
- a deletion/retention process exists; and
- synthetic examples are clearly marked and are not the only positive examples.

If an adequate labelled dataset is unavailable, deliver a **rules-based proof of concept and data-collection plan**, not a falsely confident trained model.

---

## 16. Security, Privacy, and Fairness Checklist

- Obtain explicit policy approval before face comparison.
- Perform a privacy and threat assessment before production release.
- Encrypt data in transit and at rest.
- Keep object storage private and use short-lived authorized access.
- Separate admin, reviewer, support, and tutor permissions.
- Rate-limit uploads, reports, and rescan endpoints.
- Scan uploaded files and validate decoders in an isolated worker.
- Protect against SQL injection, stored XSS from biographies/reviews, IDOR, and malicious file content.
- Do not log raw sensitive documents, tokens, biometric embeddings, or full identity numbers.
- Define retention for uploads, embeddings, signals, reports, and appeals.
- Provide correction and appeal workflows.
- Monitor outcomes for disproportionate false positives.
- Require human review for adverse decisions during the MVP.
- Record changes to scoring configuration and require approval before activation.

---

## 17. Monitoring and Operational Runbook

Monitor:

- scan queue depth and oldest queued job;
- scan completion and failure rate;
- detector latency and timeout rate;
- retry and dead-letter counts;
- scores and signals by version;
- number and age of manual-review cases;
- approval, restriction, rejection, appeal, and overturn rates;
- false positives reported by reviewers;
- changes in input quality or language distribution; and
- unauthorized access attempts.

The runbook must explain:

1. how to pause automated scanning or an individual detector;
2. how to roll back a scoring configuration/model version;
3. how to reprocess affected profiles safely;
4. how to handle a queue backlog;
5. how to respond to exposed images or credentials;
6. how to correct a bad decision without deleting audit history; and
7. who owns product, security, compliance, data, and operational escalation.

---

## 18. Effort Assumptions and Risks

The 134-hour estimate is feasible only for a constrained MVP using existing authentication, profile/rating data, UI components, infrastructure, and pretrained/baseline methods.

| Risk | Impact | Response |
|---|---|---|
| No labelled dataset | Cannot train or validate credible model | Deliver rules baseline and approved data-collection plan |
| External credential verification expected | New vendors, contracts, integration, and legal review | Treat as separate phase |
| Internet-wide stolen-image detection expected | Search/API/licensing and privacy complexity | Limit MVP to platform corpus or approved provider |
| Face API approval unavailable | Biometric feature cannot ship | Use non-biometric hashes and manual review |
| Existing rating/lesson linkage is incomplete | Review anomalies may be unreliable | Fix data lineage or mark feature unavailable |
| Admin UI framework is not reusable | Dashboard takes longer | Revise estimate after technical discovery |
| Production scale is high | Performance and infrastructure work expands | Run load model and revise architecture |
| Multilingual profiles required | NLP accuracy varies by language | Define supported languages and evaluate each |

### Recommended milestone acceptance

- **Milestone 1:** Secure submission, SQL schema, scan jobs, and exact/near-duplicate image baseline.
- **Milestone 2:** Text consistency, duplicate biography, rating/account features, and explainable scoring.
- **Milestone 3:** Admin dashboard, decision/audit flow, security hardening, evaluation, and handover.

---

## 19. Questions the member Must Resolve Before Coding

1. What fields and documents are mandatory for each tutor category?
2. Is profile publication blocked until review, or can low-risk profiles publish?
3. What is the precise meaning of approve, restrict, suspend, reject, and appeal?
4. Who can view qualification documents and matched profile images?
5. Is biometric face comparison legally and organizationally approved?
6. What external credential sources, if any, are authorized?
7. What data retention and deletion periods apply?
8. How are eligible lessons linked to ratings?
9. What review volume and turnaround SLA must the system support?
10. Which languages must be supported at launch?
11. What precision/recall and false-positive targets are acceptable?
12. Who approves threshold or model changes?

No member should invent answers to these policy questions. Record them, obtain an owner, and keep blocked work visible.

---

## 20. Final Demo Script

The member should demonstrate these cases without modifying the database manually:

1. Submit a legitimate profile and show scan completion.
2. Submit an exact or near-duplicate image and show the evidence signal.
3. Submit contradictory qualification text and show the reason code.
4. Generate a coordinated rating pattern using test fixtures and show the anomaly.
5. Open the review queue, filter cases, inspect evidence, and make a decision.
6. Show the audit trail and previous assessment after a profile update.
7. Attempt an admin action with a tutor account and show access denial.
8. Force a detector failure and show retry/failure handling.
9. Submit an appeal and show the linked case.
10. Show the evaluation report, monitoring view, known limitations, and rollback procedure.

The final delivery must emphasize that AI assists investigation; authorized humans remain accountable for consequential decisions.
