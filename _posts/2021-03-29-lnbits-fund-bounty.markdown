---
layout: post
title:  "LNbits Bounty Fundraising"
date:  Mon 29 Mar 16:20:30 UTC 2021
categories: post
---

During the [fulmo lightning hacksprint] that happened this weekend I came across a project to the [Sputn1ck `github-bounty` project][sputn1ckgit] which aims to fundraise the money to github issues through lightning. As for now the `github-bounty` uses the `lnd` backend API in order to generate the invoices and count the amount of contributions that are then displayed in the issue page.

Using a self hosted node is an ideal trustless setup that leaves you in control of your funds at any point. However, the relatively complicated setup and management of the lightning node is too intimidating for many potential users. The LNbits project is a self-hosted custodial service that allows to delegate the management of the node to a trusted third party. Although this idea is somehow against the mindset of many bitcoiners, because the provider might disappear with your funds at any moment. It is a risk that many users are willing to exchange for the convenience when dealing with smaller amount of money. Moreover, if you self-host the LNbits on your server you can benefit from the advanced features it offers without necessary loosing the control.

The LNbits system allows creating custom made extensions with their own API. At first I have thought that creating a new Bounty fundraising API would be a good idea and started to explore the possibility of such implementation. The LNbits is written in Python and uses Assynchronous Server Gateway Interface (ASGI) that serves the HTML with JavaScript code (in Vue) generated on the fly.

As I later realised, there is already a way of creating the fundraising system using a few existent LNbits extensions. Here I am going to describe how such system could work.

## Wallet Setup

The previous article I have written was about [deploying the LNbits on your server]. Other possibility is to use one of the existent deployments such as [lnbits.com]. 

When you hit the initial page of LNbits you get the option to create your own wallet by giving it a name and click confirm.

![LNbits wallet setup](/assets/fundbounty/wallet_setup.png)

This will take you to your wallet interface where you can see the funds that are are assigned to you, you can create invoices to receive funds from other lightning network users, or send funds to others.

Your wallet access credentials appear in the URL of the website, so you should make sure that you **never send the link to anyone else**, because they would be able to open the same webpage, and obtain the same rights as you have. Also, it is necessary that the address is only accessible through SSL encrypted secure HTTPS and not the legacy HTTP system, as it would make it vulnerable to the man in the middle attack.

![LNbits wallet interface](/assets/fundbounty/wallet_interface.png)

In the wallet interface you can create additional separate wallets that you can use for different purposes.

## Choosing the extensions

To create the bounty fundraising we are going to use three extensions:

+ Paywall -- extension aimed to create payment walls to fundraise for a web links, we are going to use it to rise money for the bounty
+ Support Tickets -- extension to create payed support tickets to prevent spam or earn for answering questions, we are going to provide the link for the users claiming the bounty
+ LNURLw -- extension to send withdraw links to bounty winners

In your wallet interface you can find an option to manage extension. When you click there, you enter to another page where you can enable those extensions for yourself. Then the extensions get activated and you can access to their interface through the link just under the list of your wallets:

![Claim form initiation](/assets/fundbounty/active_extensions.png)

## Paywall/Bounty fundraising setup

We are going to abuse the Paywall extension for the bounty fundraising. So having enabled it, you can enter the Paywall extension in the LNbits wallet interface. On the top of the page click on the "NEW PAYWALL" button. A form will appear for you to fill it up. Select the wallet to which the fundraising should go. It is a good idea to have a separate wallet for each fundrising for easy accounting of how much fund have been raised.

Then complete the a title and a description in the form. The redirect URL and minimal amount fields are obligatory, however not particularly useful for our purposes. We can just add a link that will be shown after the donor has made a successful payment so ideally we could create something like a thank you page, or just put a link to the github issue we are fundraising for. 

We could also choose a minimal amount, but it makes sense to leave it on quite a low value, so that anyone can decide what amount they contribute with.

![Paywall initiation](/assets/fundbounty/paywall_initiation.png)

After confirming the form, the new paywall will appear in the list of the Paywall extension interface. In there you can find and copy the link to the paywall and publish it wherever prospective donors will find it.

With the link they will then be able to access the webpage as shown in the next figure.

![Fundraising interface](/assets/fundbounty/bounty_fundraising.png)

After they confirm the amount to contribute an invoice appears. The funds will be added to your LNbits wallet upon a successful payment.

## Support Tickets/Bounty Claim

Another extension that we are going to use is called Support Ticket. In the interface of this extension we can create a form to be filled by the user. The submission of the form is upon a payment of a submission fee, that is currently based on the number of words that the user fills in the claim.

In the Support Ticket we click on the "NEW FORM" button that opens a form to set the properties of the claim interface. Again we select the wallet, where the submission fees should be collected and fill in a form name and a description. As the spam protection we submit the Amount per word that the contributor for each form in the claim.

![Claim form initiation](/assets/fundbounty/bounty_form.png)

