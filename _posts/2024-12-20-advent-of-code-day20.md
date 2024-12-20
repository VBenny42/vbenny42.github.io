---
title: "Advent of Code 2024 Day 20 – Race Condition"
layout: post
date: 2024-12-20 15:48
image: /assets/images/2024-12-14/christmas_tree_scaled.png
headerImage: true
tag:
    - python
    - advent-of-code
star: false
projects: true
category: blog
author: vineshbenny
description: Vinesh Benny's Advent of Code 2024 Day 20 solution in Python.

excerpt: "Vinesh Benny's Advent of Code 2024 Day 20 solution in Python."
seo_title: "Advent of Code 2024 Day 20 -- Race Condition by Vinesh Benny"
seo_description: "Vinesh Benny's Advent of Code 2024 Day 20 solution in Python."
---

# Day 20: [Race Condition](https://adventofcode.com/2024/day/20)

## Part 1 {#Part1}

For today's puzzle, we have a map of a racetrack and programs are competing to
see who finishes the fastest. Looking at an example:

```
###############
#...#...#.....#
#.#.#.#.#.###.#
#S#...#.#.#...#
#######.#.#.###
#######.#.#...#
#######.#.###.#
###..E#...#...#
###.#######.###
#...###...#...#
#.#####.#.###.#
#.#...#.#.#...#
#.#.#.#.#.#.###
#...#...#...###
###############
```

All the programs start at `S` and the finish line is at `E`, the track cells are
those marked as `.`, and the walls are marked as `#`.

When a program runs through the track, it can only move in the four cardinal
directions, and every move costs `1` unit of time. Since the racetrack is a
_single_ path, normally all the programs would finish at the same time.
Therefore, the fastest time to finish the example racetrack is `84` picoseconds.

However, each program is allowed to cheat _exactly once_, by disabling
collisions for up to `2` picoseconds. This allows a program to pass through
walls as if they were the track cells. After `2` picoseconds have passed, if the
program is still inside a wall, it will be disqualified.

So a program could finish the race in `72` picoseconds (saving `12` picoseconds)
by cheating for the two moves marked `1` and `2` below:

```
###############
#...#...12....#
#.#.#.#.#.###.#
#S#...#.#.#...#
#######.#.#.###
#######.#.#...#
#######.#.###.#
###..E#...#...#
###.#######.###
#...###...#...#
#.#####.#.###.#
#.#...#.#.#...#
#.#.#.#.#.#.###
#...#...#...###
###############
```

This cheat saves `64` picoseconds and takes the program directly to the end:

```
###############
#...#...#.....#
#.#.#.#.#.###.#
#S#...#.#.#...#
#######.#.#.###
#######.#.#...#
#######.#.###.#
###..21...#...#
###.#######.###
#...###...#...#
#.#####.#.###.#
#.#...#.#.#...#
#.#.#.#.#.#.###
#...#...#...###
###############
```

Each cheat has a distinct start position (the position where the cheat is
activated, just before the first move that is allowed to go through walls) and
end position; cheats are uniquely identified by their start position and end
position.

In this example, the total number of cheats (grouped by the amount of time they
save) are as follows:

- There are `14` cheats that save `2` picoseconds.
- There are `14` cheats that save `4` picoseconds.
- There are `2` cheats that save `6` picoseconds.
- There are `4` cheats that save `8` picoseconds.
- There are `2` cheats that save `10` picoseconds.
- There are `3` cheats that save `12` picoseconds.
- There is `1` cheat that saves `20` picoseconds.
- There is `1` cheat that saves `36` picoseconds.
- There is `1` cheat that saves `38` picoseconds.
- There is `1` cheat that saves `40` picoseconds.
- There is `1` cheat that saves `64` picoseconds.

Our task is to count the number of cheats for a given map that would save _at
least_ `100` picoseconds.

### My Solution {#Part1Solution}

To solve this problem, I thought about how I could find valid cheats. Since
there is only one valid path, I could use a breadth-first search to find the
shortest path from the start to the end. I could then iterate through every cell
on the path and see if there's a single wall between the current cell and
another cell that should be later on in the path. I only need to check for a
single wall, as only one wall cell can be bypassed in `2` picoseconds.

