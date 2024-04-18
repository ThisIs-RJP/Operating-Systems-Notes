
# Background

- Processes can execute concurrently/in parallel
	- May be <u>interrupted</u> at any time, partially completing execution
- Concurrent access to shared data may result in data inconsistency
- So, maintaining data consistency requires mechanisms to ensure the orderly execution of cooperating processes
- We illustrated this simple example before... which lead to a race condition

# Race Condition

- Processes P0 and P1 are creating child processes using the ```fork()``` system call
- Race condition on kernel variable ```next_available_pid``` which represents the next available process identifier (pid)
![[Pasted image 20240418122822.png]]

- Unless there is a mechanism to prevent P0 and P1 from accessing the variable ```next_available_pid```, the same pid can be assigned to two different processes!

# Critical Section Problem

- Consider system of ***n*** processes {*p0*, *p1*, ... *p(n -1)*}
- Each process has **critical section** segment of code
	- Process may be changing common variables, updating table, **writing file**, etc
	- When one process in critical section, **no other may be in its critical section**
- ***Critical section problem*** is design **protocol to solve this**
- Often structured: Each process must ask permission to enter critical section in **entry section**, may follow critical section with **exit section**, then **remainder section**
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

- Definition
![[Pasted image 20240418132054.png]]

- Properties
	- **Executed atomically**
	- Returns the original value of passed parameter ``value```
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
	- **Hold and  Wait** - Must guarantee that whenever a process requests a resource, it does hold any other resources
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
- Claim edge converts to request edge when a process requests a resource
- Requests edge converted to an assignment edge when the resource is allocated to the process
- When a resource is released by a process, assignment edge reconverts to a claim edge
- Resources **must be claimed a priori** in the system