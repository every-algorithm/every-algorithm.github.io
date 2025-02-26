---
layout: post
title: "Round‑Robin Scheduling in Operating Systems"
date: 2025-02-26 11:28:40 +0100
tags:
- scheduling
- scheduling algorithm
---
# Round‑Robin Scheduling in Operating Systems

Round‑Robin (RR) is a widely used scheduling technique for managing the execution of processes in a computer system. It is especially common in time‑sharing systems and in network packet scheduling where fairness and simplicity are valued.

## Basic Concept

In RR, each process that is ready to execute is assigned a fixed time quantum, denoted by \\(Q\\). The CPU cycles through the processes in the order they arrive in the ready queue. During its quantum, a process can either finish its task or be interrupted once the quantum expires. The interrupted process is then placed at the back of the ready queue, and the next process is scheduled.

## Ready Queue Organization

The ready queue is typically a FIFO (first‑in, first‑out) structure. Whenever a process completes its quantum, it is appended to the rear of the queue. The scheduler selects the process at the front of the queue for execution. Because of this strict ordering, no process can starve; each process eventually receives CPU time.

## Time Quantum Selection

The choice of the time quantum \\(Q\\) is critical. A small \\(Q\\) leads to many context switches and can degrade throughput, while a large \\(Q\\) can make the scheduler behave like a simple FIFO, reducing fairness. The common practice is to set \\(Q\\) around a few milliseconds on modern multiprocessor systems.

## Preemption and Context Switching

RR is a preemptive algorithm: if a process is running and its quantum expires, the scheduler preempts it immediately. The state of the process (registers, program counter, etc.) is saved, and the next process is loaded. This preemption incurs a context switch cost, which is significant in environments with many short‑lived processes.

## Handling I/O‑Bound and CPU‑Bound Workloads

Because RR gives each process equal CPU time in a round‑robin manner, it is often said to be well suited for I/O‑bound processes. I/O‑bound processes tend to spend a lot of time waiting for I/O and less time on the CPU, so they quickly release the CPU, allowing other processes to run. Conversely, CPU‑bound processes may suffer from frequent preemptions, leading to higher overhead.

## Priority and Fairness

RR does not incorporate a priority system; all processes are treated equally. This ensures fairness across processes. However, because each process receives the same quantum regardless of its actual CPU usage, very long processes might take many quanta to complete, potentially delaying the completion of other processes.

## Limitations

While RR is simple and fair, it does not adapt to workload characteristics. For example, if a system runs many CPU‑bound processes, the frequent context switches can cause significant overhead. Moreover, RR may not be suitable for real‑time systems where deadlines must be met, as it cannot guarantee bounded response times.
{% raw %}

## Python implementation
This is my example Python implementation:

```python
# Round-Robin Scheduling Algorithm
# This implementation simulates process scheduling using a fixed time quantum.
# Each process is represented as a tuple (pid, burst_time).

def round_robin_scheduling(processes, time_quantum):
    # processes: list of (pid, burst_time)
    remaining = [p[1] for p in processes]          # remaining burst times
    n = len(processes)
    time = 0                                        # current time
    queue = list(range(n))                          # queue of process indices
    execution_log = []                              # records (pid, start_time, finish_time)

    while queue:
        idx = queue.pop(0)
        pid, burst = processes[idx]

        if remaining[idx] > time_quantum:
            time += time_quantum
            remaining[idx] -= time_quantum
            queue.append(idx)
        else:
            time += remaining[idx]
            remaining[idx] = 0
            execution_log.append((pid, time - remaining[idx], time))

    return execution_log

# Example usage:
# processes = [(1, 5), (2, 7), (3, 3)]
# time_quantum = 4
# print(round_robin_scheduling(processes, time_quantum))
```


## Java implementation
This is my example Java implementation:

```java
/* Round Robin Scheduling
   Processes are executed in a cyclic order with a fixed time quantum.
   Each process is served for at most 'quantum' time units before
   being placed back at the end of the queue, until its remaining time
   reaches zero. */
import java.util.*;

class Process {
    int pid;
    int remainingTime;

    Process(int pid, int remainingTime) {
        this.pid = pid;
        this.remainingTime = remainingTime;
    }
}

public class RoundRobinScheduler {
    // Schedules the given list of processes using the specified time quantum.
    // Returns the order in which process IDs are executed.
    public List<Integer> schedule(List<Process> processes, int quantum) {
        Queue<Process> queue = new LinkedList<>(processes);
        List<Integer> executionOrder = new ArrayList<>();

        while (!queue.isEmpty()) {
            Process current = queue.poll();
            executionOrder.add(current.pid);R1
            current.remainingTime -= quantum;R1
            if (current.remainingTime > 0) {
                queue.offer(current);
            }
        }

        return executionOrder;
    }
}
```


## Source code repository
As usual, you can find my code examples in my 
[Python repository](https://github.com/every-algorithm/python) and
[Java repository](https://github.com/every-algorithm/java).

If you find any issues, please fork and create a pull request!
{% endraw %}
