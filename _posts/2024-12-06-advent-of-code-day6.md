---
title: "Advent of Code 2024 Day 6 – Guard Gallivant"
layout: post
date: 2024-12-06 13:54
image: /assets/images/favicon/apple-touch-icon.png
headerImage: false
tag:
    - python
    - advent-of-code
star: false
projects: true
category: blog
author: vineshbenny
description: Vinesh Benny's Advent of Code 2024 Day 6 solution in Python.

excerpt: "Vinesh Benny's Advent of Code 2024 Day 6 solution in Python."
seo_title: "Advent of Code 2024 Day 6 -- Guard Gallivant by Vinesh Benny"
seo_description: "Vinesh Benny's Advent of Code 2024 Day 6 solution in Python."
---

# Day 6: [Guard Gallivant](https://adventofcode.com/2024/day/6)

## Part 1

Today's puzzle is called "Guard Gallivant".
The input for today's puzzle is a map that has a guard and some obstacles.
This is the example input that was given.

```plaintext
....#.....
.........#
..........
..#.......
.......#..
..........
.#..^.....
........#.
#.........
......#...
```

The guard, `^`, in the sample input is currently going up, and the guard can only move in the direction it's facing.
When the guard hits an obstacle, `#`, they will turn right 90 degrees and continue moving.
This pattern will continue until the guard moves off the map, i.e., the guard is currently at an edge of the map and they are facing the edge.

In this example, the positions occupied by the guard until they move off the map are marked with `X`.

```plaintext
....#.....
....XXXXX#
....X...X.
..#.X...X.
..XXXXX#X.
..X.X.X.X.
.#XXXXXXX.
.XXXXXXX#.
#XXXXXXX..
......#X..
```

We can see that for the sample input, the guard will visit `41` distinct positions before moving off the map.

We need to find the number of distinct positions the guard will visit before moving off the map, given the input map.

### My Solution

So to solve this problem, I need to simulate the guard's movement.
I can start by parsing the input map.

```python
with open("input.txt") as f:
    grid = [list(line.strip()) for line in f]
```

Now I need to find the guard's starting position and direction.

```python
def get_starting_position(grid: list[list[str]]) -> Coordinate:
    for y, row in enumerate(grid):
        for x, cell in enumerate(row):
            if cell == "^":
                return Coordinate(x, y)
    raise ValueError("No starting position found")
```

This function loops through the grid and returns the coordinates of the guard's
starting position.
We know for a fact that the guard is always facing up in the input, so no need
to check for other directions.

Now we need to simulate the guard's movement.
To do this, at a given coordinate, I need to know:

-   The direction the guard is facing.
-   If the position in front of the guard is empty or an obstacle.

To keep track of the direction the guard is facing, I can use the `[cycle](https://docs.python.org/3/library/itertools.html#itertools.cycle)` iterator to provide an infinite iterator that goes through the directions that the guard will face.

```python
class Directions(Enum):
    UP = auto()
    RIGHT = auto()
    DOWN = auto()
    LEFT = auto()

directions = cycle(Directions)
current_direction = next(directions)
```

Everytime I need to change the direction, I can call `next(directions)` to get the next direction the guard will face.

Also, I need to keep track of the where the guard has visited already.
This can be done by just marking the position with an `X` when the guard visits it,
like the sample input answer.

To get the next position the guard will visit, I can use the current direction and the guard's current position.

```python
def get_next_position(
    grid_bounds: tuple[int, int], position: Coordinate, direction: Directions
) -> Coordinate:
    m, n = grid_bounds
    match direction:
        case Directions.UP:
            if position.y == 0:
                raise IndexError
            return Coordinate(position.x, position.y - 1)
        case Directions.LEFT:
            if position.x == 0:
                raise IndexError
            return Coordinate(position.x - 1, position.y)
        case Directions.DOWN:
            if position.y == n - 1:
                raise IndexError
            return Coordinate(position.x, position.y + 1)
        case Directions.RIGHT:
            if position.x == m - 1:
                raise IndexError
            return Coordinate(position.x + 1, position.y)
```

This function takes the grid bounds, the guard's current position, and the
direction the guard is facing.
If the guard is currently facing an edge, I raise an error that I make use of
later on.

Now I can finally simulate the guard's movement.

