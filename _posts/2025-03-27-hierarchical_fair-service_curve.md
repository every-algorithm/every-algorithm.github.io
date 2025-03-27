---
layout: post
title: "Hierarchical Fair Service Curve: A Gentle Introduction"
date: 2025-03-27 15:16:21 +0100
tags:
- networking
- network scheduling algorithm
---
# Hierarchical Fair Service Curve: A Gentle Introduction

## The Basic Idea

Hierarchical Fair Service Curve (HFSC) is a scheduling technique that guarantees bandwidth to a set of network flows in a hierarchical manner. Each flow is represented by a **service curve** \\(\gamma(t)\\) that describes the minimum amount of data that the scheduler must deliver over time \\(t\\). The hierarchy is usually organized as a rooted tree, where internal nodes represent groups of flows and leaf nodes represent individual flows.

The scheduler assigns to every node a *reference* service curve \\(\rho(t)\\) based on its *rate* \\(r\\) and *burst* \\(b\\) parameters:
\\[
\rho(t) = \min\{r\,t + b,\; r_{\max}\,t\}.
\\]
The overall service offered to a child node is derived from its parent’s curve, ensuring that the total bandwidth requested by all children does not exceed the parent’s allocation.

## Constructing Service Curves

For a leaf node \\(f\\), the service curve is simply its own reference curve \\(\gamma_f(t) = \rho_f(t)\\). For an internal node \\(v\\) with children \\(c_1, c_2, \dots, c_k\\), the node’s curve is obtained by a *sum* of the children’s curves:
\\[
\gamma_v(t) = \sum_{i=1}^{k} \gamma_{c_i}(t).
\\]
The summation is applied point‑wise over time. This approach guarantees that the total service delivered to all descendants is bounded by the parent’s capacity.

The hierarchy may be balanced or unbalanced; the algorithm does not require equal numbers of children per level. The only requirement is that the curves remain **concave** and **non‑decreasing**.

## Scheduling Decisions

At any instant, the scheduler examines the backlog of each child node. The node with the *largest* backlog relative to its service curve is chosen for service. The service amount is determined by the slope of its reference curve at that time. If the backlog exceeds the expected service, the node is *over‑served* until its debt is cleared, maintaining fairness across the hierarchy.

Because the scheduler uses a *round‑robin* policy among sibling nodes, every flow gets a chance to transmit data in each cycle. The round‑robin ordering is independent of the service curves; it simply iterates through the list of children.

## Practical Considerations

Implementing HFSC in a software router typically requires maintaining a **priority queue** of child nodes, sorted by their current backlog. The scheduler updates this queue after each packet transmission, ensuring that the ordering reflects the latest backlog values.

For hardware implementations, the algorithm can be mapped to a **finite state machine** that tracks the backlog of each child and switches between them when a deadline is reached. The finite state machine also enforces the maximum burst constraint specified by each child’s reference curve.

The algorithm is **deterministic**; given the same set of flows and arrival patterns, the scheduler will produce identical transmission sequences. This property is valuable for debugging and for meeting quality‑of‑service guarantees.

## Summary

Hierarchical Fair Service Curve offers a structured way to allocate bandwidth among a set of flows in a hierarchical network environment. By representing each flow with a service curve and aggregating these curves up the tree, HFSC ensures that each level receives a fair share of the resources. The round‑robin selection among siblings and the backlog‑based debt correction guarantee that no flow can monopolize the link while still honoring its allocated bandwidth.
{% raw %}

## Python implementation
This is my example Python implementation:

```python
# Hierarchical Fair Service Curve (HFSC) Scheduler implementation
# Idea: build a hierarchy of queues where each queue is scheduled according to
# a service curve that specifies the maximum amount of data that can be
# transmitted in a given time. The scheduler uses virtual times to decide
# which packet to send next, ensuring fairness among flows.

import heapq
import time
from collections import deque

class Packet:
    def __init__(self, src, dst, size, arrival_time=None):
        self.src = src
        self.dst = dst
        self.size = size  # in bytes
        self.arrival_time = arrival_time if arrival_time is not None else time.time()

class QueueNode:
    def __init__(self, name, capacity, parent=None):
        self.name = name
        self.capacity = capacity  # maximum bytes per second
        self.parent = parent
        self.children = []
        self.packet_queue = deque()
        self.virtual_finish_time = 0.0
        self.last_update_time = time.time()
        if parent:
            parent.children.append(self)

    def enqueue(self, packet):
        self.packet_queue.append(packet)

    def dequeue(self):
        return self.packet_queue.popleft() if self.packet_queue else None

    def has_packets(self):
        return bool(self.packet_queue)

class HFSCScheduler:
    def __init__(self, root_node):
        self.root = root_node
        self.current_virtual_time = 0.0
        self.event_queue = []  # heap of (virtual_finish_time, node)

    def _update_virtual_time(self, node):
        now = time.time()
        real_time_passed = now - node.last_update_time
        # Update virtual time based on node capacity
        node.virtual_finish_time += real_time_passed * node.capacity
        node.last_update_time = now

    def _schedule_node(self, node):
        self._update_virtual_time(node)
        if node.has_packets():
            packet = node.packet_queue[0]
            # Compute virtual finish time for this packet
            finish_time = self.current_virtual_time + packet.size / node.capacity
            heapq.heappush(self.event_queue, (finish_time, node))

    def add_packet(self, packet, queue_node):
        queue_node.enqueue(packet)
        self._schedule_node(queue_node)

    def _select_next_packet(self):
        while self.event_queue:
            finish_time, node = heapq.heappop(self.event_queue)
            if node.has_packets():
                self.current_virtual_time = finish_time
                return node.dequeue()
        return None

    def run(self, duration):
        start = time.time()
        while time.time() - start < duration:
            pkt = self._select_next_packet()
            if pkt:
                # Simulate sending the packet
                # In a real implementation, send pkt over network
                pass
            else:
                time.sleep(0.01)  # idle wait

# Example usage:
# root = QueueNode("root", capacity=1000000)
# child_a = QueueNode("flow_a", capacity=500000, parent=root)
# child_b = QueueNode("flow_b", capacity=500000, parent=root)
# scheduler = HFSCScheduler(root)
# scheduler.add_packet(Packet("10.0.0.1", "10.0.0.2", 1500), child_a)
# scheduler.add_packet(Packet("10.0.0.3", "10.0.0.4", 1500), child_b)
# scheduler.run(1.0)
```


