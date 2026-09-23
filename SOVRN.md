# Sovrn fork notes

Upstream: https://github.com/aws-samples/aws-organizations-tag-inventory

This fork exists so Sovrn changes to the tag-inventory solution have a home.
Before it, the solution was deployed from a personal clone of the public AWS
sample, so no change could be reviewed, reproduced, or survive its author.

## Deployment

Hub (central) account: TLZ management `541104075663`, region `us-east-2`.
Glue database `o-tkj06h2nvy-tag-inventory-database`, S3
`tag-inventory-o-tkj06h2nvy-541104075663-us-east-2`, Athena workgroup
`TagInventoryAthenaWorkGroup-us-east-2`. Surfaced in QuickSight from the
`finance-reports` account.

## Sovrn changes on this fork

- Every stack applies the two mandatory tags `product=ace` and an
  `application` value, so resources are tagged at creation rather than by
  the AFT `SetProductTags` backfill.

## Known defects in the deployed solution

These are real and unfixed. Anything quoted from this pipeline is not a total.

1. **The legacy org contributes no data.** Only the TLZ org reports. 17 active
   accounts are invisible, including SOVRN-AWS-CORE, SOVRN-AWS-OPS and
   Sovrn-PRD.
2. **Every account is capped at 1,000 resources.** Resource Explorer's Search
   API returns at most 1,000 results per call and the spoke does not paginate
   past it; no account exceeds 1,000 distinct ARNs on any partition.
3. **IAM roles report as untagged when they are not.** Resource Explorer does
   not return tags for `iam:role`; 3,414 rows show `NoTag` including roles
   that demonstrably carry tags. Several other types are suspect.
4. **The raw table contains duplicate rows**, by a factor that varies from 2x
   to 12x per partition, so any `COUNT(*)` over it is wrong.

The pre-existing Glue views are also defective — their exclusion lists are
inverted, tagged and untagged counts come from different populations, and
`LOWER()` on the tag key masks the exact casing violation the SCP enforces.
A rebuilt QuickSight dashboard, `sovrn-tag-compliance`, works around them.
