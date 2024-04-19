
# File Concept

- Logical storage unit
- Types
	- Data
		- -> Numeric
		- -> Character
		- -> Binary
	- Program
- Contents defined by file's creator
	- Many types
		- **text file**
		- **source file**
		- **executable file**

# File Attributes

- **Name:** Only information kept in human-readable form
- **Identifier:** Unique tag (number) identifies file within file system
- **Type:** Needed for systems that support different types
- **Location:** Pointer to file location on device
- **Size:** Current file size
- **Protection:** Controls who can do reading, writing, executing
- **Time, date, and user ID:** Data for protection, security and usage monitoring
- Info about files are kept in the directory structure, which is maintained on the disk
- Many variations, including **extended file attributes** such as file checksum...

# File Operations

- **Create, Open, Close**
- **Write -** at **write pointer** location
- **Read** - at **read pointer** location
- **Reposition within file** - **seek**
- **Delete**
- **Truncate**

# Open Files

- Several pieces of data are needed to manage open files:
	- **Open-file table:** track open files
	- File pointer: pointer to last read/write location, per process that has the file open
	- **File-open count:** counter of number of times a file is open - to allow removal of data from open-file table when last process closes it
	- Disk location of the file: cache of data access information
	- Access rights: per-process access mode information

### Note
**On Linux, we can make a file executable using**
	```chmod +x file.name```

# Sequential Access for Files

- Operations
	- ```read next```
	- ```write next```
	- ```Reset```
- Figure
	![[Pasted image 20240419114458.png]]

# Direct Access (Relative Access)

- Operations
	- ```read n```
	- ```write n```
	- ```position to n```
		- ```read next```
		- ```write next```
		- ```rewrite n```
		Where *n* = relative block number

# Simulation of Sequential Access on Direct-Access  file
![[Pasted image 20240419114708.png]]

# Operations Performed on Directories

- Search for a file
- Create a file
- Delete a file
- List a directory
- Rename a file
- (conveniently) Traverse a file system

# Directory Organisation

The directory is organised logically to enable:
- Efficiency - locating a file quickly
- Naming - convenient for users e.g
	- Two users can have same name for different files
	- The same file can have several different names (**linking**)
- Grouping - logical grouping of files by properties, (e.g all Java programs, all games, ...)

# Tree Structured Directories

![[Pasted image 20240419115129.png]]

# Acylic-Graph Directories

- Have **shared** sub-directories and files
- With acylic graph directories, we avoid cycles that may cause issues
- Example
	![[Pasted image 20240419115245.png]]

# Soft vs Hard link

- But first, what is a hard link?
	Think of a hard link like a signpost pointing to a house. Let's say you have a house (a file) on a street (a file system). Normally, you can access the house using the street address (the file's path). Now, imagine you put another signpost (a hard link) on a different street that points to the same house. Even though there are two signposts, there's still only one house. If you remove one signpost, the house remains accessible through the other one. Similarly, if you delete one hard link, the file remains accessible through the other hard link(s). It's like having multiple entrances to the same building.
- As for soft links...
	A soft link, also known as a symbolic link or symlink, can be likened to a shortcut or a signpost with directions. Instead of pointing directly to the house (like a hard link does), a symbolic link points to the signpost that points to the house. It's like saying, "Hey, if you want to find the house, go to this signpost first, and then follow the directions." 
	So, if you delete the signpost that's being pointed to by the symbolic link, you can't find the house anymore, because the directions are gone. Symbolic links are more flexible than hard links because they can span different file systems and even link to directories, but they're also more fragile because if the original file or directory is moved or deleted, the symbolic link breaks.


![[Pasted image 20240419115336.png]]

![[Pasted image 20240419115403.png]]

- If I delete the original file...
	![[Pasted image 20240419115506.png]]
- Since the (hard) link points to the inode, the link (file) remains

- But what happens if I created a new file with the same name?
	Answer: They'll have different inodes
	- Explanation
		When you create a new file in a file system, even if it's replacing the last one in a location, a new inode is typically allocated to it. This happens because each file has its own unique inode, which contains metadata about the file (like permissions, ownership, timestamps, etc.), as well as pointers to the actual data blocks where the file's contents are stored on the disk.
		
		Even if the new file has the same name and is created in the same location as the previous one, it's considered a distinct entity with its own set of metadata and data blocks. This ensures that each file can have its own properties and content, even if they share the same name or location at different points in time.
		
		Therefore, even if you delete the last file in a location and then create a new one with the same name, it gets its own unique inode, separate from the previous one. This is why a new inode is created for the new file.


