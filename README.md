# SmartFit: The Member Intelligence Challenge

## Data Curation Architecture

**Project:** Curation Foundations\
**Prepared by:** Murali Manohara and Ullas

------------------------------------------------------------------------

## 1. Introduction

SmartFit is a digital fitness platform that helps members follow workout
plans, attend live sessions, and track their progress. Its information
comes from four main sources: the mobile app, fitness coaches, wearable
devices, and the customer support team.

Each source captures different aspects of a member's journey. Before
this information can be trusted for coaching, member support, or
business reporting, SmartFit must bring it together, check its quality,
identify which records belong to the same member, apply consistent
business definitions, and preserve a clear history of how the data was
prepared.

This proposal presents a central data curation architecture that turns
information from multiple systems into consistent, traceable, and useful
data products for SmartFit's teams.

## 2. Business Problems and Proposed Solutions

  -----------------------------------------------------------------------
  Business problem        Proposed solution       Why this helps
  ----------------------- ----------------------- -----------------------
  **Member records do not Use **deduplication and Reduces duplicate
  match across systems.** member identity         member counts while
  The same person may     resolution** to link    avoiding unsafe
  appear under different  records to a canonical  automatic merges.
  names or contact        member ID. Use strong   
  details.                identifiers first and   
                          send uncertain matches  
                          to an identity review   
                          queue.                  

  **Member details change Maintain curated member Gives teams a
  over time.** A name or  records with source     consistent current view
  contact detail may be   identifiers and         while retaining history
  updated in one system   relevant change         needed to understand
  but not another.        history. Define which   earlier values.
                          source is authoritative 
                          for each field.         

  **Teams disagree about  Define approved         Ensures dashboards and
  "active member."** One  business metrics in a   teams use the same
  team uses subscription  **semantic layer**.     meaning and
  status; another uses    Keep "subscribed        calculation.
  recent platform         member" and "engaged    
  activity.               active member" distinct 
                          unless stakeholders     
                          formally agree          
                          otherwise. Document     
                          qualifying events, time 
                          window, exclusions, and 
                          refresh schedule.       

  **Workout information   Preserve the original   Keeps reports accurate
  may be corrected        payload and correction  without losing the
  later.** A session      history. Apply an       ability to explain what
  first marked completed  accepted correction     changed.
  may later be identified through documented      
  as interrupted.         transformation rules    
                          and refresh affected    
                          curated data products.  

  **Incoming data can be  Run schema/contract     Prevents unreliable
  incomplete, invalid,    checks, validation,     records from silently
  duplicated, or          standardization,        contaminating trusted
  inconsistent.**         deduplication, and      data.
                          data-quality checks.    
                          Quarantine records that 
                          fail checks until       
                          corrected or reviewed.  

  **Teams cannot easily   Maintain **data         Makes data easier to
  explain where a value   lineage, audit history, investigate, validate,
  came from.**            quality monitoring, and and trust.
                          correction tracking**   
                          across ingestion,       
                          transformations,        
                          identity decisions, and 
                          curated outputs.        

  **Different teams need  Publish curated data    Supports each team's
  different views of the  products: a Coach       work without creating
  same information.**     Progress View, Member   separate, conflicting
                          Success View, and       versions of core
                          Business Metrics and    metrics.
                          Dashboards. Base them   
                          on shared curated       
                          models and approved     
                          definitions.            
  -----------------------------------------------------------------------

## 3. Final Architecture

The following diagram is the final proposed architecture for this case
study. It shows the source systems, ingestion and raw storage, schema
and quality handling, identity resolution, curated models, shared
business definitions, data products, and cross-cutting governance
controls.

![SmartFit final data curation architecture](smartfit-architecture.png)

### Architecture flow

1.  **Source systems:** The mobile app, fitness coaches, wearable
    devices, and customer support provide member, membership, and
    workout information.
2.  **Ingestion and raw storage:** Ingestion collects records and
    handles retries and metadata. Raw data landing preserves original
    payloads and history so records can be investigated or reprocessed.
3.  **Schema and contract checks:** The platform checks expected fields,
    data types, and schema compatibility. Incompatible data is
    quarantined until an approved mapping or fix is available.
