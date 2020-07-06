# Smart Contract Overview

![Overview of Smart Contracts](../.gitbook/assets/smart_contracts.png)

## Discovery

A proxy contract that exists solely for upgradeability. DApps, libraries, and other contracts use it to discover contracts they can interact with. It contains addresses of the following contracts:

* `TLDRegistrar`
* `NameRegistry`
* `ReverseRegistry`

The discovery process is further detailed in the [Interoperability](interoperability.md) chapter.

## NameRegistry

### Forward Records

Forward records \(or just records\) represent all domains in the system, indexed by their name. There is an implied hierarchy of domains - ownership of a domain allows you to create or replace sub-domains. For every domain, the following information is stored:

* **Owner** \(`address`\) is an account authorized to make changes in the record and manage subdomains of the given domain.
* **Resolution address**, the optional `address` the name resolves to.
* **Additional data**, a map with any additional data clients wish to store with the domain.
* **Validity reference**, a reference inside the validity map, which contains timestamps for every second-level domain. This timestamp represents a point in time when the domain ceases to be valid.
* **Label validator reference** - an index of a contract used for validating labels of new subrecords.

Supported **operations** on records are:

* resolving a name and returning the resolved address via callback,
* updating records,
* creating new sub-records.

### Reverse Records

Reverse records represent mapping of addresses to their names. Reverse records are optional, but if a reverse record exists for a given address, the name has to resolve back to that address \(both contracts cooperate to guarantee consistency\). For every address, the following information is stored:

* **Owner** \(`address`\) is an account authorized to make changes in the record.
* **Name** is the name this reverse record resolves to.

Supported **operations** on reverse records are:

* resolving an address and returning the resolved name via callback,
* claiming records for the sender \(and removing previous owner\),
* updating records.

## Label Validators

These smart contracts validate labels according to the [IDNA](https://en.wikipedia.org/wiki/Internationalized_domain_name) rules and the specific rules for the respective top-level domain. They provides a `validate` entry-point accepting a label. The entry-point fails the transaction if the label is not valid or if the `bytes` contain an invalid UTF-8 string. See the [Interoperability](interoperability.md) chapter for more information about normalization and validation.

## TLDRegistrar

This smart contract is responsible for managing the top-level domains. It keeps track of registered second-level domains, their owners, and expiration times. More details on the smart contact are available in the [next chapter](top-level-domain-registrar.md).

