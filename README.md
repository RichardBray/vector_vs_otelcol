# Vector vs OpenTelemetry Collector

## Overview

This project is a containerized setup for tracing and monitoring using OpenTelemetry, Vector, Jaeger, and Prometheus. It is designed to collect, process, and visualize both trace and metric data from applications, providing insights into application performance and behavior.

## Components

- **Vector**: A tool for building observability pipelines. It collects trace and metric data and forwards it to Jaeger and Prometheus.
- **Jaeger**: An open-source, end-to-end distributed tracing system. It is used for monitoring and troubleshooting microservices-based distributed systems.
- **Prometheus**: An open-source systems monitoring and alerting toolkit. It is used for collecting and storing metrics data.

## Prerequisites

- Docker and Docker Compose installed on your machine.

## Setup Instructions

1. **Clone the Repository**

   Clone the repository to your local machine:

   ```bash
   git clone <repository-url>
   cd <repository-directory>
   ```

2. **Configure Vector for Traces**

   The Vector configuration for traces is located in `traces/vector.yaml`. This file defines the sources, transforms, and sinks for trace data.

   ```yaml
   sources:
     traces:
       type: opentelemetry
       grpc:
         address: 0.0.0.0:4317
       http:
         address: 0.0.0.0:4318

   transforms:
     format_traces:
       type: "remap"
       inputs: ["traces.traces"]
       source: |
         . = {"spans": .}

   sinks:
     jaeger:
       type: "http"
       inputs: ["format_traces"]
       uri: "http://jaeger:14268/api/traces"
       method: "post"
       encoding:
         codec: "json"
       request:
         headers:
           "Content-Type": "application/json"
       compression: "none"
   ```

3. **Configure Vector for Metrics**

   The Vector configuration for metrics is located in `metrics/vector.yaml`. This file defines the sources and sinks for metric data.

   ```yaml
   sources:
     docker_stats:
       type: prometheus_scrape
       endpoints: ["http://host.docker.internal:9323/metrics"]

   sinks:
     prometheus:
       type: prometheus_exporter
       inputs: ["docker_stats"]
   ```

4. **Configure OpenTelemetry Collector (Optional)**

   If you wish to use the OpenTelemetry Collector, uncomment the relevant sections in both `traces/docker-compose.yml` and `metrics/docker-compose.yml`, and ensure the configuration files `traces/otel-collector-config.yaml` and `metrics/otel-collector-config.yaml` are correctly set up.

5. **Start the Services**

   Use Docker Compose to start the services for both traces and metrics:

   ```bash
   docker-compose -f traces/docker-compose.yml up -d
   docker-compose -f metrics/docker-compose.yml up -d
   ```

   This command will start the Vector, Jaeger, and Prometheus services. If you have enabled the OpenTelemetry Collector, it will start as well.

6. **Access Jaeger and Prometheus UIs**

   - **Jaeger UI**: Open your web browser and navigate to `http://localhost:16686` to visualize trace data.
   - **Prometheus UI**: Open your web browser and navigate to `http://localhost:9090` to view and query metrics data.

## Usage

- **Sending Trace Data**: Configure your application to send trace data to the Vector service running on port `4317`.
- **Sending Metrics Data**: Ensure your application exposes metrics in a format that Prometheus can scrape.
- **Viewing Traces**: Use the Jaeger UI to view and analyze the trace data collected from your application.
- **Viewing Metrics**: Use the Prometheus UI to view and query the metrics data collected from your application.

## Troubleshooting

- Ensure that the ports specified in the `docker-compose.yml` files are not being used by other services on your machine.
- Check the logs of each service for any errors or warnings:

  ```bash
  docker-compose -f traces/docker-compose.yml logs
  docker-compose -f metrics/docker-compose.yml logs
  ```

## Contributing

Contributions are welcome! Please submit a pull request or open an issue for any improvements or bug fixes.

## License

This project is licensed under the MIT License. See the `LICENSE` file for more details.j