---
title: "Advent of Code 2024 Day 25 – Code Chronicle"
layout: post
date: 2024-12-25 12:04
image: /assets/images/2024-12-14/christmas_tree_scaled.png
headerImage: true
tag:
    - python
    - advent-of-code
star: false
projects: true
category: blog
author: vineshbenny
description: Vinesh Benny's Advent of Code 2024 Day 25 solution in Python.

excerpt: "Vinesh Benny's Advent of Code 2024 Day 25 solution in Python."
seo_title: "Advent of Code 2024 Day 25 -- Code Chronicle by Vinesh Benny"
seo_description: "Vinesh Benny's Advent of Code 2024 Day 25 solution in Python."
---

# Day 25: [Code Chronicle](https://adventofcode.com/2024/day/25)

It's Christmas day and the final day of Advent of Code 2024! Today's was a
bittersweet one as it was the last day of the event. The puzzle was a fun one,
and I enjoyed solving it. Let's dive into the problem!

## Part 1 {#Part1}

We need to get into a locked room, but we don't know which key to use. We do
however have the schematics of all the locks and keys. We want to rule out the
key/lock combinations that won't work at all. Let's look at an example:

```
#####
.####
.####
.####
.#.#.
.#...
.....

#####
##.##
.#.##
...##
...#.
...#.
.....

.....
#....
#....
#...#
#.#.#
#.###
#####

.....
.....
#.#..
###..
###.#
###.#
#####

.....
.....
.....
#....
#.#..
#.#.#
#####
```

If a schematic's first row is `#####`, then the schematic must be for a lock,
and if the last row is `#####`, then the schematic must be for a key. For a
lock, the `#` are the pins that need to be pushed down, and for a key, the `#`
would be the shape that aligns with the pins. We can think of each schematic as
the heights of each column, or the number of `#` in each column. For example,
the first lock has column heights `0,5,3,4,3`:

```
#####
.####
.####
.####
.#.#.
.#...
.....
```

And the first key has column heights `5,0,2,1,3`:

```
.....
#....
#....
#...#
#.#.#
#.###
#####
```

For the first `4` columns, the key and lock look like they could fit together,
but in the rightmost column, the lock's pin overlaps with the key's shape. This
is because the sum of the heights of the key and lock in the rightmost column is
greater than the available space.

Converting the schematics into column heights for locks and keys:

- Locks:
  ```
  0,5,3,4,3
  1,2,0,5,3
  ```
- Keys:
  ```
  5,0,2,1,3
  4,3,4,0,2
  3,0,2,0,1
  ```

Now trying every key with every lock, we can rule out the key/lock combinations
that won't work:

- Lock `0,5,3,4,3` and key `5,0,2,1,3`: _overlap_ in the last column.
- Lock `0,5,3,4,3` and key `4,3,4,0,2`: _overlap_ in the second column.
- Lock `0,5,3,4,3` and key `3,0,2,0,1`: _all_ columns fit!
- Lock `1,2,0,5,3` and key `5,0,2,1,3`: _overlap_ in the first column.
- Lock `1,2,0,5,3` and key `4,3,4,0,2`: _all_ columns fit!
- Lock `1,2,0,5,3` and key `3,0,2,0,1`: _all_ columns fit!

So for this example, the number of unique lock/key combinations that fit
together without any overlap is `3`.

Our task is to find the number of unique lock/key combinations that fit together
without any overlap.

### My Solution {#Part1Solution}

I need to:

- Read the input schematic by schematic.
- Convert the schematics into column heights.
- Check all the key/lock combinations for overlap.

First, my schematic reading function:

```python
def parse_schematic(lines: list[str]) -> tuple[int, ...]:
    heights = [-1 for _ in range(5)]
    for _, line in enumerate(lines):
        for j, char in enumerate(line):
            if char == "#":
                heights[j] += 1
    return tuple(heights)
```

This function takes a list of strings, where each string is a row of the
schematic. It returns a tuple of the column heights.

Next, I need to read the input:

```python
with open("input.txt", "r", encoding="utf-8") as f:
	lines = f.read().splitlines()

locks = set()
keys = set()

for i in range(0, len(lines), 8):
	if lines[i] == "#####":
		locks.add(parse_schematic(lines[i : i + 8]))
	else:
		keys.add(parse_schematic(lines[i : i + 8]))
```

I iterate over the input lines in chunks of `8` to get each schematic. If the
top row is `#####`, then it's a lock, else it's a key. I add the column heights
as a tuple to the respective set.

Now, I need to check all the key/lock combinations for overlap:

```python
fits = 0
for lock in locks:
	for key in keys:
		if all(
			lock_height + key_height <= 5
			for lock_height, key_height in zip(lock, key)
		):
			fits += 1
```

I iterate over all the locks and keys and check if the sum of the column heights
is less than or equal to `5`. If it is, then the key/lock combination fits.

Finally, I print the number of unique lock/key combinations that fit:

```python
print(f"ANSWER: { fits }")
```

As always, I've only included the relevant parts of the code here, but to see my
full solution, you can check out my
[Advent of Code GitHub repository](https://github.com/VBenny42/AoC/blob/main/2024/day25/solution.py).

## Part 2 {#Part2}

There was no real part 2 for today's puzzle, but I did have some fun with it.
Each puzzle so far referred to days from previous years, and it all wrapped up
pretty nicely. I'm really proud of myself for completing all the puzzles this
year, and I hope to do the same next year!
