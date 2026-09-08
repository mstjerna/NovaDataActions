# NovaDataActions

Genesys Cloud **Web Services Data Actions** for Nova / `gcapi.alektumgroup.com` integrations.

## Actions

| File | Action | Endpoint |
| --- | --- | --- |
| [`dataactions/nova-get-pop-port-by-email.json`](dataactions/nova-get-pop-port-by-email.json) | Nova - Get POP Port by Agent Email | `GET /Service/novaPopPort/{userEmail}` |

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
