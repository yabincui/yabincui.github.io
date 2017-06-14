---
layout: post
title:  "Linux kprobe"
date:   2017-05-24 14:14:00 -0700
categories: data structure
---

# Kprobe
[linux/Documentation/kprobes.txt](https://www.kernel.org/doc/Documentation/kprobes.txt)

kprobes is an API interface used in kernel. Given a function, it replaces the first instruction with int3, so an exception is raised when running the function, and the pre_handler of the registered kprobe is called before running the real function.

The interface to check kprobes situation is at /sys/kernel/debug/kprobes.

Jprobes is implemented using kprobes to hook at the entry of a function. Return Probes is implemented to hook at the exit of a function.

# Kprobe-based event tracing
[linux/Documentation/trace/kprobetrace.txt](https://www.kernel.org/doc/Documentation/trace/kprobetrace.txt)

kprobe-based event tracing is to use tracing interface to access kprobe. By setting kprobe functions and dumped items (arguments or return value), we can trace dynamic kernel function calls. It is very interesting.

[perf-tools](https://github.com/brendangregg/perf-tools) contains many examples of using kprobe-based event  tracing. Below is an example of tracing executed commands on x86_64.
```
#!/usr/bin/python

import sys
import os

class Trace(object):
    def __init__(self):
        self.kprobe_list = []
        self.clean()

    def clean(self):
        self.write_file('0', 'tracing_on')
        for kprobe_name in self.kprobe_list:
            self.write_file('0', 'events/kprobes/%s/enable' % kprobe_name)
        self.write_file('', 'kprobe_events')

    def write_file(self, content, filename, append=False):
        flag = 'a' if append else 'w'
        with open('/sys/kernel/debug/tracing/' + filename, flag) as fh:
            fh.write(content)

    def add_kprobe(self, kprobe_name, kprobe_content):
        s = 'p:%s %s %s' % (kprobe_name, kprobe_name, kprobe_content)
        try:
            self.write_file(s, 'kprobe_events')
        except:
            print('wrong kprobe format: %s' % s)
        self.kprobe_list.append(kprobe_name)

    def trace(self):
        for kprobe_name in self.kprobe_list:
            self.write_file('1', 'events/kprobes/%s/enable' % kprobe_name)
        self.write_file('1', 'tracing_on')
        with open('/sys/kernel/debug/tracing/trace_pipe', 'r') as fh:
            while True:
                a = fh.readline()
                print(a)

def trace_sys_execve():
    max_argc = 16
    step = 8
    kprobe_content = ''
    for i in range(max_argc):
        kprobe_content += ' +0(+%d(%%si)):string' % (i * step)
    trace = Trace()
    trace.add_kprobe('sys_execve', kprobe_content)
    try:
        trace.trace()
    except KeyboardInterrupt:
        pass
    finally:
        trace.clean()

def main():
    trace_sys_execve()

if __name__ == '__main__':
    main()
```

linux function call convention for x86_64 is [here](http://courses.cs.washington.edu/courses/cse378/10au/sections/Section1_recap.pdf).
linux system call convention is [here](http://man7.org/linux/man-pages/man2/syscall.2.html).

# ftrace design
[linux/Documentation/trace/ftrace-design.txt](https://www.kernel.org/doc/Documentation/trace/ftrace-design.txt)



# perf-tools
[perf-tools](https://github.com/brendangregg/perf-tools) contains many examples of using kprobe-based event  tracing. These tools are listed as below:

execsnoop traces all executed commands. The script misses `echo 0 >/sys/kernel/debug/tracing/tracing_on`. After adding it, it works perfect.

opensnoop traces all open file system calls.

 kprobe is a convenient way to set kprobe functions and start tracing.
 
 functrace uses `function` tracer to trace kernel functions.
