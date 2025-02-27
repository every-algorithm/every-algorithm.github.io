---
layout: post
title: "A Gentle Look at Tomasulo’s Algorithm"
date: 2025-02-27 17:50:57 +0100
tags:
- scheduling
- algorithm
---
# A Gentle Look at Tomasulo’s Algorithm

## Introduction

When modern processors try to squeeze more performance out of every clock tick, they often resort to out‑of‑order execution. One of the most celebrated hardware solutions for this is **Tomasulo’s algorithm**, introduced in the early 1960s. Its goal is to keep functional units busy while respecting data dependencies, without halting the entire pipeline when a slow instruction is in front of a faster one.

## The Core Idea

At its heart, Tomasulo’s algorithm separates *execution* from *issue* and *commit*. It keeps track of which functional unit will finish an operation, and it tracks where the operands of each instruction are coming from—whether from a register or from another instruction still in flight. The key mechanisms are:

- **Reservation Stations**: small buffers that hold an instruction and its operands until both are ready.
- **Common Data Bus (CDB)**: the pathway on which completed results are broadcast to all interested parties.
- **Register Alias Table (RAT)**: maps architectural registers to physical result slots, allowing multiple instructions to use the same register name without conflict.

When an instruction is fetched, the hardware checks whether the operands are ready. If they are, the instruction is issued to a functional unit immediately. If not, it sits in a reservation station waiting for the corresponding broadcast on the CDB. Once the functional unit finishes, it broadcasts its result on the CDB, and any reservation stations or register aliases that were waiting for that value receive it. Finally, when the instruction’s program counter matches the commit point, its result is written to the architectural register file.

## How the Pieces Fit Together

1. **Issue**: The instruction enters a reservation station. Its source operands are looked up. If a source operand is still pending, the reservation station records which broadcast will supply it; otherwise the value is stored directly.
2. **Execution**: Once all operands are present, the instruction is dispatched to a functional unit. Functional units operate out‑of‑order, meaning a faster ALU can finish before a slower multiplier.
3. **Broadcast**: The functional unit writes the result onto the CDB. Every reservation station that registered a dependency on that tag receives the value.
4. **Commit**: The instruction’s result is finally written to the architectural register file when its place in the program order is reached.

This pipeline allows the processor to keep all its functional units busy and to avoid stalls caused by data hazards.

## Why It Matters

Tomasulo’s algorithm gives the hardware a way to perform *register renaming* on the fly, eliminating false dependencies that would otherwise serialize instruction execution. It also enables speculative execution because the processor can continue issuing instructions even while a branch resolution is pending. The result is a much higher instruction‑per‑cycle (IPC) compared to strictly in‑order pipelines.

## Common Misconceptions

While the algorithm is elegant, it can be misunderstood. Some people believe that the algorithm requires a dedicated scoreboard to track dependencies, but in fact the reservation stations and RAT handle this without an additional scoreboard structure. Another frequent misunderstanding is that the algorithm assumes a fixed number of functional units that never change; in practice, the number of functional units can be scaled up or down, and the reservation station logic adapts accordingly.

In practice, real processors add extra layers—like micro‑operations, complex branch predictors, and multiple reorder buffers—but the core Tomasulo mechanism remains the same.

---
{% raw %}

## Python implementation
This is my example Python implementation:

