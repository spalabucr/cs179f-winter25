Lab 2: Memory Allocation for XV6
================================

For this lab we have replaced the page allocator in the xv6 kernel with a buddy allocator. You will modify xv6 to use this allocator to allocate and free file structs so that xv6 can have more open file descriptors than the existing system-wide limit NFILE. Furthermore, you will make memory allocation to be done in a lazy manner.

To start the lab, switch to the alloc branch:

      $ git fetch
      $ git checkout alloc

Task No.1: Heap Allocator
-------------------------

xv6 has only a page allocator and cannot dynamically allocate objects smaller than a page. To work around this limitation, xv6 declares objects smaller than a page statically. For example, xv6 declares an array of file structs, an array of proc structures, and so on. As a result, the number of files the system can have open is limited by the size of the statically declared file array, which has NFILE entries (see kernel/file.c and kernel/param.h).

### The solution

The solution is to adopt the buddy allocator, which we have added to xv6 in kernel/buddy.c and kernel/list.c.

### Your job

Your job is to further improve xv6's memory allocation: modify kernel/file.c to use the buddy allocator so that the number of file structures is limited by available memory rather than NFILE.

### The alloctest program

To help you test your implementation, we've provided an xv6 program called alloctest (source in user/alloctest.c). It has two tests.

The first test allocates more than NFILE file structures by creating many processes, each opening many file descriptors. The first test will fail on unmodified xv6.

The second test creates a process that allocates as much memory as possible, and fails if less than a certain amount is available. It's effectively a test to see how much memory the kernel is using. The test will fail with the unoptimized buddy allocator that we have given you.

When you are done, your kernel should be able to run both alloctest and usertests. That is:

            $ alloctest
            filetest: start
            filetest: OK
            $

### Hints

*   You'll want to remove line 19 in kernel/file.c, which declares file\[NFILE\]. Instead, allocate struct file in filealloc using bd\_malloc. In fileclose you will free the allocated memory. Note that you can simplify fileclose, because ff isn't needed.
*   fileclose still needs to acquire ftable.lock because the lock protects f->ref.
*   bd\_malloc doesn't clear the memory it returns; instead, allocated memory starts out with whatever content it had from its last use. Callers should not assume that it starts out containing zeroes.
*   You can use bd\_print to print the state of the allocator.

Task No.2: Lazy Page Allocation
-------------------------------

One of the many neat tricks an O/S can play with page table hardware is lazy allocation of user-space heap memory. Xv6 applications ask the kernel for heap memory using the sbrk() system call. In the kernel we've given you, sbrk() allocates physical memory and maps it into the process's virtual address space. However, there are programs that use sbrk() to ask for large amounts of memory but never use most of it, for example to implement large sparse arrays. To optimize for this case, sophisticated kernels allocate user memory lazily. That is, sbrk() doesn't allocate physical memory, but just remembers which addresses are allocated. When the process first tries to use any given page of memory, the CPU generates a page fault, which the kernel handles by allocating physical memory, zeroing it, and mapping it. You'll add this lazy allocation feature to xv6 in this lab.

### Eliminate allocation from sbrk()

Your first task is to delete page allocation from the sbrk(n) system call implementation, which is the function sys\_sbrk() in sysproc.c. The sbrk(n) system call grows the process's memory size by n bytes, and then returns the start of the newly allocated region (i.e., the old size). Your new sbrk(n) should just increment the process's size (myproc()->sz) by n and return the old size. It should not allocate memory -- so you should delete the call to growproc() (but you still need to increase the process's size!).

Try to guess what the result of this modification will be: what will break?

Make this modification, boot xv6, and type echo hi to the shell. You should see something like this:

            init: starting sh
            $ echo hiusertrap(): unexpected scause 0x000000000000000f pid=3
               sepc=0x0000000000001258 stval=0x0000000000004008
            va=0x0000000000004000 pte=0x0000000000000000
            panic: uvmunmap: not mapped

The "usertrap(): ..." message is from the user trap handler in trap.c; it has caught an exception that it does not know how to handle. Make sure you understand why this page fault occurs. The "stval=0x0..04008" indicates that the virtual address that caused the page fault is 0x4008.

### Lazy allocation

Modify the code in trap.c to respond to a page fault from user space by mapping a newly-allocated page of physical memory at the faulting address, and then returning back to user space to let the process continue executing. You should add your code just before the printf call that produced the "usertrap(): ..." message. Your solution is acceptable if it passes usertests.

A good way to start this lab is by fixing usertrap() in trap.c so that you can run "echo hi" in the shell again. Once that works, you will find some additional problems that have to be solved to make usertests to work correctly. Here are some hints to get going.

*   You can check whether a fault is a page fault by seeing if r\_scause() is 13 or 15 in usertrap().
*   Look at the arguments to the printf() in usertrap() that reports the page fault, in order to see how to find the virtual address that caused the page fault.
*   Steal code from uvmalloc() in vm.c, which is what sbrk() calls (via growproc()). You'll need to call kalloc() and mappages().
*   Use PGROUNDDOWN(va) to round the faulting virtual address down to a page boundary.
*   uvmunmap() will panic; modify it to not panic if some pages aren't mapped.
*   If the kernel crashes, look up sepc in kernel/kernel.asm
*   Use your print function from above to print the content of a page table.
*   If you see the error "incomplete type proc", include "proc.h" (and "spinlock.h").

If all goes well, your lazy allocation code should result in echo hi working. You should get at least one page fault (and thus lazy allocation) in the shell, and perhaps two.

### Usertests

Now you have the basics working, fix your code so that all of usertests passes:

*   Handle negative sbrk() arguments.
*   Kill a process if it page-faults on a virtual memory address higher than any allocated with sbrk().
*   Handle fork() correctly.
*   Handle the case in which a process passes a valid address from sbrk() to a system call such as read or write, but the memory for that address has not yet been allocated.
*   Handle out-of-memory correctly: if kalloc() fails in the page fault handler, kill the current process.
*   Handle faults on the invalid page below the stack.

Your solution is acceptable if your kernel passes lazytests and usertests:

            $  lazytests
            lazytests starting
            running test lazy alloc
            test lazy alloc: OK
            running test lazy unmap...
            usertrap(): ...
            test lazy unmap: OK
            running test out of memory
            usertrap(): ...
            test out of memory: OK
            ALL TESTS PASSED
            $ usertests
            ...
            ALL TESTS PASSED
            $