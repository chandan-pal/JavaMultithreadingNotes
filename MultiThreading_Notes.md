# Concurrent execution of two or more parts of a program for maximum utilization of CPU. In java each execution is called threads

# Threads can be created by:
	1. Extending thread class
	2. Implementing the runnable interface
- If we extend thread class, our class cannot extend any other class. But if we implement Runnable interface, our class can still extend base classes.
- Thread class is the main class on which Java's Multithreading system is based.
- Thread class also implements Runnable interface.
- Runnable interface is also used to create thread and should be used if planning to override only the run() method and no other Thread methods.
- When we extend the Thread class, each of the thread creates a unique object associated with it. Whereas in
Runnable interface, the same Runnable Object can be shared by multiple threads

## Lifecycle of a Thread in Java:
	1. New (a newly created thread, not yet ready for running)
	2. Runnable (The thread is fully prepared for running or already running - when thread.start() has been called)
	3. Blocked (The thread is temporarily inactive. The thread has to execute some protected code which is locked by some other thread)
	4. Waiting (The thread is temporarily inactive. The thread is waiting for some other thread on condition)
		A thread can be put in waiting state by calling:
			a. Object.wait with no timeout
			b. Thread.join with no timeout
			c. LockSupport.park
	5. Timed Waiting (The Thread called a method with timeout parameter and it is waiting until the timeout)
		A thread can be put in timed waiting state by calling:
			a. Thread.sleep
			b. Object.wait with timeout
			c. Thread.join with timeout
			d. LockSupport.parkNanos
			e. LockSupport.parkUntil
	6. Terminated (The thread has completed its execution either successfully or got terminated due to some exception)
		
	
	To get the current state of Thread: thread.getState();

## MAIN THREAD:
	- The thread in which a java application starts/begins
	- for each program a main thread is created by JVM
	- The main thread first verifies the existence of main method and then it initializes the class.
	- to obtain a reference of main thread: Thread.currentThread();

## YIELD, SLEEP & JOIN (methods on thread)
	yield():
		- means that the thread is not doing anything particularly important and if any other thread or process need to run, they should run. Otherwise the current thred will run.
		- it tells the Thread scheduler that this thread is ready to pause if needed
		- if there are multiple threads with yield, then the thread scheduler decides on priority.
		- if priority is also same then thread scheduler to pick any one
		- yield() does not directly put any thread in waiting. It is thread scheduler to decide.
	sleep():
		- sleep causes the thread to pause for a specified number of milliseconds.
		- even if there are no other thhread or process to run, then also the current thread will wait for specied number of time.
	join():
		- join method is  used to join the start of currennt thread's execution to end of the this thread's execution.
		- It means the current thread will not start running until the this thread ends
		- Thread.join(long millis, int nanos) : the instance thread will wait for given milliseconds and nanoseconds before dieing.
		- a timeout of zero means wait forever
		- If any executing thread t1 calls join() on t2 i.e; t2.join() immediately t1 will enter into waiting state until t2 completes its execution


## WAIT, NOTIFY, NOTIFYALL (methods on object)
	All these methods belong to object class as final so that all classes have them. They must be used within a synchronized block only.
	synchronized block ensures only one thread running at a time
	wait():
		- it tells the calling thread to give up lock and go to sleep until some other thread enters the same object and calls notify.
	notify():
		- It wakes up one single thread that called wait on the same object.
		- If you use notify method, It's not guaranteed which thread will be informed.
		- It should be noted that calling notify() does not actually give up a lock on a resource. It tells a waiting thread that that thread can wake up. However, the lock is not actually given up until the notifier’s synchronized block has completed.
	notifyAll():
		- It wakes up all the threads that called wait on the same object.

## Thread Priority:
	- Priority is the preference order for the thread scheduler to decide which thread to execute first.
	- Thread priority is an integer with minimum of 1 and maximum of 10.
	- 10 means max priority and 1 means minimum priority.
	- Thread.getPriority()
	- Thread.setPriority(int newPriority)
	- If two threads have same priority then we can’t expect which thread will execute first.



## The famous producer-consumer problem
To make sure that the producer won’t try to add data into the buffer if it’s full and that the consumer won’t try to remove data from an empty buffer.

Solution -
		- The producer is to either go to sleep or discard the data if the buffer is full. The next time consumer removes an item from buffer, it notifies the producer
		- In the same way consumer can go to sleep if it finds the buffer to be empty. The next time the producer puts data into buffer, it wakes up the sleeping consumer.

https://www.geeksforgeeks.org/producer-consumer-solution-using-threads-java/

* We can use BlockingQueue to easily solve this problem


# java.util.concurrent API

