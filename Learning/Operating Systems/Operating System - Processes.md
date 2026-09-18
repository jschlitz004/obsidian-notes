
CHAPTER OBJECTIVES

- To introduce the notion of a process—a program in execution, which forms the basis of all computation.
- To describe the various features of processes, including scheduling, creation, and termination.
- To explore interprocess communication using shared memory and message passing.
- To describe communication in client–server systems.

## 3.1 Process Concept
- A batch system executes jobs
- A time-shared system has user programs or tasks
- Even on a single-user system, a user may be able to run several programs at one time.
- All of these activities are similar so they are called **Processes**
- Job and process are generally interchangeable 

## 3.1.1 The Process
- A process is a program in execution
- A process is more than program code which is sometimes known as the text section
- A process also includes current activity, represented by the value of the program counter, and the contents of the processor's registers.
### A process
- A process is made up of the following
	- The **program code** - also known as the text section
	- **Current activity** - represented by the value of the program counter (PC) and the contents of the processor's registers
	- A processes **stack** - which contains temporary data (such as function parameters return addresses and local variables)
	- A **data section**, which contains global variables
	- A **heap** which is memory that is dynamically allocated during process run time. 
	![[Pasted image 20260915084157.png]]
Although two processes may be associated wiht the same program they are nevertheless considered two separate execution sequences. For example a user may invoke many c opies of the web browser program. Each of those is a separate process and while the text sections are equivalent, the data, heap and stack sections vary. 
## 3.1.3 Process Control Block

- Each Process in the operating system is represented by a process control block (PCB)- also called a task control block
- A process can be in the following states
	- new
	- ready 
	- running
	- waiting
	- halted
	- etc
- Program Counter - the counter indicates the address of the next instruction to be executed for this process
- CPU Registers - The registers vary in number and type, depending on the computer architecture. They include accumulators, index registers ,stack pointers and general-purpose registers, plus any condition code information
	- Along with  the program counter this state information must be saved when an interrupt occurs in order to allow the process to be continued correctly afterward
- CPU-scheduling information. This information includes a process priority, pointers
- Memory-management information - This information may include such items as the value of the base and limit registers and the page tables or the segment tables, depending on the memory system used by the OS.
- ![[Pasted image 20260911144226.png]]
- **Accounting Information** - This information includes the amount of CPU and real time used, time limits, account numbers, job or process numbers, and so on.
- **I/O Status information**. This information includes the list of I/0 devices allocated to the process, a list of open files, and so on.

#### TLDR, the process control block contains all information about the process,

### Threads

- A program that performs a single thread of execution can be exemplified by a process running a word-processor-program where a user cannot simultaneously type and run a spell checker
- Most modern operating systems allow a process to have multiple threads of execution.

### Process Representation in Linux
- A process control block (PCB) in Linux OS is represented by the C Structure task_struct
	- Can be found in linux/sched.h in the kernel
	- The structure contains all necessary information for representing a process
		- State of the process
		- scheduling and memory management info
		- list of open files
		- pointers to it's parent
		- list of its children and siblings
  
# 3.2 Process Scheduling 
- The objective of multiprogramming is to have some process running at all times, to maximize CPU utilization.
- The objective of time-sharing is to switch the CPU among processes so frequently that the user can interact with each program while it is running. 
- To meet these objectives, the **process scheduler** selects an available process for program execution on the CPU
- For a single-processor system, there will never be more than one running process.
	- Examples
		- Digital Alarm Clock
		- Microwave ovens
		- Car Key Fobs
## 3.2.1 Scheduling Queues

- As processes enter the system, they are put into a job queue, which consists of all processes in the system. The processes that are residing in main memory are ready and waiting to execute are kept on a list called the ready queue. (Generally stored as a linked list)
- The ready-queue header contains pointers to the first and final Process control blocks (PCBs) in the list. Each PCB includes a pointer field that points to the next PCB in the ready queue. 
- The system also includes other queues
	- device queue is a list of processes waiting for a particular I/0 device 
