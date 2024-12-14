---
title: "Advent of Code 2024 Day 13 – Claw Contraction"
layout: post
date: 2024-12-13 16:18
image: /assets/images/favicon/apple-touch-icon.png
headerImage: false
tag:
    - python
    - advent-of-code
star: false
projects: true
category: blog
author: vineshbenny
description: Vinesh Benny's Advent of Code 2024 Day 13 solution in Python.

excerpt: "Vinesh Benny's Advent of Code 2024 Day 13 solution in Python."
seo_title: "Advent of Code 2024 Day 13 -- Claw Contraction by Vinesh Benny"
seo_description: "Vinesh Benny's Advent of Code 2024 Day 13 solution in Python."
---

# Day 13: [Claw Contraction](https://adventofcode.com/2024/day/13)

## Part 1

The situation for today's problem is that we want to play with claw machines in
an arcade. Each claw machine has two buttons, `A` and `B`, and each button moves
the claw a specific amount along the x-axis and y-axis. To use the `A` button,
it costs `3` tokens, while the `B` button costs `1` tokens. Each machine has a
prize at a specific location, and we want to find the minimum number of tokens
needed to reach the prize.

Our input is a list of claw machines, each element has the button behavior and
the prize location for that machine. For example:

```
Button A: X+94, Y+34
Button B: X+22, Y+67
Prize: X=8400, Y=5400

Button A: X+26, Y+66
Button B: X+67, Y+21
Prize: X=12748, Y=12176

Button A: X+17, Y+86
Button B: X+84, Y+37
Prize: X=7870, Y=6450

Button A: X+69, Y+23
Button B: X+27, Y+71
Prize: X=18641, Y=10279
```

The first machine has the following behavior:

- Button `A` moves the claw `94` units along the x-axis and `34` units along the
  y-axis.
- Button `B` moves the claw `22` units along the x-axis and `67` units along the
  y-axis.
- The prize is located at `X=8400, Y=5400`.

If `A` is pressed `80` times, and `B` is pressed `40` times, the claw will reach
the prize location. This costs `80 * 3 + 40 * 1 = 240 + 40 = 280` tokens. This
is the minimum number of tokens needed to reach the prize. For the second and
fourth machine, no combination of button presses will reach the prize location.
For the third machine, the prize could be won by pushing the `A` button `38`
times and the `B` button `86` times. Doing this would cost a total of `200`
tokens.

So for the given input, the most possible prizes we can win is `2`, costing a
total of `480` tokens.

Our task is to find the fewest number of tokens needed to win all the possible
prizes.

### My Solution

At first, I thought of solving this problem using dynamic programming. To find
the minimum number of tokens needed to reach the prize, at each step, I would
check if the prize could be reached by pressing either button `A` or button `B`,
decrementing the number of steps left to reach the prize. If the prize could be
reached, I would store the number of tokens used to reach the prize. I would
then repeat this process for all the machines and find the minimum number of
tokens needed to win all the possible prizes.

So I started by parsing the input and storing the button behavior and prize
location.

```python
with open("input.txt", "r") as f:
    lines = f.readlines()                                                        
machines = [read_machine_info(lines[i : i + 3]) for i in range(0, len(lines), 4)]
```

```python
def read_machine_info(lines: List[str]) -> dict:
    pattern_behavior = r"([A|B]):.*X\+(\d+).*Y\+(\d+)"
    res = {}
    for line in lines[:2]:
        matches = search(pattern_behavior, line)
        assert matches is not None
        res[matches.group(1)] = (int(matches.group(2)), int(matches.group(3)))
    pattern_prize = r"X=(\d+), Y=(\d+)"
    matches = search(pattern_prize, lines[2])
    assert matches is not None
    res["prize"] = tuple(map(int, matches.groups()))
    return res
```

Explaining the `read_machine_info` function:

- `pattern_behavior = r"([A|B]):.*X\+(\d+).*Y\+(\d+)"` is a regex pattern that
  matches the button behavior of the machine. It matches the button `A` or `B`
  and the number of units the claw moves along the x-axis and y-axis.
- `for line in ... res[matches.group(1)] = (int(matches.group(2)), int(matches.group(3)))`
  extracts the button behavior and stores it in a dictionary.
- `pattern_prize = r"X=(\d+), Y=(\d+)"` is a regex pattern that matches the
  prize location of the machine.
- `matches = search(pattern_prize, lines[2]) ... matches.groups()))` extracts
  the prize location and stores it in the dictionary.

Once I've parsed the input, I would then find the minimum number of tokens
needed for each machine with my DP function.

```python
def find_cheapest_combination(machine: dict):
    a_cost = 3
    b_cost = 1

    p_x, p_y = machine["prize"]
    a_x, a_y = machine["A"]
    b_x, b_y = machine["B"]

    memo = {}

    def dp(x, y):
        if x < 0 or y < 0:
            return float("inf")
        if x == 0 and y == 0:
            return 0
        if (x, y) in memo:
            return memo[(x, y)]
        cost = min(a_cost + dp(x - a_x, y - a_y), b_cost + dp(x - b_x, y - b_y))
        memo[(x, y)] = cost
        return cost

    result = dp(p_x, p_y)
    return result if result < float("inf") else None
```

Explaining the `find_cheapest_combination` function:

- `a_cost = 3` and `b_cost = 1` are the costs of pressing buttons `A` and `B`.
- `p_x, p_y = machine["prize"]` extracts the prize location.
- `a_x, a_y = machine["A"]` and `b_x, b_y = machine["B"]` extract the button
  behavior.
- `memo = {}` is a dictionary that stores the minimum number of tokens needed to
  reach the origin.
