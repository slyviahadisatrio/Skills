# OpenTelemetry Specification: Compliance Checklist

> **How to use:** Check boxes as your implementation satisfies each requirement. MUST NOT items are violations if present. Organized by component for targeted auditing.

---

## Appendix A: Compliance Checklist

### A.1 General / Error Handling
[Source: error-handling.md, performance.md, versioning-and-stability.md]
- [ ] MUST NOT throw unhandled exceptions at runtime
- [ ] API methods MUST NOT throw unhandled exceptions when used incorrectly by end users
- [ ] SDK MUST NOT throw unhandled exceptions for errors in their own operations
- [ ] API methods accepting external callbacks MUST handle all errors
- [ ] SDK MUST return a "no-op" or default object when API returns non-null value and error occurs
- [ ] Library MUST log suppressed errors using language-specific conventions (SHOULD)
- [ ] SDK MUST allow end users to change the library's default error handling behavior
- [ ] MUST NOT cause the application to fail later at runtime due to dynamic config settings
- [ ] Backward-incompatible changes to stable API packages MUST NOT be made without major version bump
- [ ] All existing API calls MUST continue to compile and function against all future minor versions of same major
- [ ] Public portions of SDK packages MUST remain backwards compatible
- [ ] Signals MUST NOT be marked deprecated unless replacement is stable
- [ ] Major version MUST be bumped when breaking change to stable interface or deprecated signal removed
- [ ] OpenTelemetry clients MUST follow Semantic Versioning 2.0.0
- [ ] All stable API packages MUST version together across all signals (no separate version numbers)
- [ ] SDK packages for all signals MUST version together across all signals
- [ ] API stability MUST be maintained for minimum of 3 years after next major API version
- [ ] Terms denoting stability MUST NOT be used as part of directory or import names

### A.2 Context & Propagation
[Source: context/README.md, context/api-propagators.md]
- [ ] Context MUST be immutable
- [ ] Write operations on Context MUST result in creation of new Context
- [ ] CreateKey() MUST accept key name parameter
- [ ] CreateKey() MUST return opaque object representing newly created key
- [ ] Value() MUST accept Context and key; MUST return value for key
- [ ] WithValue() MUST accept Context, key, value; MUST return new Context
- [ ] GetCurrent() MUST return Context for caller's current execution unit
- [ ] Attach() MUST accept Context; MUST return Token to restore previous Context
- [ ] Detach() MUST accept Token from previous Attach()
- [ ] Propagators MUST define Inject and Extract operations
- [ ] Extract MUST NOT throw if value cannot be parsed from carrier
- [ ] Extract MUST NOT store new value in Context if extraction fails
- [ ] TextMapCarrier key/value pairs MUST only consist of US-ASCII valid HTTP header characters
- [ ] Getter and Setter MUST be stateless
- [ ] Setter.Set() MUST preserve casing for case-sensitive protocols
- [ ] Getter.Keys() MUST return all keys in carrier
- [ ] Getter.Get() MUST return first value or null; MUST be case-insensitive for HTTP
- [ ] Getter.GetAll() MUST return all values; MUST be case-insensitive for HTTP
- [ ] Implementations MUST offer facility to group multiple Propagators (CompositePropagator)
- [ ] GetTextMapPropagator() MUST exist
- [ ] SetTextMapPropagator() MUST exist
- [ ] API MUST use no-op propagator unless explicitly configured
- [ ] Pre-configured propagators MUST allow to be disabled or overridden
- [ ] W3C TraceContext propagator MUST parse and validate traceparent and tracestate headers
- [ ] B3: MUST attempt extract using single and multi-header formats; single-header takes precedence
- [ ] B3: MUST preserve debug trace flag; MUST set sampled flag when debug flag set
- [ ] B3: MUST NOT reuse X-B3-SpanId as server-side span id
- [ ] B3: MUST default to injecting using single-header format
- [ ] B3: MUST provide configuration to change injection to multi-header format
- [ ] B3: MUST NOT propagate X-B3-ParentSpanId
- [ ] OTEL_PROPAGATORS values MUST be deduplicated

