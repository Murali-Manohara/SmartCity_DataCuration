# SmartFit: The Member Intelligence Challenge

## Data Curation Architecture

**Prepared by:** Murali Manohara and Ullas\
**Focus:** Data Curation Foundations

## 1. Introduction

SmartFit is a digital fitness platform that collects member, membership,
workout, and activity information from its mobile app, fitness coaches,
wearable devices, and customer support team. Because these sources
represent information differently and may be updated at different times,
teams can end up with conflicting or duplicate records.

This proposal introduces a central data curation architecture that
prepares consistent, trustworthy, and traceable data for Coaches, the
Member Success Team, and the Business Team.

## 2. Objectives

-   Establish a reliable, central view of members and their activities.
-   Identify duplicate records and resolve member identities carefully.
-   Standardize and validate data before downstream use.
-   Establish shared business definitions, especially for "active
    member."
-   Preserve corrections and enough history to explain how data changed.
-   Track data lineage so users can understand where curated information
    came from.

## 3. Business Problems and Proposed Solutions

1.  **Member records differ across systems:** The same person may have
    different names, contact details, or identifiers.\
    **Solution:** Use deduplication and member identity resolution to
    link records to a canonical member ID. Send uncertain matches to an
    identity review queue instead of merging automatically.

2.  **Member details change over time:** Names and contact details may
    be updated.\
    **Solution:** Maintain a canonical member record and retain relevant
    versions or effective dates so current values and historical values
    can be distinguished.

3.  **Teams disagree on "active member":** Subscription status and
    recent platform activity are different measures.\
    **Solution:** Define approved business metrics in the semantic
    layer. For example, report "subscribed members" separately from
    "engaged active members," with an agreed activity window and
    qualifying events.

4.  **Workout records can be corrected later:** A workout first marked
    complete may later be identified as interrupted.\
    **Solution:** Preserve the original record and record the accepted
    correction as a traceable version or event. Recompute affected
    curated data and downstream views.

5.  **Teams need reliable, explainable information:** Source data may be
    invalid, inconsistent, duplicated, or difficult to trace.\
    **Solution:** Apply schema and data-quality checks, quarantine
    problematic records, standardize valid data, and maintain lineage,
    audit history, access controls, and correction tracking.

## 4. Architecture Flow

The architecture is organized into five stages, with cross-cutting
controls that support the full pipeline.

1.  **Source Systems**
    -   **Mobile App:** Member profiles, app usage, and workout
        activity.
    -   **Fitness Coaches:** Session outcomes, progress observations,
        and corrections.
    -   **Wearable Devices:** Activity and fitness measurements.
    -   **Customer Support:** Member detail updates and support-related
        information.
2.  **Ingestion and Raw Storage**
    -   **Ingestion:** Receives data from the source systems.
    -   **Metadata and Retry Handling:** Records useful source/load
        metadata and retries recoverable failures.
    -   **Raw Data Landing:** Preserves original payloads and history
        for investigation and reprocessing.
    -   **Schema Registry / Contract Check:** Checks expected fields,
        data types, and schema versions.
    -   **Schema Compatibility:** Determines whether an incoming schema
        can be processed safely.
    -   **Schema Quarantine:** Holds incompatible data until a mapping,
        fix, or approved change is available.
3.  **Validation and Identity Resolution**
    -   **Record Validation:** Checks required values, formats, valid
        ranges, timestamps, and other quality rules.
    -   **Data Quality Quarantine:** Holds invalid records rather than
        allowing them to contaminate curated data.
    -   **Standardization:** Normalizes formats and values, then reruns
        appropriate checks after correction.
    -   **Deduplication and Member Identity Matching:** Identifies
        duplicate events and records likely to represent the same
        member.
    -   **Identity Match Confidence:** High-confidence matches can
        proceed under approved rules; uncertain matches enter the
        **Identity Review Queue** for human review or later resolution.
4.  **Curated Data and Business Rules**
    -   **Curated Data Models:** Organize trusted data into reusable
        member, membership, and workout datasets.
    -   **Semantic Layer:** Holds approved business definitions and
        calculations, including clearly specified member activity
        metrics.
