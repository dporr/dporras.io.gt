---
title: "Kage bunshin no jutsu: clone(2)"
date: 2023-01-07T23:07:09Z
draft: true
image: images/kagebunshin.jpeg
---

This entry talks about the clone(2) system call and the corresponding libc thin wrapper.
I talk about some issues I found whle using the syscall to solve [Advent - day 2](https://osg.tuhh.de/Advent/02-clone/).

#### TL;DR
- Process creation in linux
- Namespaces in linux: CLONE_NEW*, CLONE_NEWNS, user namespaces.
- Linux capabilities. CAP_SYS_ADMIN
- Practical use of the namespaces API: clone(2)
- My full solutions are available here: https://github.com/dporr/kernel_syscalls/tree/master/02-clone

#### It all starts with fork(2):

https://man7.org/tlpi/download/TLPI-24-Process_Creation.pdf

The most basic way of creating a new process in linux is using the `fork(2)` sycall. This syscall creates a new process with a new PID and the value of parent PID pointing to the caller of `fork`. If we take a look at the process representation inside of the kernel, we see that the task definition of te current process points to that PID in [`struct task_struct->parent`](https://elixir.bootlin.com/linux/v5.16/source/include/linux/sched.h#L958) and  [`struct task_struct->real_parent`](https://elixir.bootlin.com/linux/v5.16/source/include/linux/sched.h#L955) *Note that task_struct resides on kernel memory and thus is unmapped in userspace, therefore we need a driver or something allowed to read kernel memory to appreciate this fact at runtime.

The disadvantage of fork is that the new process shares a copy of the parent's stack, namespace and open file descriptors among other things. This by itself may not be an issue, but flexibility during process creation perhaps is.

#### clone(2)
In the other hand we have a more flexible way of instantiating a process by calling clone(2) and clone3(2). As stated in the manpages 
>By contrast with fork(2), these system calls provide more precise
       control over what pieces of execution context are shared between
       the calling process and the child process.  For example, using
       these system calls, the caller can control whether or not the two
       processes share the virtual address space, the table of file
       descriptors, and the table of signal handlers.  These system
       calls also allow the new child process to be placed in separate
       namespaces(7).<< The clone wrapper in libc has the following prototype:
```c
extern int clone (int (*__fn) (void *__arg), void *__child_stack,
                  int __flags, void *__arg, ...) __THROW;
```

Interesting diferences is that we need to specify the function from where execution will start in the child and we also need to specify a memory area for the child stack, using different flags we can specify other properties of the child like  wich group thread this process will belong, if we need the child to have a separate namespace from the parent and other interesting stuff.

### Solving advent of code day 2

There are 4 tasks for this day:
    
    1. Implement fork() with clone().
    2. Create a process-thread chimera, i.e a process which shares its address space with a different process, with clone().
    3. Create a real thread with clone() that shares the address space and that is put in the same thread group.
    4. (Extra) Fork a process into a new UID namespace and become root therein. getuid() will show you your success.

While solving the first of them with the most basic code I faced some errors:
```c
...
if (!strcmp(argv[1], "fork")) {
        int child_pid = clone(child_entry, &stack[4096] , CLONE_NEWNS, argv[1]);
        syscall_write("Created process: ", child_pid);
        if(child_pid == -1) perror("clone");
...

$ make
$ ./clone fork
Created process: -1
clone: Operation not permitted
```

The flexibility clone offers is subject to the user having the right access level, that's when capabilities come to play. The concept of capabilities  in linux isn't new to me. I have used them for privesc before, but observing why and when are they required teaches in a different way. If you observe the code you can notice I'm passing the `CLONE_NEWNS` flag and this flag requires `CAP_SYS_ADMIN`. In the manpage for linux capabilities [capabilities(7)](https://man7.org/linux/man-pages/man7/capabilities.7.html) is stated that
> employ CLONE_* flags that create new namespaces with
                clone(2) and unshare(2) (but, since Linux 3.8, creating
                user namespaces does not require any capability);
              * access privileged perf event information; 

So, what exactly does this mean? Do we need `CAP_SYS_ADMIN` or not? There are, as time of writting, 8 different [namespaces(7)](https://man7.org/linux/man-pages/man7/namespaces.7.html) that offer different type of isolation for global resources. Namespaces are identified with a named numerical costant, like `CLONE_NEWNS`. CLONE_NEWNS indicates that we are creating a new mount namespace, so the exception that appeared since linux 3.8 doesn't benefit us as we are creating a completely different flavor of namespace here.

In summary, this means that only a privileged user/process with the `CAP_SYS_ADMIN` can create a child process in a separate [mount] namespace from its parent.

My solution to the tasks is the follwoing:

##### Task 1
```c
...
if (!strcmp(argv[1], "fork")) {
        flags = SIGCHLD;
    }
[...]
int child_pid = clone(child_entry, &stack[4096] , flags, arg);
    syscall_write("Created process: ", child_pid);
    if(child_pid == -1) perror("clone");
...
```

Nothing special for the "fork" call, with `SIGCHLD` we are simply indicaing that the child will notify the parent process that its execution is completed.

##### Task 2
```c
...
else if(!strcmp(argv[1], "chimera")){
        flags = CLONE_VM;
}
[...]
int child_pid = clone(child_entry, &stack[4096] , flags, arg);
    syscall_write("Created process: ", child_pid);
    if(child_pid == -1) perror("clone");
```
This is basically indicating that the child will share the virtual address space with the parent, meaning all local and global variables could be mutated from either the child or the parent and both parties are able to see those changes. In the end, they are functioning on the very same memory space.

##### Task 3
```c
...
else if(!strcmp(argv[1], "thread")){
        /*
        *Since Linux 2.5.35, the flags mask must also include CLONE_SIGHAND if  CLONE_THREAD
        *is  specified  (and  note  that,  since  Linux  2.6.0,  CLONE_SIGHAND also requires
        *CLONE_VM to be included).
        */
        flags = CLONE_VM | CLONE_THREAD | CLONE_SIGHAND;
    }
[...]
int child_pid = clone(child_entry, &stack[4096] , flags, arg);
    syscall_write("Created process: ", child_pid);
    if(child_pid == -1) perror("clone");
```

This one was a little tricky since `CLONE_THREAD` has a dependency on `CLONE_SIGHAND` and the later has a dependency on `CLONE_VM` since linux 2.5.35 according to the docs.

##### Task 4
```c
...
/*Inside the chikd*/
if(arg != NULL){
        int uid_map_fd;
        /*
        https://man7.org/linux/man-pages/man7/user_namespaces.7.html
        The easies way o complying with the permissions requirements
        is to run this as root.
        */
        if((uid_map_fd = open("/proc/self/uid_map", O_RDWR)) == -1) perror("open");
        char* uid_map = "0 0 1\n";
        if(write(uid_map_fd,uid_map, sizeof(uid_map) - 1) == -1) perror("write");
        close(uid_map_fd);
        syscall_write(": getuid()  = ", getuid());
        if(setuid(0) == -1 ) perror("setuid");
        syscall_write(": setuid() = ", getuid());
    }
...

/*Inside the parent*/
...
else if(!strcmp(argv[1], "user")){
        flags = CLONE_NEWUSER;
        /*We need to signal the child that we want to override /proc/self/uid_map*/
        arg = 1;
    }
[...]
int child_pid = clone(child_entry, &stack[4096] , flags, arg);
    syscall_write("Created process: ", child_pid);
    if(child_pid == -1) perror("clone");
```
The last one was a headache because writting to the file that sets the user and group mappingsa plus using setuid implies a set of permissions... in the end I gave up troubleshooting and just ran this part as root.

I learned a lot reading manpages and observing the different concepts materializing through code. In the end we only truly understand what we can code...
See you arround space cowboy!

#### Additional reading
1. https://ypl.coffee/parent-and-real-parent-in-task-struct/  (Good explanation of task_struct access + Kernel and Hacking stuff from a great hacker and ctf player)  
2. https://lwn.net/Articles/531114/ (Great searies on namespaces, basically the soul of modern containers)
 