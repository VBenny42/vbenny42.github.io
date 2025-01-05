---
title: "Advent of Code 2024 Day 8 – Resonant Collinearity"
layout: post
date: 2024-12-08 13:36
image: /assets/images/2024-12-14/christmas_tree_scaled.png
headerImage: true
tag:
    - python
    - advent-of-code
star: false
projects: true
category: blog
author: vineshbenny
description: Vinesh Benny's Advent of Code 2024 Day 8 solution in Python.

excerpt: "Vinesh Benny's Advent of Code 2024 Day 8 solution in Python."
seo_title: "Advent of Code 2024 Day 8 -- Resonant Collinearity by Vinesh Benny"
seo_description: "Vinesh Benny's Advent of Code 2024 Day 8 solution in Python."
---

# Day 8: [Resonant Collinearity](https://adventofcode.com/2024/day/8)

## Part 1 {#Part1}

Today's puzzle was pretty quick! The input was a grid of antennas of different
frequencies, where each frequency was represented by a single lowercase letter,
uppercase letter, or digit. The sample input was:

```
............
........0...
.....0......
.......0....
....0.......
......A.....
............
............
........A...
.........A..
............
............
```

In this grid, the frequencies are only `0` and `A`. _Antinodes_ are formed at
any point that is perfectly in line with two or more antennas, but only when one
antenna is twice as far away as the other one. This means that for any pair of
antennas, there should be an antinode on each side of the line connecting the
two antennas, and the distance from the antinode to the closest antenna should
be the same as the distance between the two antennas. Let's look at a small
example:

```
..........
...#......
#.........
....a.....
........a.
.....a....
..#.......
......#...
..........
..........
```

For the 3 antennas that are transmitting the `a` frequency, their antinodes are
marked with `#`. For the rightmost antenna, there should be an antinode to the
right of it as well, but it is out of bounds in the map, so it is _not marked_.

Now, let's look at the sample input, with the antinodes marked:

```
......#....#
...#....0...
....#0....#.
..#....0....
....0....#..
.#....A.....
...#........
#......#....
........A...
.........A..
..........#.
..........#.
```

For the topmost `A` antenna, an antinode for the above and to the right `0`
antennas is on the same spot. This is still an antinode that we need to count,
even though it is not marked with a `#`. So the number of unique antinodes in
this grid is `14`.

Our task is to find the number of unique antinodes that occur in the grid.

### My Solution {#Part1Solution}

Let's start with some pseudocode:

- Read the input grid into a 2D list.
- Find all the antennas in the grid, and store their coordinates, as well as the
  frequency they are transmitting.
- For each frequency:
  - Go through all pairs of antennas transmitting that frequency.
    - For each pair, calculate the distance between them.
    - Add the distance to the coordinates of the first antenna to get the
      coordinates of one antinode.
    - Check if the antinode is within the bounds of the grid.
    - If it is, store the antinode in a set.
    - Repeat the process for the other side of the line connecting the two
      antennas.
  - Return the set of antinodes for that frequency.
- Combine all the sets of antinodes for each frequency and return the length of
  the set.

Reading the input grid into a 2D list:

```python
with open("input.txt", "r") as f:
    grid = [list(line.strip()) for line in f]
```

Finding all the antennas in the grid:

```python
def find_frequencies(grid: Grid) -> dict[str, Set[Coordinate]]:
    frequencies = defaultdict(set)
    for y, row in enumerate(grid):
        for x, cell in enumerate(row):
            if cell != ".":
                frequencies[cell].add((x, y))
    return frequencies
```

I used a `defaultdict` to store the antennas transmitting each frequency. This
is because I wanted to avoid having to check if the frequency was already in the
dictionary before adding the antenna to the set. I go through each cell in the
grid and add the coordinates of the cell to the set corresponding to the
frequency of the cell, once I find a non-empty cell.

Now that I can find all the antennas and their frequencies, I need to find the
antinodes for each frequency:

```python
frequencies = find_frequencies(grid)
antinodes = set()
for frequency in frequencies:
    antinodes.update(find_antinodes_1(frequencies[frequency], grid))
```

`antinodes` is a set that will store all the unique antinodes in the grid. I go
through each frequency and find the antinodes for that frequency using the
`find_antinodes_1` function, then add them to the `antinodes` set.

The `find_antinodes_1` function is where most of the work happens:

```python
def find_antinodes_1(frequency: Set[Coordinate], grid: Grid) -> set[Coordinate]:
    antinodes = set()
    for f1, f2 in combinations(frequency, 2):
        diff_x, diff_y = f2[0] - f1[0], f2[1] - f1[1]
        pos_x, pos_y = f1[0] - diff_x, f1[1] - diff_y
        neg_x, neg_y = f2[0] + diff_x, f2[1] + diff_y
        if in_bounds((pos_x, pos_y), grid):
            antinodes.add((pos_x, pos_y))
        if in_bounds((neg_x, neg_y), grid):
            antinodes.add((neg_x, neg_y))
    return antinodes
```

