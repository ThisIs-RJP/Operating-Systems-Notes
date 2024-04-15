# What is an Operating System
-  A program that <u>acts as an intermediary</u> between a user of a computer and the computer hardware
- OS goals
	- Execute user programs + make solving user problems easier
	- Make the computer system easier to use
	- Use the computer hardware in an efficient matter
- *"The one program running at all times on the computer"* is called the <u>kernel</u>, which is part of the operating system
- Everything else is either
	- A <u>system program</u> (ships with the operating system, but not part of the kernel), or
	- An <u>application program</u>, all programs not associated with the operating system
# Computer System Structure
- Computer system can be divided into <u>four components</u>
	- ***Hardware*** - provides basic computing resources
		- **Example:** CPU, memory, I/O devices
	- ***Operating System***
		- -> Controls + coordinates use of hardware among various applications and users
	- ***Application Programs*** - Defines the ways in which the system resources are used to solve the computing problems of the users
		-  **Example:** Word processors, compilers, web browser
	- ***Users*** 
		- **Example:** People, machines, other computers

# Abstract View of the Components of a Computer
![[Pasted image 20240415144312.png]]

# Some major/common operating systems
- Microsoft Windows
- POSIX Compliant (or mostly compliant) operating systems
	- **Linux**: MX Linux, EndeavorOS, Manjaro, Mint, Ubuntu...
	- **BSDs:** FreeBSD, NetBSD, Darwin...
- Some are proprietary (Windows), some are not (Linux, BSD), some are for embedded devices (QNX, VxWorks, WinCE)

### Note that some operating systems have different goals and and different supported platforms

# Real-Time Embedded Systems
- Real-time embedded systems most prevalent form of computers
	- Vary considerably
- Many other special computing environments as well
	- Some have OSes, some perform tasks without an OS
- Real-time OS has well-defined fixed time constraints
	- Processing ***must*** be done within constraint
	- Correct operation only if constraints are met
	- *"a late answer is a wrong answer"*

# Free and Open-Source Operating Systems
- Operating systems made available in source-code format rather than just binary <u>closed-source</u> and <u>proprietary</u>
- Counter to the <u>copy protection</u> and <u>Digital Rights Management (DRM)</u> movement
- Started by <u> Free Software Foundation (FSF)</u>, which has *"copyleft"* <u>GNU Public License (GPL)</u>
- **Examples:**
	- GNU/Linux
	- BSD UNIX (including core of Mac OS X)
	- and many more...

# Computer-System Architecture
- Historically, most systems used a single general-purpose processor
	- Aside from special-purpose processors e.g GPU/TPU...
- <u>Multiprocessors</u> systems have grown in use and importance
	- Advantages include
		- **Increased throughput**
		- **Economy of scale**
		- **Increased reliability** -> graceful degradation or fault tolerance
	- **Asymmetric Multiprocessing** -> each processor is assigned a specific task
	- **Symmetric Multiprocessing** -> each processor performs all tasks *(abbreviated as SMP)*

# Symmetric Multiprocessing Architecture
- Multi-chip and <u>multicore</u>
- Systems containing all chips
	- Chassis containing multiple separate systems
![[Pasted image 20240415193245.png]]

# Storage-Device Hierarchy
![[Pasted image 20240415193327.png]]

# Characteristics of Various Types of Storage
![[Pasted image 20240415193427.png]]

# A view of Operating System Services
![[Pasted image 20240415193458.png]]
# Operating System Services
- Operating systems provide an environment for execution of programs and services to programs and users
- Operating-system services provides functions that are helpful to the user:
	- **User Interface** - Almost all operating systems have a user interface (UI)
		- Varies between <u>COmmand-Line</u> (CLI),  <u>Graphics User Interface</u> (GUI), touch-screen, Batch...
	- **Program execution** - The system must be able to <u>load a program into memory and to run that program, end execution, either normally or abnormally (indicating error)</u>
	- **I/O Operations** - A running program may require I/O, which may involve a file or an I/O device
	- **File-system manipulation** - Programs need to read and write files and directories, create and delete them, search them, list file information, permission management
	- **Communications** - Processes may exchange information, on the same computer or between computers over a network
		- Communications may be via shared memory or through message passing (packets moved by the OS)
	- **Error Detection** - OS needs to be constantly aware of possible error
		- May occur in the CPU and memory hardware, in I/O devices, in user program...
		- For each type of error, the OS should take the appropriate action to ensure correct and consistent computing
		- Provide debugging facilities
