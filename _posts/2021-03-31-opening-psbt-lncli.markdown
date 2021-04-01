---
layout: post
title:  "Opening channels with PSBT in `lncli`"
date:   2021-03-31 16:20:42 +0100
categories: post
---

The channel opening on lightning network involves broadcasting an on-chain bitcoin transaction assigning a transaction fee per byte of transaction size to the miners. It's possible to batch the transactions together to save space on the blockchain which also implies lower transaction fee. One of the ways to batch the channel opening is to use PSBT in which the inputs of transaction can be used for a large number of channels.

I have already written an [article about PSBT channel opening from bitcoin-cli][psbt] which can be used when you have never used lightning network yet or wish to use the funds that you have within `bitcoin-cli`. Recently, that two of the channels I had on the lighting network got closed by a peer and I had two UTXO available in the `lncli`. This article is an extension of the [PSBT channel opening from bitcoin-cli][psbt] to describe how `lncli` psbt can be made.

The workflow goes through the points of the following list:

   1. connect all relevant nodes
   2. start funding using openchannel --psbt to all nodes (10 minutes counter starts), all but the last with `--no_publish` option 
   3. create the funding PSBT transaction from `lncli` (explanation below)
	  - you can check it in the `bitcoin-cli decodepsbt <psbt>` (but perhaps later, now the time counter is on)
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
lncli wallet psbt fund --conf_target 420 --inputs='[<utxo_1>, ... , <utxo_N>]' --outputs='{<addr_1>:<amount_1>,...,<addr_M>:<amount_M>}'
```

The `--inputs` options are used to select the inputs numbers that we would like to spend. We can find out which UTXOs we have available using `lncli listunspend` command. The `<utxo_x>` are exactly those which were specified in the `lncli openchannel` example and for you need to make sure, that you insert the amounts in satoshi rather than bitcoins.

As I wanted to use all the funds in the inputs to be transferred to the channels I selected the output amount plus desired fee (fairly low as I don't care if the channels open in a few days) to be just a bit lower than the input amounts. Then I had a bit of trouble telling `lncli` that it should not create yet another output for the change, as I wanted all the funds to be used for the channel. I finally I overcame the issue by setting the confirmation target to a large value using `--conf_target 420` which means that I target for channel to be opened in 420 blocks.

Once the command is finalized you obtain the `<PSBT>` string that can be entered to all the terminal windows with the prompt for the channel opening.

Finally, use the

```
lncli wallet psbt finalize <PSBT>
```
That gives you a `final_tx` hexadecimal string that is again entered to the channel opening prompts.  Just make sure, that the channel opening without `--no_publish` option goes as the last one.

If everything goes through smoothly, you are returned the TXID of the transaction opening your channels. However my experience was not that straight forward. As one of the peers I have tried to open the channel with had a large minimal channel amount that I was planing to submit, my transaction failed before the final transaction could be broadcast. The channels with the other peers stayed pending. I had to abandon the pending channels with `lncli abandonchannel` that is only available in dev build of `lncli` (so I had to recompile from the source using `make tags=dev && make install tags=dev`. Before I could re-initiate the channel opening as specified in the [lnd git issue][gitissue].

[psbt]: /channels/2020/12/10/opening-psbt.html
[lnd git issue]: https://github.com/lightningnetwork/lnd/issues/5081

## Ad

Thanks for reading this article. If you liked it please share, [subscribe to RSS feed][RSS] and to recognise its value consider a [donation]. Also, feel free to [get in touch][mailme] if you have any advice for improvements or suggestions for further research.

[RSS]: https://blog.lightningconductors.net/feed.xml
[donation]: https://btcpay.lightningconductors.net/api/v1/invoices?storeId=FFPzRyoNZHuENk4uyNSehkscnDsRLWZpLedmCzipt9tU&checkoutDesc=Thanks+for+donating+to+lightningconductors.net&price=42&currency=sats

[mailme]: mailto:info@lightningconductors.net


