## Process Thread and Syncronisation in QNX


# PID, Process, Thread, and Synchronization Notes

> [!TIP]
> Quick reading guide:
> - Focus first on <mark>fork vs exec vs spawn</mark>
> - Then review <mark>process termination flow</mark>
> - Finally, review <mark>thread attributes and scheduling</mark>

### Process Creation
------------
#### <u>`fork()`</u>
- `fork()` creates a copy of calling process.
    - It is an identical copy of parent.
    - Starts from `fork()`.
    - Initially has the same data as parent.
    - In a multithreaded process application this should be avoided, so should be avoided in QNX.
    - `fork()` returns child PID and 0 for child.

- <u>`What gets inherited?`</u>
    - File descriptor
    - Priority
    - Scheduling algorithm
    - Signal mask
    - I/O privilege
    - UID, GID, type ID
    - Address space is replicated
- <u>`What is not inherited?`</u>
    - Side channel connections `(coids)`
    - Channels `(chids)`
    - Timers

> [!WARNING]
> `NOTE - Assume parent has 2 threads T1 and T2. T1 is currently writing a memory location and is not finished yet but T2 created a new child process. As you know fork() creates a copy of memory this means when the child is created the child will have old data. Which is not desirable.`

-------------
#### <u>`exec*()`</u>
- `exec*()` Loads a program from store to change the calling process. <u> `It replaces the caller`</u>.
    - Process ID `PID` remains same.
- <u>`What gets inherited?`</u>   
    - It is exactly the same as `fork()` **except**
        - Address space is created new.
        - Inheritance of `fd` is configurable per `fd` basis.
- Argument or `ENV` variables may be passed to new program.
- These functions do not return unless there is **`error`**.
---------------------
#### <u> `posix_spawn(), spawn*()`</u>
- `posix_spawn(), spawn*()` loads new process by creating new process.
    - Used to load and run a program in `new process`.
    - Returns `PID` of child process.
    - Inheritance is same as `fork()` and `exec*()`. <u> And it is done in the sequence first inherit as `fork()` and then `exec*()` </u>.

> [!NOTE]
> `These are convinence function and and portable to conventional unix systems`

---------------------

#### <u> `fork() exec*() vs spawn` </u>
| `fork() & exec*()`     | `spawn()`                           |
|------------------|---------------------------------------|
| Traditional unix style           | Avoids copy of data segment                  |
| Portable | Avoid intialisation and setup which gets torn down again|
| Inefficient    | Less calls and messages                |
| Complex to implement safely in multithreaded application    | <u> `Recomended way of doing things in QNX` </u>                |


--------------------------------

### Detecting Process Termination

> [!IMPORTANT]
> Keep this section in mind for debugging orphaned or zombie processes.

- When child dies, parent receives `SIGCHLD`.
- `SIGCHLD` does not terminate parent.
- Parent can check why child dies by `waitpid()` or other `wait*()` functions.
    - `wait*()` will block till child is active and returns if child dies.
- If the parent does not `wait*()` on child, the child becomes `ZOMBIE`.
    - `ZOMBIE` does not use CPU.
    - Most resources owned by `ZOMBIE` process are freed.
    - `ZOMBIE` `PID` is not deleted from `PID` table.
    - `ZOMBIE` exit status is held in `PID` table.
- `signal(SIGCHLD, SIG_IGN)` in parent lets parent `IGNORE` the death of child and creation of `ZOMBIE`.

#### <u> `Process termination in case of client - server` </u>

- Server gets notified if client dies.
- Client can get notified if server dies.
- Notification happens on death but can happen whenever client-server relationship is terminated.

#### <u> `Process termination - Death Pulse` </u>

- Application can register for process death using `procmgr_event_notify()`.
- Request `PROCMGR_EVENT_PROCESS_DEATH` notification.
- Application can request any kind of event, but `Death Pulse` is the easiest.


--------------------------

### Thread APIs

> [!NOTE]
> All code snippets in this page are C language snippets and are marked accordingly for GitHub rendering.

- Kernel Functions
    - `ThreadCreate()`
    - `SyncMutexLock()`
- POSIX Functions
    - `pthread_create()`
    - `pthread_mutex_lock()`
- C11 thread functions
    - `thrd_create()`
    - `mtx_lock()`

<u> `POSIX and C11 functions are created on TOP of Kernel functions. These are portable` </u>

#### <u> `Thread Creation` </u>

```c
pthread_create (pthread_t *tid, pthread_attr_t *attr, void *(func) (void *), void *arg);
```
- `void *(func) (void *)` Third argument to the function pthread_create() is called thread function. This is where thread starts executing code.
- `arg` argument passed to your `func()`. It is optional.
- `attr` used to specify thread attribute e.g., `priority`

#### <u> `Initialising attribute` </u>
- `pthread_att_init()`

#### <u> `Destroying attribute` </u>
- `pthread_attr_destry()`

#### <u> `Thread attributes` </u>
- `pthread_attr_setdetachstate()`
- `pthread_attr_setinheritsched()`
- `pthread_attr_setschedparam()`
- `pthread_attr_setschedpolicy()`
- `pthread_attr_setstacksize()`

--------------------------------
#### <u> `Thread attributes - Scheduling Parameters ` </u>
Setting priority and scheduling algorithm:

```c
struct sched_param param;

pthread_attr_setinheritsched(&attr, PTHREAD_EXPLICIT_SCHED);
param.sched_priority = 15;
pthread_attr_setschedparam(&attr, &param);
pthread_attr_setschedpolicy(&attr, SCHED_RR);
pthread_create(NULL, &attr, func, arg);
```

Scheduling policies available include:
- `SCHED_FIFO` - first-in-first-out
- `SCHED_RR` - round-robin
- `SCHED_SPORADIC` - sporadic scheduler
- `SCHED_OTHER` - same as round-robin
- `SCHED_NOCHANGE` - keep same policy, but change parameters (e.g. priority)

-----------------------

#### <u>`Thread Attributes - Stack Allocation`</u>

You can control the thread's stack size:

- To set the stack size:

```c
pthread_attr_setstacksize(&attr, size);
```

- Requirements and behavior:
  - This must be at least `PTHREAD_STACK_MIN`
  - The value will be rounded up to a multiple of the page size (4K)
  - A guard page will be allocated as well, but no RAM will be used for it
    - This can be controlled with `pthread_attr_setguardsize()`

- The default stack size for new threads is **256K** (plus a **4K guard page**)
- The **main** thread gets a **512K** stack by default

---

> [!TIP]
> Revision order suggestion:
> 1. Read `fork() exec*() vs spawn` table
> 2. Read `ZOMBIE` behavior with `wait*()`
> 3. Read scheduling + stack configuration snippets