- Another set of OS functions exists for ensuring the efficient operation of the system itself via resource sharing
	- **Resource allocation** - When multiple users or multiple jobs running concurrently, resources must be allocated to each of them
		- Many types of resources - CPU cycles, main memory, file storage, I/O devices
	- **Logging** - To keep track of which users use how much and what kinds of computer resources
	- **Protection and Security** - The owners of information stored in a multiuser or networked computer system may want to control use of that information, <u>processes</u> should not interfere with each other
		- **Protection** involves ensuring that all access to system resources is controlled
		- **Security** of the system from outsiders requires user authentication, extends to defending external I/O devices from invalid access attempts

# System Calls
- System calls enable software to use/interact/do things with the operating system kernel to make it do something
- In effect, it's a programming interface to the services provided by the OS
- Typically written in a high-level language (C or C++)
- Mostly accessed by programs via a high-level <u>Application Programming Interface</u> (API) rather than a direct system call use
- Common APIs are Win API for Windows, POSIX API for POSIX-based systems

![[Pasted image 20240415194741.png]]

# Example of System Calls
- Here is a system call sequence to copy the contents of one file to another file
![[Pasted image 20240415195150.png]]

# Example of Standard API
![[Pasted image 20240415195257.png]]

# System Call Implementation
- Typically, a number is associated with each system call
	- <u>System-call Interface</u> maintains a table indexed according to these numbers
- The system call interface invokes the intended system call in OS **kernel** and returns status of the system call and any return values
- The caller need know nothing about how the system call is implemented
	- Just needs to <u>obey API and understand what OS will do as a result call</u>
	- Most details of OS interface hidden from programmer by API
		- Managed by run-time support library (set of functions built into libraries included with compiler)

# API - System Call - OS Relationship
![[Pasted image 20240415195820.png]]

# Types of System Calls
- Process control
	- Create/terminate process
	- End/abort
	- Load/execute
	- Get process attributes, set process attributes
	- Wait for time
	- Wait event, signal event
	- Allocate and free memory
	- Dump memory if error
	- <u>Debugger</u> for determining <u>bugs, single step</u> execution
	- <u>Locks for managing access to shared data between processes</u>
- File management
	- Create/delete file
	- Open/close file
	- Read, write, reposition
	- Get/set file attributes
- Device Management
	- Request device, release device
	- Read, write, reposition
	- Get device attributes, set device attributes
	- Logically attach/detach devices
- Information Maintenance
	- Get/set time/date
	- Get/set system data
	- Get/set process, file or device attributes
 - Communications
	 - Create, delete communication connection
	 - Send, receive messages if <u>message passing model</u> to <u>host name or process name</u>
		 - From **client** to **server**
	- **Shared-memory model** create + gain access to memory regions
	- Transfer status information
	- Attach + detach remote devices
- Protection
	- Control access to resources
	- Get/set permissions
	- Allow + deny user access

# Examples of Windows and UNIX System Calls

![[Pasted image 20240415200432.png]]

# Standard C Library Example
![[Pasted image 20240415200514.png]]

# Quick Detour - What's a kernel?
- *"The one program running at all times on the computer"*
- if we think of an operating system ( as layers )
	- Application Program e.g opening a file in a text editor
	- Libraries
	- ...
	- Libraries
		- System calls
	- Kernel e.g Linux Kernel
	- Hardware

# System Services / Programs
- System Programs provide a convenient environment for program development and execution. They can be divided into
	- File manipulation
	- Status information sometimes stored in a file
	- Programming language support
	- Program loading + execution
	- Communications
	- Background services
	- Application Programs
- Most user' view of the OS is <u>defined by system programs, not the actual system calls</u>
- System Programs provide a convenient environment for program development and execution. 
	- Some of them are simply user interfaces to system calls; others are considerably more complex
- **File Management** - Create (touch, mkdir), delete (rm), copy (cp), rename, print, dump, list (ls, find), and generally manipulate files and directories
- **Status Information**
	- Some ask the system for info - date (date), time, amount of available memory (free, top), disk space (du), number of users (w)
	- Others provide detailed performance, logging and debugging information (dmesg)
	- Typically, these programs format and print the output to the terminal or other output devices
	- Some systems implement a <u>registry</u> - used to store and retrieve configuration information (Linux)
- **File Modification**
	- Text editors to create and modify files (nano, vim, emacs)
	- Special commands to search contents of files or perform transformations of the text ( find, sed, gawk)
- **Programming-Language Support** - Compilers, assemblers, debuggers and interpreters sometimes provided (gcc, Python, nasm)
- **Communications** - Provide the mechanism for creating virtual connections among processes, users, and computer systems
	- Allow users to send messages to one another's screens, browse web pages, send electronic-mail messages, log in remotely, transfer files from one machine to another (ssh, sshd, netcat)
