# GDB (GNU Debugger)

## Overview

GDB (GNU Debugger) is the most powerful debugging tool for C and C++ applications. You can use it to:
✅ Step through code line by line
✅ Set breakpoints and watch variables
✅ Analyze crashes and core dumps
✅ Debug multi-threaded programs
✅ Modify variables at runtime

- [Steps to perform while debugging the code](#Steps-to-perform-while-debugging-the-code)
- [Advanced GDB Features](#Advanced-GDB-Features)
- [Uses of TUI](#Uses-of-TUI)
- [Using GDB for large applications for Seg faults/termination](#Using-GDB-for-large-applications-for-Seg-faults-termination)
- [How to Detect Deadlocks Using info threads in GDB](#How-to-Detect-Deadlocks-Using-info-threads-in-GDB)
- [What is Paging in GDB?](#What-is-Paging-in-GDB)
- [How to Control Paging in GDB](#How-to-Control-Paging-in-GDB)
- [Setting up the coredump](#Setting-up-the-coredump)
- [Summary of Important GDB Commands](#Summary-of-Important-GDB-Commands)
- [Resources and roadmap](#Resources-and-roadmap)


## Steps to perform while debugging the code

`Step 1:` Write a Sample C++ Program (test.cpp)
```c++
#include <iostream>

void buggyFunction(int x) {
    int y = 5 / x;  // Division by zero issue!
    std::cout << "y = " << y << std::endl;
}

int main() {
    int num;
    std::cout << "Enter a number: ";
    std::cin >> num;
    buggyFunction(num);
    return 0;
}
```
`Step 2:` Compile with Debug Symbols (-g flag)
```bash
`g++ -g test.cpp -o test`
🔹 The -g flag includes debugging symbols so GDB can analyze the program.
```
`Step 3:` Start GDB
```bash
`gdb ./test`
Now you will enter the GDB interactive shell.
```
`Step 4:` Set Breakpoints
```
To stop execution at buggyFunction(), type:
`break buggyFunction`
✅ GDB will pause execution before entering this function.
```

`Step 5:` Run the Program
```
`run`
🔹 GDB executes the program and waits for user input. If you enter 0, the program will crash due to division by zero.
```

`Step 6:` Step Through Code
```
To go line by line, use:
`next`   # Move to the next line (does not go inside functions)
`step`   # Move into function calls (useful for debugging deeply)

📌 If you want to check values:
`print x`  # Print the value of x
`print y`  # Print the value of y
```

`Step 7:` Inspect Variables
```
You can check variable values:
`info locals`   # Show all local variables
`info args`     # Shows all input args
`print x`       # Print a specific variable
`watch x`       # Watch x for any changes
```
`Step 8:` Backtrace (Find Crash Point)
```
If the program crashes, type:
`backtrace` or `bt`
✅ It will show the call stack, which tells you where the error happened.

Example Output:
#0  buggyFunction (x=0) at test.cpp:5
#1  main () at test.cpp:12
🔹 You can see that buggyFunction() crashed at line 5 in test.cpp.
```

`Step 9:` Modify Variable at Runtime
```
Sometimes, instead of restarting the program, you can change values on the fly:
`set variable x = 1`
`continue`   # Resume execution
✅ This allows you to bypass division by zero without modifying the source code.
```
`Step 10:` Exit GDB
```
To quit:
`quit`
```

## Advanced GDB Features
✅ Debugging Core Dumps
ulimit -c unlimited  # Enable core dumps
./test               # Run program (if it crashes, core file is generated)
gdb ./test core      # Load core dump in GDB
backtrace            # Analyze crash point

✅ Debugging Multi-Threaded Programs
info threads    # Show all threads
thread 2        # Switch to thread 2

✅ Debugging Shared Libraries
info sharedlibrary  # Show loaded shared libraries


✅ GDB TUI Mode (Graphical Debugging)
gdb -tui ./test
🔹 Opens a split-screen UI where you see code + execution flow.


## Uses of TUI
```
-Compile the application ->  `g++ -g Demo.cpp`
-Run the app using gdb ->    `gdb ./a.out`
-If dont want to add any breakpoint, hit the command -> `start`
-After that if you enter the command `list` it will show the function in which you are currently present.
-The output of `list` will be in TUI i.e. text user interface mode from (1970).
-Now if we hit `ctrl + x + a` the TUI will show some different view which is more like a code editor.
-`ctrl + x + 2` shows the assebly program. 
-If one more time `ctrl + x + 2` is pressed, it will show the registers.
-`tui reg float` will show the floating registers.
-`ctrl + x + 1` will go one window back.
-When the TUI is on, the up & down arrow keys will move the code, but we want to take the previous or next commands,
    we have to use `ctrl + p` for previous, `ctrl + n` for next command on the terminal.
```


## Using GDB for large applications for Seg faults termination
GDB is one of the best tools to debug your large, multi-threaded C++ application. Since manually setting breakpoints everywhere isn't feasible, you need a systematic approach for debugging. Here's how you can do it effectively:

🔹 `Step 1:` Compile with Debug Symbols
```
First, ensure your CMake build includes debugging symbols (-g flag). Modify your CMakeLists.txt:
`set(CMAKE_BUILD_TYPE Debug)`
`set(CMAKE_CXX_FLAGS_DEBUG "-g -O0 -pthread")`
Then rebuild: `cmake --build . --config Debug`
🔹 This ensures GDB can provide detailed debugging information.
```
🔹 `Step 2:` Run the Application Inside GDB
```
Instead of running your program directly: `./my_app`
Run it inside GDB: `gdb --args ./my_app arg1 arg2`
Then, start execution: `run`
🔹 If the program crashes, GDB will pause at the exact failure point.
```
🔹 `Step 3:` Find the Crash Point (Backtrace)
```
When the application crashes, type:
`backtrace`
This will show where the crash happened, including function calls:
#0  some_function() at file.cpp:123
#1  main() at main.cpp:45
🔹 This tells you which file and line caused the issue.
```
🔹 `Step 4:` Debugging Segmentation Faults & std::bad_function_call
```
Enable Core Dumps (if the app crashes without GDB)
`ulimit -c unlimited`
`./my_app`
If it crashes, a core dump file (core.*) is generated.
Analyze the Core Dump with GDB
`gdb ./my_app core`
Then run:
`backtrace`
🔹 This tells you where it crashed, even if GDB wasn't running.
```
🔹 `Step 5:` Set Conditional Breakpoints (Only When Necessary)
```
Since you have too many files, instead of setting breakpoints everywhere, set a conditional breakpoint only when a variable is incorrect:
`break some_file.cpp:123 if my_var == nullptr`
This will stop execution only if the condition is met.
```
🔹 `Step 6:` Debugging Multi-Threaded Code
```
Check All Running Threads
`info threads`

Example Output:
1. Thread 1 (main)  main() at main.cpp:45
2. Thread 2        worker_thread() at worker.cpp:78

Switch to a Specific Thread
`thread 2`

Now, check where it is stuck.

Set Breakpoints in a Specific Thread
`break worker.cpp:78 thread 2`

Check What a Thread is Doing
`info registers`
```
🔹 `Step 7:` Debug Termination Without Error Message
```
If your app just exits suddenly without errors, do this:
`catch throw`
This will stop the execution when an exception is thrown, even if it’s caught later.

To catch program termination:
`catch syscall exit`
This helps find where the unexpected exit() is happening.
```

🔹 `Step 8:` Log All Debug Info to a File
```
Since debugging large apps can be hard, log everything:
set logging on
run
backtrace
info threads
🔹 This saves everything into gdb.txt for later analysis.
```

## How to Detect Deadlocks Using info threads in GDB
When you run: `info threads`
You will see output like this:
```
  Id   Target Id         Frame
  1    Thread 0x7f1d5c64d700 (LWP 1234) "my_app" pthread_mutex_lock () at mutex.c:80
  2    Thread 0x7f1d5c64c700 (LWP 1235) "my_app" pthread_mutex_lock () at mutex.c:80
  3    Thread 0x7f1d5c64b700 (LWP 1236) "my_app" pthread_cond_wait () at condvar.c:110
  4    Thread 0x7f1d5c64a700 (LWP 1237) "my_app" __lll_lock_wait () at lowlevellock.c:50
```
🔹 What This Means?
Thread 1 & 2 are both stuck at pthread_mutex_lock(), meaning they are waiting for a lock.
Thread 3 is in pthread_cond_wait(), meaning it’s waiting for a condition variable to be signaled.
Thread 4 is in __lll_lock_wait(), meaning it's waiting for a low-level lock (often caused by a deadlock).
This suggests a possible deadlock because:
Multiple threads are stuck at pthread_mutex_lock(), which means they could be waiting on each other.
A thread waiting on pthread_cond_wait() may never wake up if the signaling thread is deadlocked.

🔹 `Step 1:` Identify the Stuck Threads
After running info threads, find the threads stuck on a lock (typically pthread_mutex_lock or __lll_lock_wait).
Switch to one of the stuck threads:
```
thread 1
backtrace
```
Example output:
#0  pthread_mutex_lock () at mutex.c:80
#1  MyClass::someFunction() at myfile.cpp:120
🔹 This means Thread 1 is waiting for a mutex in someFunction() at myfile.cpp:120.

Switch to another stuck thread:
```bash
thread 2
backtrace
```
If this thread is also stuck at the same mutex, it's likely a deadlock.

🔹 `Step 2:` Check Which Mutex is Locked
Once you identify a thread stuck in pthread_mutex_lock(), inspect its arguments:
```
frame 0
info registers
```
If your program uses named mutexes, check their values:
```
print my_mutex
```
If using `std::mutex`, it might not be directly visible in GDB, but you can enable libpthread debugging:
```
set debug libpthread 1
```

🔹 `Step 3:` Find Who Owns the Mutex
To see which thread is holding a lock:
```
info threads
thread apply all backtrace
```
Look for a thread that is inside a critical section but hasn’t exited.

🔹 `Step 4:` Force Unlock a Mutex (If Needed)
If you just want to break the deadlock, forcefully unlock the mutex:
```
set variable my_mutex.__data.__lock = 0
continue
```
🚨 Caution: This is a hack and should be used only for debugging.

🔹 `Alternative:` Use GDB Python Script for Deadlock Detection
```
You can automate deadlock detection with a Python script inside GDB:
define check_deadlock
  info threads
  thread apply all backtrace
end
Run it inside GDB:
source check_deadlock.gdb
check_deadlock
```
This will print all threads + stack traces, helping you spot deadlocks quickly.


## What is Paging in GDB
Paging in GDB refers to how GDB displays large outputs by pausing after a screenful of text. 
This behavior is similar to how the less or more commands work in Linux.

For example, when running commands like:
```
info threads
backtrace
disassemble
```
GDB might display a lot of information. Instead of dumping everything at once, 
it pauses after filling the screen and waits for user input.

## How to Control Paging in GDB
1️⃣ `Disable Paging` (Show Everything at Once)
If you don’t want GDB to pause, disable paging:
`set pagination off`
Now, GDB will print everything without stopping.
To re-enable:
`set pagination on`
✅ Use Case: Useful when logging output to a file.

2️⃣ `Navigate Through Paged Output`
If paging is enabled, GDB shows --Type <return> to continue, or q to quit-- at the bottom.
```
Key	        Action
Enter	    Scroll one line down
Space	    Scroll one page down
q	        Quit and stop output
/keyword	Search for keyword in output
```
✅ Use Case: Useful when inspecting large outputs like backtrace full.

3️⃣ `Automatically Disable Paging for a Single Command`
If you don’t want to disable paging globally, but just for one command, use:
`set height 0`
This forces GDB to display all lines without pausing.

To restore normal behavior:
`set height 25`  # Default value
🔹 `Example:` Debugging a Large Stack Trace
If you have a deep call stack and want to see all frames without paging, run:
```
set pagination off
backtrace full
Or:
set height 0
backtrace
```


## Setting up the coredump
```
-The path to store the coredump is mentioned in the (/proc/sys/kernel/core_pattern)
-Initially it has the data like (/usr/share/apport/apport %p %s %c %P")
-This denotes that the coredump is being handled by the 'apport'
-But we want to use it for our gdb, so we have to rewrite this wile with the data as 'core' by using
 'echo core | sudo tee /proc/sys/kernel/core_pattern'
-Now the coredump file will be stored in the current dir.
```

=>`below are the steps to analyse large applications for crash point`
```
1. 'ulimit -c unlimited'  # Enable core dumps
2. './test'               # Run program (if it crashes, core file is generated)
3. 'gdb ./test core'      # Load core dump in GDB
4. 'backtrace'            # Analyze crash point
```
=> `What Does ulimit -c unlimited Do?`
```
ulimit is a shell command that controls various system resource limits for processes started by that shell.
-c is the option that deals with core dump size.
unlimited means there is no limit on the size of the core dump.
```

=>📌 `What is a Core Dump?`
```
A core dump (or core file) is a file created by the operating system when a running program crashes or terminates abnormally. 
It contains a snapshot of the program's memory at the time of the crash, including:
The contents of memory (stack, heap, etc.)
Registers (values of CPU registers)
The program's execution state
Debug information (if compiled with -g)
Core dumps are useful for debugging because they allow developers to examine the exact state of a program at the time of the crash.
```

📌 `Why Set ulimit -c unlimited?`
```
By default, many systems have a limit on the size of core dumps. If the limit is too small (e.g., 0), 
core dumps will not be created, even if a program crashes. 
This can make it difficult to debug crashes because no information is generated about what caused the crash.

By setting the core dump size to unlimited, you ensure that:
Core dumps are generated even if the crash produces a large memory dump.
You have the full memory state of the crashed program to help identify and fix the issue.
You can use tools like GDB (GNU Debugger) to analyze the core dump and track down the cause of the crash.
```

📌 `What is -O0 in g++?`
The -O0 flag in g++ (GCC compiler) disables optimizations during compilation. 
It ensures that the compiled program closely resembles the original source code, making debugging easier.

🔹 `Why Use -O0 for Debugging?`
When you compile with -O0, the compiler preserves the structure of your code as much as possible. 
This helps GDB provide accurate debugging information.

=> `With -O0:`
Variables are not optimized away.
Function calls are not inlined (so breakpoints work properly).
The execution flow remains close to the source code.

=> `Without -O0` (Optimized Code, e.g., -O2 or -O3):
The compiler may reorder instructions.
Some variables may be removed or stored in registers (making them invisible in GDB).
Some functions may be inlined, making breakpoints difficult.

🔹 `Common Optimization Levels in g++`
```
Flag	Optimization                    Level	Description
-O0	    No optimization	                Best for debugging (preserves code structure).
-O1	    Basic optimizations	S           light performance improvement, minimal debugging impact.
-O2	    More aggressive optimizations	Faster code, harder debugging (some variables removed).
-O3	    Maximum optimizations	        Highest speed, may change loops and inlining.
-Os	    Optimize for size	            Reduces binary size (similar to -O2, but avoids code bloat).
```

🔹 `How to Compile for Debugging?`
To compile your C++ program with debugging symbols and no optimizations, use:
`g++ -g -O0 my_code.cpp -o my_program`
✅ -g: Adds debug symbols (required for GDB).
✅ -O0: Disables optimizations (preserves debugging info).


## Summary of Important GDB Commands
```
Command	            Description
gdb ./a.out	        Start GDB with a program
break <func>	    Set a breakpoint at function
run	                Run the program
next	            Execute next line (without entering functions)
step	            Execute next line (step into functions)
continue	        Continue execution after a breakpoint
backtrace	        Show function call stack
info locals	        Show local variables
print x	            Print variable x
set variable x=10	Modify variable x at runtime
watch x	            Monitor variable x for changes
info threads	    Show all threads
thread 2	        Switch to thread 2
quit	            Exit GDB
run	                                Start program execution
backtrace	                        Show crash location
info threads	                    List all threads
thread 2	                        Switch to thread 2
break file.cpp:123	                Set a breakpoint
break file.cpp:123 if x == nullptr	Set a conditional breakpoint
watch x	                            Stop when x changes
catch throw	                        Stop when an exception is thrown
catch syscall exit	                Stop when program exits 
info threads	                    Show all threads (check for stuck ones)
thread <id>	                        Switch to a specific thread
backtrace	                        Show why a thread is stuck
print my_mutex	                    See if a mutex is locked
thread apply all backtrace	        Find which thread owns the lock
set variable my_mutex.__data.__lock = 0	Force unlock a mutex (last resort)
set pagination off	                Disable paging (show everything at once)
set pagination on	                Enable paging (default)
set height 0	                    Show full output without stopping
set height 25	                    Restore default behavior
q	                                Quit from paged output
Space	                            Scroll one page down
```



## Resources and roadmap
```
Best Structured Resources to Learn GDB
Here are some excellent places to get in-depth knowledge:

📖 Books & Official Docs:
GDB User Manual (Official)
The most detailed reference for GDB commands and internals.
"The Art of Debugging with GDB and DDD" by Norman Matloff
Covers GDB usage with practical debugging strategies.
"Debugging with GDB" by Richard M. Stallman
Free PDF available here.
🎥 Best Video Tutorials
Udacity's "Linux Debugging & Performance" (Free Course)
🔗 https://www.udacity.com/course/linux-debugging-and-performance--ud421
GDB Crash Course on YouTube
🔗 https://www.youtube.com/watch?v=PorfLSr3DDI
🛠 Hands-On GDB Practice
Online GDB Playground (Test commands without installing)
🔗 https://www.onlinegdb.com/
MIT’s Practical Debugging Guide
🔗 https://missing.csail.mit.edu/2020/debugging/
🔹 How to Use GDB Practically? (Step-by-Step)
Let's debug a simple C++ program using GDB.
```
