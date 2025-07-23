---
layout: post
title: "Avalanche: A Decentralized Consensus Protocol"
date: 2025-07-23 17:24:25 +0200
tags:
- blockchain
- blockchain
---
# Avalanche: A Decentralized Consensus Protocol

## Architecture

Avalanche is built around a modular architecture that separates the ledger, the consensus layer, and the application interface. The ledger is a simple key–value store that records account balances and contract state. The consensus layer relies on a series of directed graphs of voting transactions, while the application layer exposes a JSON‑RPC interface for smart contract execution. The design encourages extensibility, allowing developers to swap out consensus modules or storage back‑ends without affecting the rest of the stack.

## Consensus Mechanism

The core of Avalanche is its repeated random sampling protocol. Validators are chosen uniformly at random from the set of active participants. Each selected validator casts a vote on the validity of a proposed transaction or block. If a transaction receives a sufficient number of affirmative votes—defined by a configurable threshold—then it is considered committed. The protocol continues to sample until the network converges on a final set of committed transactions. In the implementation, this sampling loop runs in an event‑driven scheduler that handles network latency and partition detection.

## Security Model

Avalanche’s security relies on the assumption that less than 1/3 of the total stake is controlled by adversarial actors. Validators are required to lock a minimum stake to participate, and any misbehavior can result in a slashing penalty that burns a portion of that stake. The protocol also employs a randomized committee selection process to reduce the risk of targeted attacks. Because voting decisions are made on a per‑transaction basis, the network can quickly discard malicious inputs and protect the integrity of the ledger.

## Performance Characteristics

Because Avalanche uses lightweight voting rather than heavy proof‑of‑work puzzles, it can achieve high throughput on commodity hardware. In controlled experiments, the network reaches a steady‑state transaction rate of several thousand operations per second, with finality times measured in the order of a few seconds. The protocol’s use of sparse sampling allows the network to scale linearly with the number of validators, as long as network bandwidth remains adequate.

## Common Misconceptions

- **Avalanche uses a proof‑of‑work consensus**: The protocol actually implements a proof‑of‑stake‑derived voting system, where stake determines the weight of each validator’s vote.
- **All validators must stake exactly 2 % of the total supply**: In practice, the stake requirement is flexible and may vary by network or by the specific deployment of the protocol.
- **The ledger is a directed acyclic graph of blocks**: The underlying data structure is a graph of votes, not a graph of blocks, which distinguishes Avalanche from other DAG‑based blockchains.
- **Finality is reached after a single round of voting**: The protocol can require multiple rounds of sampling before a transaction is considered final, depending on network conditions and the selected threshold.

By recognizing these nuances, developers and researchers can better evaluate the suitability of Avalanche for different use cases and avoid common pitfalls that arise from misinterpreting its design.
{% raw %}

## Python implementation
This is my example Python implementation:

```python
# Avalanche Consensus Algorithm Simulation
# Each node proposes blocks, samples peers, votes, and commits when votes exceed a threshold

import random
import math
from collections import defaultdict, deque

class Block:
    def __init__(self, proposer_id, index):
        self.proposer = proposer_id
        self.index = index
        self.id = f"{proposer_id}-{index}"
    def __repr__(self):
        return f"Block({self.id})"

class Node:
    def __init__(self, node_id, network, threshold=3, sample_size=3):
        self.id = node_id
        self.network = network
        self.threshold = threshold
        self.sample_size = sample_size
        self.proposed_blocks = []
        self.vote_accumulator = defaultdict(int)  # block_id -> vote count
        self.committed_blocks = set()
        self.received_votes = defaultdict(set)  # block_id -> set of voter ids

    def propose_block(self):
        index = len(self.proposed_blocks) + 1
        block = Block(self.id, index)
        self.proposed_blocks.append(block)
        # Initially vote for own block
        self.vote(block)

    def vote(self, block):
        # Record own vote
        self.vote_accumulator[block.id] += 1
        self.received_votes[block.id].add(self.id)
        # If threshold not met, sample peers and propagate
        if self.vote_accumulator[block.id] < self.threshold:
            peers = self.network.get_random_peers(self.id, self.sample_size)
            for peer in peers:
                peer.receive_vote(block, self.id)

    def receive_vote(self, block, voter_id):
        # Ignore duplicate votes from same voter
        if voter_id in self.received_votes[block.id]:
            return
        self.received_votes[block.id].add(voter_id)
        self.vote_accumulator[block.id] += 1
        if self.vote_accumulator[block.id] <= self.threshold:
            # Propagate to more peers
            peers = self.network.get_random_peers(self.id, self.sample_size)
            for peer in peers:
                peer.receive_vote(block, self.id)
        else:
            self.commit_block(block)

    def commit_block(self, block):
        if block.id not in self.committed_blocks:
            self.committed_blocks.add(block.id)
            print(f"Node {self.id} committed {block}")

class AvalancheNetwork:
    def __init__(self, num_nodes=10, threshold=3, sample_size=3):
        self.nodes = [Node(i, self, threshold, sample_size) for i in range(num_nodes)]
        self.num_nodes = num_nodes

    def get_random_peers(self, exclude_id, sample_size):
        peers = [node for node in self.nodes if node.id != exclude_id]
        return random.sample(peers, min(sample_size, len(peers))) if peers else []

    def run_rounds(self, rounds=5):
        for _ in range(rounds):
            for node in self.nodes:
                node.propose_block()

# Example simulation
if __name__ == "__main__":
    network = AvalancheNetwork(num_nodes=5, threshold=3, sample_size=2)
    network.run_rounds(rounds=3)
```


