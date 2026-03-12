# OpenTelemetry Specification: Baggage & Configuration

> Baggage API, OTEL_* Environment Variables, Declarative Configuration (YAML)

---

## 8. Baggage Signal

### 8.1 Baggage API
[Source: baggage/api.md]

Baggage is an immutable set of application-defined name/value pairs propagated via Context per W3C Baggage Specification.

**Baggage names:**
- Any valid, non-empty UTF-8 strings
- Language API MUST treat baggage names as case-sensitive
- For maximum compatibility, alphanumeric names strongly recommended
- W3C propagator restricts keys to RFC7230 token definition

**Baggage values:**
- Any valid UTF-8 strings
- Language API MUST accept any valid UTF-8 string as value
- Language API MUST treat baggage values as case-sensitive

**Entry:** `{name, value, metadata?}` — metadata is opaque string, no semantic meaning.

**Each name MUST be associated with exactly one value (no duplicate names).**

**Operations:**
- `GetValue(name) -> value|null` — MUST take name as input; returns value or null if absent
- `GetAllValues() -> []Entry` — order MUST NOT be significant
- `SetValue(name, value, metadata?) -> Baggage` — MUST take name+value as input; returns NEW Baggage with entry added; if same name exists, new value MUST take precedence
- `RemoveValue(name) -> Baggage` — MUST take name as input; returns NEW Baggage without that entry
- `Len() -> int`
- `FromContext(context) -> Baggage`
- `ContextWithBaggage(context, baggage) -> Context`
- `ContextWithoutBaggage(context) -> Context` — API MUST provide way to remove all baggage entries

**Baggage container MUST be immutable.** MUST be fully functional without SDK installed.

**Propagation:**
- API layer MUST include TextMapPropagator implementing W3C Baggage Specification
- W3C format: `key=value,key2=value2;metadata`
- On extract: metadata stored as single metadata instance per entry
- On inject: metadata appended per W3C format

---

## 9. Configuration

### 9.1 Environment Variables
[Source: configuration/sdk-environment-variables.md]

**General rules:**
- SDK MUST interpret empty value of env var same as unset
- Boolean MUST be true only by case-insensitive string `"true"`; any other value MUST be interpreted as false
- Enum values SHOULD be interpreted case-insensitively; unrecognized MUST generate warning and be ignored
- Numeric values: if invalid, SHOULD generate warning and ignore
- Environment-based configuration MUST have direct code configuration equivalent
- When `OTEL_CONFIG_FILE` set, all other env vars (except those used for substitution) MUST be ignored

**Full Environment Variable Table:**

