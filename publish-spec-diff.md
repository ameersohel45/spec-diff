# `on_discover` Endpoint — Spec Comparison

**Old spec:** `protocol-specifications-v2/api/beckn.yaml`
**New spec:** `rfc-protocol-spec/protocol-specifications-v2/api/v2.0.0/beckn.yaml`

---

## Level 1: `context` object

> Sources:
> - Old spec base Context: `https://raw.githubusercontent.com/beckn/protocol-specifications/.../transaction.yaml#/components/schemas/Context` + `DiscoveryContext` extension
> - New spec base Context: `https://schema.beckn.io/Context/v2.0` + `OnDiscoverAction/v2.0` extension

| Field | Old spec | Required (old) | New spec | Required (new) | Change |
|---|---|---|---|---|---|
| `domain` | `string` | ✅ required | ❌ removed | — | **Removed** |
| `location` | `Location` object | ✗ optional | ❌ removed | — | **Removed** |
| `key` | `string` | ✗ optional | ❌ removed | — | **Removed** |
| `schema_context` | `array<uri>` (discovery-specific) | ✗ optional | ❌ removed | — | **Removed** — no equivalent in new spec |
| `action` | `string`, enum `on_discover` | ✅ required | `BecknEndpoint` ref, const `"beckn/on_discover"` | ✅ required | **Value changed** — `"on_discover"` → `"beckn/on_discover"` |
| `version` | `string` | ✅ required | `string`, default `"2.0.0"` | ✅ required | Default added |
| `timestamp` | `string (date-time)` | ✅ required | `string (date-time)` | ✅ required | No change |
| `ttl` | `string (ISO8601)` | ✗ optional | `string (ISO8601)` | ✗ optional | No change |
| `bap_id` | `string` | ✅ required | ❌ → `bapId` | — | **Renamed** |
| `bapId` | ❌ | — | `string` | ✅ required | **Renamed from `bap_id`** |
| `bap_uri` | `string (URI)` | ✅ required | ❌ → `bapUri` | — | **Renamed** |
| `bapUri` | ❌ | — | `string (URI)` | ✅ required | **Renamed from `bap_uri`** |
| `transaction_id` | `string (UUID)` | ✅ required | ❌ → `transactionId` | — | **Renamed** |
| `transactionId` | ❌ | — | `string (UUID)` | ✅ required | **Renamed from `transaction_id`** |
| `message_id` | `string (UUID)` | ✅ required | ❌ → `messageId` | — | **Renamed** |
| `messageId` | ❌ | — | `string (UUID)` | ✅ required | **Renamed from `message_id`** |
| `bpp_id` | `string` | ✗ optional | ❌ → `bppId` | — | **Renamed** |
| `bppId` | ❌ | — | `string` | ✗ optional | **Renamed from `bpp_id`** |
| `bpp_uri` | `string (URI)` | ✗ optional | ❌ → `bppUri` | — | **Renamed** |
| `bppUri` | ❌ | — | `string (URI)` | ✗ optional | **Renamed from `bpp_uri`** |
| `networkId` | ❌ | — | `string` | ✗ optional | **New field** |
| `try` | ❌ | — | `boolean`, default `false` | ✗ optional | **New field** — negotiation flag |
| `lineage` | ❌ | — | `array[LineageEntry]`, maxItems: 1 | ✗ optional | **New field** — causal attribution |
| `additionalProperties` | not specified | — | `false` | — | **Now strict** |

### Key takeaways
1. **Field renames** — all `snake_case` → `camelCase` (`bap_id` → `bapId`, `transaction_id` → `transactionId`, etc.)
2. **Required set in old spec** — `domain`, `action`, `version`, `bap_id`, `bap_uri`, `transaction_id`, `message_id`, `timestamp`
3. **Required set in new spec** — `action`, `version`, `bapId`, `bapUri`, `transactionId`, `messageId`, `timestamp` (same count, `domain` dropped)
4. **Removed fields** — `domain`, `location`, `key`, `schema_context`
5. **New fields** — `networkId`, `try` (negotiation), `lineage` (causal attribution)
6. **`action` value** changed — `"on_discover"` → `"beckn/on_discover"` (path-prefixed)
7. **`additionalProperties: false`** now explicitly enforced in new spec

---

## Level 2: `message` object outer structure

### Old spec
```yaml
# top-level: required: [context, message]
message:
  type: object
  # no required list inside — catalogs is optional
  # no additionalProperties constraint
  properties:
    catalogs:
      type: array
      description: Array of catalogs containing items
      items:
        $ref: '#/components/schemas/Catalog'   # local ref
```

### New spec
```yaml
# top-level: required: [context, message]
message:
  type: object
  additionalProperties: true
  required:
    - catalogs                                  # catalogs now explicitly required
  properties:
    catalogs:
      type: array
      items:
        $ref: https://schema.beckn.io/Catalog/v2.1   # ⚠️ v2.1 not v2.0
```

| Aspect | Old spec | New spec | Change |
|---|---|---|---|
| `message` at top level | **required** | **required** | No change |
| `catalogs` inside message | present, **optional** (no required list) | present, **required** | **Now required** |
| `catalogs` description | "Array of catalogs containing items" | none | **Removed** |
| `additionalProperties` on message | not set (implicitly open) | explicitly `true` | Explicitly declared open |
| `Catalog` ref | local `#/components/schemas/Catalog` | external `schema.beckn.io/Catalog/v2.1` | **External ref + version bump to v2.1** |
| Other fields in message | none | none | No change |

### Key takeaways
1. `message` is **required** in both specs at top level — no change there
2. `catalogs` inside message was **optional** in old spec — now **explicitly required**
3. `Catalog` ref moved from local → external and bumped to **v2.1** (not v2.0 — correction from earlier)
4. `additionalProperties: true` now explicitly declared on message object