Finally we click on the "CREATE FORM" button and the form will appear in the extension interface of the Support tickets. We can now copy the link to the claim form and publish it on the github issue or any other place, where the developers contributing to our project find it. They can then open the interface as shown on the next figure and submit their claim on what they wish to have compensated and how. They also enter their name and contact details so you have a way to contact them.

![Bounty claim initiation](/assets/fundbounty/bounty_claim.png)

## LNURLw / Bounty assignment

The last extension that we need is the LNURL withdraw extension that allows us to create vouchers for the genuine contributions to the project. There is a way to create quick or advanced vouchers. I believe that quick vouchers work just fine for our purposes -- so click on the "QUICK VOUCHERS" button in the LNURLw extension to get to the voucher creation form.

![Voucher creation](/assets/fundbounty/create_voucher.png)

For larger amounts it might be reasonable to create a number of vouchers of smaller amounts. As the smaller amounts are more likely to be under the capacity constrains of the lightning channels, they are more likely to pass. As another way to overcome the small capacity problem, the bounty winner could also create his own account on the same LNbits portal as the withdrawal would be settled internally within the portal. Once we confirm the voucher creation form, we can copy the link to the LNURLw. 

Now we should be careful about the way we share the link safely. For that reason **always use only end-to-end encrypted communication channel to share the LNURLw links** to prevent email providers or other intruders to copy the link and claim the bounty instead of the winner. Once the bounty winners have obtained the link, they can access it on a page similar to the following.

![Voucher creation](/assets/fundbounty/lnurl-withdraw.png)

Of course, another way to send the bounty to the contributors is to ask them for their invoice. However, the LNURL way is more practical, as there are no time limit within which the payment has to be settled as it is the case with lightning invoices.

## Final thoughts and further work

The controversy of such fundraising system is the custody of the funds. First the operator of the LNbits server could just disappear with the funds. Second, the user that start the fundraising needs to be trusted and potentially abuse the funds and never deliver actual results. Those risks can never be completely mitigated, so the contributors should verify how trust-worthy the involved parties are to protect themselves from scams.

The fundraising initiator should consider an ideal workflow for the fundraising to offer more clarity to the donors. It makes sense that the fundraising has a specific deadline when it should finish and the same can be applied for the actual work delivery to make sure it is completed within a reasonable time frame. 

If the work is not completed by then, it would be fair to return the funds to the donors. Sending the funds through the LNURLw in this case might be too much of a hassle for the fundraising initiator, thus another way of returning the funds is needed. One way of doing it could be to allow the donor to provide an invoice key on their LNbits account, so that the system could reimburse them automatically in case the fundraising is cancelled. Other way would allow the donors above a certain amount to enter their reimbursement details. Finally, the fundraising could specify what happens with the funds in case that the work fails (e.g. charity donation, forwarding the money to fund other issues).

Further improvements of the Paywall fundraising might be beneficial, in particular an option to stop the fundraising is currently not available. It could be done in different ways, as for instance adding a fundraising target  or select point in time when it would stop. The fundraising target would also be useful for the purposes of Paywall. In this case the Paywall could allow gratis access to all users after reaching the funding target.

The support ticket extension could have a fixed submission fee (perhaps even a word limit) to motivate the users to explain themselves clearly on their contributions.

The LNbits also has an API interface that can be used from other projects. One useful application would be the adaptation of the [sputn1ck fundraising][sputn1ckgit] project to allow it to use the LNbits API in addition to `lnd` interface. Some further functionalities might need to be added to LNbits in order to use it in that way, but many are already implemented -- like the invoice creation and quoting the amount with the Invoice/read key.


## Final thoughts

This article describes a practical way of doing a bounty fundraising, assignment and payouts for github issues. It can obviously be extended to any useful work even in the physical space that others can do for you or the community.


## Acknowledgement

Thanks to the organisers of [fulmo lightning hacksprint], it was a great opportunity to connect with the community and learn more about the progress that is going on in this days. Also, I am grateful for advice of LNbits developers in particular [Ben Arc] who provided useful hints to how to use LNbits for this purpose.

## Ad

Thanks for reading this article. If you liked it please share, [subscribe to RSS feed][RSS] and to recognise its value consider a [donation]. Also, feel free to [get in touch][mailme] if you have any advice for improvements or suggestions for further research.


[fulmo lightning hacksprint]: https://wiki.fulmo.org/wiki/Lightning_HackSprint_March_2021
[sputn1ckgit]: https://github.com/sputn1ck/github-bounty

[deploying the LNbits on your server]: https://blog.lightningconductors.net/post/2021/03/22/deploying-lnbits.html
[lnbits.com]: https://lnbits.com
[Ben Arc]: https://github.com/arcbtc

[RSS]: https://blog.lightningconductors.net/feed.xml
[donation]: https://btcpay.lightningconductors.net/api/v1/invoices?storeId=FFPzRyoNZHuENk4uyNSehkscnDsRLWZpLedmCzipt9tU&checkoutDesc=Thanks+for+donating+to+lightningconductors.net&price=42&currency=sats

[mailme]: mailto:info@lightningconductors.net
