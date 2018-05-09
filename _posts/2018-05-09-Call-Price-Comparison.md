---
layout:		post
title:		Why is American option's value equal to European option's when there is no dividend?
subtitle:
date:		2017-12-04
author: 	Grant6899
header-img: img/post-bg-finance.jpg
catalog: true
tags:
    - Option
---

## American Call vs European Call - No Dividend

An American option can be exercised at any time, whereas a European option can only be exercised at the expiration date. This added flexibility of American options increases their value over European options in certain situations. Thus:

	American Options = European Options + Premium where the Premium is greater than or equal to zero. 

For standard American call options without dividends, there are several reasons why the call should never be exercised before the expiration date. 
-	First, for a given movement in an underlying asset, the profit from holding an in the money call is equivalent to the
profit from holding the underlying asset. The call option, however, has the added benefit of protecting against the risk of a downward price movement below the strike price.
-	Second, because of the time value of money, it costs more to exercise the option today at a fixed strike price K than in the future at K. 
-	Finally, there is an intrinsic time value of the option that would be lost by exercising the option prior to the expiration date.

Hence, the price of an American and European call option without dividends should not diverge. 


## American Call vs European Call - With Dividend

The price of an American call option on an underlying asset that pays dividends, however, may diverge from its European counterpart. For an American call with dividends it may be beneficial to exercise the option prior to expiration. **By exercising the call, the owner of the call will be entitled to dividend payments that they would not have otherwise received.** This means that prior to dividend announcement American options must include a premium based on some distribution of an expected dividend. Once the dividend is announced, the premium must be adjusted again to account for this revised information.


## American Put vs European Put

For put options, the price of American and European options can diverge **even for underlying assets that do not pay dividends**. 

-	This is due to the limit in the value of the put option P(t)≤K imposed by the fact that the underlying S cannot have a negative value. 

-	**For deep in the money puts, it may be optimal to exercise the put prior to expiration and earn the risk free rate on the profits.** Thus, the price of an American put must reflect the potential profits that can be earned by exercising the put prior to expiration and earning the risk free rate on the profits.

-	Additionally, since European puts cannot be exercised until expiration and the strike price K is fixed in the future, we can discount the strike price to today to create a more stringent limit for the price of a European put: P(t)≤Kexp[-r(T-t)]. 

In order to account for the premium added by an American option, it is necessary to consider time (t[i]) as a discrete variable. At one time step before maturity (t[T-1]), the option will be exercised if h(S[T-1]) > f(t[T-1]), where h is the value of exercising the option and f is the value of holding the option until maturity. Hence, the price of the option is max(h(S[T-1]), f(t[T-1])). For example, for an American put, it would be optimal to exercise the put if (K-S[T-1])(1+r)>f(t[T-1]), and the price would be max[(K-S[T-1])(1+r),f(t[T-1])]. This logic can then be iterated back through each time period to find the price for American options for each time period ti. Unfortunately, the Black Scholes equation cannot be used to find a closed form solution for this iterative process. 

