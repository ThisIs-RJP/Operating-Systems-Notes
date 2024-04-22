
# Background

- Processes can execute concurrently/in parallel
	- May be <u>interrupted</u> at any time, partially completing execution
- Concurrent access to shared data may result in data inconsistency
- So, maintaining data consistency requires mechanisms to ensure the orderly execution of cooperating processes
- We illustrated this simple example before... which lead to a race condition

# Race Condition

- Let's say there are two processes, A and B, <u>both trying to access and modify the same variable in memory</u>. Process A wants to increment the variable by 1, and process B wants to decrement it by 1. If there's no mechanism to synchronise their access to the variable, a race condition can occur.
	- If process A reads the variable's value, then process B reads the same value, both processes increment and decrement respectively, and finally, they write the modified values back to the variable, the final result might be incorrect because each process operated assuming the value hadn't changed.
	- For instance, if the initial value is 5, both processes read 5, then A increments it to 6, B decrements it to 4, and they both write back their respective values. The expected result would be 5, but due to the race condition, it's 4.

- Processes P0 and P1 are creating child processes using the ```fork()``` system call
- Race condition on kernel variable ```next_available_pid``` which represents the next available process identifier (pid)
![[Pasted image 20240418122822.png]]

- Unless there is a mechanism to prevent P0 and P1 from accessing the variable ```next_available_pid```, the same pid can be assigned to two different processes!

# Critical Section Problem

-  In operating systems, a critical section refers to a segment of code or a region of a program that <u>accesses shared resources (such as variables, data structures, or devices) that must not be simultaneously accessed by multiple threads or processes</u>. It's called "critical" because <u>incorrect execution or concurrent access within this section can lead to race conditions and data inconsistency</u>.

- Consider system of ***n*** processes {*p0*, *p1*, ... *p(n -1)*}
- Each process has **critical section** segment of code
	- Process may be changing common variables, updating table, **writing file**, etc
	- When one process in critical section, **no other may be in its critical section**
- ***Critical section problem*** is design **protocol to solve this**
- Often structured: Each process must <u>ask permission to enter critical section</u> in **entry section**, may follow critical section with **exit section**, then **remainder section**
- Requirements for solution to critical-section problem
	- 1. **Mutual Exclusion** - If process ***Pi*** is executing in its critical section, then no other processes can be executing in their critical sections
	- 2. **Progress** - If no process is executing in its critical section and there exist some processes <u>that wish to enter their critical section</u>, then the selection of the process that will enter the critical section next cannot be postponed indefinitely
	- 3. **Bounded Waiting** - A bound must exist on the number of times that other processes are allowed to enter their critical sections after a process has made a request to enter its critical section and before that request is granted

# Critical Section

- General structure of process Pi

![[Pasted image 20240418123507.png]]

# Interrupt-based Solution

- Entry section: **disable interrupts**
- Exit section: **enable interrupts**

# Peterson's Algorithm
- A software solution
- Two process solution
- Assume that the ```load``` and ```store``` machine-language instructions are **atomic**; that is; it cannot be interrupted
- The two processes <u>share</u> 2 variables:
	- ```int turn;```
	- ```boolean flag[2]```
- The variable ```turn``` indicates whose turn it is to enter the critical section
- The ```flag``` array is used to indicate if a process is ready to enter the critical section
	- ```flag[i] = true``` implies that process Pi is ready

# Algorithm for Process Pi

![[Pasted image 20240418125207.png]]


# Correctness of Peterson's Solution

**Note: CS = Critical Section**
- Provable that the 3 CS requirements are met:
	- 1. Mutual exclusion is preserved
		- Pi enters CS only if:
			- either ```flag[j] = false or turn = i```
	- 2. Progress requirement is satisfied
	- 3. Bounded-waiting requirement is met

# Peterson's Solution and Modern Architecture

- Although useful for demonstrating an algorithm, Peterson's Solution is not guaranteed to work on modern architecture
	- **Why?** To improve performance, processors and/or compilers may reorder operations that have no dependencies
- Understanding why it will not work is useful for better understanding race conditions
- For single-threaded this is (*probably*) okay as the result will always be the same
	- For multithreaded, the reordering may produce inconsistent or unexpected results! 

