# Process Concept
- An OS executes a variety of programs that run as processes
- **Process**- A program in execution; process execution mush progress in **sequential** fashion IE <u>No parallel execution of instructions of a single process</u>
- Multiple parts (a process in memory)
	- The program code, also called **text section**
	- Current **activity** including **program counter**, processor registers
	- **Stack** containing temporary data
		- Function parameters, return addresses, local variables
	- **Data section** containing global variables
	- **Heap** containing memory dynamically allocated during run time
- Program is **passive** entity stored on disk (**executable file**); process is **active**
	- Program becomes a process when an executable file is loaded into memory
- Execution of program started via GUI mouse clicks, command line entry of its name
- One program can be several processes
	- Consider multiple users executing the same program

# Memory Layout of a C Program
![[Pasted image 20240416124345.png]]

# Process in Memory
![[Pasted image 20240416124404.png]]

# Process State
- **New:** The process is being created
- **Running**: Instructions are being executed
- **Waiting:** The process is waiting for some event to occur
- **Ready:** The process is waiting to be assigned to a processor
- **Terminated:** The process has finished execution

![[Pasted image 20240416125306.png]]

Note you can use ```top``` to what processes are running

# Process Control Block (PCB)

### Task Control Block
Information associated with each process
- Process state - running, waiting etc
- Program counter - location of instruction to next execute
- CPU registers - Contents of all process-centric registers
- CPU scheduling information - priorities, scheduling queue pointers
- Memory-management information - memory allocated to the process
- Accounting information - CPU used, clock time elapsed since start, time limits
- I/O status information - I/O devices allocated to process, list of open files

# Process Scheduling

- **Process scheduler** selects among available processes for next execution on CPU core
- Goal - Maximise CPU use, quickly switch processes onto CPU core
- Needs to maintain **scheduling queues** of processes
	- **Ready queue** - set of all processes residing in main memory, ready and waiting to execute
	- **Wait queues** - set of processes waiting for an event e.g I/O
	- Processes migrate among the various queues

# Ready and Wait Queues
![[Pasted image 20240416125807.png]]


# Representation of Process Scheduling

![[Pasted image 20240416125835.png]]

# CPU Switch From Process to Process
A **context switch** occurs when the CPU switches from one process to another
![[Pasted image 20240416125930.png]]

# Context Switch

- **Context switch:** 
	- When CPU switches to another process, the system must **save the state** of the old process and load the **saved state** for the new process
- And the **Context** of a process represented in the PCB
- Context-switch time is <u>pure overhead</u>; **the system does no useful work while switching**
	- The more complex the OS and the PCB -> the longer the context switch

# Process Creation

- **Parent process(es)** create **children** processes, which, in turn create other processes, forming a **tree** of processes
- Processes are identified and managed via a **process identifier** (PID)

# Process Creation

- **When we create a new process, some choices might need to be made**

- **Resource sharing options**
	- Should...
		- Parent and children share all resources?
		- Children share subset of parent's resources
		- Parent and child share no resources

- **Execution Options**
	- Parent and children execute concurrently?
	- Parent waits until children terminate?

# Processes and Trees of Processes
- The bash shell is a program, so it has a PID (process id)
- We can find the PID of the current bash instance we are using by typing ```echo $$``` at the prompt
	- For other processes type ```ps```
	- For all processes type ```ps -A```
- For example, if I used multiprocessing in Python to start 10 processes
![[Pasted image 20240416132252.png]]

![[Pasted image 20240416132310.png]]

After running ```top```
![[Pasted image 20240416132336.png]]

I could also look at output from ```pstree -p <pid>```
- In our case PID = 17822
![[Pasted image 20240416132425.png]]


# A Tree of Processes in Linux
![[Pasted image 20240416132454.png]]

After running ```pstree -p 1```
![[Pasted image 20240416132520.png]]


# Process Creation
- When a process is created... **Address space**
	- Child is duplicate of parent
	- Child has a program loaded into it
