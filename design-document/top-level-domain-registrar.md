# Top-Level Domain Registrar

The registrar smart contract is responsible for managing the top-level domains. It sells second-level domains to buyers and allows renewals and transfers of ownership. It is also an implementor of the [FA2 - Multi-Asset Interface](https://gitlab.com/tzip/tzip/-/blob/master/proposals/tzip-12/tzip-12.md) \(TZIP-12\) to allow domains to be traded on secondary markets.

## Open Auction

Every domain is offered to users through auction first. The registrar implements an [open ascending price auction](https://en.wikipedia.org/wiki/English_auction) model. It is an auction model most people are likely familiar with: all bids are openly visible and every new bid is required to be higher than the last bid. If there are no bids in a given period since the last bid, the auction ends with the highest bidder winning the auction.

### Bids

Any account can submit a bid during an active auction and transfer associated funds along in the same transaction. If a bid is higher or equal to the current highest bid multiplied by [minimum\_increase\_ratio](top-level-domain-registrar.md#configuration), it becomes the new highest bid and the previous highest bid becomes refundable. Otherwise, the transaction fails. The auction ends after [bid\_period](top-level-domain-registrar.md#configuration) seconds since the last bid, but not earlier than [minimum\_auction\_period](top-level-domain-registrar.md#configuration) since the auction's start.

A special case is the first bid, which has to be greater or equal to [starting\_price](top-level-domain-registrar.md#configuration). When a previously unregistered domain name is first bid on, the name is pre-registered in the name registry to ensure it's validity.

### Settlement

After the auction ends, there is a settlement period of the same length as [minimum\_registration\_period](top-level-domain-registrar.md#configuration). The highest bidder invokes the settlement process transferring the domain to a chosen address, which becomes the new owner. The domain is registered for the [minimum\_registration\_period](top-level-domain-registrar.md#configuration) \(the time spent in the settlement period counts towards that period\). If the highest bidder doesn't invoke settlement, the domain is treated as expired after the settlement phase ends.

## First-In First-Served Registration

An expired domain becomes available for FIFS only after it's auction ended with no bids. In the FIFS model, all domains are sold for a flat fee equal to the [starting\_price](top-level-domain-registrar.md#configuration).

In the period after the launch of the smart contract, all domains are treated as recently expired.

## Renewals

Owners of domains can renew their domains at any time up until the domain expires. The chosen renewal period has to be greater or equal to [minimum\_registration\_period](top-level-domain-registrar.md#configuration). Renewals are priced proportionally to the price the domain was initially sold for - the formula is:

```text
price_of_renewal = chosen_renewal_period * initial_price / initial_registration_period
```

## Top-level Domain List

The susceptibility to spoofing attacks using look-alike characters is reduced by limiting TLDs to predefined character scripts. Neither whole-script and mixed-script [confusable](https://www.unicode.org/reports/tr39/#Confusable_Detection) names can be created in one TLD.

The top-level domains available are:

| Name | Description |
| :--- | :--- |
| `.tez` | The standard TLD for names in Latin |
| `.тез` | TLD for names in languages based on Cyrillic |

TLDs for more character scripts will be likely made available in the future.

## Configuration

There are several parameters stored to configure this contract:

* `starting_price` is the minimum amount that participants have to bid initially \(in mutez\)
* `minimum_increase_ratio` is the minimum ratio of a new bid to the current highest bid 
* `minimum_auction_period` is the minimum auction period \(in seconds\)
* `bid_period` is the period after which the auction ends measured since the last successful bid \(in seconds\)
* `initial_auction_start` is the start of the initial auction period for all domains, i.e., the date the service was officially launched
* `minimum_registration_period` is the minimum period anyone can register or renew a domain for \(in seconds\)