| Variable | Default | Description |
|---|---|---|
| `OTEL_SDK_DISABLED` | `false` | Disable SDK; if "true", use no-op SDK for all signals |
| `OTEL_RESOURCE_ATTRIBUTES` | (see semconv) | Resource attributes as key=value pairs |
| `OTEL_SERVICE_NAME` | (none) | Sets `service.name`; takes precedence over `OTEL_RESOURCE_ATTRIBUTES` |
| `OTEL_LOG_LEVEL` | `"info"` | SDK internal logger level |
| `OTEL_PROPAGATORS` | `"tracecontext,baggage"` | Comma-separated propagators; MUST be deduplicated |
| `OTEL_TRACES_SAMPLER` | `"parentbased_always_on"` | Sampler for traces |
| `OTEL_TRACES_SAMPLER_ARG` | (none) | Sampler argument; only used if `OTEL_TRACES_SAMPLER` set; invalid MUST be logged and ignored |
| `OTEL_TRACES_EXPORTER` | `"otlp"` | Trace exporter(s), comma-separated |
| `OTEL_METRICS_EXPORTER` | `"otlp"` | Metrics exporter(s), comma-separated |
| `OTEL_LOGS_EXPORTER` | `"otlp"` | Logs exporter(s), comma-separated |
| `OTEL_METRICS_EXEMPLAR_FILTER` | `"trace_based"` | Exemplar filter: `trace_based`\|`always_on`\|`always_off` |
| `OTEL_METRIC_EXPORT_INTERVAL` | `60000` | ms between metric exports |
| `OTEL_METRIC_EXPORT_TIMEOUT` | `30000` | ms max metric export time |
| `OTEL_EXPORTER_PROMETHEUS_HOST` | `"localhost"` | Prometheus exporter host [Development] |
| `OTEL_EXPORTER_PROMETHEUS_PORT` | `9464` | Prometheus exporter port [Development] |
| `OTEL_EXPORTER_ZIPKIN_ENDPOINT` | `http://localhost:9411/api/v2/spans` | Zipkin endpoint [Deprecated] |
| `OTEL_EXPORTER_ZIPKIN_TIMEOUT` | `10000` | Zipkin timeout ms [Deprecated] |
| `OTEL_BSP_SCHEDULE_DELAY` | `5000` | BatchSpanProcessor export delay ms |
| `OTEL_BSP_EXPORT_TIMEOUT` | `30000` | BatchSpanProcessor export timeout ms |
| `OTEL_BSP_MAX_QUEUE_SIZE` | `2048` | BatchSpanProcessor queue size |
| `OTEL_BSP_MAX_EXPORT_BATCH_SIZE` | `512` | BatchSpanProcessor batch size (≤ queue size) |
| `OTEL_BLRP_SCHEDULE_DELAY` | `1000` | BatchLogRecordProcessor export delay ms |
| `OTEL_BLRP_EXPORT_TIMEOUT` | `30000` | BatchLogRecordProcessor export timeout ms |
| `OTEL_BLRP_MAX_QUEUE_SIZE` | `2048` | BatchLogRecordProcessor queue size |
| `OTEL_BLRP_MAX_EXPORT_BATCH_SIZE` | `512` | BatchLogRecordProcessor batch size (≤ queue size) |
| `OTEL_ATTRIBUTE_VALUE_LENGTH_LIMIT` | (no limit) | Global max attribute value length |
| `OTEL_ATTRIBUTE_COUNT_LIMIT` | `128` | Global max attribute count |
| `OTEL_SPAN_ATTRIBUTE_VALUE_LENGTH_LIMIT` | (no limit) | Span-specific max attribute value length |
| `OTEL_SPAN_ATTRIBUTE_COUNT_LIMIT` | `128` | Span-specific max attribute count |
| `OTEL_SPAN_EVENT_COUNT_LIMIT` | `128` | Max span event count |
| `OTEL_SPAN_LINK_COUNT_LIMIT` | `128` | Max span link count |
| `OTEL_EVENT_ATTRIBUTE_COUNT_LIMIT` | `128` | Max attributes per span event |
| `OTEL_LINK_ATTRIBUTE_COUNT_LIMIT` | `128` | Max attributes per span link |
| `OTEL_LOGRECORD_ATTRIBUTE_VALUE_LENGTH_LIMIT` | (no limit) | LogRecord-specific max attribute value length |
| `OTEL_LOGRECORD_ATTRIBUTE_COUNT_LIMIT` | `128` | LogRecord-specific max attribute count |
| `OTEL_CONFIG_FILE` | (none) | Path to YAML config file; takes precedence over all other SDK config env vars |
| `OTEL_EXPERIMENTAL_CONFIG_FILE` | (none) | Deprecated — use `OTEL_CONFIG_FILE` |

**Known values for `OTEL_PROPAGATORS`:** `tracecontext`, `baggage`, `b3`, `b3multi`, `jaeger` (Deprecated), `xray` (3rd party), `ottrace` (3rd party, Deprecated), `none`

**Known values for `OTEL_TRACES_SAMPLER`:** `always_on`, `always_off`, `traceidratio`, `parentbased_always_on`, `parentbased_always_off`, `parentbased_traceidratio`, `parentbased_jaeger_remote`, `jaeger_remote`, `xray` (3rd party)

### 9.2 Declarative Configuration (SDK)
[Source: configuration/sdk.md]

- Config file specified via `OTEL_CONFIG_FILE`; MUST use `.yaml` or `.yml` extension; YAML 1.2+
- When set, all other env vars (except substitution references) MUST be ignored
- **Env var substitution:** `${env:VAR_NAME}` or `${VAR_NAME}`; default: `${VAR_NAME:-default_value}`; escape: `$$` → `$`; invalid refs MUST produce parse error
- **Parse operation:** Parses and validates config file; MUST differentiate missing vs. null properties; MUST return errors for invalid files, schema violations, missing required fields
- **Create operation:** Interprets config model, returns configured SDK components (TracerProvider, MeterProvider, LoggerProvider, Propagators); fail-fast if required property missing
- SDK MUST support registering custom plugin component providers (span exporters, samplers, propagators, etc.)

### 9.3 Configuration Data Model
[Source: configuration/data-model.md]

**Tier 2.** YAML schema defined in `opentelemetry-configuration` GitHub repo using JSON Schema. Top-level keys: `tracer_provider`, `meter_provider`, `logger_provider`, `propagators`, `instrumentation`. Each signal section has sub-sections for processors, exporters, samplers, readers, and plugins. Versioned via `file_format` field. Env var substitution performed during parsing.

### 9.4 Configuration API
[Source: configuration/api.md]

**Tier 2.** `ConfigProvider` API gives instrumentation libraries access to the `.instrumentation` mapping node via `GetInstrumentationConfig()`, returning a schemaless `ConfigProperties`. Supports global default and per-instance providers. `ConfigProperties` provides type-safe accessors for scalars, nested mappings, and sequences.
