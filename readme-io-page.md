# Postman Collection

Test the Penneo API end-to-end using our official Postman collection before you start integrating.

## Import the collection

In Postman, go to **Import → Link** and paste the URL below:

```
https://raw.githubusercontent.com/Penneo/Penneo-api-postman/main/penneo-api.postman_collection.json
```

## Prerequisites

Before running the collection you need:

- A Penneo account with administrator access
- An OAuth client created in Penneo (**Configure → OAuth Clients**)
- API keys generated for your account

See [Authentication](https://penneo.readme.io/docs/using-oauth) for step-by-step instructions on setting up OAuth clients and API keys.

## Setup

After importing, open the collection in Postman and go to the **Variables** tab. Fill in the four required variables:

| Variable | Description |
|----------|-------------|
| `clientId` | Your OAuth client ID |
| `clientSecret` | Your OAuth client secret |
| `apiKey` | Your Penneo API key |
| `apiSecret` | Your Penneo API secret |

The collection is pre-configured for **sandbox**. When you are ready to test against production, update the following variables:

| Variable | Production value |
|----------|-----------------|
| `baseUrl` | `https://app.penneo.com` |
| `authUrl` | `https://login.penneo.com` |

## Requests

Run the six requests in order — each one automatically saves the values needed by the next.

### 1. Get Access Token

Authenticates using the [API Keys Grant](https://penneo.readme.io/docs/using-oauth#api-keys-grant) flow. The nonce, timestamp, and digest are calculated automatically by the pre-request script — no manual steps required.

The access token is valid for **10 minutes**. If a later request returns `401`, re-run this request to get a fresh token.

### 2. Send Casefile

Uploads a PDF and creates a signing request. The request body uses `multipart/form-data` with two parts:

- `data` — a JSON string describing the casefile (title, signers, documents)
- `files` — the PDF file to be signed

> The `name` field in the documents array must exactly match the filename of the uploaded PDF.

### 3. Poll Job Status

Casefile creation is asynchronous. Re-run this request until the `jobStatus` is `completed`. The casefile ID is saved automatically once the job completes.

> Rate limited to 20 requests per minute.

### 4. Check Casefile Status

Retrieves the full casefile details including signer status. Document IDs are saved automatically for use in request 5.

### 5. Get Document

Downloads the document as a PDF. If the casefile contains multiple documents, update the `documentId` variable manually and re-run for each one.

### 6. Delete Casefile

Permanently deletes the casefile. This action is irreversible.
