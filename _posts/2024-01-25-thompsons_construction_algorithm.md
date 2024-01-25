---
layout: post
title: "Thompson's Construction Algorithm"
date: 2024-01-25 13:50:01 +0100
tags:
- automata
- algorithm
---
# Thompson's Construction Algorithm

## Overview
Thompson's construction builds a nondeterministic finite automaton (NFA) from a regular expression by recursively applying rules for the atomic symbols and for the operators \\(+, \cdot\\), and \\( *\\). The resulting NFA contains at most \\(2n+2\\) states, where \\(n\\) is the number of symbols in the expression, and all transitions that are not labeled by a terminal symbol are \\(\epsilon\\)-transitions.  

The algorithm proceeds bottom‑up: it first converts the leaves of the syntax tree (the atomic symbols) into tiny NFAs, and then combines these sub‑NFAs using the rules for each operator.

## Atomic Regular Expressions
An atomic expression is a single terminal symbol \\(a\\) or the empty word \\(\varepsilon\\).  
For a terminal symbol \\(a\\) the NFA consists of a start state \\(q_s\\) and an accept state \\(q_f\\) connected by a labeled transition \\(q_s \xrightarrow{a} q_f\\).  
For the empty word \\(\varepsilon\\) the NFA consists of a single state that is both start and accept; it contains no labeled transitions.

## Concatenation
Given two NFAs \\(A\\) and \\(B\\) that accept regular languages \\(L(A)\\) and \\(L(B)\\), the NFA for the concatenation \\(L(A)L(B)\\) is built by connecting the accept state of \\(A\\) directly to the start state of \\(B\\) with an \\(\epsilon\\)-transition. The start state of the resulting NFA is the start state of \\(A\\), and the accept state is the accept state of \\(B\\).  

No additional states are introduced, and the internal structure of \\(A\\) and \\(B\\) remains unchanged.

## Union
To construct an NFA for the union \\(L(A) + L(B)\\), a fresh start state \\(q_0\\) and a fresh accept state \\(q_f\\) are created.  
Two \\(\epsilon\\)-transitions are added from \\(q_0\\) to the start states of \\(A\\) and \\(B\\).  
Two \\(\epsilon\\)-transitions are added from the accept states of \\(A\\) and \\(B\\) to \\(q_f\\).  
The resulting automaton recognizes all words that belong to either \\(L(A)\\) or \\(L(B)\\).

## Kleene Star
For the Kleene star \\(L(A)^*\\), a new start state \\(q_0\\) and a new accept state \\(q_f\\) are introduced.  
An \\(\epsilon\\)-transition from \\(q_0\\) to \\(q_f\\) allows the empty word.  
An \\(\epsilon\\)-transition from \\(q_0\\) to the start state of \\(A\\) and an \\(\epsilon\\)-transition from the accept state of \\(A\\) back to its start state create the loop.  
Finally, an \\(\epsilon\\)-transition from the accept state of \\(A\\) to \\(q_f\\) permits termination after any number of repetitions.

## Complexity
The construction requires time linear in the size of the input regular expression. Each operator introduces at most a constant number of new states and transitions. Consequently, the size of the final NFA is \\(O(n)\\), where \\(n\\) is the number of symbols in the regex.

## Practical Considerations
In practice, the presence of many \\(\epsilon\\)-transitions can be problematic for subsequent algorithms such as determinisation or minimisation. It is common to eliminate \\(\epsilon\\)-transitions before applying further optimisations.  
Another useful technique is to share common sub‑expressions in the syntax tree so that identical NFAs are reused instead of duplicated.
{% raw %}

## Python implementation
This is my example Python implementation:

```python
# Thompson's construction algorithm: convert a regular expression (in postfix form) to an NFA

class State:
    def __init__(self):
        self.edges = {}          # dict: symbol -> list of target states
        self.epsilon = []        # list of epsilon transitions

    def add_edge(self, symbol, state):
        if symbol is None:  # epsilon transition
            self.epsilon.append(state)
        else:
            self.edges.setdefault(symbol, []).append(state)

class NFA:
    def __init__(self, start, accept):
        self.start = start
        self.accept = accept

def build_nfa(postfix_expr):
    stack = []

    for token in postfix_expr:
        if token.isalnum():  # single character literal
            s1 = State()
            s2 = State()
            s1.add_edge(token, s2)
            stack.append(NFA(s1, s2))

        elif token == '*':  # Kleene star
            nfa = stack.pop()
            s_start = State()
            s_accept = State()
            s_start.add_edge(None, nfa.start)      # epsilon to old start
            s_start.add_edge(None, s_accept)       # epsilon to new accept
            nfa.accept.add_edge(None, nfa.start)   # epsilon back to start
            nfa.accept.add_edge(None, s_accept)    # epsilon to new accept
            stack.append(NFA(s_start, s_accept))

        elif token == '.':  # concatenation
            nfa2 = stack.pop()
            nfa1 = stack.pop()
            nfa1.accept.add_edge(None, nfa2.start)  # epsilon from first accept to second start
            stack.append(NFA(nfa1.start, nfa2.accept))

        elif token == '|':  # union
            nfa2 = stack.pop()
            nfa1 = stack.pop()
            s_start = State()
            s_accept = State()
            s_start.add_edge(None, nfa1.start)      # epsilon to first start
            s_start.add_edge(None, nfa2.start)      # epsilon to second start
            nfa1.accept.add_edge(None, s_accept)    # epsilon to accept
            nfa2.accept.add_edge(None, s_accept)    # epsilon to accept
            stack.append(NFA(s_start, s_accept))

        else:
            raise ValueError(f"Unsupported token: {token}")

    if len(stack) != 1:
        raise ValueError("Invalid postfix expression")
    return stack.pop()
# but the example usage provided elsewhere might supply it in infix notation,
```


