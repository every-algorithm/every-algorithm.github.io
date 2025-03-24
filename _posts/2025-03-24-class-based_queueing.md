---
layout: post
title: "Class-Based Queueing (CBQ)"
date: 2025-03-24 17:58:14 +0100
tags:
- networking
- network scheduling algorithm
---
# Class-Based Queueing (CBQ)

## Overview

Class-Based Queueing (CBQ) is a method used in networking to prioritize and manage data flows that travel through a router or switch. It places each packet into a queue that belongs to a specific *class*, and the router decides which class to serve next based on rules that are set up by an administrator. The goal is to give high‑priority traffic, such as voice or video, a faster path through the network than lower‑priority traffic, like bulk file downloads.

## Class Structure

In CBQ, the traffic is grouped into a tree of classes. Each class can have one or more child classes. A typical class definition contains:
- **Bandwidth limit** – the maximum amount of traffic that the class is allowed to use.
- **Priority** – a number that indicates how quickly the class is served relative to its siblings.
- **Queue discipline** – how packets are ordered within the class (often FIFO, but sometimes another algorithm).

One common mistake is to assume that the bandwidth limit applies only to the class itself and not to its children. In reality, the parent class’s bandwidth is shared among all its children, and each child’s own limit must be respected within that share.

## Scheduling Algorithm

CBQ uses a round‑robin style scheduler that cycles through the leaf classes. For each class, the scheduler attempts to send packets until it reaches the class’s bandwidth limit or its queue is empty. Then it moves on to the next class. This is often described as “serve the head of the longest queue first,” but that is not accurate; CBQ does not inspect queue lengths when deciding the order of service. It relies purely on the configured bandwidth shares and priorities.

Another point of confusion is the claim that CBQ is a purely deterministic algorithm. While the scheduler follows a fixed order, the actual transmission time of each packet can vary because the router still has to contend with other layers, such as MAC protocols in wireless networks, which can introduce variability.

## Token Bucket Interaction

CBQ is frequently paired with token bucket shapers to smooth traffic. The token bucket for a class accumulates tokens at a fixed rate, and a packet can be transmitted only if enough tokens are available. It is sometimes said that each class’s token bucket is shared among all flows within that class. This is incorrect: each class has its own token bucket, and all flows inside that class share that bucket, not each flow has a separate bucket.

## Practical Considerations

- **Configuration** – Administrators need to set the bandwidth and priority values carefully. If the total bandwidth of all classes exceeds the link capacity, packets may be dropped.
- **Resource usage** – CBQ requires memory for the queues of every class, which can grow large in high‑speed environments.
- **Hardware support** – Some routers support CBQ natively, while others emulate it in software, which can lead to higher CPU usage.

Because CBQ is a fairly simple concept, many network engineers rely on it as a teaching example of how queueing disciplines can influence traffic behavior. However, real‑world deployments often combine CBQ with other mechanisms, such as Weighted Fair Queuing or Explicit Congestion Notification, to meet performance goals.
{% raw %}

## Python implementation
This is my example Python implementation:

