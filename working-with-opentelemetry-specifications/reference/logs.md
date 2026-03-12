# OpenTelemetry Specification: Logs Signal

> Logs Data Model, LoggerProvider, Logger, LogRecordProcessor, LogRecordExporter, Bridge Pattern

---

## 7. Logs Signal

### 7.1 Logs Data Model
[Source: logs/data-model.md]

#### LogRecord Fields

| Field | Type | Required? |
|---|---|---|
| Timestamp | uint64 Unix nanos | Optional |
| ObservedTimestamp | uint64 Unix nanos | SHOULD be set once observed |
| TraceId | bytes (16) | Optional |
| SpanId | bytes (8) | Optional; if present, TraceId SHOULD also be present |
| TraceFlags | byte (1) | Optional |
| SeverityNumber | int (1-24) | Optional; 0 = unspecified |
| SeverityText | string | Optional |
| Body | AnyValue | Optional; MUST support AnyValue semantics |
| Resource | Resource | — |
| InstrumentationScope | InstrumentationScope | — |
| Attributes | map<string, AnyValue> | Optional |
| EventName | string | Optional; non-empty = Event format |

#### Severity Ranges

| Range | Name | Values |
|---|---|---|
| 1-4 | TRACE | TRACE, TRACE2, TRACE3, TRACE4 |
| 5-8 | DEBUG | DEBUG, DEBUG2, DEBUG3, DEBUG4 |
| 9-12 | INFO | INFO, INFO2, INFO3, INFO4 |
| 13-16 | WARN | WARN, WARN2, WARN3, WARN4 |
| 17-20 | ERROR | ERROR, ERROR2, ERROR3, ERROR4 |
| 21-24 | FATAL | FATAL, FATAL2, FATAL3, FATAL4 |

- SeverityNumber >= 17 indicates erroneous situation; readers MAY apply special handling
- When reverse-mapping, choose target severity in same range closest numerically

#### Events (Named Log Records)
- Log record with non-empty EventName is an Event
- EventName SHOULD uniquely identify event structure (both attributes and body)
- Events SHOULD be used by OTel instrumentation per semantic conventions

---

### 7.2 Logs API
[Source: logs/api.md]

#### LoggerProvider

- `GetLogger(name, version?, schema_url?, attributes?) -> Logger`
  - MUST return working Logger as fallback for null/empty name (not null, not exception); SHOULD log warning
- Global: `SetLoggerProvider(provider)`, `GetLoggerProvider() -> LoggerProvider`
- All methods MUST be documented as safe for concurrent use

#### Logger

**`Emit(LogRecord)`** — MUST support:
- Parameters API MUST accept: Timestamp, ObservedTimestamp, Context (explicit or current), SeverityNumber, SeverityText, Body, Attributes, EventName
- Parameters API MAY accept: Exception
- When implicit Context supported: Context SHOULD be optional; if unspecified MUST use current Context
- When only explicit Context supported: Context SHOULD be required

**`Enabled(context?, severity?, event_name?) -> bool`** — SHOULD support:
- MUST return bool; true = Logger is enabled for provided arguments
- Value not static; can change over time

**All Logger methods MUST be documented as safe for concurrent use.**

---

### 7.3 Logs SDK
[Source: logs/sdk.md]

#### LoggerProvider (SDK)

- MUST implement Get a Logger API
- Configuration (LogRecordProcessors, LoggerConfigurator) MUST be owned by LoggerProvider
- If config updated, MUST apply to all already-returned Loggers
- `Shutdown()` — MUST be called only once; MUST invoke Shutdown on all LogRecordProcessors; MUST include effects of ForceFlush
- `ForceFlush()` — MUST invoke ForceFlush on all registered LogRecordProcessors

#### Logger (SDK)

**LoggerConfig [Development]:**
| Parameter | Type | Default | Rule |
|---|---|---|---|
| `enabled` | bool | true | If false, Logger MUST behave as No-op Logger |
| `minimum_severity` | SeverityNumber | 0 | If LogRecord's SeverityNumber specified (≠0) and < minimum_severity, MUST drop |
| `trace_based` | bool | false | If true, records associated with unsampled traces MUST be dropped |

