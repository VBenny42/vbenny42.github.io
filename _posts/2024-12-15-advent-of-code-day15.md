---
title: "Advent of Code 2024 Day 15 – Warehouse Woes"
layout: post
date: 2024-12-15 21:30
image: /assets/images/2024-12-14/christmas_tree_scaled.png
headerImage: true
tag:
    - python
    - advent-of-code
star: false
projects: true
category: blog
author: vineshbenny
description: Vinesh Benny's Advent of Code 2024 Day 15 solution in Python.

excerpt: "Vinesh Benny's Advent of Code 2024 Day 15 solution in Python."
seo_title: "Advent of Code 2024 Day 15 -- Warehouse Woes by Vinesh Benny"
seo_description: "Vinesh Benny's Advent of Code 2024 Day 15 solution in Python."
---

# Day 15: [Warehouse Woes](https://adventofcode.com/2024/day/15)

## Part 1

Today's puzzle took me some time to figure out my bugs, but I eventually got it
working. The puzzle was about simulating a warehouse with a robot that can move
around. In the warehouse, there are boxes and walls, if the robot encounters a
wall, it will stop moving, but if it encounters a box, it will push it in the
direction it is moving. If pushing a box would cause it to move into a wall, the
robot leaves the box where it is and stops moving. The robot can also push
multiple boxes at once, same rules apply.

Our input file has a grid where the robot is represented by `@`, boxes are `O`,
and walls are `#`. The robot can move in four directions: up, down, left, and
right. It also has a list of directions the robot should move in from the
starting position. Looking at a small example:

```
########
#..O.O.#
##@.O..#
#...O..#
#.#.O..#
#...O..#
#......#
########

<^^>>>vv<v>>v<<
```

<script src="https://asciinema.org/a/1slO9NCY0dphNUHFiqU321FQ3.js" id="asciicast-1slO9NCY0dphNUHFiqU321FQ3" async="true" async data-start-at="2" data-speed="0.3"></script>
<figcaption class="caption">The robot moving around the warehouse.</figcaption>

In the example above, the final state of the warehouse is:

```
########
#....OO#
##.....#
#.....O#
#.#O@..#
#...O..#
#...O..#
########
```

After the final state is reached, we need to find the Goods Positioning System
(GPS) coordinate of each box. The GPS coordinate of a box is equal to 100 times
its distance from the top edge of the map plus its distance from the left edge
of the map. We want to find the sum of the GPS coordinates of all the boxes
after the robot has finished moving.

For the small example above, the sum of the GPS coordinates of all the boxes is
`2028`.

### My Solution

I broke down the problem into a few steps:

- Parse the input file and store the warehouse grid and the robot's directions.
- Find the robot's starting position.
- Simulate the robot's movement and update the warehouse grid.
- Find the GPS coordinates of each box and sum them up.

```python
def parse_input(lines: List[str]) -> tuple[Grid, List[Directions]]:
    i = 0
    warehouse_grid = []
    while lines[i] != "\n":
        warehouse_grid.append(list(lines[i].strip()))
        i += 1

    movements = [
        parse_movement(movement)
        for j in range(i + 1, len(lines))
        for movement in lines[j].strip()
    ]
    return warehouse_grid, movements
```

The `parse_input` function reads the input file and returns the warehouse grid,
and a list of directions the robot should move in. The `parse_movement` function
just converts the character directions to a tuple of `(dx, dy)`.

Finding the robot's starting position is simple, just iterate through the grid
until the `@` character is found. If the robot is not found, raise a
`ValueError`, can't continue without the robot.

```python
def find_robot(grid: Grid) -> Coordinate:
    for y, row in enumerate(grid):
        for x, spot in enumerate(row):
            if spot == "@":
                return (x, y)
    raise ValueError("Robot not found")
```

My `move_robot` function is where most of the logic is.

```python
def move_robot(grid: Grid, robot: Coordinate, direction: Directions) -> Coordinate:
    x, y = robot
    dx, dy = direction.value
    new_x, new_y = x + dx, y + dy
    if grid[new_y][new_x] == "#":
        return robot
    if grid[new_y][new_x] == ".":
        grid[y][x] = "."
        grid[new_y][new_x] = "@"
        return (new_x, new_y)
    if grid[new_y][new_x] == "O":
        try:
            boxes = boxes_to_move(grid, (new_x, new_y), direction)
        except EdgeError:
            return robot
        for box in boxes:
            box_x, box_y = box
            new_box_x, new_box_y = box_x + dx, box_y + dy
            grid[box_y][box_x] = "."
            grid[new_box_y][new_box_x] = "O"
        grid[y][x] = "."
        grid[new_y][new_x] = "@"
        return (new_x, new_y)
    raise ValueError("Spot is invalid")
```

