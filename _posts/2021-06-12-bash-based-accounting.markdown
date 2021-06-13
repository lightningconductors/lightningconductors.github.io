---
layout: post
title:  "Bash Based Accounting"
date:  Sat 12 Jun 19:57:43 UTC 2021
categories: post
---

In this article I explain a simple account system in GNU operating system based on `awk` and `bash` utilities. Using this system you can manage the debt of you to your friends and family and viceversa.

## System setup

Let's get practical and open your first account. For that you need to define one alias and two bash functions in an `~/.accounting` file and create `accounting` directory (`mkdir accounting`). Then place the following content there:

```bash
alias ds='date +%F@%H-%M' # date stamp

ACCOUNT_SATS_PERSON=~/accounting/person.txt
saldo() {
    echo `ds` $@ >> $ACCOUNT_SATS_PERSON
    echo New transaction: $@
    echo Saldo: `awk '{sum+=$2}END{print sum}' $ACCOUNT_SATS_PERSON`
}

zustatek() {
    echo "saldo       date      amount description "
    awk '{sum+=$2;print sum, $0;}' $ACCOUNT_SATS_PERSON
}
```

This can be sourced to bash using `source ~/.accounting` (also, if you are happy with that you can just add the sourcing at the end of `~/.bashrc`).

The usage is `saldo <amount> <description>` where amount is a single number with a specific sign convention, e.g. minus when I borow money and plus when I lend to other person.

In case you handle accounts for more than one person, you can obviously create similar functions for each of them.

## Example spending

So now let's assume that I have got a coffee for 3024 payed by the other person, then I payed membership fee for a car rental company that the person wish to contribute 33442 sats. We went for a trip and the other bought petrol to which I should contribute 54622 sats. I have added those expenses to the balance sheet with the correct sign using `saldo` function.

Finally, I add the price for the car rental to demonstrate how the output of the command `saldo` looks like. What is important to note is that the first argument after the command is the amount in satoshi.

```bash
 $  saldo 163232 car rental
New transaction: 163232 car rental
Saldo: 139028
```

This will add another entry to the `~/accounting/person.txt` file (notice the last line). This file is a mere table of the account with date, amount of change and a description and does not tell you the final saldo.

```
2021-06-12@20-24 -3024 payment for coffee
2021-06-12@20-24 33442 membership fee
2021-06-12@20-25 -54622 contribution for petrol
2021-06-12@20-27 163232 car rental
```

Then you can run the balance command by the `zustatek` command 

```bash
 $  zustatek
saldo       date      amount description 
-3024 2021-06-12@20-24 -3024 payment for coffee
30418 2021-06-12@20-24 33442 membership fee
-24204 2021-06-12@20-25 -54622 payment for petrol
139028 2021-06-12@20-27 163232 car rental
```
Where the first number always shows the current balance of the account and the rest is identical with the `$ACCOUNT_SATS_PERSON` file.

## Graph your spending

The spending you do during a certain time can be shown in the graph using an old good `gnuplot`. Here I provide a simple script that can be saved as `~/accounting/zustatek.plt`

```gnuplot
now = "`date +%F@%H-%M`"
from="2021-06-01@00:00"
# from = "`date +%F@%H-%M -d 'now - 2 months'`"
filename = "~/accounting/person.txt"

set xdata time
set format x "%Y/%m"
set timefmt "%Y-%m-%d@%H-%M"
set xrange [from:now]
set grid

a=0
b=0
suma(x)=(a=a+x,a)
sumb(x)=(b=b+x,b)

high(x) = x > 0 ? b+x : b
low(x)  = x < 0 ? b+x : b

plot filename using 1:2 w i t 'added' , '' u 1:(suma($2)) w l t 'saldo'
```

which can be visualised by running `gnuplot --perist ~/accounting/zustatek.plt` and shows both the intervals for changes and current balance (although improvements in the plotting would be welcome, e.g. adding finantial bars instead of lines).

![GNUploot balance](/assets/accounting/accounting.png)

## Final thoughts

Of course the accounting system works the same for any other currency besides bitcoin. You can even account in fiat, butter or anything else you decide with the friend that you keep the account with.

The limitation of the system is that it is kept on the computer of a single person, hence the updates have to be shared and agreed with the other peer by some other system, e.g. email, instant messaging.

The accounting system can be used as management of a single underlying bitcoin account within family and close friends, but then the liquidity of the account keeper remains to be established and tested on [regular basis][proof_of_keys].

In case that something happens (extreme example death of the debtor) the heirs can assess the legitimacy of the claims by the confirmation in the instant messaging program that provides signatures to the messages such as [Signal][signal].

[proof_of_keys]: https://www.proofofkeys.com/
[signal]: https://signal.org/

## Ad

Thanks for reading this article. If you liked it please share, [subscribe to RSS feed][RSS] and to recognise its value consider a [donation]. Also, feel free to [get in touch][mailme] if you have any advice for improvements or suggestions for further research.

[RSS]: https://blog.lightningconductors.net/feed.xml
[donation]: https://btcpay.lightningconductors.net/api/v1/invoices?storeId=FFPzRyoNZHuENk4uyNSehkscnDsRLWZpLedmCzipt9tU&checkoutDesc=Thanks+for+donating+to+lightningconductors.net&price=42&currency=sats

[mailme]: mailto:info@lightningconductors.net


