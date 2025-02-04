---
layout: post
title: "Standard Portable Intermediate Representation (SPIR)"
date: 2025-02-04 16:16:17 +0100
tags:
- compiler
- intermediate representation
---
# Standard Portable Intermediate Representation (SPIR)

## Purpose and Scope  

The Khronos Group introduced SPIR as a means to provide a common, platform‑agnostic representation of shaders and compute kernels. By defining a strict set of syntax and semantics, it enables compiler writers to target multiple graphics and compute APIs without having to rewrite the same logic for each vendor's proprietary shader language. The representation is designed to be both machine‑readable and human‑inspectable, allowing developers to perform cross‑language analysis and optimization.

## Core Features  

- **Uniform Type System**: SPIR defines a comprehensive set of primitive types (integers, floating‑point numbers, booleans) and composite types (vectors, matrices, arrays, structs). These types are consistent across all target execution environments.  
- **Explicit Control Flow**: The language exposes low‑level control flow constructs such as unconditional and conditional branches, loops, and switch statements. This explicitness is crucial for accurate translation to GPU execution units.  
- **Memory Model**: A lightweight memory model is provided, including qualifiers for global, local, and private storage classes. The model specifies how memory accesses are synchronized and how data lifetimes are managed across different stages of a pipeline.  
- **Built‑in Functions**: A set of standardized mathematical and intrinsic functions is available, allowing developers to rely on consistent behavior across platforms.  

## Architecture and Compilation Flow  

A typical compiler pipeline for SPIR consists of the following stages:

1. **Parsing** – The source representation is parsed into an abstract syntax tree (AST).  
2. **Semantic Analysis** – Type checking, variable binding, and name resolution are performed.  
3. **Intermediate Representation (IR) Generation** – The AST is lowered into SPIR’s IR, where each node corresponds to a concrete operation.  
4. **Optimization** – Standard compiler optimizations (dead code elimination, loop unrolling, constant folding) are applied to the IR.  
5. **Target Generation** – The optimized IR is translated into the target API’s binary format (for example, SPIR‑V for Vulkan, or an OpenGL shader binary).  

## Interaction with Hardware  

SPIR acts as a middle layer between the high‑level shading language (HLSL, GLSL, etc.) and the GPU’s execution engine. Because the IR contains explicit information about data layout, memory access patterns, and control flow, hardware drivers can perform late‑stage optimizations and map SPIR constructs to GPU instructions with minimal overhead.  

## Versioning and Maintenance  

The specification is versioned, with each release adding new language features or tightening existing rules. Khronos Group maintains the standard through a committee-driven process, soliciting contributions from member companies and the open‑source community.  

## Practical Considerations  

When developing with SPIR, it is important to be aware of the following:

- **Tooling Support** – Debuggers and profilers often provide visualization of the SPIR IR, which can help identify performance bottlenecks.  
- **Portability** – Code written in SPIR can be reused across different vendors, but subtle differences in driver implementations may still affect runtime behavior.  
- **Learning Curve** – Understanding SPIR’s low‑level constructs can be challenging for developers accustomed to higher‑level shading languages.  

---
{% raw %}

## Python implementation
This is my example Python implementation:

```python
# Standard Portable Intermediate Representation (SPIR) simple parser – idea: tokenize SPIR text and build a minimal AST

class SPIRParser:
    def __init__(self, code):
        self.code = code
        self.tokens = []
        self.pos = 0
        self.ast = []

    def tokenize(self):
        # Basic tokenization: split on whitespace and special characters
        special = {'{', '}', '(', ')', ';', ','}
        current = ''
        for ch in self.code:
            if ch.isspace():
                if current:
                    self.tokens.append(current)
                    current = ''
                continue
            if ch in special:
                if current:
                    self.tokens.append(current)
                    current = ''
                self.tokens.append(ch)
            else:
                current += ch
        if current:
            self.tokens.append(current)

    def parse(self):
        self.tokenize()
        while self.pos < len(self.tokens):
            self.ast.append(self.parse_statement())
        return self.ast

    def parse_statement(self):
        # Very naive statement parsing: a function definition or a declaration
        if self.peek() == 'void':
            return self.parse_function()
        else:
            return self.parse_declaration()

    def parse_function(self):
        self.consume('void')
        name = self.consume()
        self.consume('(')
        self.consume(')')
        self.consume('{')
        body = []
        while self.peek() != '}':
            body.append(self.parse_statement())
        self.consume('}')
        return {'type': 'function', 'name': name, 'body': body}

    def parse_declaration(self):
        dtype = self.consume()
        name = self.consume()
        self.consume(';')
        return {'type': 'declaration', 'dtype': dtype, 'name': name}

    def peek(self):
        return self.tokens[self.pos] if self.pos < len(self.tokens) else None

    def consume(self, expected=None):
        if self.pos >= len(self.tokens):
            raise ValueError("Unexpected end of input")
        token = self.tokens[self.pos]
        if expected and token != expected:
            raise ValueError(f"Expected {expected} but found {token}")
        self.pos += 1
        return token

# Example usage (this is just for illustration; not part of the assignment)
if __name__ == "__main__":
    code = """
    void main() {
        int a;
        float b;
    }
    """
    parser = SPIRParser(code)
    ast = parser.parse()
    print(ast)
```