4.  **Validation and identity resolution:** Records are validated and
    standardized. Invalid records go to a data-quality quarantine.
    Deduplication and member matching link records to the correct member
    when confidence is sufficient; uncertain matches go to an identity
    review queue.
5.  **Curated data and business rules:** Valid, resolved records feed
    curated models for members, memberships, and workouts. The semantic
    layer provides approved business definitions and calculations.
6.  **Data products:** Shared curated data powers the Coach Progress
    View, Member Success View, and Business Metrics and Dashboards.
7.  **Cross-cutting controls:** Lineage, audit history, quality
    monitoring, access control, and correction tracking support the flow
    across the platform rather than acting as a single final processing
    step.

### Important design rules

-   **Do not merge on name alone.** Prefer stable identifiers and
    multiple signals; route uncertain matches to review.
-   **Do not silently discard failed records.** Quarantine them with a
    reason so they can be corrected and reprocessed.
-   **Do not confuse subscription with engagement.** Define separate
    metrics where they represent different business concepts.
-   **Do not overwrite corrections without a trace.** Preserve source
    history and record which correction was accepted and when.
-   **Do not treat lineage as an afterthought.** Record source and
    processing details throughout the data flow.
-   **Protect member information.** Apply role-based access and
    appropriate retention rules, especially for contact details and
    wearable data.

## 4. Edge Cases and How the Architecture Handles Them

  -----------------------------------------------------------------------
  Edge case                           Handling in the proposed
                                      architecture
  ----------------------------------- -----------------------------------
  Same member appears with different  Identity resolution uses stable IDs
  names or contact details            and supporting attributes; a
                                      canonical member ID connects the
                                      records while relevant changes are
                                      retained.

  Two different members have the same Name alone is not enough to merge
  name                                records. Low-confidence matches are
                                      sent to the identity review queue.

  A source adds, removes, or changes  Schema registry/contract checks
  a field                             detect incompatibility. The record
                                      is quarantined until a compatible
                                      mapping or approved schema change
                                      is available.

  A record is missing required data   Validation flags the record and
  or contains invalid values          sends it to data-quality quarantine
                                      for correction or an approved
                                      exception.

  A source sends the same event more  Deduplication uses source event
  than once                           identifiers or documented matching
                                      rules where available.

  A wearable event arrives late or    Retain both event time and
  out of order                        ingestion time; apply a documented
                                      late-arrival policy and recompute
                                      affected curated outputs when
                                      needed.

  A coach corrects a completed        Preserve the original and
  workout to interrupted              correction history, apply the
                                      accepted correction, and update
                                      affected curated models and views.

  Identity evidence is insufficient   Keep the records separate and place
                                      the case in the identity review
                                      queue. Re-evaluate when more
                                      evidence becomes available.

  Teams produce different             Use approved semantic-layer
  active-member counts                definitions with explicit
                                      qualifying activity, time window,
                                      exclusions, and refresh rules.

  A transformation or business        Version the rule, retain
  definition changes                  lineage/audit details, and
                                      reprocess affected data when
                                      appropriate.

  Unauthorized users try to access    Apply role-based access controls
  sensitive member details            and limit each team's data view to
                                      what it needs.
  -----------------------------------------------------------------------

## 5. Conclusion

SmartFit's challenge is not simply collecting data from multiple
sources; it is making that data consistent, correct, and explainable
before teams use it. The proposed architecture addresses this through
schema and quality checks, identity resolution, curated data models, a
shared semantic layer, correction history, and cross-cutting lineage and
governance controls.

This gives coaches a more dependable view of member progress, helps
Member Success identify members who may need support, and enables the
Business Team to report membership performance using consistent
definitions.

**Final recommendation:** Build one trusted and traceable curation
foundation, then provide role-appropriate data products to each team.
Confirm metric definitions, source authority, identity-matching rules,
privacy requirements, and correction service expectations with
stakeholders before implementation.

------------------------------------------------------------------------

## Scope and assumptions

The case study does not specify a cloud provider, technology stack, data
volume, budget, or required freshness target. Therefore, this is a
**logical architecture**, not a vendor-specific implementation. Tools
should be selected after SmartFit confirms its scale, latency needs,
existing platform, team skills, cost constraints, and privacy
obligations.
