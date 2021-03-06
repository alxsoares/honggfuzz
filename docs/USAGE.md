# INTRODUCTION #

  This document describes the **honggfuzz** project.

# OBJECTIVE #

  Honggfuzz is a general-purpose fuzzing tool. Given an input corpus files, honggfuzz modifies input to a test program and utilize the **ptrace() API**/**POSIX signal interface** to detect and log crashes. It can also use software or hardware-based code coverage techniques to produce more and more interesting inputs

# FEATURES #

  * It's __multi-threaded__ and __multi-process__: no need to run multiple copies of your fuzzer. The file corpus is shared between threads (and fuzzed instances)
  * It's blazingly fast (esp. in the [persistent fuzzing mode](https://github.com/google/honggfuzz/blob/master/docs/PersistentFuzzing.md)). A simple _LLVMFuzzerTestOneInput_ function can be tested with __up to 1mo iterations per second__ on a relatively modern CPU (e.g. i7-6600K)
  * Has a nice track record of uncovered security bugs: e.g. the only (to the date) __vulnerability in OpenSSL with the [critical](https://www.openssl.org/news/secadv/20160926.txt) score mark__ was discovered by honggfuzz
  * Uses low-level interfaces to monitor processes (e.g. _ptrace_ under Linux). As opposed to other fuzzers, it __will discover and report hidden signals__ (caught and potentially hidden by signal handlers)
  * Easy-to-use, feed it a simple input corpus (__can even consist of a single, 1-byte file__) and it will work its way up expanding it utilizing feedback-based coverage metrics
  * Supports several (more than any other coverage-based feedback-driven fuzzer) hardware-based (CPU: branch/instruction counting, __Intel BTS__, __Intel PT__) and software-based [feedback-driven fuzzing](https://github.com/google/honggfuzz/blob/master/docs/FeedbackDrivenFuzzing.md) methods
  * Works (at least) under GNU/Linux, FreeBSD, Mac OS X, Windows/CygWin and [Android](https://github.com/google/honggfuzz/blob/master/docs/Android.md)
  * Supports __persistent fuzzing mode__ (long-lived process calling a fuzzed API repeatedly) with libhfuzz/libhfuzz.a. More on that can be found [here](https://github.com/google/honggfuzz/blob/master/docs/PersistentFuzzing.md)
  * [Can fuzz remote/standalone long-lasting processes](https://github.com/google/honggfuzz/blob/master/docs/AttachingToPid.md) (e.g. network servers like __Apache's httpd__ and __ISC's bind__)
  * It comes with the __[examples](https://github.com/google/honggfuzz/tree/master/examples/openssl) directory__, consisting of real world fuzz setups for widely-used software (e.g. Apache and OpenSSL)

# REQUIREMENTS #

  * A POSIX compliant operating system, [Android](https://github.com/google/honggfuzz/blob/master/docs/Android.md) or Windows (CygWin)
  * GNU/Linux with modern kernel (>= v4.2) for hardware-based code coverage guided fuzzing

  * A corpus of input files. Honggfuzz expects a set of files to use and modify as input to the application you're fuzzing. How you get or create these files is up to you, but you might be interested in the following sources:
    * Image formats: Tavis Ormandy's [Image Testuite](http://code.google.com/p/imagetestsuite/) has been effective at finding vulnerabilities in various graphics libraries.
    * PDF: Adobe provides some [test PDF files](http://acroeng.adobe.com/).

_**Note**: With the feedback-driven coverage-based modes, you can start your fuzzing with even a single 1-byte file._

## Compatibility list ##

It should work under the following operating systems:

| **OS** | **Status** | **Notes** |
|:-------|:-----------|:----------|
| **GNU/Linux** | Works | ptrace() API (x86, x86-64 disassembly support)|
| **FreeBSD** | Works | POSIX signal interface |
| **Mac OS X** | Works | POSIX signal interface/Mac OS X crash reports (x86-64/x86 disassembly support) |
| **Android** | Works | ptrace() API (x86, x86-64 disassembly support) |
| **MS Windows** | Works | POSIX signal interface via CygWin |
| **Other Unices** | Depends`*` | POSIX signal interface |

 _`*`) It might work provided that a given operating system implements **wait4()** call_

# USAGE #

<pre>
Usage: ./honggfuzz [options] -- path_to_command [args]
Options:
 --help|-h 
	Help plz..
 --input|-f VALUE
	Path to a directory containing initial file corpus
 --persistent|-P 
	Enable persistent fuzzing (use hfuzz_cc/hfuzz-clang to compile code)
 --instrument|-z 
	Enable compile-time instrumentation (use hfuzz_cc/hfuzz-clang to compile code)
 --sancov|-C 
	Enable sanitizer coverage feedback
 --keep_output|-Q 
	Don't close children's stdin, stdout, stderr; can be noisy
 --timeout|-t VALUE
	Timeout in seconds (default: '10')
 --threads|-n VALUE
	Number of concurrent fuzzing threads (default: number of CPUs / 2)
 --stdin_input|-s 
	Provide fuzzing input on STDIN, instead of ___FILE___
 --mutation_rate|-r VALUE
	Maximal mutation rate in relation to the file size, (default: '0.001')
 --logfile|-l VALUE
	Log file
 --verbose|-v 
	Disable ANSI console; use simple log output
 --verifier|-V 
	Enable crashes verifier
 --debug_level|-d VALUE
	Debug level (0 - FATAL ... 4 - DEBUG), (default: '3' [INFO])
 --extension|-e VALUE
	Input file extension (e.g. 'swf'), (default: 'fuzz')
 --workspace|-W VALUE
	Workspace directory to save crashes & runtime files (default: '.')
 --covdir VALUE
	New coverage is written to a separate directory (default: use the input directory)
 --wordlist|-w VALUE
	Wordlist file (tokens delimited by NUL-bytes)
 --stackhash_bl|-B VALUE
	Stackhashes blacklist file (one entry per line)
 --mutate_cmd|-c VALUE
	External command producing fuzz files (instead of internal mutators)
 --pprocess_cmd VALUE
	External command postprocessing files produced by internal mutators
 --iterations|-N VALUE
	Number of fuzzing iterations (default: '0' [no limit])
 --rlimit_as VALUE
	Per process memory limit in MiB (default: '0' [no limit])
 --report|-R VALUE
	Write report to this file (default: 'HONGGFUZZ.REPORT.TXT')
 --max_file_size|-F VALUE
	Maximal size of files processed by the fuzzer in bytes (default: '1048576')
 --clear_env 
	Clear all environment variables before executing the binary
 --env|-E VALUE
	Pass this environment variable, can be used multiple times
 --save_all|-u 
	Save all test-cases (not only the unique ones) by appending the current time-stamp to the filenames
 --msan_report_umrs 
	Report MSAN's UMRS (uninitialized memory access)
 --tmout_sigvtalrm|-T 
	Use SIGVTALRM to kill timeouting processes (default: use SIGKILL)
 --sanitizers|-S 
	Enable sanitizers settings (default: false)
 --monitor_sigabrt VALUE
	Monitor SIGABRT (default: 'false for Android - 'true for other platforms)
 --no_fb_timeout VALUE
	Skip feedback if the process has timeouted (default: 'false')
 --linux_symbols_bl VALUE
	Symbols blacklist filter file (one entry per line)
 --linux_symbols_wl VALUE
	Symbols whitelist filter file (one entry per line)
 --linux_pid|-p VALUE
	Attach to a pid (and its thread group)
 --linux_file_pid VALUE
	Attach to pid (and its thread group) read from file
 --linux_addr_low_limit VALUE
	Address limit (from si.si_addr) below which crashes are not reported, (default: '0')
 --linux_keep_aslr 
	Don't disable ASLR randomization, might be useful with MSAN
 --linux_perf_ignore_above VALUE
	Ignore perf events which report IPs above this address
 --linux_perf_instr 
	Use PERF_COUNT_HW_INSTRUCTIONS perf
 --linux_perf_branch 
	Use PERF_COUNT_HW_BRANCH_INSTRUCTIONS perf
 --linux_perf_bts_edge 
	Use Intel BTS to count unique edges
 --linux_perf_ipt_block 
	Use Intel Processor Trace to count unique blocks (requires libipt.so)
 --linux_perf_kernel_only 
	Gather kernel-only coverage with Intel PT and with Intel BTS
 --linux_ns_net 
	Use Linux NET namespace isolation
 --linux_ns_pid 
	Use Linux PID namespace isolation
 --linux_ns_ipc 
	Use Linux IPC namespace isolation

Examples:
 Run the binary over a mutated file chosen from the directory
  honggfuzz -f input_dir -- /usr/bin/tiffinfo -D ___FILE___
 As above, provide input over STDIN:
  honggfuzz -f input_dir -s -- /usr/bin/djpeg
 Use SANCOV to maximize code coverage:
  honggfuzz -f input_dir -C -- /usr/bin/tiffinfo -D ___FILE___
 Use compile-time instrumentation (libhfuzz/instrument.c):
  honggfuzz -f input_dir -z -- /usr/bin/tiffinfo -D ___FILE___
 Use persistent mode (libhfuzz/persistent.c):
  honggfuzz -f input_dir -P -- /usr/bin/tiffinfo_persistent
 Use persistent mode (libhfuzz/persistent.c) and compile-time instrumentation (libhfuzz/instrument.c):
  honggfuzz -f input_dir -P -z -- /usr/bin/tiffinfo_persistent
 Run the binary over a dynamic file, maximize total no. of instructions:
  honggfuzz --linux_perf_instr -- /usr/bin/tiffinfo -D ___FILE___
 Run the binary over a dynamic file, maximize total no. of branches:
  honggfuzz --linux_perf_branch -- /usr/bin/tiffinfo -D ___FILE___
 Run the binary over a dynamic file, maximize unique branches (edges) via BTS:
  honggfuzz --linux_perf_bts_edge -- /usr/bin/tiffinfo -D ___FILE___
 Run the binary over a dynamic file, maximize unique code blocks via Intel Processor Trace (requires libipt.so):
  honggfuzz --linux_perf_ipt_block -- /usr/bin/tiffinfo -D ___FILE___
</pre>

# OUTPUT FILES #

| **Mode** | **Output file** |
|:---------|:----------------|
| Linux | **SIGSEGV.PC.4ba1ae.STACK.13599d485.CODE.1.ADDR.0x10.INSTR.mov____0x10(%rbx),%rax.fuzz** |
| POSIX signal interface | **SIGSEGV.22758.2010-07-01.17.24.41.tif** |

## Description ##

  * **SIGSEGV**,**SIGILL**,**SIGBUS**,**SIGABRT**,**SIGFPE** - Description of the signal which terminated the process (when using ptrace() API, it's a signal which was delivered to the process, even if silently discarded)
  * **PC.0x8056ad7** - Program Counter (PC) value (ptrace() API only), for x86 it's a value of the EIP register (RIP for x86-64)
  * **STACK.13599d485** - Stack signature (based on stack-tracing)
  * **ADDR.0x30333037** - Value of the _siginfo`_`t.si`_`addr_ (see _man 2 signaction_ for more details) (most likely meaningless for SIGABRT)
  * **INSTR.mov____0x10(%rbx),%rax`** - Disassembled instruction which was found under the last known PC (Program Counter) (x86, x86-64 architectures only, meaningless for SIGABRT)

# FAQ #

  * Q: **Why the name _honggfuzz_**?
  * A: The term honggfuzz was coined during a major and memorable event in the city of [Zurich](http://en.wikipedia.org/wiki/H%C3%B6ngg), where a Welsh security celebrity tried to reach Höngg in a cab while singing _Another one bites the dust_.

  * Q: **Why do you prefer the ptrace() API to the POSIX signal interface**?
  * A: The ptrace() API is more flexible when it comes to analyzing a process' crash. wait3/4() syscalls are only able to determine the type of signal which crashed an application and limited resource usage information (see _man wait4_).

  * Q: **Why isn't there any support for the ptrace() API when compiling under FreeBSD or Mac OS X operating systems**?
  * A: These operating systems lack some specific ptrace() operations, including **PT`_`GETREGS** (Mac OS X) and **PT`_`GETSIGINFO**, both of which honggfuzz depends on. If you have any ideas on how to get around this limitation, send us an email or patch.

# LICENSE #

 This project is licensed under the [Apache License, Version 2.0](http://www.apache.org/licenses/LICENSE-2.0)

# CREDITS #

  * Thanks to **[taviso@google.com Tavis Ormandy]** for many valuable ideas used in the course of this project's design and implementation phases
  * Thanks to my 1337 friends for all sorts of support and distraction :) - **LiquidK, lcamtuf, novocainated, asiraP, ScaryBeasts, redpig, jln**
