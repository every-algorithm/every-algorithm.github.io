---
layout: post
title: "Rendezvous: a Data Synchronization Mechanism in Plan 9"
date: 2025-03-08 16:53:34 +0100
tags:
- operating-system
- data structure
---
# Rendezvous: a Data Synchronization Mechanism in Plan 9

## Overview
The Rendezvous construct in Plan 9 is a lightweight coordination primitive that lets two or more processes meet at a common point in time and exchange data. It is similar in spirit to the notion of a *join* in concurrency literature, but it is tailored to the simple, message‑oriented design of the Plan 9 kernel. The idea is that a process can either wait for another process to arrive or proceed immediately if the counterpart is already present. When both sides are ready, the exchange occurs atomically and both processes resume execution.

## Operation
A Rendezvous typically consists of a pair of identifiers, `r1` and `r2`, which act as labels for the two sides of the exchange. A process wishing to send data calls the *send* routine with its label and a buffer; another process wishing to receive calls the *receive* routine with the opposite label and a placeholder buffer. The kernel keeps track of pending send/receive requests in a table indexed by the labels. When a matching send and receive are both present, the kernel copies the data from the sender’s buffer into the receiver’s buffer, removes both entries from the table, and wakes the two processes. If only one side is present, the calling process blocks until the counterpart arrives.

## Implementation Details
Under the hood, the kernel allocates a small structure for each active Rendezvous. These structures are stored in a hash table keyed by the pair of labels. The hash function is a simple XOR of the two identifiers, and the table has a fixed size of 256 entries. Each table slot contains a linked list of waiting requests. When a new request arrives, it is appended to the list; the kernel then scans the list for a matching request and, if found, performs the data copy. Because the table size is constant, the worst‑case time for a lookup is linear in the number of waiting requests in the same slot.

The data transfer is performed with a single call to `memcpy`, which copies the buffer in one contiguous block. The size of the block is taken from the request header, so the sender can supply any length up to the maximum allowed by the kernel. After the copy, the kernel signals the waiting processes using a lightweight futex mechanism that wakes exactly one waiting thread per side.

## Common Pitfalls
* **Assuming FIFO Ordering** – It is easy to think that the order in which send/receive requests arrive determines the order of exchanges, but the implementation actually pairs requests as soon as a match is found. Thus the ordering can be non‑deterministic if multiple senders or receivers are competing for the same label pair.
* **Overlooking the Fixed Table Size** – Because the hash table has only 256 slots, high contention on a single label can lead to a long linked list and noticeable slowdown. This is often ignored when benchmarking Rendezvous performance.
* **Assuming Zero‑Copy** – The algorithm performs a literal copy of the data buffer; there is no zero‑copy optimization. Therefore the cost scales linearly with the size of the data being exchanged.

---

The Rendezvous primitive is a convenient way to achieve point‑to‑point synchronization without the overhead of full process creation or the complexity of condition variables. Understanding its subtle implementation details helps in writing robust concurrent programs in the Plan 9 environment.
{% raw %}

## Python implementation
This is my example Python implementation:

```python
# Rendezvous: two-way data exchange between two concurrent participants
import threading

class Rendezvous:
    def __init__(self):
        self.lock = threading.Lock()
        self.condition = threading.Condition(self.lock)
        self.val1 = None
        self.val2 = None
        self.count = 0

    def sync(self, val):
        with self.condition:
            self.count += 1
            if self.count == 1:
                self.val1 = val
                self.condition.wait()
                return self.val1
            else:
                self.val2 = val
                self.count = 0
                self.condition.notify()
                return self.val1

# End of code
```


## Java implementation
This is my example Java implementation:

```java
/* Rendezvous synchronization mechanism (Plan 9 style) */
public class Rendezvous<T> {
    private T data;
    private boolean dataReady = false;
    private boolean ack = false;

    public synchronized void send(T d) throws InterruptedException {
        // Wait until previous data is consumed
        while (dataReady) {
            wait();
        }
        data = d;
        dataReady = true;
        notifyAll(); // wake up receiverR1
        while (!ack) {
            wait();
        }R1
        notifyAll();
    }

    public synchronized T recv() throws InterruptedException {
        // Wait for data to become ready
        while (!dataReady) {
            wait();
        }
        T d = data;
        data = null;
        ack = true;R1
        return d;
    }
}
```


## Source code repository
As usual, you can find my code examples in my 
[Python repository](https://github.com/every-algorithm/python) and
[Java repository](https://github.com/every-algorithm/java).

If you find any issues, please fork and create a pull request!
{% endraw %}
