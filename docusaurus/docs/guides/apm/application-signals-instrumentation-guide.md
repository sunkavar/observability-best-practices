# Application Signals Instrumentation Guide

Comprehensive guide to Amazon CloudWatch Application Signals architecture, instrumentation strategies, and implementation approaches.

## Introduction

AWS Application Performance Monitoring (APM) has evolved significantly with the introduction of [Amazon CloudWatch Application Signals](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/CloudWatch-Application-Monitoring-Sections.html) and [Amazon CloudWatch Transaction Search](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/CloudWatch-Transaction-Search.html). Application Signals provides automatic service discovery, golden signal metrics, and comprehensive service health monitoring, while Transaction Search enables 100% span ingestion with advanced analytics capabilities and cost-effective storage through CloudWatch Logs.

Application Signals automatically instruments your applications on AWS to monitor current application health and track long-term application performance against business objectives. It provides a unified, application-centric view of your applications, services, and dependencies, helping you monitor and triage application health without writing custom code or creating dashboards.

This guide explores the technical architecture, benefits, and implementation strategies for Application Signals, with particular focus on why organizations should migrate from traditional X-Ray sampling to the new comprehensive observability approach.

## Problems and Challenges with Traditional Monitoring

### Observability Gap in Modern Applications

Traditional monitoring approaches were designed for simpler, monolithic applications. As organizations have adopted microservices, serverless, and cloud-native architectures, the limitations of legacy monitoring solutions have become increasingly apparent.

#### Fragmented Monitoring Landscape

Most organizations struggle with a patchwork of monitoring tools that don't provide unified visibility:

| Monitoring Layer | Common Challenges |
|---|---|
| **Infrastructure** | Limited application context |
| **Application Performance** | Siloed metrics, no correlation |
| **Distributed Tracing** | Sampling gaps, cost constraints |
| **Logs** | Difficult correlation with traces |
| **Business Metrics** | Disconnected from technical data |

#### Key Limitations of Traditional Monitoring

**Visibility Gaps**
- **Incomplete Data Coverage**: Sampling and aggregation hide critical edge cases and anomalies
- **Service Boundary Blindness**: Difficult to trace requests across microservice boundaries
- **Customer-Specific Issues**: Aggregate metrics mask individual customer experience problems
- **Intermittent Problems**: Transient issues disappear in averaged metrics

**Cost and Complexity**
- **Tool Sprawl**: Multiple monitoring solutions increase licensing and operational costs
- **Data Silos**: Separate storage systems for metrics, traces, and logs
- **Manual Correlation**: Engineers spend significant time connecting data across tools
- **Scaling Challenges**: Traditional tools struggle with cloud-native application volumes

**Operational Inefficiencies**
- **Slow Mean Time to Detection (MTTD)**: Issues discovered through customer complaints rather than proactive monitoring
- **Extended Mean Time to Resolution (MTTR)**: Complex troubleshooting across multiple tools and data sources
- **Alert Fatigue**: High false positive rates from disconnected monitoring systems
- **Context Switching**: Engineers lose productivity switching between monitoring interfaces


### Modern Observability Requirements

Today's cloud-native applications demand a fundamentally different approach to observability. The shift from monolithic to distributed architectures, combined with increasing customer expectations and regulatory requirements, requires unified, comprehensive visibility.

**Unified Application-Centric View**
- **Service Discovery**: Automatic identification and mapping of application components
- **Golden Signal Metrics**: Rate, errors, duration, and saturation across all services
- **Business Context Integration**: Connect technical performance to business outcomes
- **Customer Journey Tracking**: End-to-end visibility across distributed transactions

**Real-Time Intelligence**
- **Proactive Anomaly Detection**: Identify issues before they impact customers
- **Intelligent Alerting**: Context-aware notifications with reduced false positives
- **Root Cause Analysis**: Automated correlation across metrics, traces, and logs
- **Performance Optimization**: Data-driven insights for continuous improvement

**Advanced Analytics and Insights**
- **Complete Transaction Visibility**: Every request matters, especially for high-value customers
- **Advanced Query Capabilities**: Flexible analysis of telemetry data with business context
- **Machine Learning Integration**: Predictive analytics and pattern recognition
- **Custom Business Metrics**: Derive business KPIs from technical telemetry

## Why X-Ray Customers Should Adopt Application Signals + Transaction Search

### The Evolution of Observability Needs

As applications have grown in complexity and scale, customer observability requirements have evolved significantly. While AWS X-Ray has served as a reliable distributed tracing solution, the modern application landscape demands more comprehensive visibility.

### Technical Architecture Differences

**X-Ray Traditional Approach:**

![X-Ray Architecture](../../images/APM/X-ray.png)

**Application Signals + Transaction Search:**

![Application Signals + Transaction Search Architecture](../../images/APM/ap%20ts.png)

### Key Migration Benefits

| Capability | X-Ray | Application Signals + Transaction Search |
|---|---|---|
| **Data Ingestion** | 100% of transactions (when configured) | 100% of transactions (when configured) |
| **Throughput Limits** | May hit X-Ray service limits at high volume | Higher throughput capacity with CloudWatch Logs |
| **Cost Model** | Per-trace pricing (expensive at 100%) | Application Signals Bundled pricing |
| **Storage Format** | X-Ray proprietary format | OpenTelemetry standard format |
| **Storage Backend** | X-Ray optimized storage | CloudWatch Logs with selective indexing |
| **Analytics** | X-Ray console only | Transaction Search + X-Ray trace analytics |
| **Query Capabilities** | X-Ray console and APIs | Transaction Search visual analytics + X-Ray |
| **Indexing** | All traces indexed | Selective indexing (configurable %) |
| **Business Context** | Limited custom attributes | Rich OTEL span attributes + business context |

