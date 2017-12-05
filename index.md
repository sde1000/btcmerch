# Update, July 2017

We no longer accept Bitcoin.

The uncertainty around the future of the network from 1st August,
combined with high transaction fees and slow / unreliable transaction
confirmations was the final straw.

1. Very high fees mean that it isn't sensible to use Bitcoin for small
transactions.  Note that the fee must be paid twice: once by the
customer to send Bitcoin to us, and then again by us to send the
Bitcoin to an exchange to convert back to pounds.

2. If a Bitcoin transaction is sent without a high fee (or sometimes
even with a high fee), there's no guarantee that it will be included
in a block.  Blocks are frequently full, and transactions can hang
around in the mempool for ages before being discarded.

# Update, June 2017

1. The minimum transaction we're willing to accept in Bitcoin is £50.00.  This is because Bitcoin fees are now very high (on the order of £2.00 per transaction) and they must be paid both by our customers (when they send the transaction to us) and again by us (when we send the transaction to the Bitcoin exchange to convert back to pounds).
2. Bitcoin is now only accepted at the Pembury Tavern and Queen Edith pubs.  The service has been turned off at the other pubs due to repeated failed transactions.
3. Unless the Bitcoin network capacity problem is fixed by lifting the 1Mb limit per block, it is likely that we will stop accepting Bitcoin altogether by August 2017.

# Original text follows

