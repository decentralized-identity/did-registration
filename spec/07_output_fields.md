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

This output field contains the result when the [`execute()` function](#execute) function is used to
execute an operation that does not affect the DID or DID document itself.

For the [`create()` function](#create), [`update()` function](#update), and [`deactivate()` function](#deactivate) this
output field MUST be absent.

For the [`execute()` function](#execute), this output field is REQUIRED and MUST contain a JSON array of JSON object values.

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