### A.3 Resource SDK
[Source: resource/sdk.md]
- [ ] SDK MUST allow creation of Resources and associating them with telemetry
- [ ] All Spans from TracerProvider MUST be associated with its Resource
- [ ] All metrics from MeterProvider MUST be associated with its Resource
- [ ] SDK MUST provide access to Resource with SDK-provided default attributes
- [ ] Default Resource MUST be associated with TracerProvider/MeterProvider if not explicitly specified
- [ ] Resource.Create() interface MUST be provided
- [ ] Resource.Merge() interface MUST be provided
- [ ] Resulting merged resource MUST have all attributes from both inputs
- [ ] If key exists on both, updating_resource value MUST be picked (even if empty)
- [ ] Custom resource detectors MUST be implemented as packages separate from SDK
- [ ] Resource detector packages MUST provide method returning a Resource
- [ ] Failure to detect resource information MUST NOT be considered an error
- [ ] Detectors populating semantic convention attributes MUST set Schema URL
- [ ] Multiple detectors with different non-empty Schema URLs MUST be an error
- [ ] SDK MUST extract information from OTEL_RESOURCE_ATTRIBUTES
- [ ] All attribute values in OTEL_RESOURCE_ATTRIBUTES MUST be treated as strings
- [ ] `,` and `=` in OTEL_RESOURCE_ATTRIBUTES keys/values MUST be percent-encoded

### A.4 Common / Attributes
[Source: common/README.md, common/instrumentation-scope.md]
- [ ] Homogeneous arrays MUST NOT contain values of different types
- [ ] Empty values, zero, empty string, empty array MUST be stored and passed to processors/exporters
- [ ] Null values within arrays MUST be preserved as-is
- [ ] Attribute key MUST be non-null and non-empty string
- [ ] Attribute value MUST be one of types defined in AnyValue
- [ ] Implementation MUST by default enforce unique keys in exported attribute collections
- [ ] If option provided for duplicate keys, MUST be documented
- [ ] SDK MUST truncate string attribute value if exceeding length limit
- [ ] SDK MUST truncate byte array if length exceeds limit
- [ ] SDK MUST discard attribute if adding would exceed count limit
- [ ] Count limit MUST NOT apply to nested key-value pairs in maps
- [ ] Log MUST NOT be emitted more than once per record on truncation/discard
- [ ] SDK MUST provide way to change attribute limits programmatically

### A.5 Traces API
[Source: trace/api.md]
- [ ] GetTracer() MUST accept name, version, schema_url, attributes parameters
- [ ] Working Tracer MUST be returned as fallback (not null, not exception) for invalid name
- [ ] Implementations MUST NOT require repeatedly obtaining Tracer with same identity for config changes
- [ ] MUST NOT be any API for creating Span other than with Tracer
- [ ] Span creation MUST NOT set newly created Span as active in current Context by default
- [ ] StartSpan MUST NOT accept Span or SpanContext as parent — only full Context
- [ ] Each root span MUST be created with new TraceId
- [ ] Child span TraceId MUST match parent TraceId
- [ ] Child span MUST inherit all TraceState values of parent by default
- [ ] IsRemote MUST return true when extracted via Propagators
- [ ] IsRemote MUST return false for child spans
- [ ] TraceId MUST be 32-hex-character lowercase string in hex form
- [ ] SpanId MUST be 16-hex-character lowercase string in hex form
- [ ] Binary TraceId MUST be 16-byte array; Binary SpanId MUST be 8-byte array
- [ ] IsValid MUST be provided (true if non-zero TraceId AND non-zero SpanId)
- [ ] IsRemote MUST be provided
- [ ] All TraceState mutations MUST return new TraceState
- [ ] TraceState MUST always be valid per W3C Trace Context spec
- [ ] TraceState mutations MUST validate input; MUST NOT return invalid data
- [ ] Any span that is created MUST also be ended (user responsibility)
- [ ] GetContext() MUST return same SpanContext for entire Span lifetime
- [ ] IsRecording MUST return false to signal events/attributes not being recorded
- [ ] SetStatus() MUST be provided; Description MUST be IGNORED for Ok and Unset
- [ ] Ok status MUST override any prior or future Error/Unset attempts
- [ ] End() MUST NOT block on calling thread
- [ ] End() MUST NOT affect child spans
- [ ] End() MUST NOT inactivate Span in any Context it is active in
- [ ] RecordException() MUST record as Event named "exception"
- [ ] NonRecordingSpan: GetContext MUST return wrapped SpanContext; IsRecording MUST return false
- [ ] NonRecordingSpan MUST be fully implemented in API
- [ ] TracerProvider all methods MUST be safe for concurrent use
- [ ] Tracer all methods MUST be safe for concurrent use
- [ ] Span all methods MUST be safe for concurrent use
- [ ] API documentation MUST state adding attributes at span creation is preferred
- [ ] API documentation MUST state adding links at span creation is preferred

