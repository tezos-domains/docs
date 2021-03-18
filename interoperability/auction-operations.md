# Auction Operations

### Contract: TLDRegistrar.Bid

Places a new highest bid on a domain that is currently [in auction](../design-document/top-level-domain-registrar.md#open-auction). The bid will record the indicated amount and the sender as the new highest bidder. The bid amount has to be higher than `round_to_nearest_tenth(previous_highest_bid * (100 + min_bid_increase_ratio) / 100)`.

Additionally:

* If there is a previous highest bid, it is replaced and its amount is credited to the previous highest bidder.
* Any excess amount sent with the bid is credited to the sender.
* If the current auction end is lower than `NOW + bid_additional_period`, it is updated to `NOW + bid_additional_period`.

**Entrypoint**: `bid`

**Amount restriction**: The amount sent in this transaction plus the in-contract balance of the sender has to be higher or equal to the bid amount.

| Parameter | Type | Description |
| :--- | :--- | :--- |
| **label** | `bytes` | The UTF-8 encoded label the bid relates to. |
| **bid** | `tez` | Ownership duration represented in days. |

{% tabs %}
{% tab title="CamelLIGO" %}
```ocaml
type bid_param = [@layout:comb] {
    label: bytes;
    bid: tez;
}

| Bid of bid_param
```
{% endtab %}
{% endtabs %}

| Error | Description |
| :--- | :--- |
| AUCTION\_ENDED | The auction has already ended. |
| LABEL\_TAKEN | The requested **label** already exists and it is not expired. |
| LABEL\_NOT\_AVAILABLE | The requested **label** is currently not available for registration. |
| INVALID\_LABEL | The given **label** is not valid. See [Label Validation](domain-operations.md#label-validation). |
| LABEL\_EMPTY | The given label is empty. |
| LABEL\_TOO\_LONG | The label is too long. |
| NAME\_TOO\_LONG | The name \(label + parent\) is too long. |
| BID\_TOO\_LOW | The bid amount does not meet the requirement for a new highest bid. |
| AMOUNT\_TOO\_LOW | The amount sent \(plus the in-contract balance\) does not cover the bid. |

### Contract: TLDRegistrar.Settle

Settles an auction that has ended. Removes the auction record and assigns the domain to the new owner. This call can only be made by the auction's highest bidder. 

The settlement period must not be expired, i.e. this call will only succeed during the period of `min_duration` days after an auction end.

**Entrypoint**: `settle`

| Parameter | Type | Description |
| :--- | :--- | :--- |
| **label** | `bytes` | The UTF-8 encoded label of the second-level domain to buy. |
| **owner** | `address` | The new owner of the given domain. |
| **address** | `address option` | The optional address the given domain resolves to. |
| **data** | `(string, bytes) map` | A map of any additional data clients wish to store with the given domain. |

{% tabs %}
{% tab title="CamelLIGO" %}
```ocaml
type settle_param = [@layout:comb] {
    label: bytes;
    owner: address;
    address: address option;
    data: data_map;
}

| Settle of settle_param
```
{% endtab %}
{% endtabs %}

| Error | Description |
| :--- | :--- |
| LABEL\_TAKEN | The requested **label** already exists and it is not expired. |
| LABEL\_NOT\_AVAILABLE | The requested **label** is currently not available for registration. |
| NOT\_SETTLEABLE | The settlement period is over or there is no such auction. |
| NOT\_AUTHORIZED | The call has not been made by the auction's highest bidder. |
| AMOUNT\_NOT\_ZERO | The transferred **amount** of _tez_ is not zero. |

### Contract: TLDRegistrar.Withdraw

Makes a withdrawal of the caller's full in-contract balance. If the caller has no in-contract balance, the operation does nothing.

**Entrypoint**: `withdraw`

| Type | Description |
| :--- | :--- |
| `address` | The address the caller's balance should be sent to. |

{% tabs %}
{% tab title="CamelLIGO" %}
```ocaml
type withdraw_param = address

| Widthdraw of withdraw_param
```
{% endtab %}
{% endtabs %}

| Error | Description |
| :--- | :--- |
| INVALID\_RECIPIENT | The recipient address is invalid. |
| AMOUNT\_NOT\_ZERO | The transferred **amount** of _tez_ is not zero. |

