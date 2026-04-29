# CyberArk (cyberark)

CyberArk is the global leader in identity security, providing a unified Identity Security Platform that protects human, machine, and application identities across hybrid and multi-cloud environments. Product lines include Privileged Access Manager (PAM Self-Hosted) and Privilege Cloud for credential vaulting and session management; Conjur Secrets Manager (Open Source, Enterprise, Cloud) for machine-identity and DevOps secrets; CyberArk Identity for workforce SSO, MFA, and lifecycle; Endpoint Privilege Manager for least-privilege on endpoints; Secure Cloud Access for just-in-time cloud entitlements; and Customer Identity for B2B/B2C.

**URL:** [Visit APIs.json URL](https://raw.githubusercontent.com/api-evangelist/cyberark/refs/heads/main/apis.yml)

## Scope

- **Type:** Index
- **Position:** Consuming
- **Access:** 3rd-Party
- **x-type:** company

## Tags

- Authentication, Cloud Security, Conjur, Credential Vault, DevOps Secrets, Endpoint Privilege Management, Identity Security, Machine Identity, MFA, OpenAPI, PAM, Privileged Access, Privileged Access Management, Secrets Management, Session Management, SSO, Vault, Zero Trust

## Timestamps

- **Created:** 2026-03-25
- **Modified:** 2026-04-28

## APIs

### CyberArk Conjur Secrets Manager API

Secrets management for machine identities and DevOps workloads. Available as Conjur Open Source, Conjur Enterprise (Self-Hosted), and Conjur Cloud (SaaS). The canonical OpenAPI 3.1 spec is open-sourced at `github.com/cyberark/conjur-openapi-spec`.

- **Human URL:** https://docs.cyberark.com/conjur-cloud/latest/en/content/developer/conjur-api-openapi.html
- **OpenAPI:** [openapi/cyberark-conjur-openapi.yml](openapi/cyberark-conjur-openapi.yml)
- **Capabilities:** [capabilities/cyberark-conjur-capabilities.yml](capabilities/cyberark-conjur-capabilities.yml)
- **Rules:** [rules/cyberark-conjur-rules.yml](rules/cyberark-conjur-rules.yml)

### CyberArk PAM Self-Hosted REST API

Vault management for accounts, safes, platforms, users, sessions, and applications. Logon endpoint returns a session token used in the Authorization header.

- **Human URL:** https://docs.cyberark.com/pam-self-hosted/latest/en/content/webservices/implementing%20privileged%20account%20security%20web%20services%20.htm

### CyberArk Privilege Cloud REST API

SaaS counterpart to PAM Self-Hosted, running on the CyberArk Identity Security Platform.

- **Human URL:** https://docs.cyberark.com/privilege-cloud-shared-services/latest/en/content/sdk/sdk-overview.htm

### CyberArk Identity REST API

Programmatic management of users, roles, apps, MFA, SSO, and SCIM-based provisioning across the workforce identity tenant at `{tenant}.id.cyberark.cloud`.

- **Human URL:** https://docs.cyberark.com/identity/Latest/en/Content/Developer/Developer.htm

## Capabilities

- Privileged credential vaulting and rotation
- Just-in-time access provisioning
- Machine identity authentication and policy management
- Versioned secret storage and retrieval (Conjur)
- Workforce SSO, MFA, and lifecycle
- SCIM-based identity provisioning
- Endpoint least-privilege enforcement

## Use Cases

- DevOps pipelines retrieving secrets from Conjur via short-lived tokens
- Kubernetes workloads authenticating via Conjur authn-jwt or authn-k8s
- Database, SSH, and cloud admin accounts vaulted in PAM / Privilege Cloud
- Workforce SSO for SaaS apps via CyberArk Identity
- SCIM provisioning from HRIS into target apps
- Endpoint privilege elevation requests on Windows / macOS / Linux

## Common Resources

- [CyberArk Website](https://www.cyberark.com)
- [Products](https://www.cyberark.com/products/)
- [Documentation](https://docs.cyberark.com)
- [Developer Portal](https://developer.cyberark.com/)
- [GitHub Organization](https://github.com/cyberark)
- [Conjur OpenAPI Spec](https://github.com/cyberark/conjur-openapi-spec)
- [Marketplace](https://marketplace.cyberark.com/)
- [JSON-LD Context](json-ld/cyberark-context.jsonld)
- [Conjur Resource Schema](json-schema/cyberark-conjur-resource-schema.json)
- [Privileged Account Schema](json-schema/cyberark-privileged-account-schema.json)
- [Vocabulary](vocabulary/cyberark-vocabulary.yml)
- [Capabilities](capabilities/cyberark-conjur-capabilities.yml)
- [Rules](rules/cyberark-conjur-rules.yml)

## Maintainers

**FN:** Kin Lane

**Email:** kin@apievangelist.com