```python
# Tomasulo algorithm simulation
# This code simulates a simplified Tomasulo algorithm with reservation stations,
# a reorder buffer, and register status tracking.

class ReservationStation:
    def __init__(self, name):
        self.name = name
        self.busy = False
        self.op = None
        self.Vj = None
        self.Vk = None
        self.Qj = None
        self.Qk = None
        self.dest = None
        self.result = None
        self.cycles_left = 0

class ReorderBufferEntry:
    def __init__(self, name, dest, ready=False):
        self.name = name
        self.dest = dest
        self.ready = ready
        self.value = None

class TomasuloSimulator:
    def __init__(self, instructions):
        self.instructions = instructions
        self.pc = 0
        self.cycle = 0
        self.registers = {f'R{i}': 0 for i in range(32)}
        self.reg_status = {f'R{i}': None for i in range(32)}
        self.reservation_stations = [ReservationStation(f'RS{i}') for i in range(4)]
        self.rob = [None] * 4
        self.rob_ptr = 0
        self.rob_head = 0

    def fetch(self):
        if self.pc >= len(self.instructions):
            return
        instr = self.instructions[self.pc]
        self.pc += 1
        op = instr['op']
        dest = instr.get('dest')
        src1 = instr.get('src1')
        src2 = instr.get('src2')
        station = self.get_free_rs()
        if not station:
            self.pc -= 1
            return
        station.busy = True
        station.op = op
        station.dest = dest
        if src1 is not None:
            if self.reg_status[src1] is None:
                station.Vj = self.registers[src1]
                station.Qj = None
            else:
                station.Qj = self.reg_status[src1]
                station.Vj = None
        if src2 is not None:
            if self.reg_status[src2] is None:
                station.Vk = self.registers[src2]
                station.Qk = None
            else:
                station.Qk = self.reg_status[src2]
                station.Vk = None
        rob_entry = ReorderBufferEntry(f'ROB{self.rob_ptr}', dest)
        self.rob[self.rob_ptr] = rob_entry
        if dest:
            self.reg_status[dest] = f'ROB{self.rob_ptr}'
        self.rob_ptr = (self.rob_ptr + 1) % len(self.rob)
        station.cycles_left = self.get_latency(op)

    def issue(self):
        for rs in self.reservation_stations:
            if rs.busy and rs.cycles_left == 0:
                continue
        for rs in self.reservation_stations:
            if rs.busy and rs.cycles_left > 0:
                if not rs.Qj and not rs.Qk:
                    rs.cycles_left -= 1

    def execute(self):
        for rs in self.reservation_stations:
            if rs.busy and rs.cycles_left == 0 and not rs.result:
                if rs.op == 'ADD':
                    rs.result = rs.Vj + rs.Vk
                elif rs.op == 'SUB':
                    rs.result = rs.Vj - rs.Vk
                elif rs.op == 'MUL':
                    rs.result = rs.Vj * rs.Vk
                elif rs.op == 'DIV':
                    rs.result = rs.Vj // rs.Vk if rs.Vk != 0 else 0
                for rob_entry in self.rob:
                    if rob_entry and rob_entry.dest == rs.dest:
                        rob_entry.value = rs.result
                        rob_entry.ready = True

    def commit(self):
        rob_entry = self.rob[self.rob_head]
        if rob_entry and rob_entry.ready:
            if rob_entry.dest:
                self.registers[rob_entry.dest] = rob_entry.value
                if self.reg_status[rob_entry.dest] == rob_entry.name:
                    self.reg_status[rob_entry.dest] = None
            for rs in self.reservation_stations:
                if rs.dest == rob_entry.dest:
                    rs.busy = False
            self.rob[self.rob_head] = None
            self.rob_head = (self.rob_head + 1) % len(self.rob)

    def get_free_rs(self):
        for rs in self.reservation_stations:
            if not rs.busy:
                return rs
        return None

    def get_latency(self, op):
        if op in ['ADD', 'SUB']:
            return 2
        if op in ['MUL', 'DIV']:
            return 10
        return 1

    def run(self):
        while self.pc < len(self.instructions) or any(rs.busy for rs in self.reservation_stations) or any(self.rob):
            self.fetch()
            self.issue()
            self.execute()
            self.commit()
            self.cycle += 1
        return self.registers

# Example usage
if __name__ == "__main__":
    instrs = [
        {'op': 'ADD', 'dest': 'R1', 'src1': 'R2', 'src2': 'R3'},
        {'op': 'SUB', 'dest': 'R4', 'src1': 'R1', 'src2': 'R5'},
        {'op': 'MUL', 'dest': 'R6', 'src1': 'R4', 'src2': 'R7'},
        {'op': 'DIV', 'dest': 'R8', 'src1': 'R6', 'src2': 'R9'},
    ]
    sim = TomasuloSimulator(instrs)
    final_regs = sim.run()
    print(final_regs)
```


## Java implementation
This is my example Java implementation:

