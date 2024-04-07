+++
title = 'rCore Learning Ch0'
date = 2024-03-23T21:17:26+08:00
summary = "OS overview"
tags = ["Rust", "Programming", "OS"]
+++

# Chapters summary

## Ch.1

**Summary:** Solve application and hardware isolation through the operating system and simplify application programming.

- [ ] Design and implement an application that displays "Hello Word" running on `bare metal`.

**Target:** Formation of `LibOs`, a "Cambrian 'trilobite'" operating system running on bare metal, to achieve a comprehensive and in-depth understanding of the abstract concepts and concrete implementations of applications and the environments on which they depend.

# Ch.2

**Summary:** Ensure `system security` and `multi-application support`.

- [ ] Design application.
- [ ] `Batch processing mode`, supports automatic loading and running of multiple programs.
- [ ] Implement the `isolation of execution privileges` between applications and the operating system.

**Target:** Implement the "Devonian 'Deng Shiyu'" operating system that can run multiple applications - `BatchOS`, achieve the understanding and implementation of concepts such as `system calls`, `privilege levels`, `batch processing`, and understand how to improve the overall system performance through batch processing, protecting the operating system through privilege isolation and enabling system calls across privilege levels.

# Ch.3

**Summary:** Improve the overall performance of multi-program running and ensure the fairness of multiple program running.

- [ ] Load applications into memory in advance to reduce application switching overhead.
- [ ] Implement a `collaboration mechanism` between applications and support programs to proactively give up the processor.
- [ ] The preemption mechanism based on hardware interrupts, supports the program to passively give up the processor - ensuring the fairness of the program's use of processor resources and further improving the program's response efficiency to I/O time.

**Target:** Realize the "Permian 'Sawtooth'" operating system that supports multi-programming--`MultiprogOS`, the "Triassic 'Old Dragon'" operating system that supports collaboration mechanism--`CoopOS`, and the "Triassic 'Old Dragon'" operating system that supports time-sharing `multitasking` "Coelophysis" operating system--`TimesharingOS`. Extract core operating system concepts such as tasks and task switching, and have a deeper understanding of the hardware interrupt processing mechanism and the time-sharing sharing mechanism of the operating system.

# Ch.4

**Summary:** The issue of safe isolation and efficient use of memory. Allow multiple programs on one computer to obtain unlimited memory space, and isolate the memory space that programs can access to ensure `memory security` between different applications.

- [ ] Page tables and `TLB`(Translation Lookaside Buffer) mechanisms in hardware.
- [ ] The operating system builds page tables in the memory for itself and different applications - forming memory isolation between applications and applications and operating systems, thereby solving the problem of memory security isolation.
- [ ] `Page missing exception` and `dynamic page table modification` technology allow the data currently being accessed or about to be accessed by the currently running application to be located in the memory, and the infrequently used data is cached in the storage device, forming the ability to reuse memory in a time-sharing manner, that is, `virtual memory`.

**Target:** Implement the "Jurassic 'Ankylosaurus'" operating system that supports memory isolation - `AddressSpaceOS`. You can connect the abstract concept of address with the specific design of the page table, and master the implementation of the address space through the page table mechanism. Get an in-depth understanding of the `address space switching mechanism` added to `task switching`. Understand whether various page replacement strategies in the virtual memory mechanism can be effectively implemented and how to implement them specifically.

# Ch.5

**Summary:** Improve the flexibility and interactivity of dynamic execution of applications, allowing developers to control the creation, running and exit of programs in a timely manner.

- [ ] Give the user a `shell` and form a `CLI` for the user to interact with the operating system. Can start or kill applications, or monitor system health.
- [ ] Reconstruct the operating system to allow the operating system to support services such as dynamic creation/destruction/wait/pause of programs.
- [ ] Further abstraction based on the existing task abstraction - process, used to represent and manage the entire execution process of the program.

**Target:** Implement the "Cretaceous Troodon" operating system - `ProcessOS` with flexible and powerful process management functions. Establish a connection between abstract concepts such as process, `process scheduling`, `process switching`, `process status`, and `process life cycle`, and the specific design of the `process control block` data structure, and process-related `system call functions` in the operating system.

# Ch.6

**Summary:** Allow programs to easily access data on the storage device. Design `files` and `file systems`.

