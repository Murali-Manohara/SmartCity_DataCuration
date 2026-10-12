# SmartFit: The Member Intelligence Challenge

## Data Curation Architecture \| Phase 1: Business Understanding and Initial Design

**Prepared by:** Murali Manohara and Ullas\
**Project type:** Data Curation Foundations case study

------------------------------------------------------------------------

## 1. Executive Summary

SmartFit receives member and activity information from its mobile app,
fitness coaches, wearable devices, and customer support team. Because
each source captures information for a different purpose, records may
conflict, refer to the same person using different details, or be
corrected after the original event.

We recommend a central **curated data layer** that standardizes source
data, resolves member identities, applies agreed business definitions,
preserves corrections and data lineage, and publishes trusted datasets
for coaches, Member Success, and the Business Team.

The goal is not simply to collect data in one place. It is to make the
information **consistent, explainable, traceable, and fit for each
team's decisions**.

## 2. Business Context and Requirements

### Teams and their needs

  -----------------------------------------------------------------------
  Team                    Main questions          Data needs
  ----------------------- ----------------------- -----------------------
  Coaches                 Is a member             Member-level progress,
                          progressing? Were       workout and session
                          workouts completed or   history, coach
                          interrupted?            corrections

  Member Success          Which members may need  Recent engagement,
                          support?                attendance, progress
                                                  patterns, reliable
                                                  member identity

  Business Team           How is the membership   Consistent
                          base performing?        active-member counts,
                                                  engagement trends,
                                                  membership metrics
  -----------------------------------------------------------------------

### Key requirements

1.  **Combine multiple sources:** Bring together information from the
    mobile app, coaches, wearables, and customer support.
2.  **Resolve member identity:** Recognize when differently represented
    records belong to the same real-world member.
3.  **Handle changing details:** Keep member names and contact details
    current without losing relevant history.
4.  **Agree on business metrics:** Define "active member" once,
    including its calculation and time window.
5.  **Manage corrections:** Allow a coach's later correction to a
    workout record to update trusted outputs while preserving the
    correction history.
6.  **Prepare reliable data:** Standardize formats, validate records,
    and handle missing, invalid, duplicate, or conflicting values.
7.  **Trace data:** Show where curated values came from and which
    transformations or corrections were applied.
8.  **Serve different teams:** Provide consistent core definitions while
    allowing each team to use relevant views.

## 3. Recommended Architecture

The design below is a **logical architecture**. It describes
responsibilities and data movement rather than committing to a
particular vendor or product.

``` mermaid
flowchart TD
    subgraph S["1. Source Systems"]
        A["Mobile App"]
        B["Fitness Coaches"]
        C["Wearable Devices"]
        D["Customer Support"]
    end

    subgraph P["2. Central Data Platform"]
        E["Ingestion and Source Metadata"]
        F["Raw / History Layer"]
        G["Quality Checks and Standardization"]
        H["Member Identity Resolution"]
        I["Curated Member and Activity Data"]
        J["Business Rules and Semantic Layer"]
        K["Data Lineage and Audit History"]
    end

    subgraph U["3. Trusted Data Products and Consumers"]
        L["Coach Progress View"]
        M["Member Success View"]
        N["Business Metrics and Dashboards"]
    end

    A --> E
    B --> E
    C --> E
    D --> E
    E --> F
    F --> G
    G --> H
    H --> I
    I --> J
    E -. "source and load metadata" .-> K
    G -. "quality and transformation records" .-> K
    H -. "match decisions" .-> K
    I -. "curated dataset versions" .-> K
    J --> L
    J --> M
    J --> N
```

**How to read the diagram:** Source data is ingested and retained,
checked and standardized, matched to the correct member, and organized
into curated datasets. Shared business definitions are applied before
the data is delivered to team-specific views. Lineage and audit records
capture evidence about the data's origin and processing.

## 4. What Each Layer Does and Why

  -----------------------------------------------------------------------
  Layer / capability      Responsibility          Reason for including it
  ----------------------- ----------------------- -----------------------
  Ingestion and source    Collect data and record Helps identify where
  metadata                source, arrival time,   data came from and when
                          and relevant            it arrived
                          identifiers             

  Raw / history layer     Retain received         Supports reprocessing,
                          records, including      investigation, and
                          earlier versions where  comparison with later
                          practical               corrections

  Quality checks and      Check required fields,  Prevents avoidable
  standardization         types, valid ranges,    errors from flowing
                          timestamps, duplicates, into trusted datasets
                          and formats             

  Member identity         Match records using     Avoids counting or
  resolution              reliable identifiers    analyzing one member as
                          and carefully selected  several people
                          attributes              

  Curated member and      Publish cleaned,        Gives teams a reusable,
  activity data           linked, consistently    trusted foundation
                          structured data         

  Business rules and      Centralize definitions  Prevents teams from
  semantic layer          such as "active member" producing conflicting
                          and standard metric     numbers
                          calculations            

  Lineage and audit       Record source           Makes results
  history                 references,             explainable and
                          transformation          supports debugging and
                          versions, match         review
                          decisions, and          
                          corrections             

  Team-specific data      Provide fit-for-purpose Serves different needs
  products                views for coaches,      without duplicating
                          Member Success, and     core definitions
                          business reporting      
  -----------------------------------------------------------------------

