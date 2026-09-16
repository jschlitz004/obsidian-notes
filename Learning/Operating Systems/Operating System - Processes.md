
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

# # Threads

- A program that performs a single thread of execution can be exemplified by a processing running a word-processor-program where a user cannot simultaneously type and run a spell checker
- Most modern operating systems allow a process to have multiple threads of execution.

### Process Representation in Linux
- A process control block (PCB) in Linux OS is represented by the C Structure tast_struct
	- Can be found in linux/sched.h in the kernel
	- The structure contains all necessary information fo rrepresenting a processing
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
## 3.2.1 Scheduling Queues

- As processes enter the system, they are put into a job queue, which consists of all processes in the system. The processes that are residing in main memory are ready and waiting to execute are kept on a list called the ready queue. (Generally stored as a linked list)
- The ready-queue header contains pointers to the first and final Process controll blocks (PCBs) in the list. Each PCB includes a pointer field that points to the next PCB in the ready queue. 
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
- Long-term scheduler executes much less frequently; minutes may sepsarate the creation of onew new process and the next.
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
- **Context Switch -** when the switching the CPU to another process requires performing a state save of the current process and  state restore of ta different process.

- When an interrupt occurs, the system needs to save the current context of the process running on the CPU so that it can restore that context when its processing is done.
	- Essentially suspending and resuming the process
- The context is represented in the PCB of the process. 
	- It includes the value of the CPU registers, the process tate, and memory-management information. 
- Generically, we perform a state save of the current state of the CPU, be it in kernel or user mode, and then a state restore to resume operations.
- Time to switch depends on memory speed and # or registers needing to be saved
	- Very dependent on hardware support
- Average speed of a context switch is a few seconds
### Multitasking in Mobile Systems
- Due to mobile device constraints early versions of iOS did not provide user-application multitasking; only one application ran in the foreground while all other user applications were suspended
- OS tasks were multitasked because they were written by Apple and well behaved.
- IOS4 provided a limited from of multitasking for user applications
	- foreground application - is the application currently opena nd appearing on the display
	- Background application - remains in memory, but does not occupy the display\
- Multitasking limited due to battery life and memory use concerns. 

# 3.3 Operations on Processes
- The proccess in most systems can execute conceurrently and they may be created and deleted dynamically. Thus these systems must pro