---
layout:		post
title:		Yield Curve in Quantlib
subtitle:
date:		2017-11-25
author: 	Grant6899
header-img: img/post-bg-c++.jpg
catalog: true
tags:
    - c++
    - Quantlib
---


## What is a yield curve?

Yield curve is the term structure of intrest rate. In other words, it shows the relationship between time to maturity and interest rates.

## Base class: TermStructure

### Overview

TermStructure is derived from Observer, Observable and Extrapolator.

- Observer: it's observing evaluation date in global settings
- Observable: it's observed by instrments whose valuation depending on it
- Extrapolator: base class for classes possibly allowing extrapolation, contains a bool extrapolate_ to indicate if extrapolation is allowed

### Source code

```c++
class TermStructure : public virtual Observer,
                      public virtual Observable,
                      public Extrapolator {
      public:
		// term structures initialized by means of this constructor must manage their own reference date by overriding the referenceDate() method
        explicit TermStructure(const DayCounter& dc = DayCounter());
        
        //! initialize with a fixed reference date
        explicit TermStructure(const Date& referenceDate, const Calendar& calendar = Calendar(), const DayCounter& dc = DayCounter());
        
        //! calculate the reference date based on the global evaluation date
        TermStructure(Natural settlementDays,
                      const Calendar&,
                      const DayCounter& dc = DayCounter());
        virtual ~TermStructure() {}
        
        //! the day counter used for date/time conversion
        virtual DayCounter dayCounter() const;
        //! date/time conversion
        Time timeFromReference(const Date& date) const;
        //! the latest date for which the curve can return values
        virtual Date maxDate() const = 0;
        //! the latest time for which the curve can return values
        virtual Time maxTime() const;
        //! the date at which discount = 1.0 and/or variance = 0.0
        virtual const Date& referenceDate() const;
        //! the calendar used for reference and/or option date calculation
        virtual Calendar calendar() const;
        //! the settlementDays used for reference date calculation
        virtual Natural settlementDays() const;
        
        void update();
        
      protected:
        //! date-range check
        void checkRange(const Date& d,
                        bool extrapolate) const;
        //! time-range check
        void checkRange(Time t,
                        bool extrapolate) const;
        bool moving_;
        mutable bool updated_;
        Calendar calendar_;
      private:
        mutable Date referenceDate_;
        Natural settlementDays_;
        DayCounter dayCounter_;
    };

    // inline definitions

    inline DayCounter TermStructure::dayCounter() const {
        return dayCounter_;
    }

    inline Time TermStructure::maxTime() const {
        return timeFromReference(maxDate());
    }

    inline Calendar TermStructure::calendar() const {
        return calendar_;
    }

    inline Natural TermStructure::settlementDays() const {
        QL_REQUIRE(settlementDays_!=Null<Natural>(),
                   "settlement days not provided for this instance");
        return settlementDays_;
    }

    inline Time TermStructure::timeFromReference(const Date& d) const {
        return dayCounter().yearFraction(referenceDate(), d);
    }
```

### Analysis

Reference date: the date at which discount = 1.0 and/or variance = 0.0.

Three ways to track Reference date: 
- Reference date is fixed, the constructor takes a date is to be      used; the default implementation of referenceDate() will then return such date
- Determined by advancing the current date (**evaluation date in global setting**) by several business days, the constructor takes a number of days and a calendar is to be used
- Reference date is based on that of some other structure, the referenceDate() method must be overridden in derived classes so that it fetches and return the appropriate date

Its update() is not defined in the class, waiting for overwriting in derived classes.


## Intermediate class: YieldTermStructure

### Source Code

