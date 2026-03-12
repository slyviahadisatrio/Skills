# OpenTelemetry Specification: Cross-Cutting Concerns

> Context, Propagation, Resource, Common Attributes, Instrumentation Scope

---

## 2. Cross-Cutting: Context & Propagation

### 2.1 Context API
[Source: context/README.md]

**Status: Stable**

Context is an immutable propagation mechanism carrying execution-scoped values across API boundaries.

**API contracts:**
- `CreateKey(name) -> Key` — MUST accept key name; MUST return opaque key object
- `Value(context, key) -> value` — MUST return value in Context for key
- `WithValue(context, key, value) -> Context` — MUST return new Context with value set

**Immutability:** Context MUST be immutable. Write operations MUST create a new Context.

**Implicit Context (languages with implicit propagation):**
- `GetCurrent() -> Context` — MUST return Context associated with caller's current execution unit
- `Attach(context) -> Token` — MUST accept Context; MUST return Token to restore previous Context
- `Detach(token)` — MUST accept Token from Attach(); MAY emit signal on wrong call order

### 2.2 Propagators API
[Source: context/api-propagators.md]

**Status: Stable**

**TextMapPropagator interface:**
- `Inject(context, carrier, [setter])` — Injects values into carrier (e.g., HTTP headers). MUST retrieve value from Context first.
- `Extract(context, carrier, [getter]) -> Context` — Extracts values from carrier. MUST NOT throw if value cannot be parsed. MUST NOT store new value in Context if extraction fails.
- `Fields() -> []string` — Returns list of header field names this propagator uses.

**TextMapCarrier key/value pairs MUST only consist of US-ASCII characters valid for HTTP header fields (RFC 9110).**

**Getter interface:**
- MUST be stateless (save as constants)
- `Keys(carrier) -> []string` — MUST return all keys in carrier
- `Get(carrier, key) -> string` — MUST return first value of key or null; MUST be case-insensitive for HTTP
- `GetAll(carrier, key) -> []string` — MUST return all values; MUST be case-insensitive for HTTP

**Setter interface:**
- MUST be stateless
- `Set(carrier, key, value)` — MUST replace propagation field with given value; MUST preserve casing for case-sensitive protocols

**Composite Propagator:**
- Implementations MUST offer facility to group multiple Propagators
- `Create(propagators_list) -> CompositePropagator`
- Extract/Inject run all propagators in registration order

**Global Propagator:**
- `GetTextMapPropagator() -> TextMapPropagator` — MUST exist
- `SetTextMapPropagator(propagator)` — MUST exist
- MUST use no-op propagator unless explicitly configured
- Default: W3C TraceContext + W3C Baggage

**Built-in propagators:**

| Propagator | Env var value | Headers |
|---|---|---|
| W3C TraceContext | `tracecontext` | `traceparent`, `tracestate` |
| W3C Baggage | `baggage` | `baggage` |
| B3 Single | `b3` | `b3` |
| B3 Multi | `b3multi` | `X-B3-*` |

**B3 requirements:**
- MUST attempt extract using both single and multi-header formats; single-header takes precedence
- MUST preserve debug trace flag; MUST set sampled flag when debug flag set
- MUST NOT reuse X-B3-SpanId as server-side span id
- MUST default to injecting B3 using single-header format
- MUST NOT propagate X-B3-ParentSpanId

**OTEL_PROPAGATORS** env var: comma-separated list (default: `tracecontext,baggage`); MUST be deduplicated.

### 2.3 Environment Carriers
[Source: context/env-carriers.md]

**Status: Alpha.** Propagation via environment variables for process-boundary scenarios (batch jobs, CLI tools, CI/CD).

Recommended mappings: `TRACEPARENT` → `traceparent`, `TRACESTATE` → `tracestate`, `BAGGAGE` → `baggage`.

Size limits: Windows max 32,767 chars/pair; UNIX system-dependent. When truncation required, MUST truncate whole entries from end.

---

## 3. Cross-Cutting: Resource

### 3.1 Resource SDK
[Source: resource/sdk.md]

Resource is an immutable collection of attributes representing the entity producing telemetry.

**API contracts:**
- `Resource.Create(attributes, schema_url?) -> Resource`
- `Resource.Merge(old_resource, updating_resource) -> Resource`
  - Result MUST have all attributes from both inputs
  - If key exists on both, updating_resource value MUST be picked (even if empty)
  - Schema URL rules: use whichever is non-empty; if both non-empty and different → error (impossible to merge)
