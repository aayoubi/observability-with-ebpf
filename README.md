# eBPF Playground

I have recently started playing with observability tools using eBPF along with bcc, bpftrace, and gobpf.

eBPF is an enhancement of the decade-old [Berkeley Packet Filter](http://www.tcpdump.org/papers/bpf-usenix93.pdf) created to capture and process network packets. Those enhancements brought forth a new type of software, with an enhancement virtual machine and instruction set that runs in the Linux Kernel, and that allows one to build observability, security, and networking tools. eBPF programs are mere "attachments" to specific code paths in the Kernel, meaning when a specific code path is executed by the Kernel, the eBPF program is called:

* one can attach eBPF to a network socket to filter traffic, compute statistics, classify traffic, re-route packets, ...
* one can attach eBPF to tracepoints, kprobes, or uprobes for debugging purposes
* one can restrict access to certain system calls for specific processes using eBPF
* and many more

We can write eBPF programs using different tools and programming languages (as of Q2-2021):

* using eBPF `assembly` code, using the Kernel's `bpf_asm` assembler. That's not an option anyone should consider.
* using `C` — a restricted version of C, with support of LLVM Clang's compiler and its eBPF backend that compiles C into bytecode, and then that bytecode gets loaded in the Kernel using the `bpf` system call. However, to compile this program one requires the Kernel source code.
* using `BCC` — a toolchain created to help developers create eBPF programs without depending on the Kernel source code for compilation. BCC makes BPF programs easier to write, with kernel instrumentation in C (and includes a C wrapper around LLVM), and front-ends in Python and lua. It is suited for many tasks, including performance analysis and network traffic control.
* using `bpftrace` — a high-level tracing language that relies on eBPF in its implementation. `bpftrace` is what `DTrace` was to Solaris systems. `bpftrace` can run programs directly on the command-line that trace, aggregate, and output data to the end-user.
* using `libbpf-bootstrap` and **BPF CO-RE** (**c**ompile-**o**nce **r**un-**e**verywhere). Given that our eBPF programs depend heavily on the underlying Kernel version, a lot of work is put into making sure your eBPF programs are always inline with the Linux Kernel (headers, instructions, offsets, ...). **BPF CO-RE** and `libbpf-bootstrap` is meant to help eBPF developers solve this problem. I haven't touched this bit yet. Read more [here](https://nakryiko.com/posts/bpf-portability-and-co-re/)

eBPF programs run safely inside the Linux Kernel thanks to the **eBPF in-kernel verifier**. The verifier performs a series of checks on the eBPF program  and make sure that it doesn't any break constraints before loading it in the Kernel, such as:

- the program terminates and doesn't contain any loops
- the program doesn't use any restricted kernel functions
- the program doesn't access uninitialized contents / registers
- the program's size is within the limit
- the program doesn't have any out of bounds or malformed jumps

![](https://ebpf.io/static/go-1a1bb6f1e64b1ad5597f57dc17cf1350.png)

# References

* https://ebpf.io
* https://nakryiko.com/posts/libbpf-bootstrap/
* https://nakryiko.com/posts/bpf-portability-and-co-re/
* https://github.com/iovisor/bcc/blob/master/docs/reference_guide.md
* https://github.com/iovisor/bpftrace
