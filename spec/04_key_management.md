## Key Management

The DID create/update/deactivate functions raise architectural questions around key management,
since they typically involve the generation and use of private keys and other secrets.

With regard to key management, a DID Registrar can operate in the following modes:

### Internal Secret Mode

In this mode, the DID Registrar is responsible for generating the DID controller keys used by DID operations. This
means  that the DID Registrar is considered a highly trusted component which should be fully under the control of a
DID controller. If it is operated as a remotely hosted service, secure connection protocols such as TLS, DIDComm,
etc. MUST be used.

This mode has two options that control how DID controller keys are handled:
[`storeSecrets`](#storesecrets) and [`returnSecrets`](#returnsecrets).

The `storeSecrets` and `returnSecrets` options can be enabled or disabled independently. A DID Registrar
may define default values for these options, and/or it may allow a client to set them via
the [`options` input field](#options).

Note that if neither option is enabled, then control over a DID may get permanently lost, since the DID Registrar
operating in [Internal Secret Mode](#internal-secret-mode) will generate DID controller keys internally, but it will
neither store them nor return them to a client.

If a DID Registrar is configured with options `storeSecrets=false` and `returnSecrets=true`, then a DID Registrar
with option `storeSecrets=true` can be simulated by building a "wrapping DID Registrar" around an
"inner DID Registrar".

<img alt="Diagram showing Internal Secret Mode" src="images/diagram-mode-internal-secret.png">

### External Secret Mode

In this mode, the DID Registrar does not itself have access to the secrets used by DID operations, but it has a way
of accessing an external wallet in order to perform cryptographic operations such as generating signatures.

The interface between the DID Registrar and an external wallet is out of scope for this specification, but could
potentially use wallet or key management interfaces defined by other specifications, e.g.:

- [Universal Wallet](https://w3c-ccg.github.io/universal-wallet-interop-spec/)
- [WebKMS](https://w3c-ccg.github.io/webkms/)
- [WebCrypto API](https://w3c.github.io/webcrypto/#subtlecrypto-interface)
- others...

If additional information (such as a URL, or tenant ID, or secret key, etc.) is required in order for the DID Registrar
to be able to access the external wallet, then such information could be preconfigured in the DID Registrar, or
alternatively supplied by a client in the [`options`](#options) and/or [`secret`](#secret) input fields.

<img alt="Diagram showing External Secret Mode" src="images/diagram-mode-external-secret.png">

### Client-managed Secret Mode

In this mode, the DID Registrar does not itself have access to the secrets used by DID operations, but it will ask
the client to perform cryptographic operations such as generating signatures.

::: todo
Explain how the did:ion use case fits in, where the client supplies the public keys / commitments during the create operation.
:::

This mode has one option that controls how DID controller keys are handled:
[`clientSecretMode`](#clientsecretmode).

<img alt="Diagram showing Client-Managed Secret Mode" src="images/diagram-mode-client-managed-secret.png">
