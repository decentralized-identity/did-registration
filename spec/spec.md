DID Registration
================

**Specification Status:** Strawman

**Latest Draft:**
  [identity.foundation/did-registration](https://identity.foundation/did-registration)

**Editors:**
~ [Markus Sabadello](https://www.linkedin.com/in/markus-sabadello-353a0821/) (Danube Tech)
~ Others?
<!-- -->
**Authors:**
~ [Markus Sabadello](https://www.linkedin.com/in/markus-sabadello-353a0821/) (Danube Tech)
~ Others?
<!-- -->
**Participate:**
~ [GitHub repo](https://github.com/decentralized-identity/did-registration)
~ [File a bug](https://github.com/decentralized-identity/did-registration/issues)
~ [Commit history](https://github.com/decentralized-identity/did-registration/commits/master)

------------------------------------

## Abstract

Each DID method specifies how to create/read/update/deactivate DIDs within a given verifiable data
registry. How exactly this works can be very different depending on the DID method and may involve
various steps, architectural components, and network communication. This document is an attempt
to define the concept of a "DID registrar" which executes these DID write operations via a common
abstract interface, similar to how a "DID resolver" performs the DID resolution operation.

## Status of This Document

DID Registration is a draft specification under development within the
[Decentralized Identity Foundation](https://identity.foundation) (DIF (DIF), and designed to incorporate the
requirements and learnings from related work of the most active industry players
into a shared specification that meets the collective needs of the community.
This spec is regularly updated to reflect relevant changes, and we encourage
active engagement on GitHub (see above) and other mediums (e.g. DIF) where this
work is being done.

## Terminology

Term | Definition
:--- | :---------
Decentralized Identifier (DID) | A globally unique persistent identifier that does not require a centralized registration authority because it is generated and/or registered cryptographically.
DID Method | A definition of how a specific DID scheme implementeds the precise operations by which DIDs are created, resolved and deactivated and DID documents are written and updated.
DID Document | A set of data describing the DID subject, service and verification methods, that the DID subject or a DID delegate can use to authenticate itself and prove its association with the DID.
DID Registrar | A software and/or hardware component that performs the DID create/update/deactivate functions.
DID Resolver | A software and/or hardware component that performs the DID resolution function.
Wallet | A software and/or hardware component that can securely store DIDs and associated private keys and other sensitive cryptographic key material. Wallets implement various interfaces for cryptographic key generation, signing, verification, and other operations.

## Operations

### create()
___
`create(method, options) -> state, metadata`

`create(method, options, did-document) -> state, metadata`

`create(method, options, did-document, wallet) -> state, metadata`
___
To create a DID, we specify where we want it created, with optional parameters for registering a DID Document and storing keys.
___
`method`
 * sov, btcr, v1, ...

`options`
 * mainnet or testnet or other...
 * seed
 
`did-document`
 * entire new DID Document

`wallet`
 * storage for generated private keys
 * storage of existing keys
 * e.g. text file, wallet API endpoint, local wallet, etc.

### update()
____

`update(did, options, wallet, did-document) -> state, metadata`

`update(did, options, wallet, did-document-operation) -> state, metadata`
____
This function updates the DID document associated with the DID, either by completely replacing it, or by performing an incremental update (delta).
___
`did-document`
 * entire new DID Document, to replace the previous one

`did-document-operation`
 * incremental update to the existing DID Document, e.g.:
  * add-service
  * remove-service
  * add-publickey
  * remove-publickey

### deactivate()
___
`deactivate(did, options, wallet) -> state, metadata`
___
This function deactivates the DID.

## State

When executing the functions, an operation can turn into a longer-running "job" during which the operation's state might change, and multiple steps may have to be performed before the operation can be completed:

![](images/diagram-flow.png)

Explanation of states:

`state`

 * finished
   * DID
   * wallet {optional}
 * action
   * jobid
   * actiontype
 * wait
   * jobid
   * waittype
 * fail
   * error message

Possible uses for `action`:

  * Send coins to fund a wallet
  * Accept some legal agreement
  * Perform signature on a byte array

Possible uses for `wait`:

  * Wait for confirmation on chain
  * Wait for approval by someone

## Metadata

`metadata`

 * operation metadata
   * duration
 * method metadata
   * method-specific hash
   * token balance

## Architecture

In order to implement a library or tool that supports the above interfaces for creating, updating, and deactivating DIDs in a method-agnostic way, we can imagine a similar architecture as is common for a DID resolver, i.e. using a set of drivers that perform method-specific operations.

Some architectural questions that apply to the `resolve()` operation also apply to other operations, e.g.:

 * Is the abstract interface implemented as a library that can be integrated locally into an application or service, or is the abstract interface exposed by a remote service and used via HTTP or another binding?
 * How do method-specific drivers interact with the DID's target system? For example, do they have direct access to a blockchain full node?
 * What are implications of the above questions for trust and security?

Unlike the `resolve()` operation however, the other operations `create()`, `update()`, and `deactivate()` are more challenging and therefore raise additional architectural questions, since they typically involve the use of secrets such as private keys, and write operations to the DID's target system:

 * **Where are secrets generated?** Are a DID's private keys generated by the driver, or by the client that uses the Universal Registrar?
 * **Where are secrets stored?** Are a DID's private keys stored in a wallet held by the driver, or by the client that uses the Universal Registrar?
 * **Where are the identifiers generated?** Does the client generate the identifier (the DID) that gets created, or does this happen entirely inside the driver? Note that e.g. in the "btcr" DID method, the DID only becomes known at the end of the creation process, not at the beginning.

![There are different choices where wallets exist in the architecture.](images/diagram-wallets.png)

# HTTP Binding

The abstract interface above can be implemented and deployed in the form of bindings to different protocols, such as simple HTTP POST operations, with inputs and outputs encoded as JSON.

For example, the operations above can be deployed at the following endpoints:

```
https://uniregistrar.io/1.0/create
https://uniregistrar.io/1.0/update
https://uniregistrar.io/1.0/deactivate
```

See <a href="https://github.com/decentralized-identity/universal-registrar/blob/main/swagger/api.yml">for a Swagger definition</a>.

# Resources

The following projects are working on pluggable support for write operations across different DID methods.

* [DIF Universal Registrar](https://github.com/decentralized-identity/universal-registrar)
* [Universal Services](https://github.com/alenhorvat/universal-services)
* [ACA-py](https://github.com/hyperledger/aries-cloudagent-python/)
* [aries-framework-go](https://github.com/hyperledger/aries-framework-go)
* [Veramo](https://github.com/uport-project/veramo)
* [DIDKit](https://github.com/spruceid/didkit)