## 5. Important Design Decisions

### A. Separate raw data from curated data

Keep an unmodified or historically preserved copy of received data, then
create curated versions through documented transformations. This allows
SmartFit to investigate issues and rerun processing when rules change.
Access to raw personal data should be restricted.

### B. Resolve identities carefully

Use a stable member ID where available. Match records using a hierarchy
of evidence, such as trusted shared identifiers first and combinations
of normalized attributes only when appropriate. Do not automatically
merge records based on name alone. Ambiguous matches should be flagged
for review rather than guessed.

### C. Treat changing member details as history, not just overwrites

Maintain a canonical member record linked to source identifiers. For
fields where history matters, retain effective dates or record versions
so SmartFit can distinguish the current value from the value that was
valid at an earlier time. Protect contact details through access
controls.

### D. Define "active member" centrally

Agree on the intended business meaning before implementing the metric.
For example, SmartFit could define an **engaged active member** as a
member with at least one qualifying platform activity in a rolling
30-day window, while reporting **subscribed members** as a separate
metric. The exact qualifying events, time window, exclusions, and
refresh schedule must be agreed with stakeholders. Do not silently treat
subscription status and recent engagement as the same thing.

### E. Handle corrections explicitly

A coach's correction should create a traceable new version or correction
event rather than silently erasing the original record. Curated outputs
should use the latest accepted correction according to a defined rule,
while preserving enough history to explain what changed and when.
Downstream views should be refreshed or recomputed when relevant
corrections arrive.

### F. Make quality and lineage visible

Track checks such as missing member IDs, invalid timestamps, duplicate
events, impossible durations, unmatched identities, and conflicting
completion statuses. Record failures and route them for correction or
quarantine. Keep lineage from source record through transformation and
curated output, including the rule or version used.

## 6. Edge Cases and Recommended Handling

  -----------------------------------------------------------------------
  Edge case                           Recommended handling
  ----------------------------------- -----------------------------------
  Same person has different names or  Match using stable IDs and multiple
  contact details                     signals; keep a canonical ID and
                                      history

  Two different members share a name  Never merge on name alone; use
                                      stronger identifiers or manual
                                      review

  Wearable data arrives late or out   Store event time separately from
  of order                            ingestion time; process late
                                      arrivals according to a documented
                                      policy

  Coach changes "completed" to        Store a correction/version, update
  "interrupted"                       curated results, and retain audit
                                      history

  Required fields are missing or      Quarantine or flag the record; do
  values are invalid                  not silently invent values

  Duplicate activity event is         Use source event IDs or
  delivered more than once            deduplication rules where available

  Teams disagree about active-member  Publish a centrally defined metric
  counts                              and document its window, qualifying
                                      events, and exclusions

  Identity match is uncertain         Keep records separate pending
                                      review rather than risk merging two
                                      people

  Source systems disagree             Apply documented source authority
                                      and recency rules by field;
                                      escalate unresolved conflicts

  A transformation rule changes       Version the rule, record lineage,
                                      and reprocess affected data where
                                      necessary
  -----------------------------------------------------------------------

## 7. Why This Approach?

The central curated layer provides a balance between consistency and
flexibility:

-   **More reliable than separate team-specific pipelines:** Shared
    identity and metric definitions reduce conflicting results.
-   **More explainable than overwriting source records:** Retained
    history and lineage help investigate corrections and data-quality
    issues.
-   **Safer than aggressive automatic matching:** Uncertain identities
    are reviewed instead of incorrectly combined.
-   **More flexible than one giant report table:** Shared curated
    entities can support different views and future use cases.
-   **Technology-neutral:** The design can be implemented with the
    organization's existing warehouse or lakehouse, orchestration,
    transformation, and catalog tools. A specific product should be
    selected only after requirements for scale, cost, skills, security,
    and latency are known.

## 8. Assumptions and Open Decisions

This case study does not specify SmartFit's data volume, cloud provider,
budget, latency targets, or existing technology stack. Therefore, this
design does not prescribe specific vendor tools.

Before implementation, stakeholders should agree on: - Which events
count as meaningful activity and the definition of "active member." -
Which source is authoritative for each field, such as subscription
status, contact details, and workout outcome. - Which identifiers can
safely and legally be used for identity matching. - How quickly
corrected data must reach downstream views. - Retention, privacy,
consent, and access-control requirements. - Who approves ambiguous
identity matches and metric-definition changes.

## 9. Conclusion

SmartFit should treat curation as a governed process, not just a
cleaning step. A central curated layer with identity resolution, shared
metric definitions, correction handling, data-quality checks, and
lineage will help all three teams use consistent information while
retaining the ability to explain how that information was produced.

**Core principle:** One trusted, traceable foundation; shared
definitions; team-specific views.

------------------------------------------------------------------------

## Suggested GitHub file

Save this report as `README.md` in the SmartFit project repository. The
Mermaid diagram is supported by GitHub Markdown and can render directly
on the repository page.
