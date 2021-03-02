# Client Libraries

We built a set of client libraries on top of help wallets and other services integrate with Tezos Domains. You can choose to use either [Taquito](https://tezostaquito.io/) or [ConseilJS](https://cryptonomic.github.io/ConseilJS) to power the client, depending on what you are already using. To learn more and get started check out the [API documentation](https://client-docs.tezos.domains/) and [example repository](https://gitlab.com/tezos-domains/examples).

{% hint style="info" %}
The packages are written in Typescript. If you are using a different language, you will need to write your own integration logic.
{% endhint %}

We are using this library in our own dApp for all interactions with Smart Contracts to ensure it's functional and up to date. If you encounter any issues or have an idea for a new feature or improvement, feel free to create an [issue for us](https://gitlab.com/tezos-domains/client/issues).

On a higher level, we split the functionality into multiple packages:

### Resolver

This part can [resolve](https://client-docs.tezos.domains/interfaces/_tezos_domains_resolver.nameresolver-2.html) domain names to addresses and also the other way around by utilizing reverse records. You can also retrieve all domain metadata, which allows you to read arbitrary data that can be saved with a domain.

### Manager

With this, you can [execute operations](https://client-docs.tezos.domains/interfaces/_tezos_domains_manager.domainsmanager-2.html) that register or update domains, create subdomains, claim or update reverse records, interact with auctions, and more.

_\(manager functions are not yet supported in ConseilJS implementation\)_