# Modern Architecture Example

- Two threads share the data:
		```boolean flag = false;```
		```int x = 0;```
- Thread  1 performs
	 ```while (!flag);```
		 ```print x```

- Thread 2 performs
		```x = 100;```
		```flag = true```

- What is the expected output?
		```100```

	- However, since the variables ```flag``` and ```x```  are independent of each other, the instructions:
			```flag = true;```
			```x = 100```
	 for Thread 2 may be recorded
- If this occurs, the output may be 0

# Peterson's Solution Revisited

- The effects of instruction reordering in Peterson's Solution
![[Pasted image 20240418130550.png]]

- This allows both processes to be in their critical section at the same time!
- To ensure that Peterson's solution will work correctly on modern computer architecture, we must use **Memory Barrier**

# Memory Barrier

- **Memory models** are the memory guarantees a computer architecture makes to application programs
- Memory models may be either:
	- **Strongly Ordered** - where a memory modification of one processor is immediately visible to all other processors
	- **Weakly ordered** - Where a memory modification of one processor may not be immediately visible to all other processors

![[Pasted image 20240418130853.png]]

- A **memory barrier** is an instruction that forces any change in memory to be propagated (made visible) to all other processors

# Memory Barrier Instructions

- When a **memory barrier** instruction is performed, the system ensures that **all loads and stores are completed before any subsequent load or store operations are performed**
	- Therefore, even if instructions were reordered, the memory barrier ensures that the store operation are completed in memory and visible to other processors before future load or store operations are performed

# Memory Barrier Example

- Using the last example
	- We could add a memory barrier to the following instructions to ensure Thread 1 outputs 100:
	- Thread 1 now performs
		```while (!flag)```
			```memory_barrier();```
		```print x ```
	- Thread 2 now performs
		```x = 100;```
		```memory_barrier()```
		```flag = true```
	- For Thread 1 we are guaranteed that the value of ```flag``` is loaded before the value of ```x```
	- For Thread 2 we ensure that the assignment to ```x``` occurs before the assignment ```flag```

# Synchronisation Hardware

- Many systems provide hardware support for implementing the critical section code
- Uniprocessors - could disable interrupts
	- Currently running code would execute **without preemption**
	- Generally too inefficient on multiprocessor systems
		- OS's using this not broadly scalable


# Hardware Instructions

- **Special hardware instructions** that allow us to either *test-and-modify* the content of a word, or to *swap* the contents of two words **atomically** (uninterrupted)
	- **Test-and-Set** instruction
	- **Compare-and-Swap** instruction

# The test_and_set instruction
  
The `test_and_set` instruction is a fundamental synchronisation primitive used in concurrent programming and operating systems. Its purpose is to atomically test the value of a memory location and set it to a specified value if it meets certain conditions. This atomic operation ensures that no other process or thread can intervene between the test and the set operations, thus providing mutual exclusion for critical sections.

The basic functionality of `test_and_set` can be summarized as follows:

1. Atomically read the current value of a memory location.
2. Set the memory location to a specified value (usually 1, indicating that the resource is being used or locked).
3. Return the original value read in step 1.

![[Pasted image 20240418131816.png]]

- Properties
	- **Executed atomically**
	- Returns the original value of passed parameter
	- Set the new value of passed parameter to ```true```

# Solution using test_and_set()

- Shared boolean variable ```lock```, initialised to ```false```
- Solution ->
![[Pasted image 20240418132015.png]]

# The compare_and_swap Instruction

The purpose of the compare_and_swap instruction is to atomically compare the value of a memory location with an expected value and, if they are equal, update the value of the memory location to a new value. This operation is performed atomically without interruption from other threads or processes, ensuring consistency and providing a way to implement lock-free data structures and algorithms.

The basic functionality of the compare-and-swap instruction can be summarised as follows:

1. Read the current value of a memory location.
2. Compare the read value with an expected value.
3. If the read value matches the expected value, update the memory location with a new value.
4. Return a boolean indicating whether the operation was successful (true if the update was performed, false otherwise).

- Definition
![[Pasted image 20240418132054.png]]

