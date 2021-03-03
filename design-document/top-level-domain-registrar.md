# Top-Level Domain Registrar

The registrar smart contract is responsible for managing the top-level domains. It sells second-level domains to buyers and allows renewals.

## Open Auction

At service launch, every domain is offered to users through auction first. The registrar implements an [open ascending price auction](https://en.wikipedia.org/wiki/English_auction) model. It is an auction model most people are likely familiar with: all bids are openly visible and every new bid is required to be higher than the last bid. If there are no bids in a given period since the last bid, the auction ends with the highest bidder winning the auction.

### Bids

Any account can submit a bid during an active auction and transfer associated funds along in the same transaction. If a bid is higher or equal to the current highest bid multiplied by [min\_bid\_increase\_ratio](top-level-domain-registrar.md#configuration), it becomes the new highest bid and the previous highest bid becomes refundable or available for future bids \(on the same or a different domain\). Otherwise, the transaction fails. The auction ends after [bid\_additional\_period](top-level-domain-registrar.md#configuration) seconds since the last bid, but not earlier than [min\_auction\_period](top-level-domain-registrar.md#configuration) since the auction's start.

A special case is the first bid, which has to be greater or equal to [standard\_price\_per\_day](top-level-domain-registrar.md#configuration). When a previously unregistered domain name is first bid on, the name is pre-registered in the name registry to ensure its validity.

### Settlement

After the auction ends, there is a settlement period of the same length as [min\_duration](top-level-domain-registrar.md#configuration). The highest bidder invokes the settlement process transferring the domain to a chosen address, which becomes the new owner. The domain is registered for the [min\_duration](top-level-domain-registrar.md#configuration) \(the time spent in the settlement period counts towards that period\). If the highest bidder doesn't invoke settlement, the domain is treated as expired after the settlement phase ends.

## First-In First-Served Registration

An expired domain becomes available for FIFS only after its auction ended with no bids. In the FIFS model, all domains are sold for a flat fee equal to the [standard\_price\_per\_day](top-level-domain-registrar.md#configuration). In the period after the launch of the smart contract, all domains are treated as recently expired.

For FIFS registration, the contract implements a commit & reveal scheme to avoid [front-running](https://medium.com/consensys-diligence/transparent-dishonesty-taxonomy-of-front-running-attacks-on-blockchain-317d8ff78068) of transactions.

## Renewals

Owners of domains can renew their domains at any time up until the domain expires. The chosen renewal period has to be greater or equal to [min\_duration](top-level-domain-registrar.md#configuration). Renewals are priced using [standard\_price\_per\_day](top-level-domain-registrar.md#configuration).

## Top-level Domain List

The susceptibility to spoofing attacks using look-alike characters is reduced by limiting TLDs to predefined character scripts. Neither whole-script and mixed-script [confusable](https://www.unicode.org/reports/tr39/#Confusable_Detection) names can be created under one TLD.

The top-level domains available are:

| Name | Description |
| :--- | :--- |
| [.tez](../interoperability/.tez-tld.md) | The standard TLD for names in Latin |

TLDs for more character scripts will be likely made available in the future.

## Configuration

There are several numeric parameters stored to configure this contract in the `config` bigmap. The keys of the bigmap following:

* `0 = max_commitment_age` is the maximum time for a buy commitment to be valid \(in seconds\)
* `1 = min_commitment_age` is the minimum time for a buy commitment to be valid \(in seconds\)
* `2 = standard_price_per_day` is the standard FIFS price for a day and the minimum amount that participants have to bid initially in auction \(in picotezos = 1e-12 tez\)
* `3 = min_duration` is the minimum period anyone can register or renew a domain for \(in seconds\)
* `4 = min_bid_increase_ratio` is the minimum ratio of a new bid to the current highest bid \(in percent\)
* `5 = min_auction_period` is the minimum auction period \(in seconds\)
* `6 = bid_additional_period` is the period after which the auction ends measured since the last successful bid \(in seconds\)
* `1000 = launch_date` is the start of the initial auction period for all domains, i.e., the date the service was officially launched \(in second since epoch\). The value of `0` means no launch has been configured
  * `1000 + label_length` is the override of the launch date for all labels that are `label_length` characters long

