---
title: "Advent of Code 2024 Day 19 – Linen Layout"
layout: post
date: 2024-12-19 13:33
image: /assets/images/2024-12-14/christmas_tree_scaled.png
headerImage: true
tag:
    - python
    - advent-of-code
star: false
projects: true
category: blog
author: vineshbenny
description: Vinesh Benny's Advent of Code 2024 Day 19 solution in Python.

excerpt: "Vinesh Benny's Advent of Code 2024 Day 19 solution in Python."
seo_title: "Advent of Code 2024 Day 19 -- Linen Layout by Vinesh Benny"
seo_description: "Vinesh Benny's Advent of Code 2024 Day 19 solution in Python."
---

# Day 19: [Linen Layout](https://adventofcode.com/2024/day/19)

## Part 1 {#Part1}

For today's puzzle, we're at an [onsen](https://en.wikipedia.org/wiki/Onsen) and
to gain entry, we need to help arrange some towels.

Every towel has a specific striped pattern, and there are an infinite number of
each. The towel stripe colors can be white (`w`), blue (`u`), black (`b`), red
(`r`), or green (`g`). Also, a towel's pattern cannot be reversed, so a towel
with the pattern `wubrg` cannot be flipped to `grbuw`.

The staff has a list of designs that they want to display, and we need to see if
the towels they have in stock can be arranged to match the designs. So, to
display the design `rgrgr`, you could use two `rg` towels and then an `r` towel,
an rgr towel and then a `gr` towel, or even a single massive `rgrgr` towel
(assuming such towel patterns were actually available). Let's look at an
example:

```
r, wr, b, g, bwu, rb, gb, br

brwrr
bggr
gbbr
rrbgbr
ubwu
bwurrg
brgr
bbrgwb
```

The first line are all the available patterns, and the rest are the designs that
the staff wants to display. For this example, the designs are possible or not as
follows:

- `brwrr` can be made with a `br` towel, then a `wr` towel, and then finally an
  `r` towel.
- `bggr` can be made with a `b` towel, two `g` towels, and then an `r` towel.
- `gbbr` can be made with a `gb` towel and then a `br` towel.
- `rrbgbr` can be made with `r`, `rb`, `g`, and `br`.
- `ubwu` is _impossible_.
- `bwurrg` can be made with `bwu`, `r`, `r`, and `g`.
- `brgr` can be made with `br`, `g`, and `r`.
- `bbrgwb` is _impossible_.

Then, for the above example, the number of possible designs is 6. Our task is to
find the number of possible designs for the given input.

### My Solution {#Part1Solution}

I could solve this problem by using a dynamic programming approach. First I need
to parse the input and then I write a function to see if the design can be made
with the available towels.

Reading the input:

```python
def main1():
    with open("input.txt", "r", encoding="utf-8") as f:
        part1, part2 = f.read().split("\n\n")
        towels = set(part1.split(", "))

        designs = part2.splitlines()

    possible_designs = sum(is_possible(towels, design) for design in designs)

    print(f"ANSWER: { possible_designs = }")
```

```python
def is_possible(towels: set[str], design: str) -> bool:
    n = len(design)
    dp = [False] * (n + 1)
    dp[0] = True

    for i in range(1, n + 1):
        for towel in towels:
            if design.startswith(towel, i - len(towel)) and dp[i - len(towel)]:
                dp[i] = True
                break

    return dp[n]
```

Line by line explanation:

- `n = len(design)`: Get the length of the design.
- `dp = [False] * (n + 1)`: Create a list of size `n + 1` and initialize all
  values to `False`. Every index `i` in this list will store whether the design
  can be made up to the `i`th character.
- `dp[0] = True`: The design can always be made up to the 0th character, which
  is an empty string.
- `for i in range(1, n + 1)`: Iterate over the design.
- `for towel in towels`: Iterate over the available towels.
- `if design.startswith(towel, i - len(towel)) and dp[i - len(towel)]:`: Check
  if the design can be made up to the `i - len(towel)`th character and if the
  current towel can be used to make the design.
- `dp[i] = True`: If the above condition is true, then the design can be made up
  to the `i`th character. No need to check the rest of the towels.
- `return dp[n]`: Return whether the design can be made up to the `n`th
  character, which is if the entire design can be made.

As always, I've only included the relevant parts of the code here, but to see my
full solution, you can check out my
[Advent of Code GitHub repository](https://github.com/VBenny42/AoC/blob/main/2024/day19/solution.py).

## Part 2 {#Part2}

For part 2, the staff wants to know all the possible ways to display the design.
Looking at the example from part 1:

`brwrr` can be made in two different ways: `b`, `r`, `wr`, `r` _or_ `br`, `wr`,
`r`.

`bggr` can only be made with `b`, `g`, `g`, and `r`.

`gbbr` can be made 4 different ways:

- `g`, `b`, `b`, `r`
- `g`, `b`, `br`
- `gb`, `b`, `r`
- `gb`, `br`

`rrbgbr` can be made 6 different ways:

- `r`, `r`, `b`, `g`, `b`, `r`
- `r`, `r`, `b`, `g`, `br`
- `r`, `r`, `b`, `gb`, `r`
- `r`, `rb`, `g`, `b`, `r`
- `r`, `rb`, `g`, `br`
- `r`, `rb`, `gb`, `r`

`bwurrg` can only be made with `bwu`, `r`, `r`, and `g`.

`brgr` can be made in two different ways: `b`, `r`, `g`, `r` o`r` `br`, `g`,
`r`.

`ubwu` and `bbrgwb` are still impossible.

In total, there are `16` possible ways to display the valid designs. Our task is
to find the number of possible ways to display the designs for the given input.

### My Solution {#Part2Solution}

My solution for part 2 is very similar to part 1. I just need to modify my
dynamic programming function to store the number of ways to make the design
instead of just whether it is possible.

```python
def different_combos(towels: set[str], design: str) -> int:
    n = len(design)
    dp = [0] * (n + 1)
    dp[0] = 1

    for i in range(1, n + 1):
        for towel in towels:
            if design.startswith(towel, i - len(towel)):
                dp[i] += dp[i - len(towel)]

    return dp[n]
```

Instead of storing a boolean value in the `dp` list, I'm storing the number of
possible ways to make the design up to the `i`th character. I don't break out of
the loop when I find a towel that can be used to make the design, because I want
to count _all_ the possible ways to make the design.

Otherwise, my main function is the same as part 1.

```python
def main2():
    with open("input.txt", "r", encoding="utf-8") as f:
        part1, part2 = f.read().split("\n\n")
        towels = set(part1.split(", "))

        designs = part2.splitlines()

    different_possible_designs = sum(
        different_combos(towels, design) for design in designs
    )

    print(f"ANSWER: { different_possible_designs = }")
```

Again, I've only included the relevant parts of the code here, check out my
[repository](https://github.com/VBenny42/AoC/blob/main/2024/day19/solution.py)
for the full solution.

---

That's it for day 19 of Advent of Code 2024! I hope you enjoyed reading my
solution and let's see how the rest of the month goes!
