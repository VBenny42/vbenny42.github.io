---
title: "Advent of Code 2024 Day 16 – Reindeer Maze"
layout: post
date: 2024-12-16 21:39
image: /assets/images/2024-12-14/christmas_tree_scaled.png
headerImage: true
tag:
    - python
    - advent-of-code
star: false
projects: true
category: blog
author: vineshbenny
description: Vinesh Benny's Advent of Code 2024 Day 16 solution in Python.

excerpt: "Vinesh Benny's Advent of Code 2024 Day 16 solution in Python."
seo_title: "Advent of Code 2024 Day 16 -- Reindeer Maze by Vinesh Benny"
seo_description: "Vinesh Benny's Advent of Code 2024 Day 16 solution in Python."
---

# Day 16: [Reindeer Maze](https://adventofcode.com/2024/day/16)

## Part 1

The puzzle for today is a maze traversal problem. A reindeer starts on the Start
tile (marked `S`) and needs to make its way to the End tile (marked `E`) with
the shortest cost. They can move forward `1` space, which costs `1` point, or
they can rotate `90` degrees left or right, which costs `1000` points.

Looking at a small example:

```
###############
#.......#....E#
#.#.###.#.###.#
#.....#.#...#.#
#.###.#####.#.#
#.#.#.......#.#
#.#.#####.###.#
#...........#.#
###.#.#####.#.#
#...#.....#.#.#
#.#.#.###.#.#.#
#.....#...#.#.#
#.###.#.#.#.#.#
#S..#.....#...#
###############
```

Any of the shortest paths from `S` to `E` would have a total cost of `7036`.
This can be achived by rotating `7` times, and taking `1` step forward `36`
times.

```
###############
#.......#....E#
#.#.###.#.###^#
#.....#.#...#^#
#.###.#####.#^#
#.#.#.......#^#
#.#.#####.###^#
#..>>>>>>>>v#^#
###^#.#####v#^#
#>>^#.....#v#^#
#^#.#.###.#v#^#
#^....#...#v#^#
#^###.#.#.#v#^#
#S..#.....#>>^#
###############
```

Our puzzle input is a larger maze, but with the same rules. We need to find the
cost of the shortest path from `S` to `E`.

### My Solution

