# CyberArk (cyberark)

CyberArk is the global leader in identity security, providing a unified Identity Security Platform that protects human, machine, and application identities across hybrid and multi-cloud environments. Core product lines include Privileged Access Manager (PAM Self-Hosted) and Privilege Cloud for credential vaulting and session management; Conjur Secrets Manager (Open Source, Enterprise, and Cloud) for machine-identity and DevOps secrets; CyberArk Identity for workforce SSO, MFA, and lifecycle; Endpoint Privilege Manager for least-privilege enforcement on Windows / macOS / Linux endpoints; Secure Cloud Access for just-in-time cloud entitlements; and Customer Identity for B2B / B2C identity. CyberArk publishes a canonical OpenAPI 3.1 specification for Conjur Secrets Manager at github.com/cyberark/conjur-openapi-spec, and REST APIs for PAM Self-Hosted, Privilege Cloud, and CyberArk Identity are documented on docs.cyberark.com and developer.cyberark.com.

**APIs.json:** [https://raw.githubusercontent.com/api-evangelist/cyberark/refs/heads/main/apis.yml](https://raw.githubusercontent.com/api-evangelist/cyberark/refs/heads/main/apis.yml)

## Scope

- **Type:** Index
- **Position:** Consuming
- **Access:** 3rd-Party

## Tags

- Authentication
- Cloud Security
- Conjur
- Credential Vault
- DevOps Secrets
- Endpoint Privilege Management
- Identity Security
- Machine Identity
- MFA
- OpenAPI
- PAM
- Privileged Access
- Privileged Access Management
- Secrets Management
- Session Management
- SSO
- Vault
- Zero Trust

## Timestamps

- **Created:** 2026-03-25
- **Modified:** 2026-05-19

## APIs

### CyberArk Conjur Secrets Manager API

Conjur is CyberArk's secrets management platform for machine identities and DevOps workloads, delivered as Conjur Open Source, Conjur Enterprise (Self-Hosted), and Conjur Cloud (SaaS). The REST API supports authenticating hosts and users, loading and replacing policy YAML, storing and retrieving versioned secrets, managing resources and roles, and retrieving public keys. The canonical OpenAPI 3.1 spec is open-sourced at github.com/cyberark/conjur-openapi-spec.

- **Human URL:** [https://docs.cyberark.com/conjur-cloud/latest/en/content/developer/conjur-api-openapi.html](https://docs.cyberark.com/conjur-cloud/latest/en/content/developer/conjur-api-openapi.html)
- **Base URL:** `https://conjur.example.com`

#### Tags

- Authentication
- Conjur
- DevOps Secrets
- Machine Identity
- Policies
- Resources
- Roles
- Secrets
- Vault

#### Properties

- [Documentation](https://docs.cyberark.com/conjur-cloud/latest/en/content/developer/conjur-api-openapi.html)
- [OpenAPI](openapi/cyberark-conjur-openapi.yml) — [OpenAPI Specification](https://spec.openapis.org/oas/latest.html)
- [Postman Collection](collections/cyberark-conjur.postman_collection.json) — [Postman Collection 2.1](https://schema.getpostman.com/json/collection/v2.1.0/collection.json)
- [Open Collection](collections/cyberark-conjur.opencollection.json) — [Open Collection 1.0](https://schema.opencollection.com/opencollection/v1.0.0.json)
- [Canonical Open A P I](https://github.com/cyberark/conjur-openapi-spec)
- [Capabilities](capabilities/cyberark-conjur-capabilities.yml)
- [Rules](rules/cyberark-conjur-rules.yml)
- [GitHub Repository](https://github.com/cyberark/conjur)
- [Blog](https://developer.cyberark.com/blog/introducing-the-conjur-openapi-description/)

### CyberArk PAM Self-Hosted REST API

The Privileged Access Manager Self-Hosted REST API exposes the Vault for managing accounts, safes, platforms, users, sessions, and applications. Authentication uses the Logon endpoint at /PasswordVault/API/Auth/{provider}/Logon to obtain a session token used in the Authorization header for subsequent calls.

- **Human URL:** [https://docs.cyberark.com/pam-self-hosted/latest/en/content/webservices/implementing%20privileged%20account%20security%20web%20services%20.htm](https://docs.cyberark.com/pam-self-hosted/latest/en/content/webservices/implementing%20privileged%20account%20security%20web%20services%20.htm)

#### Tags

- Accounts
- PAM
- Privileged Access
- REST
- Safes
- Sessions
- Vault

#### Properties

- [Documentation](https://docs.cyberark.com/pam-self-hosted/latest/en/content/webservices/implementing%20privileged%20account%20security%20web%20services%20.htm)
- [Sample Scripts](https://github.com/cyberark/epv-api-scripts)
- [Power Shell Module](https://github.com/pspete/psPAS)
- [Postman Collection](collections/cyberark-conjur.postman_collection.json) — [Postman Collection 2.1](https://schema.getpostman.com/json/collection/v2.1.0/collection.json)
- [Open Collection](collections/cyberark-conjur.opencollection.json) — [Open Collection 1.0](https://schema.opencollection.com/opencollection/v1.0.0.json)

### CyberArk Privilege Cloud REST API

The Privilege Cloud Shared Services REST API mirrors the PAM Self-Hosted surface for accounts, safes, platforms, and users while running as a SaaS on the CyberArk Identity Security Platform. Identities and groups reference the tenant's CyberArk Identity directory.

- **Human URL:** [https://docs.cyberark.com/privilege-cloud-shared-services/latest/en/content/sdk/sdk-overview.htm](https://docs.cyberark.com/privilege-cloud-shared-services/latest/en/content/sdk/sdk-overview.htm)

#### Tags

- Accounts
- PAM
- Privilege Cloud
- Safes
- SaaS
- Vault

#### Properties

- [Documentation](https://docs.cyberark.com/privilege-cloud-shared-services/latest/en/content/sdk/sdk-overview.htm)
- [Whats New](https://docs.cyberark.com/privilege-cloud-shared-services/latest/en/content/privilege%20cloud/privcloud-whatsnew-v12.2.htm)
- [Postman Collection](collections/cyberark-conjur.postman_collection.json) — [Postman Collection 2.1](https://schema.getpostman.com/json/collection/v2.1.0/collection.json)
- [Open Collection](collections/cyberark-conjur.opencollection.json) — [Open Collection 1.0](https://schema.opencollection.com/opencollection/v1.0.0.json)

### CyberArk Identity REST API

The CyberArk Identity REST API enables programmatic management of users, roles, applications, MFA policies, SSO, and SCIM-based provisioning across the workforce identity tenant. Tenants are addressed at {tenant}.id.cyberark.cloud and authentication uses OAuth2.

- **Human URL:** [https://docs.cyberark.com/identity/Latest/en/Content/Developer/Developer.htm](https://docs.cyberark.com/identity/Latest/en/Content/Developer/Developer.htm)

#### Tags

- Identity
- MFA
- OAuth2
- SCIM
- SSO
- Workforce Identity

#### Properties

- [Documentation](https://docs.cyberark.com/identity/Latest/en/Content/Developer/Developer.htm)
- [S C I M](https://docs.cyberark.com/identity/latest/en/content/developer/scim-management/scim-overview.htm)
- [Developer Portal](https://developer.cyberark.com/)
- [Postman Collection](collections/cyberark-conjur.postman_collection.json) — [Postman Collection 2.1](https://schema.getpostman.com/json/collection/v2.1.0/collection.json)
- [Open Collection](collections/cyberark-conjur.opencollection.json) — [Open Collection 1.0](https://schema.opencollection.com/opencollection/v1.0.0.json)

## Common Properties

- [LinkedIn](https://www.linkedin.com/company/cyber-ark-software)
- [Website](https://www.cyberark.com)
- [Products](https://www.cyberark.com/products/)
- [Documentation](https://docs.cyberark.com)
- [Developer Portal](https://developer.cyberark.com/)
- [GitHub Organization](https://github.com/cyberark)
- [Conjur Open A P I Spec](https://github.com/cyberark/conjur-openapi-spec)
- [Marketplace](https://marketplace.cyberark.com/)
- [Trust](https://www.cyberark.com/trust/)
- [Terms of Service](https://www.cyberark.com/legal-terms-of-use/)
- [Privacy Policy](https://www.cyberark.com/privacy-policy/)
- [JSON-LD](json-ld/cyberark-context.jsonld) — [JSON-LD](https://www.w3.org/TR/json-ld11/)
- [JSON Schema](json-schema/cyberark-conjur-resource-schema.json) — [JSON Schema](https://json-schema.org/specification)
- [JSON Schema](json-schema/cyberark-privileged-account-schema.json) — [JSON Schema](https://json-schema.org/specification)
- [Vocabulary](vocabulary/cyberark-vocabulary.yml)
- [Capabilities](capabilities/cyberark-conjur-capabilities.yml)
- [Rules](rules/cyberark-conjur-rules.yml)
- [L L Ms Txt](https://developer.cyberark.com/llms.txt)

## Maintainers

**FN:** Kin Lane
**Email:** kin@apievangelist.com
