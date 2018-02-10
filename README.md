# ERC-ME: A Standard for Tokenized Pseudo-identities

Last Revision: 08/02/2018

Ghilia Weldesselasie -- <a href='mailto:ghiliaweld@gmail.com'>ghiliaweld@gmail.com</a></br>
<i>Computer Science and Mathematics -- @ Vanier College</i></br>

## Abstract

ERC-ME is a tokenized *Profile* specification standard that enables the creation and use of decentralized pseudo-identities.  The standard aims to build a common informational interface by which social networks, games and other services can host user experiences on their services without hosting or owning user information. ERC-ME hopes to power a standard upon which Dapps can build new social networks and games centered around users without holding user information. A future in which apps are sorely focused on the UX of their apps and crafting the best experience and where users can switch to any apps they like while sill maintaining their following and profile info.

What ERC-ME is going for can be summed up in three words: **Experiences, not services**.

## Table Of Contents

1. [Introduction](#Introduction)
2. [Specification](#specification)
   1. [Overview](#overview)
   2. [The Profile](#the-profile-in-depth)
   3. [ERC-ME Functions](#erc-me-functions)
3. [Use Cases](#use-cases)
4. [Limitations](#limitations)
5. [Roadmap](#roadmap)

## Introduction

What?:

ERC-ME is a tokenized *Profile* specification standard that enables the creation and use of decentralized pseudo-identities.  The standard aims to build a common informational interface by which social networks, games and other services can host user experiences on their services without hosting or owning user information.

Why?:

1. Protocols like uPort have identities tied to proxy contracts which have their own addresses. Moreover, they only hold a limited amount of information with the Dapp specific information staying on the Dapp's servers. This means that your followers cannot be imported across Dapps. uPort identities can: authenticate themselves to a third party, disclose private information about themselves, receive requests for disclosure about themselves, receive and store signed third party verifications about themselves, and sign Ethereum transactions. Which is useful for real identities on-chain and off-chain but lacks certain elements needed for a decentralized pseudo-identity standard.

2. Identity protocols have yet to implement tokenized identity, opting instead to tie identities to address/contracts which might not have been so bad if they implemented this with the ENS. This make identities harder to move around across wallets and impossible to sell to other users since they are locked to an address/contracts and not owned by a wallet.

3. There are no solution currently addressing pseudo-identities and their potential use on social network/game Dapps on the blockchain.


**ERC-ME intends to provide a decentralized standard upon which digital and tokenized pseudo-identities can be used in various Dapps on the blockchain without information being tied to any single service.**

## Specification

### Overview

Each `Profile` on the ERC-ME contract is represented by an *Non-Fungible [ERC-721](https://github.com/ethereum/eips/issues/721) token*, meaning they can be transferred, traded and since they are actually a `struct` that mean that state changes can be made to them. In this section we will go over the specification of each `Profile` and it's associated functions.

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

### ERC-ME Functions

`createProfile()` allows us to create new profile and mint a corresponding ERC-721 token with it.
```
function createProfile(string name, string handle) external payable {
        require(name != ""); // No empty strings as names
        require(handle != ""); // No empty strings as handles
        require(_checkUniqueName(name));
        require(_checkUniqueHandle(handle));

        // Donation logic // If somebody decides to send a donation along with the function,
        // let the function accept and send it to my address.
        CONTROLLER.transfer(msg.value);

        // Mint the profile
        _mint(msg.sender, name, handle);
}
```
`deleteProfile()` allows us to delete our profile.
```
function deleteProfile(uint256 profileId) external {
        // Delete a profile you own
        require(_owns(msg.sender, profileId));
        delete profiles[profileId];
        _transfer(msg.sender, address(0), profileId); // Destroys the ERC-721 token
        ProfileDestroyed(msg.sender, profileId);
}
```

#### `name` functions
`_checkUniqueName()` is a view function that returns a bool (true or false) which tells us whether the name we're using in the ERC-ME contract is unique or not.

```
function _checkUniqueName(string _name) internal view returns(bool) {
        // Loops through our profiles array to check if the name we want to use is already taken
        for (uint i = 0; i <= profiles.length; i++) {
            if (keccak256(_name) == keccak256(profiles[i].name)) {
                return false;
            }
        }
        return true;
}
```
`changeName()` is a function that allows us to change the name of our profile.

```
function changeName(string newName, uint256 profileId) external {
        require(_owns(msg.sender, profileId));
        require(_checkUniqueName(newName));
        profiles[profileId].name = newName;
        NameChange(profileId, newName);
}
```

#### `handle` functions
`_checkUniqueHandle()` is a view function that returns a bool (true or false) which tells us whether the name we're using in the ERC-ME contract is unique or not.

```
function _checkUniqueHandle(string _handle) internal view returns(bool) {
        // Loops through our profiles array to check if the handle we want to use is already taken
        for (uint i = 0; i <= profiles.length; i++) {
            if (keccak256(_handle) == keccak256(profiles[i].handle)) {
                return false;
            }
        }
        return true;
}
```
`changeHandle()` is a function that allows us to change the name of our profile.

```
function changeHandle(string newHandle, uint256 profileId) external {
        require(_owns(msg.sender, profileId));
        require(_checkUniqueHandle(newHandle));
        profiles[profileId].handle = newHandle;
        HandleChange(profileId, newHandle);
}
```

#### `bio` functions
`editBio()` is a function that allows us to edit our profile's bio.
```
function editBio(string newBio, uint256 profileId) external {
        require(_owns(msg.sender, profileId));
        profiles[profileId].bio = newBio;
}
```

#### `publicKey` functions
`changeKey()` is a function that allows us to edit our profile's bio.
```
function changeKey(string newPublicKey, uint256 profileId) external {
        require(_owns(msg.sender, profileId));
        profiles[profileId].publicKey = newPublicKey;
}
```

#### `follow` functions

```
function _newFollow(uint256 _initiatorId, uint256 _targetId) internal {
        /* Triggered when somebody follows another person
        followIncrease is used when deferred updating is enabled
        A following is added to the initiator and a follower is added to the target */

        require(_owns(msg.sender, _initiatorId));
        require(_initiatorId != _targetId); // make sure we're not following ourselves
        require(profiles[_initiatorId].following.indexOf(_targetId) < 0); // make sure we're not refollowing someone
        profiles[_initiatorId].following.push(_targetId);
        profiles[_targetId].followers.push(_initiatorId);
        // ^^ ** Does this work?

        // And now for the follower counts
        profiles[_initiatorId].followingCount = profiles[_initiatorId].following.length;
        profiles[_targetId].followerCount = profiles[_targetId].followers.length;
        // ^^ ** Does this work?
        NewFollow(_initiatorId, _targetId);
    }
```

```
    function _newUnfollow(uint256 _initiatorId, uint256 _targetId) internal {
        /* Triggered when somebody unfollows another person
        A following is subtracted from the initiator and a follower is subtracted from the target */

        require(_owns(msg.sender, _initiatorId));
        require(_initiatorId != _targetId); // make sure we're not unfollowing ourselves
        require(profiles[_initiatorId].following.indexOf(_targetId) >= 0);
        // make sure we're not unfollowing someone we weren't following in the first place
        delete profiles[_initiatorId].following[profiles[_initiatorId].following.indexOf(_targetId)];
        delete profiles[_targetId].followers[profiles[_targetId].followers.indexOf(_initiatorId)];
        // ^^ ** Does this work?

        // And now for the follower counts
        profiles[_initiatorId].followingCount = profiles[_initiatorId].following.length;
        profiles[_targetId].followerCount = profiles[_targetId].followers.length;
        // ^^ ** Does this work?
        NewUnfollow(_initiatorId, _targetId);
    }
    // The 2 functions below will take care of deferred/bulk updating
    // The bulk functions will be the ones called even in the case of a single following
    // since newFollow/newUnfollow have been made internal.
```
The `bulkFollow()` and `bulkUnfollow()` functions allow a profile to take an array of profileIds they want to follow and follow them all at once. This is how deferred updating would be implemented, however there is still more work to do in order to further optimize the follow function since they will consume the most gas an a very frequent rate.
```
    function bulkFollow(uint initiatorId, uint[] multipleTargetIds) external {
        for (uint8 i = 0; i <= multipleTargetIds.length; i++) {
            _newFollow(initiatorId, multipleTargetIds[i]);
        }
    }
```

```
    function bulkUnfollow(uint initiatorId, uint[] multipleTargetIds) external {
        for (uint8 i = 0; i <= multipleTargetIds.length; i++) {
            _newUnfollow(initiatorId, multipleTargetIds[i]);
        }
}
```
#### `metadata` functions
There are two kinds of metadata functions: the **global** metadata functions that can be used to edit global metadata like location or age and the **standard** metadata functions that can be used to accessed Dapp-specific metadata with different namespaces.

```
function editMetadata(string metaKey, string metaValue, uint256 profileId, string namespace) external {
        require(_owns(msg.sender, profileId));
        require(namespace != ""); // Makes sure namespace isn't an empty string
        require(metaKey != ""); // Makes sure metaKey isn't an empty string
        // The code below works for both editing an existing key/value pair
        // and for creating a new pair as well
        // namespace is the app from which this metadata comes from. Ex: Social Dapp
        // ** Do default parameters work in Solidity?
        // metaKey is the key (a string) we will be looking up. Ex: website
        //metaValue is the value that search will return. Ex: https://ghiliweld.github.io
        // profiles[profileId].metadata["Social Dapp:website"] = "https://ghiliweld.github.io"
        profiles[profileId].metadata[namespace + ":" + metaKey] = metaValue;
    }
  ```

  ```
    // editGlobalMetadata and removeGlobalMetadata are subject to removal
    /* I'm deciding between makings seperate functions for global metadata or
    making global a standard the in the docs specification unless the Dapp needs to be specified.*/
    function editGlobalMetadata(string metaKey, string metaValue, uint256 profileId) external {
        require(_owns(msg.sender, profileId));
        require(metaKey != ""); // Makes sure metaKey isn't an empty string
        profiles[profileId].metadata["global:" + metaKey] = metaValue;
    }
```

```
    function removeMetadata(string metaKey, uint256 profileId, string namespace) external {
        require(_owns(msg.sender, profileId));
        // The code below will delete a key/value pair from the metadata mapping
        delete profiles[profileId].metadata[namespace + ":" + metaKey];
        // ^^ ** Does delete work like this?
    }
```

```
    function removeGlobalMetadata(string metaKey, uint256 profileId) external {
        require(_owns(msg.sender, profileId));
        delete profiles[profileId].metadata["global:" + metaKey];
}
```
## Use Cases

You profile/identity could be exported to different social networks where your ERC-ME token can represent your profile (with name, handle, follower/following count). Instead of storing user info, social networks could simply concern themselves with the UX. user info would be hosted on the blockchain instead of being hosted off chain. For example, that means people with high follower counts on one platform wouldn't have to start from zero again on other platforms.

Your profile can also be brought to different games as well. If the game studio wants, you progress in other games could also determine your current level in this game (meaning you could potentially start the game at a high level). Current rank could also be based on current follower count or follower-to-following ratio on other social networks.

You could sell your social media profile via a smart contract if it has a high follower count.

## Limitations

ERC-ME could potentially be very expensive since you would need to pay gas every time you want to make any state changes to your profile on the blockchain. However, this problem could be mitigated by something I call deferred updating where updating the `Profile` struct is deferred until a larger update can be made in one shot. For example, instead of updating your following count every time you follow someone and paying a gas fee for that, you could decide that every 10 new followings your following count should be updated. that way you only pay 1 gas fee but you can add 10 more profiles to your following count.

Another minor limitation is something inherent to Solidity. Because Solidity does not yet support regexes(regular expressions) we cannot limit what characters can be used for names and handles. That means someone could theoretically have a name like *"#Gt~@? FwI9$^"* and a handle like *"H@<>t&2"*. While no user would be incentivized to create a name and handle like since it would be hard to find and hard for others to remember not to mention terrible branding as well, however depending on a lack of incentive is not enough. If not for the odd names, the lack of regex support also means the *real* Vitalik Buterin could have a name like *"Vitalik Buterin"* and someone else could create a similar name like *"VITALIK BUTERIN"* and it would be allowed which is a worst-case but more likely scenario. One solution that can mitigate this would be to only allow the creation of new profiles through our own Dapp and use regexes via JavaScript on the frontend, however that doesn't prevent savvy users from calling the contracts function directly. Solutions against this are still being looked into.

We are still looking into other modifications we can make to further optimize the contracts and future-proof them.

## Roadmap

This roadmap is subject to change, I'm hoping it'll take less time than planned but I made conservative estimates just in case.

1. Q1 2018
   - Write whitepaper
   - Deploy website
   - Create initial version of Profile contracts
   - Hold the crowdfunding event
2. Q2 2018
   - Hire team members and/or contractors to help build ERC-ME
   - Redesign contract architecture to make it clearer and upgradeable
   - Code and audit smart contracts
   - Beta test on the Rinkeby test network with those who helped fund the project
3. Q3 2018
   - Audit and deploy on mainnet
   - Make release announcement
   - Develop Profile creator Mini-Dapp
   - Develop Mini-Dapps like the Follow Dapp
4. Q4 2018
   - Invest in promising Dapps that use the ERC-ME standard
   - Anything else that sounds cool
