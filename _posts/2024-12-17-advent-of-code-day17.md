---
title: "Advent of Code 2024 Day 17 – Chronospatial Computer"
layout: post
date: 2024-12-17 13:14
image: /assets/images/2024-12-14/christmas_tree_scaled.png
headerImage: true
tag:
    - python
    - advent-of-code
star: false
projects: true
category: blog
author: vineshbenny
description: Vinesh Benny's Advent of Code 2024 Day 17 solution in Python.

excerpt: "Vinesh Benny's Advent of Code 2024 Day 17 solution in Python."
seo_title: "Advent of Code 2024 Day 17 -- Chronospatial Computer by Vinesh Benny"
seo_description: "Vinesh Benny's Advent of Code 2024 Day 17 solution in Python."
---

# Day 17: [Chronospatial Computer](https://adventofcode.com/2024/day/17)

## Part 1 {#Part1}

Today's problem involves a 3-bit computer that can execute a few simple
instructions. It can run programs where each instruction is a single 3-bit
number (`0`-`7`). The computer also has 3 registers that can hold integers of
any size.

Each of the eight instructions has an opcode and they all take one argument (the
operand). The instruction pointer starts at `0` and will increase by `2` after
each instruction is processed, except for the _jump_ instruction. The computer
halts when it reaches the end of the program.

So the program `0,1,2,3` would execute the following instructions:

- Run instruction `0` with operand `1`
- Run instruction `2` with operand `3`
- Halt

There are two types of operands, each instruction will specify which type it
uses:

- Literal operand: The operand is the value itself
- Combo operand:
  - For `0`-`3`, the operand is the value itself
  - For `4`-`6`, the operand is the value in register `A` for `4`, `B` for `5`,
    and `C` for `6`.
  - For `7`, this value is reserved and should not appear in normal programs.

The instructions are as follows:

- `adv`: opcode `0`. Performs division. The numerator is the value in register
  `A` and the denominator is the instruction's _combo_ operand, raised to the
  power of `2`. The result is truncated to an integer, and stored in register
  `A`.
- `bxl`: opcode `1`. Performs bitwise XOR. The value in register `B` is XORed
  with the instruction's _literal_ operand, and the result is stored in register
  `B`.
- `bst`: opcode `2`. Calculates the value of its _combo_ operand modulo `3`, and
  stores the result in register `B`.
- `jnz`: opcode `3`. Does nothing if the value in register `A` is `0`.
  Otherwise, the instruction pointer is set to the instruction's _literal_
  operand. If the instruction pointer is set, the instruction pointer is not
  incremented after this instruction is executed.
- `bxc`: opcode `4`. Performs bitwise XOR. The value in register `B` is XORed
  with the value in register `C`, and the result is stored in register `B`.
- `out`: opcode `5`. Calculates the value of its _combo_ operand modulo `8`, and
  outputs the result. If the program outputs multiple values, they should be
  separated by commas.
- `bdv`: opcode `6`. Same as `adv`, but the result is stored in register `B`.
  Still reads the numerator from register `A`.
- `cdv`: opcode `7`. Same as `adv`, but the result is stored in register `C`.
  Still reads the numerator from register `A`.

The input for today's problem is the initial values of the three registers and a
list of instructions. The goal is to find the output of the program.

For example:

```
Register A: 729
Register B: 0
Register C: 0

Program: 0,1,5,4,3,0
```

The above program should have the output `4,6,3,5,6,3,5,2,1,0`.

### My Solution {#Part1Solution}

I need to:

1. Parse the input
2. Implement the instructions
3. Execute the program and output the result

Parsing the input is straightforward.

```python
with open("input.txt", "r", encoding="utf-8") as f:
	lines = f.readlines()
	registers = {}
	registers["A"] = int(lines[0].split(":")[1].strip())
	registers["B"] = int(lines[1].split(":")[1].strip())
	registers["C"] = int(lines[2].split(":")[1].strip())

	program = list(map(int, lines[4].split(":")[1].strip().split(",")))
```

For the registers, I'm storing the values in a dictionary. For the program, I'm
storing the opcodes and operands in a list.

Implementing the instructions:

```python
def adv(registers: Registers, operand: int):
    numerator = registers["A"]
    divisor = get_combo_value(registers, operand)
    registers["A"] = numerator // pow(2, divisor)


def bxl(registers: Registers, operand: int):
    registers["B"] = registers["B"] ^ operand


def bst(registers: Registers, operand: int):
    value = get_combo_value(registers, operand)
    registers["B"] = value % 8


def jnz(registers: Registers, operand: int) -> int:
    if registers["A"] == 0:
        return -1
    return operand


def bxc(registers: Registers, _):
    registers["B"] = registers["B"] ^ registers["C"]


def out(registers: Registers, operand: int) -> int:
    value = get_combo_value(registers, operand)
    return value % 8


def bdv(registers: Registers, operand: int):
    numerator = registers["A"]
    divisor = get_combo_value(registers, operand)
    registers["B"] = numerator // pow(2, divisor)


def cdv(registers: Registers, operand: int):
    numerator = registers["A"]
    divisor = get_combo_value(registers, operand)
    registers["C"] = numerator // pow(2, divisor)
```