- `Resource.Empty()` — Recommended shortcut for empty resource
- `Resource.Attributes()` — Read-only collection; order not guaranteed

**SDK Default Resource:**
- SDK MUST provide a Resource with attributes from [Semantic Attributes with SDK-provided Default Value](https://github.com/open-telemetry/semantic-conventions/blob/main/docs/resource/README.md)
- This MUST be associated with TracerProvider or MeterProvider if no other resource explicitly specified
- All Spans from TracerProvider MUST be associated with its Resource
- All metrics from MeterProvider MUST be associated with its Resource

**Environment variables:**
- `OTEL_RESOURCE_ATTRIBUTES` — `key=value,key2=value2` pairs; all values MUST be treated as strings; `,` and `=` in keys/values MUST be percent-encoded (RFC 3986); MUST be merged as secondary resource (user-provided resource wins); invalid encoding MUST discard entire value and SHOULD report error
- `OTEL_SERVICE_NAME` — Sets `service.name`; takes precedence over `OTEL_RESOURCE_ATTRIBUTES`

**Resource Detectors:**
- Custom detectors MUST be implemented as packages separate from SDK
- Detector packages MUST provide method returning a Resource
- Failure to detect MUST NOT be considered an error
- Detectors populating semantic convention attributes MUST set Schema URL
- Multiple detectors with different non-empty Schema URLs MUST be an error

### 3.2 Resource Data Model
[Source: resource/data-model.md]

**Tier 2.** Resource consists of Entities (set of Entity objects) and Attributes. Entity has a type, identifying attributes (immutable), and descriptive attributes (mutable). Two resources differ if their entity sets or raw attributes differ. Schema URL field links resource to its semantic convention version.

---

## 4. Cross-Cutting: Common

### 4.1 AnyValue
[Source: common/README.md]

AnyValue supports: `string`, `bool`, `int64`, `double` (IEEE 754), `bytes`, `ArrayValue` (list of AnyValue), `KeyValueList` (ordered list of key-AnyValue pairs), or empty/null.

- Arbitrary deep nesting allowed for arrays and maps
- Empty values, zero values, empty strings, empty arrays MUST be stored and passed to processors/exporters
- For homogeneous arrays: null values SHOULD be avoided; if unavoidable, MUST be preserved as-is
- Exporters not supporting null MAY replace with 0, false, or empty strings

### 4.2 Attributes
[Source: common/README.md]

- Attribute key MUST be non-null and non-empty string (case-sensitive)
- Attribute value MUST be one of types defined in AnyValue
- Implementation MUST by default enforce unique keys in exported collections
- Setting attribute with duplicate key SHOULD overwrite existing value
- If option provided for duplicate keys, MUST be documented that handling is unpredictable

### 4.3 Attribute Limits
[Source: common/README.md]

**Defaults:**
- `AttributeCountLimit` = 128 (max attributes per record)
- `AttributeValueLengthLimit` = Infinity/unlimited (max string/byte length)

**Truncation rules (when limit configured):**
- String values MUST be truncated if exceeding limit (each character = 1)
- Byte arrays MUST be truncated if exceeding limit (each byte = 1)
- Array of strings: limit applied per element
- Otherwise: value MUST NOT be truncated

**Drop rules:**
- Adding attribute exceeding count limit: SDK MUST discard that attribute
- Count limit applies only to top-level attributes, MUST NOT apply to nested key-value pairs in maps
- Otherwise: attribute MUST NOT be discarded

**Signal-specific overrides:** MUST attempt model-specific first, then general, then model-specific default, then global default.

**Logging:** MAY emit log on truncation/discard. MUST NOT emit more than once per record.

**Exemptions:** Resource attributes SHOULD be exempt. Metric attributes exempt (see metrics SDK).

### 4.4 Instrumentation Scope
[Source: common/instrumentation-scope.md]

Identity tuple: `(name, version, schema_url, attributes)` — same tuple = same scope.

- `name` (required): Identifies scope (e.g., library/package fully-qualified name)
- `version` (optional): Version of the scope
- `schema_url` (optional): Telemetry Schema URL
- `attributes` (optional): Additional scope metadata

Obtained via `Provider.GetTracer/GetMeter/GetLogger(name, version?, schema_url?, attributes?)`.

### 4.5 Attribute Naming
[Source: common/attribute-naming.md]

Moved to [Semantic Conventions Naming](https://opentelemetry.io/docs/specs/semconv/general/naming/). See semantic conventions repo for guidelines.
