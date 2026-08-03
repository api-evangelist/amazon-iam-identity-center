# Amazon IAM Identity Center (amazon-iam-identity-center)

<!-- API-EVANGELIST-PROVENANCE:BEGIN -->
> ### About this repository
>
> **This is not our API.** This repository is an independent, third-party profile of a company's
> **publicly available** API surface, maintained by [API Evangelist](https://apievangelist.com).
> API Evangelist does not operate, host, resell, or support this company's APIs, and is not
> affiliated with or endorsed by the company unless stated on the profile.
>
> **Where the information came from.** Everything here is assembled from material a member of the
> public can reach with a browser and no credentials — the company's own website, developer portal
> and documentation, the specifications it publishes for public use (OpenAPI, AsyncAPI, JSON Schema,
> `apis.json`, `llms.txt` and similar), its public repositories, and its public status, pricing and
> changelog pages. **Nothing here is obtained by breaching a system, defeating an access control, or
> using credentials of any kind.**
>
> **The rating is an independent assessment.** The Kin Score and Agent Readiness rating are
> independently calculated scores of a company's *public* API artifacts, produced by API Evangelist
> against a published rubric. They are not certifications, endorsements, security assessments, or
> audits, and they score published artifacts — not the quality, safety, or security of the software.
>
> **Corrections, re-scores, and removal are free.** No partnership, contract, or purchase is
> required, and you do not need to justify the request.
>
> - **Something wrong?** Open an issue on this repository, or email
>   [info@apievangelist.com](mailto:info@apievangelist.com).
> - **Published something new?** Ask for a re-score and we will re-run the rating.
> - **Want the listing taken down?** Say so and we will honor it. The profile is reduced to your
>   company name, a factual description, and a link to your own site, and the company is recorded as
>   **unrated** — never scored zero for having asked.
>
> **Response times.** Acknowledgement within **one business day**; removal or restriction within
> **two business days**; corrections and re-scores within **five business days**.
>
> **On a security or compliance team?** Email
> [info@apievangelist.com](mailto:info@apievangelist.com) with *security* in the subject line and
> you will get a person, not a form. We will tell you exactly which public URLs this profile was
> built from so your team can see the same surface we did, and we will take the listing down on
> request while you work through it.
>
> Full detail: **[Where this data comes from](https://apievangelist.com/about/where-our-data-comes-from)**
<!-- API-EVANGELIST-PROVENANCE:END -->

AWS IAM Identity Center (successor to AWS Single Sign-On) is where you create, or connect, your workforce identities in AWS once and manage access centrally across your AWS organization. You can create user identities directly in IAM Identity Center, or bring them from Microsoft Active Directory, and then use IAM Identity Center to manage user access to AWS accounts and business applications with single sign-on.

**URL:** [Visit APIs.json URL](https://raw.githubusercontent.com/api-evangelist/amazon-iam-identity-center/refs/heads/main/apis.yml)

**Run:** [Capabilities Using Naftiko](https://github.com/naftiko/fleet?utm_source=api-evangelist&utm_medium=readme&utm_campaign=company-api-evangelist&utm_content=repo)

## Tags:

 - Access Control, Authentication, AWS, Identity Management, Single Sign-On

## Timestamps

- **Created:** 2026-03-16
- **Modified:** 2026-04-19

## APIs

### AWS IAM Identity Center SSO Admin API
Manages permission sets, account assignments, instances, and SSO configurations for centralized identity and access management.

**Human URL:** [https://aws.amazon.com/iam/identity-center/](https://aws.amazon.com/iam/identity-center/)

#### Tags:

 - Access Control, Identity Management, Single Sign-On

#### Properties

- [Documentation](https://docs.aws.amazon.com/singlesignon/latest/APIReference/Welcome.html)
- [OpenAPI](openapi/amazon-iam-identity-center-sso-admin-openapi-original.yml)
- [GettingStarted](https://aws.amazon.com/iam/identity-center/getting-started/)
- [Pricing](https://aws.amazon.com/iam/identity-center/pricing/)
- [FAQ](https://aws.amazon.com/iam/identity-center/faqs/)

### AWS IAM Identity Center Identity Store API
Manages users, groups, and memberships in the IAM Identity Center identity store.

**Human URL:** [https://docs.aws.amazon.com/singlesignon/latest/IdentityStoreAPIReference/welcome.html](https://docs.aws.amazon.com/singlesignon/latest/IdentityStoreAPIReference/welcome.html)

#### Tags:

 - Access Control, Groups, Identity Management, Users

#### Properties

- [Documentation](https://docs.aws.amazon.com/singlesignon/latest/IdentityStoreAPIReference/welcome.html)
- [OpenAPI](openapi/amazon-iam-identity-center-identitystore-openapi-original.yml)

## Common Properties

- [Portal](https://aws.amazon.com/iam/identity-center/)
- [Website](https://aws.amazon.com/iam/identity-center/)
- [Documentation](https://docs.aws.amazon.com/singlesignon/)
- [TermsOfService](https://aws.amazon.com/service-terms/)
- [PrivacyPolicy](https://aws.amazon.com/privacy/)
- [Support](https://aws.amazon.com/premiumsupport/)
- [Blog](https://aws.amazon.com/blogs/security/tag/aws-iam-identity-center/)
- [GitHubOrganization](https://github.com/aws)
- [Console](https://console.aws.amazon.com/singlesignon/)
- [SignUp](https://portal.aws.amazon.com/billing/signup)
- [Login](https://signin.aws.amazon.com/)
- [StatusPage](https://health.aws.amazon.com/health/status)
- [Contact](https://aws.amazon.com/contact-us/)

## Artifacts

Machine-readable API specifications organized by format.

### OpenAPI

- [AWS IAM Identity Center SSO Admin API](openapi/amazon-iam-identity-center-sso-admin-openapi-original.yml)
- [AWS IAM Identity Center Identity Store API](openapi/amazon-iam-identity-center-identitystore-openapi-original.yml)

### JSON Schema

171 schema files covering permission sets, account assignments, users, groups, and instances.

### JSON Structure

171 JSON Structure files converted from JSON Schema.

### JSON-LD

- [Amazon IAM Identity Center Context](json-ld/amazon-iam-identity-center-context.jsonld)

### Examples

171 example JSON files generated from schemas.

## Capabilities

Naftiko capabilities organized as shared per-API definitions composed into customer-facing workflows.

### Shared Per-API Definitions

- [SSO Admin API](capabilities/shared/sso-admin.yaml) — operations for permission sets and account assignments
- [Identity Store API](capabilities/shared/identitystore.yaml) — operations for users and groups

### Workflow Capabilities

| Workflow | APIs Combined | Tools | Persona |
|----------|--------------|-------|---------|
| [Identity and Access Management](capabilities/identity-access-management.yaml) | SSO Admin, Identity Store | 9 | IT Administrator, IAM Administrator |

## Vocabulary

- [Amazon IAM Identity Center Vocabulary](vocabulary/amazon-iam-identity-center-vocabulary.yaml) — Unified taxonomy mapping 7 resources, 7 actions, 1 workflow, and 2 personas

## Rules

- [Amazon IAM Identity Center Spectral Rules](rules/amazon-iam-identity-center-spectral-rules.yml) — 18 rules across 7 categories enforcing Amazon IAM Identity Center API conventions

## Maintainers

**FN:** Kin Lane

**Email:** kin@apievangelist.com
