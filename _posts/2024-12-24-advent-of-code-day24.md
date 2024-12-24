---
title: "Advent of Code 2024 Day 24 – Crossed Wires"
layout: post
date: 2024-12-24 14:28
image: /assets/images/2024-12-14/christmas_tree_scaled.png
headerImage: true
tag:
    - python
    - advent-of-code
star: false
projects: true
category: blog
author: vineshbenny
description: Vinesh Benny's Advent of Code 2024 Day 24 solution in Python.

excerpt: "Vinesh Benny's Advent of Code 2024 Day 24 solution in Python."
seo_title: "Advent of Code 2024 Day 24 -- Crossed Wires by Vinesh Benny"
seo_description: "Vinesh Benny's Advent of Code 2024 Day 24 solution in Python."
---

# Day 24: [Crossed Wires](https://adventofcode.com/2024/day/24)

## Part 1 {#Part1}

For today's puzzle, there's a device that's trying to produce a number using
some boolean logic gates. The gates it has access to are `AND`, `OR`, and `XOR`.
The gates will wait until they have all the inputs they need to produce an
output. Each wire can only be the output of one gate, but it can be the input to
multiple gates.

The puzzle input is a list of starting values for some wires and all the gates
in the device. The device ultimately wants to produce a value using all the
wires that start with `z`. The `z00` is the least significant bit, then `z01`,
and so on. For example:

```
x00: 1
x01: 1
x02: 1
y00: 0
y01: 1
y02: 0

x00 AND y00 -> z00
x01 XOR y01 -> z01
x02 OR y02 -> z02
```

In this example, `z` would be the binary number `100`, which is `4` in decimal
notation.

Our task is to find the decimal value of `z` for the given input.

### My Solution {#Part1Solution}

My strategy:

- Parse the input to get the starting values of the wires and the gates, storing
  the wires in a dictionary.
- Create a function to evaluate what the output of a gate would be given the
  inputs.
- For each gate, use the function to calculate the output and store it in the
  dictionary.
- Find all the wires that start with `z` and calculate the decimal value of `z`.

```python
def main1():
    with open("input.txt", "r", encoding="utf-8") as f:
        inputs, gates = f.read().split("\n\n")

    inputs = inputs.splitlines()
    inputs = {input.split(": ")[0]: int(input.split(": ")[1]) for input in inputs}
    gates = gates.splitlines()

    while gates:
        gate = gates.pop(0)
        if not run_gate(gate, inputs):
            gates.append(gate)

    z_bits = ((k, v) for k, v in inputs.items() if k.startswith("z"))
    z_bits = sorted(z_bits, key=lambda x: x[0], reverse=True)

    binary_number = int("".join(str(x[1]) for x in z_bits), 2)
    print(f"ANSWER1: { binary_number = }")
```

Let's break down the code:

- I read the input file and split it into the starting values of the wires and
  the gates.
- I parse the starting values of the wires into a dictionary.
- `while gates:` loop through the gates until there are no more gates left.
  - Pop the first gate from the list.
  - If the gate can be run, remove it from the list. Otherwise, add it back to
    the end of the list.
- I filter out the wires that start with `z` and sort them in reverse order, so
  that the most significant bit is first.
- I convert the binary number to a decimal number and print the answer.

My `run_gate` function is as follows:

```python
def run_gate(gate: str, inputs: dict[str, int]):
    left_input, operator, right_input, _, output = gate.split(" ")
    if left_input not in inputs or right_input not in inputs:
        return False
    match operator:
        case "AND":
            inputs[output] = inputs[left_input] & inputs[right_input]
        case "OR":
            inputs[output] = inputs[left_input] | inputs[right_input]
        case "XOR":
            inputs[output] = inputs[left_input] ^ inputs[right_input]
        case _:
            raise NotImplementedError(f"Operator {operator} not implemented")
    return True
```

It returns `False` if the gate can't be run because one of the inputs is
missing. Eventually, all the inputs will be available, and the gate will be run.

