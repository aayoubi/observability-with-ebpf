### kprobes

Kernel probes allow you to define dynamic traps or breaks (similar to debugging breakpoints) for any Kernel instruction. 

### How does it work?

The kernel creates a trap around the attached instruction. When your program reaches that instruction, the kernel triggers an event that has your probe function as a callback. Kernel probes give you access to any instruction executed by the Kernel, and you can trace those calls if you know the correct name for the instruction. While programs are running on your Linux host, if the Kernel reaches any of those defined traps, it will execute the code attached to that probe, and then resume its usual routine. As such, we can execute code when a process opens a file, or creates a new socket, or executes any system call. 

`kprobes` allow us to attach eBPF code to any kernel instruction **before** it gets executed. `kretprobes` allow us to attach eBPF code to any kernel instruction **after** after it gets executed and returns a value.

https://www.kernel.org/doc/Documentation/kprobes.txt

### kprobes with bpftrace

We can trace which process is calling the system call `do_nanosleep`:

```bash
$ bpftrace -e 'kprobe:do_nanosleep {printf("sleep will be called by %d-%d\n", pid, tid)}'
#              ^^^^^^ 
#              type of probe
#                     ^^^^^^^^^^^^ 
#                     name of the system call to attach
#                                   ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
#                                  code to execute whenever the Kernel goes through `do_nanosleep`
```

Similarly, a `kretprobe` would be exactly similar, except that our bpftrace code would be executed after the `do_nanosleep` function returns:

```bash
$ bpftrace -e 'kretprobe:do_nanosleep {printf("sleep got called by %d-%d\n", pid, tid)}'
```

We can list all the supported kernel probes available on your host with `bpftrace -l 'kprobe:*'`

### kprobes with BCC

Declare the function with the prefix `kprobe__<kernel-fn-name>`, or use the helper method `bpf.attach_kprobe`

```python
from bcc import BPF

bpf = BPF(text="""
int kprobe__do_nanosleep(struct pt_regs *ctx) {
  char comm[16];
  bpf_get_current_comm(&comm, sizeof(comm));
  bpf_trace_printk("executing program: %s\\n", comm);
  return 0;
}
""")

bpf.trace_print()
```

https://github.com/iovisor/bcc/blob/master/docs/reference_guide.md