This service is used to accept payment in Bitcoin in pubs run by [Individual Pubs Ltd](https://www.individualpubs.co.uk/).

The server-side code isn't published yet.  The client code is [part of the "quicktill" package](https://github.com/sde1000/quicktill).

# Customer experience

1. Order your drinks and food.
2. Ask to pay by Bitcoin.
3. The till will display a QR code for you to scan with your mobile Bitcoin wallet.  If you can't scan from the screen, it can be printed on paper.  Note that this code is for this transaction only; don't save the address to your address book, it won't be valid for any other transactions.
4. Once you've sent the Bitcoins, the till should say that they have been received.  If there's a problem (users of the blockchain.info wallet often find that there's a 20 or 30 minute delay at this point) we can set the transaction aside and check for payment later.

At the moment we offer payment by Bitcoin using the exchange rate supplied by the bitpay.com rate API.  We fetch this twice per day; we don't call them for every transaction.

## Caveats

1. If you overpay, we don't give change; there's nothing in the protocol to say where we should send it!
2. We don't offer "cashback" on Bitcoin transactions; you can't use us to convert Bitcoin to Sterling.
3. If you underpay, the transaction won't complete until you send the rest of the money.  If we've received part-payment, we can offer another QR code requesting the remaining amount.

## Future directions

We'll probably drop our own merchant service at some point and alter the till software to use [bitpay](https://bitpay.com/).  This will have a few advantages for customers, because they support the [BIP 70 payment protocol](https://github.com/bitcoin/bips/blob/master/bip-0070.mediawiki):

1. If you overpay, the overpayment can be returned to you via the change/refund address you supply as part of the protocol.
2. If you underpay, the Bitcoin payment can be returned to you and you can pay using a different method.
3. If you don't like your drink or food, we can issue a refund in Bitcoin.

# How it works

Bitcoin processing is provided to Individual Pubs Ltd by Stephen Early acting as a sole trader.  (Full disclosure: Stephen Early is also a director of Individual Pubs Ltd.)  The pub company's accounts
are entirely in Sterling; Stephen Early might make a profit or a loss on providing this service.

Steve runs the reference Bitcoin daemon on a server in a datacenter, which is connected to all our pubs over a VPN.  The server also runs a custom web service written in Python using Django that accepts requests from the tills and interfaces with the Bitcoin daemon using the [bitcoin-python library](https://github.com/laanwj/bitcoin-python).

When a till needs to request Bitcoins for a transaction, it sends a request to the web service.  The web service looks up the current exchange rate and stores it, and fetches a fresh Bitcoin address from the Bitcoin daemon; a bitcoin: URL is returned to the till requesting payment.  The till converts the URL to a QR code and displays it.  The till then makes the same request whenever it needs to check whether the transaction has been paid; the web service either returns a bitcoin: URL requesting the balance of the payment, or a "success" result.

When the till is being closed at the end of the day, it sends a request to the web service listing the transactions it believes have been paid by Bitcoin.  The web service responds with the Sterling total that is due to the pub for those transactions; hopefully this matches the amount the till is expecting!  A reconciliation record is created in the web service, and all the transactions named by the till are marked as Reconciled.  At this point, Steve pays the pub company the total amount in Sterling for those transactions.

Once a sufficient number of Bitcoins have been accumulated in reconciled transactions, Steve uses an external service to convert them to Sterling.  If the amount received in Sterling is greater than the amount paid to the pub company, Steve records a profit.  If it is lower, Steve records a loss.  At the end of the tax year, Steve will pay tax in Sterling based on the total profit or loss in the year.

# Frequently asked questions

(Journalists start here!)

## Why did you decide to accept Bitcoin?

I became aware of Bitcoin sometime in 2010, and followed news about it irregularly.  I bought myself some Bitcoins using the now-defunct "britcoin" exchange in June 2011, but then didn't do anything with them for ages.  Bitcoin being in the news in the first couple of months of 2013 reminded me.

Let's take a step back for a moment and talk about accepting cards.

We currently have cordless card terminals: we have one or two behind the bar, and we take them over to wherever the customer is standing to deal with the payment.  The workflow for a card transaction looks something like this:

1. Enter the customer's drinks into the till; the till comes up with a total.
2. Copy the total into the card machine manually (eg. if the drinks came to £12.35 then at this point we would press "1 2 3 5 Enter" on the card machine).
3. Take the card machine to the customer.  Their card goes in the front of the machine (it's all chip-and-PIN here, we generally don't accept magstripe-only cards).  If it's a debit card, the machine prompts us to ask if the customer would like cashback, and if they want it, how much.
4. The customer enters their PIN, the card machine contacts the merchant service provider, and if the transaction goes through it prints out two receipts - one for us, and one for the customer.
5. We take our copy of the receipt back to the till.  We enter any cashback amount into the till, and the four-digit receipt number.  The receipt is filed, and if there's any cashback then we take the appropriate amount out of the till and give it to the customer.

At the end of the day, each card machine produces a totals receipt.  We add together the totals on all the receipt and hope it matches up with what the till thinks we took!  Quite often, it doesn't — someone
will have made a mistake entering transaction or cashback amounts at some point in the day.

What I would _like_ to happen when accepting cards is for the till to transfer the transaction total over the network to the card machine, and for the card machine to send a response at the end of the transaction with the cashback amount and receipt number.  This would remove the need for values to be copied across manually.  It sounds like a small change - but I can't find anybody who will sell me equipment that does this! It clearly exists because I see it in supermarkets, large chain pubs, and so on — I just suspect that my company is seen as too small to be worth selling to.

Let's compare this to the workflow for accepting Bitcoin:

1. Enter the customer's drinks into the till; the till comes up with a total.
2. Press "pay by Bitcoin".  The till displays a QR code.
3. The customer scans the QR code with their wallet app and presses "Send".
4. The till display changes to say the transaction is complete.

At the end of the day, the Bitcoin total is generated automatically. It's all much easier for our staff, and with much less scope for human error.

So, why did I decide to accept Bitcoin?  Because it let me build the kind of system that I _wanted_ to build for accepting cards, but couldn't.

## How long have you been accepting Bitcoins?

I started testing with my own Bitcoins on 23rd May 2013 in Norwich.  The first Bitcoin transaction that involved a third-party took place on 1st June 2013 at the Pembury Tavern in Hackney; this is [the one mentioned on Reddit here](http://www.reddit.com/r/Bitcoin/comments/1fh10f/holy_smokes_my_first_ever_bitcoin_beer_it_works/).

## How many customers are taking advantage of your acceptance of Bitcoins?

Short answer?  Not many, but more than I expected — in the first two weeks we took about £750-worth of Bitcoins.  It's settled down to about £1000-worth per month, so it's only a small part of the business.  It's obviously still enthusiasts who have heard about it on the net!

## Have you heard of any other pubs accepting Bitcoin?

There are a few.  Have a look at [coinmap](http://coinmap.org/).

## Can I ask you some questions about how you accept Bitcoin?  I'm writing an article...

Sure, if they have not already been answered here.  If you ask particularly good questions I'll add them to this page!

## Have you heard about Litecoin, Ripples, Dogecoin, ...?

Yes.  They are not currently interesting to me.

## Are you involved in any other Bitcoin projects?

No, I'm not.
