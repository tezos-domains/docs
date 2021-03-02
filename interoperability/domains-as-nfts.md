# Domains as NFTs

The `NameRegistry` is an FA2-compliant smart contract \([TZIP-12](https://gitlab.com/tzip/tzip/-/blob/master/proposals/tzip-12/tzip-12.md)\). That means that all domains can be used as non-fungible tokens, with a few caveats:

* Operator transfer policy is set to `Owner_transfer`, which means that operators cannot be set. Only owners can currently transfer domains. This functionality could be added in the future.
* After a domain expires, the owner's balance of the token is always `0`. That means that a domain will "disappear" as a token once it expires \(although the `token_id` continues to be valid\).
* We don't implement the optional `all_tokens` view, because the number of tokens would be too large to be returned in one call.

See the [TZIP-12](https://gitlab.com/tzip/tzip/-/blob/master/proposals/tzip-12/tzip-12.md) specification for more information about the FA2 standard.

