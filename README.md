# CS179F Fall 2023

## Catalog Description
CS179F delves into the design and implementation of operating systems by conducting a quarter-long project.

## Team
- **Instructor: Emiliano De Cristofaro**
    - Email: emilianodc AT cs DOT ucr DOT edu
    - Lectures: Tuesdays 3:30 - 4:20 PM
Gordon Watkins Hall, Room 1101
    - Office hours: TBD
- **Teaching Assistant: Gao Lian**
    - Email: lian DOT gao AT email DOT ucr DOT edu
    - Office hourse: TBD

## Textbooks
- [Operating Systems: Three Easy Pieces](http://pages.cs.wisc.edu/~remzi/OSTEP/), Remzi H. Arpaci-Dusseau and Andrea C. Arpaci-Dusseau
- [XV6: A Simple UNIX-like Teaching Operating System](https://pdos.csail.mit.edu/6.S081/2020/xv6/book-riscv-rev1.pdf), Russ Cox, Frans Kaashoek, and Robert Morris
- Lions' Commentary on UNIX' 6th Edition, John Lions, Peer to Peer Communications. ISBN: 1-57398-013-7. 1st edition (June 14, 2000)

## Communication
- [Piazza](https://piazza.com/ucr/fall2023/cs179f/home) for announcement, discussions, and help
- [Canvas](https://elearn.ucr.edu/courses/110956) for assignments and grading

## Acknowledgments
- MIT 6.1810 Operating System Engineering [class](https://pdos.csail.mit.edu/6.828/2023/index.html) series for setting up the projects and maintaining xv6-riscv
- Prof. [Heng Li](https://www.cs.ucr.edu/~heng/) for sharing material related to his iteration of the class in W'21 and W'20

## XV6
- We will use the [XV6](https://pdos.csail.mit.edu/6.828/2012/xv6.html) operating system as a base for our projects
- Please familiarize yourself with XV6 to get a grasp on how it is organized and implemented, e.g., using:
    -  [Online version](http://www.lemis.com/grog/Documentation/Lions/) of the Lions commentary
	-  [Source Code](http://v6.cuzuco.com/), also [available](http://minnie.tuhs.org/cgi-bin/utree.pl?file=V6) through the Unix Heritage Society
    -  If you are curious, you can also see the original code in the PDP11/40 Processor Handbook, Digital Equipment Corporation, 1972 [[PDF](http://pdos.csail.mit.edu/6.828/2005/readings/pdp11-40.pdf)] [[Web](http://pdos.csail.mit.edu/6.828/2005/pdp11/)]

## Environment
- You need three main things:
    - _xv6-riscv_, a re-implementation of Unix Version 6 for a modern RISC-V multiprocessor using ANSI C
    - _qemu_, an open source machine emulator and virtualize
    - _class git repository_, [https://github.com/emidec/cs179f-fall23/](https://github.com/emidec/cs179f-fall23/)
- Please see [https://pdos.csail.mit.edu/6.828/2023/tools.html](https://pdos.csail.mit.edu/6.828/2023/tools.html) for instructions re. how to set up xv6 on your local machine, or see below
- _Note_: I've encountered problems with running qemu on Macs running _Apple M1/M2 silicon_. I'm still working on possible solutions, but my advice for the moment is to use a virtual machine running Ubuntu
- _Note_: I also encountered problems with latest versions of qemu. My advice is to get (and compile from source) qemu 4.1.1 from [https://download.qemu.org](https://download.qemu.org) rather than the latest version installed via packages:
```
wget https://download.qemu.org/qemu-4.1.1.tar.xz
tar xf qemu-4.1.1.tar.xz
cd qemu-4.1.1
./configure --disable-kvm --disable-werror --prefix=/usr/local --target-list="riscv64-softmmu"
make
sudo make install
```

## Install on Linux
- Debian/Ubuntu:
    - ```sudo apt-get install git build-essential gdb-multiarch qemu-system-misc gcc-riscv64-linux-gnu binutils-riscv64-linux-gnu```
- Arch Linux:
    - ```sudo pacman -S riscv64-linux-gnu-binutils riscv64-linux-gnu-gcc riscv64-linux-gnu-gdb qemu-emulators-full```
- To test your installation, run:
    - ```$ qemu-system-riscv64 --version```  
    ```QEMU emulator version 5.1.0```
    - ```$ riscv64-linux-gnu-gcc --version```  
	```riscv64-linux-gnu-gcc (Debian 10.3.0-8) 10.3.0```
	- Checkout to the `util` branch (```git checkout util```), then run ```make qemu```, you should get a shell and an output to ```ls``` like this:
```
$ ls
.              1 1 1024
..             1 1 1024
README         2 2 1982
xargstest.sh   2 3 93
cat            2 4 22360
echo           2 5 21304
forktest       2 6 11976
grep           2 7 25648
init           2 8 21928
kill           2 9 21192
ln             2 10 21056
ls             2 11 24608
mkdir          2 12 21312
rm             2 13 21296
sh             2 14 39408
stressfs       2 15 22280
usertests      2 16 104752
wc             2 17 23360
zombie         2 18 20576
cow            2 19 28168
uthread        2 20 23968
call           2 21 21136
kalloctest     2 22 25992
bcachetest     2 23 26800
mounttest      2 24 33064
crashtest      2 25 22176
console        3 26 0
```

## Projects
We will have five projects, each counting 20% of the final grade, aimed to making several improvements on xv6:
1. [Unix Uilities: sleep, find, xargs](doc/lab1.md)
2. [Memory Allocation](doc/lab2.md)
3. [Copy-On-Write](doc/lab3.md)
4. [File System: large files and symbolic links](doc/lab4.md)
5. [mmap](doc/lab5.md)

## Grading
Each lab will be graded based on:
- Source code and comments
- Lab report
- Tests created and passed
- In-person answers to questions regarding the work ("demo") with TA

## Late Submissions
- Late submissions within 48 hours will be graded with 20% penalty
- Late submissions beyond 48 hours will not be graded
- Exceptions may only be granted case by case with evidence presented
