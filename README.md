# ΞID: A Standard for Tokenized Pseudo-identities

Last Revision: 15/02/2018

Ghilia Weldesselasie -- <a href='mailto:ghiliaweld@gmail.com'>ghiliaweld@gmail.com</a></br>
<i>Computer Science and Mathematics -- @ Vanier College</i></br>

## Abstract

ΞID is a tokenized *Profile* specification standard that enables the creation and use of decentralized pseudo-identities.  The standard aims to build a common informational interface by which social networks, games and other services can host user experiences on their services without hosting or owning user information. ΞID hopes to power a standard upon which Dapps can build new social networks and games centered around users without holding user information. A future in which apps are sorely focused on the UX of their apps and crafting the best experience and where users can switch to any apps they like while sill maintaining their following and profile info.

What ΞID is going for can be summed up in three words: **Experiences, not services**.

## Table Of Contents

1. [Introduction](#Introduction)
2. [Specification](#specification)
   1. [Overview](#overview)
   2. [The Profile](#the-profile-in-depth)
3. [Use Cases](#use-cases)
4. [Limitations](#limitations)
5. [Crowdfunding](#crowdfunding)
   1. [Presale Details](#presale-details)
   2. [Mainsale Details](#mainsale-details)
6. [Roadmap](#roadmap)

## Introduction

What?:

ΞID is a tokenized *Profile* specification standard that enables the creation and use of decentralized pseudo-identities.  The standard aims to build a common informational interface by which social networks, games and other services can host user experiences on their services without hosting or owning user information.

Why?:

1. Protocols like uPort have identities tied to proxy contracts which have their own addresses. Moreover, they only hold a limited amount of information with the Dapp specific information staying on the Dapp's servers. This means that your followers cannot be imported across Dapps. uPort identities can: authenticate themselves to a third party, disclose private information about themselves, receive requests for disclosure about themselves, receive and store signed third party verifications about themselves, and sign Ethereum transactions. Which is useful for real identities on-chain and off-chain but lacks certain elements needed for a decentralized pseudo-identity standard.

2. Identity protocols have yet to implement tokenized identity, opting instead to tie identities to address/contracts which might not have been so bad if they implemented this with the ENS. This make identities harder to move around across wallets and impossible to sell to other users since they are locked to an address/contracts and not owned by a wallet.

3. There are no solution currently addressing pseudo-identities and their potential use on social network/game Dapps on the blockchain.


**ΞID intends to provide a decentralized standard upon which digital and tokenized pseudo-identities can be used in various Dapps on the blockchain without information being tied to any single service.**

## Specification

### Overview

Each `Profile` on the ΞID contract is represented by an *Non-Fungible [ERC-721](https://github.com/ethereum/eips/issues/721) token*, meaning they can be transferred, traded and since they are actually a `struct` that mean that state changes can be made to them.

## Use Cases

You profile/identity could be exported to different social networks where your ΞID token can represent your profile (with name, handle, follower/following count). Instead of storing user info, social networks could simply concern themselves with the UX. User info would be hosted on the blockchain and on IPFS instead of being hosted off chain. For example, that means people with high follower counts on one platform wouldn't have to start from zero again on other platforms.

Your profile can also be brought to different games as well. If the game studio wants, you progress in other games could also determine your current level in this game (meaning you could potentially start the game at a high level). Current rank could also be based on current follower count or follower-to-following ratio on other social networks.

You could sell your social media profile via a smart contract if it has a high follower count.

## Limitations

ΞID could potentially be very expensive since you would need to pay gas every time you want to make any state changes to your profile on the blockchain. However, this problem could be mitigated by something I call deferred updating where updating the `Profile` struct is deferred until a larger update can be made in one shot. For example, instead of updating your following count every time you follow someone and paying a gas fee for that, you could decide that every 10 new followings your following count should be updated. that way you only pay 1 gas fee but you can add 10 more profiles to your following count. This can be achieved using a Stateless design (using event logs) and IPFS for decentralized off-chain storage.

Another minor limitation is something inherent to Solidity. Because Solidity does not yet support regexes(regular expressions) we cannot limit what characters can be used for names and handles. That means someone could theoretically have a name like *"#Gt~@? FwI9$^"* and a handle like *"H@<>t&2"*. While no user would be incentivized to create a name and handle like since it would be hard to find and hard for others to remember not to mention terrible branding as well, however depending on a lack of incentive is not enough. If not for the odd names, the lack of regex support also means the *real* Vitalik Buterin could have a name like *"Vitalik Buterin"* and someone else could create a similar name like *"VITALIK BUTERIN"* and it would be allowed which is a worst-case but more likely scenario. One solution that can mitigate this would be to only allow the creation of new profiles through our own Dapp and use regexes via JavaScript on the frontend, however that doesn't prevent savvy users from calling the contracts function directly. Solutions against this are still being looked into.

We are still looking into other modifications we can make to further optimize the contracts and future-proof them.
