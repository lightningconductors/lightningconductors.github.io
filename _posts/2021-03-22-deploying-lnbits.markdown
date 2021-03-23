---
layout: post
title:  "Deploying LNbits"
date:   2021-03-22 16:20:42 UTC 2021
categories: post
---

Last time I have wrote about [LNDHub][lndhub_article] which is an application to manage user accounts on top of a single lightning node instance. [LNbits][lnbitsgit] is yet another next layer solution for the lightning network which offers a number of extensions that can work on top of lightning. In this article I will describe the deployment of LNbits.

## Basic installation

The installation is described on the [github of lnbits][lnbitsgit], however I had problems with installing the system on my machine directly, as there were some difficulties in creating virtual environment through `venv`. Eventually I have managed to install it using the `virtualenv` and activated it via `source venv/bin/activate`. In addition to the packages specified in the requirements I had to install `lndgrpc purerpc` libraries using `pip install lndgrpc purerpc` for the usage with `lnd`. Those packages require updated version of `python-setuptools` that can be installed via `pip` but it seems that the system wide installation took a preference. Removing the system wide installation with `apt remove python-setuptools` has solved the issue. My installation procedure then was

```
git clone https://github.com/lnbits/lnbits.git
cd lnbits/
virtualenv -p python3 venv
source ./venv/bin/activate
pip install -r requirements.txt
cp .env.example .env
quart assets
quart migrate
hypercorn -k trio --bind 0.0.0.0:5000 'lnbits.app:create_app()'
```

## Docker installation

As a way to avoid those issues and provide a reproducible build for other users I have also decided to use the `Dockerfile` in the lnbits repository. The first attempts have failed due to incorrect access rights of the lnbits folder which was solved by putting the SQLite database files in a separate directory owned by the user ID 1000 (as specified in the `Dockerfile`). 

```
git clone https://github.com/lnbits/lnbits.git
cd lnbits/
docker build -t lnbits .
```

## Configuring lntxbot backend

Lntxbot is yet another custodial service within Telegram messanger. It provides a way to send tips for your Telegram contacts without them necesarily having a lighting wallet. It is also possible to connect to lntxbot from outside Telegram and the LNbits developers have deployed it as the LNbits backend.

The lntxbot configuration string is obtained by `/lightningatm` message to the lntxbot in Telegram. This gives something like:
```
NDc2ZjZmNjQyMDc5NmY3NTIwNjc2Zjc0MjA2ODY1NzI2NTJlMjA0Mjc1NzQyMDc0Njg2OTczMjA2OTczMjA2ZTZmNzQyMDZkNzkyMDcwNjE3MzczNzc2ZjcyNjQyMDNiMjkwYQ==@https://lntxbot.bigsun.xyz
```
where the part before `@` is the `LNTXBOT_KEY` and after `@` is `LNTXBOT_API_ENDPOINT` that needs to be copied to the `.env` file. The `.env` file was copied from `.env.example` and modified. 

The docker container with lntxbot is then launched by:
```
docker run --detach --publish 5000:5000 --name lnbits --volume ${PWD}/.env:/app/.env --volume ${PWD}/data/:/app/data lnbits
```
I have submitted [a PR with the changes in the docker configuration][PR] to the LNbits developers and as for now they are pending for a review. 

## Configuring lnd-gRPC connection

As the lntxbot itself is a custodial service I have decided to use my own node in order to have a complete control of the funds. I have used ssh to tunnel the lnd ports to the localhost as
```
ssh -L 0.0.0.0:10009:localhost:10009 lndnode.onion
```
and made new directory for certificates `mkdir credentials` and copy the required credentials from the lndnode.onion there:
```
scp lndnode.onion:<lnd_home>/data/chain/bitcoin/simnet/{admin.macaroon,tls.cert} ./credentials
```
finally setup the `.env` file to reflect `LNBITS_BACKEND_WALLET_CLASS=LndWallet` and certificate and macaroon path.

After launching the lnbits
```
hypercorn -k trio --bind 0.0.0.0:5000 'lnbits.app:create_app()'
```
I was able to access the same user accounts that were present with lntxbot, only that the backend node now is the lnd itself. When running the command in docker, we also need to mount a directory with the lnd credentials and modify the configuration in `.env` file to reflect the paths to the certificates within docker.

