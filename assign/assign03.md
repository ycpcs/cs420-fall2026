---
layout: default
course_number: CS420
title: "Assignment 3: Semaphore Fun"
---



<br>

### About This Assignment

--- --- --- --- --- --- --- --- --- --- --- --- --- --- --- --- --- --- --- --- ---

The goal of this programming assignment is to get you working with semaphores/mutexes. You will write a program that forks some number of child processes. Each of those child processes will spawn some number of worker threads. **All threads from all processes will concurrently attempt to read and then write data to a shared file.** The reading and writing of this file must be done in a way that does not corrupt the file and produces the desired output.

There are a variety of semaphore implementations that one could use to protect access to shared files and data. For this assignment, you will use **POSIX named semaphores**. You can read everything you need to know about writing programs that use semaphores and named semaphores [here](https://web.archive.org/web/20161208112748/http://www.linuxdevcenter.com:80/pub/a/linux/2007/05/24/semaphores-in-linux.html). Please read all six pages of the linked semaphore documentation before proceeding. The information relevant to this assignment begins in the section on **POSIX named semaphores**, which starts on page 4.


<br>

### Getting Started

 --- --- --- --- --- --- --- --- --- --- --- --- --- --- --- --- --- --- --- --- ---

Download the [here](assign03_semaphores.zip). The assignment is distributed as a **CLion** project that includes multiple run configurations and unit tests to simplify development and debugging.

This assignment includes the following features:

- Automatic cleanup of semaphores left behind from a previous run -- a small utility called **`rmsem`** will automatically remove those stale semaphores at the start of each new run.
- Standalone testing and debugging of the child process -- the child process (**`fileWriter`**) can be run independently, without the parent process.


<br>

### Your Task

--- --- --- --- --- --- --- --- --- --- --- --- --- --- --- --- --- --- --- --- ---

Your task is to write two programs (**`mainProc`** and **`fileWriter`**). As noted above, **`mainProc`** will fork multiple instances of **`fileWriter`**, and **`fileWriter`** will spawn many threads. All threads must run concurrently and attempt to access, read, and write a common file. **Your goal is to ensure orderly access to the common file and to guarantee that all threads eventually get a chance to read and write it.**

> **Important:** It is strongly recommended that you begin by writing the **`fileWriter`** program. Only after **`fileWriter`** passes all supplied unit tests should you move on to writing **`mainProc`**. Focusing on **`fileWriter`** first will make debugging considerably easier.


<br>

### Creating the mainProc Program

--- --- --- --- --- --- --- --- --- --- --- --- --- --- --- --- --- --- --- --- ---

The **`mainProc`** program must accomplish the following:

 - Parse command-line arguments (described in more detail below)
 - Open and initialize the shared file (filename specified as an input argument)
 - Create and initialize the **named semaphore**
 - Fork some number of child processes (specified as an input argument)
 - Wait for child processes to complete
 - Close and clean up the **named semaphore**

<br>

#### <u>Before You Can Compile</u>

You will need to define a name for your **named semaphore**. Open the **`sem_name.h`** file, uncomment the line **`// #define YCP_USER_NAME "YOUR_YCP_USERNAME"`**, and replace **`YOUR_YCP_USERNAME`** with your YCP username. This value is used to assign a value to the provided **`const char* SEM_NAME`**, also located in **`sem_name.h`**. Use **`SEM_NAME`** throughout your code wherever you need to access the named semaphore. Using your YCP username as the semaphore name will make it easier to recognize and clean up any semaphores you create.


<br>

#### <u>Parsing Arguments</u>

Your **`mainProc`** program **must** accept three command-line arguments (**`-p`**, **`-t`**, and **`-f`**). The usage statement, with an explanation of each argument, is shown below:

<pre>
usage: ./mainProc -p &lt;num_procs&gt; -t &lt;num_threads&gt; -f &lt;filename&gt;
    -p : the number of processes to create
    -t : the number of threads to create per process
    -f : the name of the shared file in which to write output
</pre>

**`mainProc`** requires option flags to be passed on the command line along with their associated values. Using option flags makes it possible to pass arguments in any order, or in some cases to pass only a subset of all possible arguments. I highly recommend you read about and use the **`getopt`** function from **`unistd.h`** to parse the command-line options and their associated values. **Using `getopt` is required for full credit on this assignment**. The man page for **`getopt`** can be found [here](http://pubs.opengroup.org/onlinepubs/009696899/functions/getopt.html#tag_03_234).


<br>

#### <u>Writing the First Value</u>

When **`mainProc`** is run, it should create a new file using the filename specified by the **`-f`** argument. It should write the integer value `0` followed by a newline (`\n`) as the first line of the file -- this will be the only line for now. After writing the `0`, the main process should close the file; it will be reopened later. **Note:** utility functions **`open_file`** and **`close_file`** are provided in **`utils.c`**. Use them -- they accept the same arguments as **`fopen`** and **`fclose`** but handle all necessary error checking for you.



<br>

#### <u>Creating a Named Semaphore</u>

Next, **`mainProc`** will need to create a **named semaphore**. A named semaphore is maintained by the operating system and is easy to share between unrelated processes -- there is no need to manually create a shared memory space as might be required with other semaphore types. You should have already read about programming with semaphores [here](https://web.archive.org/web/20161208112748/http://www.linuxdevcenter.com:80/pub/a/linux/2007/05/24/semaphores-in-linux.html).

If you have not yet defined a name for your **named semaphore**, open **`sem_name.h`**, uncomment the line **`// #define YCP_USER_NAME "YOUR_YCP_USERNAME"`**, and replace **`YOUR_YCP_USERNAME`** with your YCP username. Use **`SEM_NAME`** throughout your code wherever you need to access the named semaphore.

When working with **named semaphores**, be sure to destroy the semaphore before your program terminates -- both on normal exit and on error exit.

**Note:** While debugging, you may end up in a situation where you created and locked the named semaphore but failed to unlock or destroy it before exiting. Because named semaphores are managed by the operating system, this means the next time you run your program it will block trying to acquire the semaphore.

A small utility program runs automatically each time you launch your program via a **CLion** Run Configuration or a **`Makefile`** target. This utility checks for and removes any semaphore left over from a previous run. The utility is included as **`rmsem.c`** and can also be run manually if needed.

To manually run **`rmsem`** with an explicit semaphore name, type the following in your terminal:

<pre>
./rmsem -s "YOUR_YCP_USERNAME"
</pre>

Alternatively, if you have set your semaphore name in **`sem_name.h`**, you can run **`rmsem`** with no arguments:

<pre>
./rmsem
</pre>

The utility outputs a message indicating whether the semaphore existed and whether it was successfully removed.



<br>

#### <u>Creating Child Processes</u>

**`mainProc`** should create **`P`** new child processes (the number specified via **`-p`**) that **run concurrently**. You must fork all processes before calling **`wait(NULL)`** -- do not wait for one child process to finish before forking the next.

Each child process should call **`execlp`** to run the **`fileWriter`** program. When launching **`fileWriter`**, you need to pass the following arguments via **`execlp`**: (1) the number of threads the child **`fileWriter`** process should create (specified via **`-t`**), and (2) the filename that **`fileWriter`** will read and write. Both values are already available as arguments to **`mainProc`** -- simply pass them along.


<br>

### Creating the fileWriter Program

--- --- --- --- --- --- --- --- --- --- --- --- --- --- --- --- --- --- --- --- ---

The **`fileWriter`** program must accomplish the following:

 - Parse command-line arguments (described in more detail below)
 - Create and initialize the named semaphore (when forked from **`mainProc`**, **`fileWriter`** simply opens the existing semaphore -- the code to create a semaphore is identical to the code to open an existing one)
 - Spawn some number of worker threads (specified as **`argv[1]`**, passed either from the command line in standalone mode or from **`execlp`** in **`mainProc`**); all worker threads must run concurrently -- do not wait for one thread to finish before spawning the next
 - In each thread -- open, read, then write the shared file (the filename is specified as **`argv[2]`**, passed either from the command line in standalone mode or from **`execlp`** in **`mainProc`**)
 - Wait for all worker threads to complete
 - Close the **named semaphore**



<br>

#### <u>Parsing Arguments</u>

Your **`fileWriter`** program takes two command-line arguments. The usage statement, with an explanation of each argument, is shown below:

<pre>
usage: ./fileWriter &lt;num_threads&gt; &lt;filename&gt;
    &lt;num_threads&gt; : the number of threads to create
    &lt;filename&gt;    : the name of the shared file to read and write
</pre>

**Note** that **`fileWriter`** accepts bare positional arguments in **`argv[1]`** and **`argv[2]`** -- there are no flags (unlike **`mainProc`**, which uses **`-t`** and **`-f`**). Because there are no flags, arguments must be passed in the exact order specified above.

<br>

#### <u>Opening a Named Semaphore</u>

Your **`fileWriter`** process will need to create or open a **named semaphore**. As described earlier, a named semaphore is maintained by the operating system and is easy to share between unrelated processes. You should have already read about programming with semaphores [here](https://web.archive.org/web/20161208112748/http://www.linuxdevcenter.com:80/pub/a/linux/2007/05/24/semaphores-in-linux.html).

If you have not yet defined a name for your **named semaphore**, open **`sem_name.h`**, uncomment the line **`// #define YCP_USER_NAME "YOUR_YCP_USERNAME"`**, and replace **`YOUR_YCP_USERNAME`** with your YCP username. Use **`SEM_NAME`** throughout your code wherever you need to access the named semaphore.

When opening the **named semaphore** in **`fileWriter`**, use the same call you used in **`mainProc`**, including setting the initial value to `1`. When **`fileWriter`** is forked from **`mainProc`**, attempts to reinitialize the existing semaphore will have no effect. When running **`fileWriter`** as a standalone process for debugging, the semaphore will be both created and initialized so it is available for use throughout the standalone **`fileWriter`** process.

**Important:** The **named semaphore** must be destroyed in **`mainProc`**, not in **`fileWriter`**. If any single **`fileWriter`** process destroys the semaphore, it will be unavailable to all other **`fileWriter`** processes. This also means that when running **`fileWriter`** as a standalone process, the semaphore will **not** be destroyed on exit. However, as described earlier, the **`rmsem`** utility included with the assignment will handle cleanup automatically when using the provided **CLion** Run Configurations and/or **`Makefile`** targets.

To run **`rmsem`** manually, type the following in your terminal:

<pre>
./rmsem -s "YOUR_YCP_USERNAME"
</pre>

Alternatively, if you have set your semaphore name in **`sem_name.h`**, you can run:

<pre>
./rmsem
</pre>

The utility outputs a message indicating whether the semaphore existed and whether it was successfully removed.



<br>

#### <u>Creating Threads</u>

Your **`fileWriter`** process must spawn **`T`** new threads that all **run concurrently** -- spawn all threads before calling **`pthread_join`** and waiting for any to finish. The value **`T`** is passed into **`fileWriter`** as **`argv[1]`**. Do not wait for one thread to finish before spawning the next.

When first creating your threads, do not attempt to make them do any work yet. Simply get thread creation and termination working correctly. The thread function should just print something simple to the terminal at this stage.

The following references may be helpful:

 - [POSIX Threads ('pthreads') Reference](http://en.wikipedia.org/wiki/POSIX_Threads)
 - [pthread.h](http://pubs.opengroup.org/onlinepubs/007908799/xsh/pthread.h.html)



<br>

#### <u>Processing the File / Worker Thread Function</u>

If you are reading this, your **`fileWriter`** program should be able to successfully create and terminate threads. **If your code does not yet do this, stop here and get those pieces working first.**

It is time to make the worker threads actually do some work. Each thread should attempt to read from and write to the same file that was originally created by **`mainProc`** (or by the build setup when running **`fileWriter`** in standalone mode). Each thread should:

 - Attempt to open the shared file
 - Read the **last** numeric value in the file (the first thread to access the file will read the `0` written by **`mainProc`**); note that the last value may be a single digit or a multi-digit number -- your code should handle both cases
 - Increment the value read from the file and **append** the newly incremented value to the file, each on its own line

Use your **named semaphore** as needed to protect access to the shared file. When all threads complete and all processes terminate, the shared file should contain the numeric values `0` through `(P * T)` **in ascending order**. If values are not in ascending order, or if any are repeated or missing, re-examine how you are using the semaphore.

**File Processing Tips:**

 - Consider using **`fseek`**, **`ftell`**, **`getc`**, and **`fscanf`** for reading and writing the file. Read the **`man`** pages for these functions and use them as appropriate.
 - Do **not** read the entire file with each new thread, as that produces O(n²) behavior. Instead, use **`fseek`** to jump to the end of the file and walk backwards to find the last line.


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

Multiple pre-defined **Run Configurations** are provided for both **`mainProc`** and **`fileWriter`**, each with different parameters. To ease debugging, **`fileWriter`** can be run as a standalone process without being forked from **`mainProc`**. Select one of the provided configurations (e.g., **`mainProc -- procs=2, threads=3`** or **`fileWriter (standalone) -- threads=5`**) and click the **Run** icon (green play button) in the toolbar. If your program runs and produces an output file, it will appear in the **`./test_outputs`** subdirectory of your project directory. You can open the file to examine its contents.

**Sample Output File**

If your program is run with **`-p 5`** and **`-t 3`**, it should create **5 `fileWriter` processes**, each with **3 threads**, for a **total of 15 threads** all attempting to access and write to the shared file. If successful, the contents of the shared file should look **exactly** like the following after all 15 threads complete. The first line of the file should contain the `0` written by **`mainProc`**, followed by each additional value on its own line.

<pre>
0
1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
</pre>

When running any of the **`fileWriter (standalone)`** configurations, you will also see output in CLion similar to the following (marked either as **SUCCESS** or **EPIC FAIL**):

<pre>
Verification of file contents: 
-SUCCESS-
</pre>

When running any of the **`mainProc`** configurations, you will also see output similar to the following:

<pre>
Verification of file contents: 
-SUCCESS-

Verification of semaphore cleanup: 
-SUCCESS-
</pre>

These verification functions are included in the assignment distribution and run automatically. They check whether your output file has the expected contents. When running **`mainProc`**, a second verification function also checks that the named semaphore was properly cleaned up. The verification functions are defined in **`verify.c`**, and the calls to them are already included in **`mainProc.c`** and **`fileWriter.c`**. **Do not modify the verification functions or the code that calls them.**


#### Running in the Terminal

After compiling, run your program from within the project directory as you would any other command-line utility. For example, the following runs **`mainProc`** with **2** `fileWriter` processes, each spawning **3** threads, writing output to **`./test_outputs/mainProcTest1_output.txt`**:

<pre>
<b>./mainProc -p 2 -t 3 -f ./test_outputs/mainProcTest1_output.txt</b>
</pre>

If your program runs correctly, it should produce the requested output file and display output similar to the sample shown above.


<br>

### Testing Your Program

--- --- --- --- --- --- --- --- --- --- --- --- --- --- --- --- --- --- --- --- ---

The assignment includes embedded tests that use the **`CTest`** tool, which can also be run from the terminal via the **`Makefile`**.

#### Testing in CLion

Select one of the pre-defined **CTest** Run Configurations (e.g., **`mainProc Test #1`** or **`fileWriter Test #2`**) and click the **Run** icon. A test window will appear, execute the selected tests, and display the results.

#### Testing in the Terminal

Use the supplied **`Makefile`** to run tests. To see all available test targets, run `make` with no arguments. The following targets are available for this assignment:

<pre>
mainProc Test Targets:
  mainProcTest1           Run Test #1: mainProc with 2 processes and 3 threads
  mainProcTest2           Run Test #2: mainProc with 8 processes and 12 threads
  mainProcTest3           Run Test #3: mainProc with 25 processes and 30 threads
  mainProcTests           Run all mainProc tests with full verbose output
  mainProcTestsSummary    Run all mainProc tests and show PASSED/FAILED summary only

fileWriter Test Targets:
  fileWriterTest1         Run Test #1: fileWriter_standalone with 5 threads
  fileWriterTest2         Run Test #2: fileWriter_standalone with 50 threads
  fileWriterTest3         Run Test #3: fileWriter_standalone with 500 threads
  fileWriterTests         Run all fileWriter tests with full verbose output
  fileWriterTestsSummary  Run all fileWriter tests and show PASSED/FAILED summary only
</pre>

To run a specific test, pass its target to `make`. For example: `make mainProcTest1`. For a summary of all test results, run `make mainProcTestsSummary`.


<br>

### Grading Criteria

--- --- --- --- --- --- --- --- --- --- --- --- --- --- --- --- --- --- --- --- ---

This assignment is graded on a 100-point scale. The breakdown is as follows:

**main process:**
 - **5 points** - Correctly parses and error-checks command-line arguments
 - **5 points** - Uses **`getopt`** to parse arguments
 - **5 points** - Successfully writes the initial `0` to the file
 - **10 points** - Correctly creates a named semaphore
 - **10 points** - Correctly forks `P` new processes
 - **5 points** - Correctly executes the child process
 - **10 points** - Correctly closes the named semaphore

**fileWriter process:**
 - **5 points** - Correctly receives arguments passed from the main process
 - **5 points** - Correctly attaches to the named semaphore
 - **10 points** - Correctly spawns `T` new threads
 - **5 points** - Correctly locks and unlocks the semaphore as necessary
 - **5 points** - Threads open, read from, and write to the shared file
 - **5 points** - All data is written correctly to the shared file
 - **10 points** - Data is written to the file without re-reading every line with every thread
 - **5 points** - Correctly closes the named semaphore

Points may be deducted for poor coding practices, such as failing to check error conditions or failing to close files when finished.


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

This produces `assign03_submission.zip` in your project directory. Upload this file through the [Marmoset web interface](https://cs.ycp.edu/marmoset).