---

## Level 3: `message.catalogs[]` — Catalog object structure

> Both specs reference external Catalog schemas.
> - Old spec: `https://raw.githubusercontent.com/beckn/protocol-specifications-new/.../attributes.yaml#/components/schemas/Catalog`
> - New spec: `https://schema.beckn.io/Catalog/v2.1`

| Property | Old spec | Required (old) | New spec | Required (new) | Change |
|---|---|---|---|---|---|
| `@context` | `string (URI)` | ✅ required | `string (URI)`, **const** `"https://schema.beckn.io/"` | ✅ required | **Now a const value — locked to `schema.beckn.io/`** |
| `@type` | `string`, e.g. `"beckn:Catalog"` | ✅ required | `string`, **const** `"beckn:Catalog"` | ✅ required | **Now a const — value locked** |
| `beckn:id` | `string` | ✅ required | ❌ renamed | — | **Renamed → `id`** |
| `id` | ❌ | — | `string` | ✅ required | **Renamed from `beckn:id`** |
| `beckn:descriptor` | `Descriptor` (local ref) | ✅ required | ❌ renamed | — | **Renamed → `descriptor`** |
| `descriptor` | ❌ | — | `Descriptor/v2.1` | ✅ required | **Renamed from `beckn:descriptor`** + version bumped |
| `beckn:bppId` | `string` | ✅ required | ❌ renamed | — | **Renamed → `bppId`** |
| `bppId` | ❌ | — | `string` | ✅ required | **Renamed from `beckn:bppId`** |
| `beckn:bppUri` | `string (URI)` | ✅ required | ❌ renamed | — | **Renamed → `bppUri`** |
| `bppUri` | ❌ | — | `string (URI)` | ✅ required | **Renamed from `beckn:bppUri`** |
| `beckn:items` | `array[Item]` (local ref) | ✅ required | ❌ renamed | — | **Renamed → `items`** |
| `items` | ❌ | — | `array[Item/v2.1]` | ✅ required | **Renamed from `beckn:items`** + version bumped |
| `beckn:providerId` | `string` | ✗ optional | ❌ renamed | — | **Renamed → `providerId`** |
| `providerId` | ❌ | — | `string` | ✗ optional | **Renamed from `beckn:providerId`** |
| `beckn:validity` | `TimePeriod` (local ref) | ✗ optional | ❌ renamed | — | **Renamed → `validity`** |
| `validity` | ❌ | — | `TimePeriod/v2.1` | ✗ optional | **Renamed from `beckn:validity`** + version bumped |
| `beckn:isActive` | `boolean`, default `true` | ✗ optional | ❌ renamed | — | **Renamed → `isActive`** |
| `isActive` | ❌ | — | `boolean`, default `true` | ✗ optional | **Renamed from `beckn:isActive`** |
| `beckn:offers` | `array[Offer]` (local ref) | ✗ optional | ❌ renamed | — | **Renamed → `offers`** |
| `offers` | ❌ | — | `array[Offer/v2.1]` | ✗ optional | **Renamed from `beckn:offers`** + version bumped |
| `additionalProperties` | `false` | — | `false` | — | No change |

### Key takeaways
1. **All `beckn:` prefixed fields** renamed to plain names — same global pattern
2. **Required set is identical** in both: `@context`, `@type`, `id`, `descriptor`, `bppId`, `bppUri`, `items`
3. **`@context` is now a const** — locked to `"https://schema.beckn.io/"` — was a free URI in old spec
4. **`@type` is now a const** — locked to `"beckn:Catalog"` — was a free string in old spec
5. **All nested refs bumped to v2.1**: `Descriptor/v2.1`, `Item/v2.1`, `Offer/v2.1`, `TimePeriod/v2.1`
6. `additionalProperties: false` — strict in both, no change

---

## Level 4: `message.catalogs[].items[]` — Item object structure

> Both specs reference external Item schemas.
> - Old spec: `https://raw.githubusercontent.com/beckn/protocol-specifications-new/.../attributes.yaml#/components/schemas/Item`
> - New spec: `https://schema.beckn.io/Item/v2.1`