- Properties
	- **Executed atomically**
	- Returns the original value of passed parameter ```value```
	- Set the variable ```value``` the value of the passed parameter ```new_value``` but only if ```*value == expected``` is true. That is, the swap takes place only under this condition

# Solution using compare_and_swap

- Shared integer ```lock```  initialised to 0;
- Solution: 
![[Pasted image 20240418132417.png]]

# Atomic Variables

- Typically, instructions such as compare_and_swap are used as building blocks for other synchronisation tools
- One tool is an **atomic variable** that provides *atomic* (uninterruptable) updates on basic data types such as integers and booleans
- For example:
	- Let ```sequence```  be an atomic variable
	- Let ```increment()``` be operation on the atomic variable ```sequence```
	- The command:
			```increment(&sequence);```
			Ensures that ```sequence``` is incremented without interruption

# Atomic Variables

- The ```increment()``` function can be implemented as follows:
![[Pasted image 20240418132743.png]]


### Note

- Hardware-based solutions to the critical section problem like ```test_and_set()``` and ```compare_and_swap``` are typically inaccessible to programmers
- Programmers are provided the functionality by the OS to solve the critical section problem

# Mutex Locks

A mutex (short for mutual exclusion) lock is a synchronisation primitive used in concurrent programming to <u>protect shared resources such as variables, data structures, or critical sections of code from simultaneous access by multiple threads or processes</u>. The purpose of a mutex lock is to <u>enforce mutual exclusion, ensuring that only one thread can access the protected resource at any given time</u>, thus preventing data corruption and race conditions.

Mutex locks operate on the principle of ownership: a thread acquires the lock before accessing the protected resource and releases the lock once it's done, allowing other threads to acquire it. While a thread holds the lock, it has exclusive access to the resource, and other threads attempting to acquire the lock are typically blocked or made to wait until it becomes available.

- Previous solutions are complicated and generally inaccessible to application programmers
- OS designers build software mechanisms to solve the critical section problem

- Simplest is mutex lock
	- Boolean variable indicating if lock is available or not
- Protect a critical section by
	- First ```acquire()``` a lock (**and blocks until it can**)
	- Then ```release()``` the lock
- Where calls to ```acquire()``` and ```release()``` are **atomic**
- But this (our previous) solutions requires **busy waiting**
	- This lock therefore called a **spinlock**
			```while (test_and_set(&lock));```

	# Solution to CS Problem using Mutex Locks
	![[Pasted image 20240418133713.png]]

# Semaphore

Semaphores are a synchronisation primitive used in concurrent programming and operating systems to control access to shared resources, coordinate the execution of multiple threads or processes, and prevent race conditions.

A semaphore is essentially a counter (non-negative integer) that can be accessed and modified atomically. It supports two primary operations: "wait" (also known as "P" or "down") and "signal" (also known as "V" or "up"). These operations are typically used to control access to shared resources and ensure exclusive access or coordinated execution among multiple threads or processes.

- **Wait Operation (P or down)**: It's like trying to get in line for something. If there are available spots (semaphore value > 0), you can go ahead and take one. If not, you have to wait until there's an open spot.
    
- **Signal Operation (V or up)**: It's like announcing you're done with something and leaving. When you finish with a spot, you let others know it's available by signaling. This might allow someone waiting to take your spot.

- Synchronisation primative that provides more sophisticated ways (than Mutex locks) for processes to synchronise their activities
- Semaphore **S** - integer variable
- Can only be accessed via two indivisible (**atomic**) operations
	- ```wait()``` and ```signal()```
		- Originally ```P()``` and ```V()```
- Definition of the ```wait() operation```
	![[Pasted image 20240418134024.png]]
- **Binary semaphore** - integer value can range only between 0 and 1
	- Like as a **mutex lock**
- **Counting semaphore** - integer value can range over an unrestricted domain
- Implement a counting semaphore ***S*** as a binary semaphore e.g S=1
- With semaphore we can solve various synchronisation problems

# Semaphore usage Example

- Solution to the CS Problem
	- Create a semaphore **"mutex"** initialised to 1
		```wait(mutex);```
			```CS```
		```signal(mutex)```

- Consider **P1** and **P2** that with 2 statements **S1** and **S2** and the requirement that **S1** to happen before **S2**
	- Create a semaphore **"synch"** initialised to 0
		![[Pasted image 20240418134437.png]]

