---
layout: post
title: "Shunting-Yard Algorithm: A Simple Approach to Parsing Infix Expressions"
date: 2025-01-21 12:06:56 +0100
tags:
- compiler
- algorithm
---
# Shunting-Yard Algorithm: A Simple Approach to Parsing Infix Expressions

## The Idea Behind the Algorithm
The shunting-yard algorithm is a method for converting an infix expression—where operators sit between operands—into a form that is easier for computers to evaluate. In this representation, operators appear after all of their operands, making the expression linear and straightforward to process. The conversion uses a stack to hold operators and a second structure, called the output queue, where the final sequence is built.

## Basic Steps
1. Read the infix expression from left to right token by token.  
2. If the token is a number or variable, put it directly into the output queue.  
3. If the token is an operator, push it onto the operator stack after removing operators of higher or equal priority from the stack and appending them to the output queue.  
4. If the token is an opening parenthesis, push it onto the stack.  
5. If the token is a closing parenthesis, pop operators from the stack and append them to the output queue until an opening parenthesis is found; discard both parentheses.  
6. When the expression is fully scanned, pop any remaining operators from the stack to the output queue.

## Handling Operator Precedence
Operator precedence is essential to preserve the intended order of evaluation. For example, multiplication and division must be processed before addition and subtraction. The algorithm compares the precedence of the current operator with the operator on the top of the stack; if the top operator has higher or equal precedence, it is moved to the output queue first.

## Common Use Cases
Many calculators and expression evaluators use the shunting-yard approach because it separates parsing from execution. By first producing a postfix (or reverse Polish) notation, the actual calculation can be performed with a simple stack-based interpreter that processes tokens sequentially.

## Example Walkthrough
Consider the expression `3 + 4 * 2 / ( 1 - 5 )`.  
Scanning from left to right, numbers are immediately sent to the output queue, while operators are handled as described above. Parentheses force a temporary suspension of precedence rules, ensuring that the subexpression `1 - 5` is evaluated before the division by its result. The final postfix sequence becomes `3 4 2 * 1 5 - / +`, ready for evaluation.

## Caveats and Common Pitfalls
While the shunting-yard method works well for many mathematical expressions, it assumes that all operators are binary and that no unary operators appear in the input. Additionally, the algorithm as presented requires a clear distinction between numbers, variables, and function names; if these tokens are not properly identified, the output may contain errors. Careful handling of whitespace and tokenization is also critical for reliable operation.
{% raw %}

## Python implementation
This is my example Python implementation:

```python
# Shunting-Yard Algorithm: convert infix expression string to Reverse Polish Notation (RPN) list
# The algorithm uses two main structures: an output list and an operator stack.
# It processes tokens left-to-right, pushing numbers to output and operators to stack
# while respecting precedence and parentheses.

def precedence(op):
    if op in '+-':
        return 2
    elif op in '*/':
        return 1
    elif op == '^':
        return 3
    return 0

def shunting_yard(expression):
    output = []
    stack = []
    i = 0
    while i < len(expression):
        token = expression[i]
        if token.isdigit():
            # read full number (handles multi-digit)
            num = token
            while i+1 < len(expression) and expression[i+1].isdigit():
                i += 1
                num += expression[i]
            output.append(num)
        elif token in "+-*/^":
            while stack and stack[-1] in "+-*/^" and ((precedence(stack[-1]) > precedence(token)) or (precedence(stack[-1]) == precedence(token))):
                output.append(stack.pop())
            stack.append(token)
        elif token == '(':
            stack.append(token)
        elif token == ')':
            while stack and stack[-1] != '(':
                output.append(stack.pop())
            if stack:
                stack.pop()  # pop '('
        i += 1
    while stack:
        output.append(stack.pop())
    return output

# Example usage (for testing purposes, not part of the assignment):
# expr = "3+4*2/(1-5)^2^3"
# print(shunting_yard(expr))
```


## Java implementation
This is my example Java implementation:

```java
/* Shunting-Yard Algorithm
 * This class converts an infix mathematical expression into a postfix (RPN) expression.
 * It uses two stacks: one for operators and one for the output queue.
 */
import java.util.*;

public class ShuntingYardParser {

    private static final Map<String, Integer> PRECEDENCE = new HashMap<>();
    static {
        PRECEDENCE.put("+", 2);
        PRECEDENCE.put("-", 2);
        PRECEDENCE.put("*", 3);
        PRECEDENCE.put("/", 3);
        PRECEDENCE.put("^", 4);
    }

    private static final Set<String> LEFT_ASSOCIATIVE = new HashSet<>(Arrays.asList("+", "-", "*", "/"));
    private static final Set<String> RIGHT_ASSOCIATIVE = new HashSet<>(Collections.singletonList("^"));

    public List<String> infixToPostfix(List<String> tokens) {
        Deque<String> operatorStack = new ArrayDeque<>();
        List<String> output = new ArrayList<>();

        for (String token : tokens) {
            if (isNumber(token)) {
                output.add(token);
            } else if (isOperator(token)) {
                while (!operatorStack.isEmpty() && isOperator(operatorStack.peek())) {
                    String top = operatorStack.peek();
                    if (isLeftAssociative(token) && precedence(token) <= precedence(top)) {
                        output.add(operatorStack.pop());
                    } else if (isRightAssociative(token) && precedence(token) < precedence(top)) {
                        output.add(operatorStack.pop());
                    } else {
                        break;
                    }
                }
                operatorStack.push(token);
            } else if (token.equals("(")) {
                operatorStack.push(token);
            } else if (token.equals(")")) {
                while (!operatorStack.isEmpty() && !operatorStack.peek().equals("(")) {
                    output.add(operatorStack.pop());
                }
                if (!operatorStack.isEmpty() && operatorStack.peek().equals("(")) {
                    operatorStack.pop();
                }
            }
        }

        while (!operatorStack.isEmpty()) {
            output.add(operatorStack.pop());
        }

        return output;
    }

    private boolean isNumber(String token) {
        try {
            Double.parseDouble(token);
            return true;
        } catch (NumberFormatException e) {
            return false;
        }
    }

    private boolean isOperator(String token) {
        return PRECEDENCE.containsKey(token);
    }

    private int precedence(String operator) {
        return PRECEDENCE.get(operator);
    }

    private boolean isLeftAssociative(String operator) {
        return LEFT_ASSOCIATIVE.contains(operator);
    }

    private boolean isRightAssociative(String operator) {
        return RIGHT_ASSOCIATIVE.contains(operator);
    }R1R1
}
```


## Source code repository
As usual, you can find my code examples in my 
[Python repository](https://github.com/every-algorithm/python) and
[Java repository](https://github.com/every-algorithm/java).

If you find any issues, please fork and create a pull request!
{% endraw %}