```c++
class YieldTermStructure : public TermStructure {
public:
	YieldTermStructure(const DayCounter& dc = DayCounter(),
                           const std::vector<Handle<Quote> >& jumps = std::vector<Handle<Quote> >(),
                           const std::vector<Date>& jumpDates = std::vector<Date>());
                           
    YieldTermStructure(const Date& referenceDate,
                           const Calendar& cal = Calendar(),
                           const DayCounter& dc = DayCounter(),
                           const std::vector<Handle<Quote> >& jumps = std::vector<Handle<Quote> >(),
                           const std::vector<Date>& jumpDates = std::vector<Date>());
                           
    YieldTermStructure(Natural settlementDays,
                           const Calendar& cal,
                           const DayCounter& dc = DayCounter(),
                           const std::vector<Handle<Quote> >& jumps = std::vector<Handle<Quote> >(),
                           const std::vector<Date>& jumpDates = std::vector<Date>());

		/*! \name Discount factors

            These methods return the discount factor from a given date or time
            to the reference date.  In the latter case, the time is calculated
            as a fraction of year from the reference date.
        */
        //@{
        DiscountFactor discount(const Date& d,
                                bool extrapolate = false) const;
        /*! The same day-counting rule used by the term structure
            should be used for calculating the passed time t.
        */
        DiscountFactor discount(Time t,
                                bool extrapolate = false) const;
        //@}

        /*! \name Zero-yield rates

            These methods return the implied zero-yield rate for a
            given date or time.  In the former case, the time is
            calculated as a fraction of year from the reference date.
        */
        //@{
        /*! The resulting interest rate has the required daycounting
            rule.
        */
        InterestRate zeroRate(const Date& d,
                              const DayCounter& resultDayCounter,
                              Compounding comp,
                              Frequency freq = Annual,
                              bool extrapolate = false) const;

        /*! The resulting interest rate has the same day-counting rule
            used by the term structure. The same rule should be used
            for calculating the passed time t.
        */
        InterestRate zeroRate(Time t,
                              Compounding comp,
                              Frequency freq = Annual,
                              bool extrapolate = false) const;
        //@}

        /*! \name Forward rates

            These methods returns the forward interest rate between two dates
            or times.  In the former case, times are calculated as fractions
            of year from the reference date.

            If both dates (times) are equal the instantaneous forward rate is
            returned.
        */
        //@{
        /*! The resulting interest rate has the required day-counting
            rule.
        */
        InterestRate forwardRate(const Date& d1,
                                 const Date& d2,
                                 const DayCounter& resultDayCounter,
                                 Compounding comp,
                                 Frequency freq = Annual,
                                 bool extrapolate = false) const;
        /*! The resulting interest rate has the required day-counting
            rule.
            \warning dates are not adjusted for holidays
        */
        InterestRate forwardRate(const Date& d,
                                 const Period& p,
                                 const DayCounter& resultDayCounter,
                                 Compounding comp,
                                 Frequency freq = Annual,
                                 bool extrapolate = false) const;

        /*! The resulting interest rate has the same day-counting rule
            used by the term structure. The same rule should be used
            for calculating the passed times t1 and t2.
        */
        InterestRate forwardRate(Time t1,
                                 Time t2,
                                 Compounding comp,
                                 Frequency freq = Annual,
                                 bool extrapolate = false) const;
        //@}

        //! \name Jump inspectors
        //@{
        const std::vector<Date>& jumpDates() const;
        const std::vector<Time>& jumpTimes() const;
        //@}

        //! \name Observer interface
        //@{
        void update();
        //@}
      protected:
        /*! \name Calculations

            This method must be implemented in derived classes to
            perform the actual calculations. When it is called,
            range check has already been performed; therefore, it
            must assume that extrapolation is required.
        */
        //@{
        //! discount factor calculation
        virtual DiscountFactor discountImpl(Time) const = 0;
        //@}
      private:
        // methods
        void setJumps();
        // data members
        std::vector<Handle<Quote> > jumps_;
        std::vector<Date> jumpDates_;
        std::vector<Time> jumpTimes_;
        Size nJumps_;
        Date latestReference_;
    };

    // inline definitions

    inline
    DiscountFactor YieldTermStructure::discount(const Date& d,
                                                bool extrapolate) const {
        return discount(timeFromReference(d), extrapolate);
    }

    inline
    InterestRate YieldTermStructure::forwardRate(const Date& d,
                                                 const Period& p,
                                                 const DayCounter& dayCounter,
                                                 Compounding comp,
                                                 Frequency freq,
                                                 bool extrapolate) const {
        return forwardRate(d, d+p, dayCounter, comp, freq, extrapolate);
    }

    inline const std::vector<Date>& YieldTermStructure::jumpDates() const {
        return this->jumpDates_;
    }

    inline const std::vector<Time>& YieldTermStructure::jumpTimes() const {
        return this->jumpTimes_;
    }

}
```

