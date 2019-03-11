# **msrest** #

[**msrest**](https://github.com/Azure/msrest-for-python) is
Microsoft's runtime library for
[AutoRest](https://github.com/Azure/autorest) generated Python
clients.  And the [AutoRest](https://github.com/Azure/autorest) tool
generates client libraries for accessing RESTful web services.  Input
to AutoRest is a spec that describes the REST API using the [OpenAPI
Specification](https://github.com/OAI/OpenAPI-Specification) format.


## module `msrest.authentication` ##

* `Authentication`(`object`)
* `BasicAuthentication`(`Authentication`)
* `BasicTokenAuthentication`(`Authentication`)
* `OAuthTokenAuthentication`(`BasicTokenAuthentication`)
* `ApiKeyCredentials`(`Authentication`)
* `CognitiveServicesCredentials`(`ApiKeyCredentials`)
* `TopicCredentials`(`ApiKeyCredentials`)

### class `CognitiveServicesCredentials` ###

Hierarchy:
* `msrest.authentication.CognitiveServicesCredentials`
  * `msrest.authentication.ApiKeyCredentials`
    * `msrest.authentication.Authentication`
	
class `Authentication` is the base class of all authentication class.
Its `signed_session()` function accepts a already existing session or
uses `requests.Session()` to create a session if not provided, and
will be overrided by its subclasses to provide a `requests.Session()`
and extra data such as headers and params.

class `ApiKeyCredentials` expects `in_headers` and `in_query` to be
added to `requests.Session.headers` and `requests.Session.params`.

class `CognitiveServicesCredentials` expects `subscription_key` to be
added to the `in_headers` of its superclass `ApiKeyCredentials`, and
finally be added to `requests.Session.headers` as
`Ocp-Apim-Subscription-Key` -- the name of the subscript key in the
Azure service API.

#### class `BasicAuthentication` ####

Hierarchy:
* `msrest.authentication.BasicAuthentication`
  * `msrest.authentication.Authentication`

It is a basic authentication which expects `username` and `password`
to be added to `requests.Session.auth`.


## module `msrest.configuration` ##

* `Configuration`(`RequestHTTPSenderConfiguration`)

### class `Configuration` ###

Hierarchy:
* `msrest.configuration.Configuration`
  * `msrest.universal_http.requests.RequestHTTPSenderConfiguration`
    * `msrest.universal_http.HTTPSenderConfiguration`

`msrest.universal_http.HTTPSenderConfiguration` uses
`configparser.ConfigParser` to load/save configuration from/into a
file.  Configurations include:
* Connection
  - timeout
  - verify
  - cert
* Proxies
  - proxies
  - env_settings
* RedirectPolicy
  - allow
  - max_redirects

`msrest.universal_http.requests.RequestHTTPSenderConfiguration` adds
more configuration:
* RetryPolicy
  - retries
  - backoff_factor
  - max_backoff

`Configuration` also includes variables:
* `credentials`: Holds something like API key
* `keep_alive`: Indicates if keep the session
* `pipeline`: `msrest.pipeline.Pipeline`

Its subclasses which targets at different Azure services such as
`ImageSearchAPIConfiguration` and `FaceClientConfiguration` will add
corresponding endpoint.


## module `msrest.universal_http` ##

* `HTTPSender`(`AbstractContextManager`, `ABC`)
* `HTTPSenderConfiguration`(`object`)
* `ClientRequest`(`object`)
* `HTTPClientResponse`(`object`)
* `ClientResponse`(`HTTPClientResponse`)
* `ClientRedirectPolicy`(`object`)
* `ClientProxies`(`object`)
* `ClientConnection`(`object`)

### class `ClientRequest` ###

It encapsulates the all info needed to make a request, which include
HTTP method verbs, URL, headers and data.


## module `msrest.universal_http.requests` ##

* `HTTPRequestsClientResponse`(`HTTPClientResponse`)
* `RequestsClientResponse`(`HTTPRequestsClientResponse`, `ClientResponse`)
* `BasicRequestsHTTPSender`(`HTTPSender`)
* `RequestsHTTPSender`(`BasicRequestsHTTPSender`)
* `ClientRetryPolicy`(`object`)
* `RequestHTTPSenderConfiguration`(`HTTPSenderConfiguration`)

### class `RequestsClientResponse` ###

Hierarchy:
* `msrest.universal_http.requests.RequestsClientResponse`
  * `msrest.universal_http.requests.HTTPRequestsClientResponse`
    * `msrest.universal_http.HTTPClientResponse`
  * `msrest.universal_http.ClientResponse`
    * `msrest.universal_http.HTTPClientResponse`

`msrest.universal_http.HTTPClientResponse` encapsulates all info for a
response, which include the request to get the response, the content
of the response, status_code, headers, and reason.

`msrest.universal_http.ClientResponse` adds streaming download
interface to `msrest.universal_http.HTTPClientResponse`.

`msrest.universal_http.requests.HTTPRequestsClientResponse` implements
`body()` and `text()` of `msrest.universal_http.HTTPClientResponse` to
return binary and text representation of the response.

`msrest.universal_http.requests.HTTPRequestsClientResponse` implements
the streaming download interface of
`msrest.universal_http.ClientResponse`.

### class `RequestsHTTPSender` ###

Hierarchy:
* `msrest.universal_http.requests.RequestsHTTPSender`
  * `msrest.universal_http.requests.BasicRequestsHTTPSender`
    * `msrest.universal_http.requests.HTTPSender`
	  * `contextlib.AbstractContextManager`
	  * `abc.ABC`

`msrest.universal_http.requests.HTTPSender` is an abstract class
declares the `send()` method which accepts
`msrest.universal_http.ClientRequest` and returns
`msrest.universal_http.ClientResponse`.  And since it is a subclass of
`contextlib.AbstractContextManager`, it inherits `__enter__()` and
`__exit__()`.

`msrest.universal_http.requests.BasicRequestsHTTPSender` receives a
`requests.Session` to keep a HTTP session, and close it when
`__exit__()`.  Its `send()` method calls `requests.Session.request()`
to make a HTTP request.

`msrest.universal_http.requests.RequestsHTTPSender` receives a
`msrest.universal_http.requests.RequestHTTPSenderConfiguration` such
as `msrest.configuration.Configuration` to get all info about a
request.  Its `send()` method will prepare configuration parameters
before calling its superclass's `send()` method.


## module `msrest.pipeline` ##

* `HTTPPolicy`(`abc.ABC`, `Generic[HTTPRequestType, HTTPResponseType]`)
* `SansIOHTTPPolicy`(`Generic[HTTPRequestType, HTTPResponseType]`)
* `_SansIOHTTPPolicyRunner`(`HTTPPolicy`, `Generic[HTTPRequestType, HTTPResponseType]`)
* `Pipeline`(`AbstractContextManager`, `Generic[HTTPRequestType, HTTPResponseType]`)
* `HTTPSender`(`AbstractContextManager`, `ABC`, `Generic[HTTPRequestType, HTTPResponseType]`)
* `Request`(`Generic[HTTPRequestType]`)
* `Response`(`Generic[HTTPRequestType, HTTPResponseType]`)
* `ClientRawResponse`(`object`)

### class `HTTPPolicy` ###

`msrest.pipeline.HTTPPolicy` is a abstract class that declares a
`send()` method which accepts `msrest.pipeline.Request` to send a
request and returns `msrest.pipeline.Response`.  Its `self.next`
variable points to another `msrest.pipeline.HTTPPolicy` so that all
policies are chained together to add additional info one-by-one before
finally do the actual sending.  So it is more like a decorator.

### class `Request` ###

It represents a request in `msrest.pipeline.Pipeline`.  It
encapsulates a request and a context.

### class `Response` ###

It represents a response in `msrest.pipeline.Pipeline`.  It
encapsulates a `msrest.pipeline.Request`, a response and a context.
The context is actually a dict that holds addtional post-processed
field.

### class `ClientRawResponse` ###

It is a wrapper for a request's response, similar to
`msrest.universal_http.ClientResponse`, as compared to
`msrest.pipeline.Response` which is a pipeline response.

### class `_SansIOHTTPPolicyRunner` ###

Hierarchy:
* `msrest.pipeline._SansIOHTTPPolicyRunner`
  * `msrest.pipeline.HTTPPolicy`

It holds a `_policy` variable which is of type
`msrest.pipeline.SansIOHTTPPolicy`.  So it is a wrapper of
`msrest.pipeline.SansIOHTTPPolicy` to make it act as a
`msrest.pipeline.HTTPPolicy`, but with additional modifications which
take place in `on_request()`, `on_response()` methods which will be
called in `send()`.

### class `Pipeline` ###

Hierarchy:
* `msrest.pipeline.Pipeline`
  * `contextlib.AbstractContextManager`

It holds a `_impl_policies` and a `_sender` variable.  `_sender` is of
type `PipelineRequestsHTTPSender` which is a wrapper of
`msrest.universal_http.requests.BasicRequestsHTTPSender`.
`_impl_policies` will wrap all `msrest.pipeline.SansIOHTTPPolicy`s
into `msrest.pipeline.HTTPPolicy` using
`msrest.pipeline._SansIOHTTPPolicyRunner` and chain all policies
together using their `self.next` variable.  In the end of the policy
chain, its `self.next` will be assigned to `_sender`.

Since `msrest.pipeline.Pipeline` is also a
`contextlib.AbstractContextManager`, it inherits `__enter__()` and
`__exit__()` within which its `_sender`'s corresponding `__enter__()`
and `__exit__()` are called.

In its `run()` method, it call the first policy's `send()` in its
`_impl_policies` list, thus all policies will have their add-ons take
effects and finally the `_sender`'s `send()` method will be invoked.


## module `msrest.pipeline.requests` ##

* `RequestsCredentialsPolicy`(`HTTPPolicy`)
* `RequestsPatchSession`(`HTTPPolicy`)
* `RequestsContext`(`object`)
* `PipelineRequestsHTTPSender`(`HTTPSender`)

### class `RequestsCredentialsPolicy` ###

Hierarchy:
* `msrest.pipeline.requests.RequestsCredentialsPolicy`
  * `msrest.pipeline.HTTPPolicy`

`msrest.pipeline.requests.RequestsCredentialsPolicy` holds a
credential which is a `msrest.authentication.Authentication`.  It
implements `send()` method by calling its `self.next.send()` on the
session in the credential.

### class `PipelineRequestsHTTPSender` ###

Hierarchy:
* `msrest.pipeline.requests.PipelineRequestsHTTPSender`
  * `msrest.pipeline.HTTPSender`

`msrest.pipeline.HTTPSender` is an abstract class declares the
`send()` method which accepts `msrest.pipeline.Request` and returns
`msrest.pipeline.Response`.  And since it is a subclass of
`contextlib.AbstractContextManager`, it inherits `__enter__()` and
`__exit__()`.  It also has a `build_context()` method.

`msrest.pipeline.requests.PipelineRequestsHTTPSender` has a `driver`
variable holds
`msrest.universal_http.requests.BasicRequestsHTTPSender`.  All its
methods are wrapper for the counterparts of its `driver`.


## module `msrest.pipeline.universal` ##

### class `HeadersPolicy` ###

Hierarchy:
* `msrest.pipeline.universal.HeadersPolicy`
  * `msrest.pipeline.SansIOHTTPPolicy`

`msrest.pipeline.SansIOHTTPPolicy` is an interface which declares
`on_request()`, `on_response()`, `on_exception()` methods which will
be invoked before sending a request to its next policy, after the
request comes back and when an exception comes back from the polcy,
respectively.

`msrest.pipeline.universal.HeadersPolicy` adds given headers in its
`on_request()` method before sending a request.

### class `UserAgentPolicy` ###

Hierarchy:
* `msrest.pipeline.universal.HeadersPolicy`
  * `msrest.pipeline.UserAgentPolicy`

Similar to `msrest.pipeline.SansIOHTTPPolicy`, it adds given user
agent in its `on_request()` method before sending a request.

### class `HTTPLogger` ###

Hierarchy:
* `msrest.pipeline.universal.HeadersPolicy`
  * `msrest.pipeline.HTTPLogger`

Similar to `msrest.pipeline.SansIOHTTPPolicy`, it log the request and
response in its `on_request()` and `on_response` method.

## module `msrest.service_client` ##

* `SDKClient`(`object`)
* `_ServiceClientCore`(`object`)
* `ServiceClient`(`_ServiceClientCore`)

### class `ServiceClient` ###

Hierarchy:
* `msrest.service_client.ServiceClient`
  * `msrest.service_client._ServiceClientCore`

`msrest.service_client._ServiceClientCore` has a
`msrest.configuration.Configuration` variable `self.config` to hold
all config data.  It also has methods such as `get()`, `put()`,
`post()` corresponds to HTTP method verbs returning prepared
`msrest.universal_http.ClientRequest`.

`msrest.service_client.ServiceClient` receives a credential and a
`msrest.configuration.Configuration` and holds a
`msrest.pipeline.Pipeline` with a
`msrest.pipeline.requests.PipelineRequestsHTTPSender` as the
pipeline's `_sender` which wraps a
`msrest.universal_http.requests.RequestsHTTPSender`.

Though `msrest.service_client.ServiceClient` is not a
`contextlib.AbstractContextManager`, it has `__enter__()` and
`__exit__()` within which its pipeline's `__enter__()` and
`__exit__()` are called.

To send a request in its `send()` method, pipeline's `run()` method is
invoked.

### class `SDKClient` ###

It is a wrapper of `msrest.service_client.ServiceClient`.



###### Reference ######

* [Azure SDK for Python - Reference - **msrest** package](https://docs.microsoft.com/en-sg/python/api/msrest/msrest?view=azure-python)
* [**msrest**](https://pypi.org/project/msrest/)


