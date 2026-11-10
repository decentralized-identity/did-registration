## Operations

This section defines interfaces for the standard DID operations `create()`, `update()`, and `deactivate()`, 
as well as a fourth `execute()`.

The signatures of these function are to be understood as abstract interfaces
for the specified DID operations, with logical named inputs and outputs. Conforming
<a>DID URL dereferencing</a> implementations are not required to
implement this function using the same names of the inputs, outputs,
or of the function itself, nor do they need to implement the algorithm in
any specific form, such as an explicit function call in a programming language,
a software library, or a protocol endpoint.
</p>
<p>
<a>Bindings</a> such as the <a href="#bindings-https">HTTP(S) binding</a>
can be used to define specific ways of implementing and exposing this function.



### `create()`

```
create(method, operationState, options, secret, didDocument) -> didState, operationState, didRegistrationMetadata, didDocumentMetadata
```
___
This function creates a new DID and associated DID document, according to a known DID method, using
various options and optionally an initial DID document.

### `update()`

```
update(did, operationState, options, secret, didDocumentOperation, didDocument) -> didState, operationState, didRegistrationMetadata, didDocumentMetadata
```
____
This function updates the DID document associated with the DID, either by completely replacing it, or
by performing an incremental update, or in another way. The specific update operation to be executed is specified in the
[`didDocumentOperation` input field](#diddocumentoperation).

### `deactivate()`

```
deactivate(did, operationState, options, secret) -> didState, operationState, didRegistrationMetadata, didDocumentMetadata
```
___
This function deactivates the DID and associated DID document.

### `execute()`

```
execute(did, operationState, options, secret, operation, operationData) -> didState, operationState, operationResult, didRegistrationMetadata, didDocumentMetadata
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

