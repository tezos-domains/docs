# Domain Operations

### Contract: NameRegistry.CheckAddress

Checks that there is a valid domain record with the specified **name** and the **address**.

**Entrypoint**: `check_address`

| Parameter | Type | Description |
| :--- | :--- | :--- |
| **name** | `bytes` | The UTF-8 encoded name to check. |
| **address** | `address` | The expected address. |

{% tabs %}
{% tab title="CameLIGO" %}
```ocaml
type check_address_param = [@layout:comb] {
    name: bytes;
    address: address
}

(* Checks that a name corresponds to an address. *)
| Check_address of check_address_param
```
{% endtab %}

{% tab title="Michelson" %}
```text
parameter (or
  (pair %check_address
    (bytes %name)
    (address %address)
  # ... more entrypoints outside of this interoperability spec
);
```
{% endtab %}
{% endtabs %}

| Error | Description |
| :--- | :--- |
| AMOUNT\_NOT\_ZERO | The transferred **amount** of _tez_ is not zero. |
| NAME\_ADDRESS\_MISMATCH | There is no valid domain record with the specified **name** or it resolves to an **address** different from the expected one. |

### Contract: NameRegistry.ClaimReverseRecord

Claims a reverse record corresponding to a domain \(a forward record\). The claimed reverse record will map the **sender**'s address to the specified domain **name**.

**Entrypoint**: `claim_reverse_record`

| Parameter | Type | Description |
| :--- | :--- | :--- |
| **name** | `bytes` | The UTF-8 encoded name to claim. |
| **owner** | `address` | The owner of the record allowed to make changes. |

{% tabs %}
{% tab title="CameLIGO" %}
```ocaml
type claim_reverse_record_param = [@layout:comb] {
    name: bytes option;
    owner: address;
}

| Claim_reverse_record of claim_reverse_record_param
```
{% endtab %}
{% endtabs %}

| Error | Description |
| :--- | :--- |
| AMOUNT\_NOT\_ZERO | The transferred **amount** of _tez_ is not zero. |
| NAME\_ADDRESS\_MISMATCH | There is no domain record with the specified **name** or it resolves to an **address** different from the **sender**'s address. |

### Contract: NameRegistry.SetChildRecord

Creates or overwrites an existing domain record. The current **sender** must be the owner of the **parent** record.

If there was an existing corresponding reverse record referencing this domain and the address of this domain changed, the reverse record's name will be updated to `None` to preserve consistency.

**Entrypoint**: `set_child_record`

| Parameter | Type | Description |
| :--- | :--- | :--- |
| **label** | `bytes` | The UTF-8 encoded label. |
| **parent** | `bytes` | The UTF-8 encoded parent domain. |
| **address** | `address option` | The optional address the record resolves to. |
| **owner** | `address` | The owner of the record allowed to make changes. |
| **data** | `(string, bytes) map` | A map of any additional data clients wish to store with the domain. |
| **expiry** | `timestamp option` | The expiry of this record. Only applicable to second-level domains as all higher-level domains share the expiry of their ancestor 2LD. |

{% tabs %}
{% tab title="CameLIGO" %}
```ocaml
type set_child_record_param = [@layout:comb] {
    label: bytes;
    parent: bytes;
    address: address option;
    owner: address;
    data: (string, bytes) map;
    expiry: timestamp option;
}

| Set_child_record of set_child_record_param
```
{% endtab %}
{% endtabs %}

| Error | Description |
| :--- | :--- |
| AMOUNT\_NOT\_ZERO | The transferred **amount** of _tez_ is not zero. |
| PARENT\_NOT\_FOUND | There is no record for the specified **parent** domain. |
| NOT\_AUTHORIZED | The current **sender** is not the current record owner. |
| INVALID\_LABEL | The given **label** is not valid. See [Label Validation](.tez-tld.md#label-validation-for-tez). |
| LABEL\_EMPTY | The given label is empty. |
| LABEL\_TOO\_LONG | The label is too long. |
| NAME\_TOO\_LONG | The name \(label + parent\) is too long. |

### Contract: NameRegistry.UpdateRecord

Updates an existing domain record. The current **sender** must be its owner.

If there was an existing corresponding reverse record referencing this domain and the address of this domain changed, the reverse record's name will be updated to `None` to preserve consistency.

**Entrypoint**: `update_record`

| Parameter | Type | Description |
| :--- | :--- | :--- |
| **name** | `bytes` | The UTF-8 encoded name of the domain to update. |
| **address** | `address option` | The optional new address the record resolves to. |
| **owner** | `address` | The new owner of the record allowed to make changes. |
| **data** | `(string, bytes) map` | The new map of any additional data that clients wish to store with the domain. |

{% tabs %}
{% tab title="CameLIGO" %}
```ocaml
type update_record_param = [@layout:comb] {
    name: bytes;
    address: address option;
    owner: address;
    data: (string, bytes) map;
}

| Update_record of update_record_param
```
{% endtab %}
{% endtabs %}

| Error | Description |
| :--- | :--- |
| AMOUNT\_NOT\_ZERO | The transferred **amount** of _tez_ is not zero. |
| RECORD\_NOT\_FOUND | There is no domain record for the specified **name**. |
| NOT\_AUTHORIZED | The current **sender** is not the current record owner. |

### Contract: NameRegistry.UpdateReverseRecord

Updates an existing reverse record. The current **sender** must be its owner. There must be a corresponding domain record.

**Entrypoint**: `claim_reverse_record`

| Parameter | Type | Description |
| :--- | :--- | :--- |
| **address** | `address` | The address of the reverse record to update. |
| **name** | `bytes option` | The new UTF-8 encoded name the record resolves to. |
| **owner** | `address` | The owner of the record allowed to make changes. |

{% tabs %}
{% tab title="CameLIGO" %}
```ocaml
type update_reverse_record_param = [@layout:comb] {
    address: address;
    name: bytes option;
    owner: address;
}

| Update_reverse_record of update_reverse_record_param
```
{% endtab %}
{% endtabs %}

| Error | Description |
| :--- | :--- |
| AMOUNT\_NOT\_ZERO | The transferred **amount** of _tez_ is not zero. |
| RECORD\_NOT\_FOUND | There is no reverse record with the specified **address**. |
| NOT\_AUTHORIZED | The current **sender** is not the current record owner. |
| NAME\_ADDRESS\_MISMATCH | There is no domain record with the specified **name** or it resolves to a different **address**. This can occur only if the **name** is specified. |

