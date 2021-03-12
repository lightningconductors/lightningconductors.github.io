---
layout: page
title: About
author: Zebra
permalink: /about/
---

This blog brings information about [Lightning
Network](/lightning/2020/12/04/welcome-to-lightningconductors.html) --
privacy focused, scalable and trustless crypto-currency payment
system.

[Please get in touch](mailto:info@lightningconductors.net), if you would like
to contribute with an article or have any comments or suggestions.

To stay up to date with the changes and new articles subscribe via [RSS
reader](/feed.xml).

Disclaimer: This website is for entertainment purposes only and should
not be taken as ultimate guide to secure your secrets. The authors and
operator of the website are not responsible for any losses and damages
resulting from use of the information provided.

<script>if(!window.btcpay){    var head = document.getElementsByTagName('head')[0];   var script = document.createElement('script');   script.src='https://btcpay.lightningconductors.net/modal/btcpay.js';   script.type = 'text/javascript';   head.append(script);}function onBTCPayFormSubmit(event){    var xhttp = new XMLHttpRequest();    xhttp.onreadystatechange = function() {        if (this.readyState == 4 && this.status == 200) {            if(this.status == 200 && this.responseText){                var response = JSON.parse(this.responseText);                window.btcpay.showInvoice(response.invoiceId);            }        }    };    xhttp.open("POST", event.target.getAttribute('action'), true);    xhttp.send(new FormData( event.target ));}</script><style type="text/css"> .btcpay-form { display: inline-flex; align-items: center; justify-content: center; } .btcpay-form--inline { flex-direction: row; } .btcpay-form--block { flex-direction: column; } .btcpay-form--inline .submit { margin-left: 15px; } .btcpay-form--block select { margin-bottom: 10px; } .btcpay-form .btcpay-custom-container{ text-align: center; }.btcpay-custom { display: flex; align-items: center; justify-content: center; } .btcpay-form .plus-minus { cursor:pointer; font-size:25px; line-height: 25px; background: #DFE0E1; height: 30px; width: 45px; border:none; border-radius: 60px; margin: auto 5px; display: inline-flex; justify-content: center; } .btcpay-form select { -moz-appearance: none; -webkit-appearance: none; appearance: none; color: currentColor; background: transparent; border:1px solid transparent; display: block; padding: 1px; margin-left: auto; margin-right: auto; font-size: 11px; cursor: pointer; } .btcpay-form select:hover { border-color: #ccc; } #btcpay-input-price { -moz-appearance: none; -webkit-appearance: none; border: none; box-shadow: none; text-align: center; font-size: 25px; margin: auto; border-radius: 5px; line-height: 35px; background: #fff; } #btcpay-input-price::-webkit-outer-spin-button, #btcpay-input-price::-webkit-inner-spin-button { -webkit-appearance: none; margin: 0; } </style>
<form method="POST"  onsubmit="onBTCPayFormSubmit(event);return false"  action="https://btcpay.lightningconductors.net/api/v1/invoices" class="btcpay-form btcpay-form--inline">
  <input type="hidden" name="storeId" value="FFPzRyoNZHuENk4uyNSehkscnDsRLWZpLedmCzipt9tU" />
  <input type="hidden" name="jsonResponse" value="true" />
  <input type="hidden" name="checkoutDesc" value="Thanks for donating to lightningconductors.net" />
  <div class="btcpay-custom-container">
    <div class="btcpay-custom">
      <input id="btcpay-input-price" name="price" type="number" min="1" max="250000" step="1" value="42" style="width: 3em;" oninput="event.preventDefault();isNaN(event.target.value) || event.target.value <= 0 ? document.querySelector('#btcpay-input-price').value = 42 : event.target.value"  />
    </div>
    <select name="currency">
      <option value="SATS">sats</option>
      <option value="USD">USD</option>
      <option value="GBP">GBP</option>
      <option value="EUR">EUR</option>
    </select>
  </div>
<button type="submit" class="submit" name="submit" style="min-width:209px; min-height:57px; border-radius: 4px;border-style: none;background-color: #0f3b21;" alt="Pay with BtcPay, Self-Hosted Bitcoin Payment Processor"><span style="color:#fff">Donate with</span>
<img src="https://btcpay.lightningconductors.net/img/logo.svg" style="height:57px;display:inline-block;padding: 5% 0 5% 5px;">
</button></form>
