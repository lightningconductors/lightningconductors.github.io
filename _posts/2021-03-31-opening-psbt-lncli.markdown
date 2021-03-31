---
layout: post
title:  "Opening channels with PSBT in `lncli`"
date:   2021-03-31 16:20:42 +0100
categories: post
---

The channel opening on lightning network involves broadcasting an on-chain bitcoin transaction. To save space on the blockchain which implies also lower transaction fees it's possible to batch the transactions together. One of the ways to do it is to use the UTXO for the channel opening on the lightning network, that allows you to save the transaction fees even further as each of the channel can be used for an unlimited number of transactions.

I have already written an [article about PSBT channel opening from bitcoin-cli][psbt] which can be used when you have never used lightning network yet or wish to use the funds that you have within `bitcoin-cli`. Recently it happened to me, that two of the channels I had on the lighting network got closed by a peer and I had two UTXO available in the `lncli`. This article is an extension of the [PSBT channel opening from bitcoin-cli][psbt] to the way `lncli` can be used.

The workflow goes through the points of the following list:

   1. connect all relevant nodes
   2. start funding using openchannel --psbt to all nodes (10 minutes counter starts), all but the last with `--no_publish` option 
   3. create the funding PSBT transaction from `lncli` (explanation below)
	  - you can check it in the `bitcoin-cli decodepsbt <psbt>` (but perhaps later, now the counter is on)
   4. enter the PSBT transaction to each and every openchannel instance
   5. finalize the transaction in `lncli` (explanation below)
   6. enter the finalized transatction to each and every openchannel instance (make sure to enter it to the one without `--no_publish` option as the last one)
   7. the channels are batched for the opening

To remind you the point 2. is done using the command:
```
lncli openchannel --psbt <node_ID> --local_amt <amount> --no_publish
```
which gives you the address where the funds should be send within the `bitcoin-cli` example

```
...

Example with bitcoind:
	bitcoin-cli walletcreatefundedpsbt [] '[{"bcrt1qqzgeg5ty6hz3v2gckwkgyg8s0596zrvrjulgvly8efaeqe3uu60snfrfd5":0.01000000}]'

...
Paste the funded PSBT here to continue the funding flow.
Base64 encoded PSBT: 
```

The point 3. and 5. need further explanation as those are the ones, which I haven't use in the `bitcoin-cli` setup. In my case I have used the command:

```
lncli wallet psbt fund --conf_target 420 --inputs='[<txid_1>, ... , <txid_N>]' --outputs='{<addr_1>:<amount_1>,...,<addr_M>:<amount_M>}'
```

The `--inputs` options are used to select the inputs numbers that we would like to spend. We can find out which UTXOs we have available using `lncli listunspend` command. The `<addr_x>` are exactly those which were specified in the `lncli openchannel` example and for you need to make sure, that you insert the amounts in satoshi rather than bitcoins.

As I wanted to use all the funds in the inputs to be transferred to the channels I selected the output amount plus desired fee (fairly low as I don't care if the channels open in a few days) to be just a bit lower than the input amounts. Then I had a bit of trouble telling to `lncli` that it should not create yet another change output. I finally succeeded by setting the confirmation target to a large value using `--conf_target 420` which means that I target for channel to be opened in 420 blocks.

Once the command is finalized you obtain the `<PSBT>` string that can be entered to all the terminal windows with the prompt for the channel opening.

Finally, use the

```
lncli wallet psbt finalize <PSBT>
```
That gives you a `final_tx` hexadecimal string that is again entered to the channel opening prompts.  Just make sure, that the channel opening without `--no_publish` option goes as the last one.

If everything goes through smoothly, you are returned the TXID of the transaction opening your channels. However my experience was not that straight forward. As one of the peers I have tried to open the channel with had a larger minimal channel amount that I was planing to submit, my transaction failed before the final transaction could be broadcast. The channels with the other peers stayed pending. I had to abandon the pending channels with `lncli abandonchannel` that is only available in dev build of `lncli` (so I had to recompile from the source using `make tags=dev && make install tags=dev`. Before I could re-initiate the channel opening as specified in the [lnd git issue][gitissue].

[psbt]: /channels/2020/12/10/opening-psbt.html
[lnd git issue]: https://github.com/lightningnetwork/lnd/issues/5081