This problem is a classic example of a wieghted shortest path problem. To solve
this I used a modified version of
[Dijkstra's algorithm](https://en.wikipedia.org/wiki/Dijkstra%27s_algorithm). I
used a priority queue to keep track of the nodes to visit, and a dictionary to
keep track of the costs to visit each cell.

First, I read in the input and find the start and end cells.

```python
with open("input.txt", "r", encoding="utf-8") as f:
	grid = [list(line.strip()) for line in f]

height, width = len(grid), len(grid[0])

start: State | None = None
ends: list[State] | None = None

for y, row in enumerate(grid):
	for x, cell in enumerate(row):
		if cell == "S":
			start = State(x, y, Direction.RIGHT)
		if cell == "E":
			ends = [
				State(x, y, Direction.UP),
				State(x, y, Direction.RIGHT),
				State(x, y, Direction.DOWN),
				State(x, y, Direction.LEFT),
			]
```

My `State` class is just a named tuple that holds the x and y coordinates of the
cell that the reindeer is on, and the direction the reindeer is facing.

Now looking at my `Dijkstra` class:

```python
class Dijkstra:
    def __init__(
        self,
        neighbors_fn: Callable[[State], list[State]],
        cost_fn: Callable[[State, State], float],
        min_cost: float,
        max_cost: float,
    ):
        self.cost_function = cost_fn
        self.neighbors_function = neighbors_fn
        self.previous = {}
        self.costs = {}
        self.min_cost = min_cost
        self.max_cost = max_cost

    def find_path(self, start: State):
        queue = []
        queue.append([0, start])
        self.previous = {}
        self.costs = {}
        self.costs[start] = self.min_cost
        self.previous[start] = []

        while queue:
            _, current = heapq.heappop(queue)

            for neighbor in self.neighbors_function(current):
                new_cost = self.costs[current] + self.cost_function(current, neighbor)

                if neighbor not in self.costs or new_cost < self.costs[neighbor]:
                    self.costs[neighbor] = new_cost
                    heapq.heappush(queue, [new_cost, neighbor])
                    self.previous[neighbor] = [current]

                elif new_cost == self.costs[neighbor]:
                    self.previous[neighbor].append(current)

    def get_cost(self, end: State) -> float:
        if end not in self.costs:
            return self.max_cost

        return self.costs[end]

    def get_paths(self, end: State) -> list[State]:
        path = []
        stack = [end]

        while stack:
            current = stack.pop()
            path.append(current)
            for previous in self.previous[current]:
                stack.append(State(*previous))

        return path
```

I'll go over each function in the class:

1. `__init__`: This function initializes the class with the cost function, the
   neighbors function, the minimum cost, and the maximum cost. The neighbors
   function should return all the valid neighbors that can be visited from the
   current state. The cost function should return the cost to move from one
   state to another state.
2. `find_path`: This function finds the shortest path from the start state to
   all the other states and stores the paths in the `previous` dictionary. It
   uses a priority queue to keep track of the states to visit. It also keeps
   track of the cost to visit each state.
3. `get_cost`: This function returns the cost to visit a particular state from
   the start state.
4. `get_paths`: This function returns the path to visit a particular state from
   the start state.

Now, I define the cost function and the neighbors function:

```python
def neighbors_fn(
    cell: State,
    grid: Grid,
    width: int,
    height: int,
) -> list[State]:
    neighbors = []
    for direction in Direction:
        if direction == cell.direction:
            continue
        neighbors.append(State(cell.x, cell.y, direction))
    match cell.direction:
        case Direction.UP:
            if cell.y > 0 and grid[cell.y - 1][cell.x] != "#":
                neighbors.append(State(cell.x, cell.y - 1, cell[2]))
        case Direction.RIGHT:
            if cell.x < width - 1 and grid[cell.y][cell.x + 1] != "#":
                neighbors.append(State(cell.x + 1, cell.y, cell[2]))
        case Direction.DOWN:
            if cell.y < height - 1 and grid[cell.y + 1][cell.x] != "#":
                neighbors.append(State(cell.x, cell.y + 1, cell[2]))
        case Direction.LEFT:
            if cell.x > 0 and grid[cell.y][cell.x - 1] != "#":
                neighbors.append(State(cell.x - 1, cell.y, cell[2]))
    return neighbors
```

This function returns all the rotations that the reindeer can make from the
current state, as well as all the adjacent cells that the reindeer can move to.

```python
def cost_fn(cell1: State, cell2: State) -> int:
    if cell1.x == cell2.x and cell1.y == cell2.y:
        if cell1.direction == cell2.direction:
            return 0
        if cell1.direction in (Direction.UP, Direction.DOWN):
            if cell2.direction in (Direction.UP, Direction.DOWN):
                return 2000
            return 1000
        if cell2.direction in (Direction.UP, Direction.DOWN):
            return 1000
        return 2000
    return 1
```

The cost function returns `0` is the two states are the same, `2000` if the two
states are 180-degree rotations of each other, `1000` if the states are
90-degree rotations of each other, and `1` if the state transition is a simple
move forward.

Now that I have all the functions defined, I can create an instance of the class
and find the cost of the shortest path from the start state to the end state.

```python
def main1():
    with open("input.txt", "r", encoding="utf-8") as f:
        grid = [list(line.strip()) for line in f]

    height, width = len(grid), len(grid[0])

    start: State | None = None
    ends: list[State] | None = None

    for y, row in enumerate(grid):
        for x, cell in enumerate(row):
            if cell == "S":
                start = State(x, y, Direction.RIGHT)
            if cell == "E":
                ends = [
                    State(x, y, Direction.UP),
                    State(x, y, Direction.RIGHT),
                    State(x, y, Direction.DOWN),
                    State(x, y, Direction.LEFT),
                ]

    dijkstra = Dijkstra(
        lambda cell: neighbors_fn(cell, grid, width, height),
        cost_fn,
        0.0,
        float("inf"),
    )

    min_cost = float("inf")
    dijkstra.find_path(start)
    for end in ends:
        min_cost = min(min_cost, dijkstra.get_cost(end))

    print(f"LOG: Least cost path { int(min_cost) }")
```

This will print the cost of the shortest path from the start state to the end
state. I have `ends` as a list of all the possible end states, as we don't know
which direction the reindeer will be facing when it reaches the end state. I use
a `lambda` function to pass the `neighbors_fn` function to the `Dijkstra` class,
since the `neighbors_fn` function requires the `grid`, `width`, and `height` but
this is not needed within the class calls.

As always, I've only included the relevant parts of the code here, but to see my
full solution, you can check out my
[Advent of Code GitHub repository](https://github.com/VBenny42/AoC/blob/main/2024/day16/solution.py).

## Part 2

For part 2, we need to find the number of cells that are on _any_ shortest path
from the start to the end.

Looking at the example maze, with the cells on any of the shortest paths marked
as `O`:

```
###############
#.......#....O#
#.#.###.#.###O#
#.....#.#...#O#
#.###.#####.#O#
#.#.#.......#O#
#.#.#####.###O#
#..OOOOOOOOO#O#
###O#O#####O#O#
#OOO#O....#O#O#
#O#O#O###.#O#O#
#OOOOO#...#O#O#
#O###.#.#.#O#O#
#O..#.....#OOO#
###############
```

The number of cells on any of the shortest paths for this example is `45`.

### My Solution

To solve this problem, my Dijsktra class already stores the paths to all the
states from the start state. I can use this information to find all the paths
that have the same cost as the shortest path.

```python
def main2():
	# Same code as main1

    min_cost = float("inf")
    end_state: State | None = None
    dijkstra.find_path(start)
    for end in ends:
        cost = dijkstra.get_cost(end)
        if cost < min_cost:
            min_cost = cost
            end_state = State(*end)

    tiles_on_all_paths = set()
    for node in dijkstra.get_paths(end_state):
        tiles_on_all_paths.add((node.x, node.y))

    print(f"LOG: Tiles on all paths {len(tiles_on_all_paths)}")
```

The changes from `main1` start from the `for end in ends` loop.

- In the for loop, I find the minimum cost of all the end states and store the
  end state with the minimum cost.
- `tiles_on_all_paths` is a set that stores all the cells that are on all the
  shortest paths from the start state to the end state.
- `for node in dijkstra.get_paths(end_state):` loops through all the states on
  the shortest path from the start state to the end state and adds the x and y
  coordinates of the state to the `tiles_on_all_paths` set.
- Finally, I print the length of the `tiles_on_all_paths` set.

Again, I've only included the relevant parts of the code here, check out my
[repository](https://github.com/VBenny42/AoC/blob/main/2024/day16/solution.py)
for the full solution.

---

That's it for day 16 of Advent of Code 2024! I hope you enjoyed reading my
solution and let's see how the rest of the month goes!
