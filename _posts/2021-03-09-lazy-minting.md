---
layout: post
title:  "Lazy Minting NFTs"
date:   2021-03-09 23:06:00 -0500
categories: blog
---
### What is Lazy Minting
[OpenSea](https://opensea.io) is an NFT marketplace that supports Lazy Minting. Lazy Minting (sometimes called light minting) enables an NFT creator to “mint” an NFT without actually declaring it’s existence to the main Ethereum blockchain. This is nice for a variety of reason, but the two I am most interested in are

1. The NFT is not actually created until there is a buyer. No gas is wasted.
2. The buyer is the one who (eventually) pays for the gas.

I am working on a project that would benefit from a “Lazy Minting” like experience. I am minting 1000 NFTs called “cryptocassettes”. I can save the details for another post, but what’s important is that each cassette is unique along a few dimensions (cassette color, tape color, size etc).

I want to create 1000 of these things, but the options for doing so (at least the easy way) are all cost prohibitive. Storing all of the information about each cassette on the contract itself is way too much data to be storing on the contract, and would make deploying the contract way too expensive. I could mint each NFT to myself before announcing the project, then Transfer the rights to interested purchasers, but this would mean I have to incur all of the gas fees myself, up front. At around 50 USD per tx at current cost, I would be looking at 50,000 USD. Yikes.

Clearly, Lazy Minting would be nice. It would be great if I could declare my intent to mint these NFTs without having the pay the gas to actually mint them into existence. I would save money, and if nobody ends up wanting these damn things then at least I’m not killing a fuck ton of trees by minting 1000 digital cassettes for nothing.

Unfortunately, I’ve been unable to find the implementation details for OpenSea’s Lazy Minting strategy, so I’ve decided to come up with my own “Lazy Minting” method that accomplishes the same sort of thing. Consider this the “poor man’s lazy minting scheme” because I’m sure there are dozens of ways this scheme could go wrong, but I’m amusing myself by believing it will work.

### The Details
My particular project has a few preconditions.
1. I want the people claiming my NFTs to pay their own gas.
2. I do not want them to be able to manipulate the NFT metadata outside of the details I define when I “lazily” instantiate the NFT. In other words, if I define a cassette with color “red” I do not want someone calling the mint function but changing the color to “green”.

I accomplished my lazy minting strategy by utilizing transaction signatures. The `mint` function of my smart contract accepts two params, the cassette metadata, and a signature. Offline, I use my private/public key pair to sign a message containing the cassette metadata. When someone is interested in buying a cassette, I give them the signature, and tell them the params that correspond to the signature. (Or most likely, all of this is abstracted behind a UI so the buyer has no idea this is what is happening…) When the buyer calls the `mint` function with these param, the function hashes the message and verifies the signature. This returns an address — ideally my address, which is saved in the contract itself. The function will throw if the messages do not match, which would happen in the case that the metadata function was maliciously submitted and incorrect. If the metadata is correct, the addresses will match and the NFT will mint.

By going through this flow, I am able to create “IOUs” with the intent of creating an NFT without actually minting the NFT itself. A buyer is able to mint an NFT from my contract and pay the gas entirely, without the ability to create any NFT with attributes that have not been previously approved by me.

There is a known issue unfortunately. Let’s say we only want one NFT with a particular set of metadata. It's possible for a bad actor to intercept or even reuse the same signature/metadata combination to continuously mint NFTs with the approved set of metadata. There is currently no way to "expire" a signature as far as I know. Only solution I have to that problem so far is keeping an array of signature indices and including that in the message you sign. When the function gets the message as a param, it can write to the array of signature indices marking it as "used". Then, all prior minting transactions have to make sure the index is unused by reading the array. Unfortunately, this imposes additional costs, so I am experimenting with more elegant solutions.

See my [repo](https://github.com/mcgingras/lazy-mint) for implementation details.