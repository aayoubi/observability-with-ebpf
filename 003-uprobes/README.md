### uprobes

Uprobes have their origins in utrace, a deprecated user-space tracing system in the Linux Kernel, and found their way into the Kernel in version 3.5. Uprobes (or User-space probes) are the equivalent of Kprobes but for programs running in user-space. That means `uprobes` gives you access to instructions executed by any program in user-space. For exemple, let's say I want to monitor any program calling `libc`'s `malloc`, I can rely on `uprobes` in order to execute a callback whenever the `malloc` function is executed by any program.

### How does it work?

The kernel creates a trap around the attached instruction. When your program reaches that instruction, the kernel triggers an event that has your probe function as a callback. Uprobes also give you access to any library that your program is linked to, and you can trace those calls if you know the correct name for the instruction.

### uprobes with bpftrace

We can trace which process is calling the `libc` function `malloc`:

```
bpftrace -e '
BEGIN
{
    printf("%-9s %-6s\n", "PID", "BYTES");
}

uprobe:/lib/x86_64-linux-gnu/libc.so.6:malloc
/comm == "python3"/
{
    if (arg0 >  1048576) {
        printf("%-6d%10d\n", pid, arg0);
    }
}'
```

The above script would define a `uprobe` and attach it to `libc.so.6:malloc` that is usually found under `/lib/x86_64-linux-gnu/` (on Ubuntu). So whenever a process linking to this library (it's usually almost all of them) calls `malloc`, the above code would be triggered; in this simple case, we're just printing the process id and the amount of the bytes that the process asked for if the allocation size exceeded 1 KB.

In another exemple, we can attach to `readline` and print out the commands executed by any `Bash` shells running on the host:

```
BEGIN
{
    printf("%-9s %-6s %s\n", "TIME", "PID", "COMMAND");
}

uretprobe:/bin/bash:readline
{
    time("%H:%M:%S  ");
    printf("%-6d %s\n", pid, str(retval));
}
```

In this case, we're attaching to the `uretprobe`, which are exactly similar to kretprobes; they get executed after function returns.