5.  **Data Products**
    -   **Coach Progress View:** Supports member-level progress and
        workout review.
    -   **Member Success View:** Supports identifying members who may
        need assistance.
    -   **Business Metrics and Dashboards:** Supports consistent
        membership and engagement reporting.

**Cross-cutting controls:** Lineage, audit history, quality monitoring,
access control, and correction tracking operate across the flow. They
record where data originated, what checks and transformations were
applied, how identity decisions were made, and which corrections
affected curated outputs.

## 5. Key Terminology

  -----------------------------------------------------------------------
  Term                                Meaning in this architecture
  ----------------------------------- -----------------------------------
  Ingestion                           Bringing data from source systems
                                      into the central platform.

  Raw data landing                    A retained copy of received data
                                      before curation.

  Schema / data contract              An agreement about expected fields,
                                      types, and structure.

  Schema quarantine                   A holding area for data that cannot
                                      safely pass schema checks.

  Data quality quarantine             A holding area for records that
                                      fail validation.

  Standardization                     Converting values and formats into
                                      agreed conventions.

  Deduplication                       Detecting and handling repeated
                                      records or events.

  Entity resolution                   Determining whether records from
                                      different systems refer to the same
                                      real-world member.

  Curated data model                  A structured, quality-checked
                                      dataset prepared for reuse.

  Semantic layer                      A shared place for approved
                                      business terms, metrics, and
                                      calculation rules.

  Data lineage                        The ability to trace data from its
                                      source through transformations to
                                      its curated output.

  Audit history                       A record of changes, decisions, and
                                      processing actions.

  Data product                        A prepared dataset or view designed
                                      for a particular team's use.
  -----------------------------------------------------------------------

## 6. Edge Cases and Resolutions

  -----------------------------------------------------------------------
  Edge case                           Resolution
  ----------------------------------- -----------------------------------
  Same member has different names or  Match using stable identifiers and
  contact details                     multiple approved signals; retain a
                                      canonical ID and relevant history.

  Two members share the same name     Do not merge on name alone; require
                                      stronger evidence or manual review.

  Identity match is ambiguous         Send it to the Identity Review
                                      Queue and keep records separate
                                      until resolved.

  Source sends an incompatible schema Quarantine the payload; review the
  change                              contract and approve a mapping or
                                      schema change before processing.

  Record has missing or invalid       Quarantine or flag it; correct it
  values                              at the source or through an
                                      approved process, then revalidate.

  Same event arrives more than once   Deduplicate using source event IDs
                                      or documented matching rules.

  Wearable events arrive late or out  Retain event time and ingestion
  of order                            time; apply a documented
                                      late-arrival policy and refresh
                                      affected outputs.

  Coach corrects a previously         Preserve the earlier version,
  completed workout                   record the correction, update
                                      curated results, and retain an
                                      audit trail.

  Teams report different              Use the same approved semantic
  active-member totals                definition, qualifying events, time
                                      window, exclusions, and refresh
                                      rules.

  Data contains sensitive contact or  Apply role-based access, data
  fitness information                 minimization, and appropriate
                                      retention and privacy controls.

  A transformation or metric          Version the rule, record lineage,
  definition changes                  and reprocess affected data when
                                      required.
  -----------------------------------------------------------------------

## 7. Conclusion

SmartFit should treat data curation as a governed process rather than a
one-time cleaning task. The proposed architecture preserves source
history, validates and standardizes data, resolves member identity with
confidence checks, centralizes business definitions, and provides
team-specific data products. Cross-cutting lineage, audit, quality, and
access controls make the resulting information easier to trust and
explain.

**Expected outcome:** Coaches, Member Success, and the Business Team can
use consistent, purpose-fit information while being able to trace how
important values were produced.

## Design Note

This is a logical architecture, not a vendor-specific implementation.
The case study does not specify data volume, latency targets, cloud
provider, budget, or existing technology. Those details should be
confirmed before selecting products or deciding between batch and
streaming processing.
