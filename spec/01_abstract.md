## Abstract

Each DID method specifies how to create/resolve/update/deactivate DIDs within a given verifiable data
registry. How exactly this works can be very different depending on the DID method and may involve
various steps, architectural components, and network communication. The process of resolving a DID
to a DID document by executing the read() operation is well-understood and specified in the
[DID Resolution](https://www.w3.org/TR/did-resolution/) specification. This document complements
the concept of a "DID Resolver" by also defining a "DID Registrar" component that can execute the three remaining
create/update/deactivate operations via a common interface.

This specification does NOT specify a DID method. It does NOT specify how to build a verifiable data
registry, only a "DID Registrar" component that can interact with a verifiable data registry. This
specification also does NOT suggest that private keys should be controlled by any other entity than the
DID controller.
