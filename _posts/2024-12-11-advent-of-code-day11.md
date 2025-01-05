---
title: "Advent of Code 2024 Day 11 – Plutonian Pebbles"
layout: post
date: 2024-12-11 17:35
image: /assets/images/2024-12-14/christmas_tree_scaled.png
headerImage: true
tag:
    - python
    - advent-of-code
star: false
projects: true
category: blog
author: vineshbenny
description: Vinesh Benny's Advent of Code 2024 Day 11 solution in Python.

excerpt: "Vinesh Benny's Advent of Code 2024 Day 11 solution in Python."
seo_title: "Advent of Code 2024 Day 11 -- Plutonian Pebbles by Vinesh Benny"
seo_description: "Vinesh Benny's Advent of Code 2024 Day 11 solution in Python."
---

# Day 11: [Plutonian Pebbles](https://adventofcode.com/2024/day/11)

## Part 1 {#Part1}

The input for today's problem is a list of numbers that represent stones with
the numbers engraved on them. The stones are changing every time we blink, and
they change according to the following rules, whichever applies first:

- If the stone is engraved with `0`, it changes to `1`.
- If the stone is engraved with a number that has an even number of digits, it
  gets split into stones. The first stone gets the first half of the number's
  digits, and the second stone gets the second half. The numbers do not keep
  leading zeros, so `1000` gets split into `10` and `0`.
- If neither of the above rules applies, the stone gets multiplied by `2024`.

Their order is preserved, so the stones are always in the same order as they
were first seen.

For example, if `5` stones are seen with the arrangement `0 1 10 99 999`, after
one blink, the following changes will occur:

- The first stone changes to `1`.
- The second stone gets multiplied by `2024`, so it changes to `2024`.
- The third stone gets split into two stones, `1` and `0`.
- The fourth stone gets split into two stones, `9` and `9`.
- The fifth stone gets multiplied by `2024`, so it changes to `2021976`.

Looking at a larger example:

```
Initial arrangement:
125 17

After 1 blink:
253000 1 7

After 2 blinks:
253 0 2024 14168

After 3 blinks:
512072 1 20 24 28676032

After 4 blinks:
512 72 2024 2 0 2 4 2867 6032

After 5 blinks:
1036288 7 2 20 24 4048 1 4048 8096 28 67 60 32

After 6 blinks:
2097446912 14168 4048 2 0 2 4 40 48 2024 40 48 80 96 2 8 6 7 6 0 3 2
```

After the 6th blink, we have `22` stones, and after `25` blinks, we would have
`55312` stones.

Our task is to find the number of stones there would be after `25` blinks from
the initial input arrangement.

### My Solution{#Part1Solution}

My pseudocode for this problem is as follows:

- Read the input and split it into a list of integers.
- For 25 iterations, apply the rules to each stone in the list.
- Sum the number of stones in the list after 25 iterations.

Reading the input and splitting it into a list of integers:

```python
with open("input.txt", "r") as f:
    stones = list(map(int, f.read().split()))
```

For 25 iterations, "blink":

```python
for _ in range(25):
    stones = blink(stones)
```

Going over my `blink` function:

```python
def blink(stones: List[int]) -> list[int]:
    return [new_stone for stone in stones for new_stone in apply_rules(stone)]
```

`apply_rules` is a function that applies the rules to a single stone and returns
the resulting list of stones.

```python
def apply_rules(stone: int) -> List[int]:
    if stone == 0:
        return [1]

    length = floor(log10(stone)) + 1

    if length % 2 == 0:
        split_point = length // 2
        first_half = stone // 10**split_point
        second_half = stone % 10**split_point
        return [first_half, second_half]

    return [stone * 2024]
```

Breaking down the function:

- If the stone is `0`, return a list with `1`.
- `length = floor(log10(stone)) + 1` calculates the number of digits in the
  stone.
- `if length % 2 == 0:` checks if the number of digits is even.
  - If it is, the stone is split into two stones, and the first half and second
    half are calculated.
- return `[stone * 2024]` returns a list with the stone multiplied by `2024`, if
  none of the above rules apply.

