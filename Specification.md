# SLA for Open API Initiative Specification

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL"
in this document are to be interpreted as described in [RFC 2119](http://www.ietf.org/rfc/rfc2119.txt).

The **SLA4OAI** specification is licensed under [The Apache License, Version 2.0](http://www.apache.org/licenses/LICENSE-2.0.html).

## Revision History

|Version  | Date         | Notes                  |
|:------- |:------------ |:---------------------- |
| 0.1     | 2015-12-18   | Initial draft.         |
| 0.2     | 2016-05-05   | Incorporate Review 01. |
| 0.3     | 2016-05-10   | Incorporate Review 02. |
| 0.4     | 2016-05-23   | Incorporate Review 03. |
| 0.5     | 2016-05-20   | Incorporate Review 04. |
| 0.6     | 2016-05-24   | Incorporate Review 05. |
| 0.7     | 2016-06-07   | Incorporate Review 06. |
| 0.8     | 2016-06-22   | Incorporate Review 07. |
| 0.9     | 2016-07-10   | Incorporate Review 08. |

## 1. Introduction
**SLA4OAI** is an open source standard for describing SLA in APIs.

Based on the standards proposed by the [OAI](https://openapis.org/specification), SLA4OAI builds on top of it to add
an optional profile for defining SLA (Service Level Agreements) over the APIs.

This SLA definition in a neutral vendor flavour will allow to foster innovation in the area where APIs expose and documents its SLA,
API Management tools can import and measure such key metrics and composed SLAs for composed services aggregated way in a standard way.

### 1.1 Contents

A. This specification describes the format for defining SLAs ove APIs in a neutral technology way.

B. The **Basic SLA Managment Service** [spec](./operationalServices.md) provides a basic not-normative runtime proposal API for measure and SLA enforcement defined in (A).

C. Reference implementations are provided to show (A) and (B) in a real examples:

   1. Server side implementations: [sla-service-mock](https://sla-service-mock.herokuapp.com/), governify/v6   
   2. Client side implementations: plugins for generated [Hivepod](https://www.hivepod.io) backends.

## 2. Execution Model
The execution model for SLA assumes as little as possible about the system to track the SLA to make it applicable in many scenarios.

!["Figure 1. Basic deployment for an API."](https://github.com/isa-group/SLA4OAI-Specification/blob/master/images/microservice.svg)

*Figure 1. Basic deployment for an API.*

Figure 1 shows a basic production deployment for a service exposing an API enhancement with a load-balancer and service redundancy
for fault tolerance.

In order to instrument for SLA enforcement, some new services can be added to the picture, these services can be grouped into one unit called *SLA Manager* or it can be one or more independent services with different elasticity and scalability.

The roles considered are:

1. Service to instrument (single node or replicated)
2. Load balancer (optional)
3. SLA Manager (for SLA tracking)

Following the principle of Separation of Concerns, the `SLA Manager` exposes a simple API for tracking SLA and reporting metrics.
The service to be instrumented could be deployed as a single instance, cluster, with or without a front load balancer, etc.

Figures 2, 3 and 4 shows different possible configurations deployments.

!["Figure 2. Stand-alone deployment."](https://github.com/isa-group/SLA4OAI-Specification/blob/master/images/standalone-deployment.svg)
<img src="https://github.com/isa-group/SLA4OAI-Specification/blob/master/images/standalone-deployment.svg">

*Figure 2. Stand-alone deployment.*

Stand-alone deployments are typical in development scenarios where debugging is prioritized over high availiability.
In this scenario load-balancers are not involved and service is provided from a single node.

!["Figure 3. Cluster with whitebox load-balancer deployment."](https://github.com/isa-group/SLA4OAI-Specification/blob/master/images/whitebox-deployment.svg)

*Figure 3. Cluster with whitebox load-balancer deployment.*

A cluster with a whitebox load-balancer contain more than one nodes for the API and a load-balancer we can parametrize
to verify the current state of the SLA on requests. Based on the state, the balancer can deny or allow the service.
The balancer is only responsible for checking SLA status, all metrices information gathered in the service node.

!["Figure 4. Cluster with blackbox load-balancer deployment."](https://github.com/isa-group/SLA4OAI-Specification/blob/master/images/blackbox-deployment.svg)

*Figure 4. Cluster with blackbox load-balancer deployment.*

On the other hand, A cluster with a blackbox load-balancer cannot be parametrized for SLA checking. Therefore, SLA check and SLA
enforce policies must be executed as middleware in the entrypoint for each service node.

Depending on the environment used: local development environment, private or pubic cloud, IaaS or PaaS you will find these different
scenarios where an API is served.

## 3. Lifecycle of Agreement

The lifecycle of any SLA agreement has two phases:

1. The service provider design the levels (plans) of the service, then define the **Plans** document. See an example at [petstore-plans.yml](./samples/petstore/petstore-plans.yml).
2. The consumer choose the levels (plans) he need, then both provider and consumer agree the **SLA** document. See an example at [pro-petstore-sla.yml](./samples/petstore/pro-petstore-sla.yml).

!["Figure 5. Lifecycle of Agreement."](https://github.com/isa-group/SLA4OAI-Specification/blob/master/images/lifecycle-agreement.svg)

*Figure 5. Lifecycle of Agreement.*

## 4. Samples

### 4.1 Petstore
This sample illustrates a simple Resource based API for Pets database.

- The full OpenAPI description is provided in [petstore-service.yml](./samples/petstore/petstore-service.yml).
- The service provides a template SLA described in [petstore-plans.yml](./samples/petstore/petstore-plans.yml).
- The provider and consumer agree the SLA described in [pro-petstore-sla.yml](./samples/petstore/pro-petstore-sla.yml).


## 5. Specification
A **SLA4API** document is build as an extension to a given OpenAPI document. This extension will add a SLA definition to all or
part of the services exposed in the API.
A full SLA definition is a JSON or a YAML document composed of the following structure.

### 5.1 Reference from an OpenAPI document
To declare a given API exposes an SLA, the OpenAPI description is extended with an optional attribute
`x-sla` inside the `info` object. The value contains an URI pointing to the document describing the SLA.

**Example:**

```
{
   "swagger": "2.0",
   "info":{
      "title": "Swagger Petstore (Simple)",
      "description": "A sample API that uses a petstore as an example to demonstrate features in the swagger-2.0 specification",
      "version": "1.0.0",
      "x-sla": "./petstore-plans.yml"
   }
}
```

```
swagger: '2.0'
info:
    title: "Swagger Petstore (Simple)"
    description: "A sample API that uses a petstore as an example to demonstrate features in the swagger-2.0 specification"
    version: "1.0.0"
    x-sla: ./petstore-plans.yml
```

### 5.2 SLA Document Schema
#### 5.2.1 SLA Object
The SLA Object must conform to the following constraints.

| Field Name     | Type                                                                 | Description  |
| :------------- | :------------------------------------------------------------------- | :----------- |
| context        | [`ContextObject`](#markdown-header-522-contextobject)                | **Required** Holds the main information of the SLA context. |
| infrastructure | [`InfrastructureObject`](#markdown-header-524-infrastructureobject)  | **Required** Provides information about tooling used for SLA storage, calculation, governance, etc. |
| pricing        | [`PricingObject`](#markdown-header-525-pricingobject)                | **Optional** Global pricing data. |
| metrics        | [`MetricsObject`](#markdown-header-526-metricsobject)                | **Required** A list of metrics to use in the context of the SLA. |
| plans          | [`PlansObject`](#markdown-header-528-plansobject)                    | **Optional** A set of plans to define different service levels per plan. |
| quotas         | [`QuotasObject`](#markdown-header-5210-quotasobject)                  | **Optional** Global quotas, these are the default quotas, later each plan can be override it. |
| rates          | [`RatesObject`](#markdown-header-5211-ratesobject)                    | **Optional** Global rates, these are the default rates, later each plan can be override it. |
| guarantees     | [`GuaranteesObject`](#markdown-header-5212-guaranteesobject)         | **Optional** Global guarantees, these are the default guarantees, later each plan can be override it. |
| configuration  | [`ConfigurationsObject`](#markdown-header-5218-configurationsobject) | **Optional** Define the default configurations, later each plan can be override it. |

#### 5.2.2 ContextObject
Holds the main information of the SLA context.

| Field Name     | Type                                                                 | Description  |
| :------------- | :------------------------------------------------------------------- | :----------- |
| id             | `string`                                                             | **Required** The identification of the SLA context. |
| version        | `string`                                                             | **Required** Indicates the version of the sla format `='1.0'`. |
| api            | `uri`                                                                | **Required** Indicates an URI (absolute or relative) describing the API to instrument described in the OpenAPI format. |
| type           | `string`                                                             | **Required** The type of SLA based on the [Lifecycle of Agreement](#markdown-header-3-lifecycle-of-agreement) (`plans` or `instance`). |
| provider       | `string`                                                             | **Optional** Provider information: data about the owner/host of the API. This field is **required** in case of the context type is `instance`. |
| consumer       | `string`                                                             | **Optional** Consumer information, data about the entity that consumes the service. This field is **required** in case of the context type is `instance`. |
| validity       | [`ValidityObject`](#markdown-header-523-validityobject)              | **Optional** Availability of the service expressed via time slots. This field is **required** in case of the context type is `instance`. |

**Example:**

```
{
  "context": {
    "id": "PetPlans",
    "version": 1,
    "api": "https://sla-petstore.herokuapp.com/api/swagger.json",
    "type": "instance",
    "provider": "ISAGroup",
    "consumer": "tenant1",
    "validity": {
      "effectiveDate": "2016-01-12T12:57:37.345Z",
      "expirationDate": "2017-01-12T12:57:37.345Z"
    }
  }
}
```

```
context:
  id: PetPlans
  version: 1.0
  api: https://sla-petstore.herokuapp.com/api/swagger.json
  type: instance
  provider: ISAGroup
  consumer: tenant1
  validity:
    effectiveDate: 2016-01-12T12:57:37.345Z
    expirationDate: 2017-01-12T12:57:37.345Z
```

#### 5.2.3 ValidityObject
Holds the availability of the service expressed via time slots.

| Field Name     | Type             | Description  |
| :------------- | :--------------- | :----------- |
| effectiveDate  | `string`         | **Required** The starting date of the SLA agreement using the [ISO 8601](https://www.w3.org/TR/NOTE-datetime) time intervals format. |
| expirationDate | `string`         | **Optional** The expiration date of the SLA agreement using the [ISO 8601](https://www.w3.org/TR/NOTE-datetime) time intervals format. |

**Example:**

```
{
  "validity": {
    "effectiveDate": "2016-01-12T12:57:37.345Z",
    "expirationDate": "2017-01-12T12:57:37.345Z"
  }
}
```

```
validity:
  effectiveDate: 2016-01-12T12:57:37.345Z
  expirationDate: 2017-01-12T12:57:37.345Z
```

#### 5.2.4 InfrastructureObject
The infrastructure object describes the operational tooling to use in the service execution.

| Field Name     | Type          | Description  |
| :------------- | :------------:| :------------|
| supervisor     | `uri`         | **Required** Location of the SLA Check service accordingly to the [Basic SLA Managment Service](./operationalServices.md) spec. |
| monitor        | `uri`         | **Required** Location of the SLA Metrics endpoint accordingly to the [Basic SLA Managment Service](./operationalServices.md) spec. |
| {name}         | `uri`         | **Optional** Optional endpoints of SLA governance infrastructure. |

**Example:**

```
{
   "infrastructure":{
      "supervisor": "http://supervisor.sla4oai.governify.io/v1/",
      "monitor": "http://monitor.sla4oai.governify.io/v1/"
   }
}
```

```
infrastructure:
  supervisor: "http://supervisor.sla4oai.governify.io/v1/"
  monitor: "http://monitor.sla4oai.governify.io/v1/"
```

#### 5.2.5 PricingObject
Describes the general information of the pricing of the API.

| Field Name     | Type          | Description  |
| :------------- | :------------:| :------------|
| cost           | `number`      | **Optional** Cost associated to this service. Defaults to `0` if unspecified. |
| currency       | `string`      | **Optional** Currency used to express the cost. Supported currency values are expressed in ISO 4217 format. Samples: `USD`, `EUR`, or `BTC` for US dollar, euro, or bitcoin, respectively. Defaults to `USD` if unspecified. |
| billing        | `string`      | **Optional** Period used for billing. Supported values are: - `onepay` Unique payment before start using the service. - `daily` Billing at end of the day. - `weekly` Billing at end of the week. - `monthly` Billing at end of the month. - `quartely` Billing at end  of the quarter. -  `yearly` Billing at end of the year. Default to `monthly` if unspecified. |

**Example:**

```
{
   "pricing":{
      "cost": 0,
      "currency": "euro",
      "billing": "monthly"
   }
}
```

```
pricing:
  cost: 0
  currency: "euro"
  billing: "monthly"
```

#### 5.2.6 MetricsObject
Contains definitions of metrics with name, types and descriptions. References can be used to reuse definitions of pre-existing metrics.

| Field Name     | Type                                                | Description  |
| :------------- | :-------------------------------------------------- | :----------- |
| {name}         | [`MetricObject`](#markdown-header-527-metricobject) | **Optional** Definitions of metrics with name, types and descriptions. |
| {name}         | `object {"$ref": uri}`                              | **Optional** Reference to pre-existing metrics in external file. |

**Example:**

```
{
  "metrics": {
    "animalTypes": {
      "type": "integer",
      "format": "int64",
      "description": "Number of different animal types.",
      "resolution": "check"
    },
    "requests": {
      "$ref": "./metrics.yml#request"
    }
  }
}
```

```
metrics:
  animalTypes:
    type: integer
    format: int64
    description: Number of different animal types.
    resolution: check
  requests:
    $ref: ./metrics.yml#request
```

In the above example, we refered to a pre-existing metrics [metrics.yml](./samples/petstore/metrics.yml) which has some predefined metrics like `requests` and `avgResponseTimeMs`.

#### 5.2.7 MetricObject
Contains definitions of metric with type, description and unit of the metric.

| Field Name     | Type                                          | Description  |
| :------------- | :-------------------------------------------- | :------------|
| type           | `string`                                      | **Required** This is the metric type accordingly to [the OAI spec format column](https://github.com/OAI/OpenAPI-Specification/blob/master/versions/2.0.md#data-types). |
| format         | `string`                                      | **Optional** The extending format for the previously mentioned type. See [Data Type Formats](https://github.com/OAI/OpenAPI-Specification/blob/master/versions/2.0.md#dataTypeFormat) for further details. |
| description    | `string`                                      | **Optional** A brief description of the metric. |
| unit           | `string`                                      | **Optional** The unit of the metric. |
| resolution     | `enum [check, consumption]`                   | **Optional** Determine when this matric will be resolved. If value is `check` the metric will be sent before the calculation of sla state, else (`consumption`) the metric will be sent after consumption. |

**Example:**

```
{
  "animalTypes": {
    "type": "integer",
    "format": "int64",
    "description": "Number of different animal types.",
    "resolution": "check"
  }
}
```

```
animalTypes:
  type: integer
  format: int64
  description: Number of different animal types.
  resolution: check
```


#### 5.2.8 PlansObject
Contains a list of plans describing different level of service and prices.

| Field Pattern  | Type                                            | Description  |
| :------------- | :---------------------------------------------- | :----------- |
| {planName}     | [`PlanObject`](#markdown-header-529-planobject) | Describes a usage plan for the API with its associate costs, availability and guarantees. |

**Example:**

```
{
  "plans": {
    "free": {
      "rates": {
        "/pets/{id}": {
          "get": {
            "requests": [
              {
                "max": 1,
                "period": "secondly"
              }
            ]
          }
        }
      }
    },
    "pro": {
      "pricing": {
        "cost": 50
      },
      "quotas": {
        "/pets": {
          "get": {
            "requests": [
              {
                "max": 20,
                "period": "secondly"
              }
            ]
          }
        }
      }
    }
  }
}
```

```
plans:
  free:
    rates:
      /pets/{id}:
        get:
          requests:
            - max: 1
              period: secondly
  pro:
    pricing:
      cost: 50
    quotas:
      /pets:
        get:
          requests:
            - max: 20
              period: secondly
```

#### 5.2.9 PlanObject
Describes a plan in full.

| Field Name     | Type                                                                 | Description  |
| :------------- | :------------------------------------------------------------------- | :----------- |
| configuration  | [`ConfigurationsObject`](#markdown-header-5218-configurationsobject) | **Optional** Configuration parameters for the service tailored for the plan. |
| availability   | `string`                                                             | **Optional** Availability of the service for this plan expressed via time slots using the ISO 8601 time intervals format. |
| pricing        | [`PricingObject`](#markdown-header-525-pricingobject)                | **Optional** Specific pricing data for this plan. Overrides default pricing data defined before. |
| quotas         | [`QuotasObject`](#markdown-header-5210-quotasobject)                  | **Optional** Specific quotas data for this plan. Overrides default quotas data defined before. |
| rates          | [`RatesObject `](#markdown-header-5211-ratesobject)                   | **Optional** Specific rates data for this plan. Overrides default rates data defined before. |
| guarantees     | [`GuaranteesObject`](#markdown-header-5212-guaranteesobject)         | **Optional** Specific guarantees data for this plan. Overrides default guarantees data defined before. |


**Example:**

```
{
  "pro": {
    "pricing": {
      "cost": 50
    },
    "configuration": {
      "filteringType": "multipleTags"
    },
    "quotas": {
      "/pets": {
        "get": {
          "requests": [
            {
              "max": 20,
              "period": "secondly"
            }
          ]
        }
      }
    },
    "guarantees": {
      "global": {
        "global": [
          {
            "objective": "avgResponseTimeMs <= 250",
            "period": "daily",
            "window": "dynamic"
          }
        ]
      }
    }
  }
}
```

```
pro:
  pricing:
    cost: 50
  configuration:
    filteringType: multipleTags
  quotas:
    /pets:
      get:
        requests:
          - max: 20
            period: secondly
  guarantees:
    global:
      global:
        - objective: avgResponseTimeMs <= 250
          period: daily
          window: dynamic
```

#### 5.2.10 QuotasObject
Contains a map from name to PathObject describing the quota limits for the service on the current plan.

| Field Pattern  | Type                                             | Description  |
| :------------- | :----------------------------------------------- | :----------- |
| {pathName}     | [`PathObject`](#markdown-header-5215-pathobject) | **Optional** Describes the API endpoint path quota configurations. |


**Example:**

```
{
  "quotas": {
    "/pets": {
      "get": {
        "requests": [
          {
            "max": 20,
            "period": "secondly"
          }
        ]
      }
    }
  }
}
```

```
quotas:
  /pets:
    get:
      requests:
        - max: 20
          period: secondly
```

#### 5.2.11 RatesObject
Contains a map from name to PathObject describing the rate limits for the service on the current plan.

| Field Pattern  | Type                                             | Description  |
| :------------- | :----------------------------------------------- | :----------- |
| {pathName}     | [`PathObject`](#markdown-header-5215-pathobject) | **Optional** Describes the API endpoint path rate configurations. |


**Example:**

```
{
  "rates": {
    "/pets/{id}": {
      "get": {
        "requests": [
          {
            "max": 1,
            "period": "secondly"
          }
        ]
      }
    }
  }
}
```

```
rates:
  /pets/{id}:
    get:
      requests:
        - max: 1
          period: secondly
```

#### 5.2.12 GuaranteesObject
Contains a map from name to GuaranteeObject describing the guarantees for the service on the current plan.

| Field Pattern  | Type                                                       | Description  |
| :------------- | :--------------------------------------------------------- | :----------- |
| {pathName}     | [`GuaranteeObject`](#markdown-header-5213-guaranteeobject) | **Optional** Describes a guarantee level supported by the plan. |

**Example:**

```
{
  "guarantees": {
    "global": {
      "global": [
        {
          "objective": "avgResponseTimeMs <= 250",
          "period": "daily",
          "window": "dynamic"
        }
      ]
    }
  }
}
```

```
guarantees:
  global:
    global:
      - objective: avgResponseTimeMs <= 250
        period: daily
        window: dynamic
```

#### 5.2.13 GuaranteeObject
Describes a guarantee level supported by the plan.

| Field Name     | Type                                                                         | Description  |
| :------------- | :--------------------------------------------------------------------------- | :----------- |
| {MethodName}   | [`GuaranteeObjectiveObject`](#markdown-header-5214-guaranteeobjectiveobject) |  **Optional** An object describes the guarantee level. |

**Example:**

```
{
  "global": {
    "global": [
      {
        "objective": "avgResponseTimeMs <= 250",
        "period": "daily",
        "window": "dynamic"
      }
    ]
  }
}
```

```
global:
  global:
    - objective: avgResponseTimeMs <= 250
      period: daily
      window: dynamic
```

#### 5.2.14 GuaranteeObjectiveObject
An object describes the guarantee level.

| Field Name     | Type                                           | Description  |
| :------------- | :--------------------------------------------- | :----------- |
| objective      | [`Expression`](#markdown-header-6-expressions) |  **Required** The objective of the guarantee. |
| period         | `string`                                       |  **Optional** The period of the objective (secondly, minutely, hourly, daily, monthly or yearly). |
| window         | `string`                                       |  **Optional** The state of the Objective (dynamic or static) |
| scope          | `string`                                       |  **Optional** The scope of who request the service. |

**Example:**

```
{
  "global": [
    {
      "objective": "avgResponseTimeMs <= 250",
      "period": "daily",
      "window": "dynamic",
      "scope": "account"
    }
  ]
}
```

```
global:
  - objective: avgResponseTimeMs <= 250
    period: daily
    window: dynamic
    scope: account
```

#### 5.2.15 PathObject
The API endpoint path.

| Field Pattern  | Type                                                       | Description  |
| :------------- | :--------------------------------------------------------- | :----------- |
| {methodName}   | [`OperationObject`](#markdown-header-5216-operationobject) | **Optional** the operations attached to this path. |


**Example:**

```
{
  "/pets/{id}": {
    "get": {
      "requests": [
        {
          "max": 1,
          "period": "secondly"
        }
      ]
    }
  }
}
```

```
/pets/{id}:
  get:
    requests:
      - max: 1
        period: secondly
```

#### 5.2.16 OperationObject
The operations attached to the path.

| Field Pattern  | Type                                               | Description  |
| :------------- | :------------------------------------------------- | :----------- |
| {metricName}   | [`LimitObject`](#markdown-header-5217-limitobject) | **Optional** The allowed limits of the request. |


**Example:**

```
{
  "get": {
    "requests": [
      {
        "max": 20,
        "period": "secondly",
        "scope": "account"
      }
    ]
  }
}
```

```
get:
  requests:
    - max: 20
      period: secondly
      scope: account
```

#### 5.2.17 LimitObject
The allowed limits of the request.

| Field Pattern  | Type                           | Description  |
| :------------- | :----------------------------- | :------------|
| max            | `number`                       |  **Required** Max value that can be accepted. |
| period         | `string`                       |  **Optional** The period of the objective (secondly, minutely, hourly, daily, monthly or yearly). |
| scope          | `string`                       |  **Optional** The scope of who request the service. |

**Example:**

```
{
  "max": 20,
  "period": "secondly",
  "scope": "account"
}
```

```
max: 20
period: secondly
scope: account
```

#### 5.2.18 ConfigurationsObject
Configurations description.

| Field Pattern  | Type                           | Description  |
| :------------- | :----------------------------- | :------------|
| {name}         | `string`                       |  **Optional** Configurations description. |

**Example:**

```
{
  "configuration": {
    "filteringType": "multipleTags",
    "xmlFormat": true
  }
}
```

```
configuration:
  filteringType: multipleTags
  xmlFormat: true
```

## 6. Expressions
Guarantee's objective is expressed as a boolean expression which produces a Boolean value when evaluated.

### 6.1  Supported expressions
Currently only simple comparassion operators are supported.

**Supported operators:**

| Operator       | Name of operator               |
| :------------- | :----------------------------- |
| `<`            | less than                      |
| `<=`           | less than or equal to          |
| `==`           | equal to                       |
| `!=`           | not equal to                   |
| `>=`           | greater than or equal to       |
| `>`            | greater than                   |

### 6.2  Expression BNF
Supported expression syntax has a single form: Property + Operator + Value

```
<expression> ::= <variable> <op> <value>
<op>         ::= '<' | '<=' | '==' | '!=' | '>=' | '>'
<value>      ::= <integer> | <string> | <double>
```

## 7. References

1. Keywords for use in RFCs to Indicate Requirement Levels. [RFC 2119](http://www.ietf.org/rfc/rfc2119.txt).
2. [JSON](http://www.json.org)
3. [YAML](http://yaml.org/spec)
4. Datetime encoding. [ISO 8601](http://www.iso.org/iso/catalogue_detail?csnumber=40874)
5. [OpenAPI Specification](https://openapis.org/specification)
6. Currency codes. [ISO 4217](https://en.wikipedia.org/wiki/ISO_4217)
