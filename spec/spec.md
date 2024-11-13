DID Registration
================

**Specification Status:** [Draft](https://github.com/decentralized-identity/org/blob/master/work-item-lifecycle.md)

**Latest Draft:**
  [identity.foundation/did-registration](https://identity.foundation/did-registration)

**Editors:**
~ [Markus Sabadello](https://www.linkedin.com/in/markus-sabadello-353a0821/) (Danube Tech)
~ Others?
<!-- -->
**Authors:**
~ [Markus Sabadello](https://www.linkedin.com/in/markus-sabadello-353a0821/) (Danube Tech)
~ [Cihan Saglam](https://www.linkedin.com/in/cihans/) (Danube Tech)
~ [Ahamed Azeem](https://www.linkedin.com/in/ahamedazeem/) (Danube Tech)
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

## Key Management

The DID create/update/deactivate functions raise architectural questions around key management,
since they typically involve the generation and use of private keys and other secrets.

With regard to key management, a DID Registrar can operate in the following modes:

### Internal Secret Mode

In this mode, the DID Registrar is responsible for generating the DID controller keys used by DID operations. This
means  that the DID Registrar is considered a highly trusted component which should be fully under the control of a
DID controller. If it is operated as a remotely hosted service, secure connection protocols such as TLS, DIDComm,
etc. MUST be used.

This mode has two options that control how DID controller keys are handled:
[`storeSecrets`](#storesecrets) and [`returnSecrets`](#returnsecrets).

The `storeSecrets` and `returnSecrets` options can be enabled or disabled independently. A DID Registrar
may define default values for these options, and/or it may allow a client to set them via
the [`options` input field](#options).

Note that if neither option is enabled, then control over a DID may get permanently lost, since the DID Registrar
operating in [Internal Secret Mode](#internal-secret-mode) will generate DID controller keys internally, but it will
neither store them nor return them to a client.

If a DID Registrar is configured with options `storeSecrets=false` and `returnSecrets=true`, then a DID Registrar
with option `storeSecrets=true` can be simulated by building a "wrapping DID Registrar" around an
"inner DID Registrar".

<img alt="Diagram showing Internal Secret Mode" src="images/diagram-mode-internal-secret.png">

### External Secret Mode

In this mode, the DID Registrar does not itself have access to the secrets used by DID operations, but it has a way
of accessing an external wallet in order to perform cryptographic operations such as generating signatures.

The interface between the DID Registrar and an external wallet is out of scope for this specification, but could
potentially use wallet or key management interfaces defined by other specifications, e.g.:

- [Universal Wallet](https://w3c-ccg.github.io/universal-wallet-interop-spec/)
- [WebKMS](https://w3c-ccg.github.io/webkms/)
- [WebCrypto API](https://w3c.github.io/webcrypto/#subtlecrypto-interface)
- others...

If additional information (such as a URL, or tenant ID, or secret key, etc.) is required in order for the DID Registrar
to be able to access the external wallet, then such information could be preconfigured in the DID Registrar, or
alternatively supplied by a client in the [`options`](#options) and/or [`secret`](#secret) input fields.

<img alt="Diagram showing External Secret Mode" src="images/diagram-mode-external-secret.png">

### Client-managed Secret Mode

In this mode, the DID Registrar does not itself have access to the secrets used by DID operations, but it will ask
the client to perform cryptographic operations such as generating signatures.

::: todo
Explain how the did:ion use case fits in, where the client supplies the public keys / commitments during the create operation.
:::

This mode has one option that controls how DID controller keys are handled:
[`clientSecretMode`](#clientsecretmode).

<img alt="Diagram showing Client-Managed Secret Mode" src="images/diagram-mode-client-managed-secret.png">

## Operations

### `create()`

```
create(method, options, secret, didDocument) -> jobId, didState, didRegistrationMetadata, didDocumentMetadata
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
by performing an incremental update, or in another way. The specific update operation to be executed is specified in the
[`didDocumentOperation` input field](#diddocumentoperation).

### `deactivate()`

```
deactivate(did, options, secret) -> jobId, didState, didRegistrationMetadata, didDocumentMetadata
```
___
This function deactivates the DID.

### `execute()`

```
execute(did, options, secret, operation, operationData) -> jobId, didState, operationResult, didRegistrationMetadata, didDocumentMetadata
```
____
This function executes an operation that uses the DID but is not related to the standard
create/update/deactivate operations. This is essentially an extensibility feature that makes
it possible to perform operations that do not affect the DID or DID document itself, but use
them for any kind of additional functionality which uses the DID. The specific operation
to be executed is specified in the [`operation` input field](#operation).

::: note
This operation is currently an experimental feature.
:::

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
should be created, or instructions that influence the DID operation. Options may be DID method-specific or may be universally
applicable across all DID methods.

This specification defines several DID method-independent properties for this field that are relevant to [Key Management](#key-management).

Example:

```json
{
	"did": null,
	"options": {
		"clientSecretMode": true,
		"network": "mainnet"
	},
	"secret": { ... },
	"didDocument": { ... }
}
```

#### `clientSecretMode`

In [Client-managed Secret Mode](#client-managed-secret-mode), if the `clientSecretMode` option is set to `true`, then the DID Registrar will
enable client-managed secret mode and let the client perform cryptographic operations such as generating signatures.

Example:

```json
{
	"did": null,
	"options": {
		"clientSecretMode": true
	},
	"secret": { ... },
	"didDocument": { ... }
}
```

#### `storeSecrets`

In [Internal Secret Mode](#internal-secret-mode), if the `storeSecrets` option is set to `true`, then the DID Registrar maintains an internal wallet where DIDs
and DID controller keys can be stored. The DID controller keys can then be used in future DID operations. For
example, if a `create()` operation is executed, then a subsequent `update()` or `deactivate()` operation will
be able to use existing DID controller keys stored in the DID Registrar.

Example:

```json
{
	"did": null,
	"options": {
		"storeSecrets": true
	},
	"secret": { ... },
	"didDocument": { ... }
}
```

#### `returnSecrets`

In [Internal Secret Mode](#internal-secret-mode), if the `returnSecrets` option is set to `true`, then the DID Registrar will return generated DID controller keys
to the client.

Example:

```json
{
	"did": null,
	"options": {
		"returnSecrets": true
	},
	"secret": { ... },
	"didDocument": { ... }
}
```

### `secret`

This input field contains an object with DID controller keys and other secrets.

In [Internal Secret Mode](#internal-secret-mode), this input field MAY contain one or more of the following:

* A `verificationMethod` property with a JSON array containing one or more [Verification Method Private Data](#verification-method-private-data) objects.

Example:

```json
{
	"did": null,
	"options": { ... },
	"secret": {
		"verificationMethod": [{
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
		}]
	},
	"didDocument": { ... }
}
```

In [Client-managed Secret Mode](#client-managed-secret-mode), this input field MAY contain one or more of the following:

* A `verificationMethod` property with a JSON array containing one or more [Verification Method Public Data](#verification-method-public-data) objects.
* A `signingResponse` property with a [Signing Response Set](#signing-response-set) data structure as a response to a [Signing Request Set](#signing-request-set) from the DID Registrar.
* A `decryptionResponse` property with a [Decryption Response Set](#decryption-response-set) data structure as a response to a [Decryption Request Set](#decryption-request-set) from the DID Registrar.

Example:

```json
{
	"did": null,
	"options": { ... },
	"secret": {
		"verificationMethod": [{
			"type": "JsonWebKey2020",
			"controller": "did:example:123",
			"purpose": ["authentication", "assertionMethod", "capabilityDelegation", "capabilityInvocation"],
			"publicKeyJwk": {
				"kty": "EC",
				"crv": "secp256k1",
				"x": "htusHse5FMBnT_4266kn9T2yMmjDllwWvVSc_I2-WZ0",
				"y": "RjE_GjsRMELYJ6XuNSFDu3mCbyJnCQ26X_YtmyM9Bfo"
			}
		}]
	},
	"didDocument": { ... }
}
```

Example:

```json
{
	"jobId": "155eae21-45e5-4e71-bb22-fef51cda5bf7",
	"did": null,
	"options": { ... },
	"secret": {
		"signingResponse": {
			"signingRequest1": {
				"signature": "<-- base64 encoded -->",
				"kid": "did:example:123#key-0",
				"alg": "EdDSA"
			}
		}
	},
	"didDocument": { ... }
}
```

Example:

```json
{
	"jobId": "155eae21-45e5-4e71-bb22-fef51cda5bf7",
	"did": null,
	"options": { ... },
	"secret": {
		"decryptionResponse": {
			"decryptionRequest1": {
				"decryptedPayload": "<-- base64 encoded -->"
			}
		}
	},
	"didDocument": { ... }
}
```

### `didDocumentOperation`

This input field indicates which update operation(s) should be applied to a DID's associated DID document.

For the [`create()` function](#create), this input field MUST be absent, and is implied to have a single value of `setDidDocument`.

For the [`update()` function](#update), this input field is OPTIONAL. If present, it MUST contain a JSON array of string
values to indicate one or more DID document operations that should be performed.
If it is absent, it is implied to have a single value of `"setDidDocument"`.

For the [`deactivate()` function](#deactivate), this input field MUST be absent.

This specification defines several standard values for this operation. Individual DID methods MAY specify other
ways of executing an [`update()` function](#update).

Possible values:

- `"setDidDocument"`: The [`didDocument` input field](#didDocument) contains the new DID document of the DID after the DID operation is executed.
- `"addToDidDocument"`: The [`didDocument` input field](#didDocument) contains properties to be added to the current DID document of the DID after the DID operation is executed.
- `"removeFromDidDocument"`: The [`didDocument` input field](#didDocument) contains properties to be removed from the current DID document of the DID after the DID operation is executed.
- ... other DID method-specific DID document operations ...

Example:

```json
{
	"did": "did:example:123",
	"options": { ... },
	"secret": { ... },
	"didDocumentOperation": ["setDidDocument"],
	"didDocument": [{
		"@context": [
			"https://www.w3.org/ns/did/v1",
			"https://w3id.org/security/suites/jws-2020/v1"
		],
		"id": "did:example:123",
		"verificationMethod": [{
			"id": "did:example:123#key-1",
			"type": "JsonWebKey2020",
			"controller": "did:example:123",
			"publicKeyJwk": {
				"kty": "OKP",
				"crv": "Ed25519",
				"x": "VCpo2LMLhn6iWku8MKvSLg2ZAoC-nlOyPVQaO3FxVeQ"
			}
		}]
	}]
}
```

Example:

```json
{
	"did": "did:example:123",
	"options": { ... },
	"secret": { ... },
	"didDocumentOperation": ["removeFromDidDocument", "addToDidDocument"],
	"didDocument": [{
			"verificationMethod": [{
				"id": "did:example:123#key-1"
			}]
		},
		{
			"verificationMethod": [{
				"id": "did:example:123#key-2",
				"type": "JsonWebKey2020",
				"controller": "did:example:123",
				"publicKeyJwk": {
					"kty": "OKP",
					"crv": "Ed25519",
					"x": "eylT8wroHZUY8Qv6tlMYQWFe88U0Mi5_lI8eud9xgPg"
				}
			}]
		}
	]
}
```

Note that some of the above update operations can be internally transformed into others, e.g.:

- If the update operation is `"setDidDocument"`, then it may be transformed into a combination of `"addToDidDocument"`
  and `"removeFromDidDocument"`, by first resolving the current DID document, and then calculating incremental
  changes (diffs) between the resolved DID document and the value of the [`didDocument` input field](#didDocument).
- If the update operation is either `"addToDidDocument"` or `"removeFromDidDocument"`, then it may be transformed
  into `"setDidDocument"`, by first resolving the current DID document, and then calculating the updated complete
  DID document after applying the incremental changes (diffs) in the [`didDocument` input field](#didDocument).

A DID Registrar may or may not support certain update operations, and it may or may not support internal
transformation between them.

### `didDocument`

This input field contains either a complete DID document, or incremental changes in a DID document,
depending on the value of the [`didDocumentOperation` input field](#diddocumentoperation).

For the [`create()` function](#create), this input field MUST contain a single JSON object value.

For the [`update()` function](#update), this input field is OPTIONAL. If present, it MUST contain a JSON array of JSON object values.

For the [`deactivate()` function](#deactivate), this input field MUST be absent.

For the [`execute()` function](#execute), this input field MUST be absent.

### `operation`

This input field indicates which operation(s) should be executed when the [`execute()` function](#execute) function is used to
execute an operation that does not affect the DID or DID document itself.

For the [`create()` function](#create), [`update()` function](#update), and [`deactivate()` function](#deactivate) this
input field MUST be absent.

For the [`execute()` function](#execute), this input field is REQUIRED and MUST contain a JSON array of string
values to indicate one or more operations that should be executed.

This specification does not define any concrete operation. It is expected that such operations will
often be DID method-specific, although it is also possible that method-independent operations could be defined
in the future by other specifications.

Example:

```json
{
	"options": {
		"network": "mainnet"
	},
	"operation": ["addToTrustRegistry"],
	"operationData": [{
		"trustRegistryUrl": "https://example.com/registry"
    }]
}
```

### `operationData`

This input field contains additional data when the [`execute()` function](#execute) function is used to
execute an operation that does not affect the DID or DID document itself.

For the [`create()` function](#create), [`update()` function](#update), and [`deactivate()` function](#deactivate) this
input field MUST be absent.

For the [`execute()` function](#execute), this input field is REQUIRED and MUST contain a JSON array of JSON object values.

## Output Fields

### `jobId`

When executing DID operations, an operation can turn into a longer-running "job", during which the operation's
state might change. In order for a DID operation to complete, the DID create/update/deactivate functions may
have to be called multiple times, and the `jobId` is used to keep track of the ongoing process.

This output field MUST be absent or have a null value if the value of the [`didState.state` output field](#didstatestate)
is either `finished` or `failed`, and MUST NOT have a null value otherwise.

Example:

```json
{
	"jobId": "2a4b5d3f-7af9-44c8-a5c8-39f893fa0f78",
	"didState": { ... },
	"didRegistrationMetadata": { ... },
	"didDocumentMetadata": { ... }
}
```

### `didState`

This output field contains an object with the following fields:

* [`didState.state`](#didstatestate): The current state of DID operations.
* [`didState.did`](#didstatedid): The DID at the end of the DID operation.
* [`didState.secret`](#didstatesecret): An object with DID controller keys and other secrets.
* [`didState.didDocument`](#didstatediddocument): The DID document after the DID operation has been successfully executed.

In [Client-managed Secret Mode](#client-managed-secret-mode), this output field MAY contain one or more of the following:

* A `verificationMethodTemplate` property with a JSON array containing one or more [Verification Method Template](#verification-method-template) objects.
* A `signingRequest` property with a [Signing Request Set](#signing-request-set).
* A `decryptionRequest` property with a [Decryption Request Set](#decryption-request-set).

#### `didState.state`

This output field contains the current state of the DID operations. It is used to indicate if a DID operation
is finished, failed, or if a longer-running "job" has been created that requires additional steps.

This specification defines four well-known values for this property: `finished`, `failed`, `action`, `wait`. These
values are further explained in [States](#states).

The following diagram illustrates the use of the [`didState.state` output field](#didstatestate) and [`jobId` output field](#jobid) output fields.

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
[`returnSecrets` option](#returnsecrets-option) is set to `true`, and MUST NOT be present otherwise.

In [Internal Secret Mode](#internal-secret-mode), this output field MAY contain one or more of the following:

* A `verificationMethod` property with a JSON array containing one or more [Verification Method Private Data](#verification-method-private-data) objects.

Example:

```json
{
	"jobId": null,
	"didState": {
		"state": "finished",
		"did": "did:key:z6MknhhUUtbXCLRmUVhYG7LPPWN4CTKWXTLsygHMD6Ah5uDN",
		"secret": {
			"verificationMethod": [{
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
			}]
		},
		"didDocument": { ... }
	},
	"didRegistrationMetadata": { ... },
	"didDocumentMetadata": { ... }
}
```

In [Client-managed Secret Mode](#client-managed-secret-mode), this output field MAY contain one or more of the following:

* A `verificationMethod` property with a JSON array containing one or more [Verification Method Public Data](#verification-method-public-data) objects.

Example:

```json
{
	"jobId": null,
	"didState": {
		"state": "finished",
		"did": "did:key:z6MknhhUUtbXCLRmUVhYG7LPPWN4CTKWXTLsygHMD6Ah5uDN",
		"secret": {
			"verificationMethod": [{
				"type": "JsonWebKey2020",
				"controller": "did:example:123",
				"purpose": ["authentication", "assertionMethod", "capabilityDelegation", "capabilityInvocation"],
				"publicKeyJwk": {
					"kty": "EC",
					"crv": "secp256k1",
					"x": "htusHse5FMBnT_4266kn9T2yMmjDllwWvVSc_I2-WZ0",
					"y": "RjE_GjsRMELYJ6XuNSFDu3mCbyJnCQ26X_YtmyM9Bfo"
				}
			}]
		},
		"didDocument": { ... }
	},
	"didRegistrationMetadata": { ... },
	"didDocumentMetadata": { ... }
}
```

The `didState.secret` output field MAY contain additional properties that are considered secrets, such as seeds, passwords, etc.

#### `didState.didDocument`

This output field contains the DID document after the DID operation has been successfully executed.

This output field is OPTIONAL.

Example:

```json
{
	"jobId": null,
	"didState": {
		"state": "finished",
		"did": "did:key:z6MknhhUUtbXCLRmUVhYG7LPPWN4CTKWXTLsygHMD6Ah5uDN",
		"secret": { ... },
		"didDocument": {
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
	},
	"didRegistrationMetadata": { ... },
	"didDocumentMetadata": { ... }
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

### `operationResult`

This input field contains the result when the [`execute()` function](#execute) function is used to
execute an operation that does not affect the DID or DID document itself.

For the [`create()` function](#create), [`update()` function](#update), and [`deactivate()` function](#deactivate) this
output field MUST be absent.

For the [`execute()` function](#execute), this input field is REQUIRED and MUST contain a JSON array of JSON object values.

Example:

```json
{
	"jobId": null,
	"didState": { ... },
	"operationResult": [{
		"myResult": "myResultValue"
	}],
	"didRegistrationMetadata": { ... },
	"didDocumentMetadata": { ... }
}
```

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
		"secret": { ... },
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

This specification defines the following well-known values for the `action` property that may be used in this state:

* [`didState.action="redirect"`](#didstateactionredirect) - Client needs to be redirected to a web page, e.g. an onboarding service.
* [`didState.action="getVerificationMethod"`](#didstateactiongetverificationmethod) - Client needs to provide a newly generated or already existing verification method.
* [`didState.action="signPayload"`](#didstateactionsignpayload) - Client needs to generate a signature on a payload.
* [`didState.action="decryptPayload"`](#didstateactiondecryptpayload) - Client needs to decrypt a payload.

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

::: issue
How does this work exactly if the client is not browser-based?
:::

::: issue
Does this need features from other well-known redirect-based protocols (e.g. OAuth), such as callback URLs, nonces, states, etc.?
:::

Example:

```json
{
	"jobId": "155eae21-45e5-4e71-bb22-fef51cda5bf7",
	"didState": {
		"state": "action",
		"action": "redirect",
		"redirectUrl" : "https://..."
	},
	"didRegistrationMetadata": { ... },
	"didDocumentMetadata": { ... }
}
```

#### `didState.action="getVerificationMethod"`

This action indicates that the client needs to provide a newly generated or already existing verification method, before the DID operation can be
continued.

This action is used in [Client Managed Secret Mode](#client-managed-secret-mode).

The [`didState` output field](#didstate) MUST contain a property `verificationMethodTemplate` with a JSON array containing one or more [Verification Method Template](#verification-method-template) data structures.

The [`didState` output field](#didstate) MAY contain additional properties that are relevant to this action.

::: todo
Explain that the generated verification method must then be included in either "secret" or "didDocument" in next request.
:::

Example:

```json
{
	"jobId": null,
	"didState": {
		"state": "action",
		"action": "getVerificationMethod",
		"verificationMethodTemplate": [{
			"id": "#key-1",
			"type": "Ed25519VerificationKey2018"
		}]
	},
	"didRegistrationMetadata": {},
	"didDocumentMetadata": {}
}
```

Example:

```json
{
	"jobId": null,
	"didState": {
		"state": "action",
		"action": "getVerificationMethod",
		"verificationMethodTemplate": [{
			"purpose": ["recovery"],
			"type": "EcdsaSecp256k1VerificationKey2019"
		}]
	},
	"didRegistrationMetadata": {},
	"didDocumentMetadata": {}
}
```

#### `didState.action="signPayload"`

This action indicates that the client needs to generate a signature on a payload, before the DID operation can be
continued.

This action is used in [Client Managed Secret Mode](#client-managed-secret-mode).

The [`didState` output field](#didstate) MUST contain a property `signingRequest` with a [Signing Request Set](#signing-request-set) data structure.

The [`didState` output field](#didstate) MAY contain additional properties that are relevant to this action.

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
				"serializedPayload": "<-base64->",
				"kid": "did:example:123#key-0",
				"alg": "EdDSA"
			},
			"signingRequest2": {
				"serializedPayload": "<-base64->",
				"alg": "ES256K",
				"purpose": "authentication" // describes the purpose of the requested signature
			}
		}
	},
	"didRegistrationMetadata": {},
	"didDocumentMetadata": {}
}
```

#### `didState.action="decryptPayload"`

This action indicates that the client needs to decrypt a payload, before the DID operation can be
continued.

This action is used in [Client Managed Secret Mode](#client-managed-secret-mode).

The [`didState` output field](#didstate) MUST contain a property `decryptionRequest` with a [Decryption Request Set](#decryption-request-set) data structure.

The [`didState` output field](#didstate) MAY contain additional properties that are relevant to this action.

Example:

```json
{
	"jobId": "1234",
	"didState": {
		"state": "action",
		"action": "decryptPayload",
		"decryptionRequest": {
			"decryptionRequest1": {
				"encryptedPayload": "<-base64->",
				"kid": null,
				"enc": "A128GCM"
			},
			"decryptionRequest2": {
				"encryptedPayload": "<-base64->",
				"kid": null,
				"enc": "A256GCM"
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

Example:

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

## DID URL Operations

In addition to the [DID Operations](#operations) defined in previous sections, this specification
also defines operations on resources identified by DID URLs.

### `createResource()`

```
createResource(did, relativeDidUrl, options, secret, data) -> jobId, didState, didRegistrationMetadata, resourceMetadata
```
___
This function creates a new DID URL and associated resource, according to a known DID method, using
various options and optionally the initial resource data.

### `updateResource()`

```
update(did, relativeDidUrl, options, secret, resourceOperation, data) -> jobId, didState, didRegistrationMetadata, resourceMetadata
```
____
This function updates the resource associated with the DID URL, either by completely replacing it, or
by performing an incremental update, or in another way. The specific update operation to be executed is specified in the
[`resourceOperation` input field](#resourceoperation).

### `deactivateResource()`

```
deactivateResource(did, relativeDidUrl, options, secret) -> jobId, didState, didRegistrationMetadata, resourceMetadata
```
___
This function deactivates the DID URL and associated resource.

## Data Structures

This specification defines a number of data structures that appear in the [input fields](#input-fields) and [output fields](#output-fields).

### Verification Method Public Data

This data structure is used as follows, when public data about a verification method is exchanged between the client and the DID Registrar:

- In [Client-managed Secret Mode](#client-managed-secret-mode) as a `verificationMethod` field inside the [`secret` input field](#secret),
  when the client invokes the DID Registrar again after it received a [`didState.action="getVerificationMethod"` output field](#didstateactiongetverificationmethod).
- In [Client-managed Secret Mode](#client-managed-secret-mode) as a `verificationMethod` field inside the [`didState.secret` output field](#didstatesecret),
  when the DID Registrar responds to a client request.

A **Verification Method Public Data** structure is a JSON object based on the verification method
data model as defined by [[spec:DID-CORE]], with the following differences:

* The `id` property is OPTIONAL.
  * If it is present, its value MUST match the `id` property of the corresponding verification method in the DID's
    associated DID document.
  * If it is absent, then the verification method does not correspond to any verification method in the DID's
    associated DID document.
* The JSON object MAY contain a property `purpose`, and the value of that property MUST be a JSON array, which contains
  verification relationships such as `authentication` or `assertionMethod`.

Example:

```json
{
	"id": "did:example:123#key-0",
	"type": "JsonWebKey2020",
	"controller": "did:example:123",
	"purpose": ["authentication", "assertionMethod", "capabilityDelegation", "capabilityInvocation"],
	"publicKeyJwk": {
		"kty": "EC",
		"crv": "secp256k1",
		"x": "htusHse5FMBnT_4266kn9T2yMmjDllwWvVSc_I2-WZ0",
		"y": "RjE_GjsRMELYJ6XuNSFDu3mCbyJnCQ26X_YtmyM9Bfo"
	}
}
```

Example:

```json
{
	"type": "Ed25519VerificationKey2020",
	"controller": "did:example:123",
	"purpose": ["recovery"],
	"publicKeyMultibase": "z5TVraf9itbKXrRvt2DSS95Gw4vqU3CHAdetoufdcKazA"
}
```

### Verification Method Private Data

This data structure is used as follows, when private data about a verification method is exchanged between the client and the DID Registrar:

- In [Internal Secret Mode](#internal-secret-mode) as a `verificationMethod` field inside the [`secret` input field](#secret),
  when the client invokes the DID Registrar again after it received a [`didState.action="getVerificationMethod"` output field](#didstateactiongetverificationmethod).
- In [Internal Secret Mode](#internal-secret-mode) as a `verificationMethod` field inside the [`didState.secret` output field](#didstatesecret),
  when the DID Registrar responds to a client request.

A **Verification Method Private Data** structure is a JSON object based on the [Verification Method Public Data](#verification-method-public-data)
structure, with the following difference:

* Instead of containing properties such as `publicKeyJwk` or `publicKeyMultibase` for expressing verification material,
  the verification method contains corresponding private key material, using properties such as `privateKeyJwk` or
  `privateKeyMultibase`.

Example:

```json
{
	"id": "did:example:123#key-0",
	"type": "JsonWebKey2020",
	"controller": "did:example:123",
	"purpose": [ "authentication", "assertionMethod", "capabilityDelegation", "capabilityInvocation" ],
	"privateKeyJwk": {
		"kty": "EC",
		"d": "-s-PwFdfgcdBPTDbJwZuiAFHCuI8r9vR13OGHo14--4",
		"crv": "secp256k1",
		"x": "htusHse5FMBnT_4266kn9T2yMmjDllwWvVSc_I2-WZ0",
		"y": "RjE_GjsRMELYJ6XuNSFDu3mCbyJnCQ26X_YtmyM9Bfo"
	}
}
```

Example:

```json
{
	"type": "Ed25519VerificationKey2020",
	"controller": "did:example:123",
	"purpose": ["recovery"],
	"privateKeyMultibase": "z5TVraf9itbKXrRvt2DSS95Gw4vqU3CHAdetoufdcKazA"
}
```

### Verification Method Template

This data structure is used as follows:

- In [Client-managed Secret Mode](#client-managed-secret-mode) as `verificationMethodTemplate` field inside the [`didState` output field](#didstate),
  when the DID Registrar responds to a client request with a [`didState.action="getVerificationMethod"` output field](#didstateactiongetverificationmethod).

A **Verification Method Template** structure is a JSON object based on the verification method
data model as defined by [[spec:DID-CORE]], with the following differences:

* The `id` property is OPTIONAL.
  * If it is present, its value MUST match the `id` property of the corresponding verification method in the DID's
    associated DID document.
  * If it is absent, then the verification method does not correspond to any verification method in the DID's
    associated DID document.
* The `type` property is OPTIONAL.
* The `controller` property is OPTIONAL.
* The JSON object MAY contain a property `purpose`, and the value of that property MUST be a JSON array, which contains
  verification relationships such as `authentication` or `assertionMethod`.
* The JSON object does not contain properties such as `publicKeyJwk` or `publicKeyMultibase` for expressing verification material.

Example **Verification Method Template** containing properties `id` and `type`:

```json
{
	"id": "#key-1",
	"type": "Ed25519VerificationKey2018"
}
```

Example **Verification Method Template** containing properties `purpose` and `type`:

```json
{
	"purpose": ["recovery"],
	"type": "EcdsaSecp256k1VerificationKey2019"
}
```

### Signing Request Set

This data structure is used as follows:

- In [Client-managed Secret Mode](#client-managed-secret-mode) as a `signingRequest` field inside the [`didState` output field](#didstate),
  when the DID Registrar responds to a client request with a [`didState.action="signPayload"` output field](#didstateactionsignpayload).

A **Signing Request Set** structure is a JSON object. Each property name in that JSON object is called a _signing request ID_, and
the corresponding property value MUST be a JSON object which is called the **Signing Request**.

A **Signing Request** contains the following properties:

* `payload`: The payload to be signed in a JSON form for informational purposes. This property is OPTIONAL.
* `serializedPayload`: The Base64-encoded byte array that represents the serialized payload to be signed. This property is REQUIRED.
* `kid`: This property is interpreted as in [[spec:RFC7517]] to indicate a specific key that should be used for signing. Example value: `did:example:123#key-0`. This property is OPTIONAL.
* `alg`: This property is interpreted as in [[spec:RFC7515]] to indicate the cryptographic algorithm to be used to sign the payload. Example values: `EdDSA`, `ES256K`, `PS256`. This property is REQUIRED.
* `purpose`: This property indicates the specific intent of the signing process. Example value: `authentication`. This property is OPTIONAL.

Example **Signing Request Set** containing two **Signing Requests** with IDs `signingRequest1` and `signingRequest2`:

```json
{
	"signingRequest": {
		"signingRequest1": {
			"payload": {
				...
			},
			"serializedPayload": "<-base64->",
			"kid": "did:example:123#key-0",
			"alg": "EdDSA"
		},
		"signingRequest2": {
			"serializedPayload": "<-base64->",
			"alg": "ES256K",
			"purpose": "authentication" // describes the purpose of the requested signature
		}
	}
}
```

### Signing Response Set

This data structure is used as follows:

- In [Client-managed Secret Mode](#client-managed-secret-mode) as a `signingResponse` field inside the [`secret` input field](#secret),
  when the client invokes the DID Registrar again after it received a [`didState.action="signPayload"` output field](#didstateactionsignpayload).

A **Signing Response Set** structure is a JSON object. Each property name MUST match a _signing request ID_ which was previously received by the
client in a [Signing Request Set](#signing-request-set). The corresponding property value MUST be a JSON object
which is called the **Signing Response**.

A **Signing Response** contains the following properties:

* `signature`: The Base64-encoded byte array that represents the signature of a payload. This property is REQUIRED.
* `kid`: This property is interpreted as in [[spec:RFC7517]] to indicate a specific key that was used for signing. Example value: `did:example:123#key-0`. This property is OPTIONAL, but SHOULD be included if an identifier for the key is known.
* `alg`: This property is interpreted as in [[spec:RFC7515]] to indicate the cryptographic algorithm that was used to sign the payload. Example values: `EdDSA`, `ES256K`, `PS256`. This property SHOULD be included.
* `purpose`: This property indicates the specific intent of the signing process. Example value: `authentication`. This property is OPTIONAL.

Example **Signing Response Set** containing two **Signing Responses**:

```json
{
	"signingResponse": {
		"signingRequest1": {
			"signature": "<-base64->",
			"kid": "did:example:123#key-0",
			"alg": "EdDSA"
		},
		"signingRequest2": {
			"signature": "<-base64->",
			"alg": "ES256K",
			"purpose": "authentication" // describes the purpose of the requested signature
		}
	}
}
```

### Decryption Request Set

This data structure is used as follows:

- In [Client-managed Secret Mode](#client-managed-secret-mode) as a `decryptionRequest` field inside the [`didState` output field](#didstate),
  when the DID Registrar responds to a client request with a [`didState.action="decryptPayload"` output field](#didstateactiondecryptpayload).

A **Decryption Request Set** structure is a JSON object. Each property name in that JSON object is called a _decryption request ID_, and
the corresponding property value MUST be a JSON object which is called the **Decryption Request**.

A **Decryption Request** contains the following properties:

* `payload`: The payload to be signed in a JSON form for informational purposes. This property is OPTIONAL.
* `encryptedPayload`: The Base64-encodedkey- byte array that represents the encrypted payload to be decrypted. This 0property is REQUIRED.
* `kid`: This property is interpreted as in [[spec:RFC7517]] to indicate a specific key that should be used for decryption. Example value: `did:example:123#key-1`. This property is OPTIONAL.
* `enc`: This property is interpreted as in [[spec:RFC7516]] to indicate the cryptographic algorithm to be used to decrypt the payload. Example values: `A128GCM`, `A256GCM`. This property is REQUIRED.
* `purpose`: This property indicates the specific intent of the decryption process. Example value: `parsing`. This property is OPTIONAL.

Example **Decryption Request Set** containing two **Decryption Requests** with IDs `decryptionRequest1` and `decryptionRequest2`:

```json
{
	"decryptionRequest1": {
		"encryptedPayload": "<-base64->",
		"kid": null,
		"enc": "A128GCM"
	},
	"decryptionRequest2": {
		"encryptedPayload": "<-base64->",
		"kid": null,
		"enc": "A256GCM"
	}
}
```

### Decryption Response Set

This data structure is used as follows:

- In [Client-managed Secret Mode](#client-managed-secret-mode) as a `decryptionResponse` field inside the [`secret` input field](#secret), 
  when the client invokes the DID Registrar again after it received a [`didState.action="decryptPayload"` output field](#didstateactiondecryptpayload).

A **Decryption Response Set** structure is a JSON object. Each property name MUST match a _decryption request ID_ which was previously received by the
client in a [Decryption Request Set](#decryption-request-set). The corresponding property value MUST be a JSON object
which is called the **Decryption Response**.

A **Decryption Response** contains the following properties:

* `decryptedPayload`: The Base64-encoded byte array that represents the decrypted payload. This property is REQUIRED.
* `kid`: This property is interpreted as in [[spec:RFC7517]] to indicate a specific key that was used for decrypting. Example value: `did:example:123#key-1`. This property is OPTIONAL, but SHOULD be included if an identifier for the key is known.
* `enc`: This property is interpreted as in [[spec:RFC7515]] to indicate the cryptographic algorithm that was used to decrypt the payload. Example values: `A128GCM`, `A256GCM`. This property SHOULD be included.
* `purpose`: This property indicates the specific intent of the decryption process. Example value: `parsing`. This property is OPTIONAL.

Example **Decryption Response Set** containing two **Decryption Responses**:

```json
{
	"decryptionRequest1": {
		"decryptedPayload": "<-base64->",
		"key": "did:example:123#key-1",
		"enc": "A128GCM"
	},
	"decryptionRequest2": {
		"decryptedPayload": "<-base64->",
		"key": "did:example:123#key-2",
		"enc": "A256GCM"
	}
}
```

## Extensions

This section documents various optional extension features that can complement the main functionality of a DID Registrar
to accommodate advanced use cases.

### `options.requestVerificationMethod`

Normally, when creating or updating a DID, only cryptographic keys that are required by the applicable DID method itself
are generated, in other words, keys that are needed for DID operations themselves. Depending on the DID method,
these keys may or may not appear in the DID document.

Irrespective of DID method-specific differences with regard to keys, it may sometimes be useful to add additional keys
to a DID document. This can be done by adding a `requestVerificationMethod` option. If present, this option MUST contain
a JSON array containing one or more [Verification Method Template](#verification-method-template) data structures.

Example:

```json
{
	"did": null,
	"options": {
		"requestVerificationMethod": [{
			"id": "#signingKey",
			"type": "Ed25519VerificationKey2018",
			"purpose": ["authentication", "assertionMethod"]
		}]
	},
	"secret": { ... },
	"didDocument": { ... }
}
```

In the above example, a client instructs a DID Registrar to add a verification method with `"id": "#signingKey"` and
`"type": "Ed25519VerificationKey2018"`, even if this verification method is not required for the DID operation itself.

### `didDocumentOperation="deactivate"`

The [`update() function`](#update) uses the [`didDocumentOperation` input field](#diddocumentoperation) to determine
what type of update operation should be applied to a DID's associated DID document. Deactivating a DID is normally
done by using the separate [`deactivate() function`](#deactivate).

However, it is also possible to model deactivation as a special type of update operation. For this purpose, this
section defines the value `deactivate` for the [`didDocumentOperation` input field](#diddocumentoperation). This can
be especially useful if atomic combinations of update and deactivate functions are required, since a single execution
of the [`update() function`](#update) can include multiple entries in the
[`didDocumentOperation` input field](#diddocumentoperation).

Example:

```json
{
	"did": "did:example:123",
	"options": { ... },
	"secret": { ... },
	"didDocumentOperation": ["removeFromDidDocument", "deactivate"],
	"didDocument": [{
		"verificationMethod": [{
			"id": "did:example:123#key-1"
		}]
	},
	null]
}
```

In the above example, a client instructs a DID Registrar to first remove a verification method from a DID document,
and then deactivate the DID.

## Architecture Considerations

In order to implement a library or tool that supports the above interfaces for
creating, updating, and deactivating DIDs in a method-agnostic way, we can imagine
a similar architecture as is common for a DID Resolver, i.e. using a set of drivers
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

## Normative References

[[spec]]

## Acknowledgements

<img align="left" src="images/logo-ngitrustchain.png" width="115">

Supported by [NGI TRUSTCHAIN](https://trustchain.ngi.eu/), which is made possible with financial support from the European Commission's [Next Generation Internet](https://ngi.eu/) programme.

## Appendix

### Other Resources

The following projects are working on DID registration across different DID methods.

* [DIF Universal Registrar](https://github.com/decentralized-identity/universal-registrar)
* [Universal Services](https://github.com/alenhorvat/universal-services)
* [ACA-py](https://github.com/hyperledger/aries-cloudagent-python/)
* [aries-framework-go](https://github.com/hyperledger/aries-framework-go)
* [Veramo](https://github.com/uport-project/veramo)
* [DIDKit](https://github.com/spruceid/didkit)
