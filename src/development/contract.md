---
label: Contract Development
icon: note
order: 900
---
# Contract Development

The [contract's repository](https://github.com/sakuracasino/roulette-contract) is built using [truffle](https://www.trufflesuite.com/) and [ganache](https://www.trufflesuite.com/ganache).

You don't need to install any of these globally, they're implemented as *`npm` dependencies* of the projects.

To get started, follow the instructions in the `README` to run the project:
https://github.com/sakuracasino/roulette-contract#running-the-project

Remember that you have to run `make migrate` and `make vrfsigner` to have it completely working

!!! VRF Signer

The `vrf-signer` is just a watcher that fulfills the randomness requests with the provided seed. If you don't have it running, it will not resolve the roulette rolls.

!!!

Once you have the ganache server running, you can use the RPC URL `http://localhost:8545` and the Chain ID `1337` for connecting it to Metamask.

### Contracts

We have a few contracts in this repo:

* `Roulete.sol`, which contains all the contract code.
* `DAIMock.sol` is a test DAI mock used for the local network and the Kovan network.
* *Other Mocks* like `LinkTokenMock.sol`, `VRFCoordinatorMock.sol`, `chainlink-mocks` are only for testing and local env.

### Roulette.sol walkthrough
### Testing environment
### NPM package
#### Linking it with the UI