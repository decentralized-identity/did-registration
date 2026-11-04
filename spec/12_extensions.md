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