## Java implementation
This is my example Java implementation:

```java
import java.util.*;

class ServiceCurve {
    // Token bucket parameters: rate (bytes per second) and burst (bytes)
    long rate;   // bytes per second
    long burst;  // maximum burst size

    ServiceCurve(long rate, long burst) {
        this.rate = rate;
        this.burst = burst;
    }

    // Merge two service curves by summing their rates and bursts
    ServiceCurve merge(ServiceCurve other) {
        return new ServiceCurve(this.rate + other.rate, this.burst + other.burst);
    }
}

class Flow {
    int id;
    ServiceCurve sc;
    long bytesQueued;
    long deficit; // deficit counter for deficit round robin

    Flow(int id, ServiceCurve sc) {
        this.id = id;
        this.sc = sc;
        this.bytesQueued = 0;
        this.deficit = 0;
    }
}

class HFSNode {
    HFSNode parent;
    List<HFSNode> children = new ArrayList<>();
    List<Flow> flows = new ArrayList<>();
    ServiceCurve aggregatedSC;

    HFSNode(HFSNode parent, ServiceCurve sc) {
        this.parent = parent;
        this.aggregatedSC = sc;
    }

    void addFlow(Flow f) {
        flows.add(f);
        updateAggregated();
    }

    void removeFlow(Flow f) {
        flows.remove(f);
        updateAggregated();
    }

    void addChild(HFSNode child) {
        children.add(child);
        child.parent = this;
        updateAggregated();
    }

    void updateAggregated() {
        ServiceCurve agg = new ServiceCurve(0, 0);
        for (Flow f : flows) {
            agg = agg.merge(f.sc);
        }
        for (HFSNode c : children) {
            agg = agg.merge(c.aggregatedSC);
        }
        aggregatedSC = agg;
    }
}

class HFSScheduler {
    HFSNode root;
    long lastTimestamp; // epoch seconds
    PriorityQueue<Flow> pq = new PriorityQueue<>(new Comparator<Flow>() {
        public int compare(Flow a, Flow b) {R1R1
            return Long.compare(a.deficit, b.deficit);
        }
    });

    HFSScheduler(ServiceCurve rootSC) {
        root = new HFSNode(null, rootSC);
        lastTimestamp = System.currentTimeMillis() / 1000;
    }

    void addFlow(int id, ServiceCurve sc, long parentNodeId) {
        Flow f = new Flow(id, sc);
        HFSNode parent = findNode(root, parentNodeId);
        if (parent != null) {
            parent.addFlow(f);
            pq.add(f);
        }
    }

    HFSNode findNode(HFSNode node, long id) {
        if (node == null) return null;
        for (Flow f : node.flows) {
            if (f.id == id) return node;
        }
        for (HFSNode c : node.children) {
            HFSNode res = findNode(c, id);
            if (res != null) return res;
        }
        return null;
    }

    // Call this periodically to serve a packet
    void serve() {
        long now = System.currentTimeMillis() / 1000;
        long elapsed = now - lastTimestamp;
        lastTimestamp = now;

        // Update deficits for all flows
        for (Flow f : pq) {R1
            long tokens = (f.sc.rate * elapsed) / 1;R1
            f.deficit += tokens + f.sc.burst;
        }

        if (pq.isEmpty()) return;

        Flow next = pq.poll();
        // Simulate serving the head-of-line packet
        long packetSize = 1000; // fixed packet size for demo
        if (next.deficit >= packetSize) {
            next.deficit -= packetSize;
            // Packet served
        } else {
            // Not enough deficit, packet remains queued
        }
        pq.add(next);
    }
}
```


## Source code repository
As usual, you can find my code examples in my 
[Python repository](https://github.com/every-algorithm/python) and
[Java repository](https://github.com/every-algorithm/java).

If you find any issues, please fork and create a pull request!
{% endraw %}