```python
def mark_guard_path(grid: list[list[str]], position: Coordinate) -> list[list[str]]:
    directions = cycle(Directions)
    current_direction = next(directions)
    next_position = None
    grid_bounds = len(grid[0]), len(grid)
    try:
        while True:
            next_position = get_next_position(grid_bounds, position, current_direction)
            if grid[next_position.y][next_position.x] == "#":
                current_direction = next(directions)
                continue
            grid[position.y][position.x] = "X"
            position = next_position
    # This is the signal that we have reached the end of the path
    except IndexError:
        grid[position.y][position.x] = "X"
        return grid
```

This function takes the grid and the guard's starting position, and returns the grid with the guard's path marked with `X`.
Breaking down the function:

-   `directions` is an infinite iterator that goes through the directions the guard will face.
-   `current_direction` is the current direction the guard is facing.
-   `next_position` is the next position the guard will visit.
-   `grid_bounds` is the bounds of the grid.
-   The `while` loop continues until the guard moves off the grid, i.e., an
    `IndexError` is raised by the `get_next_position` function.
-   `if grid[next_position.y][next_position.x] == "#":`
    If the guard is facing an obstacle, rotate 90 degrees and continue.
-   `grid[position.y][position.x] = "X"` marks the position the guard is currently at.
-   `position = next_position` moves the guard by 1 step.
-   `except IndexError:` When the guard moves off the grid, mark the last position and return the grid.

Now I can call this function with the input grid and the guard's starting position.

```python
starting_position = get_starting_position(grid)
marked_grid = mark_guard_path(grid, starting_position)
print(f"LOG: distinct positions = {sum(row.count('X') for row in marked_grid)}")
```

This code prints the number of distinct positions the guard will visit by counting the number of `X`s in the grid.