- [ ] Design and implement `easyfs`.
- [ ] Provides programs with two abstractions: `regular files` and `directories`.
- [ ] Provide open, close, read, write system calls to read and write data in files.
- [ ] Manage storage device peripherals through the storage device driver.

**Target:** Implement the "T-Rex" operating system that supports file access--`FilesystemOS`. You can learn how the operating system abstraction of files and file systems is embodied by a specific file `system-easyfs`, and learn the close connection between the file system, `process management`, and `memory management`, thereby supporting programs to conveniently access files on storage devices data.

# Ch.7

**Summary:** Allow different applications to share and cooperate with data, and introduce the concepts of `pipe` and `signal` to support the I/O redirection function between processes.

- [ ] The pipeline is regarded as a special memory file and can be implemented based on file operations.
- [ ] The signal event notification mechanism allows the process to promptly *obtain and process notifications from other processes or the operating system*.

**Target:** Forming a "Cretaceous Velociraptor" operating system - `IPCOS` that supports `data interaction` and `event notification` functions between multiple APP processes. You can learn that isolation and sharing between processes can be achieved at the same time, and understand how to use pipes to break the address space isolation between processes to achieve `data sharing`; and how to interrupt the normal execution of processes through the signal mechanism to respond to relatively urgent events in a timely manner. Thereby mastering multi-application sharing and collaboration.

# Ch.8

**Summary:** Improve the efficiency of concurrent execution of multiple applications and ensure that multiple applications correctly access shared resources. Because the address space isolation of the process will bring management `runtime overhead`, such as `TLB refresh`, page table switching, etc. If multiple tasks that can be executed in parallel within a process are scheduled by the operating system in a more fine-grained manner, concurrent execution can be achieved within the process, and because these tasks are in the address space within the process, they will **NOT** include runtime overhead such as page table switching, the task here is `thread`. The `shared address` space between threads of a process makes it more convenient for them to access shared resources, but if not handled properly, resource access conflicts and competition problems may occur.

- [ ] Realize thread.
- [ ] Implement a synchronization mechanism that coordinates the execution sequence of processes or threads.
- [ ] The mutual exclusion mechanism ensures that only one process or thread can access shared resources at the same time.

**Target:** Reconstruct on the basis of process management, design and implement a thread management mechanism, and implement a "Dakota Raptor" operating system that supports multi-threading - `ThreadOS`; And further design a `lock mechanism` and `signals` that support thread synchronization and mutual exclusion of access to shared resources, along with the `semaphore mechanism` and condition variable mechanism, ultimately form the "Cretaceous 'Mother Dragon'" operating system - `SyncMutexOS` that supports multi-threaded apps to synchronize and mutually exclude access to shared resources. Can understand the relationship and difference between threads and processes, and have an in-depth understanding of the principles and implementation of the synchronization mutual exclusion mechanism that supports concurrent access to shared resources.

# Ch.9

**Summary:** Allow applications to easily access I/O devices and give applications more awareness and interaction capabilities.

- [ ] Analyze the development history of peripherals and peripheral data transmission methods.
- [ ] The operating system establishes different levels of abstraction and different I/O execution models for peripherals to facilitate the internal management of peripherals by the operating system and efficient and convenient access to peripherals by applications.
- [ ] Analyze the operating system and analyze peripheral information in the computer through the device tree.
- [ ] Redesign the interrupt-based serial port driver - serial port device initialization and serial port data input and output.
- [ ] Improve the process/thread scheduling mechanism to allow processes/threads waiting for serial port input or output to be completed to enter a blocking state.
- [ ] Learn the virtio device architecture simulated by QEMU and the main functions of the virtio device driver.
- [ ] Conduct a more in-depth analysis of the virtio-blk device and its driver, virtio-gpu device and its driver

**Target:** Forming a "Jurassic 'Jurassic Hunter'" operating system - `DeviceOS` that supports graphics game apps and has efficient peripheral interrupt response. You can have an in-depth understanding of the characteristics of different peripherals, the I/O transmission methods of peripherals, different levels of peripheral abstraction concepts and I/O execution models, so as to have a relatively complete understanding of how the operating system effectively manages different types of peripherals.

Gradually clarify the needs and problems to be solved in each chapter, gradually understand the composition of the kernel module in the operating system in each chapter, and master the functions of the kernel module and the relationship between different kernel modules. Summarize operating system design ideas, strategies and mechanisms, principles and concepts. Finally reach the level of understanding the operating system.
