# Client Libraries

We built a set of client libraries for Javascript/Typescript to help with integrating Tezos Domains. You can choose to use either [Taquito](https://tezostaquito.io/) or [ConseilJS](https://cryptonomic.github.io/ConseilJS) to power the client, depending on what you are already using. To learn more and get started check out the [API documentation](https://client-docs.tezos.domains/) and [example repository](https://gitlab.com/tezos-domains/examples).

We are using this library in our own dApp for all interactions with Smart Contracts to ensure it's functional and up to date. If you encounter any issues or have an idea for a new feature or improvement, feel free to create an [issue for us](https://gitlab.com/tezos-domains/client/issues).

## Packages

The functionality is split into two separate packages according to your particular use case:

### Resolver

This part can [resolve](https://client-docs.tezos.domains/interfaces/_tezos_domains_resolver.nameresolver-2.html) domain names to addresses and also the other way around by utilizing reverse records. You can also retrieve all domain metadata, which allows you to read arbitrary data that can be saved with a domain.

### Manager

With this, you can [execute operations](https://client-docs.tezos.domains/interfaces/_tezos_domains_manager.domainsmanager-2.html) that register or update domains, create subdomains, claim or update reverse records, interact with auctions, and more.

_\(manager functions are not yet supported in ConseilJS implementation\)_

