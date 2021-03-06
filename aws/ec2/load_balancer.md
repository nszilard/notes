# [Load balancers](../README.md)

## General

* Spread load across multiple downstream instances
* Seamlessly handle failures for downstream instances
* Regular health checks
* Provides SSL termination __(CLB & ALB)__
* Enforce __stickiness__ with cookies
	* Cookie: `AWSELB`
	* Client redirected to the same instance behind an LB
	* Cookie has an expiration date that can be controlled
	* LB -> Attributes -> Enable stickiness (1sec - 7days)
	* Can bring imbalance
* ELB - AWS managed EC2 Load balancer
	* Types
		* `Classic` (v1) 2009
		* `Application` (v2) 2016
		* `Network` (v2) 2017
	* Can be `Private` _(aka internal)_ and `Public`
* `4xx` Client induced errors
* `5xx` Application induced errors
* `Cross Zone Load Balancing`: Distribute traffic across all registered instances in all enabled AZs
* If expecting an increased load
	* __Pre warming__ via an AWS Support ticket
		* Duration of traffic
		* Expected requests per seconds
		* Size of requests

### Connections

* If the front-end connection uses TCP or SSL, then the back-end connections can use either TCP or SSL.
* If the front-end connection uses HTTP or HTTPS, then the back-end connections can use either HTTP or HTTPS.
* If you use TCP _(layer 4)_ for both front-end and back-end connections, the load balancer __forwards__ the request to the back-end instances __without modifying the headers.__
* If you use HTTP _(layer 7)_ for both front-end and back-end connections, the load balancer parses the headers in the request and __terminates the connection before sending the request__ to the back-end instances.

## SSL

* `Server Order Preference`
	* If enabled, LB will select the first cipher in its list that is also in the client's list of ciphers.
* You can create a load balancer with the following security features
	* `SSL Server Certificates`
	* `SSL Negotiation`
	* `Back-End Server Authentication`
* For old browsers, need to change the security policy to allow weaker ciphers
	* `ELBSecurityPolicy-TLS-1-0-2015-04`

## Application Load Balancer

* __Layer 7__ (HTTP/HTTPS/Websocket)
* URL Based routing (host or path)
* __Does not support static IP, but has a fixed DNS__
* Has port mapping feature -> redirect to any port
	* Multiple application on multi machine (`target groups`)
	* Multiple application on the SAME machine (`containers`)
* Route and Host based load balancing
* Stickiness via cookies
* The application servers don't see the clients' IP directly (its in `X-Forwarded-for` headers)
* Latency: ~400ms
* Adds a `Trace ID` to the header for each request: `X-Amzn-Trace-Id`
* Not integrated with X-Ray _(yet)_

## Network Load Balancer

* __Layer 4__ (TCP)
* __Millions of requests per seconds__ (High performance)
* Supports static & elastic IPs
	* 1 static IP per subnet
* __Doesn't support__ SSL termination
* Low latency: ~100ms
* Directly see clients' IP

## Troubleshooting

* Security groups
* Health checks
* Sticky sessions
* Cross zone balancing enabled
* (Internal LB for private)
* (Deletion protection)

### Error codes

* `200`: Successful
* `4xx`: Unsuccessful at client side
	* `400`: Bad request
	* `401`: Not authorized
	* `403`: Forbidden
* `5xx`: Unsuccessful at server side
	* `500`: Internal server error -> Error with LB
	* `502`: Bad gateway
	* `503`: __Service unavailable__
	* `504`: Gateway timeout

## Cloudwatch

* All metrics are directly pushed to CloudWatch
	* `BackendConnectionErrors`
	* `HealthyHostCount`/`UnHealthyHostCount`
	* `HTTPCode_ELB_4XX` => Malformed/cancelled __client__ request
	* `HTTPCode_ELB_5XX` => __LB/Instance__ errors OR LB cannot parse response
	* `HTTPCode_Backend_2XX` => OK
	* `HTTPCode_Backend_3XX` => Redirected
	* __`HTTPCode_Backend_4XX`__ => A __client error__ response sent from the registered instances.
	* __`HTTPCode_Backend_5XX`__ => A __server error__ response sent from the registered instances.
	* `Latency`
	* `RequestCount`
	* __`SurgeQueueLength`: Max is 1024, at max length connections dropped!__
	* __`SpilloverCount`: Number of connections dropped__

## LB Access Logs

* Encrypted by default
* No extra charge _(just S3 storage)_
* Stored in S3 _(need to enable in the LB `Attributes` - __Both in same region!__)_
* Lot of information __(Time, Client IP, Latencies, Request paths, Server response, Trace id)__

## [Back](../README.md)