# NovaDataActions

Genesys Cloud **Web Services Data Actions** for Nova / `gcapi.alektumgroup.com` integrations.

## Actions

| File | Action | Endpoint |
| --- | --- | --- |
| [`dataactions/nova-get-pop-port-by-email.json`](dataactions/nova-get-pop-port-by-email.json) | Nova - Get POP Port by Agent Email | `GET /Service/novaPopPort/{userEmail}` |
| [`dataactions/nova-add-external-contact.json`](dataactions/nova-add-external-contact.json) | Nova - Add External Contact | `POST /Service/AddExternalContact` |
| [`dataactions/nova-update-nova-email.json`](dataactions/nova-update-nova-email.json) | Nova - Update Nova Email | `POST /Service/UpdateNovaEmail` |
| [`dataactions/nova-update-nova-pop.json`](dataactions/nova-update-nova-pop.json) | Nova - Update Nova POP | `POST /NovaPop/UpdateNovaPop` |

---

## Nova - Get POP Port by Agent Email

Looks up the Nova POP port assigned to an agent, using the agent's email address.

### Request

```
GET https://gcapi.alektumgroup.com/Service/novaPopPort/${input.userEmail}
Accept: application/json
```

Genesys Cloud URL-encodes values substituted into `requestUrlTemplate`, so the `@` and any
other special characters in the email address are escaped automatically. Do not pre-encode
the input.

### Input contract

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `userEmail` | string | yes | The agent's email address, e.g. `agent@alektumgroup.com` |

```json
{
  "userEmail": "agent@alektumgroup.com"
}
```

### Output contract

| Field | Type | Description |
| --- | --- | --- |
| `success` | boolean | True when the lookup succeeded |
| `port` | integer | The Nova POP port number. `0` when the API returned no result |
| `message` | string | Optional message from the API |
| `errors` | array of string | Any errors returned by the API |

```json
{
  "success": true,
  "port": 5060,
  "message": "",
  "errors": []
}
```

### Response handling

The upstream API wraps its payload in a standard envelope:

```json
{
  "success": true,
  "data": 0,
  "message": "string",
  "errors": ["string"]
}
```

The translation map flattens `$.data` into `port`. `translationMapDefaults` cover the
**HTTP 204** case (request succeeded but returned no body): the action still returns
successfully with `success: false`, `port: 0`, an empty `message` and an empty `errors`
array. Check `success` (or `port != 0`) in your flow before using the value.

**HTTP 401** means the integration credentials are missing or invalid — the data action
will fail and the failure branch in the flow will be taken.

### Prerequisites

1. A **Web Services Data Actions** integration in Genesys Cloud (Admin > Integrations),
   active and configured with credentials for `gcapi.alektumgroup.com`.
2. Authentication: this definition sends no `Authorization` header. Add the scheme your
   integration uses to `config.request.headers`, for example:

   ```json
   "headers": {
     "Accept": "application/json",
     "Authorization": "Bearer ${credentials.token}"
   }
   ```

   If the integration is configured with *User Defined (OAuth)* or *Basic* credentials,
   Genesys injects the header for you and no change is needed.

### Import

1. Admin > Integrations > **Actions**.
2. **Add Action** > select your Web Services Data Actions integration > **Import** and
   choose `dataactions/nova-get-pop-port-by-email.json`.
3. Open the **Test** tab, run it with a real agent email, then **Save & Publish**.
4. Reference the action from an Architect flow with a **Call Data Action** block, mapping
   the agent's email (e.g. `Flow.AgentEmail`) to `userEmail`.

---

## Nova - Add External Contact

Links a Genesys Cloud **external contact** to a Nova debtor, so later interactions with
that contact can be resolved back to the correct debtor in Nova.

### Request

```
POST https://gcapi.alektumgroup.com/Service/AddExternalContact
Accept: application/json
Content-Type: application/json
Authorization: ${authResponse.token_type} ${authResponse.access_token}
scope: ${credentials.scope}
```

Body:

```json
{
  "debtorNo": 123456,
  "divisionId": "SE01",
  "genesysContactId": "b1f2c3d4-5678-90ab-cdef-1234567890ab",
  "site": "SE"
}
```