## Configuring lnd-gRPC in Docker

Every docker container has their own IP address on docker network. I was trying to connect to my machine's external address to simulate this situation. At the first tries I couldn't connect to the server and the wallet status code was `UNAVAILABLE` even though the IP addresses and port mapping was correct. Finally, I have found [more information][zaphelp] about how this issue can be overcome. The problem was that the `tls.cert` for lnd is self-signed only for particular IP addresses and domains and my external IP adress was not on the list.

When configuring the container within docker we need to be careful about the domain name and IP addresses for which the TLS certificate is issued. This can be viewed using the `openssl` command. 
```
openssl x509 -text -noout -in credentials/tls.cert | grep DNS
```
which in my case gives the following list of DNS aliases:
```
DNS:3851cbfdefe0, DNS:localhost, DNS:lnd_bitcoin, DNS:unix, DNS:unixpacket, DNS:bufconn, IP Address:127.0.0.1, IP Address:0:0:0:0:0:0:0:1, IP Address:172.18.0.10
```
I have used self-signed lnd certificate that was generated for BTCPayServer docker instance in a container with a domain name `lnd_bitcoin`. So, I have changed the configuration in the `.env` file as
```
LND_GRPC_ENDPOINT=lnd_bitcoin
```

Finaly, I needed to interconnect with the BTCPayServer generated network by adding option `--network=generated_default` to `docker run` which results in 
```
docker run --detach --network=generated_default --publish 5000:5000 --name lnbits --volume ${PWD}/.env:/app/.env  --volume ${PWD}/data:/app/data  --volume ${PWD}/credentials:/app/credentials lnbits
```

## Further work

Now the lnbits is running on my domain under port 5000, however I is only under HTTP, which is not secure. One further task would be to enable secure HTTPS for it. This could also be added to BTCPayServer as an additional service similar to how BTCPay provides access to Ride the Lightning or Thunderhub.

Then it would also be great to be able to use the LndHub from the LNbits extension as a personal lightning node. This way the users of BTCPay could each connect to their own wallet which would allow for a greater flexibility and simpler user management of personal lightning nodes.

## Final thoughts

The lnbits is an useful custodial service that can simplify the usage of lightning network to the customers that choose the convenience before the security of their funds, as the server operator might disappear or the funds might get stolen. However, in case of small quantities this might be a reasonable risk to take.

If you are already operating a lightning node, this is a great way to provide lightning access to your friends and family. You should always remaind them of the risk that this option carries and in case of holding larger amount and advice for transfering the funds to a non-custodial service (such as [Phoenix wallet][phoenix]) or swap the funds onchain.

The lnbits offers a number of extensions that can be used for different purposes such as generating withdraw lnurls, offline stores, backend server for [bleskomat][bleskomat], paywalls and many more.

The lnbits is a very promising project with a large range of possible applications. Nevertheless it is still in beta, hence not ready for a serious business deployment.

## Ad

Thanks for reading this article. If you liked it please [subscribe to RSS feed][RSS] and/or [donate][donation]. Also, feel free to [get in touch][mailme] if you have any advice for improvements or suggestions for further research.


[lndhub_article]: /admin/lightning/lndhub/2021/03/10/deploying-lndhub.html
[lnbitsgit]: https://github.com/lnbits/lnbits/blob/master/docs/guide/installation.md
[PR]: https://github.com/lnbits/lnbits/pull/163
[zaphelp]: https://docs.zaphq.io/docs-desktop-lnd-configure
[phoenix]: https://phoenix.acinq.co/
[bleskomat]: https://www.bleskomat.com/

[RSS]: https://blog.lightningconductors.net/feed.xml
[donation]: https://btcpay.lightningconductors.net/api/v1/invoices?storeId=FFPzRyoNZHuENk4uyNSehkscnDsRLWZpLedmCzipt9tU&checkoutDesc=Thanks+for+donating+to+lightningconductors.net&price=42&currency=sats


[mailme]: mailto:info@lightningconductors.net
