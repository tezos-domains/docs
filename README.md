# Tezos Name Service

_This is an early alpha version of the draft._

TNS is the Tezos Name Service, a distributed, open, and extensible naming system based on the Tezos blockchain.

TNS’s job is to map human-readable names like ‘alice.xtz’ to machine-readable identifiers such as Tezos addresses, content hashes, and metadata. TNS will also supports ‘reverse resolution’, making it possible to associate metadata such as canonical names or interface descriptions with Tezos addresses.

TNS has similar goals to DNS, the Internet’s Domain Name Service, but has significantly different architecture, due to the capabilities and constraints provided by the Tezos blockchain. Like DNS, TNS operates on a system of dot-separated hierarchical names called domains, with the owner of a domain having full control over subdomains.

Top-level domains, like ‘.xtz’ and ‘.test’ are owned by smart contracts called registrars, which specify rules governing the allocation of their subdomains. Anyone may, by following the rules imposed by these registrar contracts, obtain ownership of a domain for their own use.

Because of the hierarchal nature of TNS, anyone who owns a domain at any level may configure subdomains - for themselves or others - as desired. For instance, if Alice owns 'alice.xtz', she can create 'pay.alice.xtz' and configure it as she wishes.



