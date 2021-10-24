# Concurrent execution of two or more parts of a program for maximum utilization of CPU. In java each execution is called threads

# Threads can be created by:
	1. Extending thread class
	2. Implementing the runnable interface
- If we extend thread class, our class cannot extend any other class. But if we implement Runnable interface, our class can still extend base classes.
- Thread class is the main class on which Java's Multithreading system is based.
- Thread class also implements Runnable interface.
- Runnable interface is also used to create thread and should be used if planning to override only the run() method and no other Thread methods.

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
