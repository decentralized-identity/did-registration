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
* `encryptedPayload`: The Base64-encoded byte array that represents the encrypted payload to be decrypted. This 0property is REQUIRED.
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
