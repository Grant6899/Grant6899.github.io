---
layout:		post
title:		Interest Rate Summary
subtitle:
date:		2017-11-27
author: 	Grant6899
header-img: img/post-bg-finance.png
catalog: true
tags:
    - Finance
    - Interview
---

## Interest Rate


### Definitions
| Concept  | Definition | Comment |
|:---|:-|:-|
|  Treasury Rate    |  Treasury rates are the rates an investor earns on Treasury bills and Treasury bonds. | Treasury rates are therefore totally risk-free rates in the sense that an investor who buys a Treasury bill or Treasury bond is certatin that interest and principal payments will be made as promised  |
|  LIBOR(London Interbank Offered Rate)   | LIBOR is usually considered to be an estimate of the short-term unsecured borrowing rate for a AA-rated financial institution.  |   |
|  The Fed Funds Rate    | In the United States, the overnight rate is called the federal funds rate  |   |
|  Repo Rate   | The financial institution is obtaining a loan and the interest it pays is the difference between the price at which the securities are sold and the price at which they are repurchased.  | Both LIBOR and federal funds rate are unsecured borrowing rates |
|Discount Factor| the factor by which a future cash flow must be multiplied in order to obtain the present value. | |
|  Zero Rate(Spot Rate)    | The n-year zero-coupon interest rate is the rate of interest earned on an investment that starts today and lasts for n years  | Most common type is overnight repo. Usually slightly below the corresponding fed funds rate. Zero rate's compounding method is simple.|
|  Bond Pricing   | The theoretical price of a bond can be calculated as the present value of all the cash flows that will be received by the owner of the bond based on using a different zero rate for each cash flow  |   |
|  Bond Yield(YTM)   | A bond's yield is the single discount rate that, when applied to all cash flows, gives a bond price equal to its market price.  |   |
|  Par Yield   | The par yield for a certain bond maturity is the coupon rate that causes the bond price to equal its par value.(Still use zero rate to discount, given par value, derive the coupon rate)  |   |
|  Bootstrap Method   | Objective: determine Treasury zero rates is from Treasury bills and coupon-bearing bonds. 1. Get zero rates from zero coupons. 2. Calculate a longer-term zero rate according to bond pricing equation and known zero rates.  |   |
|  Zero Curve   | A chart showing the zero rate as a function of maturity is known as the zero curve.   | A common assumption is the zero curve is linear between the points determined using the bootstrap method. |
|  Forward Rate   | Forward interest rates are the future rates of interest implied by current zero rates for periods of time in the future.  |   |
|  Forward Rate Agreement(FRA)   | A forward rate agreement is an over-the-counter transaction designed to fix the interest rate that will apply to either borrowing or lending a certatin principal during a specified future period of time.  | The present value of the payment is made at the beginning of the specified period.   |
|  Duration(continous componding)   | The duration of a bond is a measure of how long on average the holder of the bond has to wait before receiving cash payments  |   |
|  Modified Duration(componding frequency of m)   |   |   |
|  Convexcity   | A factor measures this curvature to improve the relationship between yield change and bond price change especially when yield change is large  |   |
|  Yield Curve   | Term structure of interest rate |   |

### Day Count Convention (According to quantlib)

