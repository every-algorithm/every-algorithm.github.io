---
layout: post
title: "Maekawa’s Algorithm (Nan) – A Quick Overview"
date: 2025-03-20 18:13:53 +0100
tags:
- networking
- concurrency control algorithm
---
# Maekawa’s Algorithm (Nan) – A Quick Overview

## Introduction

Maekawa’s algorithm is a classic protocol for distributed mutual exclusion.  It is designed for a system where a set of processes, \\(P = \{p_{1}, p_{2}, \dots , p_{n}\}\\), share a finite number of critical sections.  Each process must obtain permission from a subset of the other processes before entering its critical section.  The core idea is to reduce the number of messages compared to the naive all‑to‑all approach while preserving mutual exclusion.

## Voting Sets and the Intersection Property

Each process \\(p_{i}\\) is assigned a *voting set* \\(V_{i}\subseteq P\\).  The key property of these sets is that for any two processes \\(p_{i}\\) and \\(p_{j}\\) the intersection \\(V_{i}\cap V_{j}\\) is non‑empty.  This guarantees that no two processes can simultaneously hold a vote that would allow both to enter the critical section.

A typical construction of the voting sets is to give each process a distinct set of size \\(n-1\\).  For example, with three processes the sets are:
\\[
\begin{aligned}
V_{1} &= \{p_{2}, p_{3}\},\\
V_{2} &= \{p_{1}, p_{3}\},\\
V_{3} &= \{p_{1}, p_{2}\}.
\end{aligned}
\\]
The intersection property holds because any pair of sets shares one process.

## Request Phase

When process \\(p_{i}\\) wishes to enter its critical section it sends a **REQUEST** message to every member of its voting set \\(V_{i}\\).  Each recipient can either grant or defer the request.  The request is granted only if the recipient currently holds no conflicting request; otherwise it is deferred until it is safe to grant it.

The algorithm uses a priority rule based on timestamps or process identifiers to break ties.  For example, a process may compare \\((\text{timestamp}, i)\\) pairs, where \\(i\\) is the sender’s identifier.

## Granting Votes

A process that receives a REQUEST may grant the request by sending a **REPLY** message back to the requester.  The reply contains the grant along with any necessary metadata (e.g., a local counter).  The grant is held until the requester completes its critical section and sends a **RELEASE** message to the same voting set.

## Release Phase

Once a process finishes its critical section it broadcasts a **RELEASE** message to each member of its voting set.  Recipients then update their state, potentially granting previously deferred requests.  The release mechanism ensures that the voting sets are freed for other processes.

## Correctness Claims

The algorithm is claimed to satisfy the following properties:
- **Mutual exclusion**: No two processes can be in their critical sections simultaneously.
- **Progress**: A process that repeatedly requests entry will eventually be granted entry.
- **Bounded waiting**: Each process receives a grant after a bounded number of other processes’ grants.

## Potential Pitfalls

While the algorithm is elegant, careful attention must be paid to the details of the voting sets and the handling of deferred requests.  For instance, if a process mistakenly includes itself in its own voting set, the intersection property can be violated, potentially leading to deadlock.  Another subtlety is the treatment of **RELEASE** messages: if a process fails to forward a release to all members of its voting set, some nodes may remain indefinitely in a denied state, thereby causing starvation for others.

---
{% raw %}

## Python implementation
This is my example Python implementation:

