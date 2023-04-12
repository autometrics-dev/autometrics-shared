![GitHub_headerImage](https://user-images.githubusercontent.com/3262610/221191767-73b8a8d9-9f8b-440e-8ab6-75cb3c82f2bc.png)

[![Discord Shield](https://discordapp.com/api/guilds/950489382626951178/widget.png?style=shield)](https://discord.gg/kHtwcH8As9)

This repo contains resources shared between all of the [Autometrics](https://github.com/autometrics-dev) implementations.

## Prometheus Recording & Alerting Rules

The [`autometrics.rules.yml`](./autometrics.rules.yml) file contains the default recording and alerting rules for Prometheus.

**This should work for most autometrics-instrumented projects without modification.**

Specifically, these rules will work for any project that uses the following objective percentils: 90%, 95%, 99%, 99.9%.

This file sets up a number of recording and alerting rules that are dormant by default and are only enabled when the autometrics libraries product metrics with special labels: `function_calls_count{objective_name="", objective_percentile=""}` or `function_calls_duration_bucket{objective_name="", objective_latency_threshold="", objective_percentile=""}`.

To read more details about the label tricks we use to make these rules work across autometrics-instrumented projects, see [An adventure with SLOs, generic Prometheus alerting rules, and complex PromQL queries](https://fiberplane.com/blog/an-adventure-with-slos-generic-prometheus-alerting-rules-and-complex-promql-queries).

### Re-generating the alerting rules file

The file is generated using the [autometrics-cli](https://github.com/autometrics-dev/autometrics-rs/tree/main/autometrics-cli) and the [Sloth](https://sloth.dev) tool.

To regenerate the file, you'll need:
- Rust
- Docker

Then run:

```shell
git clone https://github.com/autometrics-dev/autometrics-rs.git`
cd autometrics-rs
cargo run -p autometrics-cli generate-sloth-file > sloth.yml
docker run -v $(pwd):/data  ghcr.io/slok/sloth generate -i /data/sloth.yml -o /data/autometrics.rules.yml
```



