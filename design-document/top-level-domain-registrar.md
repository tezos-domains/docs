# Top-Level Domain Registrar

The registrar smart contract is responsible for managing the `.tez` top-level domain. It sells domains to buyers and allows renewals of domains and transfers of ownership. It is also an implementor of the [FA2 - Multi-Asset Interface](https://gitlab.com/tzip/tzip/-/blob/master/proposals/tzip-12/tzip-12.md) \(TZIP-12\) to allow domains to be traded in secondary markets.

## Vickrey Auction

The registrar implements the [Vickrey auction](https://en.wikipedia.org/wiki/Vickrey_auction) model. In this auction model, all bids are sealed and the highest bidder wins, but pays only the second-highest bid. There is a minimum bid that configurable by an administrator endpoint \(see below\).

To conduct Vickrey auction so that bids are truly sealed, several phases take place: 1. In the **bidding phase**, bidders submit hashes of their bids and transfer funds that are held until the auction ends. Bidders can also choose to submit fake bids to make their real bids more difficult to guess. 2. In the **reveal phase**, bidders reveal what bids they made and which of them were fake. Every bid that pairs with a corresponding hash is accepted. Funds held for bids that are fake or lower than the current highest bid are immediately available for refund. 3. In the **settlement phase**, a participant \(typically the highest bidder\) invokes the settlement process, which transfers the domain to the highest bidder. The settlement phase does not have a limit, but the time spent in the settlement phase counts towards the domain's registration period \(users can't block the domain for longer by not invoking the settlement process\).

Special cases are handled as follows:

* If there are no bids in the bidding phase, the reveal phase does not take place and the auction ends.
* If there are no reveals in the reveal phase, the settlement phase does not take place and the auction ends.
* Once all bids are revealed, the settlement phase starts immediately.

After the auction ends, the domain is registered for the minimum registration period. Renewals are priced proportionally to the price the domain was auctioned. The formula is `priceOfRenewal = chosenRenewalPeriod * auctionPrice / minimumRegistrationPeriodAtAuction`.

## First-In First-Served Registration

The FIFS registration model after it's shown that no implied demand exists for a given domain. No implied demand exists iff:

* `currentTimestamp > (lastExpirationDate(domain) || launchDate) + biddingPhasePeriod`
* AND there is no auction phase taking place

In other words, an expired domain becomes available for FIFS after the bidding phase is over with no bids or the reveal phase is over with no reveals. In the period after the launch of TNS, all domains are treated as recently expired.

In the FIFS model, all domains are sold for a fee equal to the minimum bid.

## Configuration

There are several parameters stored which configure this contract:

* `minimumBid` is the minimum amount that participants have to bid \(in mutez\)
* `biddingPhasePeriod` is the bidding phase period \(in seconds\)
* `revealPhasePeriod` is the reveal phase period \(in seconds\)
* `launchDate` is the date when the service was officially launched \(to calculate implied demand\)
* `minimumRegistrationPeriod` is the minimum period anyone can auction a domain for \(in days\)

