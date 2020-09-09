# Buys & Renewals

### Contract: TLDRegistrar.Commit

Creates commitment to buy second-level domain without disclosing actual name. This is implemented according to [Commitment scheme](https://en.wikipedia.org/wiki/Commitment_scheme).

Entrypoint: `commit`

| Parameter Type | Description |
| :--- | :--- |
| `bytes` | SHA-512 hash of a packed tuple of **label** and **owner** corresponding to intended buy \(see below\). |

{% tabs %}
{% tab title="CamelLIGO" %}
```ocaml
type commit_param = bytes

| Commit of commit_param
```
{% endtab %}

{% tab title="Michelson" %}
```
TBD
```
{% endtab %}
{% endtabs %}

| Error | Description |
| :--- | :--- |
| TLD\_REGISTRAR\_DISABLED | This TLD registrar is disabled in its config. |
| AMOUNT\_NOT\_ZERO | Amount of _tez_ provided by sender is not zero. |

### Contract: TLDRegistrar.Buy

Buys second-level domain based on previous commitment \(see above\).

Entrypoint: `buy`

| Parameter | Type | Description |
| :--- | :--- | :--- |
| **label** | `bytes` | UTF-8 encoded label of the second-level domain to buy. |
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
TBD
```
{% endtab %}
{% endtabs %}

| Error | Description |
| :--- | :--- |
| TLD\_REGISTRAR\_DISABLED | This TLD registrar is disabled in its config. |
| AMOUNT\_NOT\_ZERO | Amount of _tez_ provided by sender is not zero. |
| COMMITMENT\_DOES\_NOT\_EXIST | Corresponding commitment \(see above\) was not created before. |
| COMMITMENT\_TOO\_OLD | The commitment is too old \(older than configured age\). Try recreating it again. |
| COMMITMENT\_TOO\_RECENT | The commitment is too recent \(younger than configured age\). Wait for some time. |
| LABEL\_NOT\_AVAILABLE | Requested label already exists and it is not expired. |
| DURATION\_TOO\_LOW | Requested duration is too low \(lower than configured minimum\). |
| AMOUNT\_TOO\_LOW | Transferred amount is lower than actual price. |
| ~~AMOUNT\_TOO\_HIGH~~ | TODO ~~Transferred amount is higher than actual price.~~ |

### Contract: TLDRegistrar.Renew

Renews second-level domain for requested duration.

Entrypoint: `renew`

| Parameter | Type | Description |
| :--- | :--- | :--- |
| **label** | `bytes` | UTF-8 encoded label of the second-level domain to buy. |
| **duration** | `nat` | Renewal duration represented in days. |

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
TBD
```
{% endtab %}
{% endtabs %}

| Error | Description |
| :--- | :--- |
| TLD\_REGISTRAR\_DISABLED | This TLD registrar is disabled in its config. |
| AMOUNT\_NOT\_ZERO | Amount of _tez_ provided by sender is not zero. |
| LABEL\_NOT\_EXIST | Requested label does not exist. |
| LABEL\_EXPIRED | Request label exists but it is expired. Therefore it can be bought, not renewed. |
| DURATION\_TOO\_LOW | Requested duration is too low \(lower than configured minimum\). |
| AMOUNT\_TOO\_LOW | Transferred amount is lower than actual price. |
| ~~AMOUNT\_TOO\_HIGH~~ | TODO ~~Transferred amount is higher than actual price.~~ |