# Semaphore Implementation

- Guarantee that no 2 processes can execute the ```wait()``` or ```signal()``` on the same semaphore at the same time
- We use the OS implementation as there's a problem where the ```wait``` and ```signal``` code and their critical section
- What about **busy waiting** in critical section implementation?
	- Maybe little busy waiting if critical section rarely occupied
- Note that applications may spend lots of time in critical sections and therefore **this is not a good solution**
# Semaphore Implementation with no busy waiting

- With each semaphore there is an associated **waiting queue**
- Each **waiting queue** has 2 data items:
	- Value (of type integer)
	- Pointer to next record in the list

- Two operations:
	- **a way to Block** - place the process invoking the operation on the appropriate waiting queue
	- **a way to Wakeup** - remove one of the processes in the waiting queue and place it in the ready queue

# No busy waiting
![[Pasted image 20240418134949.png]]

# Problems with Semaphores

- Incorrect use of semaphore operations
	- ```signal(mutex), wait(mutex)```
	- ```wait(mutex), wait(mutex)```
	- Omitting of ```wait(mutex)``` and/or ```signal(mutex)```

 - These and others are examples of what can occur when semaphores and other synchronisation tools are used incorrectly
# Liveness

- **Liveness** refers to a set of properties that a system must satisfy to ensure processes make progress
- Processes should not have to **wait indefinitely** while trying to acquire a synchronisation tool such as a mutex lock or semaphore
- Waiting indefinitely violates the progress and bounded-waiting criteria discussed earlier
- Indefinite waiting is an example of a liveness failure

	- **Deadlock** -> two or more processes are waiting indefinitely for an event that can be caused by only one of the waiting processes
	- Let *S* and *Q* be two semaphores initialised to 1
	![[Pasted image 20240418135441.png]]

- Consider if P0 executes wait(S) and P1 wait(Q). When P0 executes wait(Q), it must wait until P1 executes signal(Q)
- However, P1 is waiting until P0 executes signal(S)
-  Since these signal() operations will never be executed, P0 and P1 are **deadlocked**

# Classical Problems of Synchronisation

- Classical problems used to test newly-proposed synchronisation schemes
	- Bounded-buffer problem (aka producer-consumer problem)
	- Readers and writers problem
	- dining-philosophers problem


# Bounded-buffer/Producer Consumer

- ***n*** buffers, each can hold one item
- Semaphore ```mutex``` initialised to the value of 1
- Semaphore ```full``` initialised to the value of 0
- Semaphore ```empty``` initialised to the value of n
- Structure of the producer process ->
	![[Pasted image 20240418135916.png]]

- Structure of the consumer process
	![[Pasted image 20240418135932.png]]

	![[Pasted image 20240418135940.png]]

# Semaphore (2)

![[Pasted image 20240418140000.png]]

# Readers-Writers Problem

- A data set is shared among a number of concurrent processes
	- **Readers** - only read the data set; they do ***not*** perform any updates (**hence multiple readers at the same time is okay**)
	- **Writers** - can both read and write (**but only one writer at a time, and no readers when writing**)
- Problem - allow multiple readers to read at the same time
	- Though... only 1 single writer can access the shared data at the same time
- Several variations of how readers and writers are considered - all involve some form of priorities
	- First, second, and third
- Shared data
	- Data set
	- Semaphore ```rw_mutex``` initialised to 1
	- Semaphore ```mutex``` initialised to 1 (**used to protect read_count**)
	- Integer ```read_count``` initialised to 0 (**how many are reading - and needs to be recorded**)
- Structure
		![[Pasted image 20240418140407.png]]
- Structure of a reader process
		![[Pasted image 20240418140431.png]]

# Readers-Writers Problem Variations

- The solution in previous slide can result in a situation where a writer process **never writes**. It is referred to as the "First reader-writer" problem
- The "Second reader-writer" problem is a variation the first reader-writer problem that states:
	- Once a writer is ready to write, no "newly arrived reader" is allowed to read
- Both the first and second may result in starvation, leading to even more variations
- Problem is solved on some systems by kernel providing reader-writer locks

# Reader-Writers Problem

