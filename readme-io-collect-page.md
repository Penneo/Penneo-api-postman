# Collect Postman Collection

Test Penneo's Collect API using our official Postman collection. Create and manage Collect forms, generate prefilled form links, and retrieve submission data and signed documents.

## Import the collection

In Postman, go to **Import → Link** and paste the URL below:

```
https://raw.githubusercontent.com/Penneo/Penneo-api-postman/main/penneo-collect-api.postman_collection.json
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

All other variables are set automatically as you run the requests in order.

The collection is pre-configured for **sandbox**. When you are ready to test against production, update the following variables:

| Variable | Sandbox | Production |
|----------|---------|------------|
| `baseUrl` | `https://sandbox.penneo.com/collect/api` | `https://app.penneo.com/collect/api` |
| `authUrl` | `https://login-sandbox.penneo.com` | `https://login.penneo.com` |

## Requests

### 1. Get Access Token

Authenticates using the [API Keys Grant](https://penneo.readme.io/docs/using-oauth#api-keys-grant) flow. The nonce, timestamp, and digest are calculated automatically by the pre-request script — no manual steps required.

The access token is valid for **10 minutes** and is saved to `accessToken` automatically. If a later request returns `401`, re-run this request to get a fresh token.

---

### Form Management

Run these in order to take a form through its full lifecycle.

**`Create Simple Form`** `POST /v1/forms`

Creates a new form in DRAFT status. The form is not publicly accessible until it is published. The `formId` is saved automatically and used by all subsequent requests in this folder.

The example body includes a single section with four fields: Full Name (TEXT), Email Address (EMAIL), Phone Number (PHONE), and Additional Comments (TEXT). Modify the body to match your own form structure. Each field must have a `name` property — this is the key used when creating prefilled form requests.

> Saves: `formId`

**`Publish Form`** `POST /v1/forms/{{formId}}/publish`

Publishes the form, changing its status from DRAFT to ACTIVE. End-users can now access and submit it via its public URL.

> Requires: `formId`

**`Get Form`** `GET /v1/forms/{{formId}}`

Retrieves the full form definition including all sections, fields, current status, and public URL (if active). Useful for inspecting the form after creating or updating it.

> Requires: `formId`

**`Update Form`** `PUT /v1/forms/{{formId}}`

Updates the form with a new set of sections and fields. If the form is currently ACTIVE, a new DRAFT version is created under the same `formId` — **run Publish Form again to make the changes live**.

The example body adds a Company Name field to the original form structure.

> Requires: `formId`

---

### Prefilled Form Request

**`Create Prefilled Form Request`** `POST /v2/forms/{{formId}}/requests`

Generates a personalised form link with specific fields pre-populated. The end-user opens the link and sees the form with the provided values already filled in. Fields with `"editable": true` can be changed by the user; fields with `"editable": false` are locked.

The answer keys in the request body must match the `name` properties of the fields defined when the form was created. The example pre-populates `full_name` and `email_address`.

The public URL is saved to `requestUrl` automatically. Share it with the end-user.

> Requires: `formId`  
> Saves: `requestUrl`, `requestId`

---

### Deactivate Form

**`Deactivate Form`** `POST /v1/forms/{{formId}}/archive`

Takes the form offline by changing its status from ACTIVE to ARCHIVED. The public URL becomes inactive and no new submissions are accepted. Existing submissions are preserved and remain retrievable via the Data Retrieval requests.

> Requires: `formId`

---

### Form Examples

Ready-to-run form examples demonstrating specific Collect API features. Each request creates a form in DRAFT status — run **Publish Form** from the Form Management folder to take it live.

**`Conditional Fields and Sections`** `POST /v1/forms`

Creates a form demonstrating two types of conditional visibility:
- A **conditional section** — "Business Details" (company name, registration number) is hidden until the user selects "Business" as their customer type.
- A **conditional field** — "Agreement ID" is hidden until the user answers "Yes" to an existing agreement question, and becomes required once visible.

Conditions use an `EQUALS` rule referencing the `name` of an earlier field.

> Saves: `formId`

**`Set Casefile Name`** `POST /v1/forms`

Creates a form with a `caseFileTitleTemplate` that controls how the resulting casefile is named in Penneo Sign. The template uses merge fields resolved at submission time: `{{formName}}`, `{{primarySigner.name}}`, and `{{primarySigner.email}}`.

> Saves: `formId`

**`Merge Fields in HTML Content`** `POST /v1/forms`

Creates a loan application form demonstrating merge fields in HTML content. Answers collected in earlier sections are injected into a final summary section using `{{fieldName}}` syntax, so the applicant can review their input before signing. The merge field key matches the `name` property of the source field.

> Saves: `formId`

**`Map Fields to Signer`** `POST /v1/forms`

Creates a form demonstrating the `mapTo` property. Fields mapped to `primarySigner.name` and `primarySigner.email` automatically identify the form filler as the primary signer — no separate signer configuration required. The field mapped to any `*.email` property must be of type `EMAIL`.

> Saves: `formId`

---

### Data Retrieval

Run these in order once end-users have submitted and signed the form.

**`List Submissions`** `GET /v1/forms/{{formId}}/submissions`

Returns all completed submissions for the given form, ordered by submission ID ascending. The `submissionId` and `casefileId` from the first result are saved automatically.

If there are multiple submissions, update `submissionId` manually in the Variables tab to work with a specific one.

Supports pagination via `limit` (default 10, max 1000) and `offset` query parameters.

> Requires: `formId`  
> Saves: `submissionId`, `casefileId`

**`Get Submitted User Data`** `GET /v1/forms/{{formId}}/submissions/{{submissionId}}/answers`

Retrieves all field answers submitted by the end-user for a specific submission.

> Requires: `formId`, `submissionId`

**`Get Request Answers`** `GET /v2/forms/{{formId}}/requests/{{requestId}}/answers`

Retrieves all field answers submitted against a specific prefilled form request — keyed on `requestId` instead of `submissionId`. Use it when you started from a prefilled form request and want the answers without looking up the submission first.

The response has the same sectioned shape as `Get Submitted User Data`.

The form must be in `ACTIVE` or `ARCHIVED` status, and a submission must have been made against the request.

Note this endpoint is on **v2**, consistent with the other request-scoped endpoints.

> Requires: `formId`, `requestId`

**`Get Signed Document by submissionId`** `GET /v1/forms/{{formId}}/submissions/{{submissionId}}/signed-document`

Downloads the signed PDF for a submission. The response is streamed as `application/pdf` with a `Content-Disposition` attachment filename.

In Postman, use **Send and Download** (the arrow next to the Send button) to save the file locally.

> Requires: `formId`, `submissionId`

**`Get Signed Document by requestId`** `GET /v2/forms/{{formId}}/requests/{{requestId}}/signed-document`

Downloads the signed PDF for a prefilled form request. Identical in behaviour to the endpoint above, but keyed on `requestId` instead of `submissionId` — use it when you started from a prefilled form request and want the document without looking up the submission first.

Note this endpoint is on **v2**, consistent with Create Prefilled Form Request.

> Requires: `formId`, `requestId`

**Availability:** the signed document exists only once signing has finished. Until then both endpoints return `404 Not Found`.

---

## Variable Reference

| Variable | Set by | Description |
|---|---|---|
| `clientId` | You | OAuth client ID |
| `clientSecret` | You | OAuth client secret |
| `apiKey` | You | Penneo API key |
| `apiSecret` | You | Penneo API secret |
| `accessToken` | Get Access Token | Bearer token for all authenticated requests |
| `formId` | Create Form | Identifies the form across all requests |
| `requestUrl` | Create Prefilled Form Request | Public URL to share with the end-user |
| `requestId` | Create Prefilled Form Request | Identifies a specific prefilled form request |
| `submissionId` | List Submissions | Identifies a specific submission |
| `casefileId` | List Submissions | Links the submission to the underlying Penneo Sign casefile. Not used by any request in this collection |
