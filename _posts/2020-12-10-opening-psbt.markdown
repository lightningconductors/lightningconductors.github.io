---
layout: post
title:  "Opening channels with PSBT"
date:   2020-12-10 19:00:00 +0100
categories: channels
---

# Opening Lightning Channels in LND

This article describes how to open multiple channels with a single on-chain transaction using `lnd` and `bitcoin-cli`.

This article was inspired by a more detailed [guggero article][guggero].

## Introduction

Lightning channels are backed by bitcoin locked in a special account called multisig. In general multisig account allows to spend the funds to a number (N) possible signees, but only if the condition that at least a threshold number (M) of them agree and sign the spending transaction. Such an account is called M-of-N multisig. In lightning channels we use 2-of-2 multisig accounts.

The initial channel negotiation and opening has to satisfy the following protocol. The peers agree on the 2-of-2 multisig account to deposit the funds to. Before the funds are deposited both peers sign the spending transaction with an agreed output balances (typically all the funds go to the party that opens the channel, although part of it can be pushed to the other side). Then the funds are send to the channel in an opening transaction. Sending a transaction on lightning network then means updating the balance transaction and invalidating the previous one (by revealing a punishment key).

The channel opening as currently handled by the lightning client has two mayor limitations. First, the user has to deposit funds to the lightning wallet before being able to open the channel. Second, it is only capable to open a single channel by each transaction. Fortunately, the developers have addressed those limitations by allowing Partially Signed Bitcoin Transactions (PSBT) as proposed by [BIP174][bip174].

## The Procedure

### Get Funds to Bitcoind

Newer versions of lnd will allow PSBT in channel opening from the lnd wallet directly. Even then the user will still need to transfer the funds to the lnd. So in this tutorial we will use the bitcoin-cli of bitcoind. If you use a different wallet to manage your bitcoin such as SPV wallet (e.g. Electrum), you can transfer the private key of the coin you would like to spend using:
```
bitcoin-cli importprivkey "privkey"
```

It takes a while for the wallet balance to be updated as the daemon needs to rescan the blockchain for the transactions belonging to the key. Once the rescan is completed can prepare the PSBT for the lightning wallet.

### Initiating Channel Opening with PSBT

During the procedure you need to be quick, as the time-window for the channel opening is limited to 10 minutes by lnd and other lightning wallets.

First you need to connect to all the nodes you would like to open the channel with by running `lncli connect <node_ID>@<node_IP>:<port>`.

To initiate the channel opening run:
```
lncli openchannel --psbt <node_ID> --no_publish
```
where the `<node_ID>` is the node you would like to open the channel with. In another terminal run the command for all the nodes you are opening the channel with. For the last node remove the `--no_publish` option, but make sure you finalise it as the last one, otherwise you might loose your funds.

The channel initialisation will give you the following output:

```
Starting PSBT funding flow with pending channel ID 10fd21de74f87b59940890f61c9d8541acb3426a610d06b36b53cfe7d3ec8dae.
PSBT funding initiated with peer 0224472d59751c64d0fce2822ce5fa9ce8a349b3e8081850b9cc1c2bc60bd7b496.
Please create a PSBT that sends 0.01 BTC (1000000 satoshi) to the funding address bcrt1qqzgeg5ty6hz3v2gckwkgyg8s0596zrvrjulgvly8efaeqe3uu60snfrfd5.

Note: The whole process should be completed within 10 minutes, otherwise there
is a risk of the remote node timing out and canceling the funding process.

Example with bitcoind:
	bitcoin-cli walletcreatefundedpsbt [] '[{"bcrt1qqzgeg5ty6hz3v2gckwkgyg8s0596zrvrjulgvly8efaeqe3uu60snfrfd5":0.01000000}]'

If you are using a wallet that can fund a PSBT directly (currently not possible
with bitcoind), you can use this PSBT that contains the same address and amount:
cHNidP8BADUCAAAAAAFAQg8AAAAAACIAIACRlFFk1cUWKRizrIIg8H0LoQ2Dlz6GfIfKe5BmPOafAAAAAAAA

!!! WARNING !!!
DO NOT PUBLISH the finished transaction by yourself or with another tool.
lnd MUST publish it in the proper funding flow order OR THE FUNDS CAN BE LOST!

Paste the funded PSBT here to continue the funding flow.
Base64 encoded PSBT: 
```

The lncli prompt will wait for your input. First you need to enter unsigned PSBT and then finalised and signed transaction.

### Creating PSBT

