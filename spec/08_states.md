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