| Property | Old spec | Required (old) | New spec | Required (new) | Change |
|---|---|---|---|---|---|
| `@context` | `string (URI)` | ✅ required | `string (URI)`, **const** `"https://schema.beckn.io/"` | ✅ required | **Now a const — locked value** |
| `@type` | `string`, enum `"beckn:Item"` | ✅ required | `string`, enum `"Item"` | ✅ required | **Enum value changed** — `beckn:` prefix dropped |
| `beckn:id` | `string` | ✅ required | ❌ → `id` | — | **Renamed** |
| `id` | ❌ | — | `string` | ✅ required | **Renamed from `beckn:id`** |
| `beckn:descriptor` | `Descriptor` (local) | ✅ required | ❌ → `descriptor` | — | **Renamed** |
| `descriptor` | ❌ | — | `Descriptor/v2.1` | ✅ required | **Renamed + bumped to v2.1** |
| `beckn:provider` | `Provider` (local) | ✅ required | ❌ → `provider` | — | **Renamed** |
| `provider` | ❌ | — | `Provider/v2.1` | ✅ required | **Renamed + bumped to v2.1** |
| `beckn:itemAttributes` | `Attributes` (local) | ✅ required | ❌ → `itemAttributes` | — | **Renamed** |
| `itemAttributes` | ❌ | — | `Attributes/v2.0` ⚠️ | ✅ required | **Renamed — stays at v2.0** |
| `beckn:category` | `CategoryCode` (local) | ✗ optional | ❌ → `category` | — | **Renamed** |
| `category` | ❌ | — | `CategoryCode/v2.1` | ✗ optional | **Renamed + bumped to v2.1** |
| `beckn:availableAt` | `array[Location]` (local) | ✗ optional | ❌ → `availableAt` | — | **Renamed** |
| `availableAt` | ❌ | — | `array[Location/v2.0]` ⚠️ | ✗ optional | **Renamed — stays at v2.0** |
| `beckn:availabilityWindow` | `array[TimePeriod]` (local) | ✗ optional | ❌ → `availabilityWindow` | — | **Renamed** |
| `availabilityWindow` | ❌ | — | `array[TimePeriod/v2.1]` | ✗ optional | **Renamed + bumped to v2.1** |
| `beckn:rateable` | `boolean`, default `true` | ✗ optional | ❌ → `rateable` | — | **Renamed** |
| `rateable` | ❌ | — | `boolean` | ✗ optional | **Renamed — default removed** |
| `beckn:rating` | `Rating` (local) | ✗ optional | ❌ → `rating` | — | **Renamed** |
| `rating` | ❌ | — | `Rating/v2.1` | ✗ optional | **Renamed + bumped to v2.1** |
| `beckn:isActive` | `boolean`, default `true` | ✗ optional | ❌ → `isActive` | — | **Renamed** |
| `isActive` | ❌ | — | `boolean`, default `true` | ✗ optional | **Renamed** |
| `beckn:networkId` | `array[string]` | ✗ optional | ❌ → `networkId` | — | **Renamed** |
| `networkId` | ❌ | — | `array[string]` | ✗ optional | **Renamed** |
| `constraints` | ❌ not present | — | `array[Constraint/v2.0]` | ✗ optional | **New field** |
| `policies` | ❌ not present | — | `array[Policy/v2.0]` | ✗ optional | **New field** |
| `additionalProperties` | `false` | — | `false` | — | No change |

### Key takeaways
1. **All `beckn:` prefixed fields** renamed to plain names
2. **Required set unchanged**: `@context`, `@type`, `id`, `descriptor`, `provider`, `itemAttributes`
3. **`@context` now a const** — locked to `"https://schema.beckn.io/"`
4. **`@type` enum value** changed: `"beckn:Item"` → `"Item"`
5. **Mixed versioning** in new spec — most refs at v2.1 but `Attributes`, `Location`, `Constraint`, `Policy` stay at v2.0
6. **Two new fields**: `constraints` (v2.0) and `policies` (v2.0)
7. `rateable` default `true` removed in new spec

### Level 4.1: `item.descriptor` — Descriptor object (v2.1)

| Property | Old spec | Required (old) | New spec (v2.1) | Required (new) | Change |
|---|---|---|---|---|---|
| `@type` | `string`, enum `"beckn:Descriptor"` | ✅ required | `string`, default `"beckn:Descriptor"` | ✅ required | Enum → default; value same |
| `@context` | ❌ | — | `string (URI)`, use-case specific | ✗ optional | **New field** |
| `schema:name` | `string` | ✗ optional | ❌ → `name` | — | **Renamed** |
| `name` | ❌ | — | `string` | ✗ optional | **Renamed from `schema:name`** |
| `beckn:shortDesc` | `string` | ✗ optional | ❌ → `shortDesc` | — | **Renamed** |
| `shortDesc` | ❌ | — | `string` | ✗ optional | **Renamed from `beckn:shortDesc`** |
| `beckn:longDesc` | `string` | ✗ optional | ❌ → `longDesc` | — | **Renamed** |
| `longDesc` | ❌ | — | `string` | ✗ optional | **Renamed from `beckn:longDesc`** |
| `schema:image` | `array[URI]` | ✗ optional | ❌ → `thumbnailImage` | — | **Renamed + type changed** |
| `thumbnailImage` | ❌ | — | `string (URI)` — single value | ✗ optional | **Renamed from `schema:image`; array → single URI** |
| `docs` | ❌ | — | `array[Document]` | ✗ optional | **New field** |
| `mediaFile` | ❌ | — | `array[MediaFile]` | ✗ optional | **New field** |
| `additionalProperties` | not specified | — | `false` | — | **Now strict** |

**Key changes:**
- `schema:image` (array) → `thumbnailImage` (single URI) — **breaking type change**
- `schema:` and `beckn:` prefixes dropped from all field names
- Two new fields: `docs`, `mediaFile`
- `additionalProperties: false` now enforced

---

### Level 4.2: `item.provider` — Provider object (v2.1)

| Property | Old spec | Required (old) | New spec (v2.1) | Required (new) | Change |
|---|---|---|---|---|---|
| `beckn:id` | `string` | ✅ required | ❌ → `id` | — | **Renamed** |
| `id` | ❌ | — | `string` | ✅ required | **Renamed from `beckn:id`** |
| `beckn:descriptor` | `Descriptor` (local) | ✅ required | ❌ → `descriptor` | — | **Renamed** |
| `descriptor` | ❌ | — | `Descriptor/v2.1` | ✅ required | **Renamed + bumped to v2.1** |
| `beckn:validity` | `TimePeriod` (local) | ✗ optional | ❌ → `validity` | — | **Renamed** |
| `validity` | ❌ | — | `TimePeriod/v2.1` | ✗ optional | **Renamed + bumped to v2.1** |
| `beckn:locations` | `array[Location]` | ✗ optional | ❌ → `locations` | — | **Renamed** |
| `locations` | ❌ | — | `array[Location]` | ✗ optional | **Renamed** |
| `beckn:rateable` | `boolean` | ✗ optional | ❌ → `rateable` | — | **Renamed** |
| `rateable` | ❌ | — | `boolean` | ✗ optional | **Renamed** |
| `beckn:rating` | `Rating` (local) | ✗ optional | ❌ → `rating` | — | **Renamed** |
| `rating` | ❌ | — | `Rating/v2.1` | ✗ optional | **Renamed + bumped to v2.1** |
| `beckn:providerAttributes` | `Attributes` (local) | ✗ optional | ❌ → `providerAttributes` | — | **Renamed** |
| `providerAttributes` | ❌ | — | `Attributes/v2.0` ⚠️ | ✗ optional | **Renamed — stays at v2.0** |
| `@context` | ❌ | — | `string`, default `"https://schema.beckn.io/"` | ✗ optional | **New field** |
| `@type` | ❌ | — | `string`, default `"beckn:Provider"` | ✗ optional | **New field** |
| `alerts` | ❌ | — | `array[Alert/v2.0]` | ✗ optional | **New field** |
| `policies` | ❌ | — | `array[Policy]` | ✗ optional | **New field** |
| `additionalProperties` | `false` | — | `false` | — | No change |

