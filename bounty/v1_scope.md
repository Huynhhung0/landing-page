The Forecast Foundation looks forward to working with the security community to find security vulnerabilities in order to keep the Augur protocol safe. 

# SLA
The Forecast Foundation will make a best effort to meet the following SLAs for hackers participating in our program:

* Time to first response (from report submit) - 1 business days
* Time to triage (from report submit) - 1 business days 
* Time to bounty (from triage) - 5 business days

We’ll try to keep you informed about our progress throughout the process.

# Disclosure Policy
* Before discussing your findings publicly, please allow us time to fix the vulnerability and ask our permission before doing so.
* Follow HackerOne's [disclosure guidelines](https://www.hackerone.com/disclosure-guidelines).


# Program Rules
* Please provide detailed reports with reproducible steps. If the report is not detailed enough to reproduce the issue, the issue will not be eligible for a reward.
* Submit one vulnerability per report, unless you need to chain vulnerabilities to provide impact.
* When duplicates occur, we only award the first report that was received (provided that it can be fully reproduced).
* Multiple vulnerabilities caused by one underlying issue will be awarded one bounty.
* Social engineering (e.g. phishing, vishing, smishing) is prohibited.
* Make a good faith effort to avoid privacy violations, destruction of data, and interruption or degradation of our service. Only interact with accounts you own or with explicit permission of the account holder.
* You are ineligible for bounty rewards if the vulnerability submitted is already known by the Forecast Foundation, if it's publicly disclosed prior to the completion of the bounty process with the Forecast Foundation, or if it's found to have been exploited on the main Ethereum network.

# Rewards
Our rewards are based on severity per CVSS (the Common Vulnerability Scoring Standard). Please note these are general guidelines, and that reward decisions are up to the discretion of the Forecast Foundation.

| Critical (9.0 - 10.0) | High (7.0 - 8.9)  | Medium (4.0 - 6.9) | Low (0.1 - 3.9) |
|----------|--------|---------|------|
|  $200,000  | $5,000  |  $2,500    | $1,000 |

The above amounts are base awards and bonuses will be awarded at the discretion of Augur.

# What we’re interested in

The most important class of bugs we’re looking for are any parts of the contract code which would either result in a loss of funds or the platform becoming broken or un-usable.

### Loss Of Funds
A loss of funds bug includes any vulnerability where a user can siphon assets from other users or the platform in an unintended way. If for example a user was able to take ETH or Rep or shares in a market 
 from a contract in an unintended way, or was able to cause a state change in the contracts that made them able to take more than would normally be allowed.

Also included in this would be any bug allowing someone to lock up funds in such a way that they are irrecoverable.

### Manipulating Open Interest
Our contracts keep a record of the current Open Interest (OI) escrowed within the platform for trading. This value is used to determine the fee which is taken from traders and paid to reporters on a weekly basis. If someone can find a way to arbitrarily manipulate this value they could render the platform unusable by increasing fees to an absurd amount (33%) or keep them incorrectly low (0.01%). OI should only be “manipulatable” through the purchase and sale of complete sets and with the correct amount of corresponding ETH escrowed in markets)

### Forking State
One of the key features Augur provides is the ability to fork the platform into multiple universes, each corresponding to an outcome in a market that proved too contentious for traditional dispute mechanics. When this happens the system enters a special state called “forking” and new child universes get spawned that should behave normally. Because this behavior is complicated and difficult to create and think through completely it is worth investigating specifically to see if any broken state arises or could be caused during forking.


# Platform Architecture and Setup

At a high level the components that make up our stack are:

