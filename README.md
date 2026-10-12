# SmartFit: The Member Intelligence Challenge

## Data Curation Architecture

## 1. Introduction

SmartFit is a digital fitness platform that collects member and workout
information from its mobile app, fitness coaches, wearable devices, and
customer support team. Different teams use this information for
coaching, member support, and business reporting.

However, records can conflict across systems, business teams may define
key metrics differently, and workout information can be corrected after
it is first recorded. SmartFit therefore needs a reliable data curation
process that turns source data into consistent, validated, traceable
information for downstream use.

## 2. Objective

Design a central data curation flow that: - Integrates data from
multiple source systems while preserving original payloads and
history. - Checks schemas and data quality before data is used. -
Resolves member identities and prevents duplicate member records. -
Applies agreed business definitions and transformation rules. - Produces
trusted data products for coaches, Member Success, and business teams. -
Tracks lineage, audit history, quality, access, and corrections.

## 3. Business Problems and Proposed Solutions

1.  **The same member appears differently across systems.**\
    **Solution:** Standardize relevant fields, deduplicate records, and
    use identity matching. Send uncertain matches to an identity review
    queue rather than merging them automatically.

2.  **Member names and contact details change over time.**\
    **Solution:** Maintain a resolved member identity while preserving
    source identifiers and change history. Apply approved matching rules
    and retain an audit trail.

3.  **Teams disagree on the meaning of "active member."**\
    **Solution:** Define the metric centrally in the semantic layer,
    agree on its calculation and time window with business owners, and
    publish one approved definition for all reports. Keep subscription
    status and recent platform activity as distinct attributes.

4.  **Data may be incomplete, invalid, or incompatible with the expected
    schema.**\
    **Solution:** Check schema versions, fields, and data types at
    ingestion. Quarantine incompatible records; validate record quality
    and quarantine invalid records until corrected or an approved rule
    is applied.

5.  **Workout/session information can be corrected after it is
    recorded.**\
    **Solution:** Preserve raw history, track corrections and their
    reasons, and update curated records using controlled transformation
    logic. Ensure downstream views reflect the latest approved state
    while retaining auditability.

## 4. Architecture Flow and Key Terminology

![SmartFit Data Curation
Architecture](image.png)

### End-to-end flow

1.  **Source systems:** The mobile app, fitness coaches, wearable
    devices, and customer support create member, membership, and
    workout-related data.
2.  **Ingestion, metadata, and retry handling:** Data is collected and
    operational metadata is recorded. Failed transient ingestion
    attempts can be retried according to defined policies.
3.  **Raw data landing:** Store original payloads and history so that
    data can be reprocessed and investigated without losing the source
    record.
4.  **Schema registry / contract check:** Validate the incoming
    structure, schema version, required fields, and data types against
    an agreed data contract.
5.  **Schema quarantine:** Hold records that do not match the expected
    schema until the schema is fixed or an approved mapping is provided.
6.  **Record validation and data-quality checks:** Check required
    values, valid ranges, formats, and business rules. Invalid records
    go to **data quality quarantine** for correction or an approved
    rule-based resolution.
7.  **Standardization and identity resolution:** Normalize values such
    as names, phone numbers, and identifiers where appropriate.
    Deduplicate records and match records to a member identity.
8.  **Identity confidence and review:** Automatically resolve
    sufficiently confident matches under approved thresholds. Route
    uncertain matches to the **identity review queue**; use additional
    evidence and review before merging.
9.  **Resolved, deduplicated record:** Produce a trusted member-level
    record while retaining links to source records and the history
    needed for audit.
10. **Curated data models:** Organize validated information into
    consistent models for **Members, Memberships, and Workouts**.
11. **Semantic layer:** Publish shared, approved business definitions,
    including the agreed "active member" metric, so that different teams
    calculate and interpret metrics consistently.
12. **Data products:** Serve role-relevant outputs such as the **Member
    Success View**, **Coach Progress View**, and **Business Metrics and
    Dashboards**.
13. **Lineage and governance across the flow:** Track where data came
    from, what changed, quality outcomes, who accessed it, and how
    corrections were handled.

### Important terminology

-   **Data curation:** Preparing source data so it is validated,
    standardized, integrated, documented, and fit for reliable use.
-   **Data contract / schema registry:** The agreed structure and
    version of data expected from a source.
-   **Quarantine:** A controlled holding area for records that must not
    proceed until an issue is resolved.
-   **Standardization:** Converting inconsistent representations into an
    agreed format.
-   **Entity resolution:** Determining whether records from different
    sources refer to the same real-world member.
-   **Deduplication:** Identifying and consolidating duplicate records
    without incorrectly combining different people.
-   **Curated data model:** A structured, trusted representation of
    business entities and their relationships.
-   **Semantic layer:** A shared business-meaning layer that defines
    metrics and dimensions consistently.
-   **Data product:** A usable, purpose-specific dataset, view, or
    dashboard built from curated data.
-   **Data lineage:** The ability to trace data from its source through
    transformations to its downstream use.
-   **Audit history:** A record of changes, decisions, timestamps, and
    relevant actors.
-   **Quality monitoring and control:** Monitoring data-quality results
    and applying defined checks or corrective actions.
-   **Access control:** Restricting data and data products according to
    role and authorization.
-   **Correction tracking:** Recording corrections, their reasons, and
    their impact on curated data.

## 5. Edge Cases and Resolutions

  - **A source adds, removes, or changes a field:** Detect the change through schema/contract checks. Quarantine incompatible records until a schema update or approved mapping is available.
- **A required field is missing or a value is invalid:** Route the record to data-quality quarantine. Correct it or apply an approved rule, then validate it again.
- **Two records look similar but may belong to different members:** Use identity-confidence thresholds and send ambiguous matches to manual review. Avoid automatic merging when evidence is weak.
- **One member changes their name or contact details:** Preserve stable source identifiers and change history. Update the resolved identity using approved matching rules.
- **A coach corrects a previously completed workout:** Preserve the original event/history, record the correction and its reason, then refresh the curated workout record and downstream views.
- **The definition of “active member” changes:** Version the centrally approved metric definition and communicate its effective date so reports remain interpretable over time.
- **The same event is ingested more than once:** Use source event identifiers or other idempotency keys, where available, to detect duplicate deliveries before they create duplicate curated records.
- **A source or ingestion job is temporarily unavailable:** Use retry handling for transient failures, monitor persistent failures, and retain enough metadata to resume safely.
- **A downstream team tries to access data it is not authorized to see:** Apply role-based access controls and expose only the fields and data products required for that role.
- **A dashboard number cannot be explained:** Trace the metric through its semantic definition, curated model, transformation history, and source lineage. Investigate the relevant audit and quality records.


 
  -----------------------------------------------------------------------

## 6. Conclusion

The proposed architecture creates a controlled path from source data to
trusted SmartFit data products. Schema and quality checks prevent
unsuitable records from silently entering curated datasets. Identity
resolution reduces duplicates while review queues protect against
incorrect merges. Curated models and a shared semantic layer establish
consistent data structures and business meanings. Finally, lineage,
audit history, quality controls, access control, and correction tracking
make the results easier to explain, govern, and trust.

**Design principle:** Do not silently discard, merge, or overwrite
uncertain data. Preserve evidence, apply approved rules, and make
important decisions traceable.