**Key changes:**
- All `beckn:` prefixes dropped
- New fields: `@context`, `@type`, `alerts`, `policies`
- `rating` bumped to v2.1; `providerAttributes` stays at v2.0

---

### Level 4.3: `item.itemAttributes` — Attributes object (v2.0, unchanged)

| Property | Old spec | New spec (v2.0) | Change |
|---|---|---|---|
| `@context` | `string (URI)`, **required** | `string (URI)`, **required** | No change |
| `@type` | `string`, **required** | `string`, **required** | No change |
| *(domain-specific fields)* | any, via `@context` | any, via `@context` | No change |
| `additionalProperties` | `true` | `true` | No change |

**Key changes:** None — Attributes is still a JSON-LD open wrapper. No version bump to v2.1.

---

### Level 4.4: `item.category` — CategoryCode object (v2.1)

| Property | Old spec | Required (old) | New spec (v2.1) | Required (new) | Change |
|---|---|---|---|---|---|
| `@type` | `string`, enum `"schema:CategoryCode"` | ✅ required | `string`, default `"beckn:CategoryCode"` | ✅ required | **Namespace changed** `schema:` → `beckn:`; enum → default |
| `@context` | ❌ | — | `string (URI)`, default `"https://schema.beckn.io/"` | ✗ optional | **New field** |
| `schema:codeValue` | `string` | ✅ required | ❌ → `codeValue` | — | **Renamed** |
| `codeValue` | ❌ | — | `string` | ✅ required | **Renamed from `schema:codeValue`** — still required |
| `schema:name` | `string` | ✗ optional | ❌ → `name` | — | **Renamed** |
| `name` | ❌ | — | `string` | ✗ optional | **Renamed from `schema:name`** |
| `schema:description` | `string` | ✗ optional | ❌ → `description` | — | **Renamed** |
| `description` | ❌ | — | `string` | ✗ optional | **Renamed from `schema:description`** |
| `additionalProperties` | not specified | — | `false` | — | **Now strict** |

**Key changes:**
- `schema:` prefix dropped from all field names
- `@type` value namespace changed: `"schema:CategoryCode"` → `"beckn:CategoryCode"` *(correction from earlier — `codeValue` is still required)*
- New `@context` field with default value

---

### Level 4.5: `item.rating` — Rating object (v2.1)

| Property | Old spec | Required (old) | New spec (v2.1) | Required (new) | Change |
|---|---|---|---|---|---|
| `@context` | ❌ | — | `string (URI)`, const `"https://schema.beckn.io/"` | ✅ required | **New required field** |
| `@type` | `string`, enum `"beckn:Rating"` | ✅ required | `string`, default `"beckn:Rating"` | ✅ required | Enum → default; value same |
| `beckn:ratingValue` | `number (0–5)` | ✗ optional | ❌ → `ratingValue` | — | **Renamed** |
| `ratingValue` | ❌ | — | `number (0–5)` | ✗ optional | **Renamed from `beckn:ratingValue`** |
| `beckn:ratingCount` | `integer` | ✗ optional | ❌ → `ratingCount` | — | **Renamed** |
| `ratingCount` | ❌ | — | `integer (≥0)` | ✗ optional | **Renamed from `beckn:ratingCount`** |
| `reviewText` | ❌ | — | `string` | ✗ optional | **New field** |
| `additionalProperties` | not specified | — | `false` | — | **Now strict** |

**Key changes:**
- `@context` is now a **required** const — new breaking addition
- `beckn:` prefix dropped from field names
- New `reviewText` field

---

### Level 4.6: `item.constraints[]` — Constraint object (v2.0, new in v2.0)

> Not present in old spec. Entirely new.

| Property | Type | Required | Description |
|---|---|---|---|
| `@context` | `string (URI)`, const `"https://schema.beckn.io/"` | ✅ required | JSON-LD context |
| `@type` | `string`, const `"Constraint"` | ✅ required | Type identifier |
| `id` | `string` | ✅ required | Unique constraint identifier |
| `constraintType` | `string` | ✗ optional | Kind of restriction (extensible) |
| `operator` | `string` | ✗ optional | Comparison logic (`<=`, `>=`, `=`, etc.) |
| `value` | `number` | ✗ optional | Numeric constraint value |
| `unitCode` | `string` | ✗ optional | Unit of measurement (km, min, etc.) |
| `validity` | `TimePeriod/v2.0` | ✗ optional | When the constraint applies |
| `additionalProperties` | `false` | — | Strict |

---

### Level 4.7: `item.policies[]` — Policy object (v2.0, new in v2.0)

> Not present in old spec. Entirely new.

| Property | Type | Required | Description |
|---|---|---|---|
| `@context` | `string (URI)`, const `"https://schema.beckn.io/"` | ✅ required | JSON-LD context |
| `@type` | `string`, const `"Policy"` | ✅ required | Type identifier |
| `id` | `string` | ✅ required | Unique policy identifier |
| `descriptor` | `Descriptor/v2.0` | ✅ required | Human-readable policy description |
| `policyType` | `string` | ✗ optional | Category of policy (extensible) |
| `validity` | `TimePeriod/v2.0` | ✗ optional | When the policy applies |
| `policyAttributes` | `Attributes/v2.0` | ✗ optional | Additional policy details |
| `additionalProperties` | `false` | — | Strict |

