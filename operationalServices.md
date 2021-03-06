# Basic SLA Management Service: Operational Services Proposal for SLA Checking and Metrics report

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL"
in this document are to be interpreted as described in [RFC 2119](http://www.ietf.org/rfc/rfc2119.txt).

## Revision History

| Version | Date         | Notes                  |
|:------- |:------------ |:---------------------- |
| 0.1     | 2016-02-25   | Initial version.       |
| 0.2     | 2016-05-23   | Incorporate Review 01. |
| 0.3     | 2016-06-07   | Incorporate Review 02. |
| 0.4     | 2016-06-08   | Incorporate Review 03. |
| 0.5     | 2016-06-14   | Incorporate Review 04. |
| 0.6     | 2016-06-22   | Incorporate Review 05. |
| 0.7     | 2016-07-10   | Incorporate Review 06. |
| 0.7.1     | 2016-07-28   | Update markdown syntax. |

## 1. Introduction
This proposal presents an open and not-normative proposal for simple SLA checking and metrics reporting.

The proposal introduces a basic not-normative API to provide the following services:

1. An endpoint for checking the current state of a given SLA (Service level Agreement).
2. An endpoint for reporting metrics to calculate the current state of given SLA.

In this way, using this proposed operational standard called **Basic SLA Management Service**:

1. Different implementations of this service can be provided to define, control, manage, report and track
SLAs in different technologies.
2. Different clients of the API, microservices (or any service instrumented by an SLA) can be tracked from a
third-party service in an standard way if compliant with **Basic SLA Management Service**.

This **Basic SLA Management Service** API is introduced to provide an operational model for SLA checking and metrics reporting for
[SLA4API](README.md) specification.

## 2. Context

![Figure 1. Architecture.](/images/sla0.png?raw=true)

Figure 1. Illustrates the proposed architecture to instrument a service using **Basic SLA Management Service** approach.

The figure shows an online service accepting request from a front-load balancer. After it, a cluster of service
instances with variable nodes are setup #(1..N) to deliver the service with flexibility depending on the
workload and SLA required.

A second service playing the role of **SLA Manager** will be used to outsource the process of

1. checking the current state of the SLA and
2. to report metrics in order to compute and aggregate the current SLA status

In the rest of this document, the endpoints for both use cases are described using Open API style format.
The descriptions will explain the semantics and signatures expected in the service, leaving all the
implementation details open for implementers of this standard.

Finally, some samples and a reference implementation is provided to help implementers to comply with **Basic SLA Management Service**.

## 3. Conventions used

### 3.1 Authentication
The service could be authenticated using one of the following choices:

1. HTTP Basic Authentication using a pair of `keyId:secretId` credentials shared by the service and the client
passed via the HTTP `Authorization` header. (MUST)

    Sample:
    ```
    Authorization Basic Ym9zZ236Ym9zY28=
    ```

2. HTTP Bearer Authentication for OAuth 2.0 (COULD)

    Sample:
    ```
    Authorization Bearer BOm9zZ236Ym9zY92emUhN7m9z3146Ym9zY59
    ```

3. Certificates or any other authentication supported by the implementations (COULD)
4. No authentication at all (recommended only for dev or non-production environments). (COULD)

### 3.2 Message Encoding
JSON is used as the primary representation for messages (MUST). Therefore in the sample the following HTTP headers are required:

```
Content-Type: application/json
Accept: application/json
```
### 3.3 Dates and Datetime representation
Dates and Datetimes formats as defined in ISO-8601:2004 will be used to standardize its encoding.

- Sample date: `2016-05-01` represents the first of May of 2016.
- Sample datetime: `2016-04-30T23:59:59.999Z` representing one millisecond due for the 1st of May of 2016 on Zulu/UTC (Z) zone time.


## 4. Service Endpoints
**Basic SLA Management Service** exposes three endpoints to be described here:

- GET `/tenants` to get the scope definition.
- POST `/check` to check for current SLA state.
- POST `/metrics` to report runtime metrics to the registry.

## 4.1 SLA Scope
The SLA Scope endpoint allows to get the scope definition of the current account either by the account user name or the API key.

### Sample Invocation:
```
GET /tenants?apikey=xo1234
```
or

```
GET /tenants?account=john@tenant.com
```

### Request Message
The API parameters should contains one of the following fields:

#### Compulsory fields:

| Field Name | Type       | Description  |
| :--------- | :--------- | :----------- |
| apikey     | `string`   | The API key that used to authenticate the coming request. |
| account    | `string`   | The user identifier coming from OAuth Provider. |

### Response Message Format

| Field Name      | Type     | Description  |
| :-------------- | :------- | :----------- |
| sla             | `string` | **Optional** The URL identifier of the agreement.  |
| scope           | [`ScopeObject`](#scopeobject-definition) | **Required** The scope identifier for the user requesting the service. Quota or rate-limit are checked for this identity. |
| requestedMetrics | `array` | **Optional** List of the requested metrics that should be provided in the [SLA Check](#42-sla-check), where resolution = check. |

Sample response:

```
200 OK
Content-Type: application/json

{
   "sla":"sla01",
   "scope":{
      "tenant":"tenant1",
      "account":"john@tenant.com"
   },
   "requestedMetrics":[
      "nameLegth"
   ]
}
```

### ScopeObject definition:
| Field Name | Type      | Description  |
| :--------- | :-------- | :------------|
| tenant     | `string`  | **Required** The tenant that this account belong to. |
| account    | `string`  | **Required** The user identifier for this account scope. |

## 4.2 SLA Check
The SLA Check endpoint allows to verify the current state of the SLA for a given service and operation in context for a given scope.

In the essence, the service will respond true or false to notify the provider if it is:

- Acceptable to fulfil the request (positive case), or on the contrary,
- Not acceptable and then, the request should be denied (negative case).


### Sample Invocation:
```
POST /check
Content-Type: application/json
Accept: application/json
Authorization Basic Ym9zZ236Ym9zY28=

{
   "sla":"sla01",
   "ts":"2016-01-12T12:57:37.345Z",
   "resource":"/pets",
   "method":"POST",
   "scope":{
      "tenant":"tenant1",
      "account":"john@tenant.com"
   },
   "metrics":{
      "nameLegth": 12
   },
   "environment":"qa"
}
```

### Request Message
The payload in the body can contains the following fields:

| Field Name | Type          | Description  |
| :--------- | :------------:| :------------|
| sla        | `string`      | **Required** The identifier of the agreement to verify.  |
| ts         | `datetime`    | **Required** The timestamp where the call was initiated. Date encoded as string using ISO 8601 format: `YYYY-MM-DDTHH:mm:ss.sssZ`.  |
| resource   | `string`      | **Required** The resource identifier requested. |
| method     | `string`      | **Required** The HTTP method of the operation. |
| scope      | [`ScopeObject`](#scopeobject-definition) | **Required** The scope identifier for the user requesting the service. Quota or rate-limit are checked for this identity. |
| metrics    | `object`      | **Optional** An object contains the values of the requested metrics came from [SLA Scope](#41-sla-scope). |
| environment | `string`     | **Optional** Environment data. Sample (`devel`, `qa`, `production`). Allows to discriminate data and SLA for different deployment environments.  |


Any other field not listed here can be added for custom extensions. The recommended way of extending with custom properties is
to use the prefix `x-` to avoid collisions with future versions of this standard.

The server can use any properties sent in the payload. It will require the ones marked as compulsory (MUST) and
it will ignore any other not known (MUST).


### Calculation
Based on agreement information, service will be identified.
Based on scope provided, current plan will be recovered.
Current SLA for this scope is calculated and responded.

The calculation method is out of the scope for this spec and is let open for implementers.

#### Pre-defined Metrics
The result of the calculation may provide a list of requested metrics to be measured during the service execution.
This extensibility point allow to add *pre-defined* or *custom* metrics to be gathered after, or during, the service execution.

For example, the NodeJS module [SLA4OAI-node](https://github.com/isa-group/SLA4OAI-node) provides a list of pre-defined metrics at [metrics.yml](https://bitbucket.org/sla4oai/sla4oai-lib/src/master/predefined/metrics.yml).

### Response Message Format
The response message follows the structure:

| Field Name           | Type                | Description  |
| :------------------- | :------------------ | :----------- |
| accept               | `boolean`           | **Required** Indicates if the service is authorize for execution or denied.  |
| quotas               | [`[limit]`](#limit-object) | **Optional** When present, provides some SLA constrains to apply to the current service invocation. Quota limit info can be used to inform the client. |
| rates                | [`[limit]`](#limit-object) | **Optional** When present, provides some SLA constrains to apply to the current service invocation. Rate limit info can be used to inform the client. |
| configuration        | `object`            | **Optional** Provides extra parameters that can affect the service delivery. Quality properties can be setup here to select a given the Quality of Service (QoS). |
| requestedMetrics     | `array`             | **Optional** The supervisor will send all metrics in the agreement as requested metrics. The list should included pre-defined (domain independent) metrics and custom (domain specific) metrics, where resolution = consumption. |
| error                | `integer`           | **Optional** An error type code number if error. |
| reason               | `string`            | **Optional** A description for the error in case of error.  |

### Limit Object
| Field Name | Type        | Description  |
| :--------- | :---------- | :----------- |
| resource   | `string`    | **Required** Name of the resource protected with quota/rate.  |
| method     | `string`    | **Required** The HTTP method for the resource request. |
| metric     | `string`    | **Required** The metric of this quota/rate.  |
| limit      | `integer`   | **Optional** Max of quota/rate for the given resource.  |
| used       | `integer`   | **Optional** Current used quota/rate. |
| awaitTo    | `timestamp` | **Optional** Await time in seconds to await before retrying after a rate limit violation. |

### Positive Response
If the access to the service is granted, a positive response is sent.

Sample response:

```
200 OK
Content-Type: application/json

{
   "accept":true,
   "quotas":[{
      "resource":"pet",
      "method":"GET",
      "metric":"animalType",
      "limit":100,
      "used":100,
      "awaitTo":"2016-01-12T12:57:37.345Z"
   }],
   "rates":[],
   "configuration":{
      "filteringType":"none",
      "xmlFormat":false
   },
   "requestedMetrics":[
      "responseTime",
      "animalType",
      "resourceInstances"
   ]
}
```


### Negative Response
If the access to the service is not granted, a negative response is sent with an optional information describing
the reason for the SLA violation.

```
200 OK
Content-Type: application/json

{
   "accept":false,
   "reason":"Quota limit exceed.",
   "quotas":[{
      "resource":"pet",
      "method":"GET",
      "metric":"animalType",
      "limit":100,
      "used":100,
      "awaitTo":"2016-01-12T12:57:37.345Z"
   }]
}
```

Based on the response ``accept`` field value the service will then allow the service execution or
will deny it using a standard `403 Forbidden` HTTP Error.

If denegation reason is rate limit enforcement, then the recommendation is to use `429 Too Many Requests` HTTP Error plus
adding rate limit information and reason as metadata into the client response to notify clients the denegation
of service.

### Invalid Message
Invalid message sent will return an explicit error code for rejection.

```
400 Bad request
Content-Type: application/json

{
    "error": 400,
    "reason": "Invalid message format."
}

```

### Invalid credentials
If invalid credentials are provided, a 401 error will be raised.

```
401 Unauthorized
Content-Type: application/json

{
    "error": 401,
    "reason": "Invalid credentials provided."
}

```


## 4.3 SLA Metrics
At any moment, a service can collect a set of basic metrics and send them to a data store for aggregation and later consumption.

The SLA Metrics exposes an endpoint for gathering the metrics collected from different nodes.

The API supports buffering. Therefore, metrics can be grouped in batches or sent one by one to fine-tune performance versus real-time SLA tracking.

The service exposes a **POST** operation over the route `/metrics`.

### Sample Invocation:
The following sample shows a metrics store request using the service. An array of batch measures identifying each
of the events, measures per event and source instance information provided.

```
POST /metrics
Authentication Basic 20325asW.uNh6yHjMU
Content-Type: application/json

{
   "sla":"sla01",
   "scope":{
      "tenant":"tenant1",
      "account":"john@tenant.com"
   },
   "sender":{
      "host":"node1234",
      "environment":"qa",
      "cluster":"cl1.acme.com"
   },
   "measures":[
      {
          // measure 1
         "resource":"/pets",
         "method":"GET",
         "result":"200",
         "ts":"2016-01-12T12:57:37.345Z",
         "metrics": {
            "responseTime": 200,
            "animalTypes": 2,
            "resourceInstances": 200
         }
      },
      {
        //measure 2
        //...
      },
      {
        //Measure N
        //Extra measures (batch blocks allowed. Block-size tuneable for performance vs real-time information)
      }
   ]
}
```

### Request Message Format
The payload in the body accepts the following fields:

#### Compulsory fields:

| Field Name | Type          | Description  |
| :--------- | :------------:| :------------|
| sla        | `string`      | **Required** The identifier of the agreement.  |
| scope      | [`ScopeObject`](#scopeobject-definition) | **Required** The scope identifier for the user requesting the service. Quota or rate-limit are checked for this identity. |
| sender     | [`SenderObject`](#senderobject-definition) | **Required** An object describing the source of the metrics.  |
| measures   | [`[MeasureObject]`](#measureobject-definition) | **Required** Array of measures. At least, it must contain one item. |

#### SenderObject definition:
Sender describes the information related to the source of events and metrics. It is described only once in the payload to avoid
unnecessary repetition.

| Field Name | Type          | Description  |
| :--------- | :------------:| :------------|
| host       | `string`      | **Required** The name of the physical node reporting the data. |
| environment | `string`     | **Optional** Environment data. Sample (`devel`, `qa`, `production`). Allows to discriminate data and SLA for different deployment environments.  |
| cluster    | `string`      | **Optional** Cluster name to aggregate services.  |


#### MeasureObject definition:
Each measure structure collects a set of metrics for a service in a given point of time.

| Field Name | Type          | Description  |
| :--------- | :------------:| :------------|
| resource   | `string`      | **Required** The name of the resource the metrics reported belong to. |
| method     | `string`      | **Required** The HTTP method of the operation. |
| result     | `string`      | **Optional** Exit code of the result of the operation.  |
| ts         | `datetime`    | **Required** Timestamp in ISO 8601 format when the event occur.  |
| metrics    | `object`      | **Required** Set of metrics for a service in a given point of time.  |

**Extension metrics:**
Any other field not listed here can be added for custom extensions. The recommended way of extending with custom properties is
to use the prefix `x-` to avoid collisions with future versions of this standard.


### Response Message Format
Response proposed is very concise to indicate acceptation of the payload. It only provide information in case of error.

| Field Name | Type          | Description  |
| :--------- | :------------:| :------------|
| error      | `integer`     | **Optional** An error type code number. |
| reason     | `string`      | **Optional** A description for the error.  |

### Positive Response

```
201 Created
```

### Negative Response
#### Invalid Message
Invalid message sent will return an explicit error code for rejection.

```
400 Bad request
Content-Type: application/json

{
    error: 400,
    reason: "Invalid message format."
}
```

#### Invalid credentials
If invalid credentials are provided, a 401 error will be raised.

```
401 Unauthorized
Content-Type: application/json

{
	error: 401,
	reason: "Unauthorized. Invalid/missing credentials."
}
```


## 5. References

1. Key words for use in RFCs to Indicate Requirement Levels. [RFC 2119](http://www.ietf.org/rfc/rfc2119.txt).
2. [JSON](http://www.json.org)
3. HTTP Basic Authentication. [RFC 2617](https://tools.ietf.org/html/rfc2617)
4. HTTP Bearer Authentication. [RFC 6750](https://tools.ietf.org/html/rfc6750)
5. Datetime encoding. [ISO 8601](http://www.iso.org/iso/catalogue_detail?csnumber=40874)
6. HTTP Status Codes [RFC 7231](http://tools.ietf.org/html/rfc7231#section-6)
7. [IANA HTTP Status Code Registry](http://www.iana.org/assignments/http-status-codes/http-status-codes.xhtml)