- **Background Services**
	- Launch at boot time
		- Some for system startup, then terminate
		- Some from system boot to shutdown
	- Provide facilities like disk checking, process scheduling, error logging, printing
	- Run in user context not kernel context
	- Known as **services, subsystems daemons**
- **Application Programs**
	- Don't pertain to system
	- Run by users
	- Not typically considered part of OS
	- Launched by command line, mouse click, finger poke

# Linkers and Loaders
- Source code is compiled into object files designed to be loaded into any physical memory location - **relocatable object file**
- **Linker** combines these into single binary **executable** file
	- Also brings in libraries
- Program resides on secondary storage as binary executable
- Most be brought into memory by **loader** to be executed
	- **Relocation** assigns final addresses to program parts and adjusts code and data in program to match these addresses
- Modern general purpose systems don't link libraries into executables
	- Rather, **dynamically linked libraries** (in Windows, **DLLS**) are loaded as needed, shared by all that use the same version of that same library (loaded once)
- Object, executable files have standard formats, so the OS knows how to load and start them

# Role of the Linker and Loader
![[Pasted image 20240415202711.png]]


# Why Applications are Operating System Specific
- Apps compiled on one system usually not executable on other operating systems
- **Each OS provides its own unique system calls**
	- Own file formats etc.
- Apps can be multi-operating system
	- Written in interpreted language like Python, Ruby, and interpreter available on multiple OS's
	- App written in language that includes a VM containing the running app (like Java)
	- Use standard language (like C), compile separately on each OS to run on each

- **Application Binary Interface (ABI)** is architecture equivalent of API, defines how different components of binary code can interface for a given operating system on a given architecture, CPU, etc

# Design + Implementation

- Design + Implementation of OS is not *"solvable"*, but some **approaches have proven successful**
- Internal structure of different OS can vary widely
- Start the design by defining goals + specs
- Affected by choice of hardware, type of system
- **User** goals and **System** goals
	- User Goals - An OS should be convenient to use, easy to learn, reliable, safe and fast
	- System Goals - An OS should be easy to design, implement and maintain, as well as flexible, reliable, error-free and efficient
- Specifying + designing an OS is a highly creative task of **software engineering**

# Implementation
- Much variation
	- Early OS's in assembly language
	- Then system programming languages like Algol, PL/1
	- Now C, C++
- Actually usually a mix of languages
	- Lowest levels in assembly
	- Main body in C
	- Systems programs in C, C++, scripting languages like PERL, Python, shell scripts
- More high-level language easier to **port** to other hardware
	- But slower
- **Emulation** can allow an OS to run on non-native hardware

# Linux System Structure
![[Pasted image 20240415203611.png]]


# Building and Booting Linux
- Configure kernel via ```make menuconfig```
- Compile the kernel using ```make```
	- Produces ```vmlinuz```, the kernel image
	- Compile kernel modules via ```make modules```
	- Install kernel modules into ```vmlinuz``` via ```make modules_install```
	- Install new kernel on the system via ```make install```

# System Boot
- When power is initialised on system, execution starts at a fixed memory location
- The OS must be made available to hardware so hardware can start it
	- Small piece of code - **bootstrap loader, BIOS** stored in **ROM** or **EEPROM** locates the kernel, loads it into memory and starts it
	- Sometimes two-step process where **boot block** at fixed location loaded by ROM code, which loads bootstrap loader from disk
	- Modern systems replace BIOS with **Unified Extensible Firmware Interface** (UEFI)
- Common bootstrap loader, **GRUB**, allows selection of kernel from multiple disks, versions, kernel options
- Kernel loads and system is then running
- Boot loaders frequently allow various boot states, such as single user mode

# Operating-System Debugging
- **Debugging** is finding + fixing errors, or **bugs (e.g Linux gdb**)
- Also **performance tuning**
	- Can optimise system performance
		- Sometimes using **trace listings** of activities, recorded for analysis
		- **Profiling** is periodic sampling of instruction pointer to look for statistical trends
- OS generate **log files** containing error information (check /var/log)
- Failure of an application can generate **core dump** file capturing memory of the process
- OS failure can generate **crash dump** file containing kernel memory

# Tracing
- Collects data for a specific event, such as steps involved in a system call invocation
- Tools include
	- **strace** - trace system calls invoked by a process
	- **gdb** - source-level debugger
	- **perf** - collection of Linux performance tools
	- **tcpdump** - collects network packets

# Linux bcc/BPF Tracing Tools
![[Pasted image 20240415204606.png]]