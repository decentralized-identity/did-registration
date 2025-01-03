## Architecture Considerations

In order to implement a library or tool that supports the above interfaces for
creating, updating, and deactivating DIDs in a method-agnostic way, we can imagine
a similar architecture as is common for a DID Resolver, i.e. using a set of drivers
that perform method-specific operations.

Some architectural questions that apply to a DID Resolver also apply to a DID Registrar, e.g.:

* Is the abstract interface implemented as a library that can be integrated
  locally into an application or service, or is the abstract interface exposed by a remote
  service and used via HTTP or another binding?
* How do method-specific drivers interact with the DID's target system? For example,
  do they have direct access to a blockchain full node?
* What are implications of the above questions for trust and security?
