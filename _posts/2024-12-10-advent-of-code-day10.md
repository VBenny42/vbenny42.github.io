---
title: "Advent of Code 2024 Day 10 – Hoof It"
layout: post
date: 2024-12-10 16:26
image: /assets/images/2024-12-14/christmas_tree_scaled.png
headerImage: true
tag:
    - python
    - advent-of-code
star: false
projects: true
category: blog
author: vineshbenny
description: Vinesh Benny's Advent of Code 2024 Day 10 solution in Python.

excerpt: "Vinesh Benny's Advent of Code 2024 Day 10 solution in Python."
seo_title: "Advent of Code 2024 Day 10 -- Hoof It by Vinesh Benny"
seo_description: "Vinesh Benny's Advent of Code 2024 Day 10 solution in Python."
---

# Day 10: [Hoof It](https://adventofcode.com/2024/day/10)

## Part 1 {#Part1}

Today's puzzle's input is a topographic map, with each cell corresponding to a
height level, `0` being the lowest and `9` being the highest. For a given map,
we need to find all the hiking trails that can be formed by starting from a cell
marked `0` and ending at a cell marked `9`. A trail is valid, if from one step
to the next, the height difference is exactly `1`, and the directions can only
be up, down, left, or right. For example:

<div class="side-by-side">
    <div class="toleft">
<div class="language-plaintext highlighter-rouge"><div class="highlight"><pre class="highlight"><code>8880888
8881888
8882888
6543456
7111117
8111118
9111119
</code></pre></div></div>
    </div>

    <div class="toright">

<div class="language-plaintext highlighter-rouge"><div class="highlight"><pre class="highlight"><code>...0...
...1...
...2...
6543456
7.....7
8.....8
9.....9</code></pre></div></div>
    </div>
<figcaption>The one on the left is the topographic map, and the one on the right is the map with the trails marked and irrelevant cells removed.</figcaption>
</div>

This map has two trails that start from the singular `0` cell. Each map has a
score, where the score is the total number of distinct `9` cells that can be
reached from each `0` cell in the map. So this example map would have a score of
`2`. Looking at a larger example:

```
89010123
78121874
87430965
96549874
45678903
32019012
01329801
10456732
```

This map has 9 `0` cells (also called trailheads). For each of the trailheads in
reading order, the following number of distinct `9` cells can be reached from
them: `5`, `6`, `5`, `3`, `1`, `3`, `5`, `3`, and `5`. So the sum of these
numbers is `36`.

Our task is to find the score of the given map.

### My Solution {#Part1Solution}

This solution is a simple depth-first search (DFS) algorithm. Let's start by
parsing the input:

```python
with open("input.txt") as f:
	grid = (list(line.strip()) for line in f)
	grid = [list(map(int, line)) for line in grid]
```

I read the input file and convert it into a 2D list of integers.

Next, I find all the trailheads (cells with a value of `0`), and store them in a
dictionary,with the value initialized as `0`:

```python
trailheads: dict[Coordinate, int] = {
	(x, y): 0
	for y, row in enumerate(grid)
	for x, cell in enumerate(row)
	if cell == 0
}
```

I also find all the endpoints (cells with a value of `9`) and store them in a
list:

```python
nine_positions: list[Coordinate] = [
	(x, y) for y, row in enumerate(grid) for x, cell in enumerate(row) if cell == 9
]
```

I want to find the paths starting from each `9` cell that can reach a _distinct_
`0` cell. I could also have started from each `0` cell and found the paths that
reach a `9` cell, it works out the same.

```python
for position in nine_positions:
	find_paths_to_zero_one(position, grid, trailheads, set())
```

The `find_paths_to_zero_one` is my DFS function.

```python
def find_paths_to_zero_one(
    position: Coordinate,
    grid: Grid,
    trailheads: Dict[Coordinate, int],
    visited: set[Coordinate],
) -> None:
    if position in visited:
        return
    visited.add(position)
    if position in trailheads:
        trailheads[position] += 1
        return
    for direction in Directions:
        try:
            next_position = get_next_position(grid, position, direction)
            find_paths_to_zero_one(next_position, grid, trailheads, visited)
        except CoordinateError:
            pass
    return
```

Breaking down the function:

- `if position in visited: return` If I have already visited this cell, I
  return. This is because there could be multiple ways to reach from a `9` cell
  to a `0` cell. I only need to find one path for each of the `0` cells.
