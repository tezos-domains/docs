# Smart Contract Overview

![Overview of Smart Contracts](../.gitbook/assets/smart_contracts.png)

## Discovery

A proxy contract that exists solely for upgradeability. DApps, libraries, and other contracts use it to discover contracts they can interact with. It contains addresses of the following contracts:

* `TLDRegistrar`
* `NameRegistry`
* `ReverseRegistry`

The discovery process is further detailed in the [Interoperability](interoperability.md) chapter.

## NameRegistry

The name registry keeps track of all domains in the system, indexed by their name. There is an implied hierarchy of domains - ownership of a domain allows you to create or replace sub-domains. For every domain, the following information is stored:

* **Owner** \(`address`\) is an account authorized to make changes in the record and manage subdomains of the given domain.
* **TTL** \(optional `nat`\) is the time-to-live of the record in caches and other secondary storage mechanisms.
* **Resolution address**, the optional `address` the name resolves to.
* **Additional data**, a map with any additional data clients wish to store with the domain.

The registry supports:

* resolving a name and returning the resolved address via callback,
* managing records and creating new sub-records,
* removing records.

## LabelValidator

This smart contract validates labels according to the [IDNA](https://en.wikipedia.org/wiki/Internationalized_domain_name) rules. It provides a `validate` entry-point accepting a label. The entry-point fails the transaction if the label is not valid or if the `bytes` contain an invalid UTF-8 string. See the [Interoperability](interoperability.md) chapter for more information about normalization and validation.

## ReverseRegistry

The reverse registry stores reverse mapping of addresses to their names. Reverse records are optional, but if a reverse record exists for a given address, the name has to resolve back to that address \(both contracts cooperate to guarantee consistency\). For every address, the following information is stored:

* **Owner** \(`address`\) is an account authorized to make changes in the record.
* **TTL** \(`nat`\) is the maximum time-to-live of the record in caches and other secondary storage mechanisms.
* **Name** is the name this reverse record resolves to.

The registry supports:

* resolving an address and returning the resolved name via callback,
* claiming records for the sender,
* managing records,
* removing records.

## TLDRegistrar

This smart contract is responsible for managing the `.tez` top-level domain. It keeps track of registered second-level domains, their owners, and expiration times. More details on the smart contact are available in the [next chapter](top-level-domain-registrar.md).