First, I need to parse the input and find the start and end positions:

```python
with open("input.txt", "r", encoding="utf-8") as f:
    grid = [list(line.strip()) for line in f]

get_point = lambda value: [
    (x, y)
    for y in range(len(grid))
    for x in range(len(grid[0]))
    if grid[y][x] == value
][0]

start = get_point("S")
end = get_point("E")

grid[start[1]][start[0]] = "."
grid[end[1]][end[0]] = "."
```

I set the start and end positions to `.` so that they are considered as part of
the track. Next, I need to find the path from the start to the end.

```python
def bfs(grid: Grid, start: Coord, end: Coord) -> list[Coord]:
    q = deque([(start, [start])])
    visited = set()

    while q:
        (x, y), path = q.popleft()
        if (x, y) == end:
            return path

        if (x, y) in visited:
            continue
        visited.add((x, y))

        for dx, dy in [(0, 1), (0, -1), (1, 0), (-1, 0)]:
            new_y, new_x = y + dy, x + dx
            if 0 <= new_y < len(grid) and 0 <= new_x < len(grid[0]):
                if grid[new_y][new_x] != "#":
                    q.append(((new_x, new_y), path + [(new_x, new_y)]))

    return []
```

This is just a standard breadth-first search algorithm, that returns the first
path it finds from the start to the end. Which in this case is the only path
that can be found.

Now that I have the valid path, I can find the distance from the start to every
cell on the path.

```python
path = {cell: i for i, cell in enumerate(normal_path)}
```

After this, I can find all the cheats that make sense to count.

```python
def find_cheats(grid: Grid, path: dict[Coord, int]):
    directions = [(0, 1), (1, 0), (0, -1), (-1, 0)]

    def in_bounds(x, y):
        return 0 <= y < len(grid) and 0 <= x < len(grid[0])

    cheats = set()
    for cell in path.keys():
        x, y = cell

        for dx, dy in directions:
            nx, ny = x + dx, y + dy
            if in_bounds(nx, ny) and grid[ny][nx] == "#":
                nx, ny = nx + dx, ny + dy
                if (
                    in_bounds(nx, ny)
                    and grid[ny][nx] == "."
                    and path[(nx, ny)] > path[cell]
                ):
                    cheats.add((cell, (nx, ny)))

    return cheats
```

Explanation:

- I iterate through every cell on the path.
- For every cell, I check the four cardinal directions.
  - If the next cell in the direction is a wall, I check the cell after that.
  - If the cell after the wall is a track cell and is further down the path, I
    add the cheat to the set.

Now, to calculate the savings of a cheat,

```python
def calculate_savings(path: dict[Coord, int], cheat: tuple[Coord, Coord]):
    return path[cheat[1]] - path[cheat[0]] - 2
```

I subtract `2` from the difference in the path lengths to account for the two
picoseconds when the cheat is active.

Putting it all together:

```python
def main1():
    with open("input.txt", "r", encoding="utf-8") as f:
        grid = [list(line.strip()) for line in f]

    get_point = lambda value: [
        (x, y)
        for y in range(len(grid))
        for x in range(len(grid[0]))
        if grid[y][x] == value
    ][0]

    start = get_point("S")
    end = get_point("E")

    grid[start[1]][start[0]] = "."
    grid[end[1]][end[0]] = "."

    normal_path = bfs(grid, start, end)

    path = {cell: i for i, cell in enumerate(normal_path)}

    cheats = find_cheats(grid, path)

    threshold = 100

    savings = sum(1 for cheat in cheats if calculate_savings(path, cheat) >= threshold)

    print(f"ANSWER: { savings = }")
```