Let's break down the function:

- I start by creating an empty set to store the antinodes.
- `for f1, f2 in combinations(frequency, 2):` I go through all pairs of antennas
  transmitting the same frequency using the
  [`combinations`](https://docs.python.org/3/library/itertools.html#itertools.combinations)
  function from the `itertools` module.
- `diff_x, diff_y = f2[0] - f1[0], f2[1] - f1[1]` I calculate the difference
  between the x and y coordinates of the two antennas.
- `pos_x, pos_y = f1[0] - diff_x, f1[1] - diff_y` I calculate the coordinates of
  the antinode on one side of the line connecting the two antennas.
- `neg_x, neg_y = f2[0] + diff_x, f2[1] + diff_y` Same as above, but for the
  other side of the line.
- `if in_bounds((pos_x, pos_y), grid):` I check if the antinode is within the
  bounds of the grid. If it is, I add it to the set of antinodes.
- `if in_bounds((neg_x, neg_y), grid):` Same as above, but for the other side of
  the line.
- Return all the antinodes found for the given frequency.

The `in_bounds` function is a simple helper function to check if a coordinate is
within the grid.

```python
def in_bounds(c: Coordinate, grid: Grid) -> bool:
    return 0 <= c[0] < len(grid[0]) and 0 <= c[1] < len(grid)
```

Finally, I can print the number of unique antinodes in the grid:

```python
print(f"LOGF: Number of unique antinodes within the map {len(antinodes)}")
```

As always, I've only included the relevant parts of the code here, but to see my
full solution, you can check out my
[Advent of Code GitHub repository](https://github.com/VBenny42/AoC/blob/main/2024/python/day08/solution.py).

## Part 2 {#Part2}

For part 2, the antinodes can be formed at _any_ point that is perfectly in line
with a pair of antennas, but the distance between antinodes are still the same
as the distance between the two antennas. This means that for any pair of
antennas, they themselves could be antinodes as well. Looking at another small
example:

```
T....#....
...T......
.T....#...
.........#
..#.......
..........
...#......
..........
....#.....
..........
```

In this grid, the antennas transmitting the `T` frequency are also antinodes for
each other. This means that the number of unique antinodes in this grid is `9`.

Looking at the sample input, the number of unique antinodes is `34`, counting
the antennas as antinodes as well:

```
##....#....#
.#.#....0...
..#.#0....#.
..##...0....
....0....#..
.#...#A....#
...#..#.....
#....#.#....
..#.....A...
....#....A..
.#........#.
...#......##
```

Now, our task is to find the number of unique antinodes in the grid, now that
they can be formed at any point that is perfectly in line with a pair of
antennas.

### My Solution {#Part2Solution}

The pseudocode for part 2 is very similar to part 1, all I need to do is keep
checking for antinodes on the line connecting the two antennas until I reach an
antinode that is out of bounds, then check in the next direction.

Therefore, the only change I need to make is to the `find_antinodes_2` function:

```python
def find_antinodes_2(frequency: Set[Coordinate], grid: Grid) -> set[Coordinate]:
    antinodes = set()
    for f1, f2 in combinations(frequency, 2):
        diff_x, diff_y = f2[0] - f1[0], f2[1] - f1[1]
        start_x, start_y = f1[0], f1[1]
        while in_bounds((start_x, start_y), grid):
            antinodes.add((start_x, start_y))
            start_x, start_y = start_x - diff_x, start_y - diff_y
        start_x, start_y = f2[0], f2[1]
        while in_bounds((start_x, start_y), grid):
            antinodes.add((start_x, start_y))
            start_x, start_y = start_x + diff_x, start_y + diff_y
    return antinodes
```

The only change I made was to add a `while` loop that keeps adding antinodes in
the direction of the line connecting the two antennas until it reaches a point
out of bounds. Then, I do the same for the other side of the line.

Now, I can update the main loop to use the new function:

```python
def main2():
    with open("input.txt", "r") as f:
        grid = [list(line.strip()) for line in f]
    frequencies = find_frequencies(grid)
    antinodes = set()
    for frequency in frequencies:
        antinodes.update(find_antinodes_2(frequencies[frequency], grid))
    print(f"LOGF: Number of unique antinodes within the map {len(antinodes)}")
```

Again, I've only included the relevant parts of the code here, check out my
[repository](https://github.com/VBenny42/AoC/blob/main/2024/python/day08/solution.py)
for the full solution.

---

That's it for day 8 of Advent of Code 2024! I hope you enjoyed reading my
solution and let's see how the rest of the month goes!
