---
layout: post
title:  "LNbits Bounty Fundraising"
date:  Mon 29 Mar 16:20:30 UTC 2021
categories: post
---

I was inspired to write about the bounty fundraising system on the [fulmo lightning hacksprint] that happened this weekend. In one of the chat groups I came across a project to the [Sputn1ck `github-bounty` project][sputn1ckgit] which aims to fundraise the money to the github issues through lnd. As for now the `github-bounty` uses the `lnd` backend API in order to generate the invoices and count the amount of contributions.

One of the problem what many potential users of lightning network face is the relatively complicated setup and management of the lightning node. The LNbits project is a self-hosted custodial service that allows to delegate the solution to this issue to a trusted third party. Although this idea is somehow against the mindset of many bitcoiners. It is a risk that many users are willing to exchange for the convenience when dealing with smaller amount of money.

The LNbits system allows for creating custom made extension with their own API. At first I have thought that creating a new Bounty fundraising API would be a good idea and started to explore the possibility of such implementation. The LNbits is written in Python and uses Assynchronous Server Gateway Interface (ASGI) that serves the html with javascript code (in Vue) generated on the fly.

As I later found out, there is already a way of creating the fundraising system using a few existent LNbits extensions.

## Wallet Setup

The previous article I have written was about [deploying the LNbits on your server]. Other possibility is to use one of the existent deployments such as [lnbits.com]. 

When you hit the initial page you get the option to create your own wallet by giving it a name and click confirm.

![LNbits wallet setup](/assets/fundbounty/wallet_setup.png)

This will take you to your wallet interface where you can see the funds you are assigned in the providers server, can create invoices to receive funds from other lightning network users, or send funds to others.

Your wallet access credentials appear in the URL of the website, so you should make sure that you **never send the link to anyone else**, as they would be able to open the same webpage, and obtain the same rights as you have. Also, it is necessary that the address is only accessible through SSL encrypted secure HTTPS and not the legacy HTTP system, as it would make it vulnerable to the man in the middle attack.

![LNbits wallet interface](/assets/fundbounty/wallet_interface.png)

In the wallet interface you can create additional separate wallets that you can use for different purposes.

## Choosing the extensions

To create the bounty fundraising we are going to use three extensions. Those are:

+ Paywall -- extension aimed to create payment walls to fundraise for a web links, we are going to use it to rise money for the bounty
+ Support Tickets -- extension to create payed support tickets to prevent spam or earn for answering questions, we are going to provide the link for the users claiming the bounty
+ LNURLw -- extension to send withdraw links to bounty winners

In your wallet interface you can find the option to manage extension. When you click there, you enter to another page where you can enable those extensions for yourself. Then the extensions get activated and you can access to their interface through the link just under the list of your wallets:

![Claim form initiation](/assets/fundbounty/active_extensions.png)

## Paywall/Bounty fundraising setup

As mentioned above, we are going to abuse the Paywall extension for the bounty fundraising. So having enabled the extensions, you can enter the Paywall extension in the LNbits wallet interface. On the top of the page click on the "NEW PAYWALL" button and fill up the form. Select the wallet to which the fundraising should go and add a title and a description. The redirect URL and minimal amount fields are obligatory, however not particularly useful for our purposes. We can just add a link that will be shown after the donor has made a successful payment so ideally we could create something like a thank you page, or just put a link to the github issue we are fundraising for. We could also choose a minimal amount, but it makes sense to leave it on quite a low value, so that anyone can decide what amount they contribute with.

![Paywall initiation](/assets/fundbounty/paywall_initiation.png)

After confirming the form, the new paywall will appear in the list of the Paywall extension interface. You can then copy the link to the paywall and publish it wherever prospective donors can find it. They will then be able to access a webpage with the title and description you've set up as shown in the next figure.

![Fundraising interface](/assets/fundbounty/bounty_fundraising.png)

After they confirm the amount to contribute an invoice QR code with a copy option will appear. Upon successful payment will allow them to see the page in the link. The funds will then be added to your wallet.

## Support Tickets/Bounty Claim

Another extension that we are going to use is called Support Ticket. In the interface of this extension we can create a form to be filled by the user. The submission of the form is upon a payment of a submission fee, that is currently based on the number of words that the user fills in the claim. 

In the Support Ticket we click on the "NEW FORM" button that opens a form where we can set the properties of the claim interface. Again we select the wallet, where the submission fees should be collected and fill in the form name and the description. As the spam protection we submit the Amount per word that the contributor for each form in the claim.

![Claim form initiation](/assets/fundbounty/bounty_form.png)

