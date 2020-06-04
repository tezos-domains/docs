---
description: Creating human-readable names for cryptocurrency addresses on Tezos network.
---

# Tezos Domains

You can read our post series on Designing a Name Service on [Medium ](https://medium.com/tezos-name-service)or discuss with us on Tezos Agora

* [Part 1: Introduction](https://forum.tezosagora.org/t/designing-a-name-service-part-1-introduction/1874)
* [Part 2: Namespace and Structure](https://forum.tezosagora.org/t/designing-a-name-service-part-2-namespace-and-structure/1901)
* [Part 3: Name Distribution and Pricing](https://forum.tezosagora.org/t/designing-a-name-service-part-3-name-distribution-and-pricing/1914)
* [Part 4: Validation, Normalization, Encoding](https://forum.tezosagora.org/t/designing-a-name-service-part-4-validation-normalization-encoding/1915)
* [Part 5: Retaining Trademarks](https://forum.tezosagora.org/t/designing-a-name-service-part-5-retaining-trademarks/1931)

## Motivation

Using addresses can become an obstacle for the practical use of Tezos as a currency. It’s clear that a string like `tz1VSUr8wwNhLAzempoch5d6hLRiTh8Cjcjb` is usable when transferred by a computer program or using the operating system’s clipboard, but it is unwieldy in most other types of communication. It cannot be memorized or even realistically communicated through human speech or visual media.

Building a system where participants are in charge of creating their aliases presents a straightforward solution. If it is the best solution is a bit less clear, but more on that later. People are already used to creating names for their e-mail boxes, Instagram accounts, or their web pages. The topic of this discussion is providing them with this familiar approach inside the Tezos ecosystem.

## What Are Tezos Domains

The main function is to translate a meaningful and user-friendly alias to a Tezos address and potentially vice versa. This translation should be globally consistent so that all participants on the blockchain see the same address for a given alias.

A parallel that's often drawn is to the [DNS](https://en.wikipedia.org/wiki/Domain_Name_System), a familiar and universally adopted system:

> The Domain Name System \(DNS\) is a hierarchical and decentralized naming system for computers, services, or other resources connected to the Internet or a private network. It associates various information with domain names assigned to each of the participating entities.

An example of one such alias is `alice.tez`. Alice bought it from the central registrar managing `tez` and then assigned it the address of her personal wallet. When she sends money to Bob, he will see `alice.tez` in his wallet's received transactions, because Alice has also set up a reverse record mapping her address back to `alice.tez`.

