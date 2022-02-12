# Purpose
- Allow local experimentation / debugging of Grafana visuals and prom metrics
- Simple Dashboard sharing with Grafana json models

## Pre-requisites
- Docker for Windows installed https://docs.docker.com/docker-for-windows/install/
- python3 is available on machine (only to run demo server)

# Prom / Grafana setup
To start Prometheus / Grafana
```
docker-compose up -d
```
## Expected results after command completion
- Grafa available via http://127.0.0.1:3000
- Prom Server available via http://127.0.0.1:9090

# docker-compose / prom config explanation

## docker-compose
1. mounts ./prometheus/ folder into container
```
prometheus:
  ...
  volumes:
    - ./prometheus:/etc/prometheus
```

2. Bounds 9090 container port to 9999 local port
```
ports:
    - "9999:9090"
```

3. Sets 200h data retention for Prom db
```
prometheus:
  ...
  command:
    - '--storage.tsdb.retention.time=200h'
```

4. Eliminates a need to manually setup Prometheus as a datasource
```
grafana:
  ...
  volumes:
    - ./grafana/datasource.yaml:/etc/grafana/provisioning/datasources/default.yaml
```

## prometheus.yml
1. Sets an app running localy via 5000 port as prom target with scrape interval of 10s
```
- job_name: 'demo app'
  scrape_interval: 10s
  static_configs:
    - targets: ['host.docker.internal:5000']
```

## Dynamic prom config reload
```--web.enable-lifecycle``` start option allows dynamic promethus config reload, to reload config

- edit prometheus/prometheus.yml (specify new target, for example)
- execute ```curl -X POST http://localhost:9090/-/reload```
- see results via http://127.0.0.1:9090/config / http://127.0.0.1:9090/targets

## Tips and tricks
To stop containers
```
docker-compose down
```

To remove all images / networks / etc
```
docker system prune -a
```

To remove all volumes (completely erase docker data)
```
docker system prune -a --volumes
```

TODO: Investigate cadvisor

https://techwithcloud.com/2021/06/02/rabbitmq-monitoring-using-prometheus-grafana/