As always, I've only included the relevant parts of the code here, but to see my
full solution, you can check out my
[Advent of Code GitHub repository](https://github.com/VBenny42/AoC/blob/main/2024/day20/solution.py).

## Part 2 {#Part2}

For part 2, it turns out that the programs can cheat for _even longer_! They can
now cheat for up to `20` picoseconds. The task is the same as part 1, but now we
we have to find cheats that can last up to `20` picoseconds.

Looking at the example from part 1, this six-picosecond saves `76` picoseconds:

```
###############
#...#...#.....#
#.#.#.#.#.###.#
#S#...#.#.#...#
#1#####.#.#.###
#2#####.#.#...#
#3#####.#.###.#
#456.E#...#...#
###.#######.###
#...###...#...#
#.#####.#.###.#
#.#...#.#.#...#
#.#.#.#.#.#.###
#...#...#...###
###############
```

This is also a valid cheat that saves `76` picoseconds:

```
###############
#...#...#.....#
#.#.#.#.#.###.#
#S12..#.#.#...#
###3###.#.#.###
###4###.#.#...#
###5###.#.###.#
###6.E#...#...#
###.#######.###
#...###...#...#
#.#####.#.###.#
#.#...#.#.#...#
#.#.#.#.#.#.###
#...#...#...###
###############
```

Since both of the cheats have the same start and end positions, they are counted
as the same cheat. Cheats don't have to take up the full `20` picoseconds, but
still the cheat can only be activated once.

### My Solution {#Part2Solution}

My basic idea is the same as part 1, however I need to find cheats differently.
Cheats can now bypass multiple walls, and for that matter, cheats also don't
have to go through walls at all, even though it would make no sense to do so.

To find start and end positions now, I start looking for neighbors that are the
[Manhattan distance](https://en.wikipedia.org/wiki/Taxicab_geometry) away from a
cell on the path. The cheats for every cheat duration correspond to the
neighbors that are of that distance away from the cell. I could also have used
this for part 1 too, but my original solution worked for that. For part 2 that
solution was a bit too slow :/.

Finding the manhattan neighbors:

```python
def manhattan_neighbors(
    coord: Coord, grid_set: set[Coord], cheat_length: int
) -> set[Coord]:
    possible_neighbors = set()
    x, y = coord
    for dx in range(-cheat_length, cheat_length + 1):
        dy = cheat_length - abs(dx)
        possible_neighbors.add((x + dx, y + dy))
        possible_neighbors.add((x + dx, y - dy))
    return possible_neighbors.intersection(grid_set)
```

`grid_set` is a set of all the cells' coordinates on the grid. I intersect the
set of possible neighbors with the grid set to get the valid neighbors only.

Now, to find the savings of a cheat:

```python
savings = 0

for cell in path.keys():
    for i in range(1, 21):
        for neighbor in manhattan_neighbors(cell, grid_set, i):
            if (path[neighbor] - path[cell]) - manhattan_distance(
                cell, neighbor
            ) >= threshold:
                savings += 1
```

The savings calculation is the same as part 1, but now I iterate through every
duration of the cheat from `1` to `20` and every neighbor of the cell that is of
that distance away. The `manhattan_distance` function is just the Manhattan
distance between two cells.

```python
def manhattan_distance(coord1: Coord, coord2: Coord) -> int:
    return abs(coord1[0] - coord2[0]) + abs(coord1[1] - coord2[1])
```

Putting it all together:

```python
def main2():
    with open("input.txt", "r", encoding="utf-8") as f:
        grid = [list(line.strip()) for line in f]

    get_point = lambda value: [
        (x, y)
        for y in range(len(grid))
        for x in range(len(grid[0]))
        if grid[y][x] == value
    ][0]

    start = get_point("S")
    end = get_point("E")

    grid_set = {
        (x, y)
        for y, row in enumerate(grid)
        for x, cell in enumerate(row)
        if cell != "#"
    }

    normal_path = bfs(grid, start, end)

    path = {cell: i for i, cell in enumerate(normal_path)}

    savings = 0
    threshold = 100

    for cell in path.keys():
        for k in range(1, 21):
            for neighbor in manhattan_neighbors(cell, grid_set, k):
                if (path[neighbor] - path[cell]) - manhattan_distance(
                    cell, neighbor
                ) >= threshold:
                    savings += 1

    print(f"ANSWER: { savings = }")
```

Again, I've only included the relevant parts of the code here, check out my
[repository](https://github.com/VBenny42/AoC/blob/main/2024/day20/solution.py)
for the full solution.

---

That's it for day 20 of Advent of Code 2024! I hope you enjoyed reading my
solution and let's see how the rest of the month goes!