**Emit processing:**
- If ObservedTimestamp unspecified, SHOULD set to current time
- If Exception provided, SDK MUST set attributes from exception per exception semantic conventions; user-provided attrs MUST take precedence (MUST NOT be overwritten)
- MUST apply filtering rules: minimum_severity drop, trace_based drop (even if Enabled not called)

**Enabled MUST return false when:**
- No registered LogRecordProcessors
- Logger disabled (LoggerConfig.enabled=false) [Development]
- Provided severity < configured minimum_severity [Development]
- trace_based=true and current context is unsampled [Development]
- All registered processors implement Enabled AND all return false

#### LogRecord Limits

| Config | Env var (specific) | Fallback env var | Default |
|---|---|---|---|
| AttributeCountLimit | `OTEL_LOGRECORD_ATTRIBUTE_COUNT_LIMIT` | `OTEL_ATTRIBUTE_COUNT_LIMIT` | 128 |
| AttributeValueLengthLimit | `OTEL_LOGRECORD_ATTRIBUTE_VALUE_LENGTH_LIMIT` | `OTEL_ATTRIBUTE_VALUE_LENGTH_LIMIT` | unlimited |

- If SDK implements limits, MUST provide way to change per LoggerProvider config
- Discard message MUST be logged at most once per LogRecord

#### ReadableLogRecord / ReadWriteLogRecord

**ReadableLogRecord:** Access all info added to LogRecord; MUST access InstrumentationScope and Resource; trace context fields MUST be populated from resolved Context.

**ReadWriteLogRecord (superset):** Additionally allows modifying Timestamp, ObservedTimestamp, SeverityText, SeverityNumber, Body, Attributes, TraceId, SpanId, TraceFlags, EventName.

#### LogRecordProcessor

**Interface:**
- `OnEmit(logRecord, context)` — called synchronously on emit thread; SHOULD NOT block; logRecord mutations MUST be visible to next registered processors; processor MAY freely modify logRecord during OnEmit
- `Enabled(context?, scope?, severity?, event_name?) -> bool` (optional) — for filtering via Logger.Enabled; filtering logic in OnEmit and Enabled MAY differ
- `Shutdown() -> result` — MUST be called only once; MUST include effects of ForceFlush
- `ForceFlush() -> result` — if has associated exporter, MUST call Export with pending records then ForceFlush on exporter

**Built-in: SimpleLogRecordProcessor** — passes each record to exporter immediately; MUST synchronize Export calls.

**Built-in: BatchLogRecordProcessor** — MUST synchronize Export calls.
| Config | Env var | Default |
|---|---|---|
| maxQueueSize | `OTEL_BLRP_MAX_QUEUE_SIZE` | 2048 |
| scheduledDelayMillis | `OTEL_BLRP_SCHEDULE_DELAY` | 1000ms |
| exportTimeoutMillis | `OTEL_BLRP_EXPORT_TIMEOUT` | 30000ms |
| maxExportBatchSize | `OTEL_BLRP_MAX_EXPORT_BATCH_SIZE` | 512 (MUST be ≤ maxQueueSize) |

#### LogRecordExporter

**Interface:**
- `Export(batch) -> ExportResult` — MUST NOT block indefinitely; result: Success or Failure
- `Shutdown() -> result` — SHOULD be called only once; subsequent Export SHOULD return Failure
- `ForceFlush() -> result`
- **Default SDK processors SHOULD NOT implement retry logic**

**Concurrency:** LoggerProvider (Logger creation, ForceFlush, Shutdown), Logger (all methods), LogRecordExporter (ForceFlush, Shutdown) MUST all be safe to call concurrently.

---

### 7.4 Logs SDK Exporters
[Source: logs/sdk_exporters/stdout.md]

**Stdout Exporter:** For debugging/learning only; output format unspecified. By default SHOULD be paired with SimpleLogRecordProcessor (when auto-configured via `OTEL_LOGS_EXPORTER=console`).

### 7.5 Supplementary Guidelines
[Source: logs/supplementary-guidelines.md]

**Tier 2.** Bridge pattern: Log appender acquires Logger, calls Emit for records from existing logging framework. Implicit Context injection: rely on automatic propagation (e.g., MDC in Log4j). Explicit Context injection: end user captures Context and passes to logging subsystem. Logger name from logging library (e.g., Log4j logger name, Monolog channel name) SHOULD be used as InstrumentationScope name.