- A common representation of process scheduling is a queueing diagram.

 ![[Pasted image 20260915085032.png]]
- Each rectangle represents a queue
- Two types of queues are present - ready queue and a set of device queues
- Circles represent the resources that serve the queues
- Arrows indicate the flow of processes in the system
- Once a processes is selected for execution or dispatched, one of the following events could occur
	- The process could issue an I/0 request and then be placed in an I/O queue
	- The process could create a new child process and wait for the child's termination
	- The process could be removed forcibly from the CPU as a result of an interrupt, and be put back in the ready queue. 
- A process continues the above cycle until it terminates, at which time it is removed from all queues and has its PCB and resources deallocated. 
## 3.2.2. Schedulers

A process migrates among the various scheduling queues throughout its lifetime. The operating system must select, for scheduling purposes, processes from these queues in some fashion. The selection process is carried out by the appropriate **scheduler.** 
- long-term scheduler/job scheduler - selects processes from the pool of spooled processes on a disk drive and loads them into memory for execution
- short-term scheduler or CPU scheduler selects from among the processes that are ready to execute and allocates the CPU to one of them. 

#### Short Term Scheduler
The short-term scheduler must select a new process for the CPU frequently,
	- Often only a few milliseconds before waiting for an I/O request.
	- Often the short term scheduler executes at least every 100 milliseconds
	- The short-term scheduler must be fast. 
		- if it takes 10 ms to decide to execute a process for 100 ms, then  9% of the cpu is being used for scheduling, and is being WASTED
	
### Long Term Scheduler LT
- Long-term scheduler executes much less frequently; minutes may separate the creation of  a new new process and the next.
- The LT scheduler controls the degree of multiprogramming (number of processes in memory). If the degree of multiprogramming is stable, the average rate of process creation must be equal to the average departure rate of processes leaving the system. Thus, the long-term scheduler may need to be invoked only when a process leaves the system. 
- Due to the longer interval between executions, the LT scheduler can afford to take more time to decide which process should be selected for execution. 
- It is important that the LT scheduler make a careful selection.
- In general, most processes can be described as either I/O bound or CPU bound. 
	- And I/O-bound process is one that spends more of its time doing I/O than it spends doing computations
	- A CPU_bound process, in contract generates I/O requests infrequently using more of its time doing computations.
- It is important that the LT scheduler select a good mix of I/O-bound and CPU-bound processes. 
	- If all processes selected are I/O bound, the ready queue will almost always be empty and the ST scheduler will have little to do. 
	- If all processes are CPU bound, the I/O waiting queue will almost always be empty, devices will go unused and again the system will be unbalanced.
	- The system with the best performance will thus have a combination of CPU-bound and I/O-bound processes.
- Time-sharing systems such as UNIX and Windows often have no LT scheduler, and simply put every new process in memory for the ST scheduler
### Medium Term Scheduler MT
- Some operating systems, such as time-sharing systems utilize this MT scheduler
- Reason for existence
	- Sometimes it can be advantageous to remove a process from memory and from action contention for the CPU and thus reduce the degree of multiprogramming. 
	- Later the process can be reintroduced into memory, and its execution can be continued where it left off. 
- This scheme is called **swapping**
- Swapping may be necessary to improve the process mix or because a change in memory requirements has overcommitted acailable memory, requiring memory to be freed up. 

## Context Switch
- **Context Switch -** the switching of  the CPU to another process requires performing a state save of the current process and  state restore of a different process.

- When an interrupt occurs, the system needs to save the current context of the process running on the CPU so that it can restore that context when its processing is done.
	- Essentially suspending and resuming the process
- The context is represented in the PCB of the process. 
	- It includes the value of the CPU registers, the process state, and memory-management information. 
- Generically, we perform a state save of the current state of the CPU, be it in kernel or user mode, and then a state restore to resume operations.
- Time to switch depends on memory speed and # or registers needing to be saved
	- Very dependent on hardware support
