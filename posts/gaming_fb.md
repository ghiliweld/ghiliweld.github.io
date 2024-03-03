After following the MEV space for a while, I wanted to try my hand at building a searcher bot during my winter break from school. I had an idea for an interesting sandwiching strategy (might write about it later), but since my bot would be using flashbots I'd have to find a way to win the flashbots auction while still maximizing profits. Sandwiching is a competitive field and so vying for the same sandwiching opportunity as countless other searchers would force me to give away profits to guarantee winning the auction. You walk away with less, but at least it's something.

But what if there was a way to game the auctions so that you can secure a spot at the top of a block *without* giving away most of your profit? It's like having your cake and eating it too, but the cake is money (loads of it). My attempt at doing so is what this post is about. Althought it didn't work, I still think it wasn't a terrible idea and I learned something new about geth in the end.

## Some Background

For my searchers already familiar with flashbots, recall that bundles are sorted by effective gas price (`mevGasPrice` in mev-geth, i may also refer to it as "score"). Searchers aiming for the same opportunity in a block have to outbid other searchers for it, leaving barely any profit for themselves, since only one searcher can profit from the opportunity.

Recall some properties about flashbots bundles and the auction:
- `mevGasPrice := [delta_coinbase + (gasUsed * gasPrice)] / gasUsed` where `delta_coinbase` is the amount directly sent to the miner's coinbase address. (note: it's a really a sum of `gasUsed * gasPrice` across all txs in a bundle, but this definition is simpler for what I want to show)
- if we don't make any direct payments to the miner (i.e. `delta_coinbase = 0`) then `mevGasPrice = gasPrice`.
- if any tx in a bundle reverts then the bundle won't land onchain *unless* the hash of the tx in question is included in the `revertingTxHashes` field of our bundle request.

We can look at the effectice gas price as a measure of "profit per compute" for the miner, and so to maximize revenue for the miner flashbots prioritizes bundles that pay more per gas used. Making a direct payment to the miner requires additional gas and so it's more economical to omit the direct payment and pay by upping the gas price on your swaps (or wtv txs your strat entails).

If we have to up the gas price on our txs to ensure we win the opportunity, then we need to lower our gas used in order to bank more profit. Hence why you've probably noticed your favourite 0xAnimeFemboy mev account on the tl shilling yul+. Writing the most gas efficient contract means you can safely up your gas price which means easy $$$.

I'm lazy and I only had about 2 weeks until I started school again, so I wasn't about to become an optimizoooor. Instead, I was gonna try gaming the auction. Someone already gamed the auction before, where they [reduced their gas price after their bundle got merged](https://twitter.com/bertcmiller/status/1407305924600029189). I was gonna try using way less gas than flashbots would expect.

## The Strategy

The strat entailed making use of this fun little fact about gas limits:
> [I]f you put a gas limit of 50,000 for a simple ETH transfer, the EVM would consume 21,000, and you would get back the remaining 29,000. However, if you specify too little gas, for example, a gas limit of 20,000 for a simple ETH transfer, the EVM will consume your 20,000 gas units attempting to fulfill the transaction, but it will not complete. The EVM then reverts any changes, but since the miner has already done 20k gas units worth of work, that gas is consumed.

from [ethereum.org](https://ethereum.org/en/developers/docs/gas/#what-is-gas-limit)

Note that the definition of the effective gas price didn't require that any of our smart contract txs pay the bribe to the miner. Say we included an eth transfer in our bundle that handled our bribe, and that we set the gas limit to `1 gas` and the gas price to `100k gwei/gas`. We'd need to allow this tx to revert in our bundle (via `revertingTxHashes`) since any tx with a gas limit lower than 21k gas would revert. Our bundle could then land onchain, revert, and only consume 1 gas in doing so. This will only cost us `1 gas * 100k gwei/gas = 100k gwei` and our bundle's score would be boosted by `100k` which guarantees us first priority in the auction by a large margin. We can then safely set the gas price on our swaps to whatever the `base_fee` was that block since we don't wanna compete on gas price with high gas txs.

Those familiar with geth have probably noticed the flaw in this strategy by now and already know where this is going, but I won't spoil the (unfortunate) surprise just yet.

## The Attempt

Before my attempt, I needed to check if this would be possible in mev-geth. So I took a look at the [`computeBundleGas`](https://github.com/flashbots/mev-geth/blob/master/miner/worker.go#L1414) method, which is what returns our bundle's score (`mevGasPrice`). These lines in particular were of interest.

```go
receipt, err := core.ApplyTransaction(w.chainConfig, w.chain, &w.coinbase, gasPool, state, header, tx, &tempGasUsed, *w.chain.GetVMConfig())
if err != nil {
    return simulatedBundle{}, err
}
if receipt.Status == types.ReceiptStatusFailed && !containsHash(bundle.RevertingTxHashes, receipt.TxHash) {
    return simulatedBundle{}, errors.New("failed tx")
}
```

The first line applies the tx and returns a receipt and any errors encoutered. If the receipt showed the tx failed and reverted and wasn't included in our bundle's `revertingTxHashes` then we would throw an error and reject the bundle. But if we did include our tx in the `revertingTxHashes` then our method would proceed as normal and add our `gasUsed` and `gasPrice` to the tally, gaming the flashbots auction for the low. I thought the `err` variable was interesting but didn't give it much thought at first. That proved to be a fatal mistake later on.

So I get to testing, submiting my bundles with this lil hack thinking it's gonna go through. That's when I get hit with an `intrinsic gas too low` error saying I need to supply 21k gas instead of just 1 gas. After some digging I found out this wasn't a flashbots error but rather an [error that geth itself was throwing](https://github.com/ethereum/go-ethereum/blob/master/core/state_transition.go#L298) (coming from `core.ApplyTransaction`). Turns out geth *won't even broadcast* a tx destined to fail at all, which makes sense since that's pointless compute being spent by the miner for no real reason. Unfortunately, my strategy depended on the fact that we were intentionally letting a tx land onchain so it would fail, and geth completely foiled those plans.

---

Like I said in the beginning, this wasn't my worst idea ever. I just wasn't expecting geth to intervene like that when I saw on ethereum's website that any gas under 21k is consumed whether the tx reverts or not. I later realized that that was describing the EVM's behaviour, not the client's (geth) which was free to implement any checks beforehand. Unfortunate.

Edit: Apparently this is standard client behaviour as described in section 6 of the [yellowpaper](https://ethereum.github.io/yellowpaper/paper.pdf) and not unique to geth (s/o to the homie [@0xalpharush](https://twitter.com/0xalpharush) for pointing this out). But that was in the Berlin version of the yellowpaper, so now I'm still not sure if the official site was completely wrong or just outdated. That's on me though, I should've just read the code for myself first instead of going off the website.

I'm still not interested in being an optimizooor despite not being able to game the auction, and I learned just how involved building a competitive searcher bot is over my winter break. It's definitely not something I can slap together in 2 weeks like I originally planned. I'll see how I end up going about this mev shit but I'll have to take my time with it if I wanna be competitive.

Thank you for reading, and see you in the mempool :)
