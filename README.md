# Prometheus-Cadvisor-Grafana

[![License](https://img.shields.io/github/license/iskandaryansergey/Prometheus-Cadvisor-Grafana)](LICENSE)

This repository contains a complete solution for Prometheus Cadvisor Grafana using Docker compose.

It allows you to expose your docker's engine metrics and host's metrics using Cadvisor. Grafana comes with comprehensive visualization and a prebuilt powerful dashboard, while Prometheus will using  as a metrics datasource.

Based on the official Docker images from Grafana, Prometheus, Cadvisor

[Grafana](https://github.com/grafana/grafana)
[Prometheus](https://github.com/prometheus/prometheus)
[Cadvisor](https://github.com/google/cadvisor)

## Philosophy

This project is built for those who want to monitor Docker's host and engine through Grafana dashboards. There is only one preconfigured dashboard, but you have the ability to create any dashboard that you need for your purposes. Metrics are exposed by the Cadvisor container it is the only container with root privileges because it should access your system paths due to collecting metrics via [Cgroup](https://man7.org/linux/man-pages/man7/cgroups.7.html) and [systemd slice](https://www.freedesktop.org/software/systemd/man/systemd.slice.html). The main advantage of the project is that you will have all those services up and run with docker compose one-line command.

## Requirements

### Host setup

* [Docker Engine](https://docs.docker.com/get-docker/)version **23.0.1** or newer
* [Docker Compose](https://docs.docker.com/compose/install/)version **2.16.0** or newer (including [Compose V2](https://docs.docker.com/compose/compose-v2/))
* 1.5 GB of RAM
* OS should be Linux's kernel based (Arch, Ubuntu, RedHat)
* Ensure that your user is able to work with docker daemon and interact with docker sock

> **Note**  
> Especially on Linux, make sure your user has the [required permissions](https://docs.docker.com/engine/install/linux-postinstall/) to interact with the Docker
> daemon. 

By default, the stack exposes the following ports:

* 9090:Prometheus
* 8080:Cadvisor
* 3000:Grafana

## Usage

### Bringing up the stack

Clone this repository onto the Docker host that will run the stack with the command below:

```sh
git clone https://github.com/iskandaryansergey/Prometheus-Cadvisor-Grafana.git
```

Start stack components:

```sh
docker-compose up -p gpc
```
![Animated demo](https://github.com/iskandaryansergey/Prometheus-Cadvisor-Grafana/blob/main/gif/up.gif)

> **Note**  
> You can also run all services in the background (detached mode) by appending the `-d` flag to the above command.

Give each service about a minute to initialize(depending on your system parameters), then access the Grafana web UI by opening <http://localhost:3000> in a web
browser

### Cleanup

Grafana and Prometheus data is persisted inside a volume by default.

In order to entirely shutdown the stack and remove all persisted data, use the following Docker Compose command:

```sh
docker-compose -p gpc down -v
```
![Animated demo](https://github.com/iskandaryansergey/Prometheus-Cadvisor-Grafanablob/main/gif/remove.gif)


### How to configure Grafana

The Grafana datasources configuration is stored in [`grafana/provisioning/datasources/datasource.yml`](https://github.com/iskandaryansergey/Prometheus-Cadvisor-Grafana/blob/main/grafana/provisioning/datasources/datasource.yml).

The Grafana dashboards configuration is stored in [`grafana/provisioning/dashboards/`](https://github.com/iskandaryansergey/Prometheus-Cadvisor-Grafana/blob/main/grafana/provisioning/dashboards/).

You can also specify the admin  user password and login name by setting environment variables inside the Compose file:
``` yml
  environment:
    #grafana defaulr login pass vars
    - GF_SECURITY_ADMIN_USER=${ADMIN_USER:-admin}
    - GF_SECURITY_ADMIN_PASSWORD=${ADMIN_PASSWORD:-admin}
    - GF_USERS_ALLOW_SIGN_UP=false
```

Grafana is preconfigured with dashboards and Prometheus as the default data source:

* Name: Prometheus
* Type: Prometheus
* Url: http://prometheus:9090
* Access: proxy

***Docker Containers Dashboard***

![Containers](image)

The Docker Containers Dashboard shows key metrics for monitoring running containers:

* Total containers CPU load, memory and storage usage
* Running containers graph, system load graph, IO usage graph
* Container CPU usage graph
* Container memory usage graph
* Container cached memory usage graph
* Container network inbound usage graph
* Container network outbound usage graph

### How to configure Prometheus

The Prometheus configuration is stored in [`prometheus/prometheus.yml`](https://github.com/iskandaryansergey/Prometheus-Cadvisor-Grafana/blob/main/prometheus/prometheus.yml).

In the mentioned configuration file you can specify scrape parameters.

## Healthcheck

Grafana  service has its own healthcheck, which you can change or update in the compose file. 

``` yml
  grafana:
    healthcheck:
      test: curl --silent --fail http://grafana:3000/api/health|| exit 1

```

The Cadvisor image comes with prebuilt healthchecks.

Also, there are dependencies for Kibana and Grafana services. They will start after the Cadvisor container has a healthy state. 
