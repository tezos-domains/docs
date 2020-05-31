# Interoperability

## Discovery

Clients and contracts use the discovery process to obtain the current smart contract addresses they can interact with. The address of the `Discovery` contract for Mainnet and testnets will be published here when available.

Addresses are represented as a `map` containing at least the following key-values:

| Key | Value |
| :--- | :--- |
| `TLDRegistrar` | address of the `tez` TLD registrar |
| `NameRegistry` | address of the name registry |
| `ReverseRegistry` | address of the reverse registry |

### Instructions for Clients

Off-chain clients retrieve the individual addresses directly from the contract's storage. The `Discovery` contract contains the following storage structure:

**SmartPy:**

```python
sp.TRecord(
    contracts = sp.TMap(tkey = sp.TString, tvalue = sp.TAddress),

    # ... more fields outside of this interoperability spec
)
```

**Michelson:**

```text
storage (pair
    (map %contracts string address)
    (
        # ... more fields outside of this interoperability spec
    )
);
```

_Note: Clients **must not** rely on the ordering of the top-level nested pairs. They should always use annotations to find the correct value._

### Instructions for Contracts

Contracts can use the `get_addresses` entry-point to get all known smart contract addresses via a callback.

**SmartPy:**

```python
# Gets all known addresses
@sp.entry_point
def get_addresses(self, callback):
    sp.set_type(callback, sp.TContract(sp.TMap(tkey = sp.TString, tvalue = sp.TAddress)))
    # ...
```

**Michelson:**

```text
parameter (or 
    (contract %get_addresses (map string address))
    # ... more endpoints outside of this interoperability spec
);
```

## Name Resolution

Forward name resolution is done using the `NameRegistry` contract. After retrieving its address via discovery, one of the following processes should be used.

### Instructions for Clients

Off-chain clients resolve names using the contract's storage directly. The `NameRegistry` contract uses the following structure:

**SmartPy:**

```python
sp.TRecord(
    records = sp.big_map(
        tkey = sp.TBytes,
        tvalue = sp.TRecord(
            address = sp.TOption(sp.TAddress),
            owner = sp.TAddress,
            ttl = sp.TOption(sp.TNat),
            data = sp.TMap(sp.TString, sp.TVariant(string = sp.TString, nat = sp.TNat, int = sp.TInt, bytes = sp.TBytes))
        )
    ),

    # ... more fields that are outside of this interoperability spec
)
```

**Michelson:**

```text
storage (pair
    (big_map %records bytes (pair
        (address %owner)
        (pair
            (option %ttl nat)
            (pair
                (map %data string (or
                    (or
                        (bytes %bytes)
                        (int %int)
                    )
                    (or
                        (nat %nat)
                        (string %string)
                    )
                ))
                (option %address address)
            )
        )
    ))
    # ... more fields outside of this interoperability spec
);
```

_Note: Clients **must not** rely on the ordering of the top-level nested pairs. They should always use annotations to find the correct value._

The resolution algorithm is as follows:

1. Normalize and validate the full domain name using the encode algorithm. See the section [Name Validation and Normalization](interoperability.md) for more details.
2. Look up the name in the `records` bigmap. If the bigmap contains no such key, the given domain is not resolvable.
3. Extract the optional `address` value. If the optional value is `None`, the given domain is not resolvable. Otherwise use the `address` value.

### Instructions for Contracts

Contracts can use the `resolve` entry-point and get the resolved address via a callback.

**SmartPy:**

```python
# Resolves a given name
@sp.entry_point
def resolve(self, params):
    sp.set_type(params, sp.TRecord(name = sp.TBytes, callback = sp.TContract(sp.TOption(sp.TAddress))))
    sp.set_record_layout(params, ("name", "callback"))
    # ...
```

**Michelson:**

```text
parameter (or
    (pair %resolve (bytes %name) (contract %callback (option address)))
    # ... more endpoints outside of this interoperability spec
);
```

The name has to be a full domain name that has been normalized and validated. The callback contract will be invoked with the resolved address or with `None` if the name could not be resolved.

## Reverse Resolution

Reverse resolution is done using the `ReverseRegistry` contract. After retrieving its address via discovery, one of the following processes should be used depending on the type of the caller.

### Instructions for Clients

Off-chain clients resolve names using the contract's storage directly. The `ReverseRegistry` contract uses the following structure:

**SmartPy:**

```python
sp.TRecord(
    records = sp.big_map(
        tkey = sp.TBytes,
        tvalue = sp.TRecord(
            name = sp.TOption(sp.TBytes),
            owner = sp.TAddress,
            ttl = sp.TOption(sp.TNat)
        )
    ),

    # ... more fields that are outside of this interoperability spec
)
```

_Note: Clients **must not** rely on the ordering of the top-level nested pairs. They should always use annotations to find the correct value._

The resolution algorithm is as follows:

1. Look up the address in the `records` bigmap. If the bigmap contains no such key, the given address is not resolvable.
2. Extract the optional `name` value. If the optional value is `None`, the given address is not resolvable. Otherwise, use the `name` value.

### Instructions for Contracts

Contracts can use the `resolve` entry-point and get the resolved address via a callback.

**SmartPy:**

```python
# Resolves a given name
@sp.entry_point
def resolve(self, params):
    sp.set_type(params, sp.TRecord(address = sp.TAddress, callback = sp.TContract(sp.TOption(sp.TBytes))))
    sp.set_record_layout(params, ("name", "callback"))
    # ...
```

The callback contract will be invoked with the resolved name or with `None` if the address could not be resolved.

## Name Validation and Normalization

Domain names generally have to conform to the [IDNA 2008](https://en.wikipedia.org/wiki/Internationalized_domain_name) mechanism \(RFC 5891, 5892, 5893\). The Unicode Standard [UTS 46](https://www.unicode.org/reports/tr46/) is a more specific \(and stricter\) application standard which is to be used for validation and normalization of domain names to achieve IDNA conformance.

### Length Limit

The length limitations are currently:

* 1 to 100 characters on a **single label**,
* 1 to 250 characters on a **full domain name**.

It is not necessary for clients to validate length for lookup or reverse lookup purposes, as it's validated by the name registry. It is however strongly recommended to do so when creating new records to avoid transaction failures.

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

