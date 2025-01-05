---
title: "Advent of Code 2024 Day 18 – RAM Run"
layout: post
date: 2024-12-18 18:12
image: /assets/images/2024-12-14/christmas_tree_scaled.png
headerImage: true
tag:
    - python
    - advent-of-code
star: false
projects: true
category: blog
author: vineshbenny
description: Vinesh Benny's Advent of Code 2024 Day 18 solution in Python.

excerpt: "Vinesh Benny's Advent of Code 2024 Day 18 solution in Python."
seo_title: "Advent of Code 2024 Day 18 -- RAM Run by Vinesh Benny"
seo_description: "Vinesh Benny's Advent of Code 2024 Day 18 solution in Python."
# %s/\v([D|d]ay.{0,1})11/\118/g
# %s/\vRAM Run/RAM Run/g
---

# Day 18: [RAM Run](https://adventofcode.com/2024/day/18)

## Part 1 {#Part1}

Today's puzzle involves a grid (yet again) and obstacles that are falling onto
the grid. Let's look at an example:

```
5,4
4,2
4,5
3,0
2,1
6,3
2,4
1,5
0,6
3,3
2,6
5,1
1,2
5,5
2,5
6,5
1,4
0,4
6,4
1,1
6,1
1,0
0,5
1,6
2,0
```

Each obstacle from this list is going to fall onto a `7` by `7` grid. We want to
get from the top-left corner `(0,0)`, to the bottom-right corner which is
`(6,6)` for this grid. Everytime an obstacle falls, we cannot use the cell that
the obstacle falls on as a cell for our path from corner to corner. If we look
at the grid after the first `12` obstacles have fallen, marking their cells with
`#` and safe cells as `.`:

```
...#...
..#..#.
....#..
...#..#
..#..#.
.#..#..
#.#....
```

We can take steps up, down, left or right, but not diagonally. After the first
`12` obstacles have fallen, the shortest path from the top-left corner to the
bottom-right corner is `22` steps. One such example of the shortest path is,
with the cells used marked as `O`:

```
OO.#OOO
.O#OO#O
.OOO#OO
...#OO#
..#OO#.
.#.O#..
#.#OOOO
```

Our goal is to find the length of the shortest path from the top-left corner to
the bottom right corner of a `71` by `71` grid after the first `1024` obstacles
have fallen.

### My Solution {#Part1Solution}

