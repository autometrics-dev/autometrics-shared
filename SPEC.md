# Autometrics Specification <!-- omit in toc -->

This is a work in progress specification for Autometrics.

It aims to describe the full feature set of the Autometrics libraries, but it may have important details missing. We will attempt to update this document to describe the expectations across all of the language implementations.

- [API](#api)
  - [Service-Level Objectives (SLOs)](#service-level-objectives-slos)
- [Metric Collection Libraries](#metric-collection-libraries)
  - [Exemplars (Optional)](#exemplars-optional)
- [Metrics](#metrics)
  - [`function.calls.count`](#functioncallscount)
  - [`function.calls.duration`](#functioncallsduration)
  - [`build_info`](#build_info)
  - [`function.calls.concurrent`](#functioncallsconcurrent)
- [Labels](#labels)
  - [`branch`](#branch)
  - [`caller`](#caller)
  - [`commit`](#commit)
  - [`function`](#function)
  - [`module`](#module)
  - [`objective.name`](#objectivename)
  - [`objective.percentile`](#objectivepercentile)
  - [`objective.latency_threshold`](#objectivelatency_threshold)
  - [`result`](#result)
  - [`version`](#version)


## API

Libraries SHOULD expose a decorator, macro, wrapper function, or use another metaprogramming technique offered by the language to instrument functions and methods in the user's source code. Ideally, the function attribute should simply be called `autometrics` or `Autometrics`, but libraries MAY append a suffix to the name if necessary.

Libraries MAY enable the decorator, macro, etc to apply to an entire class definition. If they do, they SHOULD provide an option for users to skip or ignore particular methods.

Libraries MAY need an initialization function.

Libraries MAY expose additional functionality for exporting metrics to Prometheus and/or other metrics collection servers. This MAY include serializing the metrics to the Prometheus text format, OpenMetrics export format, the OpenTelemetry Protocol and/or exposing the metrics on a specific port and HTTP path to be scraped.

**Note:** there is an [open discussion](https://github.com/orgs/autometrics-dev/discussions/7) about whether libraries should export metrics on a default port and path. There is another [open discussion](https://github.com/orgs/autometrics-dev/discussions/34) about support for pushing metrics to a collector.

### Service-Level Objectives (SLOs)

Libraries SHOULD expose functionality to create objectives within the source code. Objectives can be "attached" to functions by passing the objective to the Autometrics decorator, macro, etc for one or more functions.

Objectives can relate to functions' success rate and/or latencies.

Success rate objectives add the [`objective.name`](#objectivename) and [`objective.percentile`](#objectivepercentile) labels to the [`function.calls.count`](#functioncallscount) metric.

Latency objectives add the [`objective.name`](#objectivename), [`objective.percentile`](#objectivepercentile), and [`objective.latency_threshold`](#objectivelatency_threshold) labels to the [`function.calls.duration`](#functioncallsduration) metric.

## Metric Collection Libraries

Libraries MUST support producing metrics using an OpenTelemetry library. Libraries MAY also support Prometheus client libraries and allow users to configure which one should be used to produce metrics.

Libraries MUST support exporting metrics to Prometheus, or provide documentation for how users can export the metrics from the OpenTelemetry format to the Prometheus exposition format.

### Exemplars (Optional)

Autometrics libraries MAY support attaching exemplars to the metrics generated if the underlying metrics library or libraries they use support them. See [Grafana's explainer](https://grafana.com/docs/grafana/latest/fundamentals/exemplars/), the [OpenMetrics Spec](https://github.com/OpenObservability/OpenMetrics/blob/main/specification/OpenMetrics.md#exemplars), and the [OpenTelemetry Spec](https://github.com/open-telemetry/opentelemetry-specification/blob/main/specification/metrics/data-model.md#exemplars) for more details about exemplars.

Libraries that support exemplars SHOULD integrate with popular tracing libraries and/or the OpenTelemetry library to extract exemplar fields from the context or span a given function is called within.

Libraries SHOULD support extracting the `trace_id` field and attaching it as an exemplar label or attribute. Libraries MAY support extracting other fields automatically or provide the user functionality to customize which fields are used.

## Metrics

Autometrics uses the [OpenTelemetry Metric Semantic Conventions](https://github.com/open-telemetry/opentelemetry-specification/blob/main/specification/metrics/semantic_conventions/README.md) for naming metrics, including using `.`'s as separators.

When the metrics are exported to Prometheus, all dot (`.`) separators are replaced by underscores (`_`).

### `function.calls.count`

> **Required Labels:** [`function`](#function), [`module`](#module), [`result`](#result), [`caller`](#caller)
>
> **Additional Labels** (if a success rate [objective](#service-level-objectives-slos) is attached to the given function): [`objective.name`](#objectivename) and [`objective.percentile`](#objectivepercentile)

**Note:** there is an [open discussion](https://github.com/orgs/autometrics-dev/discussions/4#discussioncomment-5839198) about changing this metric name to `function.calls` or `function.calls.total`.

This metric is a 64-bit monotonic counter that tracks the number of times a given function was invoked.

### `function.calls.duration`

> **Required Labels:** [`function`](#function), [`module`](#module)
>
> **Additional labels** (if a latency [objective](#service-level-objectives-slos) is attached to the given function): [`objective.name`](#objectivename), [`objective.percentile`](#objectivepercentile), [`objective.latency_threshold`](#objectivelatency_threshold)

This is a 64-bit floating point histogram that tracks the duration or latency of function calls.

It MUST track the duration in seconds (**not** milliseconds).

Libraries SHOULD support the [default OpenTelemetry histogram buckets](https://opentelemetry.io/docs/reference/specification/metrics/sdk/#histogram-aggregations) as label values. Libraries MAY allow users to specify custom histogram buckets.

### `build_info`

> **Required Labels:** [`version`](#version), [`commit`](#commit), [`branch`](#branch)

This is a gauge or up/down counter.

It MUST always have the value of `1.0`.

### `function.calls.concurrent`

> **Required Labels:** [`function`](#function), [`module`](#module)

This metric is optional. Libraries MAY provide an option to the user for enabling this on a per-function basis.

This is a gauge or "up/down counter" used for tracking concurrent calls to the specific function. When the function is initially called, the gauge is incremented by 1 and when it finishes, the value is decremented by 1.

## Labels

Label values MAY contain any Unicode characters.

See the [metrics](#metrics) for which labels are valid on each metric.

### `branch`

The Git branch of the user's project. If this information is not available, this label MAY be absent or empty (`""`).

### `caller`

**Note:** there is an [ongoing discussion](https://github.com/orgs/autometrics-dev/discussions/33) about whether this should be replaced with multiple labels such as `caller_function` and `caller_module`.

The name of the `function` that invoked the given function. If the caller is not known, this label MAY be absent or empty (`""`).

This SHOULD refer to Autometrics-instrumented functions. Therefore, if Function A calls Function B, which calls Function C and only Functions A and C are instrumented but not B, the `caller` of Function C would be Function A.

Libraries MAY make this label optional (on an opt-out basis) if collecting this information has a non-negligible performance overhead.

### `commit`

The short (8-byte) Git commit hash of the user's project. If this information is not available, this label MAY be absent or empty (`""`).

### `function`

The name of the function or method, exactly as it appears in the source code.

### `module`

The fully-qualified module or file path of the `function`. The combination of the `function` and `module` labels MUST be sufficient to uniquely identify the function within the project's source code. The exact contents of this label value are assumed to be language-specific.

**Note:** There is an [ongoing discussion](https://github.com/orgs/autometrics-dev/discussions/28) about whether the class should be added to the `module` label or if there should be a separate `class` label.

### `objective.name`

If a function has an [SLO](#service-level-objectives-slos) attached, this label contains the user-specified name of the objective. If there is no SLO attached, this label MAY be absent or empty (`""`).

### `objective.percentile`

If a function has an [SLO](#service-level-objectives-slos) attached, this label specifies the percentage of requests that should return the `result="ok"` OR the percentage of requests that should meet the specified [`objective.latency_threshold`](#objectivelatency_threshold).

The value MUST be expressed as a percentage, so 99.9% would be `"99.9"` (without the `%` symbol).

If there is no SLO attached, this label MAY be absent or empty (`""`).

Libraries SHOULD support the following percentiles: `"90"`, `"95"`, `"99"`, `"99.9"`. Libraries MAY allow users to specify custom percentiles but care should be taken to ensure that users generate separate Prometheus recording rules for the custom percentiles.

### `objective.latency_threshold`

If a function has an [SLO](#service-level-objectives-slos) attached, this specifies the maximum duration of function calls that are considered meeting the objective.

This MUST be specified in seconds (**not** milliseconds).

Libraries SHOULD support the [default OpenTelemetry histogram buckets](https://opentelemetry.io/docs/reference/specification/metrics/sdk/#histogram-aggregations) as label values. Libraries MAY allow users to specify custom latencies but care should be taken to ensure that the value of this label matches one of the histogram buckets supported by the [`function.calls.duration`](#functioncallsduration) metric.

### `result`

Whether the function executed successfully or errored. An error MAY either mean that the function returned an error or that it threw an exception.

The value of this label MUST either be `"ok"` or `"error"`.

Libraries MAY offer users the ability to override the default behavior for determining whether the `result` label should be `"ok"` or `"error"`, for example to allow users to treat client-side errors as `"ok"`.

### `version`

The version of the user's project, ideally using [Semantic Versioning](https://semver.org/). It SHOULD only contain the version number and SHOULD NOT start with a `v`.
