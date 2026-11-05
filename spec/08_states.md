## States

When executing the `create()`, `update()`, and `deactivate()` DID operations, an operation can turn into a
longer-running process potentially consisting of multiple
steps. The
[DID Registrar functions](#operations)
may have to be invoked multiple times until the DID operation is completed. In each step,
additional information such as public key material or signatures may be required, and the DID operation's state
might change.

The [`didState` output field](#didstate) is used by the DID Registrar to indicate to the client the current state of
this process, and how to proceed to a next step if needed.

The [`operationState` input field](#operationstate) and [`operationState` output field](#operationstate-2) contain
DID-method specific information that is used internally by the DID Registrar to keep track of the ongoing process. They
fulfill a similar function as a cookie in web architecture and allow a DID Registrar to correlate multiple invocations
by the client and maintain state between them.

If the value of the [`didState.state` output field](#didstatestate) is either `finished` or `failed`:

- The DID operation is complete, and no further invocations of the [DID Registrar functions](#operations) are required.

If the value of the [`didState.state` output field](#didstatestate) is either `action` or `wait`:

- The client needs to perform an external action such as signing a payload, or it needs to wait for a certain period of
  time or until a certain event occurs.
- After that, the client prepares new inputs and invokes the [DID Registrar functions](#operations) again.

The following diagram illustrates the states while executing a `create()` DID operation.

![Diagram showing the flow and states of DID operations](images/diagram-flow.png)

When a client needs to invoke the [DID Registrar functions](#operations) again, the following **State Processing Algorithm**
is used for processing
the current state and preparing new inputs. Given _previousInputs_ and _previousOutputs_, the algorithm determines
the _nextInputs_ for the next invocation of the [DID Registrar functions](#operations).

1. Initialize _nextInputs_ by copying all [input fields](#input-fields) from _previousInputs_.
2. If present, copy `previousOutputs.`[`didState.did`](#didstatedid) to `nextInputs.`[`did`](#did).
3. If present, copy `previousOutputs.`[`didState.didDocument`](#didstatediddocument) to `nextInputs.`[`didDocument`](#diddocument).
4. If present, copy `previousOutputs.`[`operationState`](#operationstate-2) to `nextInputs.`[`operationState`](#operationstate).

The following sections explain the possible states of a DID registration process, which are returned as values of the
[`didState.state` output field](#didstatestate).

### `didState.state="finished"`

This state indicates that the DID operation has been completed, and that
no further invocations of the [DID Registrar functions](#operations) are required.

Example:

```json
{
	"didState": {
		"state": "finished",
		"did": "did:key:z6MknhhUUtbXCLRmUVhYG7LPPWN4CTKWXTLsygHMD6Ah5uDN",
		"secret": { ... },
		"didDocument": { ... }
	},
	"operationState": null,
	"didRegistrationMetadata": { ... },
	"didDocumentMetadata": { ... }
}
```

### `didState.state="failed"`

This state indicates that the DID operation has failed, and that
no further invocations of the [DID Registrar functions](#operations) are required.

In this state, the [`didState` output field](#didstate) MUST contain a `reason` property that indicates the reason
for the failure.

The [`didState` output field](#didstate) MAY contain additional properties that are relevant to this state.

Example:

```json
{
	"didState": {
		"state": "failed",
		"did": "did:key:z6MknhhUUtbXCLRmUVhYG7LPPWN4CTKWXTLsygHMD6Ah5uDN",
		"reason": "networkUnavailable"
	},
	"operationState": null,
	"didRegistrationMetadata": { ... },
	"didDocumentMetadata": { ... }
}
```

### `didState.state="action"`

This state indicates that the client needs to perform an action, before the DID operation can be continued.

Possible uses for `didState.state="action"`:

* Client needs to generate a new cryptographic key pair
* Client needs to cryptographically sign a payload
* Client needs to send cryptocurrency to fund a wallet addres
* Client needs to obtain an OAuth2 access token from an external service
* Client needs to accept the terms of a legal agreement

In this state, the [`didState` output field](#didstate) MUST contain an `action` property that indicates the type of
action that needs to be performed.

The [`didState` output field](#didstate) MAY contain additional properties that are relevant to this state.

This specification defines the following well-known values for the `action` property that may be used in this state:

* [`didState.action="getVerificationMethod"`](#didstateactiongetverificationmethod) - Client needs to provide a newly generated or already existing verification method.
* [`didState.action="signPayload"`](#didstateactionsignpayload) - Client needs to generate a signature on a payload.
* [`didState.action="decryptPayload"`](#didstateactiondecryptpayload) - Client needs to decrypt a payload.

Example:

```json
{
	"didState": {
		"state": "action",
		"action": "fundingRequired",
		"description": "Please fund the address mzUC2F1XgXfTJEYUBZXdG6M8wWWvhgEknG."
	},
	"operationState": "155eae21-45e5-4e71-bb22-fef51cda5bf7",
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
	"didState": {
		"state": "action",
		"action": "getVerificationMethod",
		"verificationMethodTemplate": [{
			"id": "#key-1",
			"type": "Ed25519VerificationKey2018"
		}]
	},
	"operationState": null,
	"didRegistrationMetadata": {},
	"didDocumentMetadata": {}
}
```

Example:

```json
{
	"didState": {
		"state": "action",
		"action": "getVerificationMethod",
		"verificationMethodTemplate": [{
			"purpose": ["recovery"],
			"type": "EcdsaSecp256k1VerificationKey2019"
		}]
	},
	"operationState": null,
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
	"operationState": "155eae21-45e5-4e71-bb22-fef51cda5bf7",
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
	"operationState": "155eae21-45e5-4e71-bb22-fef51cda5bf7",
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
	"didState": {
		"state": "wait",
		"wait": "Please wait until the transaction is complete.",
		"waitTime": 3600000
	},
	"operationState": "155eae21-45e5-4e71-bb22-fef51cda5bf7",
	"didRegistrationMetadata": { ... },
	"didDocumentMetadata": { ... }
}
```