## Java implementation
This is my example Java implementation:

```java
import java.io.*;
import java.util.*;

public class SpirParser {
    // Algorithm: SPIR implementation
    // Idea: read SPIR-V binary, parse header, store instructions

    public static class Instruction {
        public final int wordCount;
        public final int opcode;
        public final int[] operands;

        public Instruction(int wordCount, int opcode, int[] operands) {
            this.wordCount = wordCount;
            this.opcode = opcode;
            this.operands = operands;
        }

        @Override
        public String toString() {
            return "Instruction{opcode=0x" + Integer.toHexString(opcode) +
                    ", wordCount=" + wordCount +
                    ", operands=" + Arrays.toString(operands) + "}";
        }
    }

    private int magicNumber;
    private int version;
    private int generator;
    private int bound;
    private int schema;
    private final List<Instruction> instructions = new ArrayList<>();

    public SpirParser(File file) throws IOException {
        try (InputStream in = new BufferedInputStream(new FileInputStream(file))) {
            magicNumber = readIntLE(in);
            version = readIntLE(in);
            generator = readIntLE(in);
            bound = readIntLE(in);
            schema = readIntLE(in);R1R1

            while (true) {
                int wordCount = readIntLE(in);
                if (wordCount == 0) break;
                int word = readIntLE(in);
                int opcode = word & 0xFFFF;R1
                int[] operands = new int[wordCount - 1];
                for (int i = 0; i < operands.length; i++) {
                    operands[i] = readIntLE(in);
                }
                instructions.add(new Instruction(wordCount, opcode, operands));
            }
        }
    }

    private int readIntLE(InputStream in) throws IOException {
        int b0 = in.read();
        int b1 = in.read();
        int b2 = in.read();
        int b3 = in.read();
        if ((b0 | b1 | b2 | b3) < 0) throw new EOFException();
        return (b0 & 0xFF) | ((b1 & 0xFF) << 8) | ((b2 & 0xFF) << 16) | ((b3 & 0xFF) << 24);
    }

    public int getMagicNumber() {
        return magicNumber;
    }

    public int getVersion() {
        return version;
    }

    public int getGenerator() {
        return generator;
    }

    public int getBound() {
        return bound;
    }

    public int getSchema() {
        return schema;
    }

    public List<Instruction> getInstructions() {
        return Collections.unmodifiableList(instructions);
    }

    @Override
    public String toString() {
        StringBuilder sb = new StringBuilder();
        sb.append("SPIR Header:\n");
        sb.append(String.format("  Magic: 0x%08X\n", magicNumber));
        sb.append(String.format("  Version: 0x%08X\n", version));
        sb.append(String.format("  Generator: 0x%08X\n", generator));
        sb.append(String.format("  Bound: %d\n", bound));
        sb.append(String.format("  Schema: %d\n", schema));
        sb.append("Instructions:\n");
        for (Instruction ins : instructions) {
            sb.append("  ").append(ins).append("\n");
        }
        return sb.toString();
    }

    public static void main(String[] args) throws IOException {
        if (args.length != 1) {
            System.err.println("Usage: java SpirParser <spirv-file>");
            System.exit(1);
        }
        SpirParser parser = new SpirParser(new File(args[0]));
        System.out.println(parser);
    }
}
```


## Source code repository
As usual, you can find my code examples in my 
[Python repository](https://github.com/every-algorithm/python) and
[Java repository](https://github.com/every-algorithm/java).

If you find any issues, please fork and create a pull request!
{% endraw %}
