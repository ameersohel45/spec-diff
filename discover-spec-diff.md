# `beckn/discover` Endpoint — Spec Comparison

**Old spec:** `protocol-specifications-v2/api/beckn.yaml`
**New spec:** `rfc-protocol-spec/protocol-specifications-v2/api/v2.0.0/beckn.yaml`

---

## Level 0: Endpoint-level comparison

| Aspect | Old spec | New spec | Change |
|---|---|---|---|
| HTTP method | `GET` | `POST` | **Breaking — method changed** |
| `operationId` | `discoverItems` | not set | **Removed** |
| `tags` | `[Discovery]` | not set | **Removed** |
| `summary` | `"Discover items across multiple schemas"` | `"Beckn Discover on an HTTP POST endpoint"` | **Changed — generic in new spec** |
| `Authorization` header | ❌ not required | ✅ **required** (`Beckn Signature`) | **New requirement** |
| Request schema ref | local `#/components/schemas/DiscoverRequest` | external `schema.beckn.io/DiscoverAction/v2.0` | **External ref** |
| Inline examples | ✅ `structured_query`, `natural_language`, `grocery_search`, `combined_search`, `multi_schema_search` | ❌ none | **Removed** |
| `200` response | `oneOf: [AckResponse, DiscoverResponse]` — **synchronous OR async** | `Ack` with `CounterSignature` — **async only** | **Breaking — sync response removed** |
| `400` response | `Ack400` (local) | `NackBadRequest` | **Renamed** |
| `401` response | ❌ not present | `NackUnauthorized` | **New response code** |
| `409` response | ❌ not present | `AckNoCallback` | **New response code** |
| `500` response | `Ack500` (local) | `ServerError` | **Renamed** |

### Key takeaways
1. **HTTP method changed** `GET` → `POST` — biggest breaking change at endpoint level
2. **Sync response removed** — old spec could return `DiscoverResponse` directly in `200`; new spec always returns only `Ack` (results come via `on_discover` callback)
3. **`Authorization` header now mandatory** — Beckn Signature required
4. **Two new response codes** — `401 NackUnauthorized`, `409 AckNoCallback`

---

## Level 1: `context` object

> Same base `Context/v2.0` as `on_discover` — all the same field-level changes apply.
> See `on_discover_spec_comparison.md` Level 1 for full table.

| Field | Old spec | Required (old) | New spec | Required (new) | Change |
|---|---|---|---|---|---|
| `action` | `string`, enum `"discover"` | ✅ required | `BecknEndpoint`, const `"beckn/discover"` | ✅ required | **Value changed** — `"discover"` → `"beckn/discover"` |
| All other context fields | — | — | — | — | Same as `on_discover` context — see Level 1 of that doc |

---

## Level 2: `message` object outer structure

### Old spec
```yaml
# top-level: required: [context]  ← message is OPTIONAL at top level
message:
  type: object
  description: Discover payload containing search criteria
  # no required list at message level
  properties:
    text_search: ...
    filters: ...
    spatial: ...
    media_search: ...
  anyOf:
    - required: [text_search]
    - required: [filters]
    - required: [spatial]
    - required: [filters, spatial]
```

### New spec
```yaml
# top-level: required: [context, message]  ← message is REQUIRED
message:
  type: object
  additionalProperties: true
  required:
    - intent                               # intent object is required
  properties:
    intent:
      $ref: https://schema.beckn.io/Intent/v2.0
```

| Aspect | Old spec | New spec | Change |
|---|---|---|---|
| `message` at top level | ✗ **optional** (only `context` required) | ✅ **required** | **Now required** |
| Message structure | Flat — search fields directly on `message` | Nested — all search fields inside `message.intent` | **Breaking — one level deeper** |
| `additionalProperties` | not set | `true` | Explicitly open |
| Required inside message | `anyOf` on message directly | `intent` required, `anyOf` inside `intent` | **Moved down one level** |

### Key takeaways
1. **`message` itself** moved from optional → required
2. **Flat → nested** — search fields (`text_search`, `filters`, `spatial`, `media_search`) were directly on `message`; now all live inside `message.intent`
3. The `anyOf` validation rule moved from `message` level → `intent` level

---

## Level 3: `message.intent` — Intent object (new) vs old `message` fields

> Old spec had no `intent` wrapper — search fields were directly on `message`.
> New spec wraps everything inside `message.intent` (`Intent/v2.0`).

