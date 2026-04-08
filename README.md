# Penneo — Postman Collections

This repository contains two Postman collections for working with the Penneo API.

---

## Collections

### 1. Penneo API
For validating the core Penneo signing flow before integrating. Covers authentication, sending a document for signing, polling for completion, checking casefile status, and retrieving the signed document.

**File:** `penneo-api.postman_collection.json`

### 2. Penneo Webhooks
For testing Penneo's webhook subscription API. Use this to create, inspect, update, and delete webhook subscriptions — and to fire test events to your endpoint so you can see exactly what payload you will receive before going live.

**File:** `penneo-webhooks.postman_collection.json`

---

## Import

In Postman, go to **Import → Link** and paste the raw URL for the collection you want:

**Penneo API**
```
https://raw.githubusercontent.com/Penneo/Penneo-api-postman/main/penneo-api.postman_collection.json
```

**Penneo Webhooks**
```
https://raw.githubusercontent.com/Penneo/Penneo-api-postman/main/penneo-webhooks.postman_collection.json
```

---

## Prerequisites

- A Penneo account with an OAuth client and API keys configured
- [Postman](https://www.postman.com/downloads/)

To set up an OAuth client and generate API keys, see the [Penneo authentication docs](https://penneo.readme.io/docs/using-oauth).

---

## Setup

Both collections use the same authentication variables. After importing, open the collection and go to the **Variables** tab. Fill in the following:

| Variable | Description |
|----------|-------------|
| `clientId` | Your OAuth client ID |
| `clientSecret` | Your OAuth client secret |
| `apiKey` | Your Penneo API key |
| `apiSecret` | Your Penneo API secret |

The **Penneo Webhooks** collection requires one additional variable:

| Variable | Description |
|----------|-------------|
| `webhookEndpoint` | The URL Penneo should POST events to. Use [webhook.site](https://webhook.site) for a free disposable endpoint during testing. |

The `baseUrl` and `authUrl` are pre-configured for sandbox. Change them to the production URLs when you are ready to go live:

| Variable | Sandbox | Production |
|----------|---------|------------|
| `baseUrl` | `https://sandbox.penneo.com` | `https://app.penneo.com` |
| `authUrl` | `https://sandbox.oauth.penneo.cloud` | `https://login.penneo.com` |

---

## Requests

### Penneo API

Run the requests in order — each one automatically saves the values needed by the next.

| # | Request | Description |
|---|---------|-------------|
| 1 | Get Access Token | Authenticates using the API Keys Grant flow |
| 2 | Send Casefile | Uploads a PDF and creates a signing request |
| 3 | Poll Job Status | Polls until the casefile is created (re-run until completed) |
| 4 | Check Casefile Status | Retrieves casefile details and signer status |
| 5 | Get Document | Downloads the document as a PDF |

### Penneo Webhooks

| # | Request | Description |
|---|---------|-------------|
| 1 | Get Access Token | Authenticates using the API Keys Grant flow |
| 2 | Create Subscription | Creates a webhook subscription and saves the ID and secret |
| 3 | Get Subscription | Fetches details of the saved subscription |
| 4 | List Subscriptions | Returns all subscriptions on the account |
| 5 | Test Subscription | Fires a test event so you can inspect the payload your endpoint receives |
| 6 | Update Subscription | Updates event types or toggles the subscription on/off |
| 7 | Delete Subscription | Permanently removes the subscription |

> **Note:** The access token expires after 10 minutes. If a request fails with `401`, re-run request 1 to get a fresh token.