### A.6 Traces SDK
[Source: trace/sdk.md]
- [ ] TracerProvider MUST implement Get a Tracer API
- [ ] Input MUST be used to create InstrumentationScope on Tracer
- [ ] Configuration MUST be owned by TracerProvider
- [ ] Updated configuration MUST apply to all already-returned Tracers
- [ ] TracerProvider.Shutdown() MUST be called only once; subsequent Tracer gets return no-op
- [ ] TracerProvider.Shutdown() MUST invoke Shutdown on all SpanProcessors
- [ ] TracerProvider.ForceFlush() MUST invoke ForceFlush on all SpanProcessors
- [ ] SDK MUST NOT allow SampledFlag=true with IsRecording=false
- [ ] SpanProcessor.OnStart() SHOULD NOT block or throw
- [ ] SpanProcessor.Shutdown() MUST be called only once; MUST include effects of ForceFlush
- [ ] SpanProcessor.ForceFlush() MUST prioritize timeout over completeness
- [ ] Built-in SpanProcessors MUST call exporter's Export then ForceFlush during ForceFlush
- [ ] SDK MUST randomly generate both TraceId and SpanId by default
- [ ] SDK MUST provide mechanism for customizing ID generation
- [ ] Custom IdGenerators for vendor-specific protocols MUST NOT be in Core OTel repos
- [ ] BatchSpanProcessor.maxExportBatchSize MUST be ≤ maxQueueSize
- [ ] SpanExporter.Export() MUST NOT block indefinitely
- [ ] SpanExporter.Shutdown() — subsequent Export SHOULD return Failure
- [ ] Default SDK SpanProcessors SHOULD NOT implement retry logic
- [ ] SpanExporter.ForceFlush() and Shutdown() MUST be safe for concurrent use
- [ ] If SDK implements span limits, MUST provide way to change via config
- [ ] Discard message MUST be logged at most once per Span

### A.7 Metrics API
[Source: metrics/api.md]
- [ ] GetMeter() MUST accept name, version, schema_url, attributes parameters
- [ ] Working Meter MUST be returned as fallback (not null, not exception) for invalid name
- [ ] Meter MUST provide functions to create all instrument types
- [ ] MUST NOT be any API for creating Counter other than with Meter (same for all instrument types)
- [ ] Instrument name MUST start with letter; MUST be 1-255 chars; alphanumeric+_-./
- [ ] Instrument unit MUST be case-sensitive ASCII, max 63 chars
- [ ] Instrument description MUST support BMP Unicode, min 1023 chars
- [ ] API MUST allow flexible attributes at invocation time (variable number including none)
- [ ] Async: Every registered Callback MUST be evaluated exactly once per collection cycle
- [ ] Async: API MUST treat observations from single Callback as at single instant (identical timestamps)
- [ ] Async: User MUST be able to undo callback registration
- [ ] Async: Multiple-instrument Callbacks MUST be associated with declared instruments at registration
- [ ] MeterProvider all methods MUST be safe for concurrent use
- [ ] Meter all methods MUST be safe for concurrent use
- [ ] Instrument all methods MUST be safe for concurrent use

### A.8 Metrics SDK
[Source: metrics/sdk.md]
- [ ] MeterProvider MUST implement Get a Meter API
- [ ] Input MUST be used to create InstrumentationScope on Meter
- [ ] Configuration MUST be owned by MeterProvider
- [ ] Updated configuration MUST apply to all already-returned Meters
- [ ] MeterProvider.Shutdown() MUST be called only once; MUST invoke Shutdown on all MetricReaders/Exporters
- [ ] MeterProvider.ForceFlush() MUST invoke ForceFlush on all MetricReader instances
- [ ] SDK MUST provide functionality to create Views and register them with MeterProvider
- [ ] Views MUST accept Instrument selection criteria AND stream configuration
- [ ] SDK MUST accept name, type, unit, meter_name, meter_version, meter_schema_url as selection criteria
- [ ] If View and advisory params conflict on same aspect, View MUST take precedence
- [ ] SDK MUST provide Drop, Default, Sum, LastValue, ExplicitBucketHistogram aggregations
- [ ] SDK SHOULD provide Base2ExponentialBucketHistogram aggregation
- [ ] Cardinality overflow: SDK MUST create overflow Aggregator before reaching limit
- [ ] Cardinality: MUST NOT double-count or drop any measurement
- [ ] Cardinality cumulative: MUST continue exporting all attribute sets observed prior to overflow
- [ ] MetricReader MUST NOT be registered on >1 MeterProvider
- [ ] SDK MUST support multiple MetricReader instances
- [ ] MetricReader.Collect() MUST return data per reader's temporality preference
- [ ] MetricExporter.Export() MUST NOT block indefinitely
- [ ] When exemplar filter off, SDK MUST NOT have overhead related to exemplar sampling
- [ ] ExemplarReservoir MUST be cleared/reset during collection before reading
- [ ] Meter MUST return instrument using first-seen name casing; MUST log error on name conflict
- [ ] Duplicate instrument registration: MUST return functional instrument; warning SHOULD be emitted
- [ ] SDK MUST aggregate data from identical instruments together

