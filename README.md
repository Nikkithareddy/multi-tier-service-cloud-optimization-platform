# Multi-Tier Service Cloud Optimization Platform

A Salesforce Service Cloud configuration, represented as deployable SFDX
metadata: Email-to-Case ingestion, a public Knowledge Base setup for
self-service tier deflection, a tiered Permission Set / Role structure
segregating Tier 1 / Tier 2 agent access, and a record-triggered Flow that
auto-populates first-response metrics.

## What's in this repo

```
force-app/main/default/
├── objects/Case/fields/
│   ├── Support_Tier__c.field-meta.xml       # Picklist: Tier 1 / Tier 2 / Tier 3
│   ├── Deflected_By_KB__c.field-meta.xml     # Checkbox: resolved via self-service KB
│   └── First_Response_At__c.field-meta.xml   # DateTime: auto-populated by the flow
├── flows/
│   └── Case_Status_Notification_Flow.flow-meta.xml   # Record-triggered: sets First_Response_At__c on first status change
├── permissionsets/
│   ├── Tier1_Support_Agent.permissionset-meta.xml     # Standard case access, no cross-account visibility
│   └── Tier2_Support_Agent.permissionset-meta.xml     # Adds viewAllRecords + case transfer, still no modifyAll
├── roles/
│   ├── Tier1_Support.role-meta.xml
│   └── Tier2_Support.role-meta.xml
├── knowledgeArticleTypes/
│   └── FAQ.knowledgeArticleType-meta.xml     # Article type backing the public self-service Knowledge Base
└── settings/
    └── Case.settings-meta.xml                 # Enables Email-to-Case (HTML email, on-demand service, thread ID)
```

## How this was built and verified

This is Salesforce **declarative configuration expressed as metadata**,
which is the standard way Salesforce projects are version-controlled (via
Salesforce DX / `sfdx force:source:deploy`). Being honest about scope: I
don't have a live Salesforce org available in this environment to deploy
and click-test this end-to-end the way a real project would be verified in
a sandbox org. What I *did* verify:

- Every XML file is **well-formed** (validated with `xmllint`) and follows
  the correct Metadata API schema/tag structure for its type.
- The Flow's logic (decision on `Status` changing away from `New`, then
  setting `First_Response_At__c`) follows Salesforce Flow Builder's actual
  execution model for a record-triggered, after-save flow.
- Permission Set field/object permissions reference only fields that exist
  in this same package (the three custom `Case` fields above), so there
  are no dangling references.

**Before relying on this in an interview**, deploy it to a free
[Developer Edition](https://developer.salesforce.com/signup) org with
`sf project deploy start` and click through Setup to confirm the Flow
fires as expected — that hands-on verification is worth doing regardless,
since walking through Flow Builder yourself is exactly what builds the
intuition to defend this design in a technical screen.

## Design notes

- **Tier 2 gets `viewAllRecords: true` but not `modifyAllRecords`** —
  broader visibility for escalation handling, without full admin-level
  write access. This mirrors how real support orgs scope tiered access:
  more visibility as you go up, but modify-all stays reserved for admins.
- **`First_Response_At__c` is set by the Flow, not editable by users**
  (`editable: false` on both permission sets) — keeping this metric
  system-generated protects it from being manually backdated.
- **`Deflected_By_KB__c` is a plain checkbox**, not a formula field, since
  it's meant to be set explicitly (by a Flow or agent action) when a case
  closes via self-service rather than derived — keeping the deflection
  metric intentional rather than inferred.

## License

MIT — see [LICENSE](./LICENSE).