I've only included the relevant parts of the code here, but to see my full
solution, you can check out my [Advent of Code GitHub
repository](https://github.com/VBenny42/AoC/blob/main/2024/day06/solution.py).

## Part 2

For part 2, we need to count the number of ways we can add a single obstacle, so that the guard will be stuck in a loop.

For example, if we add an obstacle labeled `O` next to the guard's starting position:

```plaintext
....#.....
....+---+#
....|...|.
..#.|...|.
....|..#|.
....|...|.
.#.O^---+.
........#.
#.........
......#...
```

The guard will be stuck in a loop and visit the same positions over and over again.
For the example input, there are `6` ways to add an obstacle so that the guard will be stuck in a loop.

<div class="side-by-side">
    <div class="toleft">
<div class="language-plaintext highlighter-rouge"><div class="highlight"><pre class="highlight"><code>....#.....
....+---+#
....|...|.
..#.|...|.
..+-+-+#|.
..|.|.|.|.
.#+-^-+-+.
......O.#.
#.........
......#...</code></pre></div></div>
    </div>

    <div class="toright">

<div class="language-plaintext highlighter-rouge"><div class="highlight"><pre class="highlight"><code>....#.....
....+---+#
....|...|.
..#.|...|.
..+-+-+#|.
..|.|.|.|.
.#+-^-+-+.
.+----+O#.
#+----+...
......#...</code></pre></div></div>
    </div>
</div>

<div class="side-by-side">
    <div class="toleft">
<div class="language-plaintext highlighter-rouge"><div class="highlight"><pre class="highlight"><code>....#.....
....+---+#
....|...|.
..#.|...|.
..+-+-+#|.
..|.|.|.|.
.#+-^-+-+.
..|...|.#.
#O+---+...
......#...</code></pre></div></div>
    </div>

    <div class="toright">

<div class="language-plaintext highlighter-rouge"><div class="highlight"><pre class="highlight"><code>....#.....
....+---+#
....|...|.
..#.|...|.
..+-+-+#|.
..|.|.|.|.
.#+-^-+-+.
....|.|.#.
#..O+-+...
......#...</code></pre></div></div>
    </div>
</div>

```plaintext
....#.....
....+---+#
....|...|.
..#.|...|.
..+-+-+#|.
..|.|.|.|.
.#+-^-+-+.
.+----++#.
#+----++..
......#O..
```

### My Solution

To calculate the number of ways we can add an obstacle so that the guard will
be stuck in a loop, I can add an obstacle to every position on the map, and
check if that induces a loop.

I can't add an obstacle to the guard's starting position, so I can skip that.

```python
def find_loops(grid: list[list[str]], position: Coordinate) -> int:
    sum = 0
    for y, row in enumerate(grid):
        for x, cell in enumerate(row):
            if cell == "#":
                continue
            if cell == "^":
                continue
            sum += induce_loop(grid, Coordinate(x, y), position)
    return sum
```

This function loops through the grid, adds an obstacle to every position that
is not originally an obstacle or the starting position, and checks if a loop is
found with the addition.

To check if a loop is found on adding an obstacle, I can use the same logic as
part 1 to simulate the guard's movement, but I need to keep track of their
direction when marking a position as visited.

```python
def does_induce_loop(
    grid: list[list[str]], possible_obstruction: Coordinate, position: Coordinate
) -> bool:
    grid_copy: list = [row.copy() for row in grid]
    grid_copy[possible_obstruction.y][possible_obstruction.x] = "#"
    directions = cycle(Directions)
    current_direction = next(directions)
    next_position = None
    grid_bounds = len(grid[0]), len(grid)
    try:
        while True:
            next_position = get_next_position(grid_bounds, position, current_direction)
            if grid_copy[next_position.y][next_position.x] == "#":
                current_direction = next(directions)
                continue
            if type(grid_copy[position.y][position.x]) == set:
                if current_direction.value in grid_copy[position.y][position.x]:
                    return True
                grid_copy[position.y][position.x].add(current_direction.value)
            else:
                grid_copy[position.y][position.x] = {current_direction.value}
            position = next_position
    # Reached an edge of the grid, no loop found
    except IndexError:
        return False
```

Breaking down the function:

-   `grid_copy` is a copy of the original grid.
    A copy is used as the grid is being modified for every possible obstruction,
    and I don't want to modify the original grid.
-   `grid_copy[possible_obstruction.y][possible_obstruction.x] = "#"`
    Adds the possible obstruction to the grid.
-   Everything until `if type(grid_copy[position.y][position.x]) == set:`
    is the same as part 1.
-   `if type(grid_copy[position.y][position.x]) == set:`
    If the guard has visited the position already.
    -   `if current_direction.value in grid_copy[position.y][position.x]:`
        If the guard has visited the position in the same direction before, a
        loop is found.
    -   `grid_copy[position.y][position.x].add(current_direction.value)`
        Add the direction the guard is facing to the set of directions the
        guard has visited the position in.
-   `else:` Guard has not visited the position before. Create a set with the
    direction the guard is facing.
-   `position = next_position` Move the guard by 1 step.
-   `except IndexError:` Guard moves off the grid, no loop found, return `False`.

Using this function, I can check if a loop is found for every possible obstruction.
As this will be a time-consuming process, I can use the `multiprocessing`
module to parallelize the process.

I need to define a worker function that takes the grid, the possible
obstruction, and the guard position and calls the `does_induce_loop` function.

```python
def worker(task):
    grid, cell, position = task
    return does_induce_loop(grid, cell, position)
```

Now I can use the `Pool` class from the `multiprocessing` module to parallelize the process.

```python
def find_loops_multiprocessing(grid: list[list[str]], position: Coordinate) -> int:
    tasks = (
        (grid, Coordinate(x, y), position)
        for y, row in enumerate(grid)
        for x, cell in enumerate(row)
        if cell not in {"#", "^"}
    )

    with Pool() as pool:
        results = pool.map(worker, tasks)

    return sum(results)
```

This function creates a generator of tasks, where each task is a tuple of the
grid, the possible obstruction, and the guard's starting position.
The `Pool` class is used to parallelize the process, and the `worker` function
is called with each task.

This function was faster (obviously) than the non-parallelized version, when I tested it.

```plaintext
LOG: ways to induce a loop = 1888
Function 'main2' executed in 59.7202s
LOG: ways to induce a loop = 1888
Function 'main3' executed in 13.8823s
```

`main2` is the non-parallelized version, and `main3` is the parallelized version.
I know I could probably optimize my algorithm but I'm happy with the performance I'm getting.

Again, I've only included the relevant parts of the code here, check out my [repository](https://github.com/VBenny42/AoC/blob/main/2024/day06/solution.py) for the full solution.

---

That's it for day 6 of Advent of Code 2024! I hope you enjoyed reading my solution and let's see how the rest of the month goes!
