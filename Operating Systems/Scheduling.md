

# Diagram of Process State
![[Pasted image 20240417160028.png]]

# Representation of Process Scheduling
![[Pasted image 20240417160055.png]]

# CPU Switch From Process to Process
![[Pasted image 20240417160128.png]]

# Basic Concepts

- Obtain **Maximum** CPU utilisation with multi-programming
- CPU-I/O Burst Cycle - Process execution consists of a **cycle** of CPU execution and I/O wait
- **CPU burst** followed by **I/O burst**
- **CPU burst distribution is of main concern**

# Histogram of CPU-burst Times

![[Pasted image 20240417161237.png]]


# CPU Scheduler

- The **CPU scheduler** selects from among the processes in ready queue, and allocates a CPU core to one of them
	- Queue may be ordered in various ways
- CPU scheduling <u>decisions</u> may take place when a process:
	- 1. Switches from running to waiting state
	- 2. Switches from running to ready state
	- 3. Switches from waiting to ready
	- 4. Terminates
- For situations 1 and 4, there is no choice in terms of scheduling. A new process (if one exists in the ready queue) must be selected for execution
- For situations 2 and 3, however, there is a choice

# Preemptive and Non preemptive Scheduling

- When scheduling takes place only under circumstances 1 and 4, the scheduling scheme is **non preemptive**
	- Otherwise, it is preemptive
- Under **non preemptive scheduling**, once the CPU has been allocated to a process, the process keeps the CPU until it releases it either by **terminating** or by switching to the **waiting** state
- Virtually all modern OS's including Windows, MacOS, Linux and UNIX <u>use preemptive</u> scheduling algorithms

# Preemptive Scheduling and Race Conditions

- Preemptive scheduling can result in **race conditions** when data are shared among several processes
- Consider the case of two processes that share data. While one process is updating the data, it is preempted so that the second process can run. The second process then tries to read the data, which are in an inconsistent state

# Dispatcher

- **Dispatch latency** - time it takes for the dispatcher to stop one process and start another running
- **Dispatch involves** gives control of the CPU to the process selected by the CPU scheduler; this involves:
	- Switching context
	- Switching to user mode
	- Jumping to the proper location in the user program to restart that program

![[Pasted image 20240417162712.png]]

# Scheduling Criteria

- **CPU utilisation** - keep the CPU as busy as possible
- **Throughput** - Number of processes that complete their execution per time unit
- **Turnaround time** - amount of time to execute a particular process
- **Waiting time** - amount of time a process has been waiting in the ready queue
- **Response time** - amount of time it takes from when a request was submitted until the first response is produced

- Max CPU utilisation
- Max throughput
- Min turnaround time
- Min waiting time
- Min response time

# First come, first served (FCFS) Scheduling
![[Pasted image 20240417163028.png]]

![[Pasted image 20240417163142.png]]

# Shortest Job First (SJF) Scheduling

- Associate with each process the length of its next CPU burst
	- Use these lengths to schedule the process with the shortest time
- SJF is optimal - it gives min average waiting time for a given set of processes
- Preemptive version AKA **shortest-remaining-time-first**
- How do we determine the length of the next CPU burst?
	- Estimate

# Example of SJF

![[Pasted image 20240417163402.png]]

# Example of Shortest-remaining-time-first

![[Pasted image 20240417163453.png]]

# Round Robin (RR)

- Each process gets a small unit of CPU time (**time quantum** q), usually 10-100 ms. After this time has elapsed, the process is preempted and added to the end of the ready queue.
- If there are **n** processes in the ready queue and the time quantum is q, then each process gets **1/N** of the CPU time in chunks of at most q time units at once. No process waits more than **(n-1) * q** time units
- **Timer** interrupts every quantum to schedule next process
- Performance
	- *q* large => FIFO
	- *q* small => *q* must be large with respect to context switch, otherwise overhead is too high

- In Linux, there's a **scheduler_tick()** that gets called periodically

# Example of RR with Time Quantum = 4

![[Pasted image 20240417163911.png]]

- Typically, higher average turnaround than SJF, but better **response**
- *q* should be large compared to context switch time
	- *q* usually 10 ms to 100 ms
	- Context switch < 10 ms

# Turnaround Time varies with the Time Quantum
![[Pasted image 20240417164030.png]]


# Priority Scheduling

- A priority number (integer) is associated with each process
- The CPU is allocated to the process with the highest priority
	- Preemptive
	- Non preemptive

- SJF is *akin* priority scheduling where priority is the inverse of predicted next CPU burst time
- Problem = **Starvation** - low priority processes may never execute
- One Solution is **Ageing** - as time progresses, increase the priority of the process

# Example of Priority Scheduling

![[Pasted image 20240417164251.png]]

# Priority Scheduling w/ Round-Robin

![[Pasted image 20240417164334.png]]

# Multilevel Queue

- With priority scheduling, have separate queues for each priority
- Schedule the process in the highest-priority queue!

![[Pasted image 20240417164501.png]]

- Prioritisation based upon process time
![[Pasted image 20240417164559.png]]

# Multilevel Feedback Queue

- A process can move between the various queues
- **Multilevel-feedback-queue scheduler** defined by the parameters/features
	- Number of queues
	- **Different** scheduling algorithms for each queue
	- Method used to determine when to **upgrade** a process
	- Method used to determine which queue a process will enter when that process needs service
- Ageing can be implemented using multilevel feedback queue

# Example of Multilevel Feedback Queue
![[Pasted image 20240417164943.png]]


# Example of Multilevel Feedback Queue
![[Pasted image 20240417165300.png]]

# Multiple-Processor Scheduling

- CPU scheduling more complex when multiple CPU's are available
- Multiprocess may be any one of the following architectures:
	- Multicore CPU's
	- Multithreaded cores
	- NUMA systems
	- Heterogeneous multiprocessing
![[Pasted image 20240417165801.png]]

# Multiple Processor Scheduling - Processor Affinity

- When a thread has been running on one processor, the cache contents of that processor stores the memory accesses by that thread
- We refer to this as a thread having **affinity** for a processor (i.e "processor affinity")
- Load balancing may affect processor affinity as a thread may be moved from one processor to another to balance loads, yet that thread loses the contents of what it had in the cache of the processor it was moved off of
- **Soft affinity** - the OS attempts to keep a thread running on the same processor, but no guarantees
- **Hard affinity** - allows a process to specify a set of processor it may run on

# Multi Processor Scheduling

- Symmetric multiprocessing (SMP) environments
- All threads may be in a common ready queue (a)
- Each processor may have its own private queue of threads (b)

![[Pasted image 20240417170514.png]]


# Real-time CPU Scheduling

- Can present obvious challenges
- **Soft real-time systems** - Critical real-time tasks have the highest priority, but no guarantee as to when tasks will be scheduled
- **Hard real-time systems** - task must be serviced by its deadline


# Real-Time CPU Scheduling

- Event latency - the amount of time that elapses from when an event occurs to when it is serviced
- Two types of latencies affect performance
	- 1. **Interrupt latency** - time from arrival of interrupt to start of routine that services interrupt
	- 2. **Dispatch latency** - time for schedule to take current process off CPU and switch to another

![[Pasted image 20240417170736.png]]