String inputs are wrapped in `$!esc.jsonString(...)` in the `requestTemplate`, so quotes
and other special characters in the input values cannot break the JSON body. Do not
pre-escape the inputs. `debtorNo` is substituted unquoted because it is an integer.

### Input contract

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `debtorNo` | integer | yes | The Nova debtor number |
| `divisionId` | string | yes | The Nova division id |
| `genesysContactId` | string | yes | The Genesys Cloud external contact id to link to the debtor |
| `site` | string | yes | The Nova site / country code |

```json
{
  "debtorNo": 123456,
  "divisionId": "SE01",
  "genesysContactId": "b1f2c3d4-5678-90ab-cdef-1234567890ab",
  "site": "SE"
}
```

### Output contract

| Field | Type | Description |
| --- | --- | --- |
| `success` | boolean | True when the call succeeded |
| `data` | boolean | True when the external contact was added. `false` when no result was returned (HTTP 204) |
| `message` | string | Optional message returned by the API |
| `errors` | array of string | Any errors returned by the API |

```json
{
  "success": true,
  "data": true,
  "message": "",
  "errors": []
}
```

### Response handling

The upstream API uses the same envelope as the other Nova actions:

```json
{
  "success": true,
  "data": true,
  "message": "string",
  "errors": ["string"]
}
```

`translationMapDefaults` cover the **HTTP 204** case (request accepted but no body
returned): the action still completes successfully with `success: false`, `data: false`,
`message: "UNKNOWN"` and an empty `errors` array. Always check `success` **and** `data` in
your flow before treating the link as created — a 204 is not a confirmation.

**HTTP 401** means the integration credentials are missing or invalid — the data action
will fail and the failure branch in the flow will be taken.

This action is **not idempotent** from the flow's point of view; avoid retrying blindly on
timeouts without first confirming the current state in Nova.

### Prerequisites

1. A **Web Services Data Actions** integration in Genesys Cloud (Admin > Integrations),
   active and configured with credentials for `gcapi.alektumgroup.com`.
2. The integration must be configured with **User Defined (OAuth)** credentials, since the
   definition references `${authResponse.token_type}`, `${authResponse.access_token}` and
   `${credentials.scope}`. Make sure a `scope` field exists in the integration credentials.

### Import

1. Admin > Integrations > **Actions**.
2. **Add Action** > select your Web Services Data Actions integration > **Import** and
   choose `dataactions/nova-add-external-contact.json`.
3. Open the **Test** tab, run it against a known test debtor, then **Save & Publish**.
4. Reference the action from an Architect flow with a **Call Data Action** block, mapping
   the resolved debtor details and the external contact id (e.g. the contact id from a
   *Create External Contact* step) to the inputs.

---

## Nova - Update Nova Email

Updates the email address stored on a Nova debtor.

### Request

```
POST https://gcapi.alektumgroup.com/Service/UpdateNovaEmail
Accept: application/json
Content-Type: application/json
Authorization: ${authResponse.token_type} ${authResponse.access_token}
scope: ${credentials.scope}
```

Body:

```json
{
  "debtorNo": 123456,
  "site": "SE",
  "email": "debtor@example.com"
}
```

String inputs are wrapped in `$!esc.jsonString(...)` in the `requestTemplate`, so quotes
and other special characters in the input values cannot break the JSON body. Do not
pre-escape the inputs. `debtorNo` is substituted unquoted because it is an integer.

### Input contract

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `debtorNo` | integer | yes | The Nova debtor number |
| `site` | string | yes | The Nova site / country code |
| `email` | string | yes | The email address to store on the debtor |

```json
{
  "debtorNo": 123456,
  "site": "SE",
  "email": "debtor@example.com"
}
```

### Output contract

| Field | Type | Description |
| --- | --- | --- |
| `success` | boolean | True when the call succeeded |
| `data` | boolean | True when the email was updated. `false` when no result was returned (HTTP 204) |
| `message` | string | Optional message returned by the API |
| `errors` | array of string | Any errors returned by the API |

```json
{
  "success": true,
  "data": true,
  "message": "",
  "errors": []
}
```

### Response handling

The upstream API uses the same envelope as the other Nova actions:

```json
{
  "success": true,
  "data": true,
  "message": "string",
  "errors": ["string"]
}
```

`translationMapDefaults` cover the **HTTP 204** case (request accepted but no body
returned): the action still completes successfully with `success: false`, `data: false`,
`message: "UNKNOWN"` and an empty `errors` array. Always check `success` **and** `data` in
your flow before treating the update as applied — a 204 is not a confirmation.

**HTTP 401** means the integration credentials are missing or invalid — the data action
will fail and the failure branch in the flow will be taken.

### Prerequisites

1. A **Web Services Data Actions** integration in Genesys Cloud (Admin > Integrations),
   active and configured with credentials for `gcapi.alektumgroup.com`.
2. The integration must be configured with **User Defined (OAuth)** credentials, since the
   definition references `${authResponse.token_type}`, `${authResponse.access_token}` and
   `${credentials.scope}`. Make sure a `scope` field exists in the integration credentials.

### Import

1. Admin > Integrations > **Actions**.
2. **Add Action** > select your Web Services Data Actions integration > **Import** and
   choose `dataactions/nova-update-nova-email.json`.
3. Open the **Test** tab, run it against a known test debtor, then **Save & Publish**.
4. Reference the action from an Architect flow with a **Call Data Action** block, mapping
   the debtor id and updated email to the inputs.

---

## Nova - Update Nova POP

Triggers a Nova POP (screen pop) update for an agent on a conversation.
This action uses the `/NovaPop/...` base path, unlike the other actions that use `/Service/...`.

### Request

```
POST https://gcapi.alektumgroup.com/NovaPop/UpdateNovaPop
Accept: application/json
Content-Type: application/json
Authorization: ${authResponse.token_type} ${authResponse.access_token}
scope: ${credentials.scope}
```

Body:

```json
{
  "conversationId": "b1f2c3d4-5678-90ab-cdef-1234567890ab",
  "email": "agent@alektumgroup.com"
}
```

String inputs are wrapped in `$!esc.jsonString(...)` in the `requestTemplate`, so quotes
and other special characters in the input values cannot break the JSON body. Do not
pre-escape the inputs.

### Input contract

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `conversationId` | string | yes | The Genesys Cloud conversation id |
| `email` | string | yes | The agent's email address |

```json
{
  "conversationId": "b1f2c3d4-5678-90ab-cdef-1234567890ab",
  "email": "agent@alektumgroup.com"
}
```

### Output contract

| Field | Type | Description |
| --- | --- | --- |
| `success` | boolean | True when the call succeeded |
| `data` | boolean | True when the Nova POP was updated. `false` when no result was returned (HTTP 204) |
| `message` | string | Optional message returned by the API |
| `errors` | array of string | Any errors returned by the API |

```json
{
  "success": true,
  "data": true,
  "message": "",
  "errors": []
}
```

### Response handling

The upstream API uses the same envelope as the other Nova actions:

```json
{
  "success": true,
  "data": true,
  "message": "string",
  "errors": ["string"]
}
```

`translationMapDefaults` cover the **HTTP 204** case (request accepted but no body
returned): the action still completes successfully with `success: false`, `data: false`,
`message: "UNKNOWN"` and an empty `errors` array. Always check `success` **and** `data` in
your flow before treating the update as applied — a 204 is not a confirmation.

**HTTP 401** means the integration credentials are missing or invalid — the data action
will fail and the failure branch in the flow will be taken.

### Prerequisites

1. A **Web Services Data Actions** integration in Genesys Cloud (Admin > Integrations),
   active and configured with credentials for `gcapi.alektumgroup.com`.
2. The integration must be configured with **User Defined (OAuth)** credentials, since the
   definition references `${authResponse.token_type}`, `${authResponse.access_token}` and
   `${credentials.scope}`. Make sure a `scope` field exists in the integration credentials.

### Import

1. Admin > Integrations > **Actions**.
2. **Add Action** > select your Web Services Data Actions integration > **Import** and
   choose `dataactions/nova-update-nova-pop.json`.
3. Open the **Test** tab, run it with a real conversation id and agent email, then
   **Save & Publish**.
4. Reference the action from an Architect flow with a **Call Data Action** block, mapping
   the conversation id and agent email to the inputs.