---

### Item deep dive — full summary

| Sub-object | Ref in new spec | Key changes |
|---|---|---|
| `descriptor` | `Descriptor/v2.1` | `schema:image` → `thumbnailImage` (array → single URI); new `docs`, `mediaFile` |
| `provider` | `Provider/v2.1` | `beckn:` prefixes dropped; new `@context`, `@type`, `alerts`, `policies` |
| `itemAttributes` | `Attributes/v2.0` | **Unchanged** — stays v2.0, still open JSON-LD wrapper |
| `category` | `CategoryCode/v2.1` | `schema:` prefix dropped; `@type` value namespace `schema:` → `beckn:`; `codeValue` still required |
| `rating` | `Rating/v2.1` | `@context` now **required** const; new `reviewText` field |
| `availabilityWindow` | `TimePeriod/v2.1` | `schema:` prefix dropped from date/time fields |
| `availableAt` | `Location/v2.0` | ⚠️ stays at v2.0 |
| `constraints` | `Constraint/v2.0` | **Entirely new** |
| `policies` | `Policy/v2.0` | **Entirely new** |

---

## Level 5: `message.catalogs[].offers[]` — Offer object structure

> Both specs reference external Offer schemas.
> - Old spec: `https://raw.githubusercontent.com/beckn/protocol-specifications-new/.../attributes.yaml#/components/schemas/Offer`
> - New spec: `https://schema.beckn.io/Offer/v2.1`

| Property | Old spec | Required (old) | New spec | Required (new) | Change |
|---|---|---|---|---|---|
| `@context` | `string (URI)` | ✅ required | `string (URI)`, **const** `"https://schema.beckn.io/"` | ✅ required | **Now a const — locked value** |
| `@type` | `string`, enum `"beckn:Offer"` | ✅ required | `string`, enum `"Offer"` | ✅ required | **Enum value** — `beckn:` prefix dropped |
| `beckn:id` | `string` | ✅ required | ❌ → `id` | — | **Renamed** |
| `id` | ❌ | — | `string` | ✅ required | **Renamed from `beckn:id`** |
| `beckn:descriptor` | `Descriptor` (local) | ✅ required | ❌ → `descriptor` | — | **Renamed** |
| `descriptor` | ❌ | — | `Descriptor/v2.1` | ✅ required | **Renamed + bumped to v2.1** |
| `beckn:provider` | `Provider ID ref` | ✅ required | ❌ → `provider` | — | **Renamed** |
| `provider` | ❌ | — | `allOf[$ref: Provider/v2.1#/properties/id]` — string ID only | ✅ required | **Renamed — explicitly refs only the `id` field of Provider** |
| `beckn:items` | `array[Item ID]`, minItems: 1 | ✅ required | ❌ → `items` | — | **Renamed** |
| `items` | ❌ | — | `array[$ref: Item/v2.1#/properties/id]`, minItems: 1 | ✅ required | **Renamed — explicitly refs only `id` field of Item** |
| `beckn:addOns` | `array[Offer ID]` | ✗ optional | ❌ → `addOns` | — | **Renamed** |
| `addOns` | ❌ | — | `array[$ref: Offer/v2.1#/properties/id]` | ✗ optional | **Renamed — explicitly refs only `id` field of Offer** |
| `beckn:addOnItems` | `array[Item ID]` | ✗ optional | ❌ → `addOnItems` | — | **Renamed** |
| `addOnItems` | ❌ | — | `array[$ref: Item/v2.1#/properties/id]` | ✗ optional | **Renamed — explicitly refs only `id` field of Item** |
| `beckn:isActive` | `boolean`, default `true` | ✗ optional | ❌ → `isActive` | — | **Renamed** |
| `isActive` | ❌ | — | `boolean`, default `true` | ✗ optional | **Renamed** |
| `beckn:validity` | `TimePeriod` (local) | ✗ optional | ❌ → `validity` | — | **Renamed** |
| `validity` | ❌ | — | `TimePeriod/v2.1` | ✗ optional | **Renamed + bumped to v2.1** |
| `beckn:price` | `PriceSpecification` (local) | ✗ optional | ❌ → `price` | — | **Renamed** |
| `price` | ❌ | — | `PriceSpecification/v2.1` | ✗ optional | **Renamed + bumped to v2.1** |
| `beckn:eligibleRegion` | `array[Location]` | ✗ optional | ❌ → `eligibleRegion` | — | **Renamed** |
| `eligibleRegion` | ❌ | — | `array[Location/v2.0]` ⚠️ | ✗ optional | **Renamed — stays at v2.0** |
| `beckn:acceptedPaymentMethod` | `array[string]` | ✗ optional | ❌ → `acceptedPaymentMethod` | — | **Renamed** |
| `acceptedPaymentMethod` | ❌ | — | `AcceptedPaymentMethod/v2.0` ⚠️ | ✗ optional | **Renamed — stays at v2.0** |
| `beckn:offerAttributes` | `Attributes` (local) | ✗ optional | ❌ → `offerAttributes` | — | **Renamed** |
| `offerAttributes` | ❌ | — | `Attributes/v2.0` ⚠️ | ✗ optional | **Renamed — stays at v2.0** |
| `constraints` | ❌ not present | — | `array[Constraint/v2.0]` ⚠️ | ✗ optional | **New field** |
| `policies` | ❌ not present | — | `array[Policy/v2.0]` ⚠️ | ✗ optional | **New field** |
| `additionalProperties` | `false` | — | `false` | — | No change |

