# SLA for Open API Initiative Specification

### Version 0.1

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" 
in this document are to be interpreted as described in [RFC 2119](http://www.ietf.org/rfc/rfc2119.txt).

The **SLA4OAI** specification is licensed under [The Apache License, Version 2.0](http://www.apache.org/licenses/LICENSE-2.0.html).

## Introduction
SLA4OAI is an open source standard for describing SLA in APIs.

Based on the standards proposed by the [OAI](https://openapis.org/specification), SLA4OAI builds on top of it to add an optional profile
for the definition of SLA (Service Level Agreements) on the APIs.

This SLA definition in a neutral vendor flavour will allow to foster innovation in the area where APIs expose and documents its SLA, 
API Management tools can import and measure such key metrics and composed SLAs for composed services aggregated way in a standard way.

## Revision History

	Version  | Date         | Notes
	0.1      | 2015-12-18   | Initial Draft 

## Definitions

### def1
1
### def2
2

## Specification
A SLA4API document is build as an extension to a given Swagger 2.0 document. This extension will add a SLA definition to all or part of the services exposed in 
the API.
A full SLA is composed of the following parts:

1. Metrics
2. Agreement Terms
	- Service Scope
		- Service Location
		- Terms of Service
	- Tracked Properties
	- Guarantee Terms

### Metrics
todo...

### Agreement Terms
todo...