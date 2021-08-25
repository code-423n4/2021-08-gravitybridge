# Althea Gravity Bridge Contest

This `README.md` contains a set of checklists for our contest collaboration.

Your contest will use two repos: 
- **a _contest_ repo** (this one), which is used for scoping your contest and for providing information to contestants (wardens)
- **a _findings_ repo**, where issues are submitted. (We'll set that one up later.) 

Ultimately, when we launch the contest, this contest repo will be made public and will contain the smart contracts to be reviewed and all the information needed for contest participants. The findings repo will be made public after the contest is over and your team has mitigated the identified issues.

Some of the checklists in this doc are for **C4 (🐺)** and some of them are for **you as the contest sponsor (⭐️)**.

---

## ⭐️ Sponsor: Provide contest details

Under "SPONSORS ADD INFO HERE" heading below, include the following:

- [ ] Name of each contract and:
  - [ ] lines of code in each
  - [ ] external contracts called in each
  - [ ] libraries used in each
- [ ] Describe any novel or unique curve logic or mathematical models implemented in the contracts
- [ ] Does the token conform to the ERC-20 standard? In what specific ways does it differ?
- [ ] Describe anything else that adds any special logic that makes your approach unique
- [ ] Identify any areas of specific concern in reviewing the code
- [ ] Add all of the code to this repo that you want reviewed
- [ ] Create a PR to this repo with the above changes.

---

# ⭐️ Sponsor: Provide marketing details

- [X] Your logo (URL or add file to this repo - SVG or other vector format preferred)
- [X] Your primary Twitter handle
- [ ] Any other Twitter handles we can/should tag in (e.g. organizers' personal accounts, etc.)
- [ ] Your Discord URI
- [X] Your website
- [ ] Optional: Do you have any quirks, recurring themes, iconic tweets, community "secret handshake" stuff we could work in? How do your people recognize each other, for example? 
- [ ] Optional: your logo in Discord emoji format

---

# ⭐️ Sponsor: Contest prep
- [ ] Make sure your code is thoroughly commented using the [NatSpec format](https://docs.soliditylang.org/en/v0.5.10/natspec-format.html#natspec-format).
- [ ] Modify the bottom of this `README.md` file to describe how your code is supposed to work with links to any relevent documentation and any other criteria/details that the C4 Wardens should keep in mind when reviewing. ([Here's a well-constructed example.](https://github.com/code-423n4/2021-06-gro/blob/main/README.md))
- [ ] Please have final versions of contracts and documentation added/updated in this repo **no less than 8 hours prior to contest start time.**
- [x] Ensure that you have access to the _findings_ repo where issues will be submitted.
- [x] Promote the contest on Twitter (optional: tag in relevant protocols, etc.)
- [x] Share it with your own communities (blog, Discord, Telegram, email newsletters, etc.)
- [ ] Optional: pre-record a high-level overview of your protocol (not just specific smart contract functions). This saves wardens a lot of time wading through documentation.
- [ ] Designate someone (or a team of people) to monitor DMs & questions in the C4 Discord (**#questions** channel) daily (Note: please *don't* discuss issues submitted by wardens in an open channel, as this could give hints to other wardens.)
- [ ] Delete this checklist and all text above the line below when you're ready.

---

# Althea Gravity Bridge contest details
- $100,000 USDC (+ tokens) main award pot
- Join [C4 Discord](https://discord.gg/EY5dvm3evD) to register
- Submit findings [using the C4 form](https://code423n4.com/2021-08-althea-gravity-bridge-contest/submit)
- [Read our guidelines for more details](https://docs.code4rena.com/roles/wardens)
- Starts August 26 00:00 UTC
- Ends September 8 23:59 UTC

| Glossary| |
|-------------------------------|------------------------------------------------------|
| Gravity bridge | a cross chain bridge between [Cosmos-SDK](https://github.com/cosmos/cosmos-sdk/) and EVM based blockchains |
| Gravity Contract | The Gravity Bridge [Solidity contract](https://github.com/althea-net/cosmos-gravity-bridge/blob/main/solidity/contracts/Gravity.sol) that holds bridge funds on Ethereum  |
| Gravity module | The [Cosmos-SDK module](https://docs.cosmos.network/master/building-modules/intro.html) which makes up the SDK chain side of the bridge |
| Orchestrator | A special process run only by the validators of the Cosmos-SDK chain. Performing Oracle functions and producing Ethereum signatures |
| Relayer | A permissionless role that submits signatures from the validators to Ethereum as transactions |

# Contest Scoping

Gravity Bridge is a cross chain bridge between [Cosmos-SDK](https://github.com/cosmos/cosmos-sdk/) and EVM based chains.

The Gravity Bridge offers the ability to send funds from Ethereum to Cosmos and back, as well as the ability to deploy ERC20 representations of Cosmos based assets on Ethereum and transfer them bi-directionally as well.

Gravity is a complete system, made up of the Cosmos SDK 'module', a Solidity contract, and associated relaying/oracle code. This means the scope of this audit is quite large, covering the complete system across three programming languages.

The commit hash `d380f0c426c1366a82f619c01b2bffd639c1d998` of the repo [github.com/althea-net/cosmos-gravity-bridge](https://github.com/althea-net/cosmos-gravity-bridge/) describes the exact code under audit.

This helpful [link](https://github.com/althea-net/cosmos-gravity-bridge/tree/d380f0c426c1366a82f619c01b2bffd639c1d998) allows you to browse code specifically from this hash and you may find it useful when linking or otherwise referring to findings.

The [Gravity Bridge developer guide](https://github.com/althea-net/cosmos-gravity-bridge/tree/d380f0c426c1366a82f619c01b2bffd639c1d998#developer-guide) covers how to setup your development environment, provides a [detailed](https://github.com/althea-net/cosmos-gravity-bridge/blob/d380f0c426c1366a82f619c01b2bffd639c1d998/docs/developer/code-structure.md) walkthrough of how the bridge works, and finally a useful list of [hotspots](https://github.com/althea-net/cosmos-gravity-bridge/blob/d380f0c426c1366a82f619c01b2bffd639c1d998/docs/developer/hotspots.md) or key areas in the code where problems can be found.

All contributions are welcome! Gravity bridge is a complex system but you don't need to understand the entire flow and all three languages to contribute. Using the [hotspots](https://github.com/althea-net/cosmos-gravity-bridge/blob/d380f0c426c1366a82f619c01b2bffd639c1d998/docs/developer/hotspots.md) list as a guide anyone with Go and Rust know-how can get started.

## Smart Contracts

### Gravity.sol (611 sloc)

[Gravity.sol](https://github.com/althea-net/cosmos-gravity-bridge/blob/d380f0c426c1366a82f619c01b2bffd639c1d998/solidity/contracts/Gravity.sol)

Is the 'Ethereum side' of the bridge. It's purpose is to store a representation of the Cosmos validator set and allow updates to that validator set, as well as deposits and withdraws to and from the contract. The events emitted by this contract are used by the oracle component of the bridge to perform actions on the Cosmos side of the bridge.

As part of a storage optimization the full validator set and voting power is not stored, only it's hash. The client submitting transactions must provide this data when it submits a transaction, this is a dramatic reduction in cost that makes updating validator sets with hundreds of thousands of frequently changing members cost feasible.

