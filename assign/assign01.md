---
layout: default
course_number: CS420
title: "Assignment 1: File Copy"
---



<br>

### About This Assignment

--- --- --- --- --- --- --- --- --- --- --- --- --- --- --- --- --- --- --- --- ---

Before starting this assignment, ensure that you have thoroughly read **Section 2.3** of your textbook.

This assignment serves as a warm-up exercise to assess your proficiency in writing programs in **C** and your ability to use the **POSIX API** and system calls. For this and all subsequent programming assignments, you are required to develop your code in a **POSIX-compliant environment** such as Linux or UNIX. If you have your own Linux or UNIX-based system, feel free to use it. Alternatively, you can use the lab computers in KEC119. In either case, refer to the setup guides located [here](https://ycpcs.github.io/dev-env-setup-guide/) to configure your programming environment for this course.

For this assignment, you may only use C system calls for reading and writing to the terminal and disk. You may find the list of C system calls located [here](http://codewiki.wikidot.com/system-calls) useful. Specifically, you will need the following system calls: **`open`**, **`close`**, **`read`**, **`write`**, and **`stat`**. You may also find the functions **`strlen`** and **`exit`** useful. In addition to the website linked above, you can read about these functions using Linux's built-in **man pages** by typing any of the following in your terminal:

<pre>
<b>man 2 open
man 2 close
man 2 read
man 2 write
man 2 stat
man 3 strlen
man 3 exit</b>
</pre>

Searching for any of the above on Google will typically return a nicely formatted HTML version of the desired man page.



<br>

### Getting Started

 --- --- --- --- --- --- --- --- --- --- --- --- --- --- --- --- --- --- --- --- ---

Download the [assignment archive](assign01_filecopy.zip). The assignment is distributed as a **CLion** project that includes multiple run configurations and unit tests to simplify development and debugging.


<br>

### Your Task

--- --- --- --- --- --- --- --- --- --- --- --- --- --- --- --- --- --- --- --- ---

Write a C program called **`filecopy`** that uses system calls to copy one file to another. Read the following requirements carefully before you begin:

 - Your program must accept exactly two command-line arguments: the first is the source file to be copied, and the second is the name of the destination file (i.e., the new copy).

 - Your program must copy both the **contents** and the **permissions** (i.e., the mode) of the source file to the destination file.

 - You **must** use C system calls to open files and to read/write to the terminal and disk.

 - You **may not** use **`fopen`**, **`fclose`**, **`printf`**, **`fprintf`**, **`scanf`**, or similar standard I/O functions.

 - When copying the source file to the destination file, use the buffer provided in **`filecopy.c`**. The buffer is named **`data_buf`** and is `BUFFER_SIZE` bytes. You should attempt to read `BUFFER_SIZE` bytes at a time when copying, not one byte at a time.

 - You must **copy the source file's permissions** to the destination file. You can inspect file permissions by running `ls -l` in your terminal. Note that none of the provided unit tests verify that permissions were copied correctly -- you will need to check these manually. If you are unable to copy the second and third `w` permission bits, this is expected behavior in many environments. See `man 2 umask` for more information.

 - Reading the documentation for each system call carefully is a **central part of this assignment**. You will not be able to complete the assignment without understanding the interfaces you are using. Do not skip this step.

 - The **`strlen`** function, if used properly, can save significant time and effort.

 - **Include thorough error checking.** Every system call returns an error indicator when something goes wrong. **Check the return value of every system call without exception.** If an error occurs, exit gracefully by calling **`exit(EXIT_FAILURE)`**.

 - You do not need to handle filenames containing special characters (e.g., spaces).

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

Select one of the pre-defined **Run Configurations** (e.g., **`filecopy test1_input.txt test1_output.txt`**) and click the **Run** icon (green play button) in the toolbar. If your program succeeds, an output file such as **`test1_output.txt`** will appear in the project file listing.

#### Running in the Terminal

After compiling, run your program from within the project directory as you would any other command-line utility (note the leading **`./`**). For example:

<pre>
<b>./filecopy test1_input.txt test1_output.txt</b>
</pre>

If your program succeeds, **`test1_output.txt`** will appear in the project directory.


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
  test1               Run Test #1: copy a simple text file
  test2               Run Test #2: overwrite a long text file with a shorter one
  test3               Run Test #3: copy an executable binary file
  test4               Run Test #4: use copied filecopy to make a copy of filecopy
  allTests            Run all tests with full verbose output
  allTestsSummary     Run all tests and show PASSED/FAILED summary only
</pre>

To run a specific test, pass its target to `make`. For example: `make test1`. For a summary of all test results, run `make allTestsSummary`.


<br>

### Grading Criteria

--- --- --- --- --- --- --- --- --- --- --- --- --- --- --- --- --- --- --- --- ---

This assignment is graded on a 100-point scale. To receive full credit, your program must compile successfully and copy files correctly. The breakdown is as follows:

 - **20 points** - Compiles with no warnings
 - **20 points** - Correct use of system calls
 - **20 points** - Successfully copies text files (Tests #1 and #2)
 - **20 points** - Successfully copies binary files (Tests #3 and #4)
 - **10 points** - Checks all necessary error conditions
 - **10 points** - Properly closes files when finished



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

This produces `assign01_submission.zip` in your project directory. Upload this file through the [Marmoset web interface](https://cs.ycp.edu/marmoset).
