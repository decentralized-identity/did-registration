## DID URL Operations

In addition to the [DID Operations](#operations) defined in previous sections, this specification
also defines operations on resources identified by DID URLs.

### `createResource()`

```
createResource(did, relativeDidUrl, options, secret, content) -> jobId, didUrlState, didRegistrationMetadata, contentMetadata
```
___
This function creates a new DID URL and associated resource, according to a known DID method, using
various options and optionally the initial resource data.

### `updateResource()`

```
updateResource(did, relativeDidUrl, options, secret, contentOperation, content) -> jobId, didUrlState, didRegistrationMetadata, contentMetadata
```
____
This function updates the content of the resource associated with the DID URL, either by completely replacing it, or
by performing an incremental update, or in another way. The specific update resource operation to be executed is specified in the
[`contentOperation` input field](#contentoperation).

### `deactivateResource()`

```
deactivateResource(did, relativeDidUrl, options, secret) -> jobId, didUrlState, didRegistrationMetadata, contentMetadata
```
___
This function deactivates the DID URL and associated resource.
