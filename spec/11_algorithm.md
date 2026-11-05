## Algorithm

This section defines a **State Processing Algorithm** that clients can use after invoking the [DID Registrar functions](#operations),
in order to process the outputs including current state (see [States](#states)). If the DID operation consists of
multiple steps, the [DID Registrar functions](#operations) may have to be invoked multiple times until a DID operation is completed.

If the value of the [`didState.state` output field](#didstatestate) is either [`finished`](#didstatestatefinished) or [`failed`](#didstatestatefailed):

- The DID operation is complete, and no further invocations of the [DID Registrar functions](#operations) are required.

If the value of the [`didState.state` output field](#didstatestate) is either [`action`](#didstatestateaction) or [`wait`](#didstatestatewait):

- The client needs to perform an action, or it needs to wait for a certain period of
  time or until a certain event occurs.
- After that, the client prepares new inputs and invokes the [DID Registrar functions](#operations) again.

In the latter case, the current state is processed and new inputs are prepared. Given _previousInputs_ and _previousOutputs_, the algorithm determines
_nextInputs_ for the next invocation of the [DID Registrar functions](#operations).

1. Initialize _nextInputs_ by copying all [input fields](#input-fields) from _previousInputs_.
2. If present, copy `previousOutputs.`[`didState.did`](#didstatedid) to `nextInputs.`[`did`](#did).
3. If present, copy `previousOutputs.`[`didState.didDocument`](#didstatediddocument) to `nextInputs.`[`didDocument`](#diddocument).
4. If present, copy `previousOutputs.`[`operationState`](#operationstate-2) to `nextInputs.`[`operationState`](#operationstate).
5. If [`didState.state="action"`](#didstatestateaction) and [`didState.action="getVerificationMethod"`](#didstateactiongetverificationmethod)
   1. For each [Verification Method Template](#verification-method-template) in the [`didState` output field](#didstate), generate or retrieve a key pair,
      and add a [Verification Method Public Data](#verification-method-public-data) to the [`didDocument` input field](#diddocument) or [`secret` input field](#secret).
6. If [`didState.state="action"`](#didstatestateaction) and [`didState.action="signPayload"`](#didstateactionsignpayload)
   1. From the [Signing Request Set](#signing-request-set) in the [`didState` output field](#didstate), sign the payload(s),
      and add the [Signing Response Set](#signing-response-set) to the [`secret` input field](#secret).
7. If [`didState.state="action"`](#didstatestateaction) and [`didState.action="decryptPayload"`](#didstateactiondecryptpayload)
    1. From the [Decryption Request Set](#decryption-request-set) in the [`didState` output field](#didstate), decrypt the payload(s),
       and add the [Decryption Response Set](#decryption-response-set) to the [`secret` input field](#secret).

::: todo
Explain that the generated verification method must then be included in either "secret" or "didDocument" in next request.
:::
