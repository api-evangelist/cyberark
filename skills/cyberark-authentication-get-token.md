---
name: cyberark-authentication-get-token
description: Obtain a short‑lived access token for a user by first retrieving the API key and then authenticating with it.
api: openapi/cyberark-authentication-api-openapi.yml
operations:
- login
- authenticate
generated: '2026-10-03'
method: generated
generator: extract-docs-artifacts.py skills (local)
source: openapi/cyberark-authentication-api-openapi.yml ; every operationId checked against the contract
---

# cyberark-authentication-get-token

Obtain a short‑lived access token for a user by first retrieving the API key and then authenticating with it.

## Steps

1. 1. Call `login` with the path parameters `account` and `login` to retrieve the user's API key.
2. 2. Call `authenticate` with the path parameters `account` and `login` and the API key from step 1 in the request body to receive a short‑lived access token.

## Rules

- Authentication: Uses the ConjurAuth scheme (HTTP) but the login endpoint does not require a prior token.
- Idempotency: `login` is a GET request and is idempotent; `authenticate` is a POST request and may create a new token each call.
- Rate limiting: No rate limit is defined; on exhaustion the server returns no specific HTTP status.
