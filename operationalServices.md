# sla0: Operational Services Proposal for SLA Checking and Metrics report

### Version 0.1
The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" 
in this document are to be interpreted as described in [RFC 2119](http://www.ietf.org/rfc/rfc2119.txt).

The **SLA4OAI** specification is licensed under [The Apache License, Version 2.0](http://www.apache.org/licenses/LICENSE-2.0.html).

## 1. Introduction
This proposal presents an open and standard proposal for simple SLA checking and metrics reporting.

The proposal introduces a simple standard API to provide the following services:

1. An endpoint for checking the current state of a given SLA (Service level Agreement).
2. An endpoint for reporting metrics to calculate the current SLA.

In this way, using this proposed operational standard called **sla0**:

1. different implementations of this service can be provided to define, control, manage, report and track 
SLAs in different technologies.
2. different clients of the API, microservices (or any service instrumented by an SLA) can be tracked from a 
third-party service in an standard way if compliant with **sla0**.
   
## 2. Context
![Figure 1](sla0.svg "Figure 1. Architecture")

Figure 1. illustrates the proposed architecture to instrument a service using **sla0** approach.

The figure shows an online service accepting request from a front-load balancer. After it, a cluster of service 
instances with variable nodes are setup #(1..N) to deliver the service with flexibility depending on the 
workload and SLA required.

A second service playing the role of **SLA Manager** will be used to outsource the process of 
   
1. checking the current state of the SLA and
2. to report metrics in order to compute and aggregate the current SLA status 

In the rest of this document, the endpoints for both use cases are described using OpenAPI style format.
The descriptions will explain the semantics and signatures expected in the service, leaving all the 
implementation details open for implementors of this standard.

Finally, some samples and a reference implementation is provided to help implementers to comply with **sla0**.

## 3.1 Check SLA

## 3.2 Metrics Report

## 4. References


