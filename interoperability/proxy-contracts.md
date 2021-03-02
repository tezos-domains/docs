# Proxy Contracts

To ensure future upgradeability while allowing clients to rely on fixed contract addresses, we offer a set of proxy contracts. Any contract or off-chain client can interact with them directly - the transactions are automatically routed to the correct destination.

You can find the addresses of the proxy contracts in the [Deployed Contracts](../deployed-contracts/edonet.md) section.

## Finding the Underlying Contract

Off-chain clients will often need to read data from an underlying contract. They can retrieve the individual addresses of underlying contracts from the proxy contract's storage. The generic storage structure follows:

{% tabs %}
{% tab title="CameLIGO" %}
```ocaml
type proxy_storage = {
    contract: address;

    (* ... more fields outside of this interoperability spec *)
}
```
{% endtab %}

{% tab title="Michelson" %}
```text
storage (pair
    (address %contract)
    (
        # ... more fields outside of this interoperability spec
    )
);
```
{% endtab %}
{% endtabs %}

{% hint style="warning" %}
Off-chain clients **must not** rely on a particular storage layout. They should always use annotations to find the correct value.
{% endhint %}