- Several variations of how readers and writers are considered - all involve some form of priorities:
	- **First - readers-writers problem**: No reader should be kept waiting, unless a writer has **already** obtained permissions (readers-preference)
	- **Second - readers-writers problem**: If a writer is ready to write, it writes as soon as possible i.e takes priority over waiting readers (writers-preference)
	- **Third**: That a reader nor writer shall starve (because 1 & 2 could create starvation)


# Dining-Philosophers Problem

- N philosophers' sit at a round table with a bowl of rice in the middle
- They spend their lives alternating between thinking and eating
- They do not interact with their neighbours
- Occasionally try to pick up 2 chopsticks (one at a time) to eat from bowl
	- Need both to eat, then release both when done
- In the case of 5 philosophers, the shared data
	- Bowl of rice ( data set )
	- Semaphore chopstick 5 initialised to 1

# Dining-Philosophers Problem Algorithm

- Semaphore Solution
- The structure of Philosopher i:
	![[Pasted image 20240418150905.png]]


# POSIX Synchronisation

- POSIX API provides
	- mutex locks
	- semaphores
- Widely used on Linux, UNIX, and macOS


# POSIX Mutex Locks

- Creating and initialising  the lock
		![[Pasted image 20240418151607.png]]
- Acquiring and releasing the lock
		![[Pasted image 20240418151627.png]]
# POSIX Unnamed Semaphores

- Creating and initialising the semaphore

	![[Pasted image 20240418151700.png]]

- Acquiring and releasing the semaphore:
	![[Pasted image 20240418151733.png]]

# POSIX <u>NAMED</u> Semaphores

- Creating and initialising the semaphore
	![[Pasted image 20240418151806.png]]

- Another process can access the semaphore by referring to its name ```SEM```
- Acquiring and releasing the semaphore:
	![[Pasted image 20240418151841.png]]

# Linux Synchronisation

- Atomic variables
	```atomic-t``` is the type for atomic integer
- Consider the variables

	```atomic_t counter;```
	```int value;```
	![[Pasted image 20240418151944.png]]

# Deadlocks


# Deadlocks with Semaphores

- Data
	- A semaphore ```S1``` initialised to 1
	- A semaphore ```S2``` initialised to 1
- Two processes P1 and P2
	![[Pasted image 20240418152108.png]]


# System Model

- System consists of resources
- Resource types R1, R2,....Rm
	- *CPU cycles, memory space, I/O devices*
- Each resource type Ri has Wi instances.
- Each process
	- **request**
	- **use**
	- **Release**
	- **Claim**


# Resource Allocation Graph

- A set of vertices *V* and a set of edges *E*
	- *V* is partitioned into two types:
		- T = {T1, T2,... Tn}, the set consisting of all the processes in the system
		- R = {R1, R2, ... Rm}, the set consisting of all resource types in the system
	- **Request edge** - directed edge Ti -> Rj  *Ti requesting Rj*
	- **Assignment Edge** - directed edge Rj -> Ti *Rj allocated to Ti*

# Deadlock Characterisation

- Deadlock can arise if **4 conditions hold simultaneously**
	-  **Mutual Exclusion:** Only one process at a time can use a resource (non sharable e.g writing to a file)
		- Another process must wait for it
	- **Hold and wait**: A process **must be holding at least one resource, and** is **waiting** to acquire additional resources held by other processes
	-  **No preemption:** a resource can be released only voluntarily by the process holding it, after that process has completed its task i.e **resources cannot be preempted**
	- **Circular wait:** there exists a set {P0, P1,... Pn} of waiting processes such that P0 is waiting for a resource that is held by P1, P1 is waiting for a resource that is held by P2,..., Pn-1 is waiting for a resource that is held by Pn and Pn is waiting for resource that is held by P0.

# Resource Allocation Graph Example

- One instance of R1
- Two instances of R2
- One instance of R3
- Three instance of R4
- T1 holds one instance of R2 and is waiting for an instance of R1
- T2 holds one instance of R1, one instance of R2, and is waiting for an instance of R3
- T3 holds one instance of R3
	![[Pasted image 20240418153617.png]]

# Graph with a Cycle but **no Deadlock**
![[Pasted image 20240418153734.png]]

