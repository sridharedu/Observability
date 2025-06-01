# Observability

# Observability Demo Project
This repository provides a **learning demo project** designed to teach developers how to implement comprehensive observability in a microservices architecture. The project's primary **goals** are to demonstrate **distributed tracing**, **structured logging**, **metrics collection**, and **monitoring** techniques. It utilizes minimal Spring Boot microservices with **no complex business logic**, focusing exclusively on the instrumentation required for observability, allowing any developer to clone it and immediately see these concepts in action.

---

## Table of Contents

1. [Introduction](#introduction)
2. [Prerequisites](#prerequisites)
3. [Project Structure Overview](#project-structure-overview)
4. [Setup Observability Tools](#setup-observability-tools)

   * [Tracing Server (Zipkin)](#tracing-server-zipkin)
   * [Logging Stack (ELK)](#logging-stack-elk)
   * [Metrics Stack (Prometheus + Grafana)](#metrics-stack-prometheus--grafana)
5. [Build & Run Microservices](#build--run-microservices)
6. [Demo Walkthrough](#demo-walkthrough)

   * [Generate Traces & Logs](#generate-traces--logs)
   * [View in Zipkin](#view-in-zipkin)
   * [View Logs in Kibana](#view-logs-in-kibana)
   * [View Metrics in Grafana](#view-metrics-in-grafana)
7. [Simulating a Latency Spike](#simulating-a-latency-spike)
8. [Troubleshooting Tips](#troubleshooting-tips)
9. [Comments Guide](#comments-guide)
10. [FAQ & Additional Notes](#faq--additional-notes)

---

## Introduction

This demo project illustrates key observability concepts using three minimal Spring Boot microservices: **User Service**, **Inventory Service**, and **Order Service**. These services are chained (Order → Inventory → User) to simulate a typical request flow. The **explicit learning focus** of this project is on:

* **Distributed Tracing** with Spring Cloud Sleuth and Zipkin
* **Centralized JSON Logging** to an ELK stack (Elasticsearch, Logstash, Kibana)
* **Metrics Exposure** via Micrometer for Prometheus, visualized in Grafana
* **Optional SaaS Integration** (e.g., Datadog) for pushing metrics or traces

By following the instructions below, you will be able to:

1. Start up a local tracing server (Zipkin).
2. Start a logging stack (Elasticsearch, Logstash, Kibana).
3. Start a metrics stack (Prometheus, Grafana).
4. Build and run the three Spring Boot services.
5. Send a few HTTP requests to generate traces, logs, and metrics, then view them in the respective UIs.

---

## Prerequisites

Before you begin, ensure you have the following tools installed:

* **Java 11 or higher** (for building and running Spring Boot services)
* **Maven** (or Gradle) to build the services
* **Docker & Docker Compose** (to spin up Zipkin, ELK, and Prometheus/Grafana)
* **cURL** or **Postman** (to send HTTP requests to the services)

Optionally, if you want to integrate with a SaaS monitoring solution:

* **Datadog API Key** (to push metrics/traces to Datadog)

---

## Project Structure Overview

While you do not need to inspect every folder to follow this README, here is a high-level overview:

* `user-service/`
* `inventory-service/`
* `order-service/`
* `docker/zipkin/`
* `docker/elk/`
* `docker/prometheus-grafana/`
* `README.md` (this file)
* `COMMENTS_GUIDE.md` (explanations of observability concepts)

Each Spring Boot service lives in its own folder and can be built independently. The `docker` subfolders contain Docker Compose configurations for each observability stack.

---

## Setup Observability Tools

### Tracing Server (Zipkin)

1. Open a terminal and navigate to the directory containing the Zipkin Compose file:

   ```bash
   cd docker/zipkin
   ```
2. Start Zipkin:

   ```bash
   docker-compose up -d
   ```
3. Verify that Zipkin is running by visiting:

   ```
   http://localhost:9411
   ```

   You should see the Zipkin UI, where traces will appear once the services send spans.

### Logging Stack (ELK)

1. In a new terminal, navigate to the ELK Compose folder:

   ```bash
   cd docker/elk
   ```
2. Start Elasticsearch, Logstash, and Kibana:

   ```bash
   docker-compose up -d
   ```
3. Wait a minute for all containers to initialize.
4. Access Kibana at:

   ```
   http://localhost:5601
   ```
5. In Kibana, create an index pattern for logs (e.g., `springboot-logs-*`).
6. You will later see JSON logs from all services indexed here.

### Metrics Stack (Prometheus + Grafana)

1. In another terminal, go to the Prometheus & Grafana folder:

   ```bash
   cd docker/prometheus-grafana
   ```
2. Start Prometheus and Grafana:

   ```bash
   docker-compose up -d
   ```
3. Verify Prometheus at:

   ```
   http://localhost:9090
   ```
4. Verify Grafana at:

   ```
   http://localhost:3000
   ```

   * Default login: **admin / admin**
   * After logging in, add Prometheus as a data source pointing to `http://prometheus:9090` (or `http://localhost:9090` if running locally).

---

## Build & Run Microservices

Each of the three services follows the same general build and run steps.

1. **Build the Service**
   Open a terminal, navigate to the service folder, and run:

   ```bash
   cd <service-folder>
   mvn clean package
   ```

   Replace `<service-folder>` with `user-service`, `inventory-service`, or `order-service`.

2. **Run the Service**
   Still in the same folder, run the packaged JAR:

   ```bash
   java -jar target/*.jar
   ```

   * **User Service** will run on port **8083** (by default).
   * **Inventory Service** will run on port **8082**.
   * **Order Service** will run on port **8081**.

   Each service’s console logs will show that it has started, and you’ll see information about Actuator endpoints.

3. **Verify Actuator Endpoints**
   You can verify that tracing, logging, and metrics endpoints are exposed:

   * Health: `http://localhost:<port>/actuator/health`
   * Metrics (Prometheus): `http://localhost:<port>/actuator/prometheus`
   * HTTP Traces: `http://localhost:<port>/actuator/httptrace` (if enabled)
   * Loggers: `http://localhost:<port>/actuator/loggers`

Repeat the build and run steps for all three services.

---

## Demo Walkthrough

Once Zipkin, ELK, Prometheus/Grafana, and the three services are all running, you can observe the full observability pipeline in action.

### Generate Traces & Logs

1. Send a request to the first service (Order Service):

   ```bash
   curl http://localhost:8081/api/order/123
   ```
2. This call will internally trigger a downstream call to Inventory Service, which then calls User Service.
3. As you make these requests, each service will:

   * Automatically add a **trace ID** and **span ID** to logs (via Spring Cloud Sleuth).
   * Write JSON-formatted logs (containing timestamp, level, thread, logger, trace ID, span ID, message).
   * Expose metrics to Prometheus on the `/actuator/prometheus` endpoint.
   * Send trace spans to Zipkin.

You can repeat the `curl` command multiple times to generate more data.

### View in Zipkin

1. In your browser, go to:

   ```
   http://localhost:9411
   ```
2. Click on **“Traces”**. You should see a list of traces, each representing one request to the Order Service.
3. Click on a trace to view:

   * Three spans (Order → Inventory → User).
   * Parent-child relationships.
   * Timing information for each span, so you can identify latency.

> **Tip:** One of the services (e.g., Inventory) demonstrates a **custom child span** for its downstream call. Look for that extra annotation in the span list.

### View Logs in Kibana

1. Open Kibana at:

   ```
   http://localhost:5601
   ```
2. In **Discover**, select the index pattern you created earlier (`springboot-logs-*`).
3. You should see all JSON logs streaming in from the three services. Each log entry will have fields such as:

   * `@timestamp`
   * `level` (e.g., INFO, WARN)
   * `thread`
   * `logger`
   * `traceId`
   * `spanId`
   * `message`
4. Search or filter by a specific **traceId** (copy it from Zipkin) to see all log entries associated with one request.
5. Create or view a sample **visualization** showing:

   * Counts of log levels (INFO vs. ERROR) over time.
   * A saved search for one trace ID, listing only ERROR-level logs if any.

### View Metrics in Grafana

1. Open Grafana at:

   ```
   http://localhost:3000
   ```

   * Login: **admin / admin**
   * You may be prompted to change the password on first login.
2. Add **Prometheus** as a data source:

   * URL: `http://prometheus:9090` (or `http://localhost:9090`)
3. Import or create a simple dashboard with panels like:

   * **Total HTTP Request Count**

     * Query example: `sum(rate(http_server_requests_seconds_count{application="order-service"}[1m]))`
     * Visualization: time-series chart
   * **95th-Percentile HTTP Latency**

     * Query example: `histogram_quantile(0.95, sum(rate(http_server_requests_seconds_bucket{application="inventory-service"}[5m])) by (le))`
     * Visualization: time-series chart
   * **JVM Memory Usage**

     * Query example: `jvm_memory_bytes_used{application="user-service", area="heap"}`
     * Visualization: time-series chart
   * **GC Pause Durations**

     * Query example: `jvm_gc_pause_seconds_sum{application="user-service"}`
     * Visualization: time-series chart

As you send requests, you should see these panels update in real time, reflecting increased request rates, latencies, and JVM metrics.

---

## Simulating a Latency Spike

To illustrate how observability helps pinpoint issues, we can add an artificial delay in one service:

1. **Edit Inventory Service** to insert a sleep before calling User Service:

   ```java
   // Pseudo-code:
   Thread.sleep(500); // 500ms delay to simulate latency
   ```

2. **Rebuild and Restart** Inventory Service:

   ```bash
   cd inventory-service
   mvn clean package
   java -jar target/*.jar
   ```

3. Send the same `curl` request:

   ```bash
   curl http://localhost:8081/api/order/123
   ```

4. **Observe in Zipkin**

   * The Inventory Service span will show \~500ms duration.
   * The full trace will highlight that span as the longest, making it easy to identify the bottleneck.

5. **Check Kibana Logs**

   * If you added a conditional WARN log (e.g., `if duration > 200ms, log.warn(...)`), search for WARN-level entries in the time window.
   * Filter by the same trace ID to see exactly which request incurred the delay.

6. **Check Grafana Metrics**

   * The latency panel for Inventory Service should spike around 500ms.
   * The request count panel shows increased response time.
   * If you track JVM metrics, you may notice corresponding memory or GC patterns.

---

## Troubleshooting Tips

* **Zipkin UI shows no data**

  * Verify Zipkin container is running (`docker ps`).
  * Check that each service’s configuration points to `http://localhost:9411/api` (or the correct Zipkin URL).
  * Ensure Sleuth dependencies are on the classpath.

* **Kibana doesn’t show logs**

  * Confirm Logstash is running and listening on port 5000 (`docker logs logstash`).
  * Check that each service’s logging configuration points to Logstash (TCP 5000).
  * In Kibana, ensure the index pattern `springboot-logs-*` exists and is selected.

* **Grafana panels show “No data”**

  * Verify Prometheus is running (`docker ps`) and reachable at `http://localhost:9090`.
  * Check Prometheus targets page (`http://localhost:9090/targets`) to ensure each service’s `/actuator/prometheus` endpoint is up.
  * Ensure Grafana’s data source is correctly configured to point at Prometheus.

* **Build failures**

  * Make sure you are using Java 11+ and Maven 3.x (or Gradle 5.x+).
  * If a service fails to start, check its console logs for missing dependencies or port conflicts.

---

## Comments Guide

A separate document (`COMMENTS_GUIDE.md`) explains key observability concepts and best practices. Below is a summary:

1. **Distributed Tracing**

   * **What it is:** Recording the path of a request across multiple services, capturing timing information in “spans.”
   * **Why use it:** Quickly identify which service or code block causes latency or errors.
   * **Core tools:** Spring Cloud Sleuth (auto-creation of trace/span IDs), Zipkin or Jaeger (span collection and UI).
   * **Best practices:**

     * Sample only a subset of requests in production (e.g., 10%) to reduce overhead.
     * Create custom spans around critical code sections (e.g., database calls) for deeper insight.

2. **Structured JSON Logging**

   * **What it is:** Emitting log entries as JSON objects with well-defined fields (timestamp, level, trace ID, span ID, etc.).
   * **Why use it:** Enables powerful searches, filtering, and correlation (e.g., find all logs for a given trace ID).
   * **Core tools:** Logstash (parses JSON logs), Elasticsearch (stores indexed logs), Kibana (visualizes and searches).
   * **Best practices:**

     * Include `traceId` and `spanId` in every log entry for easy correlation.
     * Log at appropriate levels: INFO for normal operations, WARN for potential issues, ERROR for failures.
     * Avoid expensive logging in high-throughput paths—use sampling if needed.

3. **Metrics & Monitoring**

   * **What it is:** Collecting time-series data (counters, gauges, timers) about application health and performance (e.g., request rates, latency, memory usage).
   * **Why use it:** Provides real-time visibility into system behavior and enables alerting when thresholds are crossed.
   * **Core tools:** Micrometer (exposes metrics from Spring Boot), Prometheus (scrapes and stores metrics), Grafana (dashboards).
   * **Best practices:**

     * Tag metrics with `application` or `service` names for easy filtering.
     * Track p95 or p99 latencies rather than averages to catch tail-end slowdowns.
     * Monitor JVM metrics (heap usage, GC pause time) alongside application metrics.

4. **Optional SaaS Integration (Datadog, New Relic, etc.)**

   * **Why use it:** Managed dashboards, built-in alerts, anomaly detection, and fewer infrastructure maintenance tasks.
   * **Trade-offs:** Cost (paid service), less control over data storage and retention, vendor lock-in.

5. **High-Level Best Practices**

   * **Sampling Rate:** In production, sample only a fraction of traces (e.g., 5–10%) to minimize overhead.
   * **Log Retention:** Implement index lifecycle policies in Elasticsearch to delete old logs and free disk space.
   * **Alert Thresholds:** Define thresholds for error rates or latency spikes (e.g., if p95 latency > 500ms for 5 minutes) and configure alerting rules in Grafana or Datadog.
   * **Security:** Do not expose actuator endpoints in production without authentication. Secure Zipkin and Kibana access as well.

---

## FAQ & Additional Notes

* **Why Zipkin vs. Jaeger?**
  Both are popular open-source tracing systems. Zipkin is straightforward to set up for demos; Jaeger offers deeper integration with Kubernetes and more advanced sampling features.

* **Can I skip ELK and use a simpler logging solution?**
  Yes, you could use a lightweight log aggregator (e.g., Fluentd + Elasticsearch) or even ship JSON logs directly to a cloud logging service. This demo uses ELK because it is widely understood and free to run locally.

* **Do I need all these tools for a small project?**
  For learning purposes, yes—seeing each layer in action helps understand how they fit together. In production, you might choose a subset or a managed SaaS solution.

* **Can I use OpenTelemetry instead of Sleuth + Zipkin?**
  Absolutely. OpenTelemetry is becoming the standard. This demo uses Sleuth + Zipkin for simplicity, but you can replace it with OpenTelemetry Collector and Jaeger if desired.

* **What if I only care about metrics and logs, not tracing?**
  You can choose to instrument only Micrometer and JSON logging, skipping Sleuth and Zipkin entirely. However, distributed tracing is invaluable for multi-service debugging.

---

You now have everything you need to build and run a fully instrumented Spring Boot observability demo. Clone the repo, follow these instructions, and watch traces, logs, and metrics come to life. Enjoy learning!




1. **Ticket 1: Define Project Scope and Objectives**
   **Description:** Document the goals of this demo project, emphasizing that it will teach distributed tracing, structured logging, metrics collection, and monitoring using minimal Spring Boot microservices. Clearly state that there is no complex business logic—only instrumentation.

2. **Ticket 2: Create Shared Dependency Configuration**
   **Description:** Set up a “common-config” file (Maven or Gradle) to centralize version numbers for Spring Boot, Spring Cloud Sleuth, Micrometer, Zipkin/Jaeger, ELK, Prometheus, and Grafana. Explain why version alignment matters for compatibility across services.

3. **Ticket 3: Initialize Microservices Skeleton**
   **Description:** Generate three simple Spring Boot projects—User Service, Inventory Service, and Order Service—each with `spring-boot-starter-web` and `spring-boot-starter-actuator`. This ticket covers all three service skeletons together, since the code structure is identical.

4. **Ticket 4: Implement Basic REST Endpoints**
   **Description:** In each service, create one REST controller:

   * **User Service:** `GET /api/user/{id}` returns a dummy user.
   * **Inventory Service:** `GET /api/inventory/{orderId}` calls User Service.
   * **Order Service:** `GET /api/order/{id}` calls Inventory Service.
     Include a note explaining that these endpoints exist solely to generate real call chains.

5. **Ticket 5: Add Distributed Tracing Dependencies**
   **Description:** Add `spring-cloud-starter-sleuth` and `spring-cloud-sleuth-zipkin` to each service. In comments, explain why Sleuth is needed for trace propagation and why Zipkin is chosen as the tracing collector.

6. **Ticket 6: Configure Sleuth Sampling and Zipkin Export**
   **Description:** In each service’s configuration, set `spring.sleuth.sampler.probability` to `1.0` (with a comment on adjusting sampling rates in production) and configure `spring.zipkin.base-url` to point at the local Zipkin server. Explain why these properties are critical for capturing and exporting spans.

7. **Ticket 7: Implement Custom Child Span in Inventory Service**
   **Description:** Modify the Inventory Service to explicitly create a child span around its call to User Service. Add comments explaining how custom spans differ from auto-generated spans and why you would isolate specific code blocks.

8. **Ticket 8: Set Up Zipkin Tracing Server**
   **Description:** Create instructions (for a Docker Compose file) to start a Zipkin container on port 9411. In comments, detail why Zipkin is used for visualizing spans and how developers will use its UI to inspect parent-child relationships.

9. **Ticket 9: Add JSON Logging Dependencies**
   **Description:** Add `logstash-logback-encoder` to all three services with comments explaining why structured (JSON) logs are preferred for centralized log aggregation.

10. **Ticket 10: Configure JSON Logging in Each Service**
    **Description:** Set up each service’s `logback-spring.xml` (or equivalent) to output JSON logs including `timestamp`, `level`, `thread`, `logger`, `traceId`, `spanId`, and `message`. Include plain-English comments for each field, describing how Logstash will parse them.

11. **Ticket 11: Set Up ELK Logging Stack**
    **Description:** Provide instructions (for Docker Compose) to spin up Elasticsearch, Logstash, and Kibana. In comments, explain how Logstash listens for JSON logs, indexes them into Elasticsearch, and how Kibana will visualize them.

12. **Ticket 12: Configure Logstash Pipeline**
    **Description:** Describe a Logstash configuration that listens on TCP port 5000 for JSON logs, parses the `timestamp` field, and sends logs to an index pattern like `springboot-logs-*`. Explain each filter and output block in plain English.

13. **Ticket 13: Add Micrometer Metrics Dependencies**
    **Description:** Add `micrometer-core` and `micrometer-registry-prometheus` to all services. Explain in comments why Micrometer is used to expose metrics in a Prometheus-compatible format.

14. **Ticket 14: Expose Prometheus Metrics Endpoint**
    **Description:** Configure each service to expose `/actuator/prometheus`. Include comments describing how Prometheus will scrape this endpoint and why enabling that URL is essential.

15. **Ticket 15: Set Up Prometheus Metrics Server**
    **Description:** Provide Docker Compose instructions to start a Prometheus container configured to scrape each service’s `/actuator/prometheus` endpoint. Comment on scrape interval, target definitions, and how to verify scrapes in the Prometheus UI.

16. **Ticket 16: Configure Grafana Dashboard**
    **Description:** Document how to start Grafana (via Docker Compose), add Prometheus as a data source, and create a basic dashboard showing:

    * Request count per service over time
    * 95th-percentile HTTP latency per service
    * JVM memory usage for one service
    * GC pause durations for one service
      Explain why these specific panels are useful for monitoring service health.

17. **Ticket 17: Optional Datadog Integration**
    **Description:** Show how to add `micrometer-registry-datadog` to each service and configure Datadog API keys. In comments, explain why you might choose Datadog (SaaS dashboards, alerts) over Prometheus/Grafana in certain scenarios.

18. **Ticket 18: Create README Section for Observability Setup**
    **Description:** Draft the README content that instructs how to start Zipkin, ELK, and Prometheus/Grafana in the correct order. Explain why the sequence matters and how each tool relates to the services.

19. **Ticket 19: Create README Section for Building and Running Services**
    **Description:** Draft the README steps showing how to build each Spring Boot service (`mvn clean package`) and run the JARs. Explain why Maven (or Gradle) is required and how Actuator endpoints verify instrumentation.

20. **Ticket 20: Document Demo Workflow in README**
    **Description:** Write the part of the README that guides a user to:

    1. Send a `curl` request to the Order Service
    2. Observe the trace in Zipkin (three spans and a custom child span)
    3. Observe JSON logs in Kibana filtered by `traceId`
    4. Observe metrics in Grafana reflecting increased request count and latency
       Ensure each step explains what the user is looking for in each UI.

21. **Ticket 21: Document How to Simulate a Latency Spike**
    **Description:** Instruct users how to add an artificial `Thread.sleep` (e.g., 500ms) in Inventory Service, rebuild, and rerun. Explain how to detect that delay in Zipkin, Kibana, and Grafana, with commentary on why this exercise illustrates observability value.

22. **Ticket 22: Create COMMENTS\_GUIDE Document**
    **Description:** Draft a separate `COMMENTS_GUIDE.md` explaining:

    * What distributed tracing is, why it’s important, and how Sleuth + Zipkin implement it
    * What structured JSON logging is, why ELK is used, and how Logstash parses logs
    * What metrics are, why Micrometer + Prometheus are used, and how Grafana visualizes them
    * When to use Datadog or similar SaaS solutions, including pros/cons
      Ensure all explanations are in plain English.

23. **Ticket 23: Insert Plain-English Comments in User Service Code**
    **Description:** Add comments to the User Service’s code—around dependency declarations, configuration properties, and the REST endpoint—explaining why each piece exists and how it contributes to tracing, logging, or metrics.

24. **Ticket 24: Insert Plain-English Comments in Inventory Service Code**
    **Description:** Add comments to the Inventory Service’s code—especially around custom child span creation and downstream calls—explaining why custom spans are used, how trace propagation works, and how logging integration ties in.

25. **Ticket 25: Insert Plain-English Comments in Order Service Code**
    **Description:** Add comments to the Order Service’s code—emphasizing Sleuth’s auto-instrumentation and how trace IDs propagate from one service to the next.

26. **Ticket 26: Create Sample Kibana Visualization Instructions**
    **Description:** Draft instructions in the README (or a separate document) on building a simple Kibana visualization to show log counts by level over time and how to save a search for a single `traceId`. Explain each step in plain English—what to click, what to configure.

27. **Ticket 27: Create Sample Grafana Dashboard Instructions**
    **Description:** Draft instructions in the README (or a separate document) on creating or importing a basic Grafana dashboard with panels for request count, p95 latency, JVM memory, and GC pauses. Explain query selection and why each panel is helpful.

28. **Ticket 28: Validate Zipkin Trace Output**
    **Description:** Test that sending requests triggers a complete trace in Zipkin with three spans (plus one custom span). Document any adjustments to code or configuration needed to achieve correct span propagation.

29. **Ticket 29: Validate ELK Log Ingestion**
    **Description:** Test that JSON logs from all three services appear correctly in Elasticsearch and can be searched by `traceId`. Document any fixes needed in Logstash pipeline or logging configuration.

30. **Ticket 30: Validate Prometheus Scrape and Grafana Panels**
    **Description:** Verify that Prometheus successfully scrapes metrics from all services and that Grafana panels display the expected metrics. Adjust queries or scrape configs if panels show no data, and document those changes.

31. **Ticket 31: Add Troubleshooting Tips Section to README**
    **Description:** Draft a troubleshooting subsection covering common errors: no traces in Zipkin (check base URL/config), no logs in Kibana (check Logstash port), no metrics in Grafana (check Prometheus targets). Provide commands to inspect Docker container logs.

32. **Ticket 32: Create Automated Smoke Test Script**
    **Description:** Write a simple shell or Python script that sends sample `curl` requests to all three services and checks for HTTP 200 responses. Comment on how this script verifies that services are running and instrumented.

33. **Ticket 33: Document Smoke Test Execution in README**
    **Description:** Add instructions in the README on how to run the smoke test script and interpret its output. Explain why automated tests help confirm observability setup is correct.

34. **Ticket 34: Conduct Final Editorial Review of COMMENTS\_GUIDE**
    **Description:** Perform a thorough review of the `COMMENTS_GUIDE.md` document to ensure clarity, remove any leftover jargon, and confirm that examples align with the demo code.

35. **Ticket 35: Conduct Final Editorial Review of README**
    **Description:** Review the README end-to-end, ensuring that all instructions are accurate, steps are in logical order, and that acceptance criteria (trace, logs, metrics) are clearly described with expected outcomes.

36. **Ticket 36: Merge All Feature Branches**
    **Description:** After individual tickets are complete, merge all branches into the main branch. Ensure CI builds pass (if applicable) and update version information as needed.

37. **Ticket 37: Prepare Demo for Presentation**
    **Description:** Verify that a fresh clone and a single sequence of Docker Compose commands plus service launches yield all observability features working. Document any last-minute notes needed to present the demo smoothly.

38. **Ticket 38: Archive Final Project Artifacts**
    **Description:** Tag the final version in the repository, package a zip of the project, and store any relevant screenshots of Zipkin, Kibana, and Grafana panels in a project folder for easy reference.

39. **Ticket 39: Gather Feedback and Iterate**
    **Description:** Solicit feedback from a small group of developers, note any confusion or missing steps, and create any follow-up tickets to address gaps in documentation or setup.

40. **Ticket 40: Close Out and Document Lessons Learned**
    **Description:** Write a short postmortem summarizing what went well, what was challenging, and any improvements for future observability demos. Add it to the project’s documentation.

---

*These tickets separate out tasks that introduce new observability concepts—distributed tracing, logging, metrics—while grouping related backend setup steps when no new concept is introduced. Each ticket focuses on teaching or validating a specific piece of the demo.*