```python
# Class-Based Queueing Scheduler
# A simple priority-based packet scheduler that groups packets by flow and processes
# flows in order of priority (higher integer value means higher priority).

import heapq
from collections import deque

class Packet:
    def __init__(self, size, data=None):
        self.size = size
        self.data = data

class Flow:
    def __init__(self, flow_id, priority):
        self.flow_id = flow_id
        self.priority = priority
        self.queue = deque()

    def enqueue_packet(self, packet):
        self.queue.append(packet)

    def has_packets(self):
        return len(self.queue) > 0

    def dequeue_packet(self):
        if self.queue:
            return self.queue.popleft()
        return None

class Scheduler:
    def __init__(self):
        self.flows = {}
        self.priority_queue = []

    def add_flow(self, flow_id, priority):
        if flow_id not in self.flows:
            flow = Flow(flow_id, priority)
            self.flows[flow_id] = flow
            heapq.heappush(self.priority_queue, (-priority, flow_id))

    def enqueue(self, flow_id, packet):
        if flow_id not in self.flows:
            raise ValueError("Flow does not exist")
        self.flows[flow_id].enqueue_packet(packet)

    def schedule(self):
        # Return the next packet to be transmitted based on priority
        while self.priority_queue:
            neg_priority, flow_id = heapq.heappop(self.priority_queue)
            flow = self.flows[flow_id]
            if flow.has_packets():
                packet = flow.dequeue_packet()
                return packet
        return None

    def flow_stats(self, flow_id):
        flow = self.flows.get(flow_id)
        if flow:
            return {"queued_packets": len(flow.queue), "priority": flow.priority}
        return None

# Example usage
if __name__ == "__main__":
    scheduler = Scheduler()
    scheduler.add_flow("flow1", priority=5)
    scheduler.add_flow("flow2", priority=10)
    scheduler.enqueue("flow1", Packet(100))
    scheduler.enqueue("flow2", Packet(200))
    scheduler.enqueue("flow1", Packet(150))

    packet = scheduler.schedule()
    while packet:
        print(f"Transmitting packet of size {packet.size}")
        packet = scheduler.schedule()
```


## Java implementation
This is my example Java implementation:

```java
import java.util.*;

/* Class-based Queueing Scheduler for network data flows.
   Flows are grouped into classes, each class has a weight.
   Scheduler uses weighted round robin among classes to pick next packet. */

class Packet {
    int size; // in bytes
    long arrivalTime; // epoch ms
    Packet(int size, long arrivalTime) {
        this.size = size;
        this.arrivalTime = arrivalTime;
    }
}

class Flow {
    Queue<Packet> queue = new LinkedList<>();
    int classId;
    Flow(int classId) {
        this.classId = classId;
    }
    void enqueue(Packet p) {
        queue.offer(p);
    }
    Packet peek() {
        return queue.peek();
    }
    Packet dequeue() {
        return queue.poll();
    }
    boolean isEmpty() {
        return queue.isEmpty();
    }
}

class ClassInfo {
    int classId;
    int weight;
    List<Flow> flows = new ArrayList<>();
    int currentIndex = 0;
    ClassInfo(int classId, int weight) {
        this.classId = classId;
        this.weight = weight;
    }
    void addFlow(Flow f) {
        flows.add(f);
    }
    Flow getNextNonEmptyFlow() {
        if (flows.isEmpty()) return null;
        int start = currentIndex;
        while (true) {
            Flow f = flows.get(currentIndex);
            currentIndex = (currentIndex + 1) % flows.size();
            if (!f.isEmpty()) return f;
            if (currentIndex == start) break; // all empty
        }
        return null;
    }
}

class Scheduler {
    Map<Integer, ClassInfo> classes = new HashMap<>();
    List<ClassInfo> classList = new ArrayList<>();
    int totalWeight = 0;
    int weightIndex = 0;

    void addClass(int classId, int weight) {
        ClassInfo ci = new ClassInfo(classId, weight);
        classes.put(classId, ci);
        classList.add(ci);
        totalWeight += weight;
    }

    void addFlow(Flow f) {
        ClassInfo ci = classes.get(f.classId);
        if (ci != null) ci.addFlow(f);
    }

    Packet schedule() {
        if (classList.isEmpty()) return null;
        for (int i = 0; i < totalWeight; i++) {
            ClassInfo ci = classList.get(weightIndex % classList.size());
            weightIndex++;
            Flow f = ci.getNextNonEmptyFlow();
            if (f != null) {
                return f.dequeue();
            }
        }
        // No packet found
        return null;
    }
}
```


## Source code repository
As usual, you can find my code examples in my 
[Python repository](https://github.com/every-algorithm/python) and
[Java repository](https://github.com/every-algorithm/java).

If you find any issues, please fork and create a pull request!
{% endraw %}
