# Interoperability

## Proxy Contracts

To ensure future upgradeability while allowing clients to rely on fixed contract addresses, we offer a set of proxy contracts. Off-chain clients use their storage to fetch current contract addresses. Contracts can interact with them directly - the transactions are automatically routed to the correct destination.

The addresses of the proxy contracts are following:

| Contract | Mainnet | Carthagenet |
| :--- | :--- | :--- |
| **TLDRegistrarProxy** | TBA | `KT1FJheAi6GyX8MDpqahN5VCpFp12Kpgo7Tm` |
| **NameRegistryProxy** | TBA | `KT1JLkXGT6q4YkyHQNvGKtJA41KVHysZ1ctU` |

### Instructions for Contracts

Contracts can use all entrypoints that are described in this spec with the corresponding proxy contracts directly. All transactions will be automatically routed to the correct contract.

### Instructions for Off-chain Clients

Off-chain clients retrieve the individual addresses from the proxy contract's storage. Using automatic routing on the proxy contracts is discouraged for off-chain clients, as it unnecessarily increases the overall transaction gas consumption.

_CameLIGO**:**_

```ocaml
type proxy_storage = {
    contract: address;

    (* ... more fields outside of this interoperability spec *)
}
```

_Michelson**:**_

```text
storage (pair
    (address %contract)
    (
        # ... more fields outside of this interoperability spec
    )
);
```

_Note: Off-chain clients **must not** rely on a particular storage layout. They should always use annotations to find the correct value._

## NameRegistry

The `NameRegistry` contract provides forward and reverse resolution.

### Instructions for Off-chain Clients

After retrieving `NameRegistry`address from `NameRegistryProxy`, off-chain clients can perform resolution using the contract's storage. The `NameRegistry` contract uses the following storage structure:

_CameLIGO**:**_

```ocaml
type record = {
    (* The optional address the record resolves to *)
    address: address option;

    (* The owner of the record allowed to make changes *)
    owner: address;

    (* A map of any additional data clients wish to store with the domain *)
    data: (string, data_value) map;

    (* Validator contract reference used for validating names of new subrecords *)
    validator: nat option;

    (* The computed level of this record *)
    level: nat;

    (* Key to the validity map containing the validity of this record *)
    validity_key: bytes option
}

type reverse_record = {
    name: bytes option;
    owner: address;
}

type storage = {
    (* Map of UTF-8 encoded names to forward records *)
    records: (bytes, record) big_map;

    (* Map of addresses to reverse records *)
    reverse_records: (address, reverse_record) big_map;

    (* Map containing validity for every second-level domain *)
    validity_map: (bytes, timestamp) big_map;

    (* ... more fields outside of this interoperability spec *)
}
```

_Michelson**:**_

```text
TBD
```

_Note: Clients **must not** rely on a particular storage or record layout. They should always use annotations to find the correct value._

#### Forward Resolution \(name to address\)

The resolution algorithm is as follows:

1. Normalize and validate the full domain name using the encode algorithm. See the section [Name Validation and Normalization](interoperability.md#name-encode-algorithm) for more details.
2. Look up the name in the `records` bigmap. If the bigmap contains no such key, the given domain is not resolvable.
3. Use the `validity_key` value of the record to look up the validity in the `validity_map`. If a timestamp is found and is lower than the current time, the given domain is not resolvable.
4. Extract the optional `address` value from the record. If the optional value is `None`, the given domain is not resolvable. Otherwise use the `address` value.

#### Reverse Resolution \(address to name\)

The resolution algorithm is as follows:

1. Look up the address in the `reverse_records` bigmap. If the bigmap contains no such key, the given address is not resolvable.
2. Extract the optional `name` value. If the optional value is `None`, the given address is not resolvable. 
3. Use the `name` value to look up the corresponding forward record.  Use it's `validity_key` value to look up the validity of the record in the `validity_map`. If a timestamp is found and is lower than the current timestamp, the given address is not resolvable
4. Otherwise, use the `name` value.

### Instructions for Contracts

#### Forward Resolution \(name to address\)

Contracts can use the `resolve` entry-point on `NameRegistryProxy` and get the resolved address with a callback.

_CameLIGO**:**_

```ocaml
type resolve_param = {
    (* The UTF-8 encoded name to resolve *)
    name: bytes;

    (*
    The callback to make. If not resolved,
    None is passed (a call back is always made).
    *)
    callback: address option contract
}

(* Resolves a name to it's address via a callback. *)
| Resolve of resolve_param
```

_Michelson**:**_

```text
parameter (or
    (pair %resolve (bytes %name) (contract %callback (option address)))
    # ... more endpoints outside of this interoperability spec
);
```

The name has to be a full domain name that has been normalized and validated. The callback contract will be invoked with the resolved address or with `None` if the name could not be resolved.

#### Reverse Resolution \(address to name\)

Contracts can use the `reverse_resolve` entry-point and get the resolved address via a callback.

_CameLIGO**:**_

```ocaml
type reverse_resolve_param = {
    (* The address to resolve *)
    addr: address;

    (*
    The callback to make. If not resolved,
    None is passed (a call back is always made).
    *)
    callback: bytes option contract
}

(* Resolves an address to it's name via a callback. *)
| Reverse_resolve of reverse_resolve_param
```

_Michelson**:**_

```python
TBD
```

The callback contract will be invoked with the resolved name or with `None` if the address could not be resolved.

## Name Validation and Normalization

Domain names generally have to conform to the [IDNA 2008](https://en.wikipedia.org/wiki/Internationalized_domain_name) mechanism \(RFC 5891, 5892, 5893\). The Unicode Standard [UTS 46](https://www.unicode.org/reports/tr46/) is a more specific \(and stricter\) application standard which is to be used for validation and normalization of domain names to achieve IDNA conformance.

### Length Limit

The length limitations are currently:

* 1 to 100 **characters** in a **single label**,
* up to 1000 **bytes** in a **full domain name**.

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

