# TR8 Protocol

TR8 protocol is a permissionless public good protocol, on the Superchain, for traits (TR8s) of many types, including event attendance, credentials, in-game assets, badges, and more. TR8 protocol uses Ethereum Attestion Searvice (EAS) as a base layer, providing customization, onchain, structured data for issuers of all types of credentials. NFT metadata is store onchain in attestations.

## Live Demo: Try it Now

A basic example front end for creating event-based TR8s can be found at https://tr8.me



## Other Repos

Some code for TR8 protocol lives in other repos under the [tr8-protocol](https://github.com/tr8-protocol/) organisation on Github:
- [Frontend](https://github.com/tr8-protocol/frontend) - an example front end (work in progress)
- [Events Example Frontend](https://github.com/tr8-protocol/tr8-api/tree/main/dapp) - an example front-end focused on event attedance TR8s (working demo)
- [API](https://github.com/tr8-protocol/tr8-api) - an example API

## Permissionless

TR8 Protocol is permissionless. Anyone can create a drop/collection on _any_ topic with _any_ content. TR8 drops do *NOT* have to be vetted nor approved by a committee or company or organization or DAO. A side-effect of this permissionless nature is that content-filtering becomes the responsibility of front-ends, each of which can apply their own standards for curating content. TR8 Protocol envisions many different front-ends, some general in nature and others focused on specific applications like events, educational credentials, in-game assets, etc. 

## Composabilty

TR8 is composable through the structured data in the source attestations, and well as via optional hook contracts that issuers can deploy to tightly integrate TR8s into their own protocols and control who can claim their TR8s and what _extras_ happen when they do.

## OP Stack Superchain

The Superchain of OP Stack chains is the home for TR8 Protocol, currently deployed to `Optimism`, `Base`, and `Zora` (testnets).

## Cross-chain

All TR8 NFTs are cross-chain by nature and can currently be moved between Optimism, Base, and Zora, with more to follow. Cross-chain transportation provided by `LayerZero` general message passing (GMP).

### How it was built

At its core, TR8 protocol use EAS attestations as the base layer. There are two kinds of attestations that TR8 issuers and claimers make:

1. *"New Drop" Attestations.* This is an attestation to create a new drop or collection, whether it be a badge for event attendance, an educational certificate, or something else. The attestation includes the structured data for the drop as well as lists of who can issue and/or claim the TR8s. When a new drop attestation is made, a dedicated NFT contract/collection is deployed for the drop. The collection is _owned_ by the issuer enabling them to customize the collection's appearance on OpenSea and other marketplaces and directories. A drop can have an _expiry date_, a deadline for minting the TR8s in the drop. Drops can be configured as soulbound (non-transferable), or trade-able.
2. *Mint/Claim Attestations.* These attestion can be made either by an _issuer_ (admin) or a claimer. In both cases the `recipient` of the attestation is the one who will receive the TR8, if eligible. Configured _issuers_ for a drop can _issue_ (mint) a TR8 to _any_ address. Configured _claimers_ can claim a TR8 for themselves (they are both the `attester` and the `recipient` in these claim attestations). When these attestation are made, _hooks_ are called if configured, enabling limitless composability and providing flexibility on gating who can and cannot claim TR8s in the drop (more on hooks below). Eligible recipients will receive a TR8 NFT for the Drop, adding to their onchain identity.

### Namespaces

TR8 drops feature optional reserved _namespaces_, enabling issuing organizations to group their drops under a single namespace that is reserved for them.


### TR8 Protocol Contracts

![TR8 Contracts](https://violet-manual-egret-987.mypinata.cloud/ipfs/QmPLABzvFZf5B7dwD4u4NbuYy8NLgfcmbQapdqaLHZiZL4)

While TR8 users interact primarily through making attestations to the EAS contract, there are several contracts deployed as part of TR8 protocol.

- `TR8.sol` - The TR8 contract provides several functions.  It acts an a EAS `SchemaResolver`, hooks that get called when each attestation is made or revoked. When a minting/claiming attestation is made, the `onAttest()` hook triggers the minting of the TR8 to eigible attestation recipients. When a "new TR8 drop" attestation is made, the hook triggers the deployment of a dedicated NFT contract for the drop. In this way, the contract also acts as a Factory contract which minimally clones NFT contracts. The attestation includes NFT metadata and access permissions for issuers and claimers, stored onchain within the attestation and/or newly deployed contract. Finally, the `TR8` contract acts a registry for drops and keeps track of reserved _nameSpaces_ for issuers.
- `TR8Nft.sol` - This is the implementation contract for TR8 NFTs. Each drop gets it own contract/collection which gets [cloned](https://docs.openzeppelin.com/contracts/4.x/api/proxy#Clones) from this contract. These NFTs are cross-chain NFTs that implement the `ERC721Transportable` interface to faciliate arrivals and departures.
- `TR8Transporter.sol` - Powered by LayerZero, this contract is a transport hub for _many_ TR8 NFT contracts. One transporter is deployed per chain and all movement of NFTs move through them. If a transporter detects that a drop NFT contract has not yet been deployed on the destination chain, it calls out to the `TR8` contract on that chain to deploy, at the same address as the home chain.
![TR8 Transporters](https://violet-manual-egret-987.mypinata.cloud/ipfs/QmZt8FYKxYuTgpZsBv8NuduYWdoPY4yWRr9F882vGhYbzw)

#### Optional TR8 Hook contracts

TR8 Protocol is composable, and issuers can create their own `TR8Hook` contracts that get called each time a minting/claiming attestation is made. Hooks can add logic to allow/deny minting, integrate with third party protocols, and thousands of other things. Four example hook contracts were created and deployed:

- `TR8HookNFTClaimer.sol` - This hook enables claiming of a TR8 only if the recipient is a current owner of an NFT from another collection. This could be used to issue TR8s to members of a DAO, or to reward NFT community members.
- `TR8HookNeedEth.sol` - This hook requires the the recipient has some amount of native token (ether). This could be used to limit hoarding/scamming of TR8 drops. One can imagine a similar contract that requires a certain balance of a specific `ERC20` token.
- `TR8HookFaucet.sol` - This hook acts as faucet, sending tokens to eligible recipients. Note that when a hook is specified for a drop, it doesn't override the specified list of claimers who are allowed to mint/claim the TR8. Rather a hook can revert to prevent the issuance of the TR8 or add new addresses to the claimers list. In this example hook, it does neither, but will drip tokens only to those already eligible for the TR8.
- `TR8HookStreamer.sol` - This hook works similar to the faucet, by starts a real-time stream of tokens to the TR8 receipient, powered by the Superfluid protocol. One can imagine gaming applications that perform similar actions.


#### Deployed Contracts Addresses

Note all equivalent contracts are _deployed to the same address on each chain_.

- `TR8Nft` contract: `0x868F83D1c9349B6c7bA793A0D71Dfb66e60e01Ef`: [Optimism](https://goerli-optimism.etherscan.io/address/0x868F83D1c9349B6c7bA793A0D71Dfb66e60e01Ef) - [Base](https://goerli.basescan.org/address/0x868F83D1c9349B6c7bA793A0D71Dfb66e60e01Ef) - [Zora](https://testnet.explorer.zora.energy/address0x868F83D1c9349B6c7bA793A0D71Dfb66e60e01Ef)
- `TR8` contract: `0xC3b0c31C16D341eb09aa3698964369D2b6744108`: [Optimism](https://goerli-optimism.etherscan.io/address/0xC3b0c31C16D341eb09aa3698964369D2b6744108) - [Base](https://goerli.basescan.org/address/0xC3b0c31C16D341eb09aa3698964369D2b6744108) - [Zora](https://testnet.explorer.zora.energy/address/0xC3b0c31C16D341eb09aa3698964369D2b6744108)
- `TR8Transporter` contract: `0x54C9935e58141cc5b1B4417bb478C7D25228Bfc0`: [Optimism](https://goerli-optimism.etherscan.io/address/0x54C9935e58141cc5b1B4417bb478C7D25228Bfc0) - [Base](https://goerli.basescan.org/address/0x54C9935e58141cc5b1B4417bb478C7D25228Bfc0)

Example hook contracts were deployed only to the "home chain", Optimism:
- `TR8HookFaucet` contract: `0x6072fB0F43Bea837125a3B37B3CF04e76ddd3f19`: [Optimism](https://goerli-optimism.etherscan.io/address/0x6072fB0F43Bea837125a3B37B3CF04e76ddd3f19)
- `TR8HookStreamer` contract: `0xFc3d67C7A95c1c051Db54608313Bd62E9Cd38A76`: [Optimism](https://goerli-optimism.etherscan.io/address/0xFc3d67C7A95c1c051Db54608313Bd62E9Cd38A76)
- `TR8HookNFTClaimer` contract: `0x9F570E88B5CDef206d2633aA3278ac5bceCd4cD8`: [Optimism](https://goerli-optimism.etherscan.io/address/0x9F570E88B5CDef206d2633aA3278ac5bceCd4cD8)
- `TR8HookNeedEth` contract: `0x4a62d7B300b505e44066E51B0b635ddb5044955d`: [Optimism](https://goerli-optimism.etherscan.io/address/0x4a62d7B300b505e44066E51B0b635ddb5044955d)

## Sponsors

- *Ethereum Attestation Service (EAS)*: EAS is the foundation upon which TR8 has been built. It is the base layer for TR8 Protocol. [#](https://github.com/tr8-protocol/tr8-contracts/blob/main/contracts/TR8.sol#L85) [#](https://github.com/tr8-protocol/tr8-contracts/blob/main/contracts/TR8.sol#L172) [#](https://github.com/tr8-protocol/tr8-api/blob/main/functions/tr8/index.js#L31)
- *Optimism*: The home chain for TR8 protocol is Optimism, where all attestations and TR8 drops happen. TR8 Protocol is intended to be a permissionless public good. See OP deplyed addresses above. [#](https://github.com/tr8-protocol/tr8-contracts/blob/main/test/index.js#L22)
- *Base*: TR8 Protocol NFTs can be moved between OP Stack chain and has been deployed to Base. See Base deployed addresses above [#](https://github.com/tr8-protocol/tr8-contracts/blob/main/test/index.js#L29) 
- *Zora*: TR8 Protocol NFTs can be moved between OP Stack chain and has been deployed to Zora. See Zora deployed addresses above [#](https://github.com/tr8-protocol/tr8-contracts/blob/main/hardhat.config.js#L569) 
- *LayerZero*. Cross-chain transportation of TR8s is provided by LayerZero General Message Passing (GMP) [#](https://github.com/tr8-protocol/tr8-contracts/blob/main/contracts/TR8Transporter.sol#L48)
- *Arbitrum*: Another home chain for TR8 protocol, where all attestations and TR8 drops happen. TR8 Protocol is intended to be a permissionless public good. See Arb deployed addresses above. [#](https://github.com/hebx/tr8-contracts/blob/main/test/index.js#L43)
- *Scroll*: TR8 Protocol NFTs has been deployed to Scroll. See Scroll deployed addresses above [#](https://github.com/hebx/tr8-contracts/blob/main/test/index.js#L36) 