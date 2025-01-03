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
