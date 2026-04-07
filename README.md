# Penneo API — Postman Collection

A Postman collection for validating the Penneo API before integrating. Run through the full signing flow — authenticate, send a document for signing, poll for completion, check status, and retrieve the signed document.

## Import

In Postman, go to **Import → Link** and paste:

```
https://raw.githubusercontent.com/Penneo/Penneo-api-postman/main/penneo-api.postman_collection.json
```

## Prerequisites

- A Penneo account with an OAuth client and API keys configured
- [Postman](https://www.postman.com/downloads/)

To set up an OAuth client and generate API keys, see the [Penneo authentication docs](https://penneo.readme.io/docs/using-oauth).

## Setup

After importing, open the collection and go to the **Variables** tab. Fill in the following:

| Variable | Description |
|----------|-------------|
| `clientId` | Your OAuth client ID |
| `clientSecret` | Your OAuth client secret |
| `apiKey` | Your Penneo API key |
| `apiSecret` | Your Penneo API secret |

The `baseUrl` and `authUrl` are pre-configured for sandbox. Change them to the production URLs when you are ready to go live:

| Variable | Sandbox | Production |
|----------|---------|------------|
| `baseUrl` | `https://sandbox.penneo.com` | `https://app.penneo.com` |
| `authUrl` | `https://sandbox.oauth.penneo.cloud` | `https://login.penneo.com` |

## Requests

Run the requests in order — each one automatically saves the values needed by the next.

| # | Request | Description |
|---|---------|-------------|
| 1 | Get Access Token | Authenticates using the API Keys Grant flow |
| 2 | Send Casefile | Uploads a PDF and creates a signing request |
| 3 | Poll Job Status | Polls until the casefile is created (re-run until completed) |
| 4 | Check Casefile Status | Retrieves casefile details and signer status |
| 5 | Get Document | Downloads the document as a PDF |

> **Note:** The access token expires after 10 minutes. If a request fails with `401`, re-run request 1 to get a fresh token.
