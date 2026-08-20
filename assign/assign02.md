---
layout: default
course_number: CS420
title: "Assignment 2: Interprocess Communication"
---


<br>

### About This Assignment

--- --- --- --- --- --- --- --- --- --- --- --- --- --- --- --- --- --- --- --- ---

In this programming assignment you will learn about and use the following POSIX system calls: **`fork()`**, **`exec()`**, **`shmget()`**, **`shmat()`**, **`shmdt()`**, **`shmctl()`**.  If you do not use each of these system calls at least once, you have probably done something wrong. You should also familiarize yourself with the **man pages** for each of these system calls. 

For example, to read about the **`shmget`** function, run the following command from your terminal:

```
man shmget
```

A **`man`** query via [Google](https://letmegooglethat.com/?q=man+shmget) will also return the appropriate documentation.

Once again, you should be using a **POSIX-compliant environment**, such as Linux, to write your programs. 

For this assignment, you are permitted to use **`printf`** and other functions in the **POSIX API** for your program I/O.


<br>

### Getting Started

 --- --- --- --- --- --- --- --- --- --- --- --- --- --- --- --- --- --- --- --- ---

Download the [assignment](assign02_shared_memory.zip). The assignment is distributed as a **CLion** project that includes multiple run configurations and unit tests to simplify development and debugging.


<br>

### Your Task

--- --- --- --- --- --- --- --- --- --- --- --- --- --- --- --- --- --- --- --- ---

Your task is to write two programs (**`mainProc`** and **`childProc`**) that communicate with each other through shared memory. The **`mainProc`** must request shared memory from the operating system and store data in that shared memory. The **`mainProc`**  must then fork **`childProc`**, which reads from and writes data back into the shared memory. Finally, **`mainProc`** must read and print the contents of the shared memory.


<br>

### Creating the mainProc Program

--- --- --- --- --- --- --- --- --- --- --- --- --- --- --- --- --- --- --- --- ---

The **`mainProc`** program must accomplish the following:

 - Parse command-line arguments (described in more detail below)
 - Request a segment of shared memory from the kernel
 - Attach to the shared memory
 - Write a value to the shared memory
 - Fork a child process
 - Wait for the child process to complete
 - Print the contents of the shared memory buffer (written by the child process)
 - Detach from the shared memory
 - Destroy the segment of shared memory
 - Perform all required error checking and reporting for system calls

 
<br>

#### <u>Parsing Arguments</u>

Your **`mainProc`** program **must** accept a single command-line argument. The usage statement for **`mainProc`**, with an explanation of the argument, is shown below:

<pre>
usage: ./mainProc &lt;repeat_val&gt;
    &lt;repeat_val&gt; : the number of times the child process is required
                   to write a string to a shared memory buffer
</pre>

Your **`mainProc`** program should accept this argument as an integer. This value will eventually be passed to the child process through shared memory.


<br>

#### <u>Requesting, Attaching to, and Writing to Shared Memory</u>

Before creating the child process, **`mainProc`** should use the **`shmget()`** system call to request a chunk of shared memory from the operating system. The amount of memory requested should be **`sizeof(struct ipc_struct)`**. The definition of **`ipc_struct`** is in the included **`ipcEx.h`** header file. The **`shmget()`** system call returns a **`segment_id`** that uniquely identifies the new chunk of memory. Note that the command-line argument received by **`mainProc`** will eventually be passed to the child process by storing it in the **`repeat_val`** field of the **`ipc_struct`**.

After requesting the shared memory, **`mainProc`** should attach to it using the **`shmat()`** system call. As discussed in class, the block of shared memory has no inherent structure. To give it structure from the perspective of the main process, you can map an **`ipc_struct`** onto the shared memory as follows:

```c
/* cast the memory pointer returned by shmat as a (struct ipc_struct *),
 * then assign that to a variable defined as a (struct ipc_struct *) */
struct ipc_struct* shared_memory = (struct ipc_struct*) shmat( /* INSERT ARGS HERE */ );

/* access the members of the struct in typical C/C++ fashion */
shared_memory->repeat_val =  ...
```

Note that the above code **does not allocate any new memory**. It simply attaches to and creates a pointer to the memory already allocated by **`shmget()`**. The **`repeat_val`** can then be written to shared memory as shown.


<br>

#### <u>Creating the Child Process</u>

Now that the shared memory has been created, **`mainProc`** should use the **`fork()`** system call to create a new child process. Recall that when **`fork()`** is called, both the parent process and the child process continue executing from that point forward. To have the child process run your **`childProc`** program, use the **`execlp()`** system call to replace the child process's memory contents. The **`execlp()`** system call allows you to pass arguments to the program being launched. For this assignment, you will need to pass the **`segment_id`** as an argument to the child process so that it knows where to find the shared memory.


<br>

#### <u>Waiting for the Child Process and Printing the Shared Memory Buffer</u>

While **`childProc`** is running, the parent process should wait for the child to complete using the **`wait`** system call. When the child exits, the parent must retrieve the string data written to the shared memory by the child process and print it to the terminal. You **must** print the complete contents of shared memory using a single call to **`printf`**. Your **`printf`** call should be placed between the two provided **`printf`** statements already included in **`mainProc.c`**, as shown below. The **`Buffer start`** and **`Buffer end`** delimiters are required by the included unit tests -- **do not remove or modify them**.

```c
printf("============= Buffer start =============\n");

// Put your code to print the shared memory buffer here

printf("============= Buffer end ===============\n");
```


<br>

#### <u>Detaching from and Destroying the Shared Memory</u>

Finally, **`mainProc`** should detach from the shared memory using the **`shmdt()`** system call and release the shared memory using the **`shmctl()`** system call.


<br>

#### <u>Error Checking and Reporting</u>

You **must** check the return value of each and every system call. Use **`perror`** to print a human-readable error message to standard error for any error that occurs. Do **not** use **`printf`** for error reporting.


<br>

### Creating the childProc Program

--- --- --- --- --- --- --- --- --- --- --- --- --- --- --- --- --- --- --- --- ---

The **`childProc`** program must accomplish the following:

 - Parse incoming arguments (described in more detail below)
 - Attach to the already-existing shared memory segment
 - Read the **`repeat_val`** value from the shared memory
 - Write a data string to the shared memory buffer **`repeat_val`** number of times
 - Detach from the shared memory
 - Perform all required error checking and reporting for system calls


<br>

#### <u>Parsing Arguments</u>

Your **`childProc`** program should accept a single argument in **`argv[1]`** that represents the unique **`segment_id`** of the shared memory. This value is passed, as an argument, from **`mainProc`** to **`childProc`** via **`execlp()`**. Note that **`argv[0]`** is the name of the program as invoked from the command line (null-terminated); in this case, it should contain the string **`"childProc\0"`**.


<br>

#### <u>Attaching to and Reading Shared Memory</u>


Your **`childProc`** program should attach to the shared memory segment using the **`shmat()`** system call. Just as in **`mainProc`**, the child process should map an **`ipc_struct`** onto the shared memory to simplify access. **`childProc`** can then read and use the **`repeat_val`** integer directly from the shared memory -- there is no need to make a local copy.


<br>

#### <u>Writing to the Shared Memory</u>

Replicate the provided **`data_string`** exactly **`repeat_val`** times into the shared memory data buffer, but only if all the data will fit within the provided 128-byte **`data`** buffer. For example, if the total data to be written is 120 bytes, write it as requested. However, if the total data exceeds the buffer size (e.g., 200 bytes), do not write anything -- leave the buffer empty, report the error, and call **`exit`**. All writes to the **`data`** buffer must be done using **`snprintf()`**.


<br>

#### <u>Detaching from the Shared Memory</u>

Detach from the shared memory using the **`shmdt()`** system call, then exit **`childProc`**.


<br>

#### <u>Error Checking and Reporting</u>

You **must** check the return value of each and every system call. Use **`perror`** to print a human-readable error message to standard error for any error that occurs. Do **not** use **`printf`** for error reporting.


<br>

### Compiling Your Program

--- --- --- --- --- --- --- --- --- --- --- --- --- --- --- --- --- --- --- --- ---

The assignment is distributed as a **CLion** project but also includes a standard **`Makefile`**, so it can be compiled either in **CLion** or in a terminal. It is recommended that you use **CLion** for development and debugging. However, before submitting, verify that your program compiles and runs correctly using the provided **`Makefile`**.

#### Compiling in CLion

The quickest way to compile in CLion is to click the **Build** icon (hammer) in the toolbar, or select **Build → Build Project** from the menu. Because the **Build** command only recompiles changed files, you should occasionally run **Build → Rebuild Project** to ensure all compilation warnings have been addressed.

#### Compiling in the Terminal

Your code **must** compile with the supplied **`Makefile`** before submission. Submissions that do not compile receive very little credit. To see all available targets, run `make` with no arguments. To compile your code, run:

```
make all
```

Note that `make all` performs a `make clean` before compiling, so it is effectively a full rebuild.

<br>

### Running Your Program

--- --- --- --- --- --- --- --- --- --- --- --- --- --- --- --- --- --- --- --- ---

#### Running in CLion

Select one of the pre-defined **Run Configurations** (e.g., **`mainProc 3`**) and click the **Run** icon (green play button) in the toolbar. If your program runs correctly, it should produce output similar to the following:

<pre>
PARENT: Created shared memory with a segment ID of 393216
PARENT: The child process should store it's string in shared
        memory a total of 3 times.
  CHILD: Received 2 arguments
  CHILD: Attempting to access segment ID 393216...
  CHILD: Parent requested that I store my data 3 times
  CHILD: Done copying data, exiting


PARENT: Child with PID=27813 complete
PARENT: Child left the following in the data buffer:
============= Buffer start =============
Hello Shared Memory!
Hello Shared Memory!
Hello Shared Memory!
============= Buffer end ===============

PARENT: Done

Process finished with exit code 0
</pre>

The specific **`PARENT`** and **`CHILD`** print statements need not match this output exactly, but you should use similar statements to show the progress of both processes. Each print statement should begin with either **`PARENT:`** or **`CHILD:`** to clearly indicate which process is producing the output. As noted previously, the **`Buffer start`** and **`Buffer end`** delimiter lines must match the above exactly and are included in the provided source code.

#### Running in the Terminal

After compiling, run your program from within the project directory as you would any other command-line utility. For example, the following runs **`mainProc`** with the argument **`3`**, indicating that **`childProc`** should write three copies of the data to the buffer:

<pre>
<b>./mainProc 3</b>
</pre>

If your program runs correctly, it should produce output similar to the sample shown above.


<br>

### Testing Your Program

--- --- --- --- --- --- --- --- --- --- --- --- --- --- --- --- --- --- --- --- ---

The assignment includes embedded tests that use the **`CTest`** tool, which can also be run from the terminal via the **`Makefile`**.

#### Testing in CLion

Select one of the pre-defined **CTest** Run Configurations (e.g., **`Test #1`**) and click the **Run** icon. A test window will appear, execute the selected tests, and display the results.

#### Testing in the Terminal

Use the supplied **`Makefile`** to run tests. To see all available test targets, run `make` with no arguments. The following targets are available for this assignment:

<pre>
Test Targets:
  mainProcTest1         Run Test #1: mainProc requesting 3 copies
  mainProcTest2         Run Test #2: mainProc requesting 5 copies
  mainProcTest3         Run Test #3: mainProc requesting 6 copies
  mainProcTest4         Run Test #4: mainProc requesting 7 copies
  mainProcTest5         Run Test #5: mainProc requesting 25 copies
  mainProcTests         Run all tests with full verbose output
  mainProcTestsSummary  Run all tests and show PASSED/FAILED summary only
</pre>

To run a specific test, pass its target to `make`. For example: `make mainProcTest1`. For a summary of all test results, run `make mainProcTestsSummary`.


<br>

### Grading Criteria

--- --- --- --- --- --- --- --- --- --- --- --- --- --- --- --- --- --- --- --- ---

This assignment is graded on a 100-point scale. The breakdown is as follows:

**mainProc:**
- **5 points** - Compiles correctly
- **5 points** - Correctly parses command-line arguments
- **5 points** - Correctly creates a shared memory segment
- **5 points** - Correctly accesses a shared memory segment
- **5 points** - Correctly writes `repeatVal` to shared memory
- **10 points** - Correctly forks a child process
- **5 points** - Correctly passes arguments to `childProc`
- **10 points** - Correctly prints data stored in the shared memory buffer
- **5 points** - Correctly detaches from shared memory
- **5 points** - Correctly destroys the shared memory segment

**childProc:**
- **5 points** - Compiles correctly
- **5 points** - Correctly parses command-line arguments
- **5 points** - Correctly accesses a shared memory segment
- **5 points** - Correctly retrieves `repeatVal` from shared memory
- **5 points** - Correctly writes data to shared memory
- **10 points** - Correctly writes to shared memory without using `strlen` and/or `strcat` in a loop
- **5 points** - Correctly detaches from shared memory

Points may be deducted for poor coding practices, such as failing to check error conditions.


<br>

### Submitting to Marmoset

--- --- --- --- --- --- --- --- --- --- --- --- --- --- --- --- --- --- --- --- ---

> **<font color="red">⚠ Do not manually zip your project and upload it to Marmoset.** Use one of the submission methods below.</font>

You can submit your assignment from within **CLion** or from a terminal.

Before submitting:

1. **Rebuild** your program in **CLion** (or run `make all` in the terminal).
2. **Address every compiler warning**, there are no acceptable warnings on this or any other assignment. All warnings indicate something has been done incorrectly.
3. Goto #1 until no more warnings exist


#### Submitting from CLion

If you have not yet installed the JetBrains **[YCPCS Marmoset Submitter](https://plugins.jetbrains.com/plugin/30901-ycpcs-marmoset-submitter)** plugin, follow the instructions on the [YCPCS DevEnv Guide](https://ycpcs.github.io/dev-env-setup-guide/prog_envs/common/ycpcs_marmoset_plugin.html).

Once installed, click the **Submit to Marmoset** icon (![image](https://ycpcs.github.io/dev-env-setup-guide/prog_envs/common/common_plugin_images/MarmosetSubmitIcon.svg)) in the CLion toolbar, or select **Menu → Tools → Submit to Marmoset**.


#### Submitting from the Terminal

From the project's source directory, run:

<pre>
<b>make submit</b>
</pre>

You will be prompted for your Marmoset username and password (received by email). Your password will not be echoed to the screen.

If `make submit` does not work, you can instead create a submission archive with:

<pre>
<b>make submission.zip</b>
</pre>

This produces `assign02_submission.zip` in your project directory. Upload this file through the [Marmoset web interface](https://cs.ycp.edu/marmoset).
