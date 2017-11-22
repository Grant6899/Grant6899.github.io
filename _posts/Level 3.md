# Level 3 Notes  

标签（空格分隔）： C++ Adv_C++_Notes

---
## Thread constructed with a callable type
- Global function
- Static member function
- Callable object(class with operator ())

## Volatile
- Compilers can optimize code by storing simple variables in processor registers
- Example
```c++
volatile int i;
// The functions now use a volatile variable.
// Not optimised anymore so results are always correct.
void F1() { while (true) i++; }
void F2() { while (true) cout<<i<<endl; }
int main()
{
// Start threads that operate on the same int variable
boost::thread t1(&F1);
boost::thread t2(&F2);
}
```

## Thread Lifecycle
- Running
 - The thread has been created and started
 - Processor time is allocated for the thread
- WaitSleepJoin
 - Thread is waiting for a certain event to happen
 - Thread will be put back in running state when event has happened
- Stopped
 - The thread function was completed

## Control Thread Execution
- Yield()
 - Multitasking not guaranteed to be preemptive
 - Lengthy thread functions must give other threads a change to run
 - Can be done by the yield() functions
 - Control is returned as soon as possible
 - Example:
```c++
 class PowerClass // Class that calculates m^n
 {
 private:
 int m, n; // Variables for m^n
 public:
 double result; // Public data member for the result
 // Constructor with arguments
 PowerClass(int m, int n)
 { this->m=m; this->n=n; this->result=0.0; }
 // Calculate m^n. Supposes n>=0.
 void operator () ()
 {
 result=m; // Start with m^1
 for (int i=1; i<n; i++)
 {
 result*=m; // result=result*m
 boost::this_thread::yield(); // Allow other threads to run
 }
 if (n==0) result=1; // m^0 is always 1
 }
 };
```

- sleep()
 - The current thread can be put to sleep for a certain amount of time
 - Duration can be boost::posix_time::time_duration or subclasses
 - Other threads continue to run
 - For tasks which occur at regular intervals
 - Thread cannot force another thread to sleep

- join()
 - We can wait for another thread to finish
 - Puts calling thread in WaitSleepJoin state
 - Used when we have to wait for an operation to finish
 - Example:
```c++
 int main()
{
int m=2;
int n=200;
// Create a m^n calculation object
PowerClass pc(m, n);
// Create thread and start m^n calculation in parallel
// Since we want to read the result from pc, we must pass it as reference,
// else the result will be placed in a copy of pc.
boost::thread t(boost::ref(pc));
// Now we do our own calculation while the PowerClass is calculating m^n
double result=m*n;
// Wait till the PowerClass is finished
// Leave this out and the result will be bogus
t.join();
// Display result
cout<<"("<<m<<"^"<<n<<") / ("<<m<<"*"<<n<<") = "<<pc.result/result<<endl;
}
```
- Stop a thread
 - Thread stops when thread function finished
 - Use shared variable to indicate if a thread must stop
 - In the thread function's working loop, check variaable
 - Exit the loop when the thread must stop
- interrupt()
 - When in WaitSleepJoin state, the thread can be interrupted by another thread
 - When that happens, it will throw a boost::thread_interrupted exception
   bool boost::thread::interrupt();
 - If thread doesn't use wait/sleep/join, use:
   void boost::this_thread::interruption_point();
   
## Locking using Mutex Class
- mutex class provides method to get and release a lock on the mutex
 void boost::mutex::lock();
 void boost::mutex::unlock();
- When mutex already locked, thread enters SleepWaitJoin state till mutex gets unlocked
 - Only one thread can hold a lock on a mutex
 - Code after lock() can thus be executed by only one thread at the time
- Minimize the time you hold a lock

## Locking using lock_guard Class
- Easy to forget call to mutex::unlock()
- Can use lock_guard<Lockable> help class
 - Locks given mutex in constructor
 - Unlocks given mutex in destructor

## Thread Notification
- Sometimes a thread has to wait for another thread to perform some action
- Bosst provides an efficient way for thread notification
- Using functions of condition_variable class
- wait()
 - Calling thread will release lock during wait
 - Sleeps till another thread calls Pulse()
- notify_one()
 - Signals change in object
 - One waiting thread wakes up after lock is released
- notify_all()
 - As notify_one() but all waiting threads wake up




 



