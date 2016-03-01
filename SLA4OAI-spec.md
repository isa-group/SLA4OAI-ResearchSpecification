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

### 1.1 Contents

A. This specification describes the format for defining SLAs ove APIs in a neutral technology way.

B. The **sla0** [spec](./operationalServices.md) provides a runtime standard API for measure and SLA enforcement defined in (A). 

C. Reference implementations are provided to show (A) and (B) in a real examples:
   1. Server side implementations: [sla-service-mock](https://sla-service-mock.herokuapp.com/), governify/v6
   2. Client side implementations: plugins for generated [Hivepod](https://www.hivepod.io) backends. 


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
A **SLA4API** document is build as an extension to a given OpenAPI document. This extension will add a SLA definition to all or 
part of the services exposed in the API.
A full SLA definition is a JSON or a YAML document composed of the following structure.

### 3.1 Reference from an OpenAPI document
To declare a given API exposes an SLA, the OpenAPI description is extended with an optional attribute 
`x-sla` inside the `info` object. The value contains an URI pointing tho the document describing the SLA. 

**Example:**
```
swagger: '2.0'
info:
    title: CSPWeb Reasoner
    description: Solver for CSP Problems
    version: "1.0.0"
    x-sla: ./csp-sla.yml
```

### 3.2 SLA Document Format
The SLA document must conform to the following constraints. 

| Field Name     | Type          | Description  |
| :------------- | :------------:| :------------|
| sla            | `string`      | **Required** Indicates the version of the sla format `='1.0'`. |
| api            | `uri`         | **Required** Indicates an URI (absolute or relative) describing the API to instrument described in the OpenAPI format. |
| infrastructure | `InfrastructureObject` | **Required** Provides information about tooling used for SLA storage, calculation, governance, etc. |
| provider       | `ProviderObject`       | **Optional** Provider information: data about the owner/host of the API. |
| pricing        | `PricingObject`        | **Optional** Pricing general data. |
| metrics        | `MetricsObject`        | **Required** A list of metrics to use in the context of the SLA. |
| availability   | `string`               | **Optional** Availability of the service expressed via time slots using the ISO 8601 time intervals format. |
| plans          | `PlansObject`          | **Required** A set of plans to define different SLA per plan. |
----------------------

#### 3.2.1 InfrastructureObject
The infrastructure object describes the operational tooling to use in the service execution. 

| Field Name     | Type          | Description  |
| :------------- | :------------:| :------------|
| manager        | `uri`         | **Optional** Location of the SLA manager used for SLA governance. |
| checker        | `uri`         | **Required** Location of the SLA Check service accordingly to the [sla0](./operationalServices.md) spec. |
| store          | `uri`         | **Required** Location of the SLA data storage service accordingly to the [sla0](./operationalServices.md) spec. |
----------------------

#### 3.2.2 ProviderObject
| Field Name     | Type          | Description  |
| :------------- | :------------:| :------------|
| name           | `string`      | **Required** Name of the provider of the service. |
----------------------

#### 3.2.3 PricingObject
Describes the general information of the pricing of the API.

| Field Name     | Type          | Description  |
| :------------- | :------------:| :------------|
| cost           | `number`      | **Optional** Cost associated to this service. Defaults to `0` if unspecified. |
| currency       | `string`      | **Optional** Currency used to express the cost. Supported currency values are expressed in ISO 4217 format. Samples: `USD`, `EUR`, or `BTC` for US dollar, euro, or bitcoin, respectively. Defaults to `USD` if unspecified. |
| billingCycle   | `string`      | **Optional** Period used for billing. Supported values are: - `onepay` Unique payment before start using the service. - `daily` Billing at end of the day. - `weekly` Billing at end of the week. - `monthly` Billing at end of the month. - `quartely` Billing at end  of the quarter. -  `yearly` Billing at end of the year. Default to `monthly` if unspecified. |
----------------------

#### 3.2.4 MetricsObject
Contains definitions of metrics with name, types and descriptions.

References can be used to reuse definitions of pre-existing metrics.

*TBD*

#### 3.2.5 PlansObject
Contains a list of plans describing different Level of Service and prices.

| Field Pattern  | Type          | Description  |
| :------------- | :------------:| :------------|
| {planName}     | `PlanObject`  | Describes a usage plan for the API with its associate costs, availability and warranties. |
----------------------

#### 3.2.6 PlanObject
Describes a plan in full.

| Field Name     | Type            | Description  |
| :------------- | :--------------:| :------------|
| configuration  | `Object`        | **Optional** Configuration parameters for the service tailored for the plan. |
| availability   | `string`        | **Optional** Availability of the service for this plan expressed via time slots using the ISO 8601 time intervals format. |
| pricing        | `PricingObject` | **Optional** Specific pricing data for this plan. Overrides general pricing data defined before. |
| limits         | `LimitsObject`  | **Optional** Defines the limits for the service on the current plan. |
| guarantees     | `[GuaranteeObject]` | **Optional** Array of warranties in the current plan. |
----------------------

#### 3.2.7 LimitsObject
*TBD*

#### 3.2.8 GuaranteeObject

*TBD*

## 4. Expressions 
*TBD* Supported expressions and BNF
### 4.1  Supported expressions
*TBD* Describe: Operators, data-types supported and predefined identifiers (and semantics)
### 4.2  Expression BNF
Grammar for expressions.

-----------------
*TODO*

## 5. References

1. Key words for use in RFCs to Indicate Requirement Levels. [RFC 2119](http://www.ietf.org/rfc/rfc2119.txt).
2. [JSON](http://www.json.org)
3. [YAML](http://yaml.org/spec)
4. Datetime encoding. [ISO 8601](http://www.iso.org/iso/catalogue_detail?csnumber=40874)
5. [OpenAPI Specification](https://openapis.org/specification)
6. Currency codes. [ISO 4217](https://en.wikipedia.org/wiki/ISO_4217)