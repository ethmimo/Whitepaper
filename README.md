# D-OZ: A Standard for Tokenized Pseudo-identities

Last Revision: 08/02/2018

Ghilia Weldesselasie -- <a href='mailto:ghiliaweld@gmail.com'>ghiliaweld@gmail.com</a></br>
<i>Computer Science and Mathematics -- @ Vanier College</i></br>

## Abstract

D-OZ is a tokenized *Profile* specification standard that enables the creation and use of decentralized pseudo-identities.  The standard aims to build a common informational interface by which social networks, games and other services can host user experiences on their services without hosting or owning user information. D-OZ hopes to power a standard upon which Dapps can build new social networks and games centered around users without holding user information. A future in which apps are sorely focused on the UX of their apps and crafting the best experience and where users can switch to any apps they like while sill maintaining their following and profile info.

What D-OZ is going for can be summed up in three words: **Experiences, not services**.

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

D-OZ is a tokenized *Profile* specification standard that enables the creation and use of decentralized pseudo-identities.  The standard aims to build a common informational interface by which social networks, games and other services can host user experiences on their services without hosting or owning user information.

Why?:

1. Protocols like uPort have identities tied to proxy contracts which have their own addresses. Moreover, they only hold a limited amount of information with the Dapp specific information staying on the Dapp's servers. This means that your followers cannot be imported across Dapps. uPort identities can: authenticate themselves to a third party, disclose private information about themselves, receive requests for disclosure about themselves, receive and store signed third party verifications about themselves, and sign Ethereum transactions. Which is useful for real identities on-chain and off-chain but lacks certain elements needed for a decentralized pseudo-identity standard.

2. Identity protocols have yet to implement tokenized identity, opting instead to tie identities to address/contracts which might not have been so bad if they implemented this with the ENS. This make identities harder to move around across wallets and impossible to sell to other users since they are locked to an address/contracts and not owned by a wallet.

3. There are no solution currently addressing pseudo-identities and their potential use on social network/game Dapps on the blockchain.


**D-OZ intends to provide a decentralized standard upon which digital and tokenized pseudo-identities can be used in various Dapps on the blockchain without information being tied to any single service.**

## Specification

### Overview