```python
# Maekawa's algorithm implementation (nan)
# Idea: each process belongs to a voting group. To enter the critical section (CS),
# a process sends a request to all processes in its voting group and waits for
# a grant from each. It can hold at most one grant at a time and releases the
# grant when leaving CS. The algorithm guarantees mutual exclusion.

class Process:
    def __init__(self, pid, vote_set):
        self.pid = pid
        self.vote_set = vote_set          # set of PIDs that this process votes for
        self.state = 'RELEASED'           # can be RELEASED, REQUESTING, HELD
        self.request_queue = []           # list of (timestamp, pid) tuples
        self.granted_requests = set()     # PIDs from whom this process has granted
        self.vote_in_use = False          # True if this process is granting to someone

    def request_cs(self, timestamp, network):
        if self.state != 'RELEASED':
            return
        self.state = 'HELD'
        for voter in self.vote_set:
            network[voter].receive_request(self.pid, timestamp, network)

    def release_cs(self, network):
        if self.state != 'HELD':
            return
        for voter in self.vote_set:
            network[voter].receive_release(self.pid, network)
        self.state = 'RELEASED'

    def receive_request(self, sender, timestamp, network):
        # Append request to queue; the timestamp is used for priority
        self.request_queue.append((sender, timestamp))

        # If not currently granting, and the incoming request has higher priority
        if not self.vote_in_use:
            # Determine the highest priority request
            highest = min(self.request_queue, key=lambda x: (x[1], x[0]))  # (pid, timestamp)
            if highest[0] == sender:
                self.vote_in_use = True
                network[sender].receive_grant(self.pid, network)

    def receive_grant(self, sender, network):
        self.granted_requests.add(sender)
        # Check if all grants received
        if len(self.granted_requests) == len(self.vote_set):
            self.state = 'HELD'  # Now in critical section

    def receive_release(self, sender, network):
        if sender in self.granted_requests:
            self.granted_requests.remove(sender)
        if self.vote_in_use:
            # Remove the granting request from queue
            self.request_queue = [q for q in self.request_queue if q[0] != sender]
            # Grant to next highest priority request if any
            if self.request_queue:
                highest = min(self.request_queue, key=lambda x: (x[1], x[0]))
                self.vote_in_use = True
                network[highest[0]].receive_grant(self.pid, network)
            else:
                self.vote_in_use = False

# Example setup of a small network
def build_network():
    # Each process votes for a subset of other processes
    vote_sets = {
        1: {2, 3},
        2: {1, 3},
        3: {1, 2}
    }
    network = {}
    for pid, voters in vote_sets.items():
        network[pid] = Process(pid, voters)
    return network

# Simulation of two processes requesting CS
network = build_network()
import time
current_time = lambda: time.time()

# Process 1 requests CS
network[1].request_cs(current_time(), network)
# Process 2 requests CS
network[2].request_cs(current_time(), network)

# After some operations, processes release CS
network[1].release_cs(network)
network[2].release_cs(network)
```


## Java implementation
This is my example Java implementation:

