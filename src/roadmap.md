---
label: Roadmap
icon: rocket
order: 1100
---
# Roadmap

Sakura Casino is in beta version for the roulette contract. We're planning to futher improve the contract and the number of games available.

We're still early in the development stages. So feel free to join us in our [Discord Server](https://discord.gg/DHux5uEvrJ) if you want to get the latest updates or contribute to the project.

!!!success Looking for devs

Currently, the team only consists of one developer (me :wave:), but I'm open to expand the team and welcome any pull requests on github.

My goal is to make the project successful, but also profitable in order to get funds for its development.

Look for me in discord as `index`.

!!!

## Planned features

This is a list of planned improvements to the *Sakura Casino's ecosystem*. Some of these may not be implemented or may suffer modifications. Unfortunaly, we don't have an expected release date for the features, but we hope to have a working casino with many games by the end of the year.

We're currently dicussing these ideas, so feel free to share your thoughts or new ideas in our discord server.

### Roll queue

The basic idea is to create a queue of all the rolls in a centain timeframe (like 5 minutes), the make one single VRF request every `5 minutes` to resolve all the pending requests. This can be done by mixing the *VRF result* with ordered numbers, like how is [done here](https://docs.chain.link/docs/get-a-random-number/#making-the-most-out-of-vrf). I have not prove how much *randomness entropy* is lost for using 100 or 1000 numbers, but I assume it's very low.

There are a few reasons for this update:

* We can save a lot in `LINK` fees. This very valuable if we use the *Matic network* (which is the reason why I dropped this option for the initial release), but if we want to migrate the contract to *BSC* or even *ETH 2.0*, this would be required feature. I would suggest to not speculate for other alternativas like BAND or hope fees are lowered in the future.

* Avoid VRF bottleneck: this shouldn't be too much of a problem if the VRF Coordinator is scalable. But we might have some issues in the future if we spam the contract and nodes too much.

### Mutiple bet tokens

The contract allows for any type of ERC-20 token as a bet token. The problem is that you can only chose one on the contract's contruction. What we can do is to create a different contract instance for each token; then, we will need to update the UI to switch between contracts.

## Multiple games scaling

We want to expand our casino platform to more games, not only roulette. I think we can divide tham in two areas: simpler random games (like slot machines, blackjack, dice) and with multiple users like poker.

### Sakura Casino token

For having multiple games, we might want to consider having a single token for all of them.

### Governance and airdrops
*To be completed*
### Shared random
*To be completed*
### Multiple user games
*To be completed*