### Key takeaways
1. **All `beckn:` prefixed fields** renamed to plain names
2. **Required set unchanged**: `@context`, `@type`, `id`, `descriptor`, `provider`, `items`
3. **`@context` now a const** — locked to `"https://schema.beckn.io/"`
4. **`@type` enum value** changed: `"beckn:Offer"` → `"Offer"`
5. **`provider` and `items`** now explicitly ref only the `id` property via JSON Pointer (`#/properties/id`) — stricter ID-only reference
6. **`addOns` and `addOnItems`** also ref only `id` property — consistent ID-only pattern
7. **Mixed versioning** ⚠️ — `descriptor`, `validity`, `price` bumped to v2.1; `eligibleRegion`, `acceptedPaymentMethod`, `offerAttributes`, `constraints`, `policies` stay at v2.0
8. **Two new fields**: `constraints` and `policies`

---

### Level 5.1: `offer.descriptor` — Descriptor object (v2.1)

> Same schema as `item.descriptor` — see Level 4.1. No additional differences.

---

### Level 5.2: `offer.provider` — Provider reference (v2.1)

| Aspect | Old spec | New spec (v2.1) | Change |
|---|---|---|---|
| Type | Full Provider object ref | `allOf[$ref: Provider/v2.1#/properties/id]` — string ID only | **Breaking — full object → string ID only** |
| Resolution | `beckn:id`, `beckn:descriptor`, etc. all present | Only the `id` string value | **All provider fields gone from Offer** |

**Key change:** Old spec embedded a full Provider object. New spec uses a JSON Pointer ref to only the `id` property of `Provider/v2.1` — effectively just a string. Full provider detail lives at `Catalog` level, not inside each Offer.

---

### Level 5.3: `offer.price` — PriceSpecification object (v2.1)

| Property | Old spec (`beckn:price`) | New spec (`price`, `PriceSpecification/v2.1`) | Change |
|---|---|---|---|
| `currency` | `string (ISO 4217)`, optional | `string (ISO 4217)`, optional | No change |
| `value` | `number`, optional | `number`, optional | No change |
| `applicableQuantity` | `Quantity ref`, optional | `Quantity ref`, optional | No change |
| `components` | `array`, optional | `array`, optional | No change |
| `components[].type` | enum: `UNIT, TAX, DELIVERY, DISCOUNT, FEE, SURCHARGE` | enum: `UNIT, TAX, DELIVERY, DISCOUNT, FEE, SURCHARGE` | No change |
| `components[].value` | `number` | `number` | No change |
| `components[].currency` | `string` | `string` | No change |
| `components[].description` | `string` | `string` | No change |
| `additionalProperties` | `true` | `true` | No change |

**Key change:** Structure completely unchanged. Field renamed `beckn:price` → `price`. Description updated to "price snapshot — detailed models live in `offerAttributes`".

---

### Level 5.4: `offer.acceptedPaymentMethod` — Payment methods (v2.0, unchanged)

| Aspect | Old spec | New spec (v2.0) | Change |
|---|---|---|---|
| Type | `array[string]` | `array[string]` | No change |
| Enum values | `UPI, CREDIT_CARD, DEBIT_CARD, WALLET, BANK_TRANSFER, CASH, APPLE_PAY` | `UPI, CREDIT_CARD, DEBIT_CARD, WALLET, BANK_TRANSFER, CASH, APPLE_PAY` | No change |
| Field name | `beckn:acceptedPaymentMethod` | `acceptedPaymentMethod` | **Renamed — prefix dropped** |

**Key change:** Field name only. Enum values and type identical. Stays at v2.0.

---

### Level 5.5: `offer.validity` — TimePeriod object (v2.1)

| Property | Old spec | Required (old) | New spec (v2.1) | Required (new) | Change |
|---|---|---|---|---|---|
| `@type` | `string` (e.g. `"beckn:TimePeriod"`) | ✅ required | `string` | ✅ required | No change |
| `schema:startDate` | `string (date-time)` | ✗ optional | ❌ → `startDate` | — | **Renamed** |
| `startDate` | ❌ | — | `string (date-time)` | ✗ optional | **Renamed from `schema:startDate`** |
| `schema:endDate` | `string (date-time)` | ✗ optional | ❌ → `endDate` | — | **Renamed** |
| `endDate` | ❌ | — | `string (date-time)` | ✗ optional | **Renamed from `schema:endDate`** |
| `schema:startTime` | `string (time)` | ✗ optional | ❌ → `startTime` | — | **Renamed** |
| `startTime` | ❌ | — | `string (time)` | ✗ optional | **Renamed from `schema:startTime`** |
| `schema:endTime` | `string (time)` | ✗ optional | ❌ → `endTime` | — | **Renamed** |
| `endTime` | ❌ | — | `string (time)` | ✗ optional | **Renamed from `schema:endTime`** |
| Conditional | anyOf: startDate OR endDate OR (startTime+endTime) | — | anyOf: startDate OR endDate OR (startTime+endTime) | — | No change |
| `additionalProperties` | `false` | — | `false` | — | No change |

**Key change:** `schema:` prefix dropped from all date/time fields. Validation logic and constraints unchanged.

---

### Level 5.6: `offer.eligibleRegion[]` — Location object (v2.0, unchanged)

| Property | Old spec | Required (old) | New spec (v2.0) | Required (new) | Change |
|---|---|---|---|---|---|
| `@type` | `string`, enum `"beckn:Location"` | ✗ optional | `string`, const `"Location"` | ✗ optional | **Value changed** — `"beckn:Location"` → `"Location"` |
| `geo` | `GeoJSONGeometry` | ✅ required | `GeoJSONGeometry` | ✅ required | No change |
| `address` | `string` or `Address` object | ✗ optional | `string` or `Address` object | ✗ optional | No change |
| `additionalProperties` | `false` | — | `false` | — | No change |

