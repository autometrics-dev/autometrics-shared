# OpenTelemetry (OTel) Collector deployment ressources

The detailed guides for the different platforms can be found [here](https://docs.autometrics.dev/deploying-otel-collector)

## Using Docker with OTel Collector for:
- Local setup
- Northflank
- Railway 

The OTel Collector provides a [Docker image](https://hub.docker.com/r/prom/prometheus) for easy deployment. To customize it, we use a **Dockerfile** with the image and a **otel-collector-config.yml** file for configuration.
