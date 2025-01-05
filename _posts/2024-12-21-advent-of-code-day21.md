---
title: "Advent of Code 2024 Day 21 – Keypad Conundrum"
layout: post
date: 2024-12-21 22:14
image: /assets/images/2024-12-14/christmas_tree_scaled.png
headerImage: true
tag:
    - python
    - advent-of-code
star: false
projects: true
category: blog
author: vineshbenny
description: Vinesh Benny's Advent of Code 2024 Day 21 solution in Python.

excerpt: "Vinesh Benny's Advent of Code 2024 Day 21 solution in Python."
seo_title: "Advent of Code 2024 Day 21 -- Keypad Conundrum by Vinesh Benny"
seo_description: "Vinesh Benny's Advent of Code 2024 Day 21 solution in Python."
---

# Day 21: [Keypad Conundrum](https://adventofcode.com/2024/day/21)

## Part 1 {#Part1}

Today's puzzle involves robots and keypads. There's a door that has a numeric
keypad that we need to unlock. The numeric keypad looks like this:

```
+---+---+---+
| 7 | 8 | 9 |
+---+---+---+
| 4 | 5 | 6 |
+---+---+---+
| 1 | 2 | 3 |
+---+---+---+
    | 0 | A |
    +---+---+
```

We have a list of codes (our input) that we must use to unlock the door. A line
from the input looks like `012A`. The actual code is `012` and `A` is the
confirmation key.

The bad thing about the door is that it's in a very dangerous area, so we can't
physically press the keys. Instead, we have to use a robot to do it for us. The
robot has a keypad like this:

```
    +---+---+
    | ^ | A |
+---+---+---+
| < | v | > |
+---+---+---+
```

The robot's arm will always start at the `A` key on the keypad. We can use the
directional keys to move the robot around the keypad, and when we want the arm
to press a key, we press the `A` key. So to make the robot type `029A`, the
sequence of inputs would be:

- `<` to move the arm from `A` (its initial position) to `0`.
- `A` to push the `0` button.
- `^A` to move the arm to the `2` button and push it.
- `>^^A` to move the arm to the `9` button and push it.
- `vvvA` to move the arm to the `A` button and push it.

The bad thing about the robot is that it's _also_ in a dangerous area, so we can
use _another_ robot to control it. The second robot also has the same keypad as
the first robot, with the arm always starting at `A`.

So, to actually input the code, we have to control the second robot to control
the first robot to input the code. Since the robots are in a dangerous area, we
need to use the shortest sequence of inputs to input the code, as we don't want
them to be there for too long.

To actually input `029A`, the sequence of inputs could be:

```
<vA<AA>>^AvAA<^A>A<v<A>>^AvA^A<vA>^A<v<A>^A>AAvA^A<v<A>A>^AAAvA<^A>A
v<<A>>^A<A>AvA<^AA>A<vAAA>^A
<A^A>^^AvvvA
029A
```

The first line is our input to the second robot, the second line is the input to
the first robot, the third line is what the first robot types, and the fourth
line is the actual code.

The _complexity_ of a code is _the length of the shortest sequence of inputs_
needed to input the code, times the _numeric part_ of the code. For example, the
complexity of `029A` is `68 * 29 = 1972`.

Our task is to find the sum of the complexities of all the codes in our input.

### My Solution {#Part1Solution}

My thinking for this problem was to simulate the first robot's movements around
the numeric keypad, then pass that sequence of movements as input to my
simulation function again for how the second robot would control the first one,
then do the same thing one more time to get the final sequence that the human
would have to input.

To simulate the robot's movements around the keypad, I used a dictionary to map
all the possible movements to the corresponding coordinates on the keypad.

```python
DIRECTIONAL_KEYPAD_TRANSITIONS = {
    "^": {
        "<": ["v<"],
        ">": [">v", "v>"],
        "A": [">"],
        "^": [""],
        "v": ["v"],
    },
    "v": {
        "<": ["<"],
        ">": [">"],
        "A": [">^", "^>"],
        "^": ["^"],
        "v": [""],
    },
    ">": {
        "<": ["<<"],
        ">": [""],
        "A": ["^"],
        "^": ["<^", "^<"],
        "v": ["<"],
    },
    "<": {
        "<": [""],
        ">": [">>"],
        "A": [">>^", ">^>"],
        "^": [">^"],
        "v": [">"],
    },
    "A": {
        "<": ["<v<", "v<<"],
        ">": ["v"],
        "A": [""],
        "^": ["<"],
        "v": ["<v", "v<"],
    },
}
```

The outer dictionary has the current position of the robot, and the inner
dictionary's keys are the positions the robot can move to. Every inner value is
a list of all the sequences that the robot can take to get from the current to
the target position. For example if the robot is at `^` and wants to move to
`>`, it can either take the sequence `v<` or `<v`.

I have a similar dictionary for the keypad, I won't include the entire thing
here but it's in my repo.

