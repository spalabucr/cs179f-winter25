Lab 1: XV6 and UNIX Utilities
===========================

This lab will familiarize you with xv6 and its system calls.

Boot xv6
------------------

Fetch the xv6 source for the lab and check out the util branch:

      $ git clone git@github.com:emidec/cs179f-fall23
      Cloning into 'xv6-riscv'...
      ...
      $ cd xv6-riscv
      $ git checkout util
      Branch 'util' set up to track remote branch 'util' from 'origin'.
      Switched to a new branch 'util'
      

This repository differs slightly from the book's xv6-riscv; it mostly adds some files. If you are curious look at the git log:

      $ git log
      

The files you will need for this and subsequent lab assignments are distributed using the [Git](http://www.git-scm.com/) version control system. Above you switched to a branch (git checkout util) containing a version of xv6 tailored to this lab. To learn more about Git, take a look at the [Git user's manual](http://www.kernel.org/pub/software/scm/git/docs/user-manual.html), or, you may find this [overview of Git](http://eagain.net/articles/git-for-computer-scientists/) useful. Git allows you to keep track of the changes you make to the code. For example, if you are finished with one of the exercises, and want to checkpoint your progress, you can _commit_ your changes by running:

      $ git commit -am 'my solution for util lab exercise 1'
      Created commit 60d2135: my solution for util lab exercise 1
       1 files changed, 1 insertions(+), 0 deletions(-)
      $
      

You can keep track of your changes by using the ```git diff``` command. Running ```git diff``` will display the changes to your code since your last commit, and git diff origin/util will display the changes relative to the initial code. Here, origin/util is the name of the git branch with the initial code you downloaded for the class.

Build and run xv6:

      $ make qemu
      riscv64-unknown-elf-gcc    -c -o kernel/entry.o kernel/entry.S
      riscv64-unknown-elf-gcc -Wall -Werror -O -fno-omit-frame-pointer -ggdb -DSOL\_UTIL -MD -mcmodel=medany -ffreestanding -fno-common -nostdlib -mno-relax -I. -fno-stack-protector -fno-pie -no-pie   -c -o kernel/start.o kernel/start.c
      ...
      riscv64-unknown-elf-ld -z max-page-size=4096 -N -e main -Ttext 0 -o user/\_zombie user/zombie.o user/ulib.o user/usys.o user/printf.o user/umalloc.o
      riscv64-unknown-elf-objdump -S user/\_zombie > user/zombie.asm
      riscv64-unknown-elf-objdump -t user/\_zombie | sed '1,/SYMBOL TABLE/d; s/ .\* / /; /^$/d' > user/zombie.sym
      mkfs/mkfs fs.img README  user/xargstest.sh user/\_cat user/\_echo user/\_forktest user/\_grep user/\_init user/\_kill user/\_ln user/\_ls user/\_mkdir user/\_rm user/\_sh user/\_stressfs user/\_usertests user/\_grind user/\_wc user/\_zombie
      nmeta 46 (boot, super, log blocks 30 inode blocks 13, bitmap blocks 1) blocks 954 total 1000
      balloc: first 591 blocks have been allocated
      balloc: write bitmap block at sector 45
      qemu-system-riscv64 -machine virt -bios none -kernel kernel/kernel -m 128M -smp 3 -nographic -drive file=fs.img,if=none,format=raw,id=x0 -device virtio-blk-device,drive=x0,bus=virtio-mmio-bus.0

      xv6 kernel is booting

      hart 2 starting
      hart 1 starting
      init: starting sh
      $
      

If you type ls at the prompt, you should see output similar to the following:

      $ ls
      .              1 1 1024
      ..             1 1 1024
      README         2 2 2059
      xargstest.sh   2 3 93
      cat            2 4 24256
      echo           2 5 23080
      forktest       2 6 13272
      grep           2 7 27560
      init           2 8 23816
      kill           2 9 23024
      ln             2 10 22880
      ls             2 11 26448
      mkdir          2 12 23176
      rm             2 13 23160
      sh             2 14 41976
      stressfs       2 15 24016
      usertests      2 16 148456
      grind          2 17 38144
      wc             2 18 25344
      zombie         2 19 22408
      console        3 20 0
      

These are the files that mkfs includes in the initial file system; most are programs you can run. You just ran one of them: ls.

xv6 has no ps command, but, if you type Ctrl-p, the kernel will print information about each process. If you try it now, you'll see two lines: one for init, and one for sh.

To quit qemu type: Ctrl-a x.

Grading
-------

You can run ```make grade``` to test your solutions with the grading program. The TAs will use the same grading program to assign your lab submission a grade.

sleep
---------------

Implement the UNIX program sleep for xv6; your sleep should pause for a user-specified number of ticks. A tick is a notion of time defined by the xv6 kernel, namely the time between two interrupts from the timer chip. Your solution should be in the file user/sleep.c.

Some hints:

*   Before you start coding, read Chapter 1 of the [xv6 book](https://pdos.csail.mit.edu/6.828/2019/xv6/book-riscv-rev0.pdf).
*   Look at some of the other programs in user/ (e.g., user/echo.c, user/grep.c, and user/rm.c) to see how you can obtain the command-line arguments passed to a program.
*   If the user forgets to pass an argument, sleep should print an error message.
*   The command-line argument is passed as a string; you can convert it to an integer using atoi (see user/ulib.c).
*   Use the system call sleep.
*   See kernel/sysproc.c for the xv6 kernel code that implements the sleep system call (look for sys\_sleep), user/user.h for the C definition of sleep callable from a user program, and user/usys.S for the assembler code that jumps from user code into the kernel for sleep.
*   Make sure main calls exit() in order to exit your program.
*   Add your sleep program to UPROGS in Makefile; once you've done that, make qemu will compile your program and you'll be able to run it from the xv6 shell.
*   Look at Kernighan and Ritchie's book _The C programming language (second edition)_ (K&R) to learn about C.

Run the program from the xv6 shell:

            $ make qemu
            ...
            init: starting sh
            $ sleep 10
            (nothing happens for a little while)
            $
          

Your solution is correct if your program pauses when run as shown above. Run make grade to see if you indeed pass the sleep tests.

Note that make grade runs all tests, including the ones for the assignments below. If you want to run the grade tests for one assignment, type:

           $ ./grade-lab-util sleep
         

This will run the grade tests that match "sleep". Or, you can type:

           $ make GRADEFLAGS=sleep grade
         

which does the same.

find
------------------

Write a simple version of the UNIX find program: find all the files in a directory tree with a specific name. Your solution should be in the file user/find.c.

Some hints:

*   Look at user/ls.c to see how to read directories.
*   Use recursion to allow find to descend into sub-directories.
*   Don't recurse into "." and "..".
*   Changes to the file system persist across runs of qemu; to get a clean file system run make clean and then make qemu.
*   You'll need to use C strings. Have a look at K&R (the C book), for example Section 5.5.
*   Note that == does not compare strings like in Python. Use strcmp() instead.
*   Add the program to UPROGS in Makefile.

Your solution is correct if produces the following output (when the file system contains the files b and a/b):

          $ make qemu
          ...
          init: starting sh
          $ echo > b
          $ mkdir a
          $ echo > a/b
          $ find . b
          ./b
          ./a/b
          $
        

xargs
-------------------

Write a simple version of the UNIX xargs program: read lines from the standard input and run a command for each line, supplying the line as arguments to the command. Your solution should be in the file user/xargs.c.

The following example illustrates xarg's behavior:

          $ echo hello too | xargs echo bye
          bye hello too
          $
        

Note that the command here is "echo bye" and the additional arguments are "hello too", making the command "echo bye hello too", which outputs "bye hello too".

Please note that xargs on UNIX makes an optimization where it will feed more than argument to the command at a time. We don't expect you to make this optimization. To make xargs on UNIX behave the way we want it to for this lab, please run it with the -n option set to 1. For instance

          $ echo "1\\n2" | xargs -n 1 echo line
          line 1
          line 2
          $
        

Some hints:

*   Use fork and exec to invoke the command on each line of input. Use wait in the parent to wait for the child to complete the command.
*   To read individual lines of input, read a character at a time until a newline ('\\n') appears.
*   kernel/param.h declares MAXARG, which may be useful if you need to declare an argv array.
*   Add the program to UPROGS in Makefile.
*   Changes to the file system persist across runs of qemu; to get a clean file system run make clean and then make qemu.

xargs, find, and grep combine well:

        $ find . b | xargs grep hello
        

will run "grep hello" on each file named b in the directories below ".".

To test your solution for xargs, run the shell script xargstest.sh. Your solution is correct if it produces the following output:

        $ make qemu
        ...
        init: starting sh
        $ sh < xargstest.sh
        $ $ $ $ $ $ hello
        hello
        hello
        $ $
        

You may have to go back and fix bugs in your find program. The output has many $ because the xv6 shell doesn't realize it is processing commands from a file instead of from the console, and prints a $ for each command in the file.

Submit the lab
--------------

**This completes the lab.** Make sure you pass all of the make grade tests.

You need to submit a code diff and a lab report. You can generate a code diff by running git diff origin/util > mycode.diff. You need to write a lab report (a PDF document) to describe what changes you have made and explain why you made these changes. Include screenshots when necessary.

Submit these two files in iLearn.

Optional challenge exercises
----------------------------

*   Write an uptime program that prints the uptime in terms of ticks using the uptime system call.
    
*   Support regular expressions in name matching for find. grep.c has some primitive support for regular expressions.
    
*   The xv6 shell (user/sh.c) is just another user program and you can improve it. It is a minimal shell and lacks many features found in real shell. For example, modify the shell to not print a $ when processing shell commands from a file, modify the shell to support wait, modify the shell to support lists of commands, separated by ";", modify the shell to support sub-shells by implementing "(" and ")", modify the shell to support tab completion, modify the shell to keep a history of passed shell commands, or anything else you would like your shell to do. (If you are very ambitious, you may have to modify the kernel to support the kernel features you need; xv6 doesn't support much.)