As always, I've only included the relevant parts of the code here, but to see my
full solution, you can check out my
[Advent of Code GitHub repository](https://github.com/VBenny42/AoC/blob/main/2024/day24/solution.py).

## Part 2 {#Part2}

It turns out the device is really trying to add two numbers `x` and `y` together
and storing the result in `z`. However, some wires have been swapped, so it's
not producing the correct result. We know that in total there are `4` pairs of
gates whose wires need to be swapped, i.e. there are `8` wires in total that are
out of place.

If we were to determine that we need to swap output wires `aaa` with `eee`,
`ooo` with `z99`, `bbb` with `ccc`, and `aoc` with `z24`, our answer would be
`aaa,aoc,bbb,ccc,eee,ooo,z24,z99`.

Our task is to find the pairs of wires that need to be swapped, then sort them
and print them as a comma-separated list.

### My Solution {#Part2Solution}

This device is actually a parallel
[full adder](https://en.wikipedia.org/wiki/Adder_(electronics)#Full_adder). For
every pair of bits for `x` and `y`, there's also a carry bit from the previous
pair. The output of the full adder should be the `z` bit, and the carry bit is
used for the next pair of bits. For the first pair of bits, there's no carry bit
from the previous pair.

<div style="text-align: center;">
    <img src="https://upload.wikimedia.org/wikipedia/commons/5/57/Fulladder.gif" alt="Full adder in action." height="324" width="480">
    <figcaption class="caption">Full adder in action.</figcaption>
</div>

I just have to check if the wires are in the correct spots for every pair of
bits starting from the least significant bit. If they're not, I add them to a
list of swaps, swap the wires, and start checking the adder from the start
again.

Before I start checking the adder, I have some helper functions to find the
output wire given the two inputs and an operator:

```python
def find_output_wire(a: str, b: str, operator: str, gates: list[str]):
    search = f"{a} {operator} {b}"
    search2 = f"{b} {operator} {a}"

    for gate in gates:
        if (search in gate) or (search2 in gate):
            return gate.split(" -> ")[1]
```

And a function to swap the wires:

```python
def swap_wires(a: str, b: str, gates: list[str]):
    new_gates = []

    for gate in gates:
        lhs, rhs = gate.split(" -> ")

        if rhs == a:
            new_gates.append(f"{lhs} -> {b}")
        elif rhs == b:
            new_gates.append(f"{lhs} -> {a}")
        else:
            new_gates.append(gate)

    return new_gates
```

This function returns a new list of gates, with the only difference being that
the gates that has output wire `a` and `b` have been swapped.

Now, I can start checking the adder:

```python
def get_swaps(gates: list[str]):
    carry_wire = None
    swaps = []
    bit = 0

    while bit < 45:
        x = f"x{bit:02}"
        y = f"y{bit:02}"
        z = f"z{bit:02}"

        if bit == 0:
            carry_wire = find_output_wire(x, y, "AND", gates)
        else:
            xy_xor_wire = find_output_wire(x, y, "XOR", gates)
            xy_and_wire = find_output_wire(x, y, "AND", gates)

            xy_carry_xor_wire = find_output_wire(carry_wire, xy_xor_wire, "XOR", gates)

            if xy_carry_xor_wire is None:
                swaps.append(xy_xor_wire)
                swaps.append(xy_and_wire)
                gates = swap_wires(xy_xor_wire, xy_and_wire, gates)
                bit = 0
                continue

            if xy_carry_xor_wire != z:
                swaps.append(xy_carry_xor_wire)
                swaps.append(z)
                gates = swap_wires(xy_carry_xor_wire, z, gates)
                bit = 0
                continue

            xy_carry_and_wire = find_output_wire(xy_xor_wire, carry_wire, "AND", gates)


            carry_wire = find_output_wire(xy_and_wire, xy_carry_and_wire, "OR", gates)

        bit += 1

    return swaps
```

Let's break down the code:

- `carry_wire` is the wire that carries the carry bit from the previous pair of
  bits. It's `None` for the first pair of bits.
- `swaps` is the list of wires that need to be swapped.
- `bit` is the index of the pair of bits I'm checking.
- `while bit < 45`. I know that the most significant bit of `z` is `45` from my
  input. So the `x` and `y` digits can have at most `44` bits, since the most
  significant bit of `z` is for the carry bit.
  - I get the wires for the current pair of bits.
  - `if bit == 0`. For the first pair of bits, I find the wire that carries the
    carry bit.
  - `else`. For every other pair of bits, I find the wires for the `XOR` and
    `AND` combination of `x` and `y`.
  - I find the wire for the `XOR` of the `XOR` of `x` and `y` and the carry
    wire. I didn't come up with the best variable names for this, but I'm just
    checking if the output wires for all the gates from the image are correct.
  - If the `xy_carry_xor_wire` is `None`, swap the `xy_xor_wire` and
    `xy_and_wire` wires.
  - If the `xy_carry_xor_wire` is not the output wire `z`, swap the
    `xy_carry_xor_wire` and `z` wires.
  - `xy_carry_and_wire` is the wire for the `AND` of the `XOR` of `x` and `y`
    and the carry wire. I find the wire for the `OR` of the `AND` of `x` and `y`
    which is the new carry wire, for the next pair of bits.
- I increment the bit and continue checking the adder.
- Once this while loop is done, every wire should be in the correct spot, and I
  return the list of swaps.

Finally, I call the `get_swaps` function in my `main2` function:

```python
def main2():
    with open("input.txt", "r", encoding="utf-8") as f:
        gates = f.read().split("\n\n")[1]

    gates = gates.splitlines()

    swaps = get_swaps(gates)
    swaps = ",".join(sorted(swaps))
    print(f"ANSWER2: { swaps = }")
```

Again, I've only included the relevant parts of the code here, check out my
[repository](https://github.com/VBenny42/AoC/blob/main/2024/day24/solution.py)
for the full solution.

---

That's it for day 24 of Advent of Code 2024! I hope you enjoyed reading my
solution and let's see how the rest of the month goes!
