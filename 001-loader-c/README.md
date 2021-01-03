## Hello eBPF

Let's build our hello-world like eBPF elf object.

```
#include <linux/bpf.h>
#define SEC(NAME) __attribute__((section(NAME), used))

static int (*bpf_trace_printk)(const char *fmt, int fmt_size,
                               ...) = (void *)BPF_FUNC_trace_printk;

SEC("tracepoint/syscalls/sys_enter_execve")
int bpf_prog(void *ctx) {
  char msg[] = "Hello, BPF World!";
  bpf_trace_printk(msg, sizeof(msg));
  return 0;
}

char _license[] SEC("license") = "GPL";
```

Let's compile it!

```
clang -O2 -target bpf -c hello_bpf.c -I/kernel-src/tools/testing/selftests/bpf -o hello_bpf.o
# where /kernel-src is the kernel source code repository (must match with the kernel version running on your VM/host/container)
```

## Loader

An eBPF `loader` is a program that loads the eBPF program in the kernel.

```
#include "bpf_load.h"
#include <stdio.h>

int main(int argc, char **argv) {
  if (load_bpf_file("hello_ebpf.o") != 0) {
    printf("The kernel didn't load the BPF program\n");
    return -1;
  }
  read_trace_pipe();
  return 0;
}
```

Let's compile it!

```
clang -DHAVE_ATTR_TEST=0 -o hello-ebpf -lelf -lbpf -I/kernel-src/samples/bpf -I/kernel-src/tools/lib -I/kernel-src/tools/perf -I/kernel-src/tools/include /kernel-src/samples/bpf/bpf_load.c loader.c
```

## Run it!

Run it on one shell (output should be empty at first).

Open a different shell and run any other program (`ps`, `ls`, ...) and `hello-ebpf` should output something like this:

```
$ ./hello-ebpf
            bash-20629 [000] ....  3135.749102: 0: Hello, BPF World!
            bash-20630 [000] ....  3143.261155: 0: Hello, BPF World!
            bash-20631 [000] ....  3143.983065: 0: Hello, BPF World!
            bash-20632 [000] ....  3145.170219: 0: Hello, BPF World!
```