# Soft links behave differently, kind of...

- They don't point to the same inode on the disk, instead they point to the file
	![[Pasted image 20240419120430.png]]

- So if you delete the original file, it can't access it
	![[Pasted image 20240419120504.png]]

# But why wouldn't we want to create a hard link for a directory?
Creating hard links for directories can lead to various complications and potential issues:

1. **Circular References**: Hard links allow for the creation of loops or circular references within directory structures. This can confuse file system operations, such as directory traversal, and lead to infinite loops or other unexpected behaviours.
    
2. **File system Corruption**: Manipulating hard links within directories can inadvertently corrupt the file system structure. Since a directory contains entries for its contents, including other directories, creating hard links could disrupt this structure and cause file system corruption.
    
3. **Security Risks**: Hard links for directories could introduce security risks, as users might be able to access directories they're not supposed to by creating hard links to them from other locations.
    
4. **Maintenance Complexity**: Managing hard links for directories would add complexity to file system maintenance and administration. It would require additional checks and safeguards to prevent unintended consequences and ensure the integrity of the file system.
    

For these reasons, most operating systems do not support creating hard links for directories. Instead, symbolic links (soft links) are typically used for creating directory shortcuts or aliases, as they pose fewer risks and provide more flexibility without altering the file system structure.

# General Graph Directory

![[Pasted image 20240419120717.png]]

# Protection

- File owner/creator should be able to control:
	- What can be done
	- By whom
- Types of access
	- Read
	- Write
	- Execute
	- Append
	- Delete
	- List

# Access Lists and Groups in UNIX

- Mode of access: read, write, execute
- Three classes of users on UNIX/Linux
	 ![[Pasted image 20240419121004.png]]
- Ask managers to create a group(unique name), say G, and add some users to the group
- For a file (*say game*) or subdirectory, define an appropriate access
	![[Pasted image 20240419121150.png]]
- Attach a group to file
	![[Pasted image 20240419121212.png]]


# Sample UNIX Directory Listing
	![[Pasted image 20240419121242.png]]
	![[Pasted image 20240419121558.png]]

# Permissions

![[Pasted image 20240419121620.png]]

# Where is this user info stored?

- In ```/etc/group``` and ```/etc/password```
	![[Pasted image 20240419121725.png]]


# Disk Structure

- Disk can be subdivided into **partitions**
- Disk/Partitions can be **RAID** protected against failure
	- Now what does RAID mean?
		RAID stands for Redundant Array of Independent Disks. It's a technology used to combine multiple physical disks into a single logical unit to improve performance, reliability, or both. When <u>disks or partitions are RAID protected against failure</u>, it means that they are <u>configured in such a way that if one disk fails, the system can continue operating without losing data or experiencing downtime</u>.
- Disk or partition can be used **raw** - without a file system, or **formatted** with a file system (we are interested in the latter)
- Entity containing file system is known as a **volume**
	- Could be in a single partition, or spread across multiple disks (like with RAID)
- Each volume containing a file system also tracks that file system's info in **volume table of contents**
- In addition to **general-purpose file systems** there are many **special-purpose file systems**, frequently all within the same OS or computer

# File System

- General-purpose computers can have multiple storage devices
- Devices can be *sliced* into partitions, which hold volumes
- Volumes can also span multiple partitions/drives
- Each volume usually formatted into a file system
- ```#``` of file systems varies, typically dozens available to choose from
- Typical storage device organisation
	![[Pasted image 20240419123331.png]]

# Sample file system in a Linux box

![[Pasted image 20240419123351.png]]


# Partitions and Mounting

- Partition can be volume containing a file system ("cooked") or **raw** - just a sequence of blocks with no file system
- **Root partition** contains the OS, other partitions can hold other OSes, other file systems, or be raw
	- Mounted at boot time
	- Other partitions can mount automatically or manually on **mount points** - location at which they can be accessed
- At mount time, file system consistency checked (fsck)
	- Is all metadata correct?
		- If not, fix it and try again
		- If yes, add to mount table, allow access

# Types of File systems

**Note: FS = file system**

- Many general-purpose file systems
- But systems frequently have many file systems, some general and some special-purpose
- Consider Linux has
	- ```tmpfs```- memory-based volatile FS for fast, temporary
	- ```udev``` - exposes underlying hardware as a file (through kernel)
	- ```procfs``` - kernel interface to process structures
	- ```Ext4, reizerfs``` - general purpose file systems
![[Pasted image 20240419125108.png]]

# The Linux Proc File System