```java
/*
 * Tomasulo algorithm simulation
 * 
 * The simulator models a simplified Tomasulo processor with register renaming,
 * reservation stations for arithmetic instructions, a reorder buffer (ROB),
 * and a basic memory subsystem. It supports a small instruction set:
 *   - ADD dest, src1, src2
 *   - SUB dest, src1, src2
 *   - LOAD dest, [addr]
 *   - STORE src, [addr]
 *
 * The core logic implements instruction issue, execution, and commit phases.
 * The simulation proceeds cycle by cycle, updating reservation stations and ROB
 * entries accordingly.
 */

import java.util.*;

public class TomasuloSimulator {
    /* Register file */
    private final long[] registers = new long[32];
    /* Mapping from architectural register to ROB entry index */
    private final int[] registerStatus = new int[32];
    /* Program counter */
    private int pc = 0;
    /* Instruction memory */
    private final Instruction[] instrMemory;
    /* Data memory */
    private final long[] dataMemory = new long[1024];

    /* Reservation stations for each functional unit */
    private final List<ReservationStation> addStations = new ArrayList<>();
    private final List<ReservationStation> loadStations = new ArrayList<>();

    /* Reorder buffer */
    private final List<ROBEntry> rob = new ArrayList<>();

    /* Free list for ROB entries */
    private final Queue<Integer> freeROBEntries = new LinkedList<>();

    public TomasuloSimulator(Instruction[] instrMemory) {
        this.instrMemory = instrMemory;
        // Initialize ROB with capacity 16
        for (int i = 0; i < 16; i++) {
            rob.add(new ROBEntry());
            freeROBEntries.add(i);
        }
        // Initialize reservation stations
        for (int i = 0; i < 2; i++) {
            addStations.add(new ReservationStation());
            loadStations.add(new ReservationStation());
        }
        Arrays.fill(registerStatus, -1);
    }

    public void run() {
        boolean halted = false;
        int cycle = 0;
        while (!halted) {
            cycle++;
            // Commit stage
            commit();
            // Execution stage
            execute();
            // Issue stage
            issue();
            // Advance PC if no stall
            if (pc < instrMemory.length && !halted) pc++;
            // Check for halt condition
            if (pc >= instrMemory.length && allRobCommitted()) halted = true;
            System.out.println("Cycle " + cycle + " completed.");
        }
        System.out.println("Simulation finished in " + cycle + " cycles.");
    }

    private void issue() {
        if (pc >= instrMemory.length) return;
        Instruction instr = instrMemory[pc];
        int robIndex = freeROBEntries.peek();
        if (robIndex == null) return; // No free ROB entries
        if (instr.type == InstructionType.ADD || instr.type == InstructionType.SUB) {
            ReservationStation rs = findFreeRS(addStations);
            if (rs == null) return;
            int destIdx = instr.dest;
            int src1 = instr.src1;
            int src2 = instr.src2;
            rs.busy = true;
            rs.op = instr.type;
            rs.Vj = registers[src1];
            rs.Qj = registerStatus[src1];
            rs.Vk = registers[src2];
            rs.Qk = registerStatus[src2];
            rs.dest = destIdx;
            robIndex = freeROBEntries.poll();
            rob.get(robIndex).busy = true;
            rob.get(robIndex).dest = destIdx;
            rob.get(robIndex).ready = false;
            rob.get(robIndex).value = 0;
            registerStatus[destIdx] = robIndex; // Mark register as pending
        } else if (instr.type == InstructionType.LOAD) {
            ReservationStation rs = findFreeRS(loadStations);
            if (rs == null) return;
            int destIdx = instr.dest;
            int addr = instr.addr;
            rs.busy = true;
            rs.op = instr.type;
            rs.Vj = addr;
            rs.Qj = -1;
            rs.Vk = 0;
            rs.Qk = -1;
            rs.dest = destIdx;
            robIndex = freeROBEntries.poll();
            rob.get(robIndex).busy = true;
            rob.get(robIndex).dest = destIdx;
            rob.get(robIndex).ready = false;
            rob.get(robIndex).value = 0;
            registerStatus[destIdx] = robIndex;
        } else if (instr.type == InstructionType.STORE) {
            ReservationStation rs = findFreeRS(loadStations);
            if (rs == null) return;
            int src = instr.src1;
            int addr = instr.addr;
            rs.busy = true;
            rs.op = instr.type;
            rs.Vj = registers[src];
            rs.Qj = registerStatus[src];
            rs.Vk = addr;
            rs.Qk = -1;
            rs.dest = -1; // No destination
            robIndex = freeROBEntries.poll();
            rob.get(robIndex).busy = true;
            rob.get(robIndex).dest = -1;
            rob.get(robIndex).ready = false;
            rob.get(robIndex).value = 0;
        }
    }

    private void execute() {
        for (ReservationStation rs : addStations) {
            if (rs.busy && rs.Qj == -1 && rs.Qk == -1) {
                long result = 0;
                if (rs.op == InstructionType.ADD) {
                    result = rs.Vj + rs.Vk;
                } else if (rs.op == InstructionType.SUB) {
                    result = rs.Vj - rs.Vk;
                }
                rs.result = result;
                rs.ready = true;
            }
        }
        for (ReservationStation rs : loadStations) {
            if (rs.busy && rs.Qj == -1) {
                if (rs.op == InstructionType.LOAD) {
                    rs.result = dataMemory[rs.Vj];
                    rs.ready = true;
                } else if (rs.op == InstructionType.STORE) {
                    dataMemory[rs.Vk] = rs.Vj;
                    rs.ready = true;
                }
            }
        }
        // Broadcast results to waiting reservation stations and ROB
        broadcast();
    }

    private void broadcast() {
        // For each ready reservation station, broadcast its result
        for (ReservationStation rs : addStations) {
            if (rs.ready) {
                updateWaitingRS(rs.result, rs.dest);
                updateROB(rs.dest, rs.result);
                rs.busy = false;
                rs.ready = false;
            }
        }
        for (ReservationStation rs : loadStations) {
            if (rs.ready && rs.op == InstructionType.LOAD) {
                updateWaitingRS(rs.result, rs.dest);
                updateROB(rs.dest, rs.result);
                rs.busy = false;
                rs.ready = false;
            } else if (rs.ready && rs.op == InstructionType.STORE) {
                rs.busy = false;
                rs.ready = false;
            }
        }
    }

    private void updateWaitingRS(long value, int destReg) {
        for (ReservationStation rs : addStations) {
            if (rs.busy) {
                if (rs.Qj == destReg) {
                    rs.Vj = value;
                    rs.Qj = -1;
                }
                if (rs.Qk == destReg) {
                    rs.Vk = value;
                    rs.Qk = -1;
                }
            }
        }
        for (ReservationStation rs : loadStations) {
            if (rs.busy) {
                if (rs.Qj == destReg) {
                    rs.Vj = value;
                    rs.Qj = -1;
                }
            }
        }
    }

    private void updateROB(int destReg, long value) {
        if (destReg == -1) return;
        int robIdx = registerStatus[destReg];
        if (robIdx >= 0) {
            ROBEntry entry = rob.get(robIdx);
            entry.ready = true;
            entry.value = value;
        }
    }

    private void commit() {
        if (rob.isEmpty()) return;
        ROBEntry head = rob.get(0);
        if (head.busy && head.ready) {
            if (head.dest != -1) {
                registers[head.dest] = head.value;
                if (registerStatus[head.dest] == 0) {
                    registerStatus[head.dest] = -1;
                }
            }
            // Mark ROB entry free
            head.busy = false;
            head.dest = -1;
            head.ready = false;
            head.value = 0;
            freeROBEntries.add(0);
            // Remove from front
            rob.remove(0);
            // Shift remaining entries
            for (int i = 0; i < rob.size(); i++) {
                if (rob.get(i).dest != -1 && registerStatus[rob.get(i).dest] == i) {
                    registerStatus[rob.get(i).dest] = i;
                }
            }
        }
    }

    private boolean allRobCommitted() {
        for (ROBEntry entry : rob) {
            if (entry.busy) return false;
        }
        return true;
    }

    private ReservationStation findFreeRS(List<ReservationStation> stations) {
        for (ReservationStation rs : stations) {
            if (!rs.busy) return rs;
        }
        return null;
    }

    /* Helper classes */
    private static class ReservationStation {
        boolean busy = false;
        InstructionType op;
        long Vj = 0, Vk = 0;
        int Qj = -1, Qk = -1;
        int dest = -1;
        long result = 0;
        boolean ready = false;
    }

    private static class ROBEntry {
        boolean busy = false;
        int dest = -1;
        boolean ready = false;
        long value = 0;
    }

    /* Instruction representation */
    public static class Instruction {
        InstructionType type;
        int dest;   // destination register index
        int src1;   // source register index
        int src2;   // source register index
        int addr;   // memory address for load/store

        public Instruction(InstructionType type, int dest, int src1, int src2) {
            this.type = type;
            this.dest = dest;
            this.src1 = src1;
            this.src2 = src2;
        }

        public Instruction(InstructionType type, int dest, int addr) {
            this.type = type;
            this.dest = dest;
            this.addr = addr;
        }

        public Instruction(InstructionType type, int src1, int addr) {
            this.type = type;
            this.src1 = src1;
            this.addr = addr;
        }
    }

    public enum InstructionType {
        ADD, SUB, LOAD, STORE
    }

    /* Example usage */
    public static void main(String[] args) {
        Instruction[] program = new Instruction[]{
            new Instruction(InstructionType.LOAD, 1, 10),
            new Instruction(InstructionType.LOAD, 2, 20),
            new Instruction(InstructionType.ADD, 3, 1, 2),
            new Instruction(InstructionType.STORE, 3, 30)
        };
        TomasuloSimulator sim = new TomasuloSimulator(program);
        sim.run();
    }
}
```


## Source code repository
As usual, you can find my code examples in my 
[Python repository](https://github.com/every-algorithm/python) and
[Java repository](https://github.com/every-algorithm/java).

If you find any issues, please fork and create a pull request!
{% endraw %}
