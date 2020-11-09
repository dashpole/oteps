# Collector Conformance

Collector distributions must be able to send and recieve OTLP.

## Background

[Kubernetes conformance](https://github.com/cncf/k8s-conformance) testing, which this proposal leans heavily on, is a suite of tests that can verify that a cluster implements the minimum set of functionality to be called a Kubernetes cluster.  This is necessary to meet the project’s stated goals of consistency and portability between kubernetes clusters.

[The OpenTelemetry Collector’s Vision](https://github.com/open-telemetry/opentelemetry-collector/blob/master/docs/vision.md#extensible) mentions the possibility of building custom agents based on the core collector, which this proposal refers to as “Collector Distributions”.  However, the vision does not lay out any requirements for distributions, or describe which changes distributions can make.

As of november, 2020, [Splunk](https://github.com/signalfx/splunk-otel-collector), [AWS](https://aws.amazon.com/otel/), and [GCP](https://github.com/GoogleCloudPlatform/opentelemetry-operations-collector) have already announced or published their own collector distributions.  This proposal assumes that collector distributions are a reality, and aims to preserve a base-line level of compatibility within the newly-forming collector ecosystem.

## Motivation

As an End User, I want to:

* Write my application in a way that is compatible with all collector distributions.
* Combine features from different collector distributions in a single pipeline.
* Extend existing collector pipelines with additional custom features.

As a Collector Core maintainer, I want to:

* Ensure the core collector is compatible with collector distributions.
* Provide a path for adding features to an OpenTelemetry collector pipeline without requiring contributions to the core collector.
* … Thereby allowing us to maintain an appropriately narrow scope for the core collector.

As a Collector Distribution maintainer, I want to:

* Ensure my collector will work with other distributions and with customers using protocols in-scope for conformance.
* Demonstrate to customers and the community that my distribution is compatible with the rest of the OpenTelemetry ecosystem.

## Explanation

Defines a Collector Project Goal:

* **Collector distributions can send and receive telemetry from other distributions**

### Out-of-Scope

* **Configuration**: Distributions may define their own configuration.
* **Ports**: Distributions may expose exporters and receivers on any port.
* **Packaging and Distribution**: Distributions may be packaged however they want, and may support any CPU architectures they wish.

## Internal details

### Define Required Functionality

The goal above implies that distributions must share a minimum set of compatible receivers and exporters.  We will start with the bare-minimum set, which can be expanded in the future: 
* **All collector distributions must be able to receive and transmit OTLP**

### Testing Strategy

Add a “Conformance” [collector testbed](https://github.com/open-telemetry/opentelemetry-collector/tree/master/testbed) test suite, which contains a subset of existing collector correctness tests.  These tests must only exercise common functionality, and may not rely on implementation details of the core collector which are not defined as part of the common functionality.  Tests may need to be updated to ensure they can be easily run against other collector repositories. 

### Conformance Test Versioning

Since conformance tests will evolve over time, conformance will be versioned along with the collector repository releases.  A collector distribution which is “v1.0 Conformant” must pass conformance tests from the collector repository at the v1.0 tag, but is not required to pass at the v1.0 tag.

### Test Result Publishing

To start, collector distributions should add a github badge with the status of conformance testing against their repository.

### Aside: Backward-Compatibility

Backward-compatibility for the collector is defined by the backward compatibility rules of the protocols the collector uses in each version.  Currently, changes to OTLP [SHOULD be backward-compatible](https://github.com/open-telemetry/oteps/blob/master/text/0035-opentelemetry-protocol.md#future-versions-and-interoperability), but aren't strictly required to be.  This proposal considers OTLP backward-compatibility orthogonal, but it is important to note that without it we would need to revise our project goal above to state that interoperability is contingent on OTLP version compatibility.

## Trade-offs and mitigations

A conformance test is one additional test suite for the collector maintainers to maintain.

## Prior art and alternatives

The OpenCensus collector included all exporters, and did not have any "distributions".

[Kubernetes conformance](https://github.com/cncf/k8s-conformance) is a set of tests that can be run on kubernetes clusters to certify that they implement required functionality.

## Open questions

* How should the collector make tests available to collector distributions? 
  * Publish Github Action?
  * Publish CircleCi Orb?

## Future possibilities

In addition to ensuring collectors can communicate, it may also be useful for collectors to have similar user experiences:
* If OpenTelemetry is [granted port(s) by the IANA](https://github.com/open-telemetry/opentelemetry-specification/issues/1148), require using those port(s) as part of conformance.
* Define the configuration which produces a conformant collector.  This wouldn’t preclude extensions to the configuration, but would require that a collector distribution run with the “conformance configuration” passes conformance tests.
* Define conformance services and processors, such as health checks.
* Add additional conformance exporters and receivers.

This initial proposal should be viewed as opt-in by distributions, and has no enforcement for non-conformant collectors.  Once tests are well-established, reliable, and adopted by all existing distributions, we can consider adding enforcement provisions similar to the [trademark usage terms of Kubernetes](https://github.com/cncf/k8s-conformance/blob/master/terms-conditions/Certified_Kubernetes_Terms.md).