| Day Count Convetion | TempDate1 | TempDate2 | Convention.dayCount(TempDate1, TempDate2) | Convention.yearfraction(TempDate1, TempDate2) |
|:---|:-|:-|:-|:-|
|Actual360 |  Jan/1/2012 | Mar/30/2012 | 31 + 29 + 30 - 1 = 89 | 89/360 = 0.247222  | 
|Actural365Fixed | Jan/1/2012 | Mar/30/2012 | 31 + 29 + 30 - 1 = 89 | 89/365(always fixed at 365) = 0.243836 |
|Actural365NoLeap | Jan/1/2012 | Mar/30/2012 | 31 + 28 (No Leap) + 30 - 1 = 88 | 88/365 = 0.241096 |
|ActualActual| Jan/1/2012 | Mar/30/2012 | 31 + 29 + 30 - 1 = 89 | 89/366 = 0.243169  | 
|Business252| Jan/1/2012 | Mar/30/2012 | 22 + 19 + 22 - 1 = 62(This is by default using Brazil's calender in quantlib, Jan/01, Feb 21-22 are holidays) | 62/252 = 0.246032  | 
|OneDayCounter| Jan/1/2012 | Mar/30/2012 | 1 | 1 |
|SimpleDayCounter| Jan/1/2012 | April/1/2012 | 31 + 29 + 31 - 1 = 90 | 3 month/12 month = 0.25  | 
|Thirty360| Jan/1/2012 | Mar/30/2012 | 1 | 1 |

Comments:
- SimpleDayCounter: Simple day counter is for reproducing theoretical calculations. This day counter tries to ensure that whole-month distances are returned as a simple fraction, i.e., 1 year = 1.0, 6 months = 0.5, 3 months = 0.25 and so forth. This day counter should be used together with **NullCalendar**, which ensures that dates at whole-month distances share the same day of month. It is not guaranteed to work with any other calendar.


### Compounding Method(According to quantlib)

- Simple: ![simpleequation](http://latex.codecogs.com/gif.latex?1&plus;rt)

- Compounded: ![compundedequation](http://latex.codecogs.com/gif.latex?(1&plus;r)^t)

- Continuous: ![continuousequation](http://latex.codecogs.com/gif.latex?e^{rt})

- SimpleThenCompounded: Simple up to the first period then Compounded

- CompundedThenSimple: Compounded up to the first period then Simple


### Frequency

- NoFrequency: null frequency
- Once: only once. e.g., a zero-coupon
- Annual: once a year
- Semiannual: twice a year
- EveryFourthMonth: every fourth month
- Quarterly: every third month
- Bimonthly: every second month
- Monthly: once a month
- EveryFourthWeek: every fourth week
- Biweekly: every second week
- Weekly: once a week
- Daily: once a day

Notice that only when Compunding is "Compounded"(or any that uses "compounded"), Frequency will be used. Otherwise when it's either simple or continous, whatever Frequency you use will not make any difference.

### Relationship among discount factor, zero rate and forward rate
---

Notation: T below is year fraction.

#### Discount Factor

- Discount factor vs Zero rate

  There is no compounding in zero-coupon bond, so:

  ![dfvszero](http://latex.codecogs.com/gif.latex?DF(T)&space;=&space;\frac{1}{1&plus;rT})
  
- Discount factor vs annually-compounded rate(YTM of a US Treasury bond with annual coupons)
  
  ![dfvscompound](http://latex.codecogs.com/gif.latex?DF(T)&space;=&space;\frac{1}{(1&plus;r)^T}) 
  
- Discount factor vs money market rate(LIBOR):
  
  ![dfvsmmr](http://latex.codecogs.com/gif.latex?DF(T)&space;=&space;\frac{1}{{(1&plus;\frac{r}{360})}^{360T}})
  
- Discount factor vs money market rate whose DCC is ACT/365:
  
  ![dfvsmmr365](http://latex.codecogs.com/gif.latex?DF(T)&space;=&space;\frac{1}{{(1&plus;\frac{r}{365})}^{365T}})
  
- Discount factor in mannual/theoretical calculation:

  Sometimes, for manual calculation, the continuously-compounded hypothesis is a close-enough approximation of the daily-compounding hypothesis, and makes calculation easier (**even though it does not have any real application as no financial instrument is continuously compounded**)
  
  ![dfvsman](http://latex.codecogs.com/gif.latex?DF(T)&space;=&space;e^{-rT})
  
#### Forward Rate
  
-  The forward rate is the future yield on a bond. It is calculated using the yield curve. For example, the yield on a three-month Treasury bill six months from now is a forward rate.
  
   ![dfvsfr](http://latex.codecogs.com/gif.latex?r_{1,2}&space;=&space;\frac{1}{t_2-t_1}[\frac{DF(0,t_1)}{DF(0,t_2)}&space;-&space;1])


### Typical attributes common interest rate (pending)

| Name | Compunding | Frequency | Day Count Convetion | 
|:---|:-|:-|:-|
|LIBOR|
|Zero Bonds| Simple | Once | Actual365Fixed|
|Deposit Rate| 