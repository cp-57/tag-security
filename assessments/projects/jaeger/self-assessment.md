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

#### 



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

### Goals
The intended goals of the projects including the security guarantees the project
 is meant to provide (e.g., Flibble only allows parties with an authorization
key to change data it stores).

### Non-goals
Non-goals that a reasonable reader of the project’s literature could believe may
be in scope (e.g., Flibble does not intend to stop a party with a key from storing
an arbitrarily large amount of data, possibly incurring financial cost or overwhelming
 the servers)

## Self-assessment use

This self-assessment is created by the [project] team to perform an internal analysis of the
project's security.  It is not intended to provide a security audit of [project], or
function as an independent assessment or attestation of [project]'s security health.

This document serves to provide [project] users with an initial understanding of
[project]'s security, where to find existing security documentation, [project] plans for
security, and general overview of [project] security practices, both for development of
[project] as well as security of [project].

This document provides the CNCF TAG-Security with an initial understanding of [project]
to assist in a joint-assessment, necessary for projects under incubation.  Taken
together, this document and the joint-assessment serve as a cornerstone for if and when
[project] seeks graduation and is preparing for a security audit.

## Security functions and features

* Critical.  A listing critical security components of the project with a brief
description of their importance.  It is recommended these be used for threat modeling.
These are considered critical design elements that make the product itself secure and
are not configurable.  Projects are encouraged to track these as primary impact items
for changes to the project.
* Security Relevant.  A listing of security relevant components of the project with
  brief description.  These are considered important to enhance the overall security of
the project, such as deployment configurations, settings, etc.  These should also be
included in threat modeling.

## Project compliance

* Compliance.  List any security standards or sub-sections the project is
  already documented as meeting (PCI-DSS, COBIT, ISO, GDPR, etc.).

## Secure development practices

* Development Pipeline.  A description of the testing and assessment processes that
  the software undergoes as it is developed and built. Be sure to include specific
information such as if contributors are required to sign commits, if any container
images immutable and signed, how many reviewers before merging, any automated checks for
vulnerabilities, etc.
* Communication Channels. Reference where you document how to reach your team or
  describe in corresponding section.
  * Internal. How do team members communicate with each other?
  * Inbound. How do users or prospective users communicate with the team?
  * Outbound. How do you communicate with your users? (e.g. flibble-announce@
    mailing list)
* Ecosystem. How does your software fit into the cloud native ecosystem?  (e.g.
  Flibber is integrated with both Flocker and Noodles which covers
virtualization for 80% of cloud users. So, our small number of "users" actually
represents very wide usage across the ecosystem since every virtual instance uses
Flibber encryption by default.)

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
