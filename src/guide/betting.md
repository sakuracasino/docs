---
label: Placing bets
icon: sync
order: 900
---
# Placing bets

!!!warning
SakuraCasino roulette is in beta. Our contract hasn't been audited yet. Please be sure to know the risks before betting or pooling.
!!!

![](../assets/bet_placer.png)

Once you're connected to SakuraCasino's roulette app. You'll be able to make bets.
The user interface is very simple; it consists of three parts:

1. A *bet history* showing the lastest bets from everyone and your previous bets.
2. A *bet placing area* with an interactive roulette layout to place bets. It also contains a *"Possible outcomes"* tab with the expected gain/loss for each possible result.
3. A *bet list* with all the bets you've placed for the next *Roll* to be played.

### Types of bet

As of any european roulette, you have multiple bet types.

* By **Number** (x36 if you win)
* By **Dozen** (x3 if you win)
* By **Column** (x3 if you win)
* By **Half** (x2 if you win)
* By **Color** (x2 if you win)
* By **Parity** (x2 if you win)

You can do any combination of these bets before rolling.

![](../assets/bet_example.png)

Select you desired bet by clicking on the layout. A popup will show up where you can input how much you want to bet and check how much you can earn.

### Checking your bets

![](../assets/total_outcome.png)

After you have placed your bets, they will appear on the left side. There you can see the details, the total amount and the maximum you can win.

If you click on the **Possible outcomes** tab, you can check how much you'd earn for each possible result.

### Rolling your bets

![](../assets/roll_dialog.png)

To roll your currently placed bets, you can click on the **Roll** button and it will present a popup with the final amount you have to pay: this includes all the bets, plus a small fee for the random number generation.

Since we're using the *Matic network*, fee are very low: only `$0.01` per roll, no matter the amount.

!!! Random Fee

Chainlink's random number fee is very low in the *Matic network*. In *Binance Smart Chain* is arround `$6.00` and in the *Ethereum network* is arround `$60.00`.

We're planning to deploy the roulette contract in *BSC* and *ETH* when fees are lowered.

!!!

#### Approve and roll

![](../assets/dai_sign.png)

There're two buttons for rolling your bets: `Approve` and `Roll bets!`.

![](../assets/roll_dialog_approved.png)

**Approve** will ask for a wallet signature that allows the roulette to spend your *DAI tokens*.

**Roll bets** will send the roll request to the roulette's contract. You have to pay a small *Matic* fee for executing this function.

!!! DAI's permit signature

A great feature of using *DAI* to pay bet is that the approving function is free from fees because it uses a wallet signature instead of a contract function.

In other ERC-20 tokens like *USDC* or *USDT* you have to make an additional transaction to approve the tokens. That makes interacting with the contract more expensive.

!!!

### Roll queue

![](../assets/rolling.png)

After sending a roll, you'll see a roulette animation which means that the roll was sent and it's waiting for the random number to arrive. It can take several seconds or even minutes, but it should be fast.
You can also see your pending rolls in the list on the right.

Nothing will happen if you close or refresh the page. Independenly of what you do, you'll later have a popup with the result number and how much earned or lost. In case you win some *DAI*, the funds will automatically be sent to your wallet.

![](../assets/result_dialog.png)