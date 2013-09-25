"#SC_GetReg
This is a straightforward system call, we just read the value of the register
using the function machine::ReadRegister and then returned the value in register
2.

#SC_GetPID and SC_GetPPID
1. A new static variable called pidCount and a macro MAX_THREADS was defined in
the Thread class and header file thread.h respectively.

2. The constructor of Thread Class was modified to increment pidCount and assign
this incremented pidCount to the current Thread as it's PID. (pidCount has been
initialized in such a manner that it wraps around after MAX_THREADS)

3. In the body of the syscall, the parent assigned it's PID as the PPID of the
child thread.

4. The edge case of zero PID was handled by giving the first thread PID 1 and
assinging it's parent the PID zero.

#SC_Time
1. This syscall simply returned the value of stats->totalTicks.

#SC_Yield
1. This syscall calls the Yield() function of Thread class on the currentThread.
The only caveat was to increment PC before calling Yield.

#SC_Sleep
1. Firstly, the value of the PC is incremented.

2. If the supplied time was zero, Thread::Yield() is called on the
currentThread, after disabling interrupts.

3. If the value if non-zero, then the currentThread is added in an increasing
manner to the timerQueue.

4. The timerQueue is a global variable defined in system header file and it maintains
a queue of threads waiting on the timer Interrupt. It is a sorted queue.

5. Whenever a timer Interrupt happens, the function TimerInterruptHandler is
called, in this function we check whether timerQueue is empty or not, in which
case, we return from the function.

6. Otherwise, all those threads with an expired sleep time are put on the
readyQueu using Scheduler::ReadyToRun.

#SC_Fork
1. Tobegin with, the value of PC was incremented follwing which a new Thread
Object was created.

2. The parent pointer of this new Thread was changed to point to the
currentThread, which is the creator of this forked thread.

3. Also, every parent maintains a hash map with the child pid as the key and
child_status as the value. This status is not the same as the scheduler status,
it is used to store the values CHILD_LIVE, PARENT_WAITING or the exit status of
the child, which are used in the implementation of Join.

4. initalizeChildStatus essentially sets the value of the status of the current
child as CHILD_LIVE in the hash map.

5. The new step was to create an Address Space, the constructor of AddrSpace
class was overloaded to handle the case when we fork threads. 
    
    a. In this new constructor, the Page Table mapped virtual address to physical address with a
        function virtual address + totalPagesCount. Where totalPagesCount is a global
        which stores the total number of physical pages which have been alloted as of
        yet.
    b. Next we duplicated the parent's physical memory exactly into the child's
        physical memory location.

6. Now the return value of child was changed to zero and it's state is saved
using Thread::SaveUserState.

7. The parent's return address is set to the child's pid and the child's stack
is set to the function forkStart which is defined in thread.h. This essentially
mimics the behaviour a normal thread has on returning from a sleep. This is done
to ensure that a newly forked thread behaves like any other woken thread.

8. After this the child is scheduled to run.

#SC_Join
1.
"