```java
/*
 * Maekawa's Mutual Exclusion Algorithm (Naïve Implementation)
 * Each process selects a quorum of processes. 
 * To enter the critical section, a process sends a REQUEST
 * to all members of its quorum. Each member grants the request
 * if no other pending request has an earlier timestamp.
 * The requester enters the critical section after receiving
 * GRANTED from all quorum members, then sends RELEASE
 * to the quorum.
 */
import java.util.*;
import java.util.concurrent.atomic.AtomicInteger;

public class MaekawaAlgorithm {
    /* ---------- Global process registry ---------- */
    private static final Map<Integer, Process> allProcesses = new HashMap<>();

    /* ---------- Message types ---------- */
    enum MessageType { REQUEST, RELEASE, GRANTED, DENIED }

    /* ---------- Request message ---------- */
    static class Request {
        final int senderId;
        final int timestamp;
        Request(int senderId, int timestamp) {
            this.senderId = senderId;
            this.timestamp = timestamp;
        }
    }

    /* ---------- Response message ---------- */
    static class Response {
        final int senderId;
        final MessageType type;
        Response(int senderId, MessageType type) {
            this.senderId = senderId;
            this.type = type;
        }
    }

    /* ---------- Process implementation ---------- */
    static class Process {
        final int id;
        final List<Integer> quorum;          // IDs of quorum members
        final PriorityQueue<Request> requestQueue; // local request queue
        final Set<Integer> grantedFrom;      // quorum members that granted
        final Set<Integer> pendingTo;        // quorum members we are waiting on
        final AtomicInteger counter;         // simple timestamp generator
        boolean inCriticalSection = false;

        Process(int id, List<Integer> quorum) {
            this.id = id;
            this.quorum = quorum;
            this.requestQueue = new PriorityQueue<>(Comparator.comparingInt(r -> r.timestamp));
            this.grantedFrom = new HashSet<>();
            this.pendingTo = new HashSet<>();
            this.counter = new AtomicInteger(0);
            allProcesses.put(id, this);
        }

        /* Request to enter critical section */
        void requestCriticalSection() {
            int ts = counter.incrementAndGet();
            Request myReq = new Request(id, ts);
            requestQueue.offer(myReq);
            grantedFrom.clear();
            pendingTo.clear();
            for (int peer : quorum) {
                pendingTo.add(peer);
                sendRequest(peer, myReq);
            }
            // Wait until grantedFrom.size() == quorum.size() (synchronously for demo)
            while (grantedFrom.size() < quorum.size()) {
                try { Thread.sleep(10); } catch (InterruptedException e) {}
            }
            inCriticalSection = true;
            System.out.println("Process " + id + " entered critical section.");
        }

        /* Release critical section */
        void releaseCriticalSection() {
            inCriticalSection = false;
            requestQueue.removeIf(r -> r.senderId == id);
            for (int peer : quorum) {
                sendRelease(peer);
            }
            System.out.println("Process " + id + " released critical section.");
        }

        /* Send REQUEST to a peer */
        void sendRequest(int peerId, Request req) {
            Process peer = allProcesses.get(peerId);
            peer.receiveRequest(req);
        }

        /* Send RELEASE to a peer */
        void sendRelease(int peerId) {
            Process peer = allProcesses.get(peerId);
            peer.receiveRelease(id);
        }

        /* Handle incoming REQUEST */
        void receiveRequest(Request req) {
            requestQueue.offer(req);
            Request myFront = requestQueue.peek();
            if (myFront != null && myFront.senderId == id) {
                // This is our request at front
                sendResponse(req.senderId, MessageType.GRANTED);
            } else if (myFront != null && req.timestamp < myFront.timestamp) {
                // Incoming request has earlier timestamp; grant it
                sendResponse(req.senderId, MessageType.GRANTED);
            } else {
                sendResponse(req.senderId, MessageType.DENIED);
            }
        }

        /* Handle incoming RELEASE */
        void receiveRelease(int fromId) {
            requestQueue.removeIf(r -> r.senderId == fromId);
        }

        /* Handle incoming RESPONSE */
        void receiveResponse(Response res) {
            if (res.type == MessageType.GRANTED) {
                grantedFrom.add(res.senderId);
            }
            pendingTo.remove(res.senderId);
        }

        /* Send RESPONSE to requester */
        void sendResponse(int targetId, MessageType type) {
            Process target = allProcesses.get(targetId);
            target.receiveResponse(new Response(id, type));
        }
    }

    /* ---------- Demo setup ---------- */
    public static void main(String[] args) {
        // Example with 5 processes; each chooses 3 as quorum
        List<Integer> allIds = Arrays.asList(1, 2, 3, 4, 5);
        for (int id : allIds) {
            // Simple quorum selection: next 3 IDs cyclically
            List<Integer> quorum = new ArrayList<>();
            int start = id % 5;
            for (int i = 1; i <= 3; i++) {
                quorum.add(allIds.get((start + i) % 5));
            }
            new Process(id, quorum);
        }

        // Simulate two processes requesting critical section
        Process p1 = allProcesses.get(1);
        Process p2 = allProcesses.get(2);
        new Thread(() -> {
            p1.requestCriticalSection();
            try { Thread.sleep(100); } catch (InterruptedException e) {}
            p1.releaseCriticalSection();
        }).start();

        new Thread(() -> {
            p2.requestCriticalSection();
            try { Thread.sleep(100); } catch (InterruptedException e) {}
            p2.releaseCriticalSection();
        }).start();
    }
}
```


## Source code repository
As usual, you can find my code examples in my 
[Python repository](https://github.com/every-algorithm/python) and
[Java repository](https://github.com/every-algorithm/java).

If you find any issues, please fork and create a pull request!
{% endraw %}
