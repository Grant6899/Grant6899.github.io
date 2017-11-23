---
layout:		post
title:		Singleton
subtitle:
date:		2017-11-23
author: 	Grant6899
header-img: img/post-bg-c++.jpg
catalog: true
tags:
    - c++
    - Desgin Pattern
    - Quantlib
---


## Intent


Ensure a class only has one instance, and provide a global point of access to it.

## Applicability

- There must be exactly one instance of a classs and it must be accessible to clients from a well-known access point.
- When the sole instance should be extensible by subclassing, clients should be able to use an extended instance without modifying their code.


## Participants

- Singleton
  - defines an Instance operation that lets clients access its unique instance. Instance is a class operation, a static member function 
  - **may be** responsible for creating its own unique instance


## Collaborations

- Clients access a Singleton instance solely through Singleton's Instance operation.

## Consequences

There are several benefits using singleton:
1. Controlled access to sole instance.
2. Reduced name space.
3. Permits refinement of operations and representation. The Singleton class may be subclassed, and it's easy to configure an application with an instance of this extended class. You can configure the application with an instance of the class you need at run-time.
4. Permits a variable number of instances.
5. More flexible than class operations.


## Implementation

### 1. Ensuring a unqiue instance.

   ```c++
   class Singleton {
   public:
	   static Singleton* Instance();
   protected:
	   Singleton();
   private:
	   static Singleton* _instance;
   };
   
   Singleton* Singleton::_instance = 0;
   Singleton* Singleton::Instance () {
   if (_instance == 0) {
       _instance = new Singleton;
       }
   return _instance;
   }
   ```
   Notice that the constructor is **protected**. A client that tries to instantiate Singleton directly will get an error at compile-time. This ensures that only one instance can ever get created.
   
   Moreover, since the _instance is a pointer to a Singleton object, the Instance member function can assign a pointer to a subclass of Singleton to this variable.
   
   It isn't enough to define the singleton as a global or static object and then rely on automatic initialization.
   
### 2. Subclassing the Singleton class.

There methods to implement subclass:
- The simplest technique is to determine which singleton you want to use in the Singleton's Instance operation.
 

- Take the implementation of Instance out of the parent class and put it in the subclass.

- Use a registry of singletons. Instead of having Instance define the set of possible Singleton classes, the Singleton classes can register their singleton instance by name in a well-known registry:

  ```c++
  class Singleton {
  public:
	  static void Register(const char* name, Singleton*);
	  static Singleton* Instance();
  protected:
	  static Singleton* Lookup(const char* name);
  private:
      static Singleton* _instance;
      static List<NameSingletonPair>* _registry;
  };
  
  Singleton* Singleton::Instance () {
	  if (_instance == 0) {
		  const char* singletonName = getenv("SINGLETON");
		  // user or environment supplies this at startup
		  _instance = Lookup(singletonName);
		  // Lookup returns 0 if there's no such singleton
	  }
	  return _instance;
  }
  
  MySingleton::MySingleton() {
		  // ...
		  Singleton::Register("MySingleton", this);
  }
  ```
  
## Implementation in Quantlib

In quantlib,  Singleton pattern is a design where a class is allowed to have only one instance. For example, one application is a global repository where you store market objects. In this case it is important to have a single repository that is used by all classes, e.g. you want the pricing to be based on the same yield curve. A global variable does not meet these requirements,
since it can not be ensured that multiple objects are instantiated. 

Settings deninition:

