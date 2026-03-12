# OpenTelemetry Specification: Metrics Signal

> MeterProvider, Meter, 7 Instrument Types, Views, Aggregation, Cardinality, Exemplars, MetricReader, MetricExporter

---

## 6. Metrics Signal

### 6.1 Metrics API
[Source: metrics/api.md]

#### MeterProvider

- `GetMeter(name, version?, schema_url?, attributes?) -> Meter`
  - MUST return working Meter as fallback (not null, not exception) for null/empty name; SHOULD log warning
- Global: `SetMeterProvider(provider)`, `GetMeterProvider() -> MeterProvider`
- All methods MUST be documented as safe for concurrent use

#### Meter

Creates instruments. MUST provide functions to create all 7 instrument types.

**Instrument naming rules (ABNF):**
```
instrument-name = ALPHA 0*254 ("_" / "." / "-" / "/" / ALPHA / DIGIT)
```
- Case-insensitive, ASCII, 1-255 characters, MUST start with letter
- Unit: optional, case-sensitive ASCII, max 63 chars
- Description: optional, MUST support BMP Unicode, min 1023 chars

**Instrument creation common parameters:**
- `name` (required)
- `unit` (optional, MUST NOT obligate)
- `description` (optional, MUST NOT obligate)
- `advisory` (optional, MUST NOT obligate — e.g., ExplicitBucketBoundaries for Histogram, Attributes list)

#### The 7 Instrument Types

| Instrument | Sync/Async | Measurement | API Method | Default Aggregation | Monotonic |
|---|---|---|---|---|---|
| Counter | Sync | non-negative delta | `Add(value, attrs?)` | Sum | Yes |
| UpDownCounter | Sync | any delta | `Add(value, attrs?)` | Sum | No |
| Histogram | Sync | any value | `Record(value, attrs?)` | ExplicitBucketHistogram | N/A |
| Gauge | Sync | current/instantaneous | `Record(value, attrs?)` | LastValue | N/A |
| ObservableCounter | Async | non-negative cumulative | `Observe(value, attrs?)` in callback | Sum | Yes |
| ObservableUpDownCounter | Async | any cumulative | `Observe(value, attrs?)` in callback | Sum | No |
| ObservableGauge | Async | any current value | `Observe(value, attrs?)` in callback | LastValue | N/A |

**Counter/UpDownCounter MUST NOT have API for creation other than via Meter** (same for all instrument types).

**Synchronous instruments:**
- API MUST allow flexible attributes at invocation time (dict/map or key-value args)
- SHOULD NOT return a value
- MUST accept variable number of attributes including none
- Counter `Add()`: increment value expected non-negative; SHOULD NOT validate, SHOULD document

**Asynchronous instruments (Callbacks):**
- MUST support creation by passing zero or more callbacks
- Every registered Callback MUST be evaluated exactly once per collection cycle
- Callbacks MUST be documented as: SHOULD be reentrant-safe; SHOULD NOT take indefinite time; SHOULD NOT make duplicate observations (multiple measurements with same attributes)
- API MUST treat observations from single Callback as logically at single instant — MUST report with identical timestamps
- User MUST be able to undo callback registration
- Multiple-instrument Callbacks MUST be associated with declared set of instruments at registration time

**Enabled() function (SHOULD support on sync instruments):**
- MUST return bool: true=enabled, false=disabled
- Value not static; can change over time

**Duplicate instrument registration:**
- If >1 instrument of same name created for same Meter with different identifying fields → warning SHOULD be emitted
- Meter MUST return functional instrument; SDKs MUST aggregate data from identical instruments together

**All Meter and Instrument methods MUST be documented as safe for concurrent use.**

---

### 6.2 Metrics SDK
[Source: metrics/sdk.md]

#### MeterProvider (SDK)

- MUST implement Get a Meter API
- Input MUST be used to create InstrumentationScope stored on Meter
- Configuration (MetricExporters, MetricReaders, Views, MeterConfigurator) MUST be owned by MeterProvider
- If config updated, MUST apply to all already-returned Meters
- `Shutdown()` — MUST be called only once; MUST invoke Shutdown on all MetricReaders and MetricExporters
- `ForceFlush()` — MUST invoke ForceFlush on all registered MetricReader instances

#### View

- SDK MUST provide functionality to create Views and register them with MeterProvider
- SDK MUST accept Instrument selection criteria AND stream configuration as inputs

**Selection criteria (all optional, all additive — Instrument must match ALL provided):**
- `name` — exact match or wildcard (`*` matches all; SDK MUST support single `*` at minimum; `?` = single char)
- `type` — instrument kind
- `unit` — instrument unit
- `meter_name`, `meter_version`, `meter_schema_url`

