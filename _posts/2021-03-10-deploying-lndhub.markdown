---
layout: post
title:  "Deploying LndHub"
date:   2021-03-10 16:20:42 UTC 2021
categories: admin lightning lndhub
---

This article describes the deployment and configuration of [LNDHub][lndhub]. LNDHub is a project that allows to create accounting system on top of lightning. The LNDHub portal can be accessed from lightning enabled wallets such as [BlueWallet][bluewallet] and [Zeus][zeus]. The underlying node is connected and provides liquidity for the transactions of the users of the system. 

To setup the system you need to download the source code of the project from it's github repository:

```
git clone https://github.com/BlueWallet/LndHub
```

and enter to it's repository `cd LndHub` copy required configuration files such as tls certificate of the node and admin macaroon to the working directory and setup the config. You will also need a database server redis installed on your machine for the data persistence.

Also, you need to have a database service Redis installed on the server. In the Readme.md the manual installation is described but I have managed to install under Debian distribution using the standard `apt` tool, without any further configuration needed. 

As the next step we need to copy the `tls.cert` and `admin.macaroon` to the LndHub data directory. After cloning the LndHub it can be launched with the `npm run` in the LndHub directory. Then the user can configure the server to connect from the port 3000.


## Configuration within BTCPayServer

In my case I use BTCPayServer docker implementation which has the files located in the following locations
```
/var/lib/docker/volumes/generated_lnd_bitcoin_datadir/_data/tls.cert
/var/lib/docker/volumes/generated_lnd_bitcoin_datadir/_data/admin.macaroon
```
Besides that we need to expose the ports of the lnd and bitcoind for the purposes of the LndHub. This can be done by creating the file
```
btcpayserver-docker/docker-compose-generator/docker-fragments/opt-bitcoind-listen.yml
```
with the content
```
version: "3"

services:
  bitcoind:
    ports:
    - "127.0.0.1:43782:43782"
    environment:
      BITCOIN_EXTRA_ARGS: deprecatedrpc=accounts
```

and a file
```
btcpayserver-docker/docker-compose-generator/docker-fragments/opt-lnd-listen.yml
```
with the content
```
services:
  lnd_bitcoin:
    ports:
    - "127.0.0.1:10009:10009"
```

After creating this snippets we run the command:
```
export BTCPAYGEN_ADDITIONAL_FRAGMENTS="$BTCPAYGEN_ADDITIONAL_FRAGMENTS;opt-bitcoind-listen.yml;opt-lnd-listen.yml"
```

stop and reconfigure the BTCPayServer as:
```
. btcpay-setup.sh -i
```

The configuration of my `config.js` file is

```
let config = {
  bitcoind: {
    rpc: 'http://btcrpc:btcpayserver4ever@localhost:43782/wallet/wallet.dat',
  },
  redis: {
    port: 6379,
    host: '127.0.0.1',
    family: 4,
    db: 0,
  },
  lnd: {
    url: '127.0.0.1:10009',
    password: '',
  },
};

if (process.env.CONFIG) {
  console.log('using config from env');
  config = JSON.parse(process.env.CONFIG);
}
```

which differs in the default ouptut in the port number of the bitcoind server.

## Accessing the LndHub data

I have tried to dig the data out of the database but haven't found any easy way to do it. The only think I have accomplished was to purge all the database with `rdcli flushall` command. I would like to be able to view and edit the database entries, but as for now I haven't found a way to do it.


## Configuring the Zeus and BlueWallet app

To access the LndHub we can also configure the smartphone app. For that the we can use Zeus or Bluewallet.

The for the wallet creation in the Zeus looks like this:

![Zeus: LndHub setup](/assets/LndHub/app.zeusln.zeus_lndhub_setup.jpg)
![Zeus: Account creation](/assets/LndHub/app.zeusln.zeus_account_creation.jpg)
![Zeus: Save config](/assets/LndHub/app.zeusln.zeus_save_config.jpg)
![Zeus: Wallet  interface](/assets/LndHub/app.zeusln.zeus_account_interface.jpg)

and for the BlueWallet the setup is following:

![BlueWallet: Account creation](/assets/LndHub/bluewallet.bluewallet_account_setup.jpg)
![BlueWallet: Wallet interface](/assets/LndHub/bluewallet.bluewallet_account_interface.jpg)



## Further work

For now I have not setup the SSL certificates for the server, so it's susceptible to MITM attack, which I don't care about, as my deployment was for educational purposes only at the moment.

A cool feature would be to link the LndHub with the BTCPayServer accounts and internal LND. This way the users would be able to accept lightning payments without the need to deploy and manage their own instance of lightning node. The LndHub would in this case serve as a third layer of Bitcoin, although it would remain custodial service and trust to the security of the server is required. It's then up to the each user preference how much funds they would leave on the server, and how much they would transfer to non-custodial solutions.

A similar custodial solution [LNBank][lnbank] is now under development into the BTCPayServer.

## Acknowledgement

Thanks to [Mario][tmiyc] for helpful discussion and developers of BlueWallet for assistance with the setup debugging.

[bluewallet]: https://bluewallet.io/
[zeus]: https://zeusln.app/
[lndhub]: https://github.com/bluewallet/lndhub
[lnbank]: https://github.com/dennisreimann/btcpayserver-lnbank
[tmiyc]: https://twitter.com/_TaxMeIfYouCan_
