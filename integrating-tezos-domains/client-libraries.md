# Client Libraries

We built a set of client libraries on top of [Taquito](https://tezostaquito.io/) to help wallets and other services integrate with Tezos Domains. To learn more and get started check out the [API documentation](https://client-docs.tezos.domains/).

{% hint style="info" %}
The packages are written in Typescript. If you are using a different language or don't want to use Taquito, you will need to write your own integration logic.
{% endhint %}

On a higher level, we split the functionality into multiple packages.

### Resolver

This part can resolve domain names to addresses and also the other way around by utilizing reverse records. You can also retrieve all domain metadata, which allows you to read arbitrary data that can be saved with a domain.

### Manager

With this you can execute operations that register or update domains, create subdomains, claim or update reverse records and more.

We are using this library in our dApp for all interaction with Smart Contracts to ensure it's functional and up to date. If you encounter any issues or have an idea for a new feature or improvement, feel free to create an [issue for us](https://gitlab.com/tezos-domains/client/issues).

