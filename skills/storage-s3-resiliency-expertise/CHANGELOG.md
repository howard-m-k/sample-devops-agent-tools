# Changelog

All notable changes to this skill are documented here. New entries go at the top.

## [1.1.0] - 2026-08-11

### Changed (least-privilege / security scoping)
- **Region discovery now uses `GetBucketLocation` instead of `HeadBucket`.**
  `HeadBucket`'s IAM permission is `s3:ListBucket`, which also authorizes listing a
  bucket's object contents — out of scope for a read-only review. `GetBucketLocation`
  needs only `s3:GetBucketLocation` (bucket-level, no object-listing). The replication
  destination-Region lookup uses `GetBucketLocation` too.
- **Removed account-level Block Public Access assessment.** The
  `s3control:GetPublicAccessBlock` / `s3:GetAccountPublicAccessBlock` call is gone;
  account-wide configuration is out of scope. Block Public Access is now evaluated at
  the **bucket level only**, and the "not configured at the bucket level" finding notes
  that account-level BPA should be verified separately (escalating to critical only when
  the bucket-level ACL or policy shows real public exposure).

### Removed
- `s3:ListBucket`, `s3:ListAllMyBuckets`, and `s3:GetAccountPublicAccessBlock` from the
  skill's required permissions. The only permission beyond `AIDevOpsAgentAccessPolicy`
  is now `s3:GetBucketWebsite`, granted via the CloudFormation template's
  `EnableStorageS3Resiliency` parameter.

### Fixed
- Replication "lookup failed" handling: because `GetBucketLocation` (unlike
  `HeadBucket`) does not return a Region when access is denied, the two prior
  cross/same-region "lookup failed" scenarios are collapsed into a single honest
  "destination Region unknown" finding — cross-region redundancy is reported as
  unconfirmable rather than guessed.

## [1.0.1] - 2026-08-10

### Added
- README: "Agent Types" and "Uploading to AWS DevOps Agent" sections (GitHub import,
  zip upload, and Asset API deployment paths), aligned with sibling skills.

### Changed
- `metadata.agent-types` now includes **Incident RCA** alongside Chat tasks and
  Evaluation, matching the README.

### Fixed
- Data collection: `GetBucketVersioning` and `GetBucketLogging` return a successful
  but empty response (no `Status` / no `LoggingEnabled`) when the feature was never
  configured, rather than raising a `NoSuch*` error. The error classification now
  maps an empty success to `NotConfigured` for these two calls instead of `OK`, so a
  never-versioned or never-logged bucket is classified and reported correctly.

## [1.0.0] - 2026-07-29

### Added
- Initial release for AWS DevOps Agent, adapted from the AWS Support Specialist
  `storage-s3-resiliency-expertise` skill.
- Read-only resiliency, security, and data protection review of Amazon S3 buckets
  across nine dimensions: versioning, replication, object lock, bucket policy,
  block public access, default encryption, ownership controls, server access
  logging, and static website hosting.
- Automatic single-bucket vs multi-bucket (fleet) routing by input count, including
  batched review with manifest tracking and resume for 21+ buckets.
- Resiliency Rating (High / Medium / Low / Indeterminate) with per-dimension
  findings and remediation guidance.
- Self-contained data collection via read-only control-plane API calls
  (`use_aws`); no AWS profile or credentials requested from the user.
- Pre-flight permissions and tooling-availability handling that reports unverifiable
  checks instead of inferring configuration state.
- Final Delivery Contract: the report is emitted as a persisted artifact (when the
  runtime supports it) and returned verbatim in the final response, preventing the
  host agent from summarizing or reformatting the output. Runtime-neutral so it also
  applies when the skill is ported to other agents. The contract also mandates the
  full standard report regardless of how the request is phrased ("is it safe",
  "audit", "DR posture", etc.) — no condensed or "focused view" variants.
- Object Lock finding body states that Object Lock can be enabled on an existing
  versioned bucket (corrects the outdated "creation-time only" assumption).

### Changed (from the source skill)
- Replaced the prior data-acquisition layer and separate configuration-collector
  dependency with a self-contained `use_aws` control-plane collection reference.
- Removed the AWS profile prompt and per-run profile caching (DevOps Agent operates
  under an assumed role in the target account).
- Corrected the Object Lock guidance to reflect that Object Lock can be enabled on
  existing versioned buckets.
- Removed all internal Amazon references.