### Analysis

Note that same day count convention as parent term structure is used for discount factor calculation, daycounter_ is protected in term structrue class.

Discount factor, zero rate and forward rate can be dereived among each other. So the implementation of zero rate is calling discount() to calculate compond then use InterestRate::ImpliedRate(), so does forwardRate().

You should always define discountImpl(t), which is called in discount(), in a derived class to make it work, since it's a pure virtual function here in yieldtermstructure. In other words, yieldtermstructure is an abstract class.




## Children classes

There are various ways to construct a market consistent yield curve. The methodology depends on the liquidity of available market instruments for the corresponding market. Several choices have to be made; the interpolation procedure and the choice of the market instruments have to be specified. 

QuantLib allows to construct a yield curve as:
- InterpolatedDiscountCurve, construction given discount factors
- InterpolatedZeroCurve, construction given zero coupon bond rates
- InterpolatedForwardCurve, construction given forward rates
- FittedBondDiscountCurve, construction given coupon bond prices
- PiecewiseYieldCurve, piecewise construction given a mixture of market instruments (i.e. deposit rates, FRA/Future rates, swap rates).

 All of the above constructors derive from the base class YieldTermStructure. The YieldTermStructure class derives from TermStructure, which is both, an Observer and Observable. This base class implements some useful functions. For example, functions are implemented which return the reference date, day counter, calendar, the minimum or maximum date for which the curve returns yields.
 
 
### InterpolatedDiscountCurve 

#### Source Code:
```c++
    template <class Interpolator>
    class InterpolatedDiscountCurve
        : public YieldTermStructure,
          protected InterpolatedCurve<Interpolator> {
      public:
        InterpolatedDiscountCurve(
            const std::vector<Date>& dates,
            const std::vector<DiscountFactor>& dfs,
            const DayCounter& dayCounter,
            const Calendar& cal = Calendar(),
            const std::vector<Handle<Quote> >& jumps = std::vector<Handle<Quote> >(),
            const std::vector<Date>& jumpDates = std::vector<Date>(),
            const Interpolator& interpolator = Interpolator());
        InterpolatedDiscountCurve(
            const std::vector<Date>& dates,
            const std::vector<DiscountFactor>& dfs,
            const DayCounter& dayCounter,
            const Calendar& calendar,
            const Interpolator& interpolator);
        InterpolatedDiscountCurve(
            const std::vector<Date>& dates,
            const std::vector<DiscountFactor>& dfs,
            const DayCounter& dayCounter,
            const Interpolator& interpolator);
        //! \name TermStructure interface
        //@{
        Date maxDate() const;
        //@}
        //! \name other inspectors
        //@{
        const std::vector<Time>& times() const;
        const std::vector<Date>& dates() const;
        const std::vector<Real>& data() const;
        const std::vector<DiscountFactor>& discounts() const;
        std::vector<std::pair<Date, Real> > nodes() const;
        //@}
      protected:
        InterpolatedDiscountCurve(
            const DayCounter&,
            const std::vector<Handle<Quote> >& jumps = std::vector<Handle<Quote> >(),
            const std::vector<Date>& jumpDates = std::vector<Date>(),
            const Interpolator& interpolator = Interpolator());
        InterpolatedDiscountCurve(
            const Date& referenceDate,
            const DayCounter&,
            const std::vector<Handle<Quote> >& jumps = std::vector<Handle<Quote> >(),
            const std::vector<Date>& jumpDates = std::vector<Date>(),
            const Interpolator& interpolator = Interpolator());
        InterpolatedDiscountCurve(
            Natural settlementDays,
            const Calendar&,
            const DayCounter&,
            const std::vector<Handle<Quote> >& jumps = std::vector<Handle<Quote> >(),
            const std::vector<Date>& jumpDates = std::vector<Date>(),
            const Interpolator& interpolator = Interpolator());
        //! \name YieldTermStructure implementation
        //@{
        DiscountFactor discountImpl(Time) const;
        //@}
        mutable std::vector<Date> dates_;
      private:
        void initialize();
    };
```

#### Analysis

When objects intialized, dates will be passed and converted into time vector, then gets involved into the calculation of interpolation.

nodes() are calculated on the surface of this class, it's paired by dates_ and data_, and is not part of low-level calculation.