To solve this problem, I can use my Dijkstra's algorithm implementation from
[day 16]({{ site.baseurl }}{% link _posts/2024-12-16-advent-of-code-day16.md
%}#Part1Solution). I can use the same idea of a `graph` where each cell is a
node and the edges are the possible moves to get to one cell from the other. In
this case however, the graph is not weighted. Also, my neighbors function needs
to be slightly modified, since for day 16 cells could be visited differently
depending on direction, for this one it doesn't matter.

So my only two changes are to the `cost` and `neighbors` functions passed in to
the `Dijkstra` class.

```python
def neutral_cost_fn(cell1: Coordinate, cell2: Coordinate) -> int:
    return 1
```

```python
def adjacent_neighbors_fn(
    cell: Coordinate,
    grid: Grid,
    width: int,
    height: int,
) -> list[Coordinate]:
    neighbors = []
    if cell[0] > 0 and grid[cell[1]][cell[0] - 1] == ".":
        neighbors.append((cell[0] - 1, cell[1]))
    if cell[0] < width - 1 and grid[cell[1]][cell[0] + 1] == ".":
        neighbors.append((cell[0] + 1, cell[1]))
    if cell[1] > 0 and grid[cell[1] - 1][cell[0]] == ".":
        neighbors.append((cell[0], cell[1] - 1))
    if cell[1] < height - 1 and grid[cell[1] + 1][cell[0]] == ".":
        neighbors.append((cell[0], cell[1] + 1))
    return neighbors
```

The above function checks if the cell is within the bounds of the grid and
returns all the cells that are adjacent to the current cell and are safe to move
to.

I've defined a `neutral_cost_fn` that always returns `1` as the cost of moving
from one cell to another. This is because the grid is not weighted, so the cost
should always be the same.

Other than that, I can use the same `Dijkstra` class from day 16 to solve this
problem.

```python
def main1():
    with open("input.txt", "r", encoding="utf-8") as f:
        falling_bytes: list[Coordinate] = [
            (int(x), int(y)) for line in f for x, y in [line.strip().split(",")]
        ]

    width, height = 71, 71

    grid = [["." for _ in range(width)] for _ in range(height)]

    for x, y in falling_bytes[:1024]:
        grid[y][x] = "#"

    start: Coordinate = (0, 0)
    end: Coordinate = (width - 1, height - 1)

    dijkstra = Dijkstra(
        lambda cell: adjacent_neighbors_fn(cell, grid, width, height),
        neutral_cost_fn,
        0.0,
        float("inf"),
    )

    min_cost = float("inf")
    dijkstra.find_path(start)
    min_cost = dijkstra.get_cost(end)

    print(f"LOG: Least cost path { int(min_cost) }")
```

As always, I've only included the relevant parts of the code here, but to see my
full solution, you can check out my
[Advent of Code GitHub repository](https://github.com/VBenny42/AoC/blob/main/2024/python/day18/solution.py).

## Part 2 {#Part2}

For part 2, eventually as more obstacles fall, there will be no path from the
top-left corner to the bottom-right corner. We need to find which obstacle is
the one that causes this to happen.

For the same grid as before, when the obstacle at `(1,1)` falls, there is still
a path that can be found.

```
O..#OOO
O##OO#O
O#OO#OO
OOO#OO#
###OO##
.##O###
#.#OOOO
```

But when the next obstacle at `(6,1)` falls, there is no path that can be found.

```
...#...
.##..##
.#..#..
...#..#
###..##
.##.###
#.#....
```

So for this example, the obstacle at `(6,1)` is the one that causes the path to
be blocked.

We need to find the first obstacle that causes the bottom-right corner to be
unreachable for a `71` by `71` grid.

### My Solution {#Part2Solution}

Again, I can use my `Dijkstra` class to solve this. I know that at least the
first `1024` obstacles do not cause the exit corner to be unreachable, so I need
to check from that point onwards. I can add an obstacle to the grid, check if
any path exists, and if not, that is the obstacle that causes the exit corner to
be unreachable.

```python
def main2():
    with open("input.txt", "r", encoding="utf-8") as f:
        falling_bytes: list[Coordinate] = [
            (int(x), int(y)) for line in f for x, y in [line.strip().split(",")]
        ]

    width, height = 71, 71

    grid = [["." for _ in range(width)] for _ in range(height)]

    for x, y in falling_bytes[:1024]:
        grid[y][x] = "#"

    start: Coordinate = (0, 0)
    end: Coordinate = (width - 1, height - 1)

    i = 1024
    while i < len(falling_bytes):
        falling_byte = falling_bytes[i]
        grid[falling_byte[1]][falling_byte[0]] = "#"

        dijkstra = Dijkstra(
            lambda cell: adjacent_neighbors_fn(cell, grid, width, height),
            neutral_cost_fn,
            0.0,
            float("inf"),
        )

        min_cost = float("inf")
        dijkstra.find_path(start)
        min_cost = dijkstra.get_cost(end)
        if min_cost == float("inf"):
            break

        i += 1

    print(f"LOG: Byte that breaks the path { falling_bytes[i] }")
```

Within the `while` loop, I add an obstacle to the grid, find the path from the
top-left corner to the bottom-right corner, and if the cost is `inf`, then no
path exists and I break out of the loop. The obstacle that caused the exit
corner to be unreachable is the one at `falling_bytes[i]`.

Again, I've only included the relevant parts of the code here, check out my
[repository](https://github.com/VBenny42/AoC/blob/main/2024/python/day18/solution.py)
for the full solution.

---

That's it for day 18 of Advent of Code 2024! I hope you enjoyed reading my
solution and let's see how the rest of the month goes!