- UNIX examples
	- ```fork()```system call wrapper (uses ```clone()```) to create new processes
		- ```fork()``` does a "copy" of heap, stack etc but the copy is not realised (because it's not used)
		- A mechanism called **copy on write & paging** (covered later) handles this
	- ```exec()```system call wrapper used after a ```fork()```to replace the process' memory space with a new program
	- Parent process calls ```wait()```waiting for the child to terminate
![[Pasted image 20240416132727.png]]

# C Program Forking Separate Process

# Process Termination
- Process executes last statement and then asks the OS to delete it using the ```exit()```system call
	- Returns status data from child to parent (via ```wait()```)
	- Process' resources are deallocated by OS
- Parent may terminate the execution of children processes using the ```kill()```system call. Some reasons for doing so are:
	- Child has exceeded allocated resources
	- Task assigned to child is no longer required
	- The parent is exiting, and the OS does not allow a child to continue if its parent terminates
- Some OS configs do not allow child to exist if its parent has terminated. if a process terminates, then all its children must also be terminated
	- **Cascading termination** -> All children, grandchildren, etc are terminated
	- The termination is initiated by the OS
- The parent process may wait for termination of a child process by using the ```wait()```system call. The call returns status info and the PID of the terminated process
	- ```pid = wait(&status)```
- If parent terminated without invoking ```wait()```, process is an **orphan**
	- The process can still be doing something
- Sometimes a process has finished running and called ```exit()```, but still has a PCB -> It's a **zombie**
	- OS can clean this up

# If I kill a process which process is the new parent?
![[Pasted image 20240416133504.png]]

# We can also kill a process group (parent, children...)

![[Pasted image 20240416133542.png]]

# Multiprocess Architecture

- Many web browsers in the past were single process (maybe some still do?)
	- If one web site causes trouble, entire browser can hang or cash
- Google Chrome Browser is multiprocess with 3 different types of processes:
	- **Browser** process manages UI, disk and network I/O
	- **Renderer** process renders web pages, deals with HTML, JavaScript. A new renderer created for each website opened
		- Runs in **sandbox** restricting disk and network I/O, minimising effect of security exploits
	- **Plug-in** process for each type of plug-in

# Interprocess Communication

- Processes within a system may be **independent** or **cooperating**
- **Cooperating** processes can affect or be affected by other processes, including shared data
- Reasons for cooperating processes:
	- Information sharing
	- Computation speed
	- Modularity
- Cooperating processes need **interprocess communication (IPC)**
- Two (primary) models of IPC
	- **Shared memory**
	- **Message passing**

# Communications Model
![[Pasted image 20240416134143.png]]

# Producer-Consumer Problem

- ***Producer process*** *produces* information that is *consumed* by a *consumer* process. There is also a <u>buffer</u> they share
- Two variations
	- **Unbounded-buffer** places no practical limit on the size of the buffer:
		- Producer never waits
		- Consumer waits if there is no buffer to consume
	- **Bounded-buffer** assumes that there is a fixed buffer size
		- Producer must wait if all buffers are full
		- Consumer waits if there is no buffer to consume
### We'll use the bounder-buffer producer-consumer problem to explain these methods for the IPC

# IPC - Shared Memory
### Solution 1 to the bounded-buffer problem using <u>Shared Memory for IPC</u>

- An area of memory shared among the processes that wish to communicate
- The communication is <u>under the control of the user processes</u> not the OS
- Major issue is to provide mechanism that will allow the user processes to <u>synchronise their actions</u> when they access shared memory

# Bounded-Buffer Shared Memory Solution
![[Pasted image 20240416134718.png]]

# Consumer **Process - Shared Memory**
![[Pasted image 20240416134754.png]]

# Producer Process

![[Pasted image 20240416134843.png]]

# What about Filling all the Buffers

- Suppose that we wanted to provide a solution to the consumer-producer problem that <u>fills all the buffers</u>
- We can do so by having an integer ```counter```that keeps track of the number of full buffers
- Initially, ```counter = 0```
- The integer ```counter```is incremented by the producer after it produces a new buffer
- The integer ```counter```is and is decremented by the consumer after it consumes a buffer

# Consumer 
![[Pasted image 20240416135030.png]]

# Producer
![[Pasted image 20240416135040.png]]

# IPC - Message Passing
### Solution 2 to the bounded-buffer problem using IPC & Message Passing

- Processes can communicate with each other without resorting to shared variables
- IPC (Message Passing) facility provides two operations:
	- ```send(message)```
	- ```receive(message)```
- The *message* size is either fixed or variable
- If processes **P** and **Q** wish to communicate, they need to:
	- Establish a **communication link** between them
	- Exchange messages via **send/receive**
- (some) Implementation considerations:
	- Can a link be associated with more than **two processes?**
	- How many links can there be **between every pair** of communicating processes
	- What is the capacity of a link?
	- Will the size of a message that the link can accommodate fixed or variable?
	- Will a link be **unidirectional** or **bi-directional?**
# Direct / Indirect Communication

- **Direct** - Processes must name each other <u>explicitly</u>:
	- ```send(P, message)``` - send a message to process P
	- ```receive(Q, message)```- receive a message from process Q

- **Indirect** - Messages are directed and received from **mailboxes** (also referred to as ports) - basically some intermediary entity:
	- Each **mailbox** has a unique ID
	- Processes can communicate only if they share a mailbox

# Synchronisation

- Message passing may be either blocking or non-blocking
	- **Blocking** is considered **synchronous**
		- Blocking ```send()```- the sender is blocked until the message is received
		- Blocking ```receive()```- the receiver is blocked until a message is available
	- **Non-blocking send** - the sender sends the message and continues
	- **Non-blocking receive** - the receiver receives:
		- A valid message or
		- Null message
	- Different combinations possible
		- If both send and receive block, we have a **rendezvous**

# Producer-Consumer: Message Passing
![[Pasted image 20240416222313.png]]

# (considerations) Buffering

- Queue of messages attached to the link
- Implemented in one of 3 ways
	- 1. Zero Capacity - no messages are queued on a link. Sender must <u>wait</u> for receiver (rendezvous)
	- 2. Bounded capacity - finite lengths of *n* messages. Sender must <u>wait</u> if link is full
	- 3. Unbounded capacity - infinite length. Sender never waits (but practically, what about RAM issues?)

# Examples of IPC systems - POSIX

- Let's look at setting up shared memory on Linux
	- In Linux, this is a file, but we map it into memory!
- POSIX shared memory
	- Process first created shared memory segment
		```shm = shm_open(name, O_CREAT | O_RDWR, 0666)```
	- Set the size of the object
		```ftruncate(shm_fd), 4096```
	- Use ```mmap()```to memory-map a file pointer to the shared memory object
	- Reading and writing to shared memory is done by using the pointer returned by ```mmap()```


# Pipes

- Acts as a **conduit** allowing two processes to communicate
- Configuration:
	- (typically) Communication is **unidrectional**
	- FIFO
	- ...
- **Ordinary pipes** - cannot be accessed from outside the process that created it. Typically, a parent process creates a pipe and uses it to communicate with a child process that it created
- **Named pipes** - can be accessed without a parent-child relationship

# Ordinary Pipes

- Ordinary Pipes allow communication in standard producer-consumer style
- Producer writes to one end (the **write-end** of the pipe)
- Consumer reads from the other end (the **read-end** of the pipe)
- Require parent-child relationship between communicating processes
- Windows calls these **anonymous pipes**

# Named Pipes

- Named Pipes are more powerful than ordinary pipes
- Communication can be birectional
- No parent-child relationship is necessary between the communicating processes
- Several processes can use the named pipe for communication
- Provided on both UNIX AND Windows systems
- Appears as a file, processes attach to it for IPC

# Communications in Client-Server Systems (Sockets)

- A **socket** is defined as an endpoint for communication
- Concatenation of IP address and **port** - a number included at start of message packet to differentiate network services on a host
- The socket **161.25.19.8:1625** refers to port **1625** on host **161.25.19.8**
- Communication consists between a pair of sockets
- All ports below 1024 are **well known**, used for standard services
- Special IP address 127.0.0.1 (**loopback**) to refer to system on which process is running

# Socket Communication
![[Pasted image 20240416225150.png]]

# Processes & **Threads**

# Benefits

- **Threads and processes are similar in that they allow concurrent/parallel execution of code (depending on OS & programming language**
	- **Responsiveness** - may allow continued execution if part of process is blocked, especially important for UI (**like processes**)
	- **Resource Sharing** - threads share resources of process, easier than shared memory or message passing (**unlike between child/parent/child processes**)
	- **Economy** - cheaper than process creation, thread switching lower overhead than context switching

# Single and Multithreaded Processes
![[Pasted image 20240416232705.png]]

# Multicore Programming
- **Multicore** or **multiprocessor** systems put pressure on programmers. challenges include:
	- **Dividing activities**
	- **Balance**
	- **Data splitting**
	- **Data dependency**
	- **Testing and debugging (e.g heisenbugs)**
- ***Parellelism*** implies a system can perform more than one task simultaneously
- ***Concurrency*** supports more than one task making progress
	- Single processor/core, scheduler providing concurrency

# Concurrency vs Parallelism
![[Pasted image 20240416233005.png]]

# Multicore Programming

- Types of parallelism
	- **Data parallelism** - distributes subsets of the same data across multiple cores, **same operation on each**
	- **Task parallelism** - distributing threads across cores, each thread performing a unique operation

![[Pasted image 20240416233150.png]]


# Amdahl's Law

- Identifies performance gains from adding additional cores to an application that has both serial and parallel components
- S is serial portion
- N processing cores
	- ![[Pasted image 20240416233311.png]]
- That is, if application is 75% parallel/ 25% serial, moving from 1 to 2 cores results in speedup of 1.6 times
- As *N* approaches infinity, speedup approaches 1/S
- **Serial portion of an application impacts performance gained by adding additional cores**

![[Pasted image 20240416233454.png]]

![[Pasted image 20240416233501.png]]
![[Pasted image 20240416233512.png]]

# User Threads and Kernel Threads
- **User threads** - management done by user-level threads library
- **Kernel threads** - **Supported** by the Kernel (meaning if you've 4 kernel threads, these could run on 4 cores e.g pthreads on POSIX UNIX)
- Virtually all general-purpose OS support kernel threads, including:
	- Windows
	- Linux
	- Mac OS X
	- iOS
	- Android

# Many-to-One

- Many user-level threads mapped to so single kernel thread
- One thread blocking causes all to block
- Multiple threads may not run in parallel on multicore system because <u>only one</u> may be in kernel at a time
- Few systems currently use this model
- Examples:
	- **Solaris Green Threads**
	- **GNU Portable Threads**

# One-to-One
- Each user-level thread maps to kernel thread
- Creating a user-level thread creates a kernel thread
- More concurrency that many-to-one
- Number of threads per process sometimes restricted due to overhead
- Examples
	- Windows
	- Linux

# Thread Libraries

- **Thread library** provides programmer with API for creating and managing threads
- Two primary ways of implementing
	- Library entirely in user space
	- Kernel-level library supported by the OS

# Pthreads

- A POSIX standard (IEEE 1003.1c) API for thread creation and synchronisation. It is a **specification**, not **implementation**
- API specifies behaviour of the thread library, implementation is up to the development of the library
- Common in UNIX OS (Linux & Mac OS X)

# Thread Cancellation

- Terminating a thread before it has finished
- Thread to be cancelled is **target thread**
- Two general approaches:
	- **Asynchronous cancellation** terminates the target thread immediately
	- **Deferred cancellation** allows the target thread to periodically check if it should be cancelled
- Pthread code to create and cancel a thread: 
	- ![[Pasted image 20240416234956.png]]

# Single and Multithreaded Processes

![[Pasted image 20240416235016.png]]


# Linux Process vs Threads

- Thread creation is done through ```clone()```system call
- ```clone()```allows a child task to share the address space of the parent task (process)
	- Example flags control behaviour 
	- Both ```fork()```and ```pthread_create()```use the clone system call
	- ![[Pasted image 20240416235202.png]]

# Signal Handling

- **Signals** are used in UNIX systems to notify a process that a particular event has occured
- A **signal handler** is used to process signals
	- 1. Signal is generated by particular event
	- 2. Signal is delivered to a process
	- 3. Signal is handled by one of two signal handlers:
		- 1. Default
		- 2. User-defined
- Every signal has a **default handler** that kernel runs when handling signal
	- **User-defined signal** can override default
	- For single-threaded, signal delivered to process
- When a signal is sent to a process, it interrupts its normal execution
	- Uses the default signal handler
	- Or uses a process defined signal handler
- use ```kill -l``` to see a list of available signals