Each `Profile` on the D-OZ contract is represented by an *Non-Fungible [ERC-721](https://github.com/ethereum/eips/issues/721) token*, meaning they can be transferred, traded and since they are actually a `struct` that mean that state changes can be made to them. In this section we will go over the specification of each `Profile`.

### The `Profile` (In-Depth)

Here's what our `Profile` `struct` looks like:
```
struct Profile {
        string name; // Must be unique
        string handle; // Must be unique
        string bio;
        string publicKey; // stores a public key for our profile, still looking into it
        mapping (string => string) public metadata;
        uint32 followerCount;
        uint32 followingCount;
        uint[] public followers; // Tracks who is following this profile
        uint[] public following; // Tracks who this profile is following
        address createdBy;
        uint64 dateCreated;
}
```
#### The `name` string
The `name` property is pretty self-explanatory, it's the name of your `Profile`.
I'm currently looking into limiting which characters can be used for the `name` property (alphabetical characters only), but I might just allow people to use any characters they want (like @, #, $, ^ or even ~).

#### The `handle` string
The `handle` property is pretty self-explanatory, it's the handle of your `Profile` (think Twitter handle, your @ username).
I'm currently looking into limiting which characters can be used for the `handle` property (alphabetical characters only), but I might just allow people to use any characters they want (like @, #, $, ^ or even ~).

#### The `bio` string
The `bio` property is pretty self-explanatory, it's your `Profile`'s bio.

#### The `publicKey` string
The `publicKey` property is a string that can hold your public key in case you want to share it. It defaults to an empty string upon creation but can later be modified. I'm still looking into this should be incorporated thus this property is still subject to removal.

#### The `metadata` mapping
The `metadata` property is a string to string mapping that can hold all kinds of miscellaneous information that can be accessed by Dapps. The `namespace` and `metaKey` make the key of this mapping and the `metaValue` is the value the key returns. The default `namespace` that is meant for use in any Dapp is the `global` namespace, but you can have metadata specific to a certain Dapp thanks to the `namespace` feature.

Here's an example of what that would look like:
```
metadata["namespace:metaKey"] = "metaValue";
metadata["global:website"] = "mywebsite.com";
metadata["global:location"] = "Montreal";
metadata["some game:guild name"] = "Da Best Guild";
```

#### The `followerCount` & `followingCount` uint32s
The `followerCount` and `followingCount` are `uint32`s that track how many followers you have and how many people you're following.

#### The `followers` & `following` arrays
The `followers` and `following` are `uint` arrays that track who exactly is following you and who you're following. `uint` is used since the profiles are tracked through their `profileId` and not their `name`.

#### The `createdBy` address
The `createdBy` property stores the address of whoever created a `profile` (it might not necessarily be who currently owns the profile).

#### The `dateCreated` uint64
The `dateCreated` property stores a timestamp of when a `profile` was created on the blockchain.

## Use Cases

You profile/identity could be exported to different social networks where your D-OZ token can represent your profile (with name, handle, follower/following count). Instead of storing user info, social networks could simply concern themselves with the UX. user info would be hosted on the blockchain instead of being hosted off chain. For example, that means people with high follower counts on one platform wouldn't have to start from zero again on other platforms.

Your profile can also be brought to different games as well. If the game studio wants, you progress in other games could also determine your current level in this game (meaning you could potentially start the game at a high level). Current rank could also be based on current follower count or follower-to-following ratio on other social networks.

You could sell your social media profile via a smart contract if it has a high follower count.

## Limitations

D-OZ could potentially be very expensive since you would need to pay gas every time you want to make any state changes to your profile on the blockchain. However, this problem could be mitigated by something I call deferred updating where updating the `Profile` struct is deferred until a larger update can be made in one shot. For example, instead of updating your following count every time you follow someone and paying a gas fee for that, you could decide that every 10 new followings your following count should be updated. that way you only pay 1 gas fee but you can add 10 more profiles to your following count.

Another minor limitation is something inherent to Solidity. Because Solidity does not yet support regexes(regular expressions) we cannot limit what characters can be used for names and handles. That means someone could theoretically have a name like *"#Gt~@? FwI9$^"* and a handle like *"H@<>t&2"*. While no user would be incentivized to create a name and handle like since it would be hard to find and hard for others to remember not to mention terrible branding as well, however depending on a lack of incentive is not enough. If not for the odd names, the lack of regex support also means the *real* Vitalik Buterin could have a name like *"Vitalik Buterin"* and someone else could create a similar name like *"VITALIK BUTERIN"* and it would be allowed which is a worst-case but more likely scenario. One solution that can mitigate this would be to only allow the creation of new profiles through our own Dapp and use regexes via JavaScript on the frontend, however that doesn't prevent savvy users from calling the contracts function directly. Solutions against this are still being looked into.

We are still looking into other modifications we can make to further optimize the contracts and future-proof them.

## Crowdfunding

In Q1, we will be holding a crowdfunding event to fund D-OZ's development. There will both a presale and a main sale. **The presale will take the form of a game** where people who beat the game will gain access to the D-OZ beta test as well as help fund D-OZ's development. The mainsale will be a normal sale where ERC20 tokens are minted in exchange for ether.

The ERC-721 tokens minted during the presale will serve as passes for access to the beta and special status shall be given to beta testers's `Profile`s once the D-OZ contracts are deployed on the mainnet while no use has been devised for the minted ERC-20 tokens. We do not want to make our platform inconvenient to use by forcing the use of a token therefore the utility of the ERC-20 will be subject to change until a suitable use case has been conceived.

Both the presale and mainsale have a combined cap of 15000 ETH (subject to change). If the cap is reached during the presale then the mainsale will not be held.

#### Presale Details

The goal of participants is to mint an ERC721 token proving they have managed to beat the game. This token will grant them access to the D-OZ beta. Once the contracts are deployed, the opening of our presale will be announced on Twitter, Reddit and our blog. There is no time limit, the sale will go on until the cap (15000 ETH) is reached. If the cap is never reached then the mainsale will be held a month later.

#### Mainsale Details

*Details coming soon.*

## Roadmap

This roadmap is subject to change, I'm hoping it'll take less time than planned but I made conservative estimates just in case.

1. Q1 2018
   - Write whitepaper
   - Deploy website
   - Create initial version of Profile contracts
   - Hold the presale event
2. Q2 2018
   - Hire team members and/or contractors to help build D-OZ
   - Redesign contract architecture to make it clearer and upgradeable
   - Code and audit smart contracts
   - Beta test on the Rinkeby test network with those who helped fund the project
3. Q3 2018
   - Audit and deploy on mainnet
   - Make release announcement
   - Develop Profile creator Mini-Dapp
   - Develop Mini-Dapps like the Follow Dapp
4. Q4 2018
   - Invest in promising Dapps that use the D-OZ standard
   - Anything else that sounds cool