**Key change:** `@type` value: `"beckn:Location"` → `"Location"`. Stays at v2.0.

---

### Level 5.7: `offer.constraints[]` and `offer.policies[]`

> Same schemas as `item.constraints[]` and `item.policies[]` — see Levels 4.6 and 4.7. Both stay at v2.0.

---

### Offer deep dive — full summary

| Sub-object | Ref in new spec | Key changes |
|---|---|---|
| `descriptor` | `Descriptor/v2.1` | Same as item.descriptor — see Level 4.1 |
| `provider` | `Provider/v2.1#/properties/id` | **Breaking** — full Provider object → string ID only |
| `price` | `PriceSpecification/v2.1` | Structure unchanged; field renamed `beckn:price` → `price` |
| `acceptedPaymentMethod` | `AcceptedPaymentMethod/v2.0` | Enum values identical; field name prefix dropped; stays v2.0 |
| `validity` | `TimePeriod/v2.1` | `schema:` prefix dropped from all date/time fields |
| `eligibleRegion` | `Location/v2.0` | `@type` value `"beckn:Location"` → `"Location"`; stays v2.0 |
| `offerAttributes` | `Attributes/v2.0` | Unchanged — stays v2.0 open JSON-LD wrapper |
| `constraints` | `Constraint/v2.0` | **New** — same as item.constraints |
| `policies` | `Policy/v2.0` | **New** — same as item.policies |

---

## Level 6: Remaining sub-objects

### Level 6.1: `descriptor.docs[]` — Document object *(new in v2.0)*

> Not present in old spec — `schema:image` array was the only media field. Entirely new.

| Property | Type | Required | Description |
|---|---|---|---|
| `label` | `string` | ✅ required | Display name of the document |
| `url` | `string (URI)` | ✅ required | Document location |
| `mimeType` | `string` | ✅ required | e.g. `application/pdf`, `image/png` |
| `standard` | `string` | ✗ optional | Schema classification e.g. `verifiableCredential` |
| `security.checksum` | `string` | ✗ optional | Hash value |
| `security.checksumAlgorithm` | `enum: SHA256, SHA512, OTHER` | ✗ optional | Hash algorithm |
| `security.signature` | `string` | ✗ optional | Digital signature |
| `security.signatureAlgorithm` | `string` | ✗ optional | e.g. `Ed25519`, `ES256` |
| `security.keyId` | `string` | ✗ optional | Key identifier |
| `additionalProperties` | `false` | — | Strict |

---

### Level 6.2: `descriptor.mediaFile[]` — MediaFile object *(new in v2.0)*

> Not present in old spec — `schema:image` array was the only media field. Entirely new.

| Property | Type | Required | Description |
|---|---|---|---|
| `label` | `string` | ✗ optional | Display name |
| `mimeType` | `string (URI)` | ✗ optional | MIME type when `data` is provided |
| `uri` | `string (URI)` | ✗ optional | URL to the media file |
| `additionalProperties` | `false` | — | Strict |

> **Note:** No required fields — all optional. Used for images, audio, video for display purposes.

---

### Level 6.3: `provider.alerts[]` — Alert object *(new in v2.0)*

> Not present in old spec. Entirely new — used to notify about issues affecting provider entities.

| Property | Type | Required | Description |
|---|---|---|---|
| `@context` | `string (URI)`, const `"https://schema.beckn.io/"` | ✅ required | JSON-LD context |
| `@type` | `string`, const `"Alert"` | ✅ required | Type identifier |
| `id` | `string` | ✅ required | Unique alert identifier |
| `descriptor` | `Descriptor/v2.1` | ✅ required | Human-readable alert description |
| `affectedEntities` | `array[string]` | ✗ optional | IDs of affected entities (route/order/fulfillment etc.) |
| `severity` | `string` | ✗ optional | Alert severity level |
| `validity` | `TimePeriod/v2.1` | ✗ optional | When the alert is active |
| `additionalProperties` | `false` | — | Strict |

---

### Level 6.4: `location.geo` — GeoJSONGeometry object

| Aspect | Old spec | New spec (v2.0) | Change |
|---|---|---|---|
| Supported types | `Point, LineString, Polygon, MultiPoint, MultiLineString, MultiPolygon, GeometryCollection` | Same | No change |
| `type` | `string`, **required** | `string`, **required** | No change |
| `coordinates` | `array` | `array` | No change |
| `geometries` | `array` (GeometryCollection only) | `array` (GeometryCollection only) | No change |
| `bbox` | `array[4]` optional | `array[4]` optional | No change |
| CRS | EPSG:4326 (WGS-84), `[lon, lat, alt?]` | EPSG:4326 (WGS-84), `[lon, lat, alt?]` | No change |
| `additionalProperties` | not specified | permitted | No change |

**Key change:** No structural changes. Circles explicitly documented as unsupported — use `SpatialConstraint` with `S_DWITHIN` + Point instead.

---

### Level 6.5: `location.address` — Address object

| Property | Old spec | New spec (v2.0) | Change |
|---|---|---|---|
| `streetAddress` | `string`, optional | `string`, optional | No change |
| `extendedAddress` | `string`, optional | `string`, optional | No change |
| `addressLocality` | `string`, optional | `string`, optional | No change |
| `addressRegion` | `string`, optional | `string`, optional | No change |
| `postalCode` | `string`, optional | `string`, optional | No change |
| `addressCountry` | `string`, optional | `string`, optional | No change |
| `additionalProperties` | not specified | `false` | **Now strict** |

**Key change:** `additionalProperties: false` now enforced. All fields remain optional, aligned with `schema.org/PostalAddress`.

---

