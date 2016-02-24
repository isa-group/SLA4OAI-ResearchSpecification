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
The Check SLA endpoint allows to verify the current state of the SLA for a given service and operation in context: 
(for a given user, or role, organization, time of the date, etc.).

In the essence, the service will respond true or false to notify the provider if it is:

- acceptable to fulfill the request (positive case), or on the contrary,
- not acceptable and then, the request should be denied (negative case).


### Sample Invocation:

```
POST /slaCheck
Content-Type: application/json
Accept: application/json
Authorization Basic Ym9zZ236Ym9zY28=

{
    "aggreement": "papamoscas/v1",
    "scope": "serviceSla908923",
    "userId": "u7862neui3",
    "operation": "/bird/get",
    "ts": "2016-01-12T12:57:37.345Z",
    "organizationId": "acme.com",
    "env": "production"
}
```

The service exposes a **POST** operation over the route `/slaCheck`.

### Authentication
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

### Encoding
JSON is used as the primary representation for messages (MUST). Therefore in the sample the following HTTP headers are required:
```
Content-Type: application/json
Accept: application/json
```

### Request Message
The payload in the body of the request can contains the following fields:

#### Compulsory fields:

| Field Name | Type          | Description  |
| :--------- | :------------:| :------------|
| aggreement | `string`      | **Required** The identifier of the agreement to verify.  |
| userId     | `string`      | **Required** The user identifier for the user requesting the service. Quota or rate-limit are checked for this identity. |
| operation  | `string`      | **Required** The operation identifier requested. |
| ts         | `datetime`    | **Required** The timestamp where the call was initiated. Date encoded as string using ISO 8601 format: `YYYY-MM-DDTHH:mm:ss.sssZ`.  |

#### Optional fields:

| Field Name | Type          | Description  |
| :--------- | :------------:| :------------|
| scope      | `string`      | **Optional** A scope identifier. Depends on the agreement. Used to group operations logically. |
| organizationId | `string`  | **Optional** A unique identifier for the organization of the user. Used for aggregating SLAs per organization if needed.  |
| env        | `string`      | **Optional** Environment data. Sample (`devel`, `qa`, `production`). Allows to discriminate data and SLA for different deployment enviroments.  |


Any other field not listed here can be added for custom extensions. The recommended way of extending with custom properties is
to use the prefix `x-` to avoid collisions with future versions of this standard. 

The server can use any properties sent in the payload. It will require the ones marked as compulsory (MUST) and
it will ignore any other not known (MUST).

 
### Calculation:
Based on request host (caller) information, service will be identified. 
Based on user-id provided, role, tenant, organization, and agremment current plan will be recovered. 
Current SLA for this user is calculated and responded.

The calculation method is out of the scope for this spec and is let open for implementators.

### Positive Response:
If the access to the service is granted, a possitive response is sent.

Sample response:

```
200 OK
Content-Type: application/json
            	
{
    "accept": true,
    "serviceProperties": {
        "quotaLimit": 100,
        "quotaUsed": 80,
        "quotaTimePeriodSec": 60,  //60 secs
    },
    "serviceConfiguration": {
        "codingAlgorithm" : "FAST",
        "bitRate": 192,  
        "maxOptimizationTime" : 100  //(sec)
    },
    "requestedMetrics" : {
    	"requestSize":1, 
        "responseSize":0, //0 -> avoid it
        "responseTime":1 // domain-independent (pre-implemented)
        "optimizacionTime":1, 
        "ProblemSize":1, //domain-specific (plugin necessary)
    }
}
```

### Negative Response:
If the access to the service is not granted, a negative response is sent with an optional information describing 
the reason for the SLA violation.

``` 
200 OK
Content-Type: application/json
 
{
    "accept": false,
    "reason": "Quota limit exceed.",
    "quotaLimit": 100,
    "quotaUsed": 100,
    "quotaTimePeriodSec": 60,
    "awaitBeforeRetrySec": 451   //Operation not allowed in the following 451 secs
}
```
 
Based on the response ``accept`` field value the service will then allow the service execution or 
will deny it using a standard `403 Forbidden` HTTP Error.

If denegation reason is quota enforcement, then the recommendation is to use `429 Too Many Requests` HTTP Error plus 
adding Quota information and reason as metadata into the client response to notify clients the denegation
of service.

### Invalid Messages:
Invalid messages sent will return an explicit error code for rejection.

``` 
400 Bad request
Content-Type: application/json

{
    "error": 400,
    "reason": "Invalid message format."
}

```

### Invalid credentials:
If invalid credentials are provided, a 401 error will be raised.

``` 
401 Unauthorized
Content-Type: application/json

{
    "error": 401,
    "reason": "Invalid credentials provided."
}

```


## 3.2 Metrics Report

POST /slaMetrics	 
Sends one or more measures to an SLA Metric Store Service. Measures can be buffered and send in blocks to fine-tune the production system for real-time metrics versus performance.

Service Authentication: Authentication Basic: appId:apiKey pair

Sample request:  Array of batch measures identifying the events and measures per event and source instance information provided (only once).

POST /slaMetrics
Authentication Basic 20325asW.uNh6yHjMU
Content-Type: application/json

{
  "src" : {
     "host" : "node1234",
     "env" : "qa",
     "cluster" : "cl1.acme.com"
  },
  "measures"
scope: "/papamocas/qa/n1"
metrics:[{
    	"service": "/birds/get", 	
//(Optional: Complete request)
    	
"t": "2016-01-12T12:57:37.345Z",
    	"ellapsedMs": 350,
    	"cpu": 20.5,
    	"result": "200",
    	"userId": "13456789aadfc"
	}{
    //Extra measures (batch blocks. Block-size tuneable for performance vs real time information)
	}]
}

Response:
Format:  application/json
Positive response:
201 Created

Format:  application/json
Negative response:
412 Created
Content-Type: application/json

{
	error: 412,
	reason: "Invalid message format."
    	}

Format:  application/json
Negative response:
401 Unauthorized
Content-Type: application/json

{
	error: 401,
	reason: "Unauthorized. Invalid/missing credentials."
}




## 4. References

1. JSON
2. HTTP Basic Authentication
3. HTTP Bearer Authentication
4. Datetime encoding [ISO 8601](http://www.iso.org/iso/catalogue_detail?csnumber=40874)