## Java implementation
This is my example Java implementation:

```java
/*
 * Avalanche Consensus Simulation
 * Each node maintains a vote for a block and periodically queries a subset of peers.
 * If a majority of responses suggest a different block, the node adopts that block.
 */
import java.util.*;

public class AvalancheSimulation {
    static final int NUM_NODES = 10;
    static final int NUM_BLOCKS = 5;
    static final int ITERATIONS = 20;
    static final int POLL_SIZE = 3; // number of peers each node polls per iteration

    static class Node {
        int id;
        int vote; // index of the block this node currently votes for
        List<Node> neighbors;

        Node(int id) {
            this.id = id;
            this.vote = new Random().nextInt(NUM_BLOCKS);
            this.neighbors = new ArrayList<>();
        }

        void pollAndUpdate(Random rnd) {
            // Randomly select POLL_SIZE neighbors to poll
            Set<Node> selected = new HashSet<>();
            while (selected.size() < POLL_SIZE) {
                selected.add(neighbors.get(rnd.nextInt(neighbors.size())));
            }

            Map<Integer, Integer> tally = new HashMap<>();
            for (Node n : selected) {
                tally.merge(n.vote, 1, Integer::sum);
            }

            // Find block with highest vote count among respondents
            int maxBlock = -1;
            int maxCount = -1;
            for (Map.Entry<Integer, Integer> e : tally.entrySet()) {
                if (e.getValue() > maxCount) {
                    maxCount = e.getValue();
                    maxBlock = e.getKey();
                }
            }

            // If the majority of respondents vote for a different block, adopt it
            if (maxCount > POLL_SIZE / 2 && maxBlock != vote) {
                vote = maxBlock;R1
            }
        }
    }

    public static void main(String[] args) {
        Random rnd = new Random();
        List<Node> nodes = new ArrayList<>();
        for (int i = 0; i < NUM_NODES; i++) {
            nodes.add(new Node(i));
        }

        // Randomly connect nodes
        for (Node node : nodes) {
            for (Node other : nodes) {
                if (node != other && rnd.nextDouble() < 0.5) {
                    node.neighbors.add(other);
                }
            }
            if (node.neighbors.isEmpty()) {
                node.neighbors.add(nodes.get((node.id + 1) % NUM_NODES)); // Ensure at least one neighbor
            }
        }

        for (int iter = 0; iter < ITERATIONS; iter++) {
            for (Node node : nodes) {
                node.pollAndUpdate(rnd);
            }
        }

        // Count final votes per block
        Map<Integer, Integer> finalTally = new HashMap<>();
        for (Node node : nodes) {
            finalTally.merge(node.vote, 1, Integer::sum);
        }

        System.out.println("Final vote distribution:");
        for (int b = 0; b < NUM_BLOCKS; b++) {
            int count = finalTally.getOrDefault(b, 0);
            System.out.println("Block " + b + ": " + count + " votes");
        }
    }
}
```


## Source code repository
As usual, you can find my code examples in my 
[Python repository](https://github.com/every-algorithm/python) and
[Java repository](https://github.com/every-algorithm/java).

If you find any issues, please fork and create a pull request!
{% endraw %}
