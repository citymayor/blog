---
title: "Why The Standardization Of ERC-721 Is A Bad Idea"
date: 2018-04-25T16:20:00Z
draft: false
---

If you know about [Cryptokitties](https://www.cryptokitties.co/), you might wonder what it has done for Ethereum. Because yes, as much as it is "just a game", Cryptokitties was the introduction of a new concept called **non-fungible tokens**: tokens that are different from one another and that can be traded for different prices.

![cryptokitties](/img/erc721/cryptokitties.png)

After the [success](https://hackernoon.com/how-we-made-100k-trading-cryptokitties-2d69aebe715b) of Cryptokitties, a number of non-fungible tokens contracts followed with the same recipe: [cryptocelebrities](https://cryptoclarified.com/2018/02/08/22-million-spent-on-celebrity-trading-card-beta-crypto-celebrities-is-the-new-crypto-craze/), [ethermon](http://ethermon.net/), ... The idea became so popular that it started to make sense to standardize the concept. The [ERC-721](http://erc721.org/) draft was born. 

![nft examples](/img/erc721/nft.png)

"ERC" stands for **Ethereum Request for Comment** and if you know about standards and the IETF in particular, you might have recognized the "Request for Comment" part. The idea behind some of these ERC standards is to facilitate support of Ethereum concepts in order to democratize their ideas. For example, [ERC-20](https://en.wikipedia.org/wiki/ERC20) is the standard for **tokens**. All ICOs use it. If you want your token to be supported by a trading exchange you pretty much have no other choice but to support that standard (by implementing the functions it specifies). You'll see that [Etherscan](http://etherscan.io), [Metamask](https://metamask.io), [most](https://wallet.gnosis.pm/) [multi-signature](https://github.com/paritytech/parity) [wallets](https://github.com/BitGo/eth-multisig-v2), etc... actually recognize all tokens that have been implemented via the ERC-20 standard and display information on them. For example, see the [Etherscan page for the CityMayor contract](https://etherscan.io/address/0x4bdde1e9fbaef2579dd63e2abbf0be445ab93f10), because yes the CityCoin of CityMayor is an ERC-20 token :)

![etherscan citycoin](/img/erc721/citycoin.png)

Standardizing non-fungible tokens would allow for the same kind of interaction, for example both [OpenSea](https://opensea.io/) and [RareBits](https://rarebits.io/) are platforms to trade such assets from contracts like CityMayor or Cryptokitties.

Back to the [ERC-721](https://github.com/ethereum/EIPs/issues/721) draft (which has actually spawned more than just one ERC draft: [ERC-821](https://github.com/ethereum/EIPs/issues/821), [ERC-875](https://github.com/ethereum/EIPs/issues/875)). As you can see if you glance over the ERC, things look really similar to the ERC-20 with functions like `transfer()` and `getBalance()`. 

**One might wonder why the standard has chosen the same names as the ERC-20 standard.**

And indeed, they ran into problems, CityMayor being the first.

To understand the issue, you need to understand the [CityMayor smart contract](https://etherscan.io/address/0x4bdde1e9fbaef2579dd63e2abbf0be445ab93f10#code) which is an ERC-20 smart contract (CityCoins) that also includes a non-fungible tokens (cities). The concept is explained in details on our [whitepaper](https://citymayor.co/whitepaper) but to make it short it's **Cryptokitties meets an ICO**: coins can only be obtained by buying assets in the CityMayor smart contract. More coins can also be obtained by holding assets.

This idea of mixing tokens with non-fungible tokens was something that we wanted to explore early on due to the lack of interaction during ICOs. **Most (if not all) ICOs do not really need a smart contract to operate, and surprisingly some actually do not even have one**. The alternative was to force interaction with a smart contract to obtain coins, which is what we did with CityMayor where the CityCoin token is minted during the initial purchase of each assets (which are limited by the contract).

Because of this deep integration, **CityMayor needs to use a single smart contract to handle both the ERC-20 tokens and the non-fungible tokens**. Following widely adopted standards like the ERC-20 token standard is a no-brainer, but following the upcoming ERC-721 standard was a decision we wish we could have taken as well. Because of the function **naming colliding** between the two standards, this was not possible.

We're hopping to raise awareness about these issues, as more and more contracts will look into pushing the idea of non-fungible tokens to its limit. **Limiting the interaction of this ERC with other ERC will limit progress and innovation.** 

Note that this is issue is actually a **security vulnerability** in cryptography called "domain separation".

Because of this confusion between the standards, we might see **theft and loss of ethers** happen in the future in cases where clients would wrongly detect ERC-721 smart contracts as ERC-20 contracts.
Take a look at Etherscan and how it handles ERC-721 contracts like [Cryptokitties](https://etherscan.io/address/0xb1690C08E213a35Ed9bAb7B318DE14420FB57d8C), you will see that ERC-721 is already causing trouble.

![cryptokitties ERC-20](/img/erc721/cryptokitties_token.png)

[CityMayor](https://www.citymayor.co) is a new concept born from mixing Crypokitties with an ICO. Tokens can be obtained by buying a city and observing trades of cities in the region. More information on our [whitepaper](https://www.citymayor.co/whitepaper).

<i class="fas fa-university"></i> **The United Nations**