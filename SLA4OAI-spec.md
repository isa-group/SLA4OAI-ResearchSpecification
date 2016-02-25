# SLA for Open API Initiative Specification

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" 
in this document are to be interpreted as described in [RFC 2119](http://www.ietf.org/rfc/rfc2119.txt).

The **SLA4OAI** specification is licensed under [The Apache License, Version 2.0](http://www.apache.org/licenses/LICENSE-2.0.html).

## Revision History

|Version  | Date         | Notes          |
|:--------|:-------------|:---------------|
| 0.1     | 2015-12-18   | Initial draft. |

## 1. Introduction
SLA4OAI is an open source standard for describing SLA in APIs.

Based on the standards proposed by the [OAI](https://openapis.org/specification), SLA4OAI builds on top of it to add 
an optional profile for the definition of SLA (Service Level Agreements) on the APIs.

This SLA definition in a neutral vendor flavour will allow to foster innovation in the area where APIs expose and documents its SLA, 
API Management tools can import and measure such key metrics and composed SLAs for composed services aggregated way in a standard way.

## 2. Samples

### 2.1 CSP WebReasoner
This sample illustrates a simple API for a problem solver. 

- The full OpenAPI description is provided in [csp-service.yml](./samples/CSP-WebReasoner/csp-service.yml).
- The service provides a SLA described in [csp-sla.yml](./samples/CSP-WebReasoner/csp-sla.yml).

### 2.2 Papamoscas
This sample illustrates a simple Resource based API for Birds database. 

- The full OpenAPI description is provided in [papamoscas-service.yml](./samples/papamoscas/papamoscas-service.yml).
- The service provides a SLA described in [papamoscas-sla.yml](./samples/papamoscas/papamoscas-sla.yml).


## 3. Specification
A SLA4API document is build as an extension to a given Swagger 2.0 document. This extension will add a SLA definition to all or part of the services exposed in 
the API.
A full SLA definition is composed of the following parts:





## x. References

1. Key words for use in RFCs to Indicate Requirement Levels. [RFC 2119](http://www.ietf.org/rfc/rfc2119.txt).
2. [JSON](http://www.json.org)
3. Datetime encoding. [ISO 8601](http://www.iso.org/iso/catalogue_detail?csnumber=40874)