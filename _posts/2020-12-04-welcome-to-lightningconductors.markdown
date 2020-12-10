---
layout: post
author: Zebra
title:  "Welcome to LightningConductors!"
date:   2020-12-04 11:29:45 +0100
categories: lightning
---

Bitcoin and other cryptocurrencies have gained mainstream media attention a while ago. They batch the payments in so called blocks saved into a distributed ledger of transactions called blockchain. 

To maintain their decentralised nature it is necessary to limit the size of the new blocks. However, this leads to another problem -- a competition for the space in the blockchain. In the busy times the transaction fees grow expensive making the system impractical for small payments. Even then there is just not enough space for every transaction on the planet to be added to the blockchain. 

Lightning network is a system that aims to address this issue and allow to scale Bitcoin. It works based on so called payment channels between interconnected nodes. 

During the opening of a payment channel the participants deposit Bitcoin satoshi (basic accounting units of Bitcoin) that guarantee their intention to play according to the rules of the network. The payments allow the participants to update ownership of satoshi deposited within the channel. 

The payments between participants that lack a direct channel is resolved by finding a multi-hop path. During the payment the nodes on the path reveal a secret in exchange for the transacted value.

This way the lightning network brings private, affordable systems for even more bitcoin users and enables a whole new space for experimentation and development of novel technologies.

# Use Lightning Network

The simplest way to use lightning network is through one of smart-phone application. Some application offer full control of the keys in a trust-less setup, other keep custody of your funds, but offer a better user experience. 

The non-custodial solutions with smooth user experience are Breez Wallet or Phoenix Wallet. Both those applications only rely on a third party to provide data about the bitcoin transactions and network status.

A solution for maintaining the full bitcoin/lightning node and your you can refer to one of many articles how to [safely run a full node](https://degreesofzero.com/article/lightning-network-node-setup-backup-and-recovery.html).
