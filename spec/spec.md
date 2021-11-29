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
Decentralized Identifier (DID) | A globally unique persistent identifier that does not require a centralized registration authority and is often generated and/or registered cryptographically.
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
create(method, did, options, secret, didDocument) -> jobId, didState, didRegistrationMetadata, didDocumentMetadata
```
___
This function creates a new DID and associated DID document, according to a known DID method, using
various options and optionally an initial DID document.

### `update()`

```
update(did, options, secret, didDocumentOperation, didDocument) -> jobId, didState, didRegistrationMetadata, didDocumentMetadata
```
____
This function updates the DID document associated with the DID, either by completely replacing it, or
by performing an incremental update (see the [`didDocumentOperation` input field](#diddocumentoperation)).

### `deactivate()`

```
deactivate(did, options, secret) -> jobId, didState, didRegistrationMetadata, didDocumentMetadata
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

This input field contains an object with DID controller keys and other secrets.

If present, the structure of this input field MUST follow the same rules as the
[`didState.secret` output field](#didstatesecret)

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

This specification defines four well-known values for this property: `finished`, `failed`, `action`, `wait`. These
values are further explained in [States](#states).

The following diagram illustrates the use of the `didState.state` and `jobId` output fields.

![Diagram showing the flow and states of DID operations](images/diagram-flow.png)

#### `didState.did`

This output field contains the DID at the end of the DID operation.

For the [`create()` function](#create), if the value of the [`didState.state` output field](#didstatestate) is `"finished"`,
then the value of this output field MUST NOT be null.

For the [`update()` function](#update) and [`deactivate()` function](#deactivate), this output field MUST NOT
be null, and its value MUST match the [`did` input field](#did) that was used when executing the function.

#### `didState.secret`

This output field contains an object with DID controller keys and other secrets.

It MUST be present if the DID Registrar is operating in [Internal Secret Mode](#internal-secret-mode) and the
[`secretReturning` option](#secretreturning-option) is set to `true`, and MUST NOT be present otherwise.

If present, the `didState.secret` output field MUST contain a JSON object with either a property `verificationMethod`, or
`keys`, or both.

The `didState.secret` output field MAY contain additional properties that are considered secrets, such as seeds, passwords, etc.

##### `didState.secret.verificationMethod`

If the `didState.secret` output field contains a property `verificationMethod`, then the value of that property MUST be a
JSON array, which MAY be empty. Each element of that JSON array MUST be a JSON object based on the verification method
data model as defined by [[DID-CORE]], with the following differences:

* The `id` property is OPTIONAL.
  * If it is present, its value MUST match the `id` property of the corresponding verification method in the DID's
    associated DID document, which MAY be returned separately in the
    [`didState.didDocument` output field](#didstatediddocument).
  * If it is absent, then the verification method does not correspond to any verification method in the DID's
    associated DID document. 
* The JSON object MUST contain a property `purpose`, and the value of that property MUST be a JSON array, which contains
  verification relationships such as `authentication` or `assertionMethod`.
* Instead of containing properties such as `publicKeyJwk` or `publicKeyMultibase` for expressing verification material,
  the verification method contains corresponding private key material, using properties such as `privateKeyJwk` or
  `privateKeyMultibase`.

Example: 

```json
{
	"verificationMethod": [{
			"id": "did:example:123#key-0",
			"type": "JsonWebKey2020",
			"controller": "did:example:123",
			"purpose": ["authentication", "assertionMethod", "capabilityDelegation", "capabilityInvocation"],
			"privateKeyJwk": {
				"kty": "EC",
				"d": "-s-PwFdfgcdBPTDbJwZuiAFHCuI8r9vR13OGHo14--4",
				"crv": "secp256k1",
				"x": "htusHse5FMBnT_4266kn9T2yMmjDllwWvVSc_I2-WZ0",
				"y": "RjE_GjsRMELYJ6XuNSFDu3mCbyJnCQ26X_YtmyM9Bfo"
			}
		},
		{
			"id": "did:example:123#key-1",
			"type": "Ed25519VerificationKey2020",
			"controller": "did:example:123",
			"purpose": ["authentication"],
			"privateKeyMultibase": "z5TVraf9itbKXrRvt2DSS95Gw4vqU3CHAdetoufdcKazA"
		}
	]
}
```

##### `didState.secret.keys`

If the `didState.secret` output field contains a property `keys`, then it MUST be a valid JWK Set (JWKS) according to
[[spec:RFC7517]]. The value of that property MUST be a JSON array, which MAY be empty. Each element of that JSON array
MUST be a JWK, with the following rule:
* The `kid` parameter is OPTIONAL.
  * If it is present, its value MUST match the `id` property of the corresponding verification method in the DID's 
  associated DID document, which MAY be returned separately in the
[`didState.didDocument` output field](#didstatediddocument).
  * If it is absent, then the JWK does not correspond to any verification method in the DID's associated DID document.

Example:

```json
{
	"seed": "bwT5J3lXaclZzMWfvNOFNr5maUHxZajj",
	"keys": [{
		"kid": "did:example:123#key-1",
		"kty": "OKP",
		"d": "YndUNUozbFhhY2xaek1XZnZOT0ZOcjVtYVVIeFphamo",
		"crv": "Ed25519",
		"x": "zY_7_5gb3AJ033yM9HG5D0tP_ypk0Ozr7x2vzgE279c"
	}]
}
```

#### `didState.didDocument`

This output field contains the DID document after the DID operation has been successfully executed.

This output field is OPTIONAL.

Example:

```json
{
	"@context": [
		"https://www.w3.org/ns/did/v1"
	],
	"id": "did:example:123",
	"verificationMethod": [{
		"type": "Ed25519VerificationKey2018",
		"id": "did:example:123#key-1",
		"publicKeyBase58": "EqRvGzVX3aoLYwZSdKhNd2q5Ez7EVbdPA4DVZW3ngn1U"
	}],
	"authentication": ["did:example:123#key-1"],
	"assertionMethod": ["did:example:123#key-1"]
}
```

### `didRegistrationMetadata`

This output field contains metadata about the registration process itself.

Possible uses of the `didRegistrationMetadata` output field:

* Duration of the DID registration process.
* Various URLs, IP addresses or other network information that was used during the DID registration process.
* Current balance or other information about tokens or cryptocurrencies that were used during the DID registration process.
* Information about third parties that were involved in the DID registration process, such as trust anchors, onboarding services, transaction endorsers, etc.
* Proofs added by a DID registrar (e.g. to establish trusted registration).

### `didDocumentMetadata`

This output field contains metadata about the DID's associated DID document.

Possible uses of the `didDocumentMetadata` output field:

* Hash values, smart contract addresses, blockchain heights, transaction numbers, etc.
* Proofs added by a DID controller (e.g. to establish control authority).

## States

The following sections explain the possible states of a DID registration process, which are returned as values of the
[`didState.state` output field](#didstatestate).

### `didState.state="finished"`

This state indicates that the DID operation has been completed.

Example:

```json
{
	"jobId": null,
	"didState": {
		"state": "finished",
		"did": "did:key:z6MknhhUUtbXCLRmUVhYG7LPPWN4CTKWXTLsygHMD6Ah5uDN",
		"secret": {
			"keys": [{
				...
			}]
		},
		"didDocument": { ... }
	},
	"didRegistrationMetadata": { ... },
	"didDocumentMetadata": { ... }
}
```

### `didState.state="failed"`

This state indicates that the DID operation has failed.

In this state, the [`didState` output field](#didstate) MUST contain a `reason` property that indicates the reason
for the failure.

The [`didState` output field](#didstate) MAY contain additional properties that are relevant to this state.

Example:

```json
{
	"jobId": null,
	"didState": {
		"state": "failed",
		"did": "did:key:z6MknhhUUtbXCLRmUVhYG7LPPWN4CTKWXTLsygHMD6Ah5uDN",
		"reason": "networkUnavailable"
	},
	"didRegistrationMetadata": { ... },
	"didDocumentMetadata": { ... }
}
```

### `didState.state="action"`

This state indicates that the client needs to perform an action, before the DID operation can be continued.

Possible uses for `didState.state="action"`:

* Client needs to perform an action at an external service
* Client needs to perform a cryptographic operation, e.g. generate a signature
* Client needs to send coins to fund a wallet
* Client needs to accept the terms of a legal agreement
* Client needs to upload a file to a webserver

In this state, the [`didState` output field](#didstate) MUST contain an `action` property that indicates the type of
action that needs to be performed.

The [`didState` output field](#didstate) MAY contain additional properties that are relevant to this state.

This specification defines two well-known values for the `action` property that may be used in this state:

* [`didState.action="redirect"`](#didstateactionredirect) - Client needs to be redirected to a web page, e.g. an onboarding service
* [`didState.action="signPayload"`](#didstateactionsignpayload) - Client needs to generate a signature on a payload

Example:

```json
{
	"jobId": "155eae21-45e5-4e71-bb22-fef51cda5bf7",
	"didState": {
		"state": "action",
		"action": "fundingRequired",
		"description": "Please fund the address mzUC2F1XgXfTJEYUBZXdG6M8wWWvhgEknG."
	},
	"didRegistrationMetadata": { ... },
	"didDocumentMetadata": { ... }
}
```

#### `didState.action="redirect"`

This action indicates that the client needs to be redirected to a web page, e.g. an onboarding service where the
DID controller takes certain steps, before the DID operation can be continued.

With this action, the [`didState` output field](#didstate) MUST contain a `redirectUrl` property, which indicates
the URL of the web page where the client needs to be redirected.

The [`didState` output field](#didstate) MAY contain additional properties that are relevant to this action.

TODO: How does this work exactly if the client is not browser-based?

TODO: Does this need features from other well-known redirect-based protocols (e.g. OAuth), such as callback URLs,
nonces, states, etc.?

Example:

```json
{
	"jobId": "155eae21-45e5-4e71-bb22-fef51cda5bf7",
	"didState": {
		"state": "action",
		"action": "redirect",
        "redirectUrl" : "https://..."
	},
	"registrarMetadata": { ... },
	"methodMetadata": { ... }
}
```

#### `didState.action="signPayload"`

This action indicates that the client needs to generate a signature on a payload, before the DID operation can be
continued.

This action is used in [Client Managed Secret Mode](#client-managed-secret-mode).

With this action, the [`didState` output field](#didstate) MUST contain properties that indicate the payload
to be signed, as well as additional information such as a key identifier or the signing algorithm to be used.

The [`didState` output field](#didstate) MAY contain additional properties that are relevant to this action.

TODO: Need to specify in more detail how a DID Registrar requests signing, and how the client responds.

TODO: Mention how this could relate to other specs such as Universal Wallet or WebKMS.

Example:

```json
{
	"jobId": "1234",
	"didState": {
		"state": "action",
		"action": "signPayload",
		"signingRequest": {
			"signingRequest1": {
				"payload": {
					...
				},
				"serializedPayload": "<-multibase->",
				"kid": null,
				"alg": "EdDsa",
				"verificationMethod": "..." // could point to a verificationMethod, incl. VerifiableConditions with threshold, etc.
				"proofPurpose": ".." // describes the purpose of the requested signature/proof
			},
			"signingRequest2": {
				"payload": "8784566",
				"kid": null,
				"alg": "ES256K"
			}
		}
	},
	"didRegistrationMetadata": {},
	"didDocumentMetadata": {}
}
```

### `didState.state="wait"`

This state indicates that the client needs to wait, before the DID operation can be continued.

Possible uses for `didState.state="wait"`:

* Client needs to wait for confirmation on chain
* Client needs to wait for approval by someone

In this state, the [`didState` output field](#didstate) MUST contain a `wait` property, and MAY contain additional
properties, to explain the reason for the failure.

```json
{
	"jobId": "155eae21-45e5-4e71-bb22-fef51cda5bf7",
	"didState": {
		"state": "wait",
		"wait": "Please wait until the transaction is complete.",
		"waitTime": 3600000
	},
	"didRegistrationMetadata": { ... },
	"didDocumentMetadata": { ... }
}
```

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

## HTTPS Binding

The abstract interfaces defined by this specification can be implemented and deployed in the form of bindings to
different concrete protocols and transports. This section defines a binding based on HTTP POST operations, with inputs
and outputs sent in the HTTP body, encoded as JSON.

In this binding, a secure HTTPS connection with at least TLS 1.2 MUST be used.

For example, the abstract interfaces can be deployed at the following HTTPS endpoints:

```
https://uniregistrar.io/1.0/create
https://uniregistrar.io/1.0/update
https://uniregistrar.io/1.0/deactivate
```

See <a href="https://github.com/decentralized-identity/universal-registrar/blob/main/swagger/api.yml">for an OpenAPI definition</a>.

## Normative References

[[spec]]

## Appendix

### Other Resources

The following projects are working on DID registration across different DID methods.

* [DIF Universal Registrar](https://github.com/decentralized-identity/universal-registrar)
* [Universal Services](https://github.com/alenhorvat/universal-services)
* [ACA-py](https://github.com/hyperledger/aries-cloudagent-python/)
* [aries-framework-go](https://github.com/hyperledger/aries-framework-go)
* [Veramo](https://github.com/uport-project/veramo)
* [DIDKit](https://github.com/spruceid/didkit)
