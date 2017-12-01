---
layout:		post
title:		Interest Rate Swap
subtitle:
date:		2017-11-28
author: 	Grant6899
header-img: img/post-bg-finance.png
catalog: true
tags:
    - Finance
	- Derivatives
    - Interview
---

## Definition

An interest rate swap is a financial derivative instrument in which two parties agree to exchange interest rate cash flows. It is used in order to **hedge against** or **speculate on** changes in interest rates.

The payer of fixed rate is "buyer", the other party is the "seller" of the swap.

## Characteristics:

### Swap Attributes

| Term | Definition | Note/Example |
|:-|:-|:-|
| Notional | this notional amount is only used for calculating the size of cash flows to be exchanged | The notional amount is not exchanged if the 2 legs have the **same** currency |
| Currency | typically, currencies are the same for both legs | Cross Currency Swap will have different currencies |
| Trade Date | The date at which the swap is traded | |
| Value Date | The date at which the swap is really effective. The date from which cash flows are calculated. | Also called **effective date** |
| End Date | The maturity date of the swap | |
| Reset Date | The date at which the new rate is used | a swap could use the 3m LIBOR rate as of 2 London business days prior to the Reset Date. The 2 business day lag is known as the "Floating Leg Fixing Lag"|

### Leg-level Attributes

| Term | Definition | Note/Example |
|:-|:-|:-|
| Rate type | Fixed rate or floating rate |  |
| Frequency | The frequency at which cash flows are paid or received | Can be different between legs |
| Time basis | The day count convention used for each leg | Can be different between legs |


## Valuation

At inception date, the rate of the fixed leg is generally determined in order to calculate a valuation equal to 0 at this date. If the valuation is not equal to 0, a cash payment will occur.

Two yield curves will be used:

- The zero-coupon yield curve, used to calculate the discount rates of future cash flows, paid or received, fixed or floating. Cash flows of each leg have to be discounted.

- The forward rate curve, used to calculate the size of the floating cash flows paid (or received). If the rate of the floating leg is 6 month Libor, this curve will inform on the level of the 6 month Libor at each fixing date (we calculate therefore the size of the cash flows). **This curve can be deducted from the zero-coupon yield curve, or collect directly on a data market provider (Bloomberg, Reuters, etc.)**.

Once cash flows calculated, we have to sum each discounted cash flow on each leg.

Finally, the swap valuation is the difference between the sum of the discounted received cash flows and the sum of the discounted paid cash flows.


## Example
Example can be found from this link: ![http://www.sport-histoire.fr/en/Finance/Interest_rate_swap.php](http://www.sport-histoire.fr/en/Finance/Interest_rate_swap.php)