## Blocking Queue
The Java BlockingQueue interface, **java.util.concurrent.BlockingQueue**, represents a queue which is thread safe to put elements into, and take elements out of from. In other words, multiple threads can be inserting and taking elements concurrently from a Java BlockingQueue, without any concurrency issues arising.

A blocking queue is a queue that blocks when you try to dequeue from it and the queue is empty, or if you try to enqueue items to it and the queue is already full. A thread trying to dequeue from an empty queue is blocked until some other thread inserts an item into the queue. A thread trying to enqueue an item in a full queue is blocked until some other thread makes space in the queue, either by dequeuing one or more items or clearing the queue completely.

Use Java ‘BlockingQueue’ instead of using ‘wait/notify’. BlockingQueue supports operations that wait for the space to becoming available in the queue when storing an element and wait for the queue to become non-empty when retrieving an element.

## CountDownLatch
A synchronization aid that allows one or more threads to wait until a set of operations are being performed.

This class enables a java thread to wait until other set of threads completes their tasks. eg. Application’s main thread want to wait, till other service threads which are responsible for starting framework services have completed started all services.

CountDownLatch works by having a counter initialized with number of threads, which is decremented each time a thread complete its execution. When count reaches to zero, it means all threads have completed their execution, and thread waiting on latch resume the execution.

Pseudo code for CountDownLatch can be written like this :
- //Main thread start
- //Create CountDownLatch for N threads
- //Create and start N threads
- //Main thread wait on latch
- //N threads completes there tasks are returns
- //Main thread resume execution

Some practical usage of CountDownLatch
1. Achieving maximum parallelism
2. wait N threads to before starting execution of any specific task
3. Deadlock detection

```java
public class CountDownLatchDemo
{
    public static void main(String args[]) 
                   throws InterruptedException
    {
        // creation of countdown latch in main thread
        CountDownLatch latch = new CountDownLatch(3);  // countdown latch initialized with 3
  
        // creation of dependent threads
	// these worker thread are taking a parameter of the coutdown latch and they would decrease the countdown.
	// latch.countDown(); will be called inside worker threads
        MyWorkerThread first = new MyWorkerThread(1000, latch, "WORKER-1"); 
        MyWorkerThread second = new MyWorkerThread(2000, latch, "WORKER-2");
        MyWorkerThread third = new MyWorkerThread(3000, latch,  "WORKER-3");
        
	// start the worker threads
        first.start();
        second.start();
        third.start();

        // The main task waits for four threads
        latch.await();
  
        // Main thread has started
        System.out.println(Thread.currentThread().getName() +  " has finished");
    }
}
```

## CyclicBarrier
CyclicBarrier is used to make threads wait for each other. It is used when different threads process a part of the computation and when all threads have completed the execution, the results need to be combined in the parent thread.

After completing its execution, threads call await() method and wait for other threads to reach the barrier.

The constructor for a CyclicBarrier is simple. It takes a single integer that denotes the number of threads that need to call the await() method on the barrier instance to signify reaching the common execution point.

```java
public class CountDownLatchDemo
{
    public static void main(String args[]) 
                   throws InterruptedException
    {
        // creation of CyclicBarrier in main thread
        CyclicBarrier  cyclicBarrier  = new CyclicBarrier(3);  // CyclicBarrier initialized with 3
  
        // creation of dependent threads
	// these worker thread are taking a parameter of the cyclicBarrier and they would call await() after finishing their task.
	// cyclicBarrier.await(); will be called inside worker threads
        MyWorkerThread first = new MyWorkerThread(1000, cyclicBarrier, "WORKER-1"); 
        MyWorkerThread second = new MyWorkerThread(2000, cyclicBarrier, "WORKER-2");
        MyWorkerThread third = new MyWorkerThread(3000, cyclicBarrier,  "WORKER-3");
        
	// start the worker threads
        first.start();
        second.start();
        third.start();
	
  
        // Main thread has started
        System.out.println(Thread.currentThread().getName() +  " has finished");
    }
}
```

### Use synchronizers, such as ‘CountdownLatch’ and ‘CyclicBarrier’ instead of directly using ‘join’.


# Non-Blocking Data Structures
## Obstruction-Free
A data structure/algorithm provides obstruction freedom if, a thread is guaranteed to proceed if all other threads are suspended. (A thread is guaranteed to proceed if there are no obstructions)
i.e. if an algorithm is not obstruction free and two thread try to access the same data structure then the progress of both the threads would get stopped.

All lock-free algorithms are obstruction-free.

## Lock-Free
A data structure provides Lock Freedom if, at any time, at-least one thread can proceed.