Breaking down the function:

- `x, y = robot` gets the robot's current position.
- `dx, dy = direction.value` gets the direction the robot should move in.
- `new_x, new_y = x + dx, y + dy` Calculates the new position the robot should
  move to.
- `if grid[new_y][new_x] == "#":` Checks if the robot is moving into a wall. If
  so, return the robot's current position, since it can't move into the wall.
- `if grid[new_y][new_x] == ".":` Checks if the robot is moving into an empty
  spot. If so, update the grid and return the new position.
- `if grid[new_y][new_x] == "O":` Checks if the robot is moving into a box.
  - `boxes_to_move` is a helper function that returns a list of boxes the robot
    should move. If moving a box would cause it to move into a wall, the
    function raises an `EdgeError`. In this case, return the robot's current
    position.
  - `for box in boxes:` Move each box in the list of boxes.
  - `grid[y][x] = "." ... (new_x, new_y)` Update the robot's current position
    and return the new one.
- `raise ValueError("Spot is invalid")` Shouldn't reach here, but if it does,
  raise an error.

The `boxes_to_move` function is pretty simple.

```python
def boxes_to_move(
    grid: Grid, box: Coordinate, direction: Directions
) -> list[Coordinate]:
    x, y = box
    dx, dy = direction.value
    new_x, new_y = x + dx, y + dy
        raise EdgeError("Edge of the grid")
    if grid[new_y][new_x] == ".":
        return [box]
    if grid[new_y][new_x] == "O":
        return boxes_to_move(grid, (new_x, new_y), direction) + [box]
    return []
```

This function recursively finds all the boxes the robot should move in the given
direction. If the new position is empty, return the current box. If the new
position is a box, call the function again with the new position and add the
current box to the list. If the new position is a wall, raise an `EdgeError`,
since the box can't move into a wall.

Now we need to find all the boxes' positions and calculate the GPS coordinates.

```python
def find_boxes(grid: Grid) -> list[Coordinate]:
    return [
        (x, y)
        for y, row in enumerate(grid)
        for x, cell in enumerate(row)
        if cell in {"O", "["}
    ]

def calculate_gps_coordinate(box: Coordinate) -> int:
    return box[1] * 100 + box[0]
```

The `[` character is used in part 2.

Finally, we can put everything together and get the sum of the GPS coordinates.

```python
def main1():
    with open("input.txt", "r", encoding="utf-8") as f:
        lines = f.readlines()

    warehouse_grid, movements = parse_input(lines)
    robot = find_robot(warehouse_grid)

    for movement in movements:
        robot = move_robot(warehouse_grid, robot, movement)

    boxes = find_boxes(warehouse_grid)
    coordinates = sum(map(calculate_gps_coordinate, boxes))
    print(f"LOG: { coordinates = }")
```