**Stream configuration (all optional):**
- `name` — if set, View SHOULD select at most one instrument (ambiguous = warning)
- `description` — if not provided, use instrument's description
- `attribute_keys` — allow-list; all others MUST be ignored; if not provided, use Attributes advisory param or keep all
- `aggregation` — if not provided, use MetricReader's default aggregation for instrument kind
- `exemplar_reservoir` — factory for reservoir; if not provided, use MeterProvider default
- `aggregation_cardinality_limit` — if not provided, use MetricReader's default (or global default 2000)

**View matching behavior:**
- If no View registered → apply default aggregation per instrument kind per MetricReader; honor advisory params
- If View matches → apply View's stream config; if conflict → SHOULD apply and emit warning
- If View and advisory params conflict on same aspect → View MUST take precedence
- If instrument doesn't match any View → SDK SHOULD enable using default aggregation

#### Aggregations

**SDK MUST provide:** Drop, Default, Sum, LastValue, ExplicitBucketHistogram
**SDK SHOULD provide:** Base2ExponentialBucketHistogram

**Default aggregation by instrument kind:**
| Instrument | Default Aggregation |
|---|---|
| Counter, AsyncCounter, UpDownCounter, AsyncUpDownCounter | Sum |
| Gauge, AsyncGauge | LastValue |
| Histogram | ExplicitBucketHistogram (with ExplicitBucketBoundaries advisory param if provided) |

**ExplicitBucketHistogram config:**
| Key | Default |
|---|---|
| Boundaries | `[0,5,10,25,50,75,100,250,500,750,1000,2500,5000,7500,10000]` |
| RecordMinMax | true |

**Base2ExponentialBucketHistogram config:**
| Key | Default |
|---|---|
| MaxSize | 160 (buckets per range) |
| MaxScale | 20 |
| RecordMinMax | true |

**Sum:** collects arithmetic sum; monotonicity determined by instrument type.
**LastValue:** collects last measurement + timestamp.
**Histograms:** collect count, sum (SHOULD NOT collect sum for UpDownCounter/ObservableGauge), min (optional), max (optional).
**Explicit buckets:** (lower, upper] inclusive; `-∞` to first boundary; last boundary to `+∞`.
**Drop:** discards all measurements.

#### Cardinality Limits

Default: **2000** data points per instrument per collection cycle.

**Priority order:**
1. View's `aggregation_cardinality_limit`
2. MetricReader's default cardinality limit for instrument kind
3. Global default (2000)

**Overflow attribute set:** `{otel.metric.overflow=true}`
- SDK MUST create Aggregator with overflow attribute set prior to reaching limit
- SDK MUST use overflow aggregator for measurements that couldn't get correct aggregator
- MUST guarantee overflow won't happen if ≤ limit distinct non-overflow attribute sets

**Cumulative temporality:** MUST continue exporting all attribute sets observed prior to overflow.
**Delta temporality:** MAY choose arbitrary subset to maintain limit.
**MUST NOT double-count or drop any measurement.**

#### Exemplars

`OTEL_METRICS_EXEMPLAR_FILTER` (default: `trace_based`): `trace_based` | `always_on` | `always_off`

**ExemplarFilter built-ins:**
- `AlwaysOn` — all measurements sampled
- `AlwaysOff` — no measurements sampled
- `TraceBased` (default) — only measurements associated with sampled trace span

**ExemplarReservoir:**
- MUST provide method to offer measurements
- New reservoir MUST be created for every known timeseries data point
- MUST be cleared/reset during collection before reading
- `collect()` MUST return accumulated Exemplars
- MUST retain attributes available in measurement that filter did not exclude

**Built-in reservoirs:**
1. `SimpleFixedSizeExemplarReservoir` — uniformly-weighted sampling; default size: 1 for LastValue/Sum; count of histogram buckets for Histogram
2. `AlignedHistogramBucketExemplarReservoir` — at most one Exemplar per histogram bucket

SDK MUST provide mechanism for custom ExemplarReservoir; configurable on metric View.

#### MetricReader

**Interface:**
- `Collect(exporter) -> MetricCollection`
- `Shutdown() -> result`
- `ForceFlush() -> result`

**Temporality:**
- Default: CUMULATIVE for all instrument kinds
- Reader configures temporality preference (cumulative or delta) per instrument kind
- `Collect` MUST return data per reader's temporality preference
- Successive Collect calls MUST repeat same Cumulative data points OR advance start timestamp for Delta