Notice that Settings is a derived from Singleton<Settings>, meanwhile Singleton<Settings> is also a friend class of Settings, so they can access each other.

   ```c++
 class Settings : public Singleton<Settings> {
        friend class Singleton<Settings>;
      private:
        Settings();
        class DateProxy : public ObservableValue<Date> {
          public:
            DateProxy();
            DateProxy& operator=(const Date&);
            operator Date() const;
        };
        friend std::ostream& operator<<(std::ostream&, const DateProxy&);
      public:
        //! the date at which pricing is to be performed.
        /*! Client code can inspect the evaluation date, as in:
            \code
            Date d = Settings::instance().evaluationDate();
            \endcode
            where today's date is returned if the evaluation date is
            set to the null date (its default value;) can set it to a
            new value, as in:
            \code
            Settings::instance().evaluationDate() = d;
            \endcode
            and can register with it, as in:
            \code
            registerWith(Settings::instance().evaluationDate());
            \endcode
            to be notified when it is set to a new value.
            \warning a notification is not sent when the evaluation
                     date changes for natural causes---i.e., a date
                     was not explicitly set (which results in today's
                     date being used for pricing) and the current date
                     changes as the clock strikes midnight.
        */
        DateProxy& evaluationDate();
        const DateProxy& evaluationDate() const;

        /*! Call this to prevent the evaluation date to change at
            midnight (and, incidentally, to gain quite a bit of
            performance.)  If no evaluation date was previously set,
            it is equivalent to setting the evaluation date to
            Date::todaysDate(); if an evaluation date other than
            Date() was already set, it has no effect.
        */
        void anchorEvaluationDate();
        /*! Call this to reset the evaluation date to
            Date::todaysDate() and allow it to change at midnight.  It
            is equivalent to setting the evaluation date to Date().
            This comes at the price of losing some performance, since
            the evaluation date is re-evaluated each time it is read.
        */
        void resetEvaluationDate();

        /*! This flag specifies whether or not Events occurring on the reference
            date should, by default, be taken into account as not happened yet.
            It can be overridden locally when calling the Event::hasOccurred
            method.
        */
        bool& includeReferenceDateEvents();
        bool includeReferenceDateEvents() const;

        /*! If set, this flag specifies whether or not CashFlows
            occurring on today's date should enter the NPV.  When the
            NPV date (i.e., the date at which the cash flows are
            discounted) equals today's date, this flag overrides the
            behavior chosen for includeReferenceDate. It cannot be overridden
            locally when calling the CashFlow::hasOccurred method.
        */
        boost::optional<bool>& includeTodaysCashFlows();
        boost::optional<bool> includeTodaysCashFlows() const;

        bool& enforcesTodaysHistoricFixings();
        bool enforcesTodaysHistoricFixings() const;
      private:
        DateProxy evaluationDate_;
        bool includeReferenceDateEvents_;
        boost::optional<bool> includeTodaysCashFlows_;
        bool enforcesTodaysHistoricFixings_;
    };
   ```


Singleton<T>::instance() arrages all T objects in a std::map, and pair them with a session ID. 
   ```c++
    template <class T>
    T& Singleton<T>::instance() {

        #if (QL_MANAGED == 0) && !defined(QL_SINGLETON_THREAD_SAFE_INIT)
        // Integer
        static std::map<Integer, boost::shared_ptr<T> > instances_;
        #endif

        // thread safe double checked locking pattern with atomic memory calls
        #if defined(QL_SINGLETON_THREAD_SAFE_INIT) 

        T* instance =  instance_.load(boost::memory_order_consume);
        
        if (!instance) {
            boost::mutex::scoped_lock guard(mutex_);
            instance = instance_.load(boost::memory_order_consume);
            if (!instance) {
                instance = new T();
                instance_.store(instance, boost::memory_order_release);
            }
        }

        #else //this is not thread safe

        #if defined(QL_ENABLE_SESSIONS)
        Integer id = sessionId();
        #else
        Integer id = 0;
        #endif

        boost::shared_ptr<T>& instance = instances_[id];
        if (!instance)
            instance = boost::shared_ptr<T>(new T);

        #endif

        return *instance;
    }
   ```

### Question

Why we apply different logic when putting the static map? If we put it into the creation method, each time when we call instance(), why isn't it complaining for redefinition (maybe for threaded programs?)?

```c++
// excerpt of singleton.hpp
#if (_MANAGED == 1) || (_M_CEE == 1)
// One of the Visual C++ /clr modes. In this case, the global instance
// map must be declared as a static data member of the class.
#define QL_MANAGED 1
#else
// Every other configuration. The map can be declared as a static
// variable inside the creation method.
#define QL_MANAGED 0
#endif

    template <class T>
    class Singleton : private boost::noncopyable {
    #if (QL_MANAGED == 1) && !defined(QL_SINGLETON_THREAD_SAFE_INIT)
      private:
        static std::map<Integer, boost::shared_ptr<T> > instances_;
    #endif

	//...
      public:
        //! access to the unique instance
        static T& instance();
      protected:
        Singleton() {}
    };
```

	
Answer: http://www.cnblogs.com/dongzhiquan/archive/2009/07/21/1994792.html	











