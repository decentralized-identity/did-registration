DID Registration
================

**Specification Status:** Pre-Draft

**Latest Draft:**
  [identity.foundation/did-registration](https://identity.foundation/did-registration)

**Editors:**
~ [Markus Sabadello](https://www.linkedin.com/in/markus-sabadello-353a0821/) (Danube Tech)
~ Others?
<!-- -->
**Authors:**
~ [Markus Sabadello](https://www.linkedin.com/in/markus-sabadello-353a0821/) (Danube Tech)
~ [Cihan Saglam](https://www.linkedin.com/in/cihans/) (Danube Tech)
~ Others?
<!-- -->
**Participate:**
~ [GitHub repo](https://github.com/decentralized-identity/did-registration)
~ [File a bug](https://github.com/decentralized-identity/did-registration/issues)
~ [Commit history](https://github.com/decentralized-identity/did-registration/commits/master)

------------------------------------

## Abstract

Each DID method specifies how to create/resolve/update/deactivate DIDs within a given verifiable data
registry. How exactly this works can be very different depending on the DID method and may involve
various steps, architectural components, and network communication. The process of resolving a DID
to a DID document by executing the read() operation is well-understood and specified in the
[DID Resolution](https://w3c-ccg.github.io/did-resolution/) specification. This document complements
the concept of a "DID Resolver" by also defining a "DID Registrar" component that can execute the three remaining
create/update/deactivate operations via a common interface.

This specification does NOT specify a DID method. It does NOT specify how to build a verifiable data
registry, only a "DID Registrar" component that can interact with a verifiable data registry. This
specification also does NOT suggest that private keys should be controlled by any other entity than the
DID controller.

## Status of This Document

DID Registration is a draft specification under development within the
[Decentralized Identity Foundation](https://identity.foundation) (DIF (DIF), and designed to incorporate the
requirements and learnings from related work of the most active industry players
into a shared specification that meets the collective needs of the community.
This spec is regularly updated to reflect relevant changes, and we encourage
active engagement on GitHub (see above) and other mediums (e.g. DIF) where this
work is being done.

## Terminology

Term | Definition
:--- | :---------
Decentralized Identifier (DID) | A globally unique persistent identifier that does not require a centralized registration authority because it is generated and/or registered cryptographically.
DID Method | A definition of how a specific DID scheme implementeds the precise operations by which DIDs are created, resolved and deactivated and DID documents are written and updated.
DID Document | A set of data describing the DID subject, service and verification methods, that the DID subject or a DID delegate can use to authenticate itself and prove its association with the DID.
DID Registrar | A software and/or hardware component that implements the DID create/update/deactivate functions.
DID Resolver | A software and/or hardware component that implements the DID resolution function.
Wallet | A software and/or hardware component that can securely store DIDs and associated private keys and other sensitive cryptographic key material. Wallets implement various interfaces for cryptographic key generation, signing, verification, and other operations.

## Architecture Modes

The DID create/update/deactivate functions raise architectural questions around key management,
since they typically involve the generation and use of private keys and other secrets.

With regard to key management, a DID Registrar can operate in the following modes:

### Internal Secret Mode

In this mode, the DID Registrar is responsible for generating the DID controller keys used by DID operations. This
means  that the DID Registrar is considered a highly trusted component which should be fully under the control of a
DID controller. If it is operated as a remotely hosted service, secure connection protocols such as TLS, DIDComm,
etc. MUST be used.

This mode has two options that control how DID controller keys are handled.

#### secretStoring Option

If the `secretStoring` option is set to `true`, then the DID Registrar maintains an internal wallet where DIDs
and DID controller keys can be stored. The DID controller keys can then be used in future DID operations. For
example, if a `create()` operation is executed, then a subsequent `update()` or `deactivate()` operation will
be able to use existing DID controller keys stored in the DID Registrar.

TODO: Mention potential import/export of keys, and how this could relate to other specs such as Universal Wallet or WebKMS.

#### secretReturning Option

If the secretReturning option is set to `true`, then the DID Registrar will return generated DID controller keys
to the client.

#### Considerations

The `secretStoring` and `secretReturning` options can be enabled or disabled independently. A DID Registrar
may define default values for these options, and/or it may allow a client to set them via
the [`options` input field](#options).

If `secretReturning==false`, then the 

Note that if neither option is enabled, then control over a DID may get permanently lost, since the DID Registrar
operating in [Internal Secret Mode](#internal-secret-mode) will generate DID controller keys internally, but it will
neither store them nor return them to a client.

If a DID Registrar is configured with options `secretStoring=false` and `secretReturning=true`, then a DID Registrar
with option `secretStoring=true` can be simulated by building a "wrapping DID Registrar" around an
"inner DID Registrar".

<img alt="Diagram showing Internal Secret Mode" src="images/diagram-mode-internal-secret.png">

### External Secret Mode

In this mode, the DID Registrar does not itself have access to the secrets used by DID operations, but it has a way
of accessing an external wallet in order to perform cryptographic operations such as generating signatures.

TODO: Mention how this could relate to other specs such as Universal Wallet or WebKMS.

<img alt="Diagram showing External Secret Mode" src="images/diagram-mode-external-secret.png">

### Client-managed Secret Mode

In this mode, the DID Registrar does not itself have access to the secrets used by DID operations, but it will ask
the client to perform cryptographic operations such as generating signatures.

TODO: Discuss how the did:ion use case fits in, where the client supplies the public keys / commitments during the create operation.

<img alt="Diagram showing Client-Managed Secret Mode" src="images/diagram-mode-client-managed-secret.png">

## Operations

### `create()`

```
create(method, did, options, secret, didDocument) -> jobId, didState, metadata
```
___
This function creates a new DID and associated DID document, according to a known DID method, using
various options and optionally an initial DID document.

### `update()`

```
update(did, options, secret, didDocumentOperation, didDocument) -> jobId, didState, metadata
```
____
This function updates the DID document associated with the DID, either by completely replacing it, or
by performing an incremental update (see the [`didDocumentOperation` input field](#diddocumentoperation)).

### `deactivate()`

```
deactivate(did, options, secret) -> jobId, didState, metadata
```
___
This function deactivates the DID.

## Input Fields

### `method`

This input field indicates the DID method that should be used for the DID operation.

For the [`create()` function](#create), if the [`did` input field](#did) is absent or its value is null, then
this input field MUST NOT be null, otherwise it MUST be null.

For the [`update()` function](#update) and [`deactivate()` function](#deactivate), this input field MUST be
null, since the DID method is determined by the [`did` input field](#did).

Possible values:

- `btcr`
- `sov`
- `key`
- `web`
- ... other supported DID method names ...

### `did`

This input field indicates the DID that is the target of the DID operation.

For the [`create()` function](#create), this input field is OPTIONAL. Specific DID methods typically
restrict how this input field is used.

For the [`update()` function](#update) and [`deactivate()` function](#deactivate), this input field MUST NOT be null.

### `options`
 
This input field contains an object with various options for the DID operation, such as the network where the DID
should be created.

Possible keys and values:

* `network=mainnet`
* ... others ...

### `secret`

This input field contains an object with DID controller keys and other secrets needed for performing the DID operations.

Possible keys and values:

* `seed=...`
* `privateKey=...`
* ... others ...

### `didDocumentOperation`

This input field indicates which update operation should be applied to a DID's associated DID document.

For the [`create()` function](#create), this input field MUST be absent, and is implied to have a value of `setDidDocument`.

For the [`update()` function](#update), this input field is OPTIONAL.
If it is absent or has a null value, it is implied to have a value of `"setDidDocument"`.

For the [`deactivate()` function](#deactivate), this input field MUST be absent.

This specification defines several standard values for this operation. Individual DID methods MAY specify other
ways of executing an [`update()` function](#update).

Possible values:

- `"setDidDocument"`: The [`didDocument` input field](#didDocument) contains the new DID document of the DID after the DID operation is executed.
- `"addToDidDocument"`: The [`didDocument` input field](#didDocument) contains properties to be added to the current DID document of the DID after the DID operation is executed.
- `"removeFromDidDocument"`: The [`didDocument` input field](#didDocument) contains properties to be removed from the current DID document of the DID after the DID operation is executed.
- ... other DID method-specific DID document operations ...

Note that some of the above update operations can be internally transformed into others, e.g.:

- If the update operation is `"setDidDocument"`, then it may be transformed into either `"addToDidDocument"`
  or `"removeFromDidDocument"`, by first resolving the current DID document, and then calculating an incremental
  change (diff) between the resolved DID document and the value of the [`didDocument` input field](#didDocument).
- If the update operation is either `"addToDidDocument"` or `"removeFromDidDocument"`, then it may be transformed
  into `"setDidDocument"`, by first resolving the current DID document, and then calculating the updated complete
  DID document after applying the incremental change (diff) in the [`didDocument` input field](#didDocument).

A DID Registrar may or may not support certain update operations, and it may or may not support internal
transformation between them.

### `didDocument`

This input field contains either a complete DID document, or an incremental change (diff) to a DID document, depending
on the value of the [`didDocumentOperation` input field](#operation).

For the [`deactivate()` function](#deactivate), this input field is absent.

## Output Fields

### `jobId`

When executing DID operations, an operation can turn into a longer-running "job", during which the operation's
state might change. In order for a DID operation to complete, the DID create/update/deactivate functions may
have to be called multiple times, and the `jobId` is used to keep track of the ongoing process.

This output field MUST be absent or have a null value if the value of the [`didState.state` output field](#didstatestate)
is either `finished` or `failed`, and MUST NOT have a null value otherwise.

### `didState`

#### `didState.state`

This output field contains the current state of the DID operations. It is used to indicate if a DID operation
is finished, failed, or if a longer-running "job" has been created that requires additional steps.

The following diagram illustrates the use of the `didState.state` and `jobId` output fields.

![Diagram showing the flow and states of DID operations](images/diagram-flow.png)

The following sections explain the possible values for `didState.state`.

##### `didState.state=finished`

This state indicates that the DID operation has been completed.

Example:

```
{
	"jobId": null,
	"didState": {
		"did": "did:key:z6MknhhUUtbXCLRmUVhYG7LPPWN4CTKWXTLsygHMD6Ah5uDN",
		"state": "finished",
		"secret": {
			"keys": [{
				...
			}]
		}
	},
	"registrarMetadata": { ... },
	"methodMetadata": { ... }
}
```

##### `didState.state=failed`

This state indicates that the DID operation has failed.

In this state, the [`didState` output field](#didstate) MUST contain a `reason` property, and MAY contain additional
properties, to explain the reason for the failure.

Example:

```
{
	"jobId": null,
	"didState": {
		"did": "did:key:z6MknhhUUtbXCLRmUVhYG7LPPWN4CTKWXTLsygHMD6Ah5uDN",
		"state": "failed",
		"reason": "networkUnavailable"
	},
	"registrarMetadata": { ... },
	"methodMetadata": { ... }
}
```

##### `didState.state=action`

This state indicates that the client needs to perform an action, before the DID operation can be continued.

In this state, the [`didState` output field](#didstate) MUST contain an `action` property, and MAY contain additional
properties, to specify the nature of the action that has to be taken.

Possible uses for `didState.state==action`:

  * Send coins to fund a wallet
  * Accept some legal agreement
  * Perform signature on a byte array

Example 1:

```
{
	"jobId": "155eae21-45e5-4e71-bb22-fef51cda5bf7",
	"didState": {
		"state": "action",
		"action": "fundingRequired",
		"description": "Please fund the address mzUC2F1XgXfTJEYUBZXdG6M8wWWvhgEknG."
	},
	"registrarMetadata": { ... },
	"methodMetadata": { ... }
}
```

Example 2:

TODO: This is used in Client-managed Secret Mode. Need to specify in more detail how a DID Registrar requests signing, and how the client responds.

```
{
	"jobId": "155eae21-45e5-4e71-bb22-fef51cda5bf7",
	"didState": {
		"state": "action",
		"action": "signPayload",
		"signingRequests": [
		  {
		    "payload": "...",
		    "payloadId": "...",
		    "signingAlgorithm": "EdDsa",
		    "keyReference": "... OPTIONAL ..."
		  },
		  {
		    "payload": "...",
		    "payloadId": "...",
		    "signingAlgorithm": "EdDsa",
		    "keyReference": "... OPTIONAL ..."
		  }
		]
	},
	"registrarMetadata": { ... },
	"methodMetadata": { ... }
}
```

##### `didState.state=wait`

This state indicates that the client needs to wait, before the DID operation can be continued.

In this state, the [`didState` output field](#didstate) MUST contain a `wait` property, and MAY contain additional
properties, to explain the reason for the failure.

Possible uses for `didState.state==wait`:

  * Wait for confirmation on chain
  * Wait for approval by someone

```
{
	"jobId": "155eae21-45e5-4e71-bb22-fef51cda5bf7",
	"didState": {
		"state": "wait",
		"wait": "Please wait until the transaction is complete.",
		"waitTime": 3600000
	},
	"registrarMetadata": { ... },
	"methodMetadata": { ... }
}
```

#### `didState.did`

This output field contains the DID at the end of the DID operation.

For the [`create()` function](#create), if the value of the [`didState.state` output field](#didstatestate) is `"finished"`,
then the value of this output field MUST NOT be null.

For the [`update()` function](#update) and [`deactivate()` function](#deactivate), this output field MUST NOT
be null, and its value MUST match the [`did` input field](#did) that was used when executing the function.

#### `didState.secret`

This output field contains DID controller keys and other secrets.

It is only used if the DID Registrar is operating in [Internal Secret Mode](#internal-secret-mode), and if the
[`secretReturning` option] is set to `true`.

TODO: Specify the format of returned DID controller keys.

#### `didState.didDocument`

This output field contains the DID document after the DID operation has been successfully executed.

This output field is OPTIONAL.

### `metadata`

This output field contains various metadata about the DID operation.

Possible uses for the `metadata` output field:

* operation metadata
   * duration
* method metadata
   * method-specific hash
   * token balance

## Implementation Considerations

In order to implement a library or tool that supports the above interfaces for
creating, updating, and deactivating DIDs in a method-agnostic way, we can imagine
a similar architecture as is common for a DID resolver, i.e. using a set of drivers
that perform method-specific operations.

Some architectural questions that apply to a DID Resolver also apply to a DID Registrar, e.g.:

* Is the abstract interface implemented as a library that can be integrated
  locally into an application or service, or is the abstract interface exposed by a remote
  service and used via HTTP or another binding?
* How do method-specific drivers interact with the DID's target system? For example,
  do they have direct access to a blockchain full node?
* What are implications of the above questions for trust and security?

## HTTP Binding

The abstract interface above can be implemented and deployed in the form of bindings to different protocols, such as simple HTTP POST operations, with inputs and outputs encoded as JSON.

For example, the operations above can be deployed at the following endpoints:

```
https://uniregistrar.io/1.0/create
https://uniregistrar.io/1.0/update
https://uniregistrar.io/1.0/deactivate
```

See <a href="https://github.com/decentralized-identity/universal-registrar/blob/main/swagger/api.yml">for an OpenAPI definition</a>.

## Resources

The following projects are working on pluggable support for write operations across different DID methods.

* [DIF Universal Registrar](https://github.com/decentralized-identity/universal-registrar)
* [Universal Services](https://github.com/alenhorvat/universal-services)
* [ACA-py](https://github.com/hyperledger/aries-cloudagent-python/)
* [aries-framework-go](https://github.com/hyperledger/aries-framework-go)
* [Veramo](https://github.com/uport-project/veramo)
* [DIDKit](https://github.com/spruceid/didkit)
