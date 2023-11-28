# Self-assessment
The Self-assessment is the initial document for projects to begin thinking about the
security of the project, determining gaps in their security, and preparing any security
documentation for their users. This document is ideal for projects currently in the
CNCF **sandbox** as well as projects that are looking to receive a joint assessment and
currently in CNCF **incubation**.

For a detailed guide with step-by-step discussion and examples, check out the free 
Express Learning course provided by Linux Foundation Training & Certification: 
[Security Assessments for Open Source Projects](https://training.linuxfoundation.org/express-learning/security-self-assessments-for-open-source-projects-lfel1005/).

# Self-assessment outline

## Table of contents

* [Metadata](#metadata)
  * [Security links](#security-links)
* [Overview](#overview)
  * [Actors](#actors)
  * [Actions](#actions)
  * [Background](#background)
  * [Goals](#goals)
  * [Non-goals](#non-goals)
* [Self-assessment use](#self-assessment-use)
* [Security functions and features](#security-functions-and-features)
* [Project compliance](#project-compliance)
* [Secure development practices](#secure-development-practices)
* [Security issue resolution](#security-issue-resolution)
* [Appendix](#appendix)

## Metadata

A table at the top for quick reference information, later used for indexing.

|   |  |
| -- | -- |
| **Software** | [A link to Jaeger’s repository.](https://github.com/jaegertracing/jaeger)  |
| **Security Provider** | No, the main function of this project is to enable distributed tracing in an organization’s tech infrastructure. Security is not the primary objective.  |
| **Languages** | <ul><li>Go</li><li>Shell</li><li>Makefile</li><li>Python</li><li>Jsonnet</li><li>Dockerfile</li></ul> |
| **SBOM** | Jaeger has an issue about making an SBOM but does not currently have one. Find that issue [here](https://github.com/jaegertracing/jaeger/issues/3943) |
| | |

### Security links

Provide the list of links to existing security documentation for the project. You may
use the table below as an example:
| Doc | url |
| -- | -- |
| Security file | [SECURITY.md](https://github.com/jaegertracing/jaeger/blob/main/SECURITY.md) |
| Default and optional configs | [Securing Jaeger installation](https://www.jaegertracing.io/docs/1.51/security/) |

## Overview

Jaeger is a cloud native, infinitely scalable, distributed data traversal performance tracker. Jaeger collector paired with a tracing SDK like OpenTelemetry’s SDK is used to monitor microservice interactions through traces and “spans” on distributed systems to identify bottlenecks, timeouts and other performance issues.

### Background

Jaeger provides real time monitoring and analytics of complex ecosystems, which is especially crucial in microservice architectures. Importantly, Jaeger can be used to identify potential communication bottlenecks in these systems. When requests are made to microservices, the Jaeger client creates a “span” which is a logical unit of work and provides information like start time, end time, and other operation metadata. These span components are then used to construct traces which are stored and can be visualized via the centralized dashboard provided by Jaeger. These comprehensive traces provide end-to-end tracing which are helpful in identifying issues and getting a full view of the system, as opposed to traditional tracing which isolate individual components.

Jaeger is cross-platform compatible and provides client libraries for a variety of languages and frameworks including Java, Go, Python, and Node.js. These client libraries can be integrated with existing services to capture tracing data and send this to Jaeger agents and collectors which can be reported back to the system.

With its dashboard UI, users are able to make complex queries and gather insight from collected data. They can gather information on both a more granular or broad scale – on specific traces (by trace ID), by operation (HTTP POST requests), error flags, or by time period. 


### Actors
These are the individual parts of your system that interact to provide the 
desired functionality.  Actors only need to be separate, if they are isolated
in some way.  For example, if a service has a database and a front-end API, but
if a vulnerability in either one would compromise the other, then the distinction
between the database and front-end is not relevant.

The means by which actors are isolated should also be described, as this is often
what prevents an attacker from moving laterally after a compromise.

The following are the different actors found in the Jaeger project:

 I. OpenTelemetry SDK

 II. Deprecated [Jaeger agent](https://github.com/jaegertracing/jaeger/issues/4739) (NOT REQUIRED)
 
 III. The Jaeger Collector
 
 IV. Jaeger Query
 
 V. Jaeger Ingester


<img width="726" alt="jaeger_1" src="https://github.com/cp-57/tag-security/assets/109479938/b4b36eea-dd4b-46e2-b8ea-3239eeecf21c">

<i>Graphic #1 – Jaeger system architecture</i>


#### Tracing/ OpenTelemetry SDK

A Tracing or OpenTelemetry’s SDK downloaded on the client or host is used to generate tracing data. An “instrumented application” (ex. OpenTelemetry API) creates spans when receiving a request and attaches context info (trace id, span id, and baggage). Only Ids and baggage are propagated. Other info sent to Jaeger backend asynchronously (Jaeger SDK)

#### Deprecated [Jaeger Agent](https://github.com/jaegertracing/jaeger/issues/4739) (NOT REQUIRED)

The Jaeger Agent is a deprecated network daemon used for listening for spans sent over UDP.

#### Jaeger Collector

The Jaeger Collector receives processes, validates, cleans up/ enriches and stores traces in some backend data storage [(see supported)](https://www.jaegertracing.io/docs/1.50/deployment).

#### Jaeger Query

Jaeger Query exposes [APIs](https://www.jaegertracing.io/docs/1.50/apis) for receiving traces from a database and hosts a web interface for searching and analyzing traces.

#### Jaeger Ingester

Reads traces from Kafka and writes to a database. (stripped down version of jaeger collector supporting Kafka).


### Actions
These are the steps that a project performs in order to provide some service
or functionality.  These steps are performed by different actors in the system.
Note, that an action need not be overly descriptive at the function call level.  
It is sufficient to focus on the security checks performed, use of sensitive 
data, and interactions between actors to perform an action.  

For example, the access server receives the client request, checks the format, 
validates that the request corresponds to a file the client is authorized to 
access, and then returns a token to the client.  The client then transmits that 
token to the file server, which, after confirming its validity, returns the file.

Sampling is necessary to reduce the number of traces stored in the backend. For larger applications this is especially important given the millions (or billions) of requests being made. It reduces overhead.

#### Remote Sampling Mode
Remote sampling centralizes all sampling configurations of Jaeger collectors. It's a feature that lets you adjust how quickly traces get simplified. Jaeger can employ remote sampling to determine the server-side sample method in place of sampling every trace on the client side. 

#### Adaptive Sampling Mode
The Jaeger Collector analyzes the incoming spans received from services with a tracing client like the OpenTelemetry SDK to automatically adjust the sampling rate. Incoming spans and samples are sent to some storage backend configured to the larger system.

#### Direct to Storage
When sending traces directly to storage, containers generate traces using a client like the OpenTelemetry SDK and push that data to the jaeger collector set to either adaptively or remotely sample traces. Those traces are then sent from the Jaeger collector to some backend storage and can be viewed and analyzed on Jaeger Query’s Jaeger UI.

#### Jaeger with Kafka
The Jaeger Ingester is a stripped down version of the Jaeger collector made to accept data from Kafka. Kafka can be used with Jaeger software as an intermediary queue. Jaeger guarantees excellent availability and reliability for trace data, particularly in dispersed and high-throughput settings, by using Kafka as the transport for trace data. The tracing data in Jaeger with Kafka is sent to Kafka instead of the Jaeger collector. After that, the Jaeger Ingester reads and interprets the tracing data from Kafka to send to backend data storage. 

#### Software Release: 
Jaeger is released under the Apache License 2.0, which allows it to be freely used, modified, and distributed.


### Goals
The intended goals of the projects including the security guarantees the project
 is meant to provide (e.g., Flibble only allows parties with an authorization
key to change data it stores).

##### General Goals: 

* Distributed tracing: Jaeger allows tracing the flow of requests and understanding how they propagate through various services in a microservice architecture.
* Monitoring: Jaeger aids in monitoring performance of individual services and the overall system by providing insights into the response times, latency and dependencies between services. Jaeger simplifies the process of debugging and optimizing performance.
Root cause analysis: Jaeger assists in identifying the performance bottlenecks in distributed systems. 
* Scalability: Jaeger has high scalability to handle tracing in large and complex microservices. 
Compatibility: Jaeger is designed to support openTracing, open Telemetry, and multiple storage backends including two NoSQL databases, Cassandra and Elasticsearch.
* Web UI: Implemented in Javascript to handle large volumes of data and display traces with thousands of spans.
Sampling Strategies: To control the amount of trace data that is gathered and kept, offer customizable sampling techniques.

##### Security Goals: 

* Maintain Security: Jaeger includes security features like data encryption in transit and at rest to protect trace data.
* Integration: Ensure user-friendliness and acceptance by integrating seamlessly with operational workflows and development tools.


### Non-goals
Non-goals that a reasonable reader of the project’s literature could believe may
be in scope (e.g., Flibble does not intend to stop a party with a key from storing
an arbitrarily large amount of data, possibly incurring financial cost or overwhelming
 the servers)

##### General Non-Goals:
* Jaeger does not provide real-time alerts and lacks automated notification capabilities for system downtime.
* Detailed system metrics, such as CPU usage, memory usage, and disk input/output, are not collected; reserved for system monitoring tools.
* Lacks automatic anomaly detection (no ML capabilities), requiring users to visually identify anomalies.
* Not intended for business analytics, including user behavior and conversion rates.
* Jaeger complements, rather than replaces, existing monitoring systems.

##### Security Non-Goals:
* Not designed for security monitoring or security compliance monitoring purposes.
* While including security features, Jaeger does not focus on comprehensive security measures.
* Jaeger does not specifically focus on preventing insider data leaks.



## Self-assessment use

This self-assessment is created by the Jaeger team to perform an internal analysis of the
project's security.  It is not intended to provide a security audit of Jaeger, or
function as an independent assessment or attestation of Jaeger's security health.

This document serves to provide Jaeger users with an initial understanding of
Jaeger's security, where to find existing security documentation, Jaeger plans for
security, and general overview of Jaeger security practices, both for development of
Jaeger as well as security of Jaeger.

This document provides the CNCF TAG-Security with an initial understanding of Jaeger
to assist in a joint-assessment, necessary for projects under incubation.  Taken
together, this document and the joint-assessment serve as a cornerstone for if and when
Jaeger seeks graduation and is preparing for a security audit.

## Security functions and features

* Critical.  A listing critical security components of the project with a brief
description of their importance.  It is recommended these be used for threat modeling.
These are considered critical design elements that make the product itself secure and
are not configurable.  Projects are encouraged to track these as primary impact items
for changes to the project.
#### Encryption: 

* Jaeger is capable of encrypting data using Transport Layer Security (TLS) in conjunction with mutual TLS (mTLS). Using mTLS offers a better level of security because it necessitates the validity of certificates on both the client and the server (and thus mitigates man-in-the middle attacks). 
* The OpenTelemetry SDK can communicate to the jaeger collector by gRPC or HTTP with the option to enable TLS with mTLS.
* The Jaeger Collector, Ingester and Query can communicate to storage via third-party software like Cassandra, ElasticSearch and Kafka all with TLS with mTLS support. 
* Elasticsearch supports bearer token propagation and Kafka also supports Kerberos and plaintext authentication.


* Security Relevant.  A listing of security relevant components of the project with
  brief description.  These are considered important to enhance the overall security of
the project, such as deployment configurations, settings, etc.  These should also be
included in threat modeling.

#### Authentication and Authorization
* Bearer tokens are an option offered by Jaeger for these purposes.
* Users and apps are granted restricted access to Jaeger's capabilities through the use of OAuth2 tokens. Jaeger also supports plaintext and Kerberos authentication.

#### Access Control: 
* Role-based access control (RBAC) is a feature that Jaeger provides for both users and applications. 
* This enables system administrators to specify the roles and permissions of various users. This guarantees that sensitive information and functions can only be accessed by authorized users.

#### Security Auditing:
* Jaeger provides tools for monitoring and auditing. It has the ability to monitor user actions such logins, searches, and sensitive data access. Administrators can utilize this data to quickly identify and fix possible security issues.

## Project compliance

* Compliance.  List any security standards or sub-sections the project is
  already documented as meeting (PCI-DSS, COBIT, ISO, GDPR, etc.).

Jaeger does not currently document meeting particular compliance standards.

## Secure development practices

### Development Pipeline.  
A description of the testing and assessment processes that
the software undergoes as it is developed and built. Be sure to include specific
information such as if contributors are required to sign commits, if any container
images immutable and signed, how many reviewers before merging, any automated checks for
vulnerabilities, etc.

* To contribute change, an issue must be opened first. The issue should describe:
  * Requirement: what use case is being solved
  * Problem: what is missing from Jaeger to solve the requirement
  * Proposal: what changes are going to be made to solve the problem 
* All packages must have a complementary test to go along with it.
* After the approach is agreed upon, code changes can be made and a pull request can be opened
* If one wants to work on an existing issue, they can leave a comment under the issue and submit a pull request
* Create a new local branch and sign all your commits
* All dependencies are automatically checked and updated using dependabot
* Push your changes and look for an output containing an URL to create a pull request
* Each PR should have a contain:
  * <50 characters
  * Capitalized title
  * Title not end with a period
  * A description of the problem it’s solving or a reference to the corresponding issue
  * Summary of what changes were made
* The pull request will then be reviewed and merged by the maintainer


#### Communication Channels. 
Reference where you document how to reach your team or
describe in corresponding section.
  
##### Internal
* Jaeger maintainers and contributors have a monthly zoom meeting every 3rd thursday at 11am EST.

##### Inbound
* Inbound Users can contact the Jaeger team via email at jaeger-tracing@googlegroups.com, open an issue on GitHub or send a message to the #jaeger channel on the CNCF Slack.

##### Outbound
* Outbound the Jaeger team communicates with their users on their website and the #jaeger channel on the CNCF Slack.

### Ecosystem. 
How does your software fit into the cloud native ecosystem?  (e.g. Flibber is integrated with both Flocker and Noodles which covers virtualization for 80% of cloud users. So, our small number of "users" actually
represents very wide usage across the ecosystem since every virtual instance uses
Flibber encryption by default.)

#### OpenTelemetry Integration: 
OpenTelemetry can be used in place of the deprecated Jaeger Agent.
* The OpenTelemetry SDK can be used to create trace data for Jaeger’s collector to collect and then store in a backend database.

#### Istio Integration: 
Jaeger offers tracing capabilities for microservices running on Kubernetes by integrating with Istio, a well-known service mesh.
* When the services are instrumented with the Jaeger tracer, Jaeger's client libraries by default produce and send traces to Jaeger.
* These traces are accessible through the Jaeger Query User Interface and are kept in Jaeger's storage backend (Cassandra, Elasticsearch).

#### Cloud Native storage option: 
Jaeger is compatible with a number of storage backends that can be used to store and analyze traces in a cloud-native environment, including Cassandra, Elasticsearch, and Kafka.
* Features and benefits that are specific to each storage backend include cost-effectiveness, scalability, and high availability.
* Trace volume, retention needs, and infrastructure already in place are a few of the factors that must be taken into consideration while selecting a storage backend.

#### Kubernetes Integration: 
Jaeger's Helm chart allows it to be deployed in Kubernetes.
* The Helm chart offers flexibility in setting up different Kubernetes resources, including services, deployments, and configmaps, as well as in configuring Jaeger's components, including the choice of storage backend.
* Jaeger can now be easily deployed and managed in a cloud-native environment, and Kubernetes' sophisticated features, such as self-healing and rolling upgrades, may be utilized.

#### Prometheus Integration: 
Prometheus is a well-liked open-source monitoring and alerting toolkit that can be combined with Jaeger.
* Metrics data and traces can be correlated by merging Jaeger and Prometheus. For instance, you can investigate the related traces in Jaeger to find the source of a latency spike that you see in Prometheus.
* Additionally, you may make custom dashboards to keep an eye on your microservices in a cloud-native environment by utilizing Jaeger's interface with Grafana, a potent data visualization and monitoring tool.
* Jaeger excels in distributed trace capture, while Prometheus focuses on time-series metrics for system monitoring. Together, they provide a comprehensive view of distributed system behavior and performance.


## Security issue resolution

* Responsible Disclosures Process. A outline of the project's responsible
  disclosures process should suspected security issues, incidents, or
vulnerabilities be discovered both external and internal to the project. The
outline should discuss communication methods/strategies.
  * Vulnerability Response Process. Who is responsible for responding to a
    report. What is the reporting process? How would you respond?
* Incident Response. A description of the defined procedures for triage,
  confirmation, notification of vulnerability or security incident, and
patching/update availability.

## Appendix

* Known Issues Over Time. List or summarize statistics of past vulnerabilities
  with links. If none have been reported, provide data, if any, about your track
record in catching issues in code review or automated testing.
* [CII Best Practices](https://www.coreinfrastructure.org/programs/best-practices-program/).
  Best Practices. A brief discussion of where the project is at
  with respect to CII best practices and what it would need to
  achieve the badge.
* Case Studies. Provide context for reviewers by detailing 2-3 scenarios of
  real-world use cases.
* Related Projects / Vendors. Reflect on times prospective users have asked
  about the differences between your project and projectX. Reviewers will have
the same question.