- Average speed of a context switch is a few ms 
### Multitasking in Mobile Systems
- Due to mobile device constraints early versions of iOS did not provide user-application multitasking; only one application ran in the foreground while all other user applications were suspended
- OS tasks were multitasked because they were written by Apple and well behaved.
- IOS4 provided a limited from of multitasking for user applications
	- foreground application - is the application currently opena nd appearing on the display
	- Background application - remains in memory, but does not occupy the display\
- Multitasking limited due to battery life and memory use concerns. 

# 3.3 Operations on Processes
- The processes in most systems can execute concurrently and they may be created and deleted dynamically. Thus these systems must provide a mechanism for process creation and termination. 
## 3.3.1 Process Creation
- During the course of execution a process may create several new processes. 
- A process that creates a process is a parent process and the process created is a child process. Processes can create new processes, so the end result is a process tree.
- Most OS (Unix, Linux, Windows) identify processes according to a process identifier (or PID)
- pid - process identifier, typically an integer number
	- a unique value for each process in the system
	- can be used as index to access various attributes of a process within the kernel
- `ps -el` - command used to obtain a list of processes
![[Pasted image 20260916151111.png]]

### Child Processes 

- Child Processes need resources when they are created
	- A child process can have unfettered access to resources through the Operating system
	- A child process can be limited or restricted to a subset of it's parent process's resources
- Parent process may pass along initialization data (input) to the child process
	- Example: A process whose function is to display the contents of a file --say, image.jpeg---on the terminal screen. 
- When a process creates a new process, two possibilities for execution exist:
	- The parent continues to execute concurrently with it's children
	- The parent waits until some or all of its children have terminated
- There are also two address-space possibilities for the new process:
	- The child process is a duplicate of the parent process (it has the same program and data as the parent)
	- The child process has a new program loaded into it. 
#### Example of process creation differences
##### Unix Example 
- A new process is created by the `fork()` system call
	- The new process consists of a copy of the address space of the original process. This mechanism allows the parent process to communicate easily with its child process.
	- Both processes (parent and child) continue execution at the instruction after the `fork()`  with one difference, the return code for the `fork()` is zero for the new child process whereas the nonzero pid of the child is returned to the parent
- After a `fork()` call  one of the two processes typically uses the `exec()` system call to replace the process's memory space with a new program. 
- The `exec()` system call loads a binary file into memory (destroying the memory image of the program containing the `exec()` system call) and starts its execution
- The call to `exec()` does not return control unless an error occurs, because it overlays the process's address space with a new program
- 
![images](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781118063330/files/images/ch003-f010.jpg)

**Figure 3.10** Process creation using the fork() system call.

##### Windows Example
- A new process is created by the `CreateProcess()` Function in the Windows API
- Differences between `fork()` and `CreateProcess()`
	- `fork()` has a child process inheriting the address space of its parent
	- `CreateProcess()` requires loading a specified program into the address space of the child process at process creation.
	- `fork()` is passed no parameters where as `CreateProcess()` expects no less than 10 parameters

## 3.3.2 Process Termination
- A process terminates when it finishes executing its final statement
- A process asks the operating system to delete it using the `exit()` system call 
- When a process exits, it returns a status value (usually int) to it's parent process using the `wait()` system call.
- All resources of the process-- Including physical and veirtual memory, open files, and I/O buffers are deallocated by the OS
- A process can cause the termination of another process vai an appropriate system call.
- A system call like that can only be invoked by the parent of the process to be terminated.
- The parent needs to know the id of its children if it is to terminate them.
- Therefore when one process creates a new process, the id of the new process is passed to the parent.
- A parent may terminate the execution of one of its childen for a variety of reasons
	- The child has exceeded its usage of some of the resources that it has been allocated
	- The task assigned to the child is no longer require.
	- The parent is exiting and the OS does not allow for the child to continue if the parent terminates
- **Cascading termination** - If a process terminates then all of it's children must also be terminated. This is usually initiated by the OS
- `exit()` - can be called either directly or indirectly
- A parent process may wait for the termination of a child process by using the `wait()` system call. The wait() system call is passed a parameter that allows the parent to obtain the exit status of the child.
	- This system call returns the pid of the terminated child in addition to it's exit status
