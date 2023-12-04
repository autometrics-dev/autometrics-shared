FROM prom/prometheus

USER root
RUN mkdir -p /etc/config
USER nobody

ADD prometheus.yml /etc/prometheus/
ADD autometrics-rules.yml /etc/config/

# Options:
# Enable the remote write endpoint, if you want to push metrics via the Otel-collector
# Change log level if you need to debug
# CMD ["--web.enable-remote-write-receiver", "--log.level=debug"]