Finally we click on the "CREATE FORM" button and the form will appear in the extension interface of the Support tickets. We can now copy the link to the claim form and publish it on the github issue or any other place, where the makers that contribute to our project can find it. They can then open the interface  as shown on the next figure and submit their claim on what they wish to have compensated and how. They also enter their name and contact details so you have a way to contact them.

![Bounty claim initiation](/assets/fundbounty/bounty_claim.png)

## LNURLw / Bounty assignment

The last extension that we need is the LNURL withdraw extension that allows us to create vouchers for the genuine contributions to the project. There is a way to create quick or advanced vouchers. For our purposes I believe that the quick vouchers work just fine -- so click on the "QUICK VOUCHERS" button in the LNURLw extension and the interface as shown on the next figure will appear.

![Voucher creation](/assets/fundbounty/create_voucher.png)

For larger amounts it might be reasonable to create a number of vouchers of smaller amounts. The smaller payments can go through more easy as they are are more likely to be under the capacity constrains of the lightning channels. As another way to overcome this problem the user can always create his own account on the same LNbits portal and claim the bounty there. Once we confirm the voucher creation form, we can copy the link to the LNURLw. 

Now we should be careful about the way we share the link in a way that nobody can copy it and claim it instead of the person it is aimed for. For that reason **always use only end-to-end encrypted communication channel to share the LNURLw links**. Once the user has obtained the link, they can access it on a page similar to the following.

![Voucher creation](/assets/fundbounty/lnurl-withdraw.png)

Of course, another way to send the bounty to the contributors is to ask them for their invoice. The LNURL way is more practical, as there are no time restrictions as it is the case with invoices due to their limited validity.

## Final thoughts

This article describes a practical way of doing a bounty fundraising, assignment and payouts for github issues. It can obviously be extended to any useful work even in the physical space that others can do for you or the community.

The LNbits also has an API interface that can be used from other projects. One useful application would be the adaptation of the [sputn1ck fundraising][sputn1ckgit] project to allow it to use the LNbits API in addition to `lnd` interface.

Some further functionalities could also be beneficial to be added to LNbits in order to use it in that way. One of them being the ability to quote the amount already raised and the specification of the fundraising target. 

The fundraising target would also be useful for the purposes of Paywall. In this case the Paywall could allow gratis access to all users after reaching the funding target.

Further improvements of the Paywall fundraising that might be worthy are the possibility of adding a fundraising target and end of the fundrising. Once the target or the selected time is reached, the fundraising would stop. 

Also, some tasks only makes sense, if it's done before a certain deadline as a way to make sure it is completed within a reasonable time frame. If the work is not completed by then, it would be fair to return the funds to the donors. Sending the funds through the LNURLw in this case might be too much of a hassle for the fundraising initiator, thus another way of returning the funds is needed. One way of doing it could be to allow the donor to provide an invoice key on their LNbits account, so that the system could reimburse them automatically in case the fundraising is cancelled. 

The other controversy is the custody of the funds. First the operator of the LNbits server could just disappear with the funds. Second, the user that start the fundraising needs to be trusted and potentially abuse the funds and never deliver actual results. Those risks can never be completely mitigated, so the contributors should verify how trust-worthy the involved parties are to protect himself from scams.

## Acknowledgement

Thanks to the organisers of [fulmo lightning hacksprint], it was a great opportunity to connect with the comunity and learn more about the progress that is going on in this days. Also, I am greatful for advice of LNbits developers in particular [Ben Arc] who provided useful hints to how to use LNbits for this purpose.

## Ad

Thanks for reading this article. If you liked it please [subscribe to RSS feed][RSS] and/or [donate][donation]. Also, feel free to [get in touch][mailme] if you have any advice for improvements or suggestions for further research.


[fulmo lightning hacksprint]: https://wiki.fulmo.org/wiki/Lightning_HackSprint_March_2021
[sputn1ckgit]: https://github.com/sputn1ck/github-bounty

[deploying the LNbits on your server]: https://blog.lightningconductors.net/post/2021/03/22/deploying-lnbits.html
[lnbits.com]: https://lnbits.com
[Ben Arc]: https://github.com/arcbtc



[RSS]: https://blog.lightningconductors.net/feed.xml
[donation]: https://btcpay.lightningconductors.net/api/v1/invoices?storeId=FFPzRyoNZHuENk4uyNSehkscnDsRLWZpLedmCzipt9tU&checkoutDesc=Thanks+for+donating+to+lightningconductors.net&price=42&currency=sats


[mailme]: mailto:info@lightningconductors.net