# Basic Facts

- If graph contains no cycles => no deadlock
- If graph contains a cycle =>
	- If only one instance per instance type, then deadlock
	- If several instances per resource type, **possibility** of deadlock


# Methods for Handling Deadlocks

- Ensure that the system will **never** enter a deadlock state:
	- **Deadlock prevention** (invalidate one of the 4 conditions for deadlock)
	- **Deadlock avoidance** - determine before granting a resource whether it may lead to deadlock e.g Banker's algorithm

- Could also do things like...
	- Allow the system to enter a deadlock state and then recover
	- Ignore the problem and pretend that deadlocks never occur in the system
		- **Often this is what happens... "it froze so we rebooted it"**

# Deadlock Prevention

- Invalidate one of the four necessary conditions for deadlock:
	- **Mutual Exclusion** - not required for sharable resources (e.g read-only files); must hold for non-sharable resources
			- Some resources are intrinsically non-sharable
	- **Hold and  Wait** - Must guarantee that whenever a process requests a resource, it doesn't hold any other resources
		- Require process to request and be allocated all its resources before it begins execution /// allow process to request resources only when the process has none allocated to it
		- Low resource utilisation; <u>starvation</u> possible
- **No preemption**:
	- If a process that is holding some resources requests another resource that cannot be immediately allocated to it, then all resources currently being held together are released
	- Process will be restarted only when it can regain its old resources, as well as the new ones that is requesting
- **Circular wait:**
	- Impose a total ordering of all resource types, and require that each process requests resources in an increasing order of enumeration (responsibility of the programmer)

# Deadlock with Semaphores

![[Pasted image 20240418155300.png]]


# Circular Wait

- Invalidating the circular wait condition is most common
- Simply assign each resource (i.e mutex locks) a unique number
- Resources must be acquired in order
- If
	```first_mutex = 1```
	```second_mutex = 5```
	code for ```thread_two``` could not be written as follows
![[Pasted image 20240418155449.png]]

- That's deadlock prevention
- Let's look at deadlock avoidance

# Deadlock Avoidance

- Requires that the system has some sort **a priori** information available
	- Simplest and most useful model requires that each process declare the ***maximum number*** of resources of each type that it may need
	- The **deadlock-avoidance algorithm** dynamically examines the resource-allocation **state** <u>to ensure that there can never be a circular-wait condition</u>
	- Resource-allocation *state* is defined by the number of available and allocated resources, and the maximum demands of the process

# Basic Facts

- If a system is in a safe state => no deadlocks
- If a system is in an unsafe state => possibility of deadlock
- Avoidance => ensure that a system will never enter an unsafe state
	![[Pasted image 20240418155840.png]]


# Safe State

1. **Define System State**: Know what resources are allocated to processes.
    
2. **Check Resource Availability**: Ensure there are enough resources for each process to run without deadlock.
    
3. **Use Algorithms**: Employ methods like the Banker's Algorithm to simulate resource allocation and check for a safe state.
    
4. **Implement Policies**: Enforce resource allocation rules to prevent deadlock situations.
    
5. **Monitor Continuously**: Keep an eye on resource usage to adapt policies and maintain system safety.

- When a process requests an available resource, system must decide if immediate allocation leaves the system in a **safe state**
- System is in a **safe state** if there exists a sequence {P1, P2... Pn} of ALL the processes in the system such that for each Pi, the resources that Pi <u>can still request</u> can be satisfied by currently available resources + resources held by all the Pj, with j < i
	- e.g if we know P3 might need X resources, and Y resources are already in use (by P0, P1, P2), are X+Y <= Available?
- So:
	- If Pi resource needs are not immediately available, then Pi waits until all Pj have finished (resource becomes available)
	- When Pj is finished, Pi can obtain needed resources, execute return allocated resources, and terminate
	- When Pi terminates, P(i+1) can obtain its needed resources, and so on

# Avoidance Algorithms

- Single instance of a resource type
	- Use a resource-allocation graph
- Multiple instances of a resource type
	- Use the Banker's Algorithm
![[Pasted image 20240418160501.png]]



# Resource-Allocation Graph Scheme

