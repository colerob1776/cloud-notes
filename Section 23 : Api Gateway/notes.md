# API Gateway - Overview

- Lambda + API Gateway: No infra to manage
- Support for WebSockets
- Handle API versioning
- Handle different envs (dev, prod, ...)
- Handle Auth
- Create API keys, handle request throttling
- Swagger / Open API import to quickle define APIs
- Transform and validate reuqests and responses
- Generate SDK and API specs
- Cache API responses

### Integration High Level

- **Lambda**
    - Invoke lambdas
    - Easy to expose REST API backed by Lambda
- **HTTP**
    - expose HTTP endpoints
    - why? add rate limiting, caching, user auth, API keys, etc...
- **AWS Service**
    - Expose any AWS API through API Gateway
    - Ex: start a Step Function, post a message to SQS
    - Why? add auth, deploy publicly, rate control...

### API Gateway - Endpoint Types

- **Edge-Optimized (default)**: for global clients
    - Requests are routed through the CloudFront Edge locations
    - API Gateway still lives in only one region
- **Regional**:
    - for clients within the same region
    - Could manually combine with CF (more control over caching strategies and distribution)
- **Private**:
    - Can only be accessed from your VPC using an interface VPC endpoint (ENI)
    - Use a resource policy to define access

### API Gateway - Security

- **User Auth through**
    - IAM roles
    - Cognito 
    - Custom Authorizer (Lambda)

- **Custom Domain Name HTTPS** security through intergrations with ACM
    - If using Edge-Optimized, then certificate must be in **us-east-1**
    - If using Regional, the certificate must be in the same region as the API Gateway
    - Must setup CNAME or A-alias in Route 53

# API Gateway Stages and Deployment

### Deployment Stages

- Making changes in the API Gateway does not mean they're effective
- you need to make a "deployment" for them to be in effect
- It's a common source of confusion
- Changes are deployd to "stages"
- Use the naming you like for stages
- Each stage has its own config parameters
- stages can be rolled back as a history of deployment is kept

### Stage Variables

- Stage variables are like env variables for API Gateway
- They can be used in:
    - the Lambda ARN
    - HTTP Endpoint
    - Parameter mapping templates
- Use cases:
    - Configure HTTP endpoints your stages talk to
    - Pass config params to Lambda 
- Stage variables are passed to the "context" object in Lambda
- format ${stageVariables.variableName}

### Stage Variables & Lambda Aliases

- We create a **stage variable** to indicate the corresponding Lambda alias
- Our API gateway will auto invoke the right lambda function

# API Gateway Canary Deployments

- Possibility to enable canary deployments for any stage (usually prod)
- Choose the % of traffic the canary channel receives
- Metrics & Logs are separate
- Possibility to override stage variables for canary
- This is blue/green deployment with Lambda and API Gateway

# API Gateway Integration Types & Mappings

### Integration Types

- Integration type MOCK
    - API Gateway returns a response without sending the request to the backend
- Integration type **HTTP / AWS (Lambda & Services)**
    - You must configure both the integration request and the integration response
    - Setup data mapping using **mapping templates** for the request & response
- Integration Type **AWS_PROXY (Lambda Proxy)**:
    - incoming requests from the client is the input to the Lambda
    - The func is responsible for the logic of request / response
    - **No mapping template, headers, query string params, ... are passed as args**
- Integration Type **HTTP_PROXY**
    - No mapping template
    - The HTTP request is passed to the backend
    - The HTTP response from the backend is forwarded by API Gateway
    - Possibility to add HTTP Headers if need be (ex. API key)

### Mapping Templates (AWS & HTTP Integration)

- Mapping templates can be used to modify request / responses
- Rename / Modify **query string params**
- Modify **body content**
- **Add headers**
- Uses Velocity Template Language (VTL): for loop, if etc...
- Filter output results (remove unnecessary data)
- Content-Type can be set to **application/json** or **application/xml**

# API Gateway Open API

### Open API Spec

- common way of defining REST APIs, using API definition as code
- Import existing OpenAPI 3.0 spec to API Gateway
    - Method
    - Method Request
    - Integration Request
    - Method Response
    - + AWS extentions for API gateway and setup every single option
- Can export current API as OpenAPI spec
- OpenAPI specs can be written in YAML or JSON
- Using OpenAPI we can generate SDK for our apps

### REST API - Request Validation

- You can configure API Gateway to perform basic validation of an API request before proceeding with the integration request
- When the validation fails, API Gateway immediately fails the request
    - returns 400-error response to the caller
- This reduces unnecessary calls to the backend
- Checks: 
    - The required request parameters in the URI, query string, and headers of an incoming request are included and non-blank
    - The application request payload adheres to the configured JSON Schema request model of the method
- setup request validation by importing OpenAPI definitions file

# API Gateway Cacheing

- Default **TTL is 300 seconds** (min: 0s, max: 3600s)
- **caches are defined per stage**
- Possible to override the cache settings **per method**
- Cache encryption option
- Cache capacity between 0.5 GB to 237 GB
- Cache is expensive, makes sense in production, may not make sense in dev / test

### Cache Invalidations

- Able to flush the entire cache immediately
- Clients can invalidate the cache with **header: Cache-Control: max-age=0**
- if you dont impose an InvalidateCache policy, any client can invalidate the API cache