### Level 6.6: `context.lineage[]` — LineageEntry object *(new in v2.0)*

> Not present in old spec. New field on context — documents causal attribution when a transaction originates from an upstream interaction.

| Property | Type | Required | Description |
|---|---|---|---|
| `action` | `BecknEndpoint` ref | ✅ required | Upstream endpoint that triggered this transaction |
| `transactionId` | `string (UUID)` | ✅ required | Upstream transaction's context `transactionId` |
| `messageId` | `string (UUID)` | ✅ required | Upstream message's `messageId` |
| `digest` | `string`, pattern `BLAKE-512=<base64>` | ✅ required | BLAKE2b-512 hash of upstream message body |
| `additionalProperties` | `false` | — | Strict |

> **Rules:** MUST NOT be included within steps of the same transaction. MUST NOT be propagated by downstream responses. Only at transaction boundaries.

---

## Level 7: `/beckn/on_discover` — Endpoint-level comparison

| Aspect | Old spec | New spec | Change |
|---|---|---|---|
| HTTP method | `POST` | `POST` | No change |
| `operationId` | `onDiscoverItems` | not set | **Removed** |
| `tags` | `[Discovery]` | not set | **Removed** |
| `summary` | `"On Discover response with catalog data"` | `"Beckn OnDiscover on an HTTP POST endpoint"` | **Changed — generic in new spec** |
| `Authorization` header | ❌ not required | ✅ **required** (`Beckn Signature`) | **New requirement** |
| Request schema ref | local `#/components/schemas/DiscoverResponse` | external `schema.beckn.io/OnDiscoverAction/v2.0` | **External ref** |
| Inline examples | ✅ `electronic_items_catalog`, `grocery_items_catalog` | ❌ none | **Removed** |
| `200` response | `AckResponse` (external ref) | `Ack` with `CounterSignature` | **New — response now includes counter-signature** |
| `400` response | `Ack400` (local) | `NackBadRequest` | **Renamed** |
| `401` response | ❌ not present | `NackUnauthorized` | **New response code** |
| `409` response | ❌ not present | `AckNoCallback` | **New response code** |
| `500` response | `Ack500` (local) | `ServerError` | **Renamed** |

### Key takeaways
1. **`Authorization` header now mandatory** — Beckn Signature required on every request
2. **Two new response codes** — `401 NackUnauthorized` and `409 AckNoCallback`
3. **`200` response now includes `CounterSignature`** — cryptographic proof receiver processed the request
4. **Removed** — `operationId`, `tags`, inline examples
5. Request schema moved fully external

---

## Level 8: `inReplyTo` — Verification

> Checked `OnDiscoverAction/v2.0` for top-level `inReplyTo` field.

**Result: Not present in `OnDiscoverAction/v2.0` body.**

`inReplyTo` is defined in the new spec's transport layer (`schema/core.yaml`) and is available as a schema for use in callbacks — but it is **not part of the `on_discover` request body**. It exists as a standalone schema for future use or optional binding at the network/implementation level.

---

## Level 9: Response schemas comparison

### Old spec response schemas

| Response | Schema | Fields |
|---|---|---|
| `200` | `AckResponse` (external) | `transaction_id` (required), `timestamp` (required), `ack_status: ACK\|NACK` (required), `error` (required if NACK) |
| `400` | `AckResponse` (same schema, NACK with error) | Same as above |
| `500` | `AckResponse` (same schema, NACK with error) | Same as above |

### New spec response schemas

| Response | Schema | Fields |
|---|---|---|
| `200` | `Ack` | `status: ACK\|NACK` (required), `signature: CounterSignature` (required) |
| `400` | `NackBadRequest` | `status: NACK` (const, required), `error: Error` (required), `signature: CounterSignature` (required) |
| `401` | `NackUnauthorized` | `status: NACK` (const, required), `error: Error` (required), `signature: CounterSignature` (required) |
| `409` | `AckNoCallback` | `status: ACK\|NACK` (required), `signature: CounterSignature` (required), `error: Error` (optional) |
| `500` | `ServerError` | empty object |

### Field-level comparison

| Aspect | Old spec (`AckResponse`) | New spec (`Ack`) | Change |
|---|---|---|---|
| Status field name | `ack_status` | `status` | **Renamed** |
| Status enum | `ACK`, `NACK` | `ACK`, `NACK` | No change |
| `transaction_id` | `string`, **required** | ❌ removed | **Removed** |
| `timestamp` | `string (date-time)`, **required** | ❌ removed | **Removed** |
| `error` | `object` (required if NACK) | `error.errorCode` + `error.errorMessage` | Restructured |
| `signature` | ❌ not present | `CounterSignature`, **required** | **New — cryptographic proof** |

### CounterSignature (new spec only)

Transmitted in the `Ack` response body. Proves the receiver authenticated and processed the inbound request.

| Field | Value |
|---|---|
| Wire format | Same as `Signature` (HTTP Signature scheme) |
| Signer | Response **receiver** (BPP/BAP) |
| `digest` | BLAKE2b-512 of the **Ack response** body |
| `(request-digest)` | BLAKE2b-512 of the **inbound request** body — MUST be present |
| `(message-id)` | messageId from inbound request context — MUST be present |
| `headers` | `"(created) (expires) digest (request-digest) (message-id)"` |

### Key takeaways
1. `ack_status` → `status` rename
2. `transaction_id` and `timestamp` **removed** from response — these are now only on the request context
3. `signature` (CounterSignature) now **required** on every response — bilateral non-repudiation
4. Error codes split into `errorCode` + `errorMessage` instead of single `error` object
5. `ServerError` (500) is now an **empty object** — no fields at all

---

## `/beckn/on_discover` — Complete comparison done ✅

---

# `beckn/discover` Endpoint Comparison

> _To be filled — starting next_
