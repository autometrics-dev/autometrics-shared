# Autometrics specification v1.0.0 <!-- omit in toc -->

This is a specification for Autometrics. Its high level goal is to enable
operability between the various different Autometrics libraries and consumers.

- [API](#api)
  - [Service-Level Objectives (SLOs)](#service-level-objectives-slos)
- [Conventions](#conventions)
- [Metrics](#metrics)
  - [`function.calls`](#functioncalls)
  - [`function.calls.duration`](#functioncallsduration)
  - [`build_info`](#build_info)
  - [`function.calls.concurrent`](#functioncallsconcurrent)
- [Labels](#labels)
  - [`autometrics.version`](#autometricsversion)
  - [`branch`](#branch)
  - [`caller.function`](#callerfunction)
  - [`caller.module`](#callermodule)
  - [`commit`](#commit)
  - [`function`](#function)
  - [`module`](#module)
  - [`objective.name`](#objectivename)
  - [`objective.percentile`](#objectivepercentile)
  - [`objective.latency_threshold`](#objectivelatency_threshold)
  - [`result`](#result)
  - [`service.name`](#servicename)
  - [`version`](#version)
- [Changelog](#changelog)
  - [v1.0.0](#v100)

## API

Libraries SHOULD expose a decorator, macro, wrapper function, or use another
meta-programming technique offered by the language to instrument functions and
methods in the user's source code. Ideally, the function attribute SHOULD be
called `autometrics` or `Autometrics`, but libraries MAY append a suffix to the
name if necessary.

Libraries MAY enable the decorator, macro, etc to apply to an entire class
definition. If they do, they SHOULD provide an option for users to skip or
ignore particular methods.

Libraries MAY need an initialization function.

Libraries SHOULD expose functionality to make the metrics available to the
user in either the Prometheus text format or the OpenTelemetry Protocol (otlp).
It is up to the user to decide whether they want to implement a push or pull
based model. Libraries MAY provide higher level implementation to expose the
metrics in a push or pull based model.

### Service-Level Objectives (SLOs)

Libraries SHOULD expose functionality to create objectives within the source
code. Objectives can be "attached" to functions by passing the objective to the
Autometrics decorator, macro, etc for one or more functions.

Objectives can relate to functions' success rate and/or latencies.

Success rate objectives MUST add the [`objective.name`](#objectivename) and
[`objective.percentile`](#objectivepercentile) labels to the
[`function.calls`](#functioncalls) metric.

Latency objectives MUST add the [`objective.name`](#objectivename),
[`objective.percentile`](#objectivepercentile), and
[`objective.latency_threshold`](#objectivelatency_threshold) labels to the
[`function.calls.duration`](#functioncallsduration) metric.

## Conventions

Autometrics uses the [OpenTelemetry Metric Semantic Conventions](https://github.com/open-telemetry/opentelemetry-specification/blob/main/specification/metrics/semantic_conventions/README.md)
for naming metrics, including using `.`'s as separators.

When the metrics are exported to Prometheus, all dot (`.`) separators in the
metric name and label keys are replaced by underscores (`_`). Suffixes are
appended where required by Prometheus/OpenMetrics.

Label values MAY contain any Unicode characters.

## Metrics

The following is a list of metrics that MUST be exported when annotated by an
autometrics library. The only exception is the `function.calls.concurrent`
metrics, which a library MAY support.

### `function.calls`

This metric is a 64-bit monotonic counter that tracks the number of times a
given function was invoked. It stores information about the result of the
function, and optionally which function called the given function.

**Metric type:** Counter

**OTEL metric name:** `function.calls`

**Prometheus metric name:** `function_calls_total`

| Label | Optional/required | Notes |
|-------|-------------------|-------|
| [`function`](#function) | required | |
| [`module`](#module) | required | |
| [`service.name`](#servicename) | required | |
| [`result`](#result) | required | |
| [`caller.function`](#callerfunction) | optional | |
| [`caller.module`](#callermodule) | optional | |
| [`objective.name`](#objectivename) | optional | if an [objective](#service-level-objectives-slos) is attached to the given function |
| [`objective.percentile`](#objectivepercentile) | optional | if an [objective](#service-level-objectives-slos) is attached to the given function |

If the library is unable to determine whether a function was successful or not,
then it MUST always set the result to `ok`.

If the library is able to determine which function called the given function,
then it MUST set the caller information in the `caller.function` and
`caller.module` labels. If the caller is not known, either it is the entrypoint
or the library is unable to determine the caller, then the `caller.function` and
`caller.module` labels MAY be absent or empty (`""`).

### `function.calls.duration`

This metric is a 64-bit floating point histogram that tracks the duration or
latency of function calls.

**Metric type:** Histogram

**OTEL metric name:** `function.calls.duration`

**Prometheus Name:** `function_calls_duration_seconds`

| Label | Optional/required | Notes |
|-------|-------------------|-------|
| [`function`](#function) | required | |
| [`module`](#module) | required | |
| [`service.name`](#servicename) | required | |
| [`objective.name`](#objectivename) | optional | if an [objective](#service-level-objectives-slos) is attached to the given function |
| [`objective.percentile`](#objectivepercentile) | optional | if an [objective](#service-level-objectives-slos) is attached to the given function |

A library MUST track the duration in seconds.

Libraries SHOULD support the following bucket boundaries by default: [ 0.005,
0.01, 0.025, 0.05, 0.075, 0.1, 0.25, 0.5, 0.75, 1.0, 2.5, 5.0, 7.5, 10.0, ]. The
Library SHOULD allow the user to override these default buckets.

### `build_info`

A metrics that contains metadata related to the application. It should be
possible to join this metric with other metrics to enrich other metrics.

**Metric type:** Gauge

**OTEL metric name:** `build_info`

**Prometheus Name:** `build_info`

| Label | Optional/required | Notes |
|-------|-------------------|-------|
| [`version`](#version) | optional | |
| [`commit`](#commit) | optional | |
| [`branch`](#branch) | optional | |
| [`service.name`](#servicename) | required | |
| [`autometrics.version`](#autometricsversion) | required | |

This is a gauge or up/down counter. It MUST always have the value of `1.0`.

### `function.calls.concurrent`

An optional metric that tracks the number of concurrent calls to a given
function.

**Metric type:** Gauge

**OTEL metric name:** `function.calls.concurrent`

**Prometheus Name:** `function_calls_concurrent`

| Label | Optional/required | Notes |
|-------|-------------------|-------|
| [`function`](#function) | required | |
| [`module`](#module) | required | |
| [`service.name`](#servicename) | required | |

## Labels

The following is a list of labels and their indented use. See the
[metrics](#metrics) section for which labels are valid on each metric.

### `autometrics.version`

The version of the specification that the library targets. This version MUST
contain the full version without the `v` prefix.

Consumers MAY only use the major and minor version, as the patch version changes
SHOULD be made in a backwards compatible way.

For the current specification this SHOULD be `1.0.0`.

### `branch`

The Git branch of the user's project. If this information is not available, this
label MAY be absent or empty (`""`).

### `caller.function`

The name of the `function` that invoked the given function. If the caller is
not known, this label MUST be absent or empty (`""`).

This SHOULD refer to Autometrics-instrumented functions. Therefore, if Function
A calls Function B, which calls Function C and only Functions A and C are
instrumented but not B, the `caller` of Function C would be Function A.

Libraries MAY make this label optional (on an opt-out basis) if collecting this
information has a non-negligible performance overhead.

### `caller.module`

The `module` of the function that invoked the given function. If the caller is
not known, this label MUST be absent or empty (`""`).

See [`caller.function`](#callerfunction)

### `commit`

The Git commit hash identifying the snapshot of the user's project. The library
MAY truncate the commit hash to its short representation. If this information is
not available, this label MUST be absent or empty (`""`).

### `function`

The name of the function or method, exactly as it appears in the source code.

### `module`

The fully-qualified module or file path of the `function`.

The combination of the `function` and `module` labels MUST be sufficient to
uniquely identify the function within the project's source code.

The exact contents of this label value are assumed to be language-specific.

### `objective.name`

If a function has an [SLO](#service-level-objectives-slos) attached, this label
MUST contain the user-specified name of the objective. If there is no SLO
attached, this label MUST be absent or empty (`""`).

### `objective.percentile`

If a function has an [SLO](#service-level-objectives-slos) attached, this label
MUST specifies the percentage of requests that should return the `result="ok"`
OR the percentage of requests that should meet the specified
[`objective.latency_threshold`](#objectivelatency_threshold).

The value MUST be expressed as a percentage, so 99.9% would be `"99.9"`
(without the `%` symbol).

If there is no SLO attached, this label MUST be absent or empty (`""`).

Libraries SHOULD support the following percentiles: `"90"`, `"95"`, `"99"`,
`"99.9"`. Libraries MAY allow users to specify custom percentiles but care
should be taken to ensure that users generate separate Prometheus recording
rules for the custom percentiles.

### `objective.latency_threshold`

If a function has an [SLO](#service-level-objectives-slos) attached, this MUST
specify the maximum duration of function calls that are considered meeting
the objective.

This MUST be specified in seconds (**not** milliseconds).

Libraries SHOULD support the following bucket boundaries by default: [ 0.005,
0.01, 0.025, 0.05, 0.075, 0.1, 0.25, 0.5, 0.75, 1.0, 2.5, 5.0, 7.5, 10.0, ].
Libraries MAY allow users to specify custom latencies, but care
should be taken to ensure that the value of this label matches one of the
histogram buckets supported by the [`function.calls.duration`](#functioncallsduration)
metric.

### `result`

Whether the function executed successfully or errored. An error MAY either mean
that the function returned an error or that it threw an exception.

The value of this label MUST either be `"ok"` or `"error"`.

Libraries MAY offer users the ability to override the default behavior for
determining whether the `result` label should be `"ok"` or `"error"`, for
example to allow users to treat client-side errors as `"ok"`.

### `service.name`

The logical name of a service. This matches the
[OpenTelemetry Service specification](https://github.com/open-telemetry/semantic-conventions/tree/main/specification/resource/semantic_conventions#service).

All metrics produced by a library from a given instance SHOULD use a single
`service.name`. All instances of a horizontally scaled service SHOULD also use
the same `service.name`.

Libraries SHOULD support setting the `service.name` using environment variables
(`AUTOMETRICS_SERVICE_NAME` and `OTEL_SERVICE_NAME`, with the first taking
precedence if both are set). Libraries MAY also support configuring this value
in an initialization function. If the service name is not set by the user, then
the library MUST set a default based on the user's project, this could be
something like a package name or the binary name.

### `version`

The version of the user's project, ideally using
[Semantic Versioning](https://semver.org/). It SHOULD only contain the version
number and SHOULD NOT start with a `v`.

## Changelog

### v1.0.0

- Rewrite of the original specification
- Add module to caller information
  - Add `caller.module` label to `function.calls` metric
  - Rename `caller` label to `caller.function` in `function.calls` metric
