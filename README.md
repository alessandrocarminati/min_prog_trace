# Introduction
This project demonstrates the use of the Linux ftrace subsystem to trace the
kernel functions involved in running a minimal C application. 
The objective is to capture all the kernel interactions of a simple user-space
program using ftrace.

# Project Components
The project consists of two main components:

`min.c`: A minimal C application that performs a basic operation and enters an 
infinite loop.
`ftrace_it.c`: An application that sets up ftrace to trace the execution of
another program.

# `min.c`
The `min.c` application is designed to do almost nothing, making it an ideal
candidate for tracing the fundamental kernel interactions. 
The source code for min.c is intentionally simple to avoid introducing 
unnecessary operations.

To compile `min.c` with minimal additional code, the following is used:

```
gcc -o min min.c --static -nostartfiles
```

This command compiles the application as a statically linked binary and omits
the standard startup files, ensuring that no additional code is included.

`ftrace_it.c`
The ftrace_it.c application sets up the ftrace subsystem to trace the execution
of a specified program, such as `min.c`. 
It performs the following tasks:

* Forks a child process.
* Configures ftrace to trace the child process.
* Starts the child process to execute the given program.
* Waits for the child process to complete.
* Disables ftrace.

Key Features:
* **Shared Memory** Synchronization: Uses a shared variable in memory to 
  synchronize the parent and child processes without impacting the trace.
* **ftrace Configuration**: Configures ftrace to trace the functions of 
  the child process, capturing all kernel interactions.

Usage:
Included Makefile provides two targets to compile the two artifacts.
After having them compiled run `ftrace_it`
```
sudo ./ftrace_it ./min
```

Note: sudo is required to access and modify ftrace settings.

# Elisa scenario
This repo, although it can run on any Linux system, is intended to be run on
a minimal aarch64 system within the ELISA community for which it was realized.
To recreate the environment where it is built, please use Buildroot and apply
the patch br_package.patch to add the repo to the available Buildroot packages.
Then, use the provided configurations `br.config` (which should be copied to 
`.config`) and `kernel.config` to build the system.

The resulting build output, located at `{br_dir}/output/images/Image`, needs 
to be used with aarch64 QEMU:
```
{br_dir} $ qemu-system-aarch64 -M virt -m 512M -cpu cortex-a72 -nographic -smp 2 -kernel output/images/Image
```
Inside the VM, use the following commands to recreate the log:
```
mount -t proc none /proc
mount -t tracefs none /sys/kernel/tracing/
sleep 20 && echo start && cat /sys/kernel/tracing/trace_pipe > /tmp/log &

/usr/bin/ftrace_it /usr/bin/min
or
/usr/bin/ftrace_it /usr/bin/min.large
```
# Summary
1. Download and extract Buildroot: [Buildroot 2024.02.4}(https://buildroot.org/downloads/buildroot-2024.02.4.tar.gz)
2. Apply the patch: `br_package.patch`
3. Use provided configurations:
    * Copy `br.config` to `.config`
    * Use `kernel.config` (place it in the BR root dir
4. Build the system.
5. Run the built image with QEMU:
```
qemu-system-aarch64 -M virt -m 512M -cpu cortex-a72 -nographic -smp 2 -kernel output/images/Image
```
5. Recreate the log inside the VM:
```
mount -t tracefs none /sys/kernel/tracing/
sleep 20 && echo start && cat /sys/kernel/tracing/trace_pipe > /tmp/log &

/usr/bin/ftrace_it /usr/bin/min
or
/usr/bin/ftrace_it /usr/bin/min.large
```

