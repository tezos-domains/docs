# Domain Updates

### Contract: NameRegistry.SetChildRecord

Entrypoint: `set_child_record`

| Parameter | Type | Description |
| :--- | :--- | :--- |
| **label** | `bytes` | UTF-8 encoded label |
| **parent** | `bytes` | UTF-8 encoded parent |
| **address** | `address option` | optional address the record resolves to |
| **owner** | `address` | the new owner of the record |
| **data** | `(string, data_value) map` | map of any additional data clients wish to store with the domain |
| **expiry** | `timestamp option` | The expiry of this record. This only applies to second-level domains. For all other cases, the value is ignored. |

This will create or overwrite an existing record. The current sender must to be the owner of the parent record.

{% tabs %}
{% tab title="CameLIGO" %}
```ocaml
type set_child_record_param = {
    label: bytes;
    parent: bytes;
    address: address option;
    owner: address;
    data: (string, data_value) map;
    expiry: timestamp option
}

(* Updates a child record. *)
| Set_child_record of set_child_record_param michelson_pair_left_comb
```
{% endtab %}

{% tab title="Michelson" %}
```text
TBD
```
{% endtab %}
{% endtabs %}

The call will **fail** in the following cases:

* the label fails validation
* parent record does not exist
* the owner of the parent record is not the current sender
* a non-zero amount is sent with this transaction

