---
name: tvv-arch-api-design
description: 'Design and document a REST API endpoint contract before implementation. Produces a complete request/response spec, error codes, and validation rules so there are no surprises during code review. Use before writing any new API endpoint. Triggers on: "design API", "API contract", "new endpoint", "REST design".'
version: 1.0.0
category: architecture
scope: [all]
triggers: [design API, API contract, new endpoint, REST design]
status: stable
created: '2026-07-23'
last_updated: '2026-07-29'
---

# tvv-arch-api-design: REST API Contract Design

Design the API endpoint described in `$ARGUMENTS` and produce a complete contract before any code is written.

**Step 1** — Read relevant existing code (routes, middleware, types, existing endpoints) to understand conventions already in use.

## Resource & URL

- Identify the resource noun (e.g. `rooms`, `events`, `readings`)
- Correct HTTP method for the operation (GET=read, POST=create, PUT=replace, PATCH=update, DELETE=remove)
- URL: `/api/{resource}/{id}/{sub-resource}` — plural nouns, no verbs
- Query parameters for filtering/sorting/pagination, not in the path

## Request

```
Method: POST /api/rooms/:roomId/readings
Headers:
  Content-Type: application/json
  x-api-key: <key>   (or Authorization: Bearer <token>)
Body: {
  "timestamp": "ISO8601",
  "occupancy": 0,
  ...
}
```

- Required vs optional fields explicitly listed
- Types and formats documented (ISO8601 for dates, integer ranges, string length limits)
- Validation rules: what gets rejected and with what error

## Response

```
201 Created
{
  "id": "uuid",
  "roomId": "string",
  "createdAt": "ISO8601",
  ...
}
```

- Success status code matches semantics (200=ok, 201=created, 204=no content)
- Response shape matches what callers actually need — no over-fetching
- Pagination shape if returning lists: `{ data: [...], nextCursor?: string }`

## Error Responses

| Status | When |
|--------|------|
| 400 | Validation failure — body explains which field and why |
| 401 | Missing or invalid auth |
| 403 | Authenticated but not authorized |
| 404 | Resource not found |
| 409 | Conflict (duplicate, stale write) |
| 422 | Business rule violation |
| 500 | Internal error — do not leak stack traces |

## Azure Functions Specifics (if applicable)

- HTTP trigger `authLevel` set appropriately (`anonymous` / `function` / `admin`)
- Cold-start implications noted if response time is latency-sensitive
- Route template in `function.json` matches the URL design above

## Checklist Before Handing to Implementation

- [ ] No verbs in URLs
- [ ] Consistent naming with existing endpoints
- [ ] Auth requirement documented
- [ ] All error cases covered
- [ ] Breaking change assessment if modifying existing endpoint

---

**Output the full contract ready for team review — implementation starts after this is agreed.**
