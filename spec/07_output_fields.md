## Output Fields

### `didState`

This output field contains a JSON object with the following fields, which contain information about the current
state of the DID:

* [`didState.state`](#didstatestate): The current state of DID operations.
* [`didState.did`](#didstatedid): The DID at the end of the DID operation.
* [`didState.secret`](#didstatesecret): An object with DID controller keys and other secrets.
* [`didState.didDocument`](#didstatediddocument): The DID document after the DID operation has been successfully executed.

In [Client-managed Secret Mode](#client-managed-secret-mode), this output field MAY contain one or more of the following:

* A `verificationMethodTemplate` property with a JSON array containing one or more [Verification Method Template](#verification-method-template) objects.
* A `signingRequest` property with a [Signing Request Set](#signing-request-set).
* A `decryptionRequest` property with a [Decryption Request Set](#decryption-request-set).

See [States](#states) for further details about how this output field is used.

#### `didState.state`

This output field contains the current state of the DID. It is used to indicate if a DID operation
is finished, failed, or if a longer-running process has been created that requires additional steps.

This specification defines four well-known values for this property: `finished`, `failed`, `action`, `wait`.

#### `didState.did`

This output field contains the DID that is the target of the operation.

For the [`create()` function](#create):

- If the value of the [`didState.state` output field](#didstatestate) is `"finished"`, the value of this output field
  MUST NOT be null and MUST contain the final DID that has been created.
- Otherwise, the value of this output field may or may not contain the DID, depending on the DID method.

For the [`update()` function](#update) and [`deactivate()` function](#deactivate):

- This output field MUST NOT be null, and its value MUST match the [`did` input field](#did) that was used when
  executing the function.

#### `didState.secret`

This output field contains a JSON object with DID controller keys and other secrets.

In [Internal Secret Mode](#internal-secret-mode), if the [`returnSecrets` option](#returnsecrets) is set to `true`:

- This output field MAY contain a `verificationMethod` property  with a JSON array containing one or more [Verification Method Private Data](#verification-method-private-data) objects.

Example:

```json
{
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
	"operationState": null,
	"didRegistrationMetadata": { ... },
	"didDocumentMetadata": { ... }
}
```

In [Client-managed Secret Mode](#client-managed-secret-mode):

- This output field MAY contain a `verificationMethod` property with a JSON array containing one or more [Verification Method Public Data](#verification-method-public-data) objects.

Example:

```json
{
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
	"operationState": null,
	"didRegistrationMetadata": { ... },
	"didDocumentMetadata": { ... }
}
```

This output field MAY contain additional properties that are considered secrets, such as seeds, passwords, etc.

#### `didState.didDocument`

This output field contains the DID document after the DID operation has been successfully executed.

This output field is OPTIONAL.

Example:

```json
{
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
	"operationState": null,
	"didRegistrationMetadata": { ... },
	"didDocumentMetadata": { ... }
}
```

### `operationState`

This output field contains data related to the current state of the DID operation. It matches the
[`operationState` input field](#operationstate).

If present, this output field MUST contain either a string or a JSON object. The type and meaning of the value is
DID method-specific and not further specified here.

If the value of the [`didState.state` output field](#didstatestate) is either `finished` or `failed`:

- This output field MUST be absent or have a `null` value.

See [States](#states) for further details about how this output field is used.

Example:

```json
{
	"didState": { ... },
	"operationState": "155eae21-45e5-4e71-bb22-fef51cda5bf7",
	"didRegistrationMetadata": { ... },
	"didDocumentMetadata": { ... }
}
```

Example:

```json
{
	"didState": { ... },
	"operationState": {
		"currentStep": 3,
		"someData": { ... }
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

This output field contains the result when the [`execute()` function](#execute) function is used to
execute an operation that does not affect the DID or DID document itself.

For the [`create()` function](#create), [`update()` function](#update), and [`deactivate()` function](#deactivate) this
output field MUST be absent.

For the [`execute()` function](#execute), this output field is REQUIRED and MUST contain a JSON array of JSON object values.

Example:

```json
{
	"didState": { ... },
	"operationState": null,
	"operationResult": [{
		"myResult": "myResultValue"
	}],
	"didRegistrationMetadata": { ... },
	"didDocumentMetadata": { ... }
}
```