Finally, print the number of stones after 25 blinks:

```python
def main1():
    with open("input.txt", "r") as f:
        stones = list(map(int, f.read().split()))
    for _ in range(25):
        stones = blink(stones)
    print(f"LOG: { len(stones) = }")
```

As always, I've only included the relevant parts of the code here, but to see my
full solution, you can check out my
[Advent of Code GitHub repository](https://github.com/VBenny42/AoC/blob/main/2024/python/day11/solution.py).

## Part 2 {#Part2}

For Part 2, we now have to calculate the number of stones after `75` blinks.

I tried running my solution from Part 1 with `75` blinks, but it essentially
stopped making progress after the 43rd blink. I realized that the number of
stones was increasing exponentially, so I needed to optimize my solution.

### My Solution {#Part2Solution}

Some observations I made:

- No stone affects another stone, so the order of processing the stones doesn't
  matter.
- We just need to keep track of the number of stones after each blink, so no
  need to keep the stones themselves.
- After a while, certain stones will occur multiple times. For example, `0` will
  always change to `1`, which will always change to `2024`, and so on. So we can
  memoize the number of stones after each blink for each stone.

I decided to implement a recursive version of my `apply_rules` function that
takes a stone and the number of blinks left and returns the number of stones
after that many blinks. I also memoized the function by using the
[`cache`](https://docs.python.org/3/library/functools.html#functools.cache)
decorator.

Let's look at my main function for Part 2:

```python
def main2():
    with open("input.txt", "r") as f:
        stones = list(map(int, f.read().split()))
    stones = blink_recursive(stones, 75)
    print(f"LOG: { stones = }")
```

This reads the input, splits it into a list of integers, and then calls the
`blink_recursive` function with the stones and the number of blinks. Now, let's
look at my `blink_recursive` function:

```python
def blink_recursive(stones: List[int], blinks: int) -> int:
	    return sum(apply_rules_recursive(stone, blinks) for stone in stones)
```

This function sums the number of stones after all the `blinks` have passed for
each stone in the list. Now, let's look at my `apply_rules_recursive` function:

```python
@cache
def apply_rules_recursive(stone: int, blinks: int) -> int:
    if blinks == 0:
        return 1
    if stone == 0:
        return apply_rules_recursive(1, blinks - 1)
    length = floor(log10(stone)) + 1
    if length % 2 == 0:
        split_point = length // 2
        first_half = stone // 10**split_point
        second_half = stone % 10**split_point
        return apply_rules_recursive(first_half, blinks - 1) + apply_rules_recursive(second_half, blinks - 1)
    return apply_rules_recursive(stone * 2024, blinks - 1)
```

Let's break down the function:

- `@cache` is a decorator that memoizes the function.
- If `blinks == 0`, Base case. Only the stone passed in is left, so return `1`.
- If the stone is `0`, return the number of stones after `blinks - 1` for `1`.
  This call will be memoized for later stones that are `1`.
- `length = floor(log10(stone)) + 1 ... stone % 10**split_point` Logic is same
  as in Part 1.
- `return apply_rules_recursive(first_half, blinks - 1) + apply_rules_recursive(second_half, blinks - 1)`
  Recursively call the function with both halves of the original stone.
- `return apply_rules_recursive(stone * 2024, blinks - 1)` Recursively call the
  function with the stone multiplied by `2024`.

And that's it! This solution is much faster than my list approach from Part 1.
Using it to compute the Part 1 answer is also much faster than the original
solution.

```
LOG:  len(stones) = 186424
Function 'main1' executed in 0.1404s
LOG:  stones = 186424
Function 'main3' executed in 0.0023s
```

Memory usage is also much lower, as we only need to store the number of stones
instead of the stones themselves. This goes to show how important it is to
utilize different algorithms and data structures for different problems.

Again, I've only included the relevant parts of the code here, check out my
[repository](https://github.com/VBenny42/AoC/blob/main/2024/python/day11/solution.py)
for the full solution.

---

That's it for day 11 of Advent of Code 2024! I hope you enjoyed reading my
solution and let's see how the rest of the month goes!

```
```