```python
NUMERIC_KEYPAD_TRANSITIONS = {
    "0": {
        "0": [""],
        "1": ["^<"],
        "2": ["^"],
        "3": [">^", "^>"],
        "4": ["^<^", "^^<"],
        "5": ["^^"],
        "6": [">^^", "^>^", "^^>"],
        "7": ["^<^^", "^^<^", "^^^<"],
        "8": ["^^^"],
        "9": [">^^^", "^>^^", "^^>^", "^^^>"],
        "A": [">"],
    }, ...
```

Also, every inner dictionary also has the current position as a key, and its
value is always just the empty string. This is because if the start and target
are the same key, the arm shouldn't move at all.

Now that I have these transition dictionaries, I can simulate the robots'
movements around the keypad.

```python
@cache
def shortest_sequence(level: int, sequence_str: str, num_robots: int):
    if level == num_robots + 1:
        return len(sequence_str)

    transitions = (
        NUMERIC_KEYPAD_TRANSITIONS if level == 0 else DIRECTIONAL_KEYPAD_TRANSITIONS
    )

    sequence = 0
    for current, target in zip("A" + sequence_str, sequence_str):
        possible_paths = (
            shortest_sequence(level + 1, path + "A", num_robots)
            for path in transitions[current][target]
        )
        sequence += min(
            (path for path in possible_paths if path is not None), default=1
        )

    return sequence
```

Let's break down the code:

- `@cache` is a decorator from the `functools` module that caches the results of
  the function so that if the function is called with the same arguments again,
  it returns the cached result instead of recalculating it.
- `if level == num_robots + 1:` is the base case of the recursive function. If
  the level is equal to the number of robots plus one, then we've reached the
  sequence that the actual human will have to input, so return the length of
  that sequence. We don't actually need to return the sequence itself, just its
  length for the complexity calculation.
- `transitions = ...` is a ternary operator that selects the transition that the
  robot will use based on the level. If the level is 0, then the robot is moving
  around the numeric keypad, otherwise it's moving around the directional
  keypad.
- `sequence = 0` is the variable that will store the length of the shortest
  sequence of inputs needed to input the code.
- `for current, target in zip("A" + sequence_str, sequence_str):` is a loop that
  iterates over the current and target keys that the robot has to move to. The
  `zip` function is used to iterate over the two strings in parallel. We always
  start off with the robot at the `A` key, so we prepend an `A` to the sequence
  string.
- `possible_paths = ...` is a generator expression that generates all the
  possible paths that the robot can take to move from the current key to the
  target key. It calls the `shortest_sequence` function recursively to get the
  length of the shortest sequence of inputs needed to move from the current key
  to the target key after the robot has moved to the next level. `A` is always
  appended to the path because the robot has to press the `A` key to confirm the
  input at the end.
- `sequence += ...` is where the length of the shortest sequence of inputs is
  calculated. It takes the minimum of all the possible paths that the robot can
  take to move from the current key to the target key. If there are no possible
  paths, then the default value is 1. This is because if there are no possible
  paths, it means that the robot is already at the target key, so it doesn't
  need to move at all.
- Finally, the function returns the length of the shortest sequence of inputs
  needed to input the code.

Putting it all together,

```python
def main1():
    with open("input.txt", "r", encoding="utf-8") as f:
        lines = f.read().splitlines()

    sum_of_shortest_sequences = 0
    for line in lines:
        number = int(line[:-1])
        shortest_sequence_length = shortest_sequence(0, line, 2)
        sum_of_shortest_sequences += shortest_sequence_length * number
    print(f"ANSWER1: { sum_of_shortest_sequences }")
```

I read the input from the file, then iterate over each line in the input. The
input always ends with a letter, so I can get the number by slicing the line
until the last character. Then I calculate the complexity for each code and sum
them up.

As always, I've only included the relevant parts of the code here, but to see my
full solution, you can check out my
[Advent of Code GitHub repository](https://github.com/VBenny42/AoC/blob/main/2024/python/day21/solution.py).

## Part 2 {#Part2}

For part 2, there's an _even more_ dangerous situation, that calls for `25`
robots to be used instead of `2` like in part 1.

Our task is to find the sum of the complexities of all the codes in our input,
just like in part 1, we just need to calculate the sequence to get the code
through `25` robots.

### My Solution {#Part2Solution}

My solution for part 2 uses the same recursive function as part 1, but with `25`
robots instead of `2`.

```python
def main2():
    with open("input.txt", "r", encoding="utf-8") as f:
        lines = f.read().splitlines()

    sum_of_shortest_sequences = 0
    for line in lines:
        number = int(line[:-1])
        shortest_sequence_length = shortest_sequence(0, line, 25)
        sum_of_shortest_sequences += shortest_sequence_length * number
    print(f"ANSWER2: { sum_of_shortest_sequences  }")
```

The `@cache` decorator was added for my part 2 solution, as there were way more
recursive calls than in part 1. Without caching, my solution was taking too long
to wait for an answer, but I knew that my solution was correct as it worked for
part 1. Caching the results of the function made it run much faster.

Again, I've only included the relevant parts of the code here, check out my
[repository](https://github.com/VBenny42/AoC/blob/main/2024/python/day21/solution.py)
for the full solution.

---

That's it for day 21 of Advent of Code 2024! I hope you enjoyed reading my
solution and let's see how the rest of the month goes!