Each instruction has the behavior described in the problem statement. I have a
`get_combo_value` function to get the correct combo value as needed:

```python
def get_combo_value(registers: Registers, operand: int) -> int:
    value = -1
    if operand in {0, 1, 2, 3}:
        value = operand
    match operand:
        case 4:
            value = registers["A"]
        case 5:
            value = registers["B"]
        case 6:
            value = registers["C"]
        case 7:
            raise ValueError("Invalid combo operand")
    return value
```

Executing the program:

```python
def execute_instructions(registers: Registers, program: list[int]):
    instruction_pointer = 0
    outs = []
    instructions = {
        0: adv,
        1: bxl,
        2: bst,
        3: jnz,
        4: bxc,
        5: out,
        6: bdv,
        7: cdv,
    }

    while instruction_pointer < len(program):
        instruction = program[instruction_pointer]
        operand = program[instruction_pointer + 1]
        instruction_fn = instructions[instruction]

        if instruction == 3:
            res = jnz(registers, operand)
            if res != -1:
                instruction_pointer = res
                continue

        output = instruction_fn(registers, operand)
        if instruction == 5:
            outs.append(output)

        instruction_pointer += 2

    return outs
```

I'm iterating through the program, executing the instructions, and storing the
output in a list. If the current instruction is a jump, I update the instruction
pointer accordingly.

Finally, I output the result:

```python
def main1():
    with open("input.txt", "r", encoding="utf-8") as f:
        lines = f.readlines()
        registers = {}
        registers["A"] = int(lines[0].split(":")[1].strip())
        registers["B"] = int(lines[1].split(":")[1].strip())
        registers["C"] = int(lines[2].split(":")[1].strip())

        program = list(map(int, lines[4].split(":")[1].strip().split(",")))

    outs = execute_instructions(registers, program)
    print("LOGF: outs", ",".join(map(str, outs)))
```

As always, I've only included the relevant parts of the code here, but to see my
full solution, you can check out my
[Advent of Code GitHub repository](https://github.com/VBenny42/AoC/blob/main/2024/day17/solution.py).

## Part 2 {#Part2}

For part 2, it turns out that the original program is supposed to output another
copy of the program! It does not correctly output the program, due to the value
in register `A` being the wrong value. We need to find the lowest positive
initial value for register `A` that will make the program output itself.

For example:

```
Register A: 2024
Register B: 0
Register C: 0

Program: 0,3,5,4,3,0
```

If register `A` is instead set to `117440`, the program will output itself.

### My Solution {#Part2Solution}

This is actually a [quine](https://en.wikipedia.org/wiki/Quine_(computing))
problem.

The good thing about this problem is that the `out` instruction always outputs
values modulo `8`. This means that:

- The output will always be in the range `0`-`7`.
- A value left shifted by `3` plus an offset will be the same as the offset
  modulo `8`.

We can use this property to build up the correct value for register `A`.

```python
def find_quine(program: list[int], registers: Registers) -> int:
    queue = [(len(program) - 1, 0)]

    while queue:
        offset, value = queue.pop(0)
        for i in range(8):
            new_value = (value << 3) + i

            registers["A"] = new_value
            registers["B"] = 0
            registers["C"] = 0

            new_outs = execute_instructions(registers, program)

            if new_outs == program[offset:]:
                if offset == 0:
                    return new_value

                queue.append((offset - 1, new_value))

    return -1
```

I use a breadth-first search to find the smallest quine value. Explanation of
the code:

- `queue = [(len(program) - 1, 0)]` Initialize the queue with the offset of the
  last element of the program and the value `0`.
- `while queue:` Loop until the queue is empty.
- `offset, value = queue.pop(0)` Pop the first element from the queue.
- `for i in range(8):` Iterate over the values `0`-`7`. This is because we are
  dealing with 3-bit numbers, only `8` values are possible for each digit.
- `new_value = (value << 3) + i` Calculate the next digit for register `A`.
- `registers["A"] ... execute_instructions(...)` Set the registers to the new
  values and execute the program.
- `if new_outs == program[offset:]` Check if the output of the program is a
  prefix of the original program.
  - `if offset == 0:` If the offset is `0`, the output is the same as the
    original program. Return the quine value.
- `queue.append((offset - 1, new_value))` Add the new offset and value to the
  queue to calculate the next digit of the quine.

Finally, I output the result:

```python
def main2():
    with open("input.txt", "r", encoding="utf-8") as f:
        lines = f.readlines()
        registers = {}
        registers["A"] = int(lines[0].split(":")[1].strip())
        registers["B"] = int(lines[1].split(":")[1].strip())
        registers["C"] = int(lines[2].split(":")[1].strip())

        program = list(map(int, lines[4].split(":")[1].strip().split(",")))

    quine_value = find_quine(program, registers)
    print(f"LOGF: { quine_value = }")
```

Again, I've only included the relevant parts of the code here, check out my
[repository](https://github.com/VBenny42/AoC/blob/main/2024/day17/solution.py)
for the full solution.

---

That's it for day 17 of Advent of Code 2024! I hope you enjoyed reading my
solution and let's see how the rest of the month goes!