**SDK MUST support multiple MetricReader instances.** A MetricReader MUST NOT be registered on >1 MeterProvider.

**Built-in: PeriodicExportingMetricReader**
| Config | Env var | Default |
|---|---|---|
| exportIntervalMilliseconds | `OTEL_METRIC_EXPORT_INTERVAL` | 60000ms |
| exportTimeoutMilliseconds | `OTEL_METRIC_EXPORT_TIMEOUT` | 30000ms |

#### MetricExporter (Push)

**Interface:**
- `Export(metrics) -> ExportResult` — MUST NOT block indefinitely; `ExportResult`: Success, FailedNotRetryable, FailedRetryable
- `ForceFlush(timeout) -> result`
- `Shutdown(timeout) -> result`
- `Temporality(instrument_kind) -> temporality`
- `DefaultAggregation(instrument_kind) -> aggregation`

**Concurrency:** ForceFlush and Shutdown MUST be safe to call concurrently.

#### Metric Attribute Limits

| Config | Env var (specific) | Fallback env var | Default |
|---|---|---|---|
| AttributeCountLimit | `OTEL_METRIC_ATTRIBUTE_COUNT_LIMIT` | `OTEL_ATTRIBUTE_COUNT_LIMIT` | 128 |
| AttributeValueLengthLimit | `OTEL_METRIC_ATTRIBUTE_VALUE_LENGTH_LIMIT` | `OTEL_ATTRIBUTE_VALUE_LENGTH_LIMIT` | unlimited |

---

### 6.3 Metrics Data Model
[Source: metrics/data-model.md]

**Tier 2.** Five point types: Sum (arithmetic sum with monotonicity + temporality), Gauge (last observed value, no temporality), Histogram (explicit bucket distribution), ExponentialHistogram (base-2 auto-scaling buckets, MaxSize=160 default), Summary (quantiles; not recommended for new systems). Temporality: CUMULATIVE (start time = first observation or epoch), DELTA (start time = previous collection). Each timeseries MUST have one logical writer. Bucket inclusivity: (lower, upper] — measurement falls into greatest-numbered bucket with boundary ≥ measurement. ExponentialHistogram uses scale integer for resolution; zero_count + zero_threshold for near-zero values.

---

### 6.4 Metrics SDK Exporters
[Source: metrics/sdk_exporters/]

#### OTLP Exporter [metrics/sdk_exporters/otlp.md] — Status: Stable
- MUST provide temporality configuration (default: Cumulative for all instrument kinds)
- MUST configure per `OTEL_EXPORTER_OTLP_METRICS_TEMPORALITY_PREFERENCE`:
  - `cumulative` (default): Cumulative for all
  - `delta`: Delta for Counter/AsyncCounter/Histogram; Cumulative for UpDownCounter/AsyncUpDownCounter
  - `lowmemory`: Delta for sync Counter/Histogram; Cumulative for others
- MUST configure per `OTEL_EXPORTER_OTLP_METRICS_DEFAULT_HISTOGRAM_AGGREGATION`: `explicit_bucket_histogram` (default) | `base2_exponential_bucket_histogram`
- When auto-configured, MUST pair with PeriodicExportingMetricReader

#### Prometheus Exporter [metrics/sdk_exporters/prometheus.md] — Status: Development
- MUST be Pull Metric Exporter responding to HTTP requests
- MUST set MetricReader temporality as CUMULATIVE for all instrument kinds
- MUST convert OTel metrics per Prometheus Compatibility spec
- MUST support text format version 0.0.4
- MUST NOT use Prometheus Remote Write format or OpenMetrics protobuf format
- MUST NOT add explicit timestamps on Metric points
- MUST have at most one target_info metric
- Config: `host` (default: `localhost`), `port` (default: 9464)
- Translation strategies: `UnderscoreEscapingWithSuffixes` (default), `UnderscoreEscapingWithoutSuffixes`, `NoUTF8EscapingWithSuffixes`, `NoTranslation`
- Content negotiation MUST take precedence over configured translation strategy

#### Stdout Exporter [metrics/sdk_exporters/stdout.md] — Status: Stable
- Output format unspecified; SHOULD warn users it's for debugging only
- Default temporality: Cumulative; when auto-configured, pairs with PeriodicExportingMetricReader (default interval: 10000ms)

#### In-Memory Exporter [metrics/sdk_exporters/in-memory.md] — Status: Stable
- Accumulates metrics in local memory; useful for unit tests
- Default temporality: Cumulative; when auto-configured, pairs with PeriodicExportingMetricReader