The most important part of the output from the psbt channel opening is the address for the channel opening, which is to be entered to the bitcoin-cli. In the second bracket enter all the addresses from the channels you would like to open, e.g.
```
	bitcoin-cli walletcreatefundedpsbt [] '[{"bcrt1qqzgeg5ty6hz3v2gckwkgyg8s0596zrvrjulgvly8efaeqe3uu60snfrfd5":0.01000000},
{"bcrt1qcghp2v8mqwne024kdf433khfp6cmxj4ckrvjzm84ju3wq2f5qz5qgw9wrc":0.01000000}	
...
	]'
```
You can also provide additional settings such as the txid (coin) you would like to spend, or the fee you would like to pay for the bitcoin transaction (choose lower fee if you don't mind the channel will take longer to open). See `bitcoin-cli walletcreatefundedpsbt help` for the full list of options.

Once you compose the command, run it and you should get something like this:

```
{
  "psbt": "cHNidP8BAH0CAAAAAa5BhzIb7Njai9cwTe37Hck4kuKnQJxsFdrQ8FsY11wjAAAAAAD+////AmdPJncAAAAAFgAULtjKcAjYUFS5LKnHqfQ2NMpxqmFAQg8AAAAAACIAIMIuFTD7A6eXqrZqaxja6Q6xs0q4sNkhbPWXIuApNACoAAAAAAABAHICAAAAAf2D5ppfnlgQzc6pDQlX4WKxLKzMZ/hdlEo2Z6XY4KGqAQAAAAD+////AkCSNXcAAAAAFgAUtcUA8Zya4/zSIdk8eHFF/KLy3U8Aypo7AAAAABepFFwYPL/mdT2Y6yhgZofwh2GEi8Ksh2UAAAABAR9AkjV3AAAAABYAFLXFAPGcmuP80iHZPHhxRfyi8t1PIgYDDsOKSmdsMV0pixM1tLvhbr+G5fjkaTUgn9VdpXnuilkQPAVsiQAAAIABAACAAgAAgAAiAgJF+RRsaFxKsByy10s8LimWEPy84i3wECW3Gcg0/Pp+0RA8BWyJAAAAgAEAAIAEAACAAAA=",
  "fee": 0.00000153,
  "changepos": 0
}

```

Copy the string in psbt paste it and press enter in all the lncli prompts for the channel opening. After that lncli will reply with the following output:

```
PSBT verified by lnd, please continue the funding flow by signing the PSBT by
all required parties/devices. Once the transaction is fully signed, paste it
again here either in base64 PSBT or hex encoded raw wire TX format.

Signed base64 encoded PSBT or hex encoded raw wire TX: 
```
Again waiting for your interaction.

### Signing the PSBT

To sign the transaction run the following command

```
bitcoin-cli walletprocesspsbt cHN...CAAAA=
```
(where the psbt string from the  `walletecreatefundedpsbt` is shortened), which gives
```
{
  "psbt": "cHNidP8BAH0CAAAAAa5BhzIb7Njai9cwTe37Hck4kuKnQJxsFdrQ8FsY11wjAAAAAAD+////AmdPJncAAAAAFgAUA212BZdE6Bc8pbXTaY5D7mDi1HhAQg8AAAAAACIAIJiIqp9BjRWtK5VWuDq0vC0S18juvccCJ/mH5yEPEnYgAAAAAAABAHICAAAAAf2D5ppfnlgQzc6pDQlX4WKxLKzMZ/hdlEo2Z6XY4KGqAQAAAAD+////AkCSNXcAAAAAFgAUtcUA8Zya4/zSIdk8eHFF/KLy3U8Aypo7AAAAABepFFwYPL/mdT2Y6yhgZofwh2GEi8Ksh2UAAAABAR9AkjV3AAAAABYAFLXFAPGcmuP80iHZPHhxRfyi8t1PAQhrAkcwRAIgDY2oqQWI6BCOleqUhNAnCuvs/VoNiQowtbJxKeSv4JgCICBfDcWcuqgyKW7Ov5dhH7AQn3hSvY2H//TjXNpxvounASEDDsOKSmdsMV0pixM1tLvhbr+G5fjkaTUgn9VdpXnuilkAIgIDxW9yaHPXaP3j0jHW4O9Gmu21hEfqqzJFo53VkOVCRzEQPAVsiQAAAIABAACABQAAgAAA",
  "complete": true
}
```
Again, copy the psbt string and paste it into all the lncli prompts. *Important* make sure you paste it into the prompt you started without `--no_publish` option as the last one. After you paste it there, the transaction is broadcast and after it gets confirmed (which might take hours or days if you payed low transaction fee), you can use all your channels.

[bip174]: https://github.com/bitcoin/bips/blob/master/bip-0174.mediawiki
[guggero]: https://github.com/guggero/lnd/blob/84dfed3fe2d28ceda343944874ab47fb57b73515/docs/psbt.md
