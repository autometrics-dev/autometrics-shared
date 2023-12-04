# Prometheus deployment ressources

The detailed guides for the different platforms can be found [here](https://docs.autometrics.dev/deploying-prometheus)

## Using Docker with Prometheus for:
- Local setup
- Northflank
- Railway 

Prometheus provides a [Docker image](https://hub.docker.com/r/prom/prometheus) for easy deployment. To customize it, we use a **Dockerfile** with the image and a **prometheus.yml** file for configuration. In our configuration, we refer to our **autometrics-rules.yml**. 

### Platform-Specific Networking:
The main difference comes in setting up networking on these platforms. We've detailed the steps in [our guides](https://docs.autometrics.dev/deploying-prometheus), so check them out for specific instructions tailored to each cloud platform.