* [__Augur Core__](https://github.com/AugurProject/augur-core): The Ethereum contracts written in Solidity
* [__Augur Node__](https://github.com/AugurProject/augur-node): The Augur Ethereum Node middleware service
* [__Augur JS__](https://github.com/AugurProject/augur.js): A JS middleware library to make interacting with our contracts simpler in client code  
* [__Augur UI__](https://github.com/AugurProject/augur): The Augur client code

##### Ethereum Nodes
Though not technically a part of our software stack a component that will always be involved is an Ethereum node.

We provide a hosted version for the Rinkeby test net at __https://rinkeby.augur.net/ethereum-http__ & __wss://rinkeby.augur.net/ethereum-ws__

We also provide docker images some of which are pre-populated with our contracts which should be used when you want to run a local Ethereum node:

* __https://hub.docker.com/r/augurproject/dev-pop-geth__ should be used normally. This image allows you to manipulate time on the Augur platform.
* __https://hub.docker.com/r/augurproject/dev-pop-normtime-geth/__ is the same as the above image but time will progress normally on chain instead of being controlled by you.
* __https://hub.docker.com/r/augurproject/dev-node-geth__ This is just a blank Ethereum node image to be used when you want to do a deployment of the contracts yourself

You can pull and run these via commands such as this:

```
docker pull augurproject/dev-pop-geth:latest
docker run -it -p 8545:8545 -p 8546:8546 augurproject/dev-pop-geth:latest
```


## Setting up your testing environment

There are a variety of ways to set up your environment and begin interacting with the platform.

Starting from easiest to most involved:

### Hosted Rinkeby UI

We host a [beta test version of the client](dev.augur.net) which can be connected to with Metamask pointed at Rinkeby.

It is configured to point at our hosted Ethereum Node and Augur Node for Rinkeby.

### Augur App

Augur App is a desktop application we provide that packages Augur Node & the Augur UI along with a GUI to select an Ethereum endpoint to connect to.

Instructions for installing or building it from source can be found [here](https://github.com/AugurProject/augur-app).

By default it points at our hosted Rinkeby Ethereum node. It will also work with one of our Ethereum node docker images if more control is desired.

### Local Setup
[Instructions can be found here](https://github.com/AugurProject/augur/blob/master/docs/dev-local-node.md)


## Architecture and Components


### [__Augur Core__](https://github.com/AugurProject/augur-core)

The actual Augur contracts, automated tests, and deployment related scripting. This repo and the associated contract code are our primary concern and should be the main area of focus in finding vulnerabilities since problems here cannot be fixed trivially once in the wild.

A very detailed and thorough explanation of the platform can be found [here](https://www.augur.net/whitepaper.pdf)

The following is a very high level summary of how the contracts work and interact:

##### Delegation

Something important to note about our contracts is that many of them (any that inherit from the `DelegationTarget` contract) are actually just instances of the `Delegator` contract which point at the one registered implementation version.

##### Deployment

The first contract uploaded and the contract which most defines an Augur deployment is the `Controller` contract. This contract maintains a registry of the legitimate Augur contracts and a whitelist that gives those contracts elevated privileges within the platform.

The second contract uploaded is the `Augur` contract which is the central authority for all logs emitted and the approval target for users to authorize the platform move their ETH around in the form of our `Cash` ETH wrapper contract.

After this all the remaining contracts are uploaded and whitelisted as appropriate.

##### Universe

the `Universe` represents a version of the deployed Augur platform that corresponds to a chain of truth. Initially there is one universe, but there could be child universes and children of those if forks occur. A fork occurs when a `Market` could be not be resolved easily, and so there are new `Universe`s for any outcome in that `Market`.

##### Markets

`Market`s are created through convenience methods found on the `Universe`, and are the central component to trading. `Market`s ask a question and users may purchase and trade shares indicating a belief in that questions outcome. Funds are escrowed within the `Market` for this purpose. At the `Market`s end time the reporting process will determine how much the system will award users holding shares for a particular position.

##### Trading

The contracts located under `trading` all deal with the creation, sale, and transfer of shares in a particular `Market`.

Users can place orders to purchase a position, fill specific orders, or request that the system fill whatever orders are possible then create an order for whatever remains. Fees are collected by the system to compensate reporters and market creators whenever a "complete set" is sold back to the system during this process. A "complete set" is just a set of all the shares offered by a `Market`.

Trading never ends, even after a `Market` has finished reporting and even during a fork in the `Universe`.

##### Reporting

Once a `Market` had reached it's end time it can be reported on to indicate the result of the question it asked.

An excellent infographic describing this process can be found [here](http://jonathanwillems.com/infographic-augur/).


### [__Augur Node__](https://github.com/AugurProject/augur-node)

As querying & parsing Ethereum logs via client code can become extremely expensive time wise we built a middleware service to run on top of Ethereum which provides a more efficient querying API for Augur. On startup it will connect to a local Ethereum node and based on the network ID will sync itself with the contracts currently specified by __Augur JS__ on the network.

It maintains a local sqlite database of relevant Augur data, such as the orderbook, reporting & disputing results, and profit / loss data based on user trades.

Since Augur Node is primarily expected to be run as a local service the ability to crash, DDOS, or SQL inject it remotely via a getter is not a primary concern, but are all still of interest.

An attack which creates false data or a corrupt state done entirely by legitimate ethereum logs would be a more concerning exploit.

Both __Augur JS__ and __Augur UI__ depend on connecting to __Augur Node__ via websocket to function correctly.


### [__Augur JS__](https://github.com/AugurProject/augur.js) + [__Augur UI__](https://github.com/AugurProject/augur)

The client code is separated into a middleware component __Augur JS__ and the actual web application __Augur UI__.

The __Augur JS__ middleware is notably where the [mapping from network id to contract addresses](https://github.com/AugurProject/augur.js/blob/master/src/contracts/addresses.json) is stored.

The typical client exploits are up for consideration in these components, though are not especially critical since they are easy to fix and the initial expectation is that people access the UI locally.

These components do provide some helpful utilities to aid in testing:

##### Flash Scripts

The __Augur JS__ repo contains a [collection of scripts](https://github.com/AugurProject/augur.js/tree/master/scripts/flash) which can be run and inspected via the command:

```
node scripts/flash
```

A few notable useful examples:

* Listing existing markets
* Moving time forward
* Creating/Filling Orders
* Reporting/Disputing Markets
* Causing a fork

##### Browser Console commands

In a development environment we expose the __Augur JS__ api and Redux state such that they can be accessed via browser console by __augur__ and __state__ respectively.

This can be very useful to see what the state of your connection is as well as what the application believes current state is.

#Out of scope vulnerabilities
When reporting vulnerabilities, please consider (1) attack scenario / exploitability, and (2) security impact of the bug. The following issues are considered out of scope:
 
* Clickjacking on pages with no sensitive actions.
* Attacks requiring MITM or physical access to a user's device.
* Previously known vulnerable libraries without a working Proof of Concept.
* Missing best practices in SSL/TLS configuration.
* Denial of service that is not a result of application engineering.
* Content spoofing and text injection issues without showing an attack vector/without being able to modify HTML/CSS.
* Game theory attacks which are noted already in the whitepaper (e.g parasitic markets, malicious resolution source).

# Known Contract Bugs
The following vulnerabilities / bugs are already known and are not eligible for bounty:

## Incorrect Reporting Fee Calculation
The reporting fee in Augur is calculated and adjusted by comparing the OI within the platform to a target OI which is based on the price of REP. The goal of this is to dynamically adjust the fee rate downward when the price is too speculative and upward when the price does not reflect the fees being collected.

There is an error in the current contracts however which will prevent the fee from ever rising. Namely the `getRepMarketCapInAttoeth` function does not properly convert units and will always be many orders of magnitude too high when comparing to the target market cap.

While this is a very serious problem for the platform long term the intention is to release a v2 of the contracts within a relatively short time frame and since the price of REP is still highly speculative relative to OI this will almost certainly not become a problem.

## Orderbook Linked List Ordering Bugs
The structure and logic for the on chain orderbook is spread throughout multiple contracts and is somewhat complex. While generally working correctly there are few known contract logic errors which can cause the orderbook to end up in a broken state.

The first is in the `OrdersFetcher` contract within the `descendOrderList` function. Note that if it finds an order of equal price it stops traversing. This is incorrect behavior since a new order of the same price should be considered worse than _all_ orders of that price rather than just the first found. The result is incorrectly ordered orders of the same price, which while not technically correct is a minor issue.

The second ordering bug is found in the `Orders` contract in the `updateWorstBidOrder` and `updateWorstAskOrder` functions. Both of these will only update the respective best and worst orders when they are _strictly_ worse price wise. A new order of the same price however should actually become the new worse order. The result of this bug is that the linked list can become broken and orders may end up being hidden in the orderbook. Steps within the UI have already been implemented to help order creators when this occurs rarely.

## 8 Outcome Categorical Markets Broken in Disputing
8 outcome categorical markets hide the last outcome in replace of invalid during a dispute, showing 8 total outcomes when it should be 9 (including invalid). This issue was reported to the Forecast Foundation by Peter Benesch on February 10th, and the Foundation issued a hotfix release of Augur App to address this UI problem. A market was about to resolve incorrectly because it couldn't be disputed effectively.

## Forking Market Payout  Not Properly Derived From Fork Outcome
When a market is resolved the ClaimTradingProceeds contract allows an address to redeem any shares in the market for the payout amount designated by the winning payout for the market. In nearly all cases this array of payouts is the one associated with the last bond filled in the market. The one exception to this rule is in the case of a market which has resulted in a fork.

When a market forks its resolution is based on the result of REP migration to child universes, where each child universe is correlated with a particular payout array. Once the fork window ends or a majority of REP migrates to a Universe the winner is considered the universe with the most REP. All markets can then migrate to this winning universe along with their OI.

A bug exists in the V1 contracts however which makes the payout for the _forking market_ still simply look at the last filled bond in the market. This means that the winning payout for a forking market is at best arbitrary and at worst the result of manipulation in the order of bonds filled leading up to a fork.

The fix for this is to use branching logic when asking a market what the payout is for an outcome after market resolution for the case of a fork and will be implemented in the V2 contracts.

# Safe Harbor 
Any activities conducted in a manner consistent with this policy will be considered authorized conduct and we will not initiate legal action against you. If legal action is initiated by a third party against you in connection with activities conducted under this policy, we will take steps to make it known that your actions were conducted in compliance with this policy. 

Thank you for helping keep the Augur protocol safe!