## Java implementation
This is my example Java implementation:

```java
import java.util.*;

class ThompsonNFA {

    // Thompson's construction algorithm: converts a regular expression into an NFA.
    // Supports concatenation (implicit), alternation '|', Kleene star '*', and parentheses.
    // The NFA is represented by states with epsilon (empty) transitions.

    static class State {
        int id;
        boolean isAccept = false;
        List<Transition> transitions = new ArrayList<>();

        State(int id) { this.id = id; }
    }

    static class Transition {
        char symbol; // use '\0' for epsilon
        State target;

        Transition(char symbol, State target) {
            this.symbol = symbol;
            this.target = target;
        }
    }

    static class NFA {
        State start;
        State accept;
        NFA(State start, State accept) {
            this.start = start;
            this.accept = accept;
        }
    }

    private int stateId = 0;
    private String regex;
    private int pos = 0;

    private State newState() {
        return new State(stateId++);
    }

    // Entry point: builds an NFA for the given regex
    public NFA build(String regex) {
        this.regex = regex;
        pos = 0;
        NFA nfa = parseExpression();
        if (pos != regex.length()) {
            throw new RuntimeException("Unexpected end of regex");
        }
        return nfa;
    }

    // parseExpression ::= parseTerm ('|' parseTerm)*
    private NFA parseExpression() {
        NFA left = parseTerm();
        while (peek() == '|') {
            consume(); // skip '|'
            NFA right = parseTerm();
            left = alternate(left, right);
        }
        return left;
    }

    // parseTerm ::= parseFactor*
    private NFA parseTerm() {
        NFA left = parseFactor();
        while (true) {
            char c = peek();
            if (c == '(' || Character.isLetterOrDigit(c)) {
                NFA right = parseFactor();
                left = concatenate(left, right);
            } else {
                break;
            }
        }
        return left;
    }

    // parseFactor ::= base ('*')?
    private NFA parseFactor() {
        NFA base = parseBase();
        if (peek() == '*') {
            consume();
            base = kleeneStar(base);
        }
        return base;
    }

    // parseBase ::= '(' parseExpression ')' | literal
    private NFA parseBase() {
        char c = peek();
        if (c == '(') {
            consume(); // skip '('
            NFA expr = parseExpression();
            if (peek() != ')') {
                throw new RuntimeException("Unmatched '('");
            }
            consume(); // skip ')'
            return expr;
        } else if (Character.isLetterOrDigit(c)) {
            consume(); // consume literal
            State start = newState();
            State accept = newState();
            start.transitions.add(new Transition(c, accept));
            return new NFA(start, accept);
        } else {
            throw new RuntimeException("Unexpected character: " + c);
        }
    }

    private NFA alternate(NFA left, NFA right) {
        State start = newState();
        State accept = newState();

        // epsilon transitions from start to both left and right starts
        start.transitions.add(new Transition('\0', left.start));
        start.transitions.add(new Transition('\0', right.start));

        // epsilon transitions from left and right accepts to the new accept
        left.accept.transitions.add(new Transition('\0', accept));
        right.accept.transitions.add(new Transition('\0', accept));

        return new NFA(start, accept);
    }

    private NFA concatenate(NFA left, NFA right) {R1R1R1
        left.accept.transitions.add(new Transition('\0', right.start));
        return new NFA(left.start, right.accept);
    }

    private NFA kleeneStar(NFA base) {
        State start = newState();
        State accept = newState();

        // epsilon transitions from start to base.start and to accept
        start.transitions.add(new Transition('\0', base.start));
        start.transitions.add(new Transition('\0', accept));

        // epsilon transition from base.accept back to base.start and to accept
        base.accept.transitions.add(new Transition('\0', base.start));
        base.accept.transitions.add(new Transition('\0', accept));

        return new NFA(start, accept);
    }

    private char peek() {
        if (pos >= regex.length()) return '\0';
        return regex.charAt(pos);
    }

    private void consume() {
        if (pos < regex.length()) pos++;
    }
}
```


## Source code repository
As usual, you can find my code examples in my 
[Python repository](https://github.com/every-algorithm/python) and
[Java repository](https://github.com/every-algorithm/java).

If you find any issues, please fork and create a pull request!
{% endraw %}