- `dp(x, y)` my DP function that finds the minimum number of tokens needed to
  reach the origin.
- `if x < 0 or y < 0: return float("inf")` if the prize location is negative, no
  combination of button presses will reach the prize, so I return infinity.
- `if x == 0 and y == 0: return 0` if the origin location is reached, no more
  tokens needed to reach origin, return 0.
- `if (x, y) in memo: return memo[(x, y)]` if the minimum number of tokens
  needed to reach the origin is already calculated, return it.
- `cost = min(a_cost + dp(x - a_x, y - a_y), b_cost + dp(x - b_x, y - b_y))`
  pick the minimum number of tokens needed to reach the prize by pressing either
  button `A` or button `B`.
- `memo[(x, y)] = cost` store the minimum number of tokens needed to reach the
  origin from the current location.
- `return cost` return the minimum number of tokens needed to reach the origin.

Finally, I would find the minimum number of tokens needed to win by calling the
function with the prize location. If it returns a value, then there is some
combination of button presses that will reach the prize from the origin, so
return the cost that was found.

Now, I would iterate through all the machines and find the minimum number of
tokens needed to win all the possible prizes.

```python
def main1():
    with open("input.txt", "r") as f:
        lines = f.readlines()
    machines = [read_machine_info(lines[i : i + 3]) for i in range(0, len(lines), 4)]
    min_tokens = 0
    for machine in machines:
        cheapest_combination = find_cheapest_combination(machine)
        if cheapest_combination is not None:
            min_tokens += cheapest_combination
    print(f"LOG: { min_tokens = }")
```

This successfully solves the problem, but for part 2 I have a better approach.

As always, I've only included the relevant parts of the code here, but to see my
full solution, you can check out my
[Advent of Code GitHub repository](https://github.com/VBenny42/AoC/blob/main/2024/day13/solution.py).

## Part 2

For part 2, the prize locations from the input file were recorded wrong due to
calibration errors! For every prize location we need to add `10000000000000` to
both the x and y-axes.

The task remains the same, to find the fewest number of tokens needed to win all
the possible prizes.

### My Solution

I tried my DP approach from part 1, but I got recursion depth errors because of
how large the prize locations were. So I had to come up with a new approach.

Looking at the input more, I realized that an `A` or `B` move is a vector that
moves the claw along the x-axis and y-axis. So I could represent the prize
location as a vector as well, and then solve the problem using linear algebra.

<script id="MathJax-script" async
          src="https://cdn.jsdelivr.net/npm/mathjax@3.0.1/es5/tex-mml-chtml.js">
  </script>

The problem can be represented as a system of linear equations:

<p>
	\[
	a \cdot A + b \cdot B = P \\
	\]
\[
\begin{bmatrix}
a_x & b_x \\
a_y & b_y
\end{bmatrix}
\begin{bmatrix}
a \\
b
\end{bmatrix}
=
\begin{bmatrix}
p_x \\
p_y
\end{bmatrix}
	\]
</p>

So now, I can solve this system of linear equations in any way I want to find
the combination of `a` and `b` that will reach the prize location. Also, there
should only be one solution to this system of linear equations, so the "minimum
number of tokens" really means just finding the one combination. It's a
guarantee that the vectors are not scalar multiples of each other, so there can
only be one solution.

I solve this system of linear equations in my `find_cheapest_combination2`
function:

```python
def find_cheapest_combination2(machine: dict):
    p_x, p_y = machine["prize"]
    a_x, a_y = machine["A"]
    b_x, b_y = machine["B"]

    determinant = a_x * b_y - a_y * b_x
    if determinant == 0:
        return None

    a = (p_x * b_y - p_y * b_x) // determinant
    b = (a_x * p_y - a_y * p_x) // determinant

    if (a_x * a + b_x * b) != p_x or (a_y * a + b_y * b) != p_y:
        return None

    return 3 * a + b
```

I use
[Cramer's Rule](https://en.wikipedia.org/wiki/Cramer%27s_rule#Explicit_formulas_for_small_systems)
to solve the system of linear equations. If the determinant is `0`, then there
is no unique solution, so I return `None`. Otherwise, I calculate the values of
`a` and `b` to find the combination of button presses needed to reach the prize.
If the combination is not valid, then I return `None`. Otherwise, I return the
cost of the combination.

Finally, I iterate through all the machines and find number of tokens needed to
win all the possible prizes.

```python
def main2():
    with open("input.txt", "r") as f:
        lines = f.readlines()
    machines = [read_machine_info(lines[i : i + 3]) for i in range(0, len(lines), 4)]
    addition = 10000000000000
    min_tokens = 0
    for machine in machines:
        machine["prize"] = (
            machine["prize"][0] + addition,
            machine["prize"][1] + addition,
        )
        cheapest_combination = find_cheapest_combination2(machine)
        if cheapest_combination is not None:
            min_tokens += cheapest_combination
    print(f"LOG: { min_tokens = }")
```

The additions to the prize locations are done here, and the
`find_cheapest_combination2` function is called to find the number of tokens
needed to win all the possible prizes.

This approach is much faster and more efficient than my DP approach from part 1,
but I had to google a bit to find Cramer's Rule as I haven't used it in a long
while. It was a fun problem to solve, showing how problems can be solved in many
different ways!

Again, I've only included the relevant parts of the code here, check out my
[repository](https://github.com/VBenny42/AoC/blob/main/2024/day13/solution.py)
for the full solution.

---

That's it for day 13 of Advent of Code 2024! I hope you enjoyed reading my
solution and let's see how the rest of the month goes!