As always, I've only included the relevant parts of the code here, but to see my
full solution, you can check out my
[Advent of Code GitHub repository](https://github.com/VBenny42/AoC/blob/main/2024/day15/solution.py).

## Part 2

For part 2, there's _another_ warehouse with a robot and boxes. This time, the
warehouse is twice as wide as part 1's warehouse. Looking at a warehouse:

```
##########				####################
#..O..O.O#				##....[]....[]..[]##
#......O.#				##............[]..##
#.OO..O.O#				##..[][]....[]..[]##
#..O@..O.#				##....[]@.....[]..##
#O#..O...#				##[]##....[]......##
#O..O..O.#				##[]....[]....[]..##
#.OO.O.OO#				##..[][]..[]..[][]##
#....O...#				##........[]......##
##########				####################
```

<figcaption class="caption">Left: Part 1 warehouse, Right: Part 2 scaled warehouse</figcaption>

The warehouse is scaled up so that:

- If the tile is `#`, the new map contains `##` instead.
- If the tile is `O`, the new map contains `[]` instead.
- If the tile is `.`, the new map contains `..` instead.
- If the tile is `@`, the new map contains `@.` instead.

Since the robot is still 1 character wide, but boxes are 2 characters wide,
boxes can be aligned such that they directly push two other boxes at once. For
example, let's look at the warehouse above:

<script src="https://asciinema.org/a/n9vH8ZCYuGydTGKG7m9Ahr3Qo.js" id="asciicast-n9vH8ZCYuGydTGKG7m9Ahr3Qo" async="true" async data-speed="0.3"></script>

The new ways that boxes can be aligned are with the robot moving upwards for
this example:

```
[]..	..[]	[][]
.[].	.[].	.[].
.@..	.@..	..@..
```

In all of these cases, the robot will move all the boxes in the direction
upwards, given that every box can move in the direction. This is just like in
part 1, where if any box would move into a wall the boxes shouldn't be moved.

The GPS coordinates are calculated the same way as in part 1, but the box
position to be used is the left side of the box.

Our task remains the same, find the sum of the GPS coordinates of all the boxes
after the robot has finished moving.

### My Solution

My main logic is the same as in part 1, but my `boxes_to_move` function is a bit
more complex.

```python
def boxes_to_move2(
    grid: Grid, box: Tuple[Coordinate, Coordinate], direction: Directions
) -> list[tuple[Coordinate, Coordinate]]:
    dx, dy = direction.value
    if direction in (Directions.LEFT, Directions.RIGHT):
        x, y = box[0] if direction == Directions.LEFT else box[1]
        bracket = "]" if direction == Directions.LEFT else "["
        new_x, new_y = x + dx, y + dy
        adjacent_box = (
            ((new_x - 1, new_y), (new_x, new_y))
            if direction == Directions.LEFT
            else ((new_x, new_y), (new_x + 1, new_y))
        )
        if grid[new_y][new_x] == "#":
            raise EdgeError("Edge of the grid")
        if grid[new_y][new_x] == ".":
            return [box]
        if grid[new_y][new_x] == bracket:
            return boxes_to_move2(grid, adjacent_box, direction) + [box]
    if direction in (Directions.UP, Directions.DOWN):
        new_left, new_right = (
            (box[0][0] + dx, box[0][1] + dy),
            (box[1][0] + dx, box[1][1] + dy),
        )

        if (
            grid[new_left[1]][new_left[0]] == "#"
            or grid[new_right[1]][new_right[0]] == "#"
        ):
            raise EdgeError("Wall")

        if (
            grid[new_left[1]][new_left[0]] == "."
            and grid[new_right[1]][new_right[0]] == "."
        ):
            return [box]

        if (
            grid[new_left[1]][new_left[0]] == "["
            and grid[new_right[1]][new_right[0]] == "]"
        ):
            return boxes_to_move2(
                grid,
                ((new_left[0], new_left[1]), (new_right[0], new_right[1])),
                direction,
            ) + [box]

        left_box, right_box = [], []
        if grid[new_left[1]][new_left[0]] == "]":
            left_box = boxes_to_move2(
                grid,
                ((new_left[0] - 1, new_left[1]), (new_left[0], new_left[1])),
                direction,
            )
        if grid[new_right[1]][new_right[0]] == "[":
            right_box = boxes_to_move2(
                grid,
                ((new_right[0], new_right[1]), (new_right[0] + 1, new_right[1])),
                direction,
            )
        return left_box + right_box + [box]
    return []
```

This function takes in a tuple of two coordinates, the left and right side of a
box, and the direction the robot should move in. The function is recursive just
like part 1. Breaking down the function:

- `if direction in (Directions.LEFT, Directions.RIGHT):` If the robot is moving
  left or right, the function checks the adjacent box to the left or right of
  the current box. This logic is mainly the same as part 1. The only difference
  is that when the recursive call is made, the _adjacent box_ is passed in as
  the box to move.
- `if direction in (Directions.UP, Directions.DOWN):` This is where the complex
  logic comes in.
  - `new_left, new_right = ...` Calculates the new positions of the left and
    right side of the box.
  - `if grid[new_left[1]][new_left[0]] == "#" or ...` Checks if the box would
    move into a wall. If so, raise an `EdgeError`.
  - `if grid[new_left[1]][new_left[0]] == "." and ...` Checks if the box would
    move into an empty spot. If so, return the current box.
  - `if grid[new_left[1]][new_left[0]] == "[" and ... "]"` Checks if the box
    would move into another box. If true, there is a box _directly_ above/below
    the current box.
    - `return boxes_to_move2(...)` Recursively call the function with the new
      box directly above/below the current box and return result plus the
      current box.
  - `left_box, right_box = [], []` At this point, there could be both a box
    above/below _and_ to the left or right of the current box. We need to check
    for both.
  - `if grid[new_left[1]][new_left[0]] == "]":` If there is a box to the left
    and above/below the current box, recursively call the function with the new
    left box.
  - `if grid[new_right[1]][new_right[0]] == "[":` If there is a box to the right
    and above/below the current box, recursively call the function with the new
    right box.
  - `return left_box + right_box + [box]` Return the list of boxes that should
    be moved.

My `move_robot` function is also updated to use the new `boxes_to_move2`
function.

```python
def move_robot2(grid: Grid, robot: Coordinate, direction: Directions) -> Coordinate:
    x, y = robot
    dx, dy = direction.value
    new_x, new_y = x + dx, y + dy
    # Can't move if there is a wall, return Early
    if grid[new_y][new_x] == "#":
        return robot
    # Can move freely, update the grid
    if grid[new_y][new_x] == ".":
        grid[y][x] = "."
        grid[new_y][new_x] = "@"
        return (new_x, new_y)
    # Box is in new spot, check if box can be moved
    if grid[new_y][new_x] == "[" or grid[new_y][new_x] == "]":
        if grid[new_y][new_x] == "[":
            box = ((new_x, new_y), (new_x + 1, new_y))
        else:
            box = ((new_x - 1, new_y), (new_x, new_y))
        try:
            boxes = boxes_to_move2(grid, box, direction)
        except EdgeError:
            return robot
        match direction:
            case Directions.UP:
                boxes = sorted(boxes, key=lambda x: x[0][1])
            case Directions.DOWN:
                boxes = sorted(boxes, key=lambda x: x[0][1], reverse=True)
            case Directions.LEFT:
                boxes = sorted(boxes, key=lambda x: x[0][0])
            case Directions.RIGHT:
                boxes = sorted(boxes, key=lambda x: x[0][0], reverse=True)
        for box in boxes:
            box_l, box_r = box
            new_box_l, new_box_r = (
                (box_l[0] + dx, box_l[1] + dy),
                (box_r[0] + dx, box_r[1] + dy),
            )
            grid[box_l[1]][box_l[0]] = "."
            grid[box_r[1]][box_r[0]] = "."
            grid[new_box_l[1]][new_box_l[0]] = "["
            grid[new_box_r[1]][new_box_r[0]] = "]"
        grid[y][x] = "."
        grid[new_y][new_x] = "@"
        return (new_x, new_y)
    raise ValueError("Spot is invalid")
```

The differences start at the `if grid[new_y][new_x] == "[" or ...` block. This
is where the new box checking logic happens.

- `if grid[new_y][new_x] == "[" or grid[new_y][new_x] == "]":` Checks if the
  robot is moving into a box.
  - `if grid[new_y][new_x] == "[":` Determine the side of the box that the robot
    is interacting with. If the robot is moving into a `[`, the box's left side
    should be the `new_x, new_y` position, and the right side should be at
    `new_x+1, new_y`.
  - `else:` If the robot is moving into a `]`, the box's left side should be at
    `new_x-1, new_y`, and the right side should be at `new_x, new_y`.
  - `try: ... except EdgeError` Try to find all the boxes that should be moved
    (same as part 1).
  - `match direction:` Sort the boxes based on the direction the robot is moving
    in. This is to ensure that the boxes are moved in the correct order, and no
    box is overwritten when they are moved.
  - `for box in boxes:` Move each box in the list of boxes to its new position.
- Rest of the function is the same as part 1.

I needed to ensure that the boxes are sorted based on the direction because if
not, the boxes could be overwritten and marked as empty spots when iterating
through the list of boxes to move.

The rest of the code is the same as part 1, just using the new functions.

```python
def main2():
    with open("input.txt", "r", encoding="utf-8") as f:
        lines = f.readlines()

    warehouse_grid, movements = parse_input(lines)
    warehouse_grid_scaled = scale_grid(warehouse_grid)
    robot = find_robot(warehouse_grid_scaled)

    for movement in movements:
        robot = move_robot2(warehouse_grid_scaled, robot, movement)

    boxes = find_boxes(warehouse_grid_scaled)
    coordinates = sum(map(calculate_gps_coordinate, boxes))
    print(f"LOG: { coordinates = }")
```

Again, I've only included the relevant parts of the code here, check out my
[repository](https://github.com/VBenny42/AoC/blob/main/2024/day15/solution.py)
for the full solution.

---

That's it for day 15 of Advent of Code 2024! I hope you enjoyed reading my
solution and let's see how the rest of the month goes!
