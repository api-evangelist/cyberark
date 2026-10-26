# CyberArk (cyberark)

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
> **Not from the company, and here with a question?** You are welcome here — we would rather be the
> front line and point you the right way than have a good report go nowhere. What this repository
> can answer is narrow, though, so it is worth knowing who you are actually looking for:
>
> - **A question about how the API works, an account, billing, or a bug in the service** — that is
>   the company's own support, not us. We profile this API; we do not operate it and cannot see
>   your account.
> - **A bug in an open-source project we only catalog** — file it on that project's own repository.
>   This has happened with a real and correct bug report that reached us instead of the people who
>   could fix it, which helped nobody.
> - **Anything about this listing itself** — the description, the tags, the rating, a missing or
>   wrong artifact — is ours. Open an issue here.
> - **Not sure, or something general about API Evangelist or APIs.io** — open an issue on the
>   [APIs.io Inbox](https://github.com/api-search/inbox) and we will route it.
>
> **This repository contains no software, and we will never ask you to download anything.** There is
> no build, release, installer, or binary here — only text and machine-readable API descriptions, so
> there is nothing here that can be "corrupt" or need "repairing". Any issue, comment, or email
> claiming otherwise and offering a download link is not from us and is hostile. Do not follow the
> link; it is a lure. Report it to GitHub and, if you like, tell us at
> [info@apievangelist.com](mailto:info@apievangelist.com) so we can take it down.
>
> **On a security or compliance team?** Email
> [info@apievangelist.com](mailto:info@apievangelist.com) with *security* in the subject line and
> you will get a person, not a form. We will tell you exactly which public URLs this profile was
> built from so your team can see the same surface we did, and we will take the listing down on
> request while you work through it.
>
> Full detail: **[Where this data comes from](https://apievangelist.com/about/where-our-data-comes-from)**
<!-- API-EVANGELIST-PROVENANCE:END -->

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