## Wait-Free
A data structure is wait-free if it's lock-free and every thread is guaranteed to proceed after a finite number of steps.

	
## Thread Pool:
	- A group of pre-created threads that are waiting for a task to be assigned and can resume many times.
	- Why? A JVM creating and destroying too many threads can consume more time in creating and destroying resources than in the actual task.
	- To avaoid that, A thread from the thread pool is pulled out and assigned a job by a "Service Provider" or a "Executor".
	- After completion of the job, thread is contained in the thread pool again.
	
	- Executor allows to take advantage of threadpool and focus on task to perform instead of thread mechanics
	- Java provides executer framework.
	- Executor Interface ----> ExecutorService Interface  ------> ThreadPoolExecutor class

	- 1. The Executors factory class is used to create an instance of an Executor, either an ExecutorService or an ScheduledExecutorService
		e.g. ExecutorService executorService = Executors.newFixedThreadPool(5); - will return a ExceuterService that has a thread pool with fixed no of threads
	- 2. The execute method of ExceutorService takes a Runnable and is useful when you want to run a task and are not concerned about checking its status or obtaining a result.
		e.g. executorService.execute(new Runnable());
	
	- 3. The Runnable will be executed as soon as a thread is available from the ExecutorService thread pool.
	- 4. shuting down the Executor Service. executorService.shutdown(); 
	
## Risk in using ThreadPools:
	- Deadlock: Threadpools can cause a case of deadlock scenario in which some thread is waiting for a thread, but this task is in queue because of unavailability of thread in the pool.
	- Thread Leakage: Thread leakage occur when some thread was pulled for some task but it did not come back to the pool when the task was completed. E.g. if the thread throws an exception and the pool class did not catch this exception.
	- Resource Thrashing: If the thread pool size is very large then the time is wasted in context switching between threads.

## CALLABLE & FUTURE:
	- we have a Runnable interface to make thread classes in java, but with Runnable, we cannot make a thread return result when it terminates.
	- For supporting this feature, Java provides Callable Interface.
	- Callable can't create a thread, but it has call() method which can return result.
	- The call() method of Callable stores the result in a Future object. Which can be used later.
	
	- Future is a way in which the main thread keeps track and progress of the result from other threads.
	
	- to create a thread, a Runnable is required. To obtain the result, a Future is required.
	- Java has a concrete type "FutureTask" which implements Runnable and Future (combining both functionalities)
	Steps:
		1. A FutureTask can be created by providing its constructor with a callable.
		2. Then the FutureTask object is provided to the constructor of Thread to create Thread object.
		3. All interaction with the thread after that is with FutureTask object, and there is no need to store the thread object.
	
	- FutureTask has following imp methods:
		1. public boolean cancel(boolean mayInterrupt): Used to stop the task. It stops the task if it has not started. If it has started, it interrupts the task only if mayInterrupt is true.
		2. public Object get() throws InterruptedException, ExecutionException: Used to get the result of the task. If the task is complete, it returns the result immediately, otherwise it waits till the task is complete and then returns the result.
		3. public boolean isDone(): Returns true if the task is complete and false otherwise
[https://www.geeksforgeeks.org/callable-future-java/](https://www.geeksforgeeks.org/callable-future-java/)

## deciding factors for number of thread pools
https://techblogstation.com/java/thread-pool-size/

## Difference between Java Threads and OS Threads
https://www.geeksforgeeks.org/difference-between-java-threads-and-os-threads/#

## Green Thread Model and Native Thread Model
### Green Thread Model
	- In this model threads are completely managed by JVM without any kind of underlying OS support.
	- These threads are implemented at the application level and managed in user space.
	- They are also called cooperative (user-level) threads.
	- Only one green thread can be processed at a time. Hence this model is considered the many-to-one model. Because of this, green threads can run on multi-core processors but cannot take advantage of multiple-cores.
	- synchronization and resource sharing is easier for green threads and hence execution time is also less.
	- Sun Solaris OS provides support for Green Thread model.
	- Java no longer user Green Thread model but uses Native thread model from Java 1.3
### Native Thread Model
	- Thread is this model are managed by the JVM with the help of underlying OS support.
	- These threads are implemented ate the OS level (by using OS multithreading API) and managed in kernel space.
	- They are also called non-green (kernel-level) threads.
	- Mutliple threads can co-exist. Therefore it is also called many-to-many model. Such characteristic of this model allows it to take complete advantage of multiple-core processors and execute threads on separate individuals cores concurrently.
	- Since this model allows multiple threads to be executed simultaneously, thread synchronization and resource sharing become complicated. This increases execution time of threads.
	- All the windows based OS provide support for native thread model.
The Green Thread model is deprecated and no longer used. Native Thread model has replaced Green Thread model and it is used widely today.