# API Gateway Usage Plans & API Keys

- If you want to make an API availableas an offering ($) to your customers
- **Usage Plan**
    - Who can access one or more deployed API stages and methods
    - how much and how fast they can access them
    - Uses API keys to identify API clients and meter access
    - configure throttling limits and quota limits that are enforced on individual client
- **API Keys**
    - alphanumeric string values to distribute to your customers
    - Can use with usage plans to control access
    - Throttling limits are applied to the API keys
    - Quotas limits is the overall number of maximum requests

### Correct Order for API keys

- **to configure a usage plan**

1. Create one or more APIs, configure the methods to require an API key, and deploy the APIs to stages
2. Generate or import API keys to distribute to application developers who will be using your API
3. Create the usage plan with the desired throttle and quota limits
4. **Associate API stages and API keys with the usage plan**

- Callers of the API must supply an assigned API key in the x-api-key header in requests to the API

# API Gateway Monitoring, Logging & Tracing

### Logging & Tracing

- **CloudWatch Logs**
    - log contains info about request/response body
    - enable CW logging at the Stage level (with log level - ERROR, DEBUG, INFO)
    - Can override settings on a per API basis
- **X-Ray**
    - enable tracing to get extra info about requests in API Gateway
    - X-Ray API Gateway + Lambda gives you the full picture

### CloudWatch Metrics

- Metrics are by stage, Possibility to enable detailed metrics
- **CacheHitCount** & **CacheMissCount**: efficiency of the cache
- **Count**: the total number of API requests in a given period
- **IntegrationLatency**: time between when API Gateway relays a request to the backend and when it receives a response from the backend
- **Latency**: the time b/w when API Geteway receives a request from a client and when it returns a response to the client. The latency includes the integration latency and other API Gateway overhead
- **4XXError** (client-side) & **5XXError** (server-side)

### Throttling

- **Account Limit**
    - API Gateway throttles requests at 10000 rps across all API
    - Soft limit that can be increased upon request
- In case of throttling => **429 Too Many Requests** (retriable error)
- Can set **Stage limit & Method limits** to improve performance
- Or you can define **Usage Plans** to throttle per customer

- **Just like Lambda Concurrency, one API that is overloaded, if not limited, can cause the other APIs to be throttled**

### Errors

- **4xx means Client Errors**
- **5xx means Server Error**

# API Gateway CORS

- CORS must be enabled when you receive API calls from another domain
- The OPTIONS pre-flight must contain the following headers:
    - Access-Control-Allow-Methods
    - Access-Control-Allow-Headers
    - Access-Control-Allow-Origin
- CORS can be enabled through the console

# API Gateway Auth

### Security **IAM Permissions**

- Create an IAM policy auth and attach to User / Role
- **Authentication = IAM | Authorization = IAM Policy**
- Good to provide access within AWS 
- Leverages "Sig v4" capability where IAM credential are in headers

### Resource Policies

- **Resource Policies** (similar to Lambda Resource Policies)
- **Allow for Cross Account Access (combined with IAM Security)**
- Allow for specific IPs
- Allow for VPC Endpoint

### Security **Cognito User Pools**

- Cognito fully manages user lifecycle, token expires automatically
- API gateway verifies identity automatically from Cognito
- No custom implementation required
- **Authentication = Cognito User Pools | Authorization = API Gateway Methods**

### Security **Lambda Authorizer**

- **Token-based authorizer** (bearer token) - ex JWT or Oauth
- A **request parameter-based** Lambda authorizer (**headers, query string**, stage var)
- Lambda must return an IAM policy for the user, result policy is cached
- **Authentication = External | Authorization = Lambda Function** 

### Security Summary

- **IAM**
    - Great for users / roles already within your AWS account, **+ resource policy for cross account**
    - Handle auth
    - Leverages Sig v4
- **Custom Authorizer**:
    - Great for 3rd party tokens
    - Very flexible in terms of what IAM p0licy is returned
    - Handle Auth verification + Authorization in the Lambda function
    - Pay per Lambda invocation, results are cached
- **Cognito User Pool**:
    - You manage your own user pool (can be backed by Facebook, Google login, etc...)
    - No need to write any custom code
    - Must implement authorization in the backend

# API Gateway Rest API vs HTTP API

- **HTTP APIs**
    - low-latency, cost-effective, Lambda Proxy, HTTP proxy APIs and private integrations
    - Support for OIDC and OAuth 2.0 auth, and built in support for CORS
    - No usage plans and API keys
    - much cheaper than REST
- **REST APIs**
    - All features (except Native OpenID Connect / OAuth 2.0)

# API Gateway Websocket API

### Overview

- useful in **real-time apps**

### Routing

- incoming JSON messages are routed to different backends
- If no routs => sent to $default
- You request a *route selection expression* to select the field on JSON to route from
- Sample expression: **$request.body.action**
- The result is evaluated against the route keys available in your API Gateway
- The route is then connected to the backend you've setup through API Gateway

# API Gateway - Architecture

- create a single interface for all the microservices in your company
- Use API endpoints with various resources
- Apply a simple domain name and SSL certs
- Can apply forwarding and transformation rules at the API Gateway level