- **Claim edge** Pi -> Rj indicated that process Pj may request resource Rj; represented by a dashed line
	-  Refers to a dependency relationship between two processes or threads, where one entity claims a resource before another can proceed.
- Claim edge converts to request edge when a process requests a resource
- Requests edge converted to an assignment edge when the resource is allocated to the process
- When a resource is released by a process, assignment edge reconverts to a claim edge
- Resources **must be claimed a priori** in the system

# Unsafe State In Resource-Allocation Graph
![[Pasted image 20240418212906.png]]

# Unsafe State in Resource-Allocation Graph

![[Pasted image 20240418213233.png]]

# Safe, Unsafe, Deadlock State

![[Pasted image 20240418213252.png]]

# Resource-Allocation Graph Algorithm

- Suppose that process Pi requests a resource Rj
- The request can be granted only if converting the request edge to an assignment edge does not result in the formation of a cycle in the resource allocation graph

# Banker's Algorithm

- <u>Multiple</u> instances of resources
- Each process must a priori claim <u>maximum</u> use
- When a process requests a resource, it may have to wait

Imagine you're a bank manager and you have a bunch of customers (processes) who want loans (resources) from your bank. Each customer has a list of how much of each type of loan they need before they can finish their project.

Here's how it works:

1. **Customers Apply for Loans**: Customers come to your bank and apply for loans. They tell you how much of each type of loan they need to complete their project.
    
2. **Check if You Can Grant the Loan**: Before you approve any loan, you check if you have enough of each type of loan available in the bank. If you have enough, great! You can grant the loan. If not, you might have to tell the customer to wait until you have enough resources available.
    
3. **Keep Track of Loans**: As you approve loans, you keep track of how much of each type of loan is left in the bank. You don't want to give out so many loans that you run out of resources for other customers.
    
4. **Avoid Deadlocks**: You want to make sure that you can always give out loans without causing a situation where everyone is waiting for a loan from the bank and no one can finish their projects. So, you carefully manage the loans you give out to avoid deadlocks.
    
5. **Keep the Bank Safe**: Your goal is to keep your bank in a safe state where you can always grant loans without causing a deadlock. You use the Banker's Algorithm to manage your resources efficiently and avoid deadlock situations.

# Data Structures for the Banker's Algorithm

- Let ***n*** = number of processes, and ***m*** = number of resource types
	- **Available:** Vector of length *m*. If available [j] = k, there are *k* instances of resource type Rj available
	- **Max:** *n* x *m* matrix. If Max[i, j] = k, then process Pi may request at most *k* instance of resource type Rj
	- **Allocation:** *n* x *m* matrix. If Allocation[i,j] = k, then Pi is currently allocated *k* instances of Rj
	- **Need:** *n* x *m* matrix. If Need[i, j] = k, then Pi may need *k* more instances of Rj to complete its task
		Need [i,j] = Max[i,j] – Allocation [i,j]

# Safety Algorithm

1. Let ***Work*** and ***Finish*** be vectors of length *m* and *n*, respectively.
	Initialise:
		***Work = Available (what's currently available)***
		***Finish[i] = false for i = 0, 1, ...., n - 1***
2. Find an ***I (process)*** such that both:
	a) ***Finish [i] = false***
	b) ***Need(i) <= Work*** <- *Find a process that we've resources for*
		if **no** such *i* exists, go to step 4
3. ***Work = Work + Allocation(i)***
	***Finish[i] = true***
		go to step 2
4. If ***Finish[i] == true*** for all ***i***, then the system is in a safe state

# Resource-Request Algorithm for Process Pi 