### Primary Value Propositions

**1. Higher Throughput and Scalability**
- **CloudWatch Logs handles higher throughput than X-Ray**, enabling customers to track all application events without hitting service limits
- **Logs as storage for trace data** removes X-Ray's throughput constraints for high-volume applications
- **Scalable infrastructure** designed for massive log ingestion volumes

**2. Enhanced Analytics and Integration Capabilities**
- **Native CloudWatch Logs features** available for span data analysis:
  - **Metrics Filters**: Create custom metrics from span attributes and patterns
  - **Subscription Filters**: Stream span data to other AWS services (Lambda, Kinesis, etc.)
  - **Log Insights**: Advanced querying capabilities beyond traditional trace analysis
- **Transaction Search provides advanced visual query interface** for span-level analytics
- **OTEL format enables richer business context** in spans with custom attributes

**3. Cost Effective 100% Sampling**
- **Bundled pricing** makes complete visibility cost-effective compared to per-trace X-Ray pricing. Please see **Example 13** in [CloudWatch pricing page](https://aws.amazon.com/cloudwatch/pricing/)
- **Predictable costs** based on data volume, not trace count
- **Selective indexing** optimizes storage costs while maintaining complete data access

### Leveraging CloudWatch Logs Features with Span Data

Since Transaction Search stores span data in CloudWatch Logs (`aws/spans` log group), you can leverage all native CloudWatch Logs capabilities:

**Metrics Filters:**
```bash
# Create custom metrics from span attributes
aws logs put-metric-filter \
  --log-group-name "aws/spans" \
  --filter-name "HighLatencyRequests" \
  --filter-pattern '[timestamp, request_id, span_id, trace_id, duration > 5000]' \
  --metric-transformations \
    metricName=HighLatencySpans,metricNamespace=CustomApp/Performance,metricValue=1
```

**Subscription Filters:**
```bash
# Stream span data to Lambda for real-time processing
aws logs put-subscription-filter \
  --log-group-name "aws/spans" \
  --filter-name "ErrorSpanProcessor" \
  --filter-pattern '[..., status_code="ERROR"]' \
  --destination-arn "arn:aws:lambda:region:account:function:ProcessErrorSpans"
```

**Log Insights Queries:**
```sql
-- Find all spans with specific business attributes
fields @timestamp, attributes.customer_id, attributes.order_value, duration
| filter attributes.service_name = "payment-service"
| filter attributes.customer_tier = "premium"
| stats avg(duration) by attributes.customer_id
| sort avg(duration) desc
```

**Integration Opportunities:**
- **Real-time Alerting**: Use subscription filters to trigger Lambda functions for immediate incident response
- **Business Intelligence**: Export span data to analytics platforms via Kinesis Data Streams
- **Custom Dashboards**: Create CloudWatch dashboards using metrics derived from span attributes
- **Compliance Auditing**: Use Log Insights to query spans for regulatory compliance reporting


## Setting up Application Signals + Transaction Search

### High-Level Setup Process

![Setup Overview](../../images/APM/overview.png)

### Step 1: Enable Application Signals in your account

Refer to [Enable Application Signals in your account](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/CloudWatch-Application-Signals-Enable.html) documentation.

### Step 2: Enable Transaction Search

Refer to [Enable transaction search](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/Enable-TransactionSearch.html) documentation.

### Step 3: Choose Your Instrumentation Strategy

Based on your requirements, select one of the instrumentation approaches:

#### Decision Matrix

| Approach | Best For | Key Benefits |
|---|---|---|
| **ADOT SDK + CloudWatch Agent** | AWS-native environments, deep service integration | Tight AWS integration, Container Insights correlation, managed experience |
| **ADOT SDK + Custom OTEL Collector** | Multi-destination telemetry with full Application Signals support | Client-side RED metrics, App Signals processor, multi-destination flexibility |
| **Upstream OTEL SDK + OTEL Collector** | Vendor-neutral strategy, non-ADOT languages, multi-cloud | Full vendor neutrality, any OTEL-supported language, no AWS SDK dependency |
| **Direct OTLP Endpoint (Collector-less tracing)** | Resource-efficient applications, minimal infrastructure | Minimal overhead, simplified architecture, reduced infrastructure |
| **X-Ray SDKs** | Legacy X-Ray users, gradual migration | Existing investment protection, minimal change requirements. ⚠️ End of support Feb 2027 |

#### Feature Comparison

| Feature | ADOT SDK + CW Agent | ADOT SDK + Custom OTEL Collector | Upstream OTEL SDK + OTEL Collector | Collector-less tracing with ADOT SDK | X-Ray SDKs |
|---|---|---|---|---|---|
| **AWS Support** | ✅ Yes | ⚠️ Only for data sent to AWS | ⚠️ Only for data sent to AWS | ✅ Yes | ✅ Yes (⚠️ End of support Feb 2027) |
| **Nonstandard language support** | ❌ No | ❌ No | ✅ Yes | ❌ No | ❌ No |
| **Container Insights integration** | ✅ Yes | ❌ No | ❌ No | ❌ No | ❌ No |
| **Out of the box logging with CloudWatch Logs** | ✅ Yes | ❌ No | ❌ No | ✅ Yes | ❌ No |
| **Out of the box runtime metrics** | ✅ Yes | ✅ Yes | ✅ Yes | ❌ No | ❌ No |
| **Always gets RED metrics on 100% of traffic** | ✅ Yes (client-side) | ✅ Yes (client-side) | ⚠️ Only with 100% sampling (server-side) | ⚠️ Only with 100% sampling | ⚠️ Only with 100% sampling |
| **Multi-destination telemetry** | ❌ No | ✅ Yes | ✅ Yes | ❌ No | ❌ No |


### Step 4: Understanding Sampling and Trace Indexing

Application Signals separates **request sampling** from **trace indexing**:
- **Request Sampling**: Determines which percentage of requests are sampled and sent to AWS
- **Selective Trace Indexing**: Percentage of spans stored in CloudWatch Logs that are sent to X-Ray backend for trace analytics

#### Request Sampling

**1. Default Application Signals Sampling Configuration**

When you enable Application Signals, **X-Ray centralized sampling is enabled by default** with these settings:

| Setting | Default Value | Description |
|---|---|---|
| **Reservoir** | 1 request/second | Fixed number of requests sampled per second |
| **Fixed Rate** | 5% | Percentage of additional requests beyond reservoir |

The environment variables for the AWS Distro for OpenTelemetry (ADOT) SDK agent are set as follows:

| Environment Variable | Value | Description |
|---|---|---|
| **OTEL_TRACES_SAMPLER** | `xray` | Uses X-Ray sampling service |
| **OTEL_TRACES_SAMPLER_ARG** | `endpoint=http://cloudwatch-agent.amazon-cloudwatch:2000` | CloudWatch agent endpoint |

**2. Configuring 100% Sampling for visibility of all requests**

**Option 1: X-Ray Centralized Sampling (Recommended)**
- Configure X-Ray sampling rules for 100% sampling
- Refer to [Configure sampling rules](https://docs.aws.amazon.com/xray/latest/devguide/xray-console-sampling.html) documentation
- Benefits: Centralized control, dynamic updates, service-specific rules

**Option 2: Local sampling in the ADOT SDK**

For local control, disable X-Ray centralized sampling:

| Environment Variable | Value | Description |
|---|---|---|
| **OTEL_TRACES_SAMPLER** | `parentbased_traceidratio` | Local sampling |
| **OTEL_TRACES_SAMPLER_ARG** | `1.0` | 100% sampling rate |


**3. Alternative: X-Ray Adaptive Sampling (Cost-Optimized Approach)**

If you don't need 100% sampling but want better anomaly coverage, consider X-Ray adaptive sampling which automatically increases sampling during error spikes and latency outliers while maintaining cost-effective baseline rates:

Key Benefits:
- **Automatic anomaly detection**: Boosts sampling during HTTP 5xx errors or high latency
- **Cost control**: Maintains low baseline sampling (e.g., 5%) during normal operations
- **Configurable boost limits**: Set maximum sampling rates and cooldown periods
- **Critical trace capture**: Ensures anomaly spans are captured even when full traces aren't sampled
- **Centralized control**: Configure through X-Ray sampling rules without application code changes

Configuration Example:
```json
{
  "RuleName": "AdaptiveProductionRule",
  "Priority": 1,
  "ReservoirSize": 1,
  "FixedRate": 0.05,
  "ServiceName": "*",
  "ServiceType": "*",
  "Host": "*",
  "HTTPMethod": "*",
  "URLPath": "*",
  "SamplingRateBoost": {
    "MaxRate": 0.25,
    "CooldownWindowMinutes": 10
  }
}
```

Requirements:
- ADOT Java SDK (v2.11.5 or higher)
- Must run with CloudWatch Agent or OpenTelemetry Collector
- Compatible with Amazon EC2, ECS, EKS, and self-hosted Kubernetes

For detailed setup instructions, refer to [X-Ray Adaptive Sampling](https://docs.aws.amazon.com/xray/latest/devguide/xray-adaptive-sampling.html) documentation.

> For more advanced sampling configurations, see [OTEL_TRACES_SAMPLER](https://opentelemetry.io/docs/concepts/sdk-configuration/general-sdk-configuration/#otel_traces_sampler) documentation.

#### Trace Indexing

**1. Default Indexing Rate:**
- 1% indexing is included at no additional charge
- Above 1% indexing incurs X-Ray pricing charges
- Refer to [CloudWatch Pricing](https://aws.amazon.com/cloudwatch/pricing/) documentation for current rates

**2. Custom Indexing Rates:**
```bash
# Higher indexing for applications requiring more X-Ray analytics (incurs charges)
aws cloudwatch put-transaction-search-configuration \
  --span-indexing-rate 0.10  # 10% indexing - X-Ray charges apply

# Lower indexing for cost optimization (still within free tier)
aws cloudwatch put-transaction-search-configuration \
  --span-indexing-rate 0.005  # 0.5% indexing - no additional charges
```


## Different Instrumentation Setups

### ADOT SDK + CloudWatch Agent

This approach provides the most integrated AWS experience with deep service integration and automatic correlation with AWS infrastructure metrics.

#### Key Benefits
- **Metrics such as call volume, availability, latency, faults, and errors** are calculated on 100% of requests at the client-side before sampling decision
- **X-Ray Sampling Integration** uses X-Ray sampling rules by default (configure for 100% if needed)
- **Out-of-the-box CloudWatch Logs integration** for seamless log correlation
- **Full AWS Support** for the entire observability stack
- **Automatic service discovery** and golden signals

#### Architecture

![ADOT SDK + CloudWatch Agent Architecture](../../images/APM/adotcw.png)

#### How ADOT SDK + CloudWatch Agent Works

**Step 1: Application Instrumentation**

When you deploy the ADOT SDK, it automatically instruments your application without requiring code changes. ADOT SDK dynamically injects code into an application at runtime, without requiring manual code changes. This injected code automatically detects calls to supported frameworks, creates spans for each operation, and propagates context across services to build a complete trace.

**Step 2: Sampling Decision**

For each request, the ADOT SDK checks your X-Ray sampling rules to decide whether to send the full trace data. You can configure this from 5% for cost savings up to 100% for complete visibility.

**Step 3: Client-Side Metrics Calculation**

Here's the key advantage: before sampling happens, the SDK calculates RED (Requests, Errors, Duration) metrics on 100% of requests when `OTEL_AWS_APPLICATION_SIGNALS_ENABLED=true`. This means you get complete golden signals even with low sampling rates:
- **Rate**: Count of requests per time window
- **Errors**: Count of requests with error status codes (4xx/5xx)
- **Duration**: Latency measurements from request start/end times

**Step 4: CloudWatch Agent Processing**

The ADOT SDK sends both sampled spans and pre-calculated metrics to the CloudWatch Agent, which processes them through a pipeline:

![ADOT SDK CloudWatch Agent Detailed Pipeline](../../images/APM/adosdkcwdetailed.jpg)

- **OTLP Receiver**: Accepts traces and metrics from your application
- **Resource Detector**: Adds AWS resource info (instance IDs, container details)
- **APM Processor**: Enriches spans with platform-specific metadata
- **Exporters**: Routes data to X-Ray (spans) and CloudWatch (metrics)

![APM Processor](../../images/APM/apmprocessor.png)


**Step 5: Data Distribution**

Your data splits into three paths:
- **Metrics** → `/aws/application-signals/data` log group for Application Maps
- **Spans** → `aws/spans` log group for Transaction Search
- **Indexed spans** → X-Ray backend for traditional trace analysis

**Step 6: Analytics Options**

This gives you three ways to analyze your data:
- **Application Signals**: Application Maps with dynamic grouping and golden signals from complete metrics
- **Transaction Search**: Query all span data with advanced filters
- **X-Ray Analytics**: Traditional trace analysis on indexed spans

#### Implementation Guides

Follow platform-specific setup guides:
- [Enable Application Signals on Amazon EKS](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/CloudWatch-Application-Signals-Enable-EKS.html)
- [Enable Application Signals on Amazon ECS](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/CloudWatch-Application-Signals-Enable-ECS.html)
- [Enable Application Signals on Amazon EC2](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/CloudWatch-Application-Signals-Enable-EC2.html)
- [Enable Application Signals on Self hosted Kubernetes](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/CloudWatch-Application-Signals-Enable-KubernetesMain.html)
- [Application Signals Demo repository](https://github.com/aws-observability/application-signals-demo)

Once done, verify service discovery and golden signals in the Application Signals console.


### ADOT SDK + Custom OTEL Collector

This approach combines the ADOT SDK's client-side RED metrics calculation with the flexibility of a custom-built OpenTelemetry Collector that includes the AWS Application Signals Processor. You get the same accurate 100%-of-traffic metrics as the CloudWatch Agent approach, plus the ability to fan out telemetry to multiple destinations.

#### Key Benefits
- **Client-side RED metrics on 100% of requests** via ADOT SDK (same as CW Agent approach) — metrics are calculated before sampling
- **Multi-destination telemetry** — fan out to AWS, Datadog, Prometheus, etc. simultaneously
- **App Signals Processor** normalizes `aws.local.*` / `aws.remote.*` attributes, resolves platform context, and controls cardinality
- **Full control over collector pipeline** — add custom processors, filters, and exporters

#### Architecture

![ADOT SDK + Custom OTEL Collector Architecture](../../images/APM/adot-sdk-custom-collector.png)

#### How ADOT SDK + Custom OTEL Collector Works

**Step 1: Application Instrumentation**

Your application gets instrumented with the ADOT SDK, which captures runtime metrics, logs, and traces in OpenTelemetry format. The ADOT SDK injects AWS-specific span attributes (`aws.local.service`, `aws.local.operation`, `aws.remote.service`, `aws.remote.operation`, etc.) that the App Signals Processor depends on.

**Step 2: Client-Side RED Metrics Calculation**

When `OTEL_AWS_APPLICATION_SIGNALS_ENABLED=true`, the ADOT SDK calculates RED metrics on 100% of requests **before** any sampling decision:
- **Rate**: Count of requests per time window
- **Errors**: Count of requests with error status codes (4xx/5xx)
- **Duration**: Latency measurements from request start/end times

**Step 3: Sampling Decision**

The ADOT SDK applies your configured sampling strategy (X-Ray sampling rules or local sampling). Only sampled traces get sent to the collector, but RED metrics are already calculated on 100% of traffic.


**Step 4: Custom OpenTelemetry Collector Processing Pipeline**

**OTLP Receivers (Data Ingestion)**
```yaml
receivers:
  otlp:
    protocols:
      grpc:
        endpoint: 0.0.0.0:4317
      http:
        endpoint: 0.0.0.0:4318
```

**Resource Detection Processor**
```yaml
processors:
  resourcedetection:
    detectors:
      - eks
      - env
      - ec2
```

**Application Signals Processor**
```yaml
processors:
  awsapplicationsignals:
    resolvers:
      - platform: ecs
```

This processor works with the `aws.local.*` / `aws.remote.*` span attributes that the ADOT SDK injects. It performs:
1. **Attribute Resolution**: Uses platform-specific resolvers to enrich telemetry with platform context
2. **Attribute Normalization**: Renames ADOT SDK attributes to CloudWatch metric dimension names
3. **Cardinality Control**: Applies user-configured `keep`/`drop`/`replace` rules
4. **Application Map Generation**: Creates topology data with dynamic grouping

**Step 5: Export Processing**

Exporters route data to AWS EMF (metrics), OTLP HTTP (logs), and OTLP HTTP (traces) endpoints with SigV4 authentication.

**Step 6: Backend Processing**
1. CloudWatch Logs: Extracts metrics from EMF logs, stores span data in `aws/spans`
2. X-Ray Backend: Indexes configurable percentage of spans for trace analytics

**Step 7: Analytics and Visualization**
- **Application Signals**: Uses client-side calculated RED metrics — accurate on 100% of traffic regardless of sampling
- **Transaction Search**: Queries span data from CloudWatch Logs
- **X-Ray Analytics**: Traditional trace analysis on indexed spans


#### Building Custom OTEL Collector with awsapplicationsignalsprocessor

**Prerequisites**: Install Go (version 1.21 or later).

**Step 1: Install the OpenTelemetry Collector Builder (ocb)**

For latest binaries, see [opentelemetry-collector-releases](https://github.com/open-telemetry/opentelemetry-collector-releases/releases).

```bash
# macOS (ARM64)
curl --proto '=https' --tlsv1.2 -fL -o ocb \
https://github.com/open-telemetry/opentelemetry-collector-releases/releases/download/cmd%2Fbuilder%2Fv0.132.4/ocb_0.132.4_darwin_arm64
chmod +x ocb
```

**Step 2: Create Builder Manifest File**

Create `builder-config.yaml`:
```yaml
dist:
  name: otelcol-appsignals
  description: OTel Collector for Application Signals
  output_path: ./otelcol-appsignals
exporters:
  - gomod: github.com/open-telemetry/opentelemetry-collector-contrib/exporter/awsemfexporter v0.113.0
  - gomod: go.opentelemetry.io/collector/exporter/otlphttpexporter v0.113.0
processors:
  - gomod: github.com/amazon-contributing/opentelemetry-collector-contrib/processor/awsapplicationsignalsprocessor v0.113.0
  - gomod: github.com/open-telemetry/opentelemetry-collector-contrib/processor/resourcedetectionprocessor v0.113.0
  - gomod: github.com/open-telemetry/opentelemetry-collector-contrib/processor/metricstransformprocessor v0.113.0
receivers:
  - gomod: go.opentelemetry.io/collector/receiver/otlpreceiver v0.113.0
extensions:
  - gomod: github.com/open-telemetry/opentelemetry-collector-contrib/extension/awsproxy v0.113.0
  - gomod: github.com/open-telemetry/opentelemetry-collector-contrib/extension/sigv4authextension v0.113.0
replaces:
  - github.com/open-telemetry/opentelemetry-collector-contrib/internal/aws/awsutil v0.113.0 => github.com/amazon-contributing/opentelemetry-collector-contrib/internal/aws/awsutil v0.113.0
  - github.com/open-telemetry/opentelemetry-collector-contrib/internal/aws/cwlogs v0.113.0 => github.com/amazon-contributing/opentelemetry-collector-contrib/internal/aws/cwlogs v0.113.0
  - github.com/open-telemetry/opentelemetry-collector-contrib/exporter/awsemfexporter v0.113.0 => github.com/amazon-contributing/opentelemetry-collector-contrib/exporter/awsemfexporter v0.113.0
  - github.com/openshift/api v3.9.0+incompatible => github.com/openshift/api v0.0.0-20180801171038-322a19404e37
```


**Step 3: Sample Collector Configuration**

```yaml
receivers:
  otlp:
    protocols:
      grpc:
        endpoint: 0.0.0.0:4317
      http:
        endpoint: 0.0.0.0:4318
processors:
  awsapplicationsignals:
    resolvers:
      - platform: eks
  resourcedetection:
    detectors:
      - eks
      - env
      - ec2
exporters:
  otlphttp/logs:
    compression: gzip
    logs_endpoint: https://logs.us-east-1.amazonaws.com/v1/logs
    auth:
      authenticator: sigv4auth/logs
  otlphttp/traces:
    compression: gzip
    traces_endpoint: https://xray.us-east-1.amazonaws.com/v1/traces
    auth:
      authenticator: sigv4auth/traces
extensions:
  sigv4auth/logs:
    region: "us-east-1"
    service: "logs"
  sigv4auth/traces:
    region: "us-east-1"
    service: "xray"
service:
  extensions: [sigv4auth/logs, sigv4auth/traces]
  pipelines:
    logs:
      receivers: [otlp]
      exporters: [otlphttp/logs]
    traces:
      receivers: [otlp]
      processors: [resourcedetection, awsapplicationsignals]
      exporters: [otlphttp/traces]
```

**Step 4: Build Docker Image**

```bash
docker buildx build --load \
  -t otelcol-appsignals:latest \
  --platform=linux/amd64 .
```


### Upstream OpenTelemetry SDK + OTEL Collector

This approach uses the standard upstream OpenTelemetry SDK (not ADOT) with an OpenTelemetry Collector. It provides maximum vendor neutrality and supports any language with an OpenTelemetry SDK, including those not supported by ADOT (Erlang, Rust, Ruby, etc.). RED metrics are calculated server-side by the X-Ray backend from sampled trace data.

#### Key Benefits
- **Full vendor neutrality** — no AWS-specific SDK dependency on the client side
- **Any OTEL-supported language** — works with Erlang, Rust, Ruby, PHP, and all other upstream OTEL SDKs
- **Multi-cloud and hybrid environments** — same SDK works across AWS, GCP, Azure, and on-premises
- **Standard upstream OTEL Collector** with standard processors and exporters
- **Existing OpenTelemetry investments** preserved — no migration to ADOT needed
- **Multi-destination telemetry** — fan out to any backend simultaneously

#### Architecture

![Upstream OpenTelemetry SDK + OTEL Collector Architecture](../../images/APM/upstream-otel-sdk-otel-collector.png)

#### How Upstream OTEL SDK + Collector Works

**Step 1: Application Instrumentation**

Your application gets instrumented with the standard upstream OpenTelemetry SDK. This produces standard OTEL spans with semantic conventions (`http.method`, `http.route`, `http.status_code`, etc.).

**Step 2: Client-Side Sampling**

The OTEL SDK applies your configured sampling strategy. For accurate RED metrics, you need `always_on` sampling (100%) since metrics are calculated server-side from sampled traces only. With partial sampling, your RED metrics will only reflect the sampled subset.

**Step 3: Standard OTEL Collector Processing Pipeline**

The collector uses standard upstream processors:

```yaml
receivers:
  otlp:
    protocols:
      grpc:
        endpoint: 0.0.0.0:4317
      http:
        endpoint: 0.0.0.0:4318
processors:
  resourcedetection:
    detectors:
      - eks
      - env
      - ec2
  batch:
    send_batch_size: 8192
    timeout: 200ms
```


**Step 4: Server-Side RED Metrics Calculation**

Since the upstream OTEL SDK does not calculate RED metrics client-side, the X-Ray frontend calculates them server-side from the sampled traces it receives:
1. **Rate**: Request counts extracted from sampled span data
2. **Errors**: Error counts identified from sampled span status codes
3. **Duration**: Latency calculated from sampled span start/end times

:::warning
RED metrics accuracy depends entirely on your sampling rate. With 5% sampling, you only get metrics on 5% of traffic. For accurate Application Signals dashboards, configure 100% sampling.
:::

**Step 5: Analytics and Visualization**
- **Application Signals**: Application Maps with golden signals from server-calculated RED metrics (accuracy depends on sampling rate)
- **Transaction Search**: Query span data from CloudWatch Logs (`aws/spans`)
- **X-Ray Analytics**: Traditional trace analysis on indexed spans

#### Key Differences from ADOT SDK Approach

| Aspect | ADOT SDK + Custom Collector | Upstream OTEL SDK + Collector |
|---|---|---|
| **RED Metrics** | Client-side, 100% of traffic | Server-side, only sampled traffic |
| **`aws.*` span attributes** | Injected by ADOT SDK | Not present |
| **Language support** | Java, Python, .NET, Node.js | Any OTEL-supported language |
| **Collector build** | Custom build with App Signals Processor | Standard upstream collector build |
| **100% sampling needed for accurate metrics** | No | Yes |

#### Sample Collector Configuration

```yaml
receivers:
  otlp:
    protocols:
      grpc:
        endpoint: 0.0.0.0:4317
      http:
        endpoint: 0.0.0.0:4318
processors:
  resourcedetection:
    detectors:
      - eks
      - env
      - ec2
  batch:
    send_batch_size: 8192
    timeout: 200ms
exporters:
  otlphttp/logs:
    compression: gzip
    logs_endpoint: https://logs.us-east-1.amazonaws.com/v1/logs
    auth:
      authenticator: sigv4auth/logs
  otlphttp/traces:
    compression: gzip
    traces_endpoint: https://xray.us-east-1.amazonaws.com/v1/traces
    auth:
      authenticator: sigv4auth/traces
extensions:
  sigv4auth/logs:
    region: "us-east-1"
    service: "logs"
  sigv4auth/traces:
    region: "us-east-1"
    service: "xray"
service:
  extensions: [sigv4auth/logs, sigv4auth/traces]
  pipelines:
    logs:
      receivers: [otlp]
      processors: [resourcedetection, batch]
      exporters: [otlphttp/logs]
    traces:
      receivers: [otlp]
      processors: [resourcedetection, batch]
      exporters: [otlphttp/traces]
```


### Collector-less Tracing with OTLP Endpoints

This approach provides minimal infrastructure complexity and reduced resource overhead by sending logs and traces directly to CloudWatch OTLP endpoints.

#### Why Choose Collector-less Tracing

Collector-less tracing is perfect when you want the simplest possible architecture with maximum resource utilization. By sending data directly to AWS endpoints, you eliminate the need for additional infrastructure components and their associated management overhead.

#### Architecture

![Collector-less Architecture](../../images/APM/collectorless.png)

#### How Collector-less Tracing Works

**Step 1: Application Instrumentation**

Your application gets automatically instrumented with the ADOT SDK. It captures logs and traces in OpenTelemetry format without requiring any code changes.

**Step 2: Client-Side Sampling**

The ADOT SDK applies your X-Ray sampling rules to decide which traces to send. All logs get processed regardless of sampling decisions.

**Step 3: Direct AWS Communication**

Instead of going through a collector, your data goes directly to AWS services:
- **Logs** → `https://logs.<region>.amazonaws.com/v1/logs` via OTLP HTTP
- **Traces** → `https://xray.<region>.amazonaws.com/v1/traces` via OTLP HTTP
- **Authentication**: Uses SigV4 with your AWS credentials

**Step 4: Server-Side Metrics Calculation**

The X-Ray frontend analyzes your received traces to calculate RED metrics on the AWS backend. This means your metrics are only as complete as your sampling rate.

**Step 5: Analytics Options**
- **Application Signals**: Application Maps with dynamic grouping and golden signals from server-calculated metrics
- **Transaction Search**: Query complete span data from CloudWatch Logs
- **X-Ray Analytics**: Traditional trace analysis on indexed spans

#### Important Considerations
- **Transaction Search is required** — you must enable it when using OTLP endpoints
- **ADOT SDK is required** — regular OpenTelemetry SDK won't work for this approach
- **Authentication is automatic** — ADOT SDK handles AWS SigV4 authentication
- **Sampling affects metrics** — your Application Signals metrics are only as complete as your sampling rate


### Existing X-Ray SDK + X-Ray Daemon (End of Support Timeline)

:::danger X-Ray SDK and Daemon End of Support Notice
**AWS X-Ray SDKs and daemon will reach end of support on February 25, 2027.**
- **Maintenance Mode:** February 25, 2026 - February 25, 2027 (critical security fixes only)
- **End of Support:** February 25, 2027 onwards (no updates or releases)

**Migration Recommended:** Migrate to OpenTelemetry solutions before February 25, 2027. See [X-Ray End of Support Timeline](https://docs.aws.amazon.com/xray/latest/devguide/xray-daemon-eos.html) and [X-Ray to OpenTelemetry Migration Guide](https://docs.aws.amazon.com/xray/latest/devguide/xray-sdk-migration.html) for details.
:::

This approach is suitable for organizations with existing X-Ray investments who want to gradually adopt Application Signals capabilities while planning their migration to OpenTelemetry.

#### How to Get Started

1. **Enable Transaction Search** for your existing X-Ray data
2. **Configure 100% Sampling** or use adaptive sampling for cost-effective anomaly detection
3. **Plan Your Migration** — start gradually migrating services to ADOT instrumentation

### RED Metrics Calculation Summary

Understanding how RED (Rate, Errors, Duration) metrics are calculated across different instrumentation setups is crucial for choosing the right approach:

| Instrumentation Setup | Calculation Method | Environment Variable | Requirements |
|---|---|---|---|
| **ADOT SDK + CloudWatch Agent** | Client-side | `OTEL_AWS_APPLICATION_SIGNALS_ENABLED=true` | None - works with any sampling |
| **ADOT SDK + Custom OTEL Collector** | Client-side | `OTEL_AWS_APPLICATION_SIGNALS_ENABLED=true` | Custom collector with App Signals Processor |
| **Upstream OTEL SDK + OTEL Collector** | Server-side | N/A (no ADOT SDK) | Transaction Search + 100% sampling for accuracy |
| **Collector-less (ADOT SDK)** | Server-side | `OTEL_AWS_APPLICATION_SIGNALS_ENABLED=false` (default) | Transaction Search + 100% sampling for accuracy |
| **X-Ray SDK + X-Ray Daemon** | Server-side (extrapolated) | N/A | Based on sampled data |

#### Client-side RED Metrics (ADOT SDK — both CW Agent and Custom Collector)

```
Application → ADOT SDK → Calculate Metrics → CW Agent or Custom Collector → AWS
                ↓
            (100% of requests)
```

- **Calculation happens in the application** before any sampling decisions
- **Always accurate** regardless of trace sampling configuration
- **Default behavior** when `OTEL_AWS_APPLICATION_SIGNALS_ENABLED=true`
- **No Transaction Search dependency** for metrics calculation

#### Server-side RED Metrics (Upstream OTEL SDK, Collector-less, X-Ray)

```
Application → Upstream OTEL SDK/Collector → AWS Backend → Calculate Metrics
                ↓
        (Requires 100% sampling for accuracy)
```

- **Calculation happens at AWS backend** (X-Ray frontend) from received span data
- **OTLP-based setups require Transaction Search** to be enabled
- **Needs 100% sampling** for accurate metrics (except X-Ray which extrapolates)


## Instrumentation Samples for Various Programming Languages

This section provides guidance for instrumenting applications with AWS Application Signals across different programming languages and frameworks.

### Demo Applications

- [Application Signals PetClinic Demo](https://github.com/aws-observability/application-signals-demo) - Multi-language Spring Boot microservices with comprehensive instrumentation examples
- [One Observability PetAdoptions Demo](https://github.com/aws-samples/one-observability-demo) - Full-stack application with Java, Python, .NET, Go, Rust, and Node.js services
- [CloudWatch Application Signals SkillBuilder Demo](https://github.com/aws-samples/sample-cloudwatch-application-signals-skillbuilder-demo) - Educational examples with step-by-step instrumentation guides

### Auto-Instrumentation

Auto-instrumentation provides zero-code observability by automatically detecting and instrumenting popular frameworks and libraries. This is the recommended approach for getting started quickly with Application Signals.

#### Java Spring Boot Applications

**Reference Microservices:**
- **PetClinic Demo:**
  - [API Gateway Service](https://github.com/aws-observability/application-signals-demo/tree/main/spring-petclinic-api-gateway)
  - [Customers Service](https://github.com/aws-observability/application-signals-demo/tree/main/spring-petclinic-customers-service)
  - [Vets Service](https://github.com/aws-observability/application-signals-demo/tree/main/spring-petclinic-vets-service)
  - [Visits Service](https://github.com/aws-observability/application-signals-demo/tree/main/spring-petclinic-visits-service)
- **One Observability Demo:** [Pet Search Service (Java)](https://github.com/aws-samples/one-observability-demo/tree/main/src/applications/microservices/petsearch-java)

#### Python Applications

**Reference Microservices:**
- **One Observability Demo:**
  - [Pet List Adoptions Service (Python)](https://github.com/aws-samples/one-observability-demo/tree/main/src/applications/microservices/petlistadoptions-py)
  - [Pet Food Agent - Strands (Python)](https://github.com/aws-samples/one-observability-demo/tree/main/src/applications/microservices/petfoodagent-strands-py)

#### Node.js Applications

**Reference Microservices:**
- **One Observability Demo (Lambda):**
  - [Pet Status Updater (Node.js Lambda)](https://github.com/aws-samples/one-observability-demo/tree/main/src/applications/lambda/petstatusupdater-node)
  - [Pet Food Stock Processor (Node.js Lambda)](https://github.com/aws-samples/one-observability-demo/tree/main/src/applications/lambda/petfood-stock-processor-node)
  - [Pet Food Cleanup Processor (Node.js Lambda)](https://github.com/aws-samples/one-observability-demo/tree/main/src/applications/lambda/petfood-cleanup-processor-node)
  - [Traffic Generator (Node.js Lambda)](https://github.com/aws-samples/one-observability-demo/tree/main/src/applications/lambda/traffic-generator-node)

#### .NET Applications

**Reference Microservices:**
- **PetClinic Demo:** [Payment Service (.NET)](https://github.com/aws-observability/application-signals-demo/tree/main/dotnet-petclinic-payment)
- **One Observability Demo:** [Pet Site Service (.NET)](https://github.com/aws-samples/one-observability-demo/tree/main/src/applications/microservices/petsite-net)


### Manual Instrumentation

Manual instrumentation involves complete end-to-end integration of OpenTelemetry SDKs without any auto-instrumentation agents. This approach provides maximum control over telemetry collection and is required for languages that don't support auto-instrumentation.

#### Go Applications

**Reference Implementation:**
- [Pay for Adoption Service (Go)](https://github.com/aws-samples/one-observability-demo/tree/main/src/applications/microservices/payforadoption-go)

#### Rust Applications

**Reference Implementation:**
- [Pet Food Service (Rust)](https://github.com/aws-samples/one-observability-demo/tree/main/src/applications/microservices/petfood-rs)

### Combined Instrumentation (Auto + Manual)

Combined instrumentation uses auto-instrumentation agents for baseline telemetry while adding custom spans, attributes, and business context through manual instrumentation.

#### Java Spring Boot - Custom Business Context with Auto-Instrumentation

**Reference Implementation:** [Customers Service](https://github.com/aws-observability/application-signals-demo/tree/main/spring-petclinic-customers-service) and [Pet Search Service](https://github.com/aws-samples/one-observability-demo/tree/main/src/applications/microservices/petsearch-java)

#### Key Manual Instrumentation Best Practices

- **Business Context:** Always add relevant business attributes (customer_id, order_value, product_category) to spans
- **Error Handling:** Record exceptions and set appropriate span status codes
- **Custom Metrics:** Create business-specific metrics alongside technical metrics
- **Span Hierarchy:** Use child spans to break down complex operations
- **Attribute Naming:** Follow OpenTelemetry semantic conventions where possible
- **Performance Impact:** Be mindful of instrumentation overhead in high-throughput paths

## Conclusion

AWS Application Signals with Transaction Search represents a significant evolution in application performance monitoring, addressing the fundamental limitations of sampling-based observability. By providing 100% span ingestion capabilities, advanced analytics, and seamless AWS integration, it enables organizations to achieve complete visibility into their application performance and business impact.

For detailed implementation guides and the latest updates, refer to the [official AWS Application Signals documentation](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/CloudWatch-Application-Monitoring-Sections.html).

## Additional Resources

### Documentation
- [AWS Application Signals User Guide](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/CloudWatch-Application-Monitoring-Sections.html)
- [Transaction Search Documentation](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/CloudWatch-Transaction-Search.html)
- [X-Ray SDK Migration Guide](https://docs.aws.amazon.com/xray/latest/devguide/xray-sdk-migration.html)
- [X-Ray Daemon End of Support](https://docs.aws.amazon.com/xray/latest/devguide/xray-daemon-eos.html)
- [ADOT Documentation](https://aws-otel.github.io/docs/)
- [OpenTelemetry Documentation](https://opentelemetry.io/docs/)

### Technical Resources
- [AWS OTEL Java Instrumentation](https://github.com/aws-observability/aws-otel-java-instrumentation)
- [OpenTelemetry Collector Contrib](https://github.com/open-telemetry/opentelemetry-collector-contrib)
- [AWS Application Signals Processor](https://github.com/amazon-contributing/opentelemetry-collector-contrib/tree/main/processor/awsapplicationsignalsprocessor)
- [ADOT Community Examples](https://github.com/aws-observability/aws-otel-community)

### Training Resources
- [AWS Observability Workshop](https://observability.workshop.aws/)
- [OpenTelemetry Community Resources](https://opentelemetry.io/community/)
