## HTTPS Binding

The abstract interfaces defined by this specification can be implemented and deployed in the form of bindings to
different concrete protocols and transports. This section defines a binding based on HTTP POST operations, with inputs
and outputs sent in the HTTP body, encoded as JSON.

In this binding, a secure HTTPS connection with at least TLS 1.2 MUST be used.

Each one of the three [operations](#operations) can be invoked via a separate HTTPS endpoint, for example:

```
https://uniregistrar.io/1.0/create
https://uniregistrar.io/1.0/update
https://uniregistrar.io/1.0/deactivate
```

The following HTTP status codes are used for the [`create()` function](#create):

* **200**, if the request was successful, but the DID may not be fully created yet, as indicated by the
  [`didState.state` output field](#didstatestate).
* **201**, if the DID has been successfully created, as indicated by the
  [`didState.state` output field](#didstatestate).
* **400**, if a problem with the input fields has occurred.
* **500**, if an internal error has occurred.

The following HTTP status codes are used for the [`update()` function](#update):

* **200**, if request was successful, and the DID may or may not be fully updated yet, as indicated by the
  [`didState.state` output field](#didstatestate).
* **400**, if a problem with the input fields has occurred.
* **500**, if an internal error has occurred.

The following HTTP status codes are used for the [`deactivate()` function](#deactivate) operation:

* **200**, if request was successful, and the DID may or may not be fully deactivated yet, as indicated by the
  [`didState.state` output field](#didstatestate).
* **400**, if a problem with the input fields has occurred.
* **500**, if an internal error has occurred.

See <a href="https://github.com/decentralized-identity/universal-registrar/blob/main/openapi/openapi.yaml">here</a> for an OpenAPI definition.