- A process that has terminated by whose parent has not yet called `wait()`, is known as a zombie process
- All processes become a zombie process, but only briefly as once the parent calls wait(), the pid of the zombie process and its entry in the process table are released.
- When a parent does not invoke wait(), and is instead terminated the child processes are known as orphans. In Unix / Linux the OS assigns the init process as the new parent to orphan processes.
- The `init()` process invokes `wait()` periodically, thereby allowing the exit status of any orphaned processes to be collected. Thus release the orphan's pid and the process-table entry

# 3.4 Inter process Communication
- Process executing concurrently in the operating system may be either independent processes or cooperating processes. 
- A process is **independent** if it cannot affect or be affected by the other processes executing in the system
- A process is **cooperating** if it can affect or be affected by the other processes executing in the system. 
- There are several reasons for providing an environment that allows process cooperation:
	- **Information Sharing**. Since several users may be interested in teh same piece of information(for instance, a shared file), we must provide an environment to allow concurrent access to such information. 
	- **Computation Speedup**. If we want a particular task to run faster, we must break it into substasks, each of which will be executing in parallel with the others. Notice that such a speedup can be achieved only if the computer has multiple processing cores.
	- **Modularity**. We may want to construct the system in a modular fashion, dividing the system functions into separate processes or threads, as we discussed in Chapter 2. 
	- **Convenience** Even an individual user may work on many tasks at the same time. For instance, a user may be editing, listening to music, and compiling in parallel. 
- Cooperating processes require an i**nterprocess communication (IPC) mechanism** that will allow them to exchange data and information.
- There are two fundamental models of interprocess communication: 
	- Shared Memory - a region of memory that is shared by cooperating processes is established.
		- Processes can then exchange information by readying and writing data to the shared region
	- Message Passing - communication takes place by means of messages exchanged between the cooperating processes. 
- Both aforementioned models are common in operating systems, and many systems implement both
- Message passing is useful for exchanging smaller amounts of data, because no conflicts need be avoided. 
- Message passing is also easier to implement in a stributed system than shared memory. 
- Shared memory can be faster than message passing, since message-passing systems are typically implemented using system calls and thus require the more time-consuming task of kernel intervention. 

![[Pasted image 20260917230520.png]]

**Figure 3.12** Communications models. (a) Message passing. (b) Shared memory.
#### Multiprocess Architecture- Chrome Browser
- Chrome uses a multiprocess architcture. The browse utilizes three processes
	- The **browser** process is responsible for managing the user interface as well as disk and network I/O. A new browser process is created when Chrome is started. Only one browser process is created.
	- **Renderer** processes contain logic for rendering web pages. Thus, they contain the logic for handling HTML, Javascript, images, and so forth. As a general rule, a new renderer process is created for each website opened in a new tab, and so several renderer processes may be active at the same time.
	- A **plug-in** process is created for each type of plug-in (such as Flash or QuickTime) in use. Plug-in processes contain the code for the plug-in as well as additional code that enables the plug-in to communicate with associated renderer processes and the browser process.

### Shared-Memory Systems
- Inter process communication using shared memory requires communicating procersses to establish a region of shared memory.
- A shared memory region resides in the address space of the process creating the share-memory segment.
- The OS tries to prevent one process from accessing another process's memroy. Shared memory requires that two or more process agree to remove this restriction.
	- They can then exchange information by reading and writing data in the shared areas.
	- The processes are also respondible for ensuring that they are not writing to the same location simultaneously. 

#### Producer Consumer Problem
- A **producer** process produces information that is consumed by a **consumer** process
- For example,  a compiler may produce assmebly code that is consumed by an assembler. The assembler, in turn, may produce object modules that are consumed by the loader. 
- One solution to the producer-consumer problem uses shared memory
- Two types of buffers can be used
	-  Unbounded buffer - places no practical limit on the size of the buffer
	- Bounded buffer assumes a fixed buffer size. The consumer must wait if the buffer is empty and the producer must wait if the buffer is full. 
