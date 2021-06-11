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

### Liquidity management



### Running rolls
#### Roll redeem

## Testing environment
## NPM package
### Linking it with the UI