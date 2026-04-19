# Amazon IAM Identity Center (amazon-iam-identity-center)
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
