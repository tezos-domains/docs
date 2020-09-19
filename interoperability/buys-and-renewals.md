# Buys & Renewals

### Contract: TLDRegistrar.Commit

Creates a commitment to buy a second-level domain without disclosing the actual name. This is implemented according to our [commit&reveal](https://en.wikipedia.org/wiki/Commitment_scheme) scheme.

Entrypoint: `commit`

| Parameter Type | Description |
| :--- | :--- |
| `bytes` | SHA-512 hash of a packed tuple of **label** and **owner** corresponding to intended buy \(see [TLDRegistrar.Buy](buys-and-renewals.md#contract-tldregistrar-buy)\). |

{% tabs %}
{% tab title="CamelLIGO" %}
```ocaml
type commit_param = bytes

| Commit of commit_param
```
{% endtab %}

{% tab title="Michelson" %}
```
parameter (or
  (bytes %commit)
  # ... more endpoints outside of this interoperability spec
);
```
{% endtab %}
{% endtabs %}

| Error | Description |
| :--- | :--- |
| TLD\_REGISTRAR\_DISABLED | This TLD registrar is disabled in its config. |
| AMOUNT\_NOT\_ZERO | The transferred **amount** of _tez_ is not zero. |

### Contract: TLDRegistrar.Buy

Buys a second-level domain based on previous commitment \(see [TLDRegistrar.Commit](buys-and-renewals.md#contract-tldregistrar-commit)\).

Entrypoint: `buy`

| Parameter | Type | Description |
| :--- | :--- | :--- |
| **label** | `bytes` | The UTF-8 encoded label of the second-level domain to buy. |
| **duration** | `nat` | Ownership duration represented in days. |
| **owner** | `address` | The new owner of the given domain. |

{% tabs %}
{% tab title="CamelLIGO" %}
```ocaml
type buy_param = {
    label: bytes;
    duration: nat;
    owner: address
}

| Buy of buy_param michelson_pair_left_comb
```
{% endtab %}

{% tab title="Michelson" %}
```
parameter (or
  (pair %buy
    (pair (bytes %label) (nat %duration))
    (address %owner))
  # ... more endpoints outside of this interoperability spec
);
```
{% endtab %}
{% endtabs %}

| Error | Description |
| :--- | :--- |
| TLD\_REGISTRAR\_DISABLED | This TLD registrar is disabled in its config. |
| COMMITMENT\_DOES\_NOT\_EXIST | Corresponding commitment \(see [TLDRegistrar.Commit](buys-and-renewals.md#contract-tldregistrar-commit)\) was not created before. |
| COMMITMENT\_TOO\_OLD | The commitment is too old \(older than configured age\). Try recreating it again. |
| COMMITMENT\_TOO\_RECENT | The commitment is too recent \(younger than configured age\). Wait for some time. |
| LABEL\_NOT\_AVAILABLE | The requested **label** already exists and it is not expired. |
| INVALID\_LABEL | The given **label** is not valid. See [Label Validation](domain-updates.md#label-validation). |
| DURATION\_TOO\_LOW | The requested **duration** is too low \(lower than the configured minimum\). |
| AMOUNT\_TOO\_LOW | The transferred **amount** is lower than the actual price. |
| AMOUNT\_TOO\_HIGH | The transferred **amount** is higher than the actual price. |

### Contract: TLDRegistrar.Renew

Renews second-level domain for requested duration.

Entrypoint: `renew`

| Parameter | Type | Description |
| :--- | :--- | :--- |
| **label** | `bytes` | The UTF-8 encoded label of the second-level domain to buy. |
| **duration** | `nat` | The renewal duration represented in days. |

{% tabs %}
{% tab title="CamelLIGO" %}
```ocaml
type renew_param = {
    label: bytes;
    duration: nat
}

| Renew of renew_param michelson_pair_left_comb
```
{% endtab %}

{% tab title="Michelson" %}
```
parameter (or
  (pair %renew
    (bytes %label)
    (nat %duration))
  # ... more endpoints outside of this interoperability spec
);
```
{% endtab %}
{% endtabs %}

| Error | Description |
| :--- | :--- |
| TLD\_REGISTRAR\_DISABLED | This TLD registrar is disabled in its config. |
| LABEL\_NOT\_EXIST | The requested **label** does not exist. |
| LABEL\_EXPIRED | The requested **label** exists but it is expired. Therefore it can be bought, not renewed. |
| DURATION\_TOO\_LOW | The specified **duration** is too low \(lower than the configured minimum\). |
| AMOUNT\_TOO\_LOW | The transferred **amount** is lower than the actual price. |
| AMOUNT\_TOO\_HIGH | The transferred **amount** is higher than the actual price. |