- `visited.add(position)` I add the current cell to the visited set.
- `if position in trailheads: trailheads[position] += 1` If the current cell is
  a `0` cell, I increment the value in the trailheads dictionary.
- `for direction in Directions:` Iterate over all the possible directions.
  - `next_position = get_next_position(grid, position, direction)` Get the next
    cell in the given direction.
  - `find_paths_to_zero_one(next_position, grid, trailheads, visited)`
    Recursively call the function with the next cell.
  - `except CoordinateError: pass` If the next cell is out of bounds, I catch
    the exception and continue.

My `get_next_position` function returns the next cell in the given direction if
it is legal, i.e. within the bounds of the grid, and the height difference is
`1`. It raises a `CoordinateError` if the next cell is out of bounds or the
height difference is not `1`, which is caught by my `find_paths` function.

```python
def get_next_position(
    grid: Grid, position: Coordinate, direction: Directions
) -> Coordinate:
    m, n = len(grid[0]), len(grid)
    value = grid[position[1]][position[0]]
    next_position = None
    match direction:
        case Directions.UP:
            if position[1] == 0:
                raise CoordinateError
            next_position = (position[0], position[1] - 1)
        case Directions.LEFT:
            if position[0] == 0:
                raise CoordinateError
            next_position = (position[0] - 1, position[1])
        case Directions.DOWN:
            if position[1] == n - 1:
                raise CoordinateError
            next_position = (position[0], position[1] + 1)
        case Directions.RIGHT:
            if position[0] == m - 1:
                raise CoordinateError
            next_position = (position[0] + 1, position[1])
    if value - grid[next_position[1]][next_position[0]] != 1:
        raise CoordinateError
    return next_position
```

Once I have found all the paths, I sum up the values in the trailheads and print
the score:

```python
print(f"LOG: score = { sum(trailheads.values()) }")
```

As always, I've only included the relevant parts of the code here, but to see my
full solution, you can check out my
[Advent of Code GitHub repository](https://github.com/VBenny42/AoC/blob/main/2024/day10/solution.py).

## Part 2 {#Part2}

For part 2, the score is calculated differently. The score is the number of
distinct trails that can be formed by starting from a `0` cell and ending at a
`9` cell. For example, the following map has `3` distinct trails, so it's score
is `3`

```
.....0.
..4321.
..5..2.
..6543.
..7..4.
..8765.
..9....
```

<div class="side-by-side">
    <div class="toleft">
<div class="language-plaintext highlighter-rouge"><div class="highlight"><pre class="highlight"><code>.....0.
..4321.
..5..2.
..6543.
..7..4.
..8765.
..9....
</code></pre></div></div>
    </div>

    <div class="toright">

<div class="language-plaintext highlighter-rouge"><div class="highlight"><pre class="highlight"><code>.....0.   .....0.   .....0.
..4321.   .....1.   .....1.
..5....   .....2.   .....2.
..6....   ..6543.   .....3.
..7....   ..7....   .....4.
..8....   ..8....   ..8765.
..9....   ..9....   ..9....</code></pre></div></div>
    </div>
<figcaption>Left: the map with all the trails shown. Right: Three trails shown separately</figcaption>
</div>

The score of the larger example map from part 1 is `81`, with the following
number of distinct trails that can be formed from each `0` cell: `20`, `24`,
`10`, `4`, `1`, `4`, `5`, `8`, and `5`.

Our task is to find the score of the given map, with the new scoring system.

### My Solution {#Part2Solution}

My solution for part 2 is part 1's solution with essentially one modification.
My `find_paths_to_zero_one` function was returning early if it found a cell it
had visited before. I removed this check, so that it finds all the paths from a
`9` cell to a `0` cell.

```python
def find_paths_to_zero_all(
    position: Coordinate,
    grid: Grid,
    trailheads: Dict[Coordinate, int],
) -> None:
    if position in trailheads:
        trailheads[position] += 1
        return
    for direction in Directions:
        try:
            next_position = get_next_position(grid, position, direction)
            find_paths_to_zero_all(next_position, grid, trailheads)
        except CoordinateError:
            pass
    return
```

Other than this, the rest of the code is the same as part 1.

Again, I've only included the relevant parts of the code here, check out my
[repository](https://github.com/VBenny42/AoC/blob/main/2024/day10/solution.py)
for the full solution.

---

That's it for day 10 of Advent of Code 2024! I hope you enjoyed reading my
solution and let's see how the rest of the month goes!