***Request(i)*** = request vector for process ***Pi***. If ***Request(i)[j] = k*** then process ***Pi*** wants ***k*** instances of resource type ***Rj***`
1. If ***Request(i) <= Need(i)*** go to step 2. Otherwise, raise error condition, since process has **exceeded** its **maximum claim**
2. If ***Request(i) <= Available***, go to step 3. Otherwise, ***Pi***, **must wait**, since resources are not available
3. **Pretend** to allocate requested resources to ***Pi*** by modifying the state as follows:
		***Available = Available - Request(i)***
		***Allocation(i) = Allocation(i)  + Request(i)***
		***Need(i) = Need(i) - Request(i)***
4. ***We run our safety algorithm***
	- If safe => the resources are allocated the ***Pi*** 
	- If unsafe => ***Pi*** must wait, and the old resource-allocation state is restored

# Example of Banker' s Algorithm

- 5 processes *P0* through *P4*
	3 resource types
		![[Pasted image 20240418215134.png]]
- Snapshot at time *T0*:
	![[Pasted image 20240418215329.png]]
- The context of the ***Need*** is defined to be ***Max - Allocation***

	![[Pasted image 20240418215424.png]]

# Example: *P1* Request (1, 0, 2)

- Check that Request <= Available, that is (1, 0, 2) <= (3, 3, 2) => true
		![[Pasted image 20240418215922.png]]
- Executing safety algorithm shows that sequence ***<P1, P3, P4, P0, P2>*** satisfies safety requirement

# Example of Banker's Algorithm

- 5 processes *P0* through *P4*:
	3 resource types:
		![[Pasted image 20240418220640.png]]
- **Can request for (0, 2, 0)** by P0 be granted? 
- Snapshot at time *T0*:
	![[Pasted image 20240418220736.png]]

# Example of Banker's Algorithm
![[Pasted image 20240418220759.png]]

# Deadlock detection

- We looked at prevention and avoidance... But perhaps we didn't prevent or avoid deadlock, and just let it happen
- Allow system to enter deadlock state
- Use a detection algorithm to know when this happens, and then
- Recovery scheme

# Single Instance of Each Resource Type

- Maintain **wait-for** graph
	- Nodes are processes
	- ***Pi -> Pj*** if ***Pi*** is waiting for ***Pj***
- Periodically invoke an algorithm that searches for a **cycle** in the graph
	![[Pasted image 20240418221425.png]]

# Several Instances of a Resource Type

- **Available:** A vector of length **m** indicates the number of available resources of each type
- **Allocation:** An ***n*** x ***m*** matrix defines the number of resources of each type currently allocated to each process
- **Request:** An ***n*** x ***m*** matrix indicates the current request of each process. If ***Request```[i][j]``` = k***, then process ***Pi*** is requesting ***k*** more instances of resource type ***Rj***

# Detection Algorithm

1. Let ***Work*** *and* ***Finish*** be vectors of length ***m*** and ***n***, respectively
	Initialise:
		a) ***Work = Available***
		b) For ***i*** **= 1, 2, ..., n**   if ***Allocation(i) != 0***, then
			***Finished```[i]``` = false***; otherwise ***Finish```[i]``` = true***
2. Find an index ***i*** such that both:
	a) ***Finished```[i]```*** **== false**
	b) ***Request(i) <= Work***
	If no such ***i*** exists, go to step 4
3. ***Work = Work + Allocation***
	***Finish```[i]```= true***
	go to step 2
4. If ***Finish```[i]```*** == false, for some ***i***, 1 <= ***i*** <= ***n***, then the system is in deadlock state. Moreover, if ***Finish```[i]``` == false***, then ***Pi*** is deadlocked

# Example of Detection Algorithm

- Five processes ***P0*** through ***P4***; three resource types A (7 instances), B (2 instances), and C (6 instances)
- Snapshot at time ***T0***:
	![[Pasted image 20240418222716.png]]
- Sequence ***<P0, P2, P3, P1, P4>*** will result in ***Finish```[i]```= true*** for all ***i***
- ***P2*** requests an additional instance of type ***C***

![[Pasted image 20240418223014.png]]

![[Pasted image 20240418223028.png]]


# Recovery from Deadlock: **Process Termination**

- Abort all deadlocked processes
- Abort one process at a time until the deadlock cycle is eliminated
- In which order should we choose to abort?
	1. Priority of the process
	2. How long process has computed, and how much longer to completion
	3. Resources the process has used
	4. Resources process needs to complete
	5. How many processes will need to be terminated
	6. Is process interactive or batch?
	7. ....

# Recovery from Deadlock: Resource Preemption

- **Selecting a victim** - minimise cost
- **Rollback** - return to some safe state, restart process for that state (not always possible though, programs affect real world)
- **Starvation** - same process may always be picked as victim, include number of rollback in cost factor
- Messy....