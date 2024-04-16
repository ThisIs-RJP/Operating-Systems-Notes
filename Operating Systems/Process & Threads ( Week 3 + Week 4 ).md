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
e t

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
- *Producer* process *produces* information that is *consumed* by a *consumer* process. There is also a <u>buffer</u> they share
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