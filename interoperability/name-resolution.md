# Name Resolution

## NameRegistry

The `NameRegistry` contract provides forward and reverse resolution.

Clients retrieve the current address of `NameRegistry` by reading it from the storage of the proxy contract [NameRegistry.CheckAddress](domain-operations.md#contract-nameregistry-checkaddress) \(as explained in the [Proxy Contracts](proxy-contracts.md#finding-the-underlying-contract) chapter\).

### Resolution by off-chain clients with view support \(recommended\)

Clients that have the ability to invoke [TZIP-16](https://gitlab.com/tzip/tzip/-/blob/master/proposals/tzip-16/tzip-16.md) views should use the following to resolve names and addresses.

#### View: resolve-name

Resolves a name to an address, optionally [other domain data](../design-document/domain-data.md), and expiry information for reference. If no such record exists or it has expired, it returns `None`.

Before passing a name for resolution, it should first be normalized using the [encode algorithm](name-resolution.md#name-validation-and-normalization).

| Parameter Type | Description |
| :--- | :--- |
| `bytes` | The UTF-8 encoded name to resolve. |

#### View: resolve-address

Resolves an address to a name, optionally [other domain data](../design-document/domain-data.md), and expiry information for reference. If no such record exists or it has expired, it returns `None`.

| Parameter Type | Description |
| :--- | :--- |
| `address` | The address to resolve. |

#### **Return type**

The return type for both `resolve-name` and `resolve-address` is as follows:

{% tabs %}
{% tab title="CamelLIGO" %}
```ocaml
type resolved_domain = [@layout:comb] {
    // The name of the resolved domain
    name: bytes;

    // The address of the resolved domain, if any
    address: address option;

    // A map of any additional data users wish to store with the domain
    data: data_map;

    // The expiration date of the domain, if any.
    // It is already
    expiry: timestamp option;
}

type return_type = resolved_domain option;
```
{% endtab %}
{% endtabs %}

### Instructions for Off-chain Clients lacking view support

Clients that lack TZIP-16 view support can read from the contract storage directly. The `NameRegistry` contract has the following storage structure:

{% tabs %}
{% tab title="CameLIGO" %}
```ocaml
(* data map with all possible types that can be stored with an entity *)
type data_map = (string, bytes) map

type record = {
    (* The optional address the record resolves to *)
    address: address option;

    (* The owner of the record allowed to make changes *)
    owner: address;

    (* A map of any additional data clients wish to store with the domain *)
    data: data_map

    (* Validator contract reference used for validating names of new subrecords *)
    validator: nat option;

    (* Key to the expiry map containing the validity of this record *)
    expiry_key: bytes option

    (* ... more fields outside of this interoperability spec *)
}

type reverse_record = {
    (* UTF-8 encoded name *)
    name: bytes option;
    
    (* The owner of the record allowed to make changes *)
    owner: address;

    (* A map of any additional data clients wish to store with the record *)
    data: data_map;
    
    (* ... more fields outside of this interoperability spec *)
}

type storage = {
    (* Map of UTF-8 encoded names to forward records *)
    records: (bytes, record) big_map;

    (* Map of addresses to reverse records *)
    reverse_records: (address, reverse_record) big_map;

    (* Map containing expiry for every second-level domain *)
    expiry_map: (bytes, timestamp) big_map;

    (* ... more fields outside of this interoperability spec *)
}

type main_storage = {
    (* Inner storage of the contract *)
    storage: storage;

    (* ... more fields outside of this interoperability spec *)
}
```
{% endtab %}
{% endtabs %}

{% hint style="warning" %}
Clients **must not** rely on a particular storage or record layout. They should always use annotations to find the correct value.
{% endhint %}

#### Forward Resolution \(name to address\)

The resolution algorithm is as follows:

1. Normalize and validate the full domain name using the encode algorithm. See the section [Name Validation and Normalization](name-resolution.md#name-encode-algorithm) for more details.
2. Look up the name in the `records` bigmap. If the bigmap contains no such key, the given domain is not resolvable.
3. Use the `expiry_key` value of the record to look up the validity in the `expiry_map`. If a timestamp is found and is lower or equal to the current time, the given domain is not resolvable.
4. Extract the optional `address` value from the record. If the optional value is `None`, the given domain is not resolvable. Otherwise use the `address` value.

#### Reverse Resolution \(address to name\)

The resolution algorithm is as follows:

1. Look up the address in the `reverse_records` bigmap. If the bigmap contains no such key, the given address is not resolvable.
2. Extract the optional `name` value. If the optional value is `None`, the given address is not resolvable. 
3. Use the `name` value to look up the corresponding forward record.  Use it's `expiry_key` value to look up the validity of the record in the `expiry_map`. If a timestamp is found and is lower or equal to the current timestamp, the given address is not resolvable
4. Otherwise, use the `name` value.

### Instructions for Contracts

**The resolution of names by contracts is currently not supported.** Sometimes, it can be useful to validate on-chain that a name corresponds to an address. For example, a wallet might group a transaction sending money with another transaction that performs this check. If the check fails, both transactions fail and no money changes hands.

Both on-chain and off-chain clients can do this by calling the `check_address` entry-point on `NameRegistry.CheckAddress`. 

{% tabs %}
{% tab title="CameLIGO" %}
```ocaml
type check_address_param = [@layout:comb] {
    (* UTF-8 encoded name *)
    name: bytes;

    (* expected address *)
    address: address
}

(* Checks that a name corresponds to an address. *)
| Check_address of check_address_param
```
{% endtab %}

{% tab title="Michelson" %}
```text
parameter (or
    (pair %check_address (bytes %name) (address %address)
    # ... more entrypoints outside of this interoperability spec
);
```
{% endtab %}
{% endtabs %}

The transaction will either do nothing \(if the address is indeed correct\) or fail with the message `NAME_ADDRESS_MISMATCH` if the address is incorrect.

## Name Validation and Normalization

Domain names generally have to conform to the [IDNA 2008](https://en.wikipedia.org/wiki/Internationalized_domain_name) mechanism \(RFC 5891, 5892, 5893\). The Unicode Standard [UTS 46](https://www.unicode.org/reports/tr46/) is a more specific \(and stricter\) application standard which is to be used for validation and normalization of domain names to achieve IDNA conformance.

### Length Limit

The length limitations are currently:

* 1 to 100 **characters** in a **single label**,
* up to 400 **bytes** in a **full domain name**.

It is not necessary for clients to check length when performing lookup or reverse lookup, as the names are already validated on-chain. It is however strongly recommended to do so when creating new records to avoid transaction failures.

### Name Encode Algorithm

This algorithm is used both for domain creation and lookup purposes when communicating with contracts. It permits the dot character \(`.`\) so it can be used both for encoding names and individual labels.

1. The [ToUnicode](https://www.unicode.org/reports/tr46/#ToUnicode) algorithm is used to produce a normalized and validated name string. The UTS 46 version of `ToUnicode` validates the string, making it notably different from `ToUnicode` defined in IDNA, which does not have to fail on invalid labels. The algorithm is invoked with the following parameters:
   * **CheckHyphens** = true
   * **CheckBidi** = true
   * **CheckJoiners** = true
   * **UseSTD3ASCIIRules** = true
   * **Transitional\_Processing** = false
2. The name or label is encoded using UTF-8 into `bytes`.

### Libraries

Some implementations of UTS 46 include:

* [idna-uts46-hx](https://github.com/hexonet/idna-uts46) for JavaScript
* [idna](https://pypi.org/project/idna/) for Python

Libraries that implement IDNA but not UTS 46 can alternatively be used. The string has to be first validated and normalized using `ToAscii` and the result converted back with `ToUnicode`. Some implementations of IDNA \(that don't include validation and normalization as part of the `ToUnicode` algorithm and have to be called using `ToUnicode(ToAscii(name))`\):

* [System.Globalization.IdnMapping](https://docs.microsoft.com/en-us/dotnet/api/system.globalization.idnmapping) in .NET
* [idna](https://docs.rs/idna/0.2.0/idna/) for Rust
* [java.net.IDN](https://docs.oracle.com/javase/8/docs/api/java/net/IDN.html) in Java \(only implements IDNA 2003\)

