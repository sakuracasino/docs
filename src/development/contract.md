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

## Contracts

We have a few contracts in this repo:

* `Roulete.sol`, which contains all the contract code.
* `DAIMock.sol` is a test DAI mock used for the local network and the Kovan network.
* *Other Mocks* like `LinkTokenMock.sol`, `VRFCoordinatorMock.sol`, `chainlink-mocks` are only for testing and local env.

## Roulette.sol walkthrough
### Chainlink VRF

We use [Chainlink VRF](https://docs.chain.link/docs/chainlink-vrf/) for generating random numbers that resolve the rolls.

The roulette contract it's based on the [VRFConsumerBase example](https://docs.chain.link/docs/get-a-random-number/) in Chainlink's documentation.

Here's the code structure from the contract that it's relevant to *Chainlink VRF*

```solidity
contract Roulette is VRFConsumerBase, ERC20, Ownable {

    // Chainlink VRF Data
    bytes32 internal keyHash;
    uint256 internal fee;
    event RequestedRandomness(bytes32 requestId);

    /**
     * Contract's constructor
     * @param _bet_token address of the token used for bets and liquidity
     * @param _vrfCoordinator address of Chainlink's VRFCoordinator contract
     * @param _link address of the LINK token
     * @param _keyHash public key of Chainlink's VRF
     * @param _fee fee to be paid in LINK to Chainlink's VRF
     */
    constructor(
        address _bet_token,
        address _vrfCoordinator,
        address _link,
        bytes32 _keyHash,
        uint _fee
    )  ERC20("SAKURA_V1", "SV1") VRFConsumerBase(_vrfCoordinator, _link) public {}

    /**
     * Creates a randomness request for Chainlink VRF
     * @param userProvidedSeed random number seed for the VRF
     * @return requestId id of the created randomness request
     */
    function getRandomNumber(uint256 userProvidedSeed) private returns (bytes32 requestId) {}

    /**
     * Randomness fulfillment to be called by the VRF Coordinator once a request is resolved
     * This function makes the expected payout to the user
     * @param requestId id of the resolved request
     * @param randomness generated random number
     */
    function fulfillRandomness(bytes32 requestId, uint256 randomness) internal override {}
}
```
The contract calls `getRandomNumber` function on every roll request, with a seed provided by the user. Then, `fulfillRandomness` gets called by the `VRFCoordinator` and it performs the logic to decided if the user won or not.

### Owner role

The contract's **owner** only has (a limited) control in two parts: fees and maximum bet limits.

#### VRF fees

VRF fees need to be paid in [LINK](https://coinmarketcap.com/currencies/chainlink/). So the contract charges a fee set by the **owner** (in the *bet token*) for each roll. These fees go to the `collected_fees` variable and can be withdrawn by the **owner**.

```solidity
uint256 public collected_fees = 0;

/**
  * Withdraws the collected fees
  */
function withdrawFees() external onlyOwner {
    uint256 _collected_fees = collected_fees;
    collected_fees = 0;
    IERC20(bet_token).transfer(owner(), _collected_fees);
}
```

The owner then buys *LINK* with those fees and fund the contract with it. This should not happen very often because LINK fees on the Matic network very low.

!!! Automated LINK swap
A proposal to get rid of this owner role is to automatically trade the `collected_fees` amount for LINK by calling the QuickSwap contract itself after a certain threshold is collected.

The interesting part is that it doesn't need any contract migration. We could just move the ownership to a contract address that does that automatically.
!!!

#### Max bet limit

There's also the concept of a max bet. Currently it's fixed at `$1`, but it can't be more that `0.27%` of the current liquidity (to prevent max-bet setting abuse). That percentage is managed by the `minLiquidityMultiplier` variable.

This is a temporary measure for the current low liquidity. We plan to change it to *infinity* and only use a percentage of current liquidity, but we don't know which should be the proper amount yet.

### Bet token

The contract works with any ERC-20 token as a `_bet_token` parameter in the constructor.

The current deployed version uses `DAI`, for which the contract has a special case that allows to use the [permit function](https://github.com/makerdao/developerguides/blob/master/dai/how-to-use-permit-function/how-to-use-permit-function.md#permit). DAI's permit function allows you to spend DAI without the need of an "approval" transaction for the tokens, it uses a signature instead. The signature parameters are available in `rollBets` and `addLiquidity` methods, they also have the version without those parameters, but those are for when you're using tokens that don't have this capability, like USDC or USDT.

!!!success Multiple tokens

For supporting different tokens we need to deploy one different contract for each bet token. This is a planned feature we have ou our [roadmap](/roadmap).

!!!

### Liquidity management

#### Locked Liquidity

When a user makes a bet, we need to lock some part of the liquidity in case it wins. The user can win at most `x36` of the betted amount. We lock that maximum amount the user can win until the roll is resolved. The total liquidity locked at the moment is tracked by the `locked_liquidity` variable.

#### Shares printing

Liquidity shares are represented by the [SV1](https://kovan.etherscan.io/token/0x4d709c0715c03536c3599ed26b48c2332e69b01a) token. When you provide liquidity, we mint tokens for you and if you withdraw liquidity, we burn them.

For calculating liquidity take these into account:

For the first deposit, we mint `BASE_SHARES` per each deposited *token satoshi* (10^10 per each satoshi).
Then, we print `current_shares / (current_liquidity + locked_liquidity)` per each deposited *token satoshi*. `current_shares` is the circulating supply of SV1. `current_liquidity + locked_liquidity` is the amount of the bet token that is currently deposited.

To prevent problems for not having floating-point, the formula for how many shares need to be printed is `(added_liquidity * current_shares) / (current_liquidity + locked_liquidity)`.

### Running rolls

The function `rollBets` is the function that you need to call to run your bets. It will do the following:

1. Check that the total amount of your bets stays whithin the bet limits
2. Collect your bet tokens along with the fee
3. Lock the max amount you can win from the withdrawable liquidity
4. Request *Chainlink VRF* for a random number
5. Save your bet data and requestId.

That's it for the rollRequest transaction. Later, when the *VRFCoordinator* responds, it will call the `fulfillRandomness` function, there:

6. Check that resquest is not already resolved
7. Unlocks the locked amount, closes the request id
8. Computes the total amount you receive from the roll
9. If needed, it sends you won your amount

#### Roll redeem

There's a `redeem` function for the almost impossible case that *Chainlink's VRF* doesn't respond within two hours. You can redeem the amount you have bet in that request. *Ideally, this function should never be used.*

## Testing environment

You can run `make test` for running all the tests. We use the truffle test runner but you need to have both a running local gananche (`make run`) and no `vrfsigner` active.

!!! .mnemonic

There's a `.mnemonic` file with the mnemonic used for the ganache wallets and the tests. You can change this mnemonic if you want for your custom one.

!!!

Test are done in *JavaScript* instead of *Solidity*. There's only one test file, `roulette.test.js` with all the tests for the Roulette contract.


**Test libs** are in the `test/libs/` folder. They contain all the *permit logic*, *bignumber math* and *wallet logic*.

As a general rule, the test file should only use `daiMockInteractor` and `rouletteInteractor` for interacting with the contract.

Feel free to add any test you want and make a Pull Request on github. The more tests we have the better. 

## NPM package and UI

The github's contract repository is associated with a [npm package](https://www.npmjs.com/package/@sakuracasino/roulette-contract).

The npm package uses data from `networks.js` and exposes the contract's `ABI` to be used for the frotnend

For connecting the contract to an UI, simply install that npm package in your node environment.