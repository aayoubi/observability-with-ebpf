### kprobes

kprobes enable a dynamic entrypoint to monitor events in the Kernel.

https://www.kernel.org/doc/Documentation/kprobes.txt

### kprobes with bpftrace

List all kprobes available on your host with `bpftrace -l 'kprobe:*'`.

Trace which process is calling the syscall `do_nanosleep`:

```bash
$ bpftrace -e 'kprobe:do_nanosleep {printf("sleep got called by %d-%d\n", pid, tid)}'
```

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