### A.9 Logs API
[Source: logs/api.md]
- [ ] GetLogger() MUST accept name, version, schema_url, attributes parameters
- [ ] Working Logger MUST be returned as fallback (not null, not exception) for invalid name
- [ ] Logger MUST provide Emit function
- [ ] Logger SHOULD provide Enabled function
- [ ] When implicit Context supported: unspecified Context MUST use current Context
- [ ] Optional parameters: API MUST accept but MUST NOT obligate user to provide
- [ ] Required parameters: API MUST obligate user to provide
- [ ] LoggerProvider all methods MUST be safe for concurrent use
- [ ] Logger all methods MUST be safe for concurrent use

### A.10 Logs SDK
[Source: logs/sdk.md]
- [ ] LoggerProvider MUST implement Get a Logger API
- [ ] Input MUST be used to create InstrumentationScope on Logger
- [ ] Configuration MUST be owned by LoggerProvider
- [ ] Updated configuration MUST apply to all already-returned Loggers
- [ ] LoggerProvider.Shutdown() MUST be called only once; MUST invoke Shutdown on all LogRecordProcessors
- [ ] LoggerProvider.ForceFlush() MUST invoke ForceFlush on all LogRecordProcessors
- [ ] If Logger disabled (LoggerConfig.enabled=false), MUST behave as No-op Logger [Development]
- [ ] If LogRecord's SeverityNumber < minimum_severity, MUST drop [Development]
- [ ] If trace_based=true and record associated with unsampled trace, MUST drop [Development]
- [ ] If Exception provided, SDK MUST set attributes from exception; user attrs MUST take precedence
- [ ] User-provided attributes MUST NOT be overwritten by exception-derived attributes
- [ ] Before processing, MUST apply filtering rules (minimum_severity, trace_based) [Development]
- [ ] Enabled MUST return false when no registered LogRecordProcessors
- [ ] ReadableLogRecord: MUST access InstrumentationScope and Resource
- [ ] Trace context fields MUST be populated from resolved Context
- [ ] LogRecordProcessor.OnEmit() mutations MUST be visible to next registered processors
- [ ] LogRecordProcessor.Shutdown() MUST be called only once; MUST include effects of ForceFlush
- [ ] BatchLogRecordProcessor.maxExportBatchSize MUST be ≤ maxQueueSize
- [ ] LogRecordExporter.Export() MUST NOT block indefinitely
- [ ] If SDK implements log limits, MUST provide way to change per LoggerProvider config
- [ ] Discard message MUST be logged at most once per LogRecord

### A.11 Baggage API
[Source: baggage/api.md]
- [ ] Each name MUST be associated with exactly one value
- [ ] Language API MUST accept any valid UTF-8 string as baggage value
- [ ] Language API MUST treat baggage names as case-sensitive
- [ ] Language API MUST treat baggage values as case-sensitive
- [ ] Baggage MUST be fully functional without SDK installed
- [ ] Baggage container MUST be immutable
- [ ] GetValue() MUST take name as input; return value or null
- [ ] GetAllValues() order MUST NOT be significant
- [ ] SetValue() MUST take name+value; return new Baggage; new value MUST take precedence on conflict
- [ ] RemoveValue() MUST take name; return new Baggage without that entry
- [ ] MUST provide FromContext(), ContextWithBaggage(), ContextWithoutBaggage()
- [ ] API MUST provide way to remove all baggage entries from context
- [ ] API layer MUST include TextMapPropagator implementing W3C Baggage Specification

### A.12 Configuration
[Source: configuration/sdk-environment-variables.md, configuration/sdk.md]
- [ ] SDK MUST interpret empty value of env var same as unset
- [ ] Boolean: MUST be true only by case-insensitive "true"; any other value MUST be interpreted as false
- [ ] Enum values: unrecognized MUST generate warning and be gracefully ignored
- [ ] Numeric values: if invalid, SHOULD generate warning and ignore
- [ ] Environment-based configuration MUST have direct code configuration equivalent
- [ ] OTEL_PROPAGATORS values MUST be deduplicated
- [ ] OTEL_TRACES_SAMPLER_ARG: invalid input MUST be logged; MUST be ignored on invalid
- [ ] When OTEL_CONFIG_FILE set, all other env vars (except substitution refs) MUST be ignored
- [ ] Config file Parse operation MUST parse and validate configuration file
- [ ] Config file Create operation MUST create SDK from parsed configuration
- [ ] SDKs MUST support registering custom plugin component providers
- [ ] PluginComponentProvider.Create() MUST interpret configuration to create SDK plugin component
- [ ] If Create() fails, SHOULD return error