- The **proc file system** does not store data, rather, its contents are computed on demand according to user file I/O requests
- **proc** implements a directory structure, with file contents within
	- When data is read from one of these files, **proc** collects the appropriate information, formats it into text form and places it into the requesting process's read buffer
- We generally don't use /proc directly (we can, but we don't). We interface with it using the sysctl command

# Journaling

Journaling is a technique used in file systems to improve their reliability and resilience against crashes or unexpected shutdowns. It works by keeping a log, or "journal," of the changes that are about to be made to the file system. Before modifying the actual file system structures, such as updating directories or allocating space for new files, the changes are first recorded in the journal.

- Ext3/ext4 implements **journaling**, with file system updates first written to a log file in the form of **transactions**
	- Once in log file, considered committed
	- Over time, log file transactions *replayed* over file system to put changes in place
- On system crash, some transactions might be in journal but not yet placed into file system
	- Must be completed once system recovers
		- No other consistency checking is needed after a crash ( much faster than older methods)
- Improves write performance on hard disks by turning random I/O into sequential I/O

# I/O Input/Output

# Typical Bus Structure
![[Pasted image 20240419125827.png]]

# I/O Hardware

- Incredible variety of I/O devices
	- Storage
	- Transmission
	- Human-Interface

- Common concept is - signals from I/O devices interface with computer
	- **Port** - connection point for device
	- **Bus** - **daisy chain** or shared direct access
		- **PCI** bus common in PCs and servers, PCI Express (PCIe)
			"PCI" stands for Peripheral Component Interconnect. It's a <u>standard for connecting devices to a computer's motherboard, allowing them to communicate with the CPU and other components</u>. The PCI standard defines the physical and electrical specifications for the interface, as well as the protocols for data transfer between devices and the CPU.
		- **expansion bus** connects relatively slow devices
- **Controller (host adapter)** - Electronics that operate port, bus, device
	- Contains processor, microcode, private memory, bus controller, etc
		Some talk to per-device controller with bus controller, microcode, memory etc
- I/O instructions control devices
- Devices usually have registers where device driver places commands, addresses, and data to write, or read data from registers after command execution
		- Data-in register, data-out register, status register, control register
		- Typically 1-4 bytes, or FIFO buffer
- Devices have addresses, used by
	- Direct I/O instructions
	- **Memory-mapped I/O**
		- Device data and command registers mapped to processor address space
			- Especially for large address space (graphics)

# Device I/O Port Locations on PCs (partial)

![[Pasted image 20240419130653.png]]

# Polling

Polling in the context of operating systems refers to a technique used by software to check the status of a hardware device or resource at regular intervals. Instead of relying on hardware interrupts to signal when a device is ready to send or receive data, the operating system periodically "polls" the device to inquire about its status.

- For each byte of I/O
	1. Read busy bit from status register until 0
	2. Host sets write bit and write copies data into data-out register
	3. Host sets command-ready bit
	4. Controller sets busy bit, executes transfer
	5. Controller clears busy bit, error bit, command-ready bit when transfer done
- Step 1 is **busy-wait** cycle to wait for I/O from device
	- Reasonable if device is fast
	- But inefficient if device is slow
	- CPU switches to other tasks?
		- But if miss a cycle, data overwritten/lost

# Interrupts

In operating systems, interrupts are <u>signals generated by hardware devices to notify the CPU that they require attention or have completed a task</u>. Interrupts allow devices to asynchronously interrupt the CPU's normal execution flow, enabling efficient multitasking and handling of various input/output (I/O) operations.

- CPU **Interrupt-request line** triggered by I/O device
	- Checked by processor after each instruction
- **Interrupt handler** receives interrupts
	- **Maskable** to ignore or delay some interrupts
- **Interrupt vector to dispatch interrupt to correct handler**
# Interrupt-Driven I/O Cycle

![[Pasted image 20240419131311.png]]

# Direct Memory Access

**Direct memory access** is a feature of computer systems that allows certain hardware subsystems to access main system <u>memory</u> independently of the CPU

- Used to avoid **programmed I/O** (one byte at a time) for large data movement
- Requires **DMA** controller
- Bypasses CPU to transfer data directly between I/O device and memory
- OS writes DMA command block into memory
	- Source and destination addresses
	- Read or write mode
	- Count of bytes
	- Writes location of command block to DMA controller
	- When done, <u>interrupts</u> to signal completion
![[Pasted image 20240419131641.png]]

# Six Step Process to Perform DMA transfer

![[Pasted image 20240419131710.png]]