| Field | Old spec (on `message`) | Required (old) | New spec (on `message.intent`) | Required (new) | Change |
|---|---|---|---|---|---|
| `text_search` | `string` | anyOf | ❌ renamed | — | **Renamed** |
| `textSearch` | ❌ | — | `string` | anyOf | **Renamed from `text_search`** — snake_case → camelCase |
| `filters` | `object` | anyOf | `filters` | anyOf | No rename |
| `filters.type` | `string`, enum `[jsonpath]`, default `jsonpath` | ✅ required | `string`, enum `[jsonpath]`, default `jsonpath` | ✅ required | No change |
| `filters.expression` | `string` | ✅ required | `string` | ✅ required | No change |
| `spatial` | `array[SpatialConstraint]` | anyOf | `spatial` | anyOf | No rename |
| `media_search` | `$ref: MediaSearch` (local) | ✗ optional | ❌ renamed | — | **Renamed** |
| `media_search` → `media_search` | — | — | `$ref: MediaSearch/v2.0` (external) | ✗ optional | **Ref moved external** |
| `anyOf` rule | on `message` directly | — | on `intent` directly | — | **Moved one level down** |
| `additionalProperties` | not set | — | `false` | — | **Now strict** |

### Key takeaways
1. `text_search` → `textSearch` — snake_case to camelCase
2. `filters` and `spatial` field names unchanged
3. `media_search` ref moved from local → external `MediaSearch/v2.0`
4. `additionalProperties: false` now enforced on `intent`
5. `anyOf` validation rule same logic — just moved from `message` → `intent`

---

## Level 4: `message.intent.spatial[]` — SpatialConstraint object

| Property | Old spec | Required (old) | New spec (`SpatialConstraint/v2.0`) | Required (new) | Change |
|---|---|---|---|---|---|
| `op` | `string`, enum (lowercase): `s_within, s_contains, s_intersects, s_disjoint, s_overlaps, s_crosses, s_touches, s_equals, s_dwithin` | ✅ required | `string`, enum (UPPERCASE): `S_WITHIN, S_CONTAINS, S_INTERSECTS, S_DISJOINT, S_OVERLAPS, S_CROSSES, S_TOUCHES, S_EQUALS, S_DWITHIN` | ✅ required | **Enum values uppercased** |
| `targets` | `oneOf: string \| array[string]` | ✅ required | `oneOf: string \| array[string]` | ✅ required | No change |
| `geometry` | `GeoJSONGeometry` (local ref) | ✗ optional | `GeoJSONGeometry/v2.0` (external ref) | ✗ optional | Ref moved external |
| `distanceMeters` | `number`, min: 0 | ✗ optional | `number`, min: 0 | ✗ optional | No change |
| `quantifier` | `string`, enum `[any, all, none]` (lowercase), default `any` | ✗ optional | `string`, enum `[ANY, ALL, NONE]` (UPPERCASE), default `ANY` | ✗ optional | **Enum values uppercased** |
| `srid` | `string`, default `"EPSG:4326"` | ✗ optional | `string`, default `"EPSG:4326"` | ✗ optional | No change |
| `additionalProperties` | `false` | — | `false` | — | No change |

### Key takeaways
1. **`op` enum values uppercased** — `s_within` → `S_WITHIN`, `s_dwithin` → `S_DWITHIN` etc. — **breaking change**
2. **`quantifier` enum uppercased** — `any` → `ANY`, `all` → `ALL`, `none` → `NONE` — **breaking change**
3. `geometry` ref moved local → external `GeoJSONGeometry/v2.0`
4. All other fields and constraints unchanged

---

## Level 5: `message.intent.media_search` — MediaSearch object

| Property | Old spec | New spec (`MediaSearch/v2.0`) | Change |
|---|---|---|---|
| `media` | `array[MediaInput]` (local ref), optional | `array[media ref]`, optional | Ref moved external |
| `options` | `$ref: MediaSearchOptions` (local), optional | `options` object, optional | Ref moved external |
| `additionalProperties` | `false` | `false` | No change |

### Level 5.1: `media_search.media[]` — MediaInput object

| Property | Old spec (`MediaInput`) | Required (old) | New spec | Required (new) | Change |
|---|---|---|---|---|---|
| `type` | `string`, enum `[image, audio, video]` | ✅ required | `string`, enum `[image, audio, video]` | ✅ required | No change |
| `url` | `string (URI)` | ✅ required | `string (URI)` | ✅ required | No change |
| `id` | `string` | ✗ optional | `string` | ✗ optional | No change |
| `contentType` | `string` (MIME) | ✗ optional | `string` (MIME) | ✗ optional | No change |
| `textHint` | `string` — pre-extracted OCR/ASR text | ✗ optional | `string` | ✗ optional | No change |
| `language` | `string` (BCP-47) | ✗ optional | `string` (BCP-47) | ✗ optional | No change |
| `startMs` | `integer` — audio/video start offset | ✗ optional | `integer` | ✗ optional | No change |
| `endMs` | `integer` — audio/video end offset | ✗ optional | `integer` | ✗ optional | No change |
| `additionalProperties` | `false` | — | `false` | — | No change |

### Level 5.2: `media_search.options` — MediaSearchOptions object

| Property | Old spec (`MediaSearchOptions`) | New spec | Change |
|---|---|---|---|
| `goals` | `array[string]`, enum: `visual-similarity, visual-object-detect, text-from-image, text-from-audio, semantic-audio-match, semantic-video-match` | same | No change |
| `augment_text_search` | `boolean`, default `true` | `boolean`, default `true` | No change |
| `restrict_results_to_media_proximity` | `boolean`, default `false` | `boolean`, default `false` | No change |
| `additionalProperties` | `false` | `false` | No change |

**Key change:** MediaSearch, MediaInput, and MediaSearchOptions are structurally **identical** — only refs moved external.

---

---

## Level 6: `schema_context` — Removed discovery-specific context field

In the old spec, `DiscoveryContext` extended the base `Context` with one extra field:

```yaml
# Old spec — DiscoveryContext extension
schema_context:
  type: array
  items:
    type: string
    format: uri
  description: Optional JSON-LD context URLs indicating item types to search across
```

| Field | Old spec | New spec | Change |
|---|---|---|---|
| `schema_context` | `array<uri>`, optional — on context | ❌ removed | **Removed — no equivalent in `DiscoverAction/v2.0`** |

**Impact:** Clients using `schema_context` to scope discovery to specific item type schemas (e.g. `ElectronicItem`, `GroceryItem`) have no direct equivalent in the new spec. The intent filtering via `textSearch` + `filters` replaces this use case.

---

## Level 7: SpatialConstraint `targets` JSONPath — field name update

Since all item fields renamed from `beckn:fieldName` → `fieldName`, all JSONPath expressions in `targets` must be updated accordingly.

| Old spec target | New spec target | Reason |
|---|---|---|
| `$['beckn:availableAt'][*]['geo']` | `$['availableAt'][*]['geo']` | `beckn:` prefix dropped |
| `$['beckn:itemAttributes']['ride:serviceArea']['geo']` | `$['itemAttributes']['ride:serviceArea']['geo']` | `beckn:` prefix dropped |
| `$['beckn:itemAttributes']['ride:dropOff']['geo']` | `$['itemAttributes']['ride:dropOff']['geo']` | `beckn:` prefix dropped |

> **Note:** Domain-specific prefixes inside `itemAttributes` (e.g. `ride:serviceArea`) are governed by the domain's JSON-LD context — those may or may not change depending on the domain spec.

---

## Level 8: Response schemas comparison

> Same response schema changes as `on_discover` — see Level 9 in `on_discover_spec_comparison.md`.

### Quick reference

| Response | Old spec | New spec | Change |
|---|---|---|---|
| `200` | `AckResponse` — `ack_status`, `transaction_id`, `timestamp` | `Ack` — `status` + `CounterSignature` | **Restructured + signature added** |
| `200` (old only) | `oneOf: [AckResponse, DiscoverResponse]` — sync result possible | `Ack` only — always async | **Sync path removed** |
| `400` | `AckResponse` with NACK | `NackBadRequest` — `status: NACK` (const) + `error` + `CounterSignature` | **Renamed + signature added** |
| `401` | ❌ not present | `NackUnauthorized` | **New** |
| `409` | ❌ not present | `AckNoCallback` — request valid but no callback follows | **New** |
| `500` | `AckResponse` with NACK | `ServerError` — empty object | **Simplified to empty object** |

---

## `/beckn/discover` — Complete comparison done ✅
