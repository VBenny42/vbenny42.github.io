---
title: "Advent of Code 2024 Day 9 – Disk Fragmenter"
layout: post
date: 2024-12-09 17:37
image: /assets/images/favicon/apple-touch-icon.png
headerImage: false
tag:
    - python
    - advent-of-code
star: false
projects: true
category: blog
author: vineshbenny
description: Vinesh Benny's Advent of Code 2024 Day 9 solution in Python.

excerpt: "Vinesh Benny's Advent of Code 2024 Day 9 solution in Python."
seo_title: "Advent of Code 2024 Day 9 -- Disk Fragmenter by Vinesh Benny"
seo_description: "Vinesh Benny's Advent of Code 2024 Day 9 solution in Python."
---

# Day 9: [Disk Fragmenter](https://adventofcode.com/2024/day/9)

<!--toc:start-->

- [Day 9:
  [Disk Fragmenter](https://adventofcode.com/2024/day/9)](#day-9-disk-fragmenterhttpsadventofcodecom2024day9)
  - [Part 1](#part-1)
    - [My Solution to Part 1](#my-solution-to-part-1)
  - [Part 2](#part-2)
    - [My Solution to Part 2](#my-solution-to-part-2)
      - [Update: Optimizations!](#update-optimizations)

<!--toc:end-->

## Part 1

Today's input is a sequence of digits corresponding to a disk map. The example
input is:

```
2333133121414131402
```

This disk map maps the blocks that files and free space are taking up on the
disk. The digits alternate between showing the number of blocks that a file
takes up, and then the number of free blocks.

So for example, `12345` would mean:

- A file with size `1` block, followed by
- A `2` block free space, followed by
- A file with size `3` blocks, followed by
- A `4` block free space, followed by
  - A file with size `5` blocks.

Also, every file has an associated ID number, which starts at `0` and increments
by `1` for each new file on disk. So if we visualize the disk map above, with
every block represented by a file ID or a `.` for free space, it would be:

```
0..111....22222
```

The example input above would be represented as:

```
00...111...2...333.44.5555.6666.777.888899
```

We would like to move blocks one at a time from the end of the desk to the
leftmost free space block, until there are no free spaces between files.

For the `12345` example above, the disk map would look like this after moving
blocks one at a time:

```
0..111....22222
02.111....2222.
022111....222..
0221112...22...
02211122..2....
022111222......
```

And for the example input, the disk map would look like this after moving
blocks:

```
00...111...2...333.44.5555.6666.777.888899
009..111...2...333.44.5555.6666.777.88889.
0099.111...2...333.44.5555.6666.777.8888..
00998111...2...333.44.5555.6666.777.888...
009981118..2...333.44.5555.6666.777.88....
0099811188.2...333.44.5555.6666.777.8.....
009981118882...333.44.5555.6666.777.......
0099811188827..333.44.5555.6666.77........
00998111888277.333.44.5555.6666.7.........
009981118882777333.44.5555.6666...........
009981118882777333644.5555.666............
00998111888277733364465555.66.............
0099811188827773336446555566..............
```

Once we have compacted the disk map, we want to calculate the checksum of the
files on the disk. This is done as such:

- Initialize the checksum to `0`.
- For each block position:
  - If the block is a file, multiply the block position by the file ID, and add
    it to the checksum.
  - If the block is free space, do nothing.

For the small example, the checksum is `60`; and for the example input, the
checksum is `1928`.

We need to calculate the checksum for the compacted input disk map.

### My Solution to Part 1

A couple things to note about the question:

- The disk map is a string of digits, and for the example input, we can
  visualize it as a string of alternating file IDs and free space blocks, where
  each file ID takes 1 character. However, the file IDs can be more than 1
  character long, so we can't represent one block as one character.
- As soon as there are no free spaces between files, we can't move any more
  blocks, so we can stop the process.

Let's go through my pseudocode solution:

- Read the input disk map as a list of integers.
- Convert this list to a list of alternating file IDs and free space blocks,
  with a block corresponding to an element in the list.
- Compact the disk map as per the rules above.
- Calculate the checksum as per the rules above.

First, I read the input disk map as a list of integers:

```python
with open("input.txt", "r") as f:
	line = f.read().strip()
	int_line = [int(x) for x in line]
```

Converting the list to a list of alternating file IDs and free space blocks:

```python
def convert(line: list) -> list:
    is_freespace = False
    id = 0
    diskmap = []
    for i in line:
        if is_freespace:
            for _ in range(i):
                diskmap.append(FREE_SPACE)
        else:
            for _ in range(i):
                diskmap.append(id)
            id += 1
        is_freespace = not is_freespace
    return diskmap
```

Breaking down the code above:

- Initialize a boolean `is_freespace` to `False`, and an integer `id` to `0`,
  and an empty list `diskmap`.
- `for i in line:` Iterate through the list of integers.
- If `is_freespace` is `True`, append `i` free space blocks to the `diskmap`.
- If `is_freespace` is `False`, append `i` file IDs to the `diskmap`,
  incrementing the `id` by `1` for the next file ID.
- Toggle the `is_freespace` boolean, as the next element in the list will be of
  the opposite type.

Compacting the disk map:

```python
def make_contiguous1(diskmap: list) -> list:
    first_free_block = 0
    last_file_block = len(diskmap) - 1

    while first_free_block < last_file_block:
        while (
            first_free_block < len(diskmap) and diskmap[first_free_block] != FREE_SPACE
        ):
            first_free_block += 1

        while last_file_block >= 0 and diskmap[last_file_block] == FREE_SPACE:
            last_file_block -= 1

        if first_free_block < last_file_block:
            diskmap[first_free_block], diskmap[last_file_block] = (
                diskmap[last_file_block],
                diskmap[first_free_block],
            )

    return diskmap
```

Breaking down the code above:

- `first_free_block` is initialized to `0`, and `last_file_block` is initialized
  to the last index of the `diskmap`.
- `while first_free_block < last_file_block:` Loop until there are no free
  spaces between files.
  - `while first_free_block < len(diskmap) and diskmap[first_free_block] != FREE_SPACE:`
    Find the first free block that occurs in the `diskmap`.
  - `while last_file_block >= 0 and diskmap[last_file_block] == FREE_SPACE:`
    Find the last file block that occurs in the `diskmap`.
  - `if first_free_block < last_file_block:` There are free spaces between
    files, so swap the first free block with the last file block.

By the end of the `make_contiguous1` function, the disk map will be compacted.

Calculating the checksum:

```python
files_only = takewhile(lambda x: x != FREE_SPACE, contiguous_diskmap)
checksum = sum(i * id for i, id in enumerate(files_only))
print(f"LOG: {checksum = }")
```

Breaking down the code above:

- `files_only  = takewhile(lambda x: x != FREE_SPACE, contiguous_diskmap)` Use
  the
  [`takewhile`](https://docs.python.org/3/library/itertools.html#itertools.takewhile)
  function to get the file IDs from the compacted disk map. Since there are no
  files once we reach a free space block, we can stop taking elements from the
  list at that point.
- `checksum = sum(i * id for i, id in enumerate(files_only))` Calculate the
  checksum by multiplying the block position by the file ID, and summing it up.

As always, I've only included the relevant parts of the code here, but to see my
full solution, you can check out my
[Advent of Code GitHub repository](https://github.com/VBenny42/AoC/blob/main/2024/day09/solution.py).

## Part 2

In part 2, we still need to compact the disk map as in part 1, but we cannot
move blocks one at a time. Instead, we need to move the entire file to free
space. We try to move each file from the end to free space on the left exactly
once, and if we can't move it to free space, we leave the file as is and try to
move the next file.

So the example input disk map when compacted looks like this instead:

```
00...111...2...333.44.5555.6666.777.888899
0099.111...2...333.44.5555.6666.777.8888..
0099.1117772...333.44.5555.6666.....8888..
0099.111777244.333....5555.6666.....8888..
00992111777.44.333....5555.6666.....8888..
```

Going through the steps:

- `99` can move to the leftmost free space. Move it and now the leftmost free
  space can only fit single block files.
- `8888` cannot be moved to any space to the left of it, as it has a filesize of
  `4` and there are no free spaces of size `4` to the left of it. So we leave it
  as is.
- `777` can move to the second free space from the left. Move it. That free
  space is entirely filled.
- Both `6666` and `5555` cannot be moved for the same reason as `8888`. Leave
  them as is.
- `44` can move to the second leftmost free space. Move it.
- `333` cannot move to any left free space, as they are all less than `3` blocks
  in size. Leave it as is.
- `2` can move to the leftmost free space. Move it.
- `111` has no left free space to move to. Leave it as is.
- Same for `00`.

Calculating the checksum is the same as in part 1, so the new checksum for the
example input is `2858`.

We need to calculate the checksum for the compacted disk map as per the new
rules.

### My Solution to Part 2

My solution is a bit more complicated than part 1, as now we need to keep track
of file sizes and free space sizes, and move files to free spaces based on their
sizes.

I'll go through my pseudocode solution:

- Read the input disk map as a list of integers.
- Convert this list to a list of alternating file IDs and free space blocks,
  with an element in the list corresponding to either a file or free space and
  its size.
- Compact the disk map as per the new rules.
- Calculate the checksum as per the rules above.

The input file is read the same as in part 1. The conversion function is
different:

```python
def convert_with_size(line: list) -> list:
    is_freespace = False
    id = 0
    diskmap = []
    for i in line:
        if is_freespace:
            diskmap.append((FREE_SPACE, i))
        else:
            diskmap.append((id, i))
            id += 1
        is_freespace = not is_freespace
    return diskmap
```

Instead of just appending the file ID or free space block, I append a tuple of
the file ID or free space block and its size.

The compaction function is different as well:

```python
def make_contiguous2(diskmap: list) -> list:
    files = list(reversed([x for x in (diskmap) if x[0] != FREE_SPACE]))
    for file in files:
        swap_file(diskmap, file)
    return diskmap
```

This function finds all the files starting from the end of the disk map, and for
each one, sees if it can be moved to a free space block to the left.

My `swap_file` function is where most of the logic is:

```python
def swap_file(diskmap: list, file: tuple):
    file_index = diskmap.index(file)
    free_space_index = -1

    for i in range(file_index):
        if diskmap[i][0] == FREE_SPACE and diskmap[i][1] >= file[1]:
            free_space_index = i
            break

    if free_space_index == -1:
        return

    if diskmap[free_space_index][1] > file[1]:
        diskmap[free_space_index] = (FREE_SPACE, diskmap[free_space_index][1] - file[1])
        diskmap[file_index] = (FREE_SPACE, file[1])
        diskmap.insert(free_space_index, file)

    elif diskmap[free_space_index][1] == file[1]:
        diskmap[free_space_index], diskmap[file_index] = (
            diskmap[file_index],
            diskmap[free_space_index],
        )

    return
```

Breaking down the code above:

- `file_index = diskmap.index(file)` Get the index of the file in the disk map.
- `free_space_index = -1` Initialize the free space index to `-1`.
- `for i in range(len(diskmap)): ... break` Find the first leftmost free space
  block before the file that can fit the file.
- `if free_space_index == -1: return` No free space block that can fit the file
  found, so return.
- `if diskmap[free_space_index][1] > file[1]: ...` The free space block is
  larger than the file.
  - Reduce the size of the free space block by the size of the file.
  - Set the element at the file index to a free space block of the size of the
    file.
  - Insert the file to the left of the free space index.
- `elif diskmap[free_space_index][1] == file[1]: ...` The free space block is
  the same size as the file.
  - Swap the file and the free space block.

Once this is done, the checksum function is a bit more involved than part 1:

```python
def checksum(diskmap: list):
    index = 0
    checksum = 0
    for x in diskmap:
        if x[0] != FREE_SPACE:
            for i in range(x[1]):
                checksum += (index + i) * x[0]
        index += x[1]
    return checksum
```

Explained:

- `index = 0` Initialize the index to `0`.
- `checksum = 0` Initialize the checksum to `0`.
- `for x in diskmap:` Iterate through the disk map.
  - `if x[0] != FREE_SPACE:` If the element is a file:
    - `for i in range(x[1]):` Iterate through the size of the file.
      - `checksum += (index + i) * x[0]` Add the block position multiplied by
        the file ID to the checksum.
  - `index += x[1]` Increment the index by the size of the file.

Finally, I call the functions in order:

```python
diskmap = convert_with_size(int_line)
contiguous_diskmap = make_contiguous2(diskmap)
print(f"LOGF: checksum for contiguous files {checksum(contiguous_diskmap)}")
```

Again, I've only included the relevant parts of the code here, check out my
[repository](https://github.com/VBenny42/AoC/blob/main/2024/day09/solution.py)
for the full solution.

#### Update: Optimizations!

<script id="MathJax-script" async
          src="https://cdn.jsdelivr.net/npm/mathjax@3.0.1/es5/tex-mml-chtml.js">
  </script>

<p>
My code runs in \(O(n^2)\) time, due to looking for free space blocks that can
fit the file for each file. I can optimize this by using 10 priority queues to
store the indices of free space blocks of size <code class="language-plaintext highlighter-rouge">0</code> to <code class="language-plaintext highlighter-rouge">9</code>. This way, I can just
pop the leftmost free space block that can fit the file from one of the priority
queues. This will reduce the time complexity to \(O(n \log n)\).
</p>

The new part 2 logic is almost the same as before, but with a few changes:

- When converting the disk map, I also initialize the priority queues.
- When trying to make the disk map contiguous, I pop the leftmost free space
  block that can fit the file from the priority queues, instead of iterating
  through the disk map.
- I also swap the items in place, instead of inserting the file at the free
  space index, to not mess up the indices in the priority queues.

Looking at my new conversion function:

```python
def convert_with_heaps(line: list) -> tuple[list, list]:
    is_freespace = False
    id = 0
    diskmap = []
    heaps = [[] for _ in range(10)]
    for i in line:
        if is_freespace:
            # Push index of free space to the heap of the size of the free space
            heapq.heappush(heaps[i], len(diskmap))
            for _ in range(i):
                diskmap.append(FREE_SPACE)
        else:
            for _ in range(i):
                diskmap.append(id)
            id += 1
        is_freespace = not is_freespace
    return diskmap, heaps
```

Going over the differences from my originial conversion function from my
[Part 1 Solution](#my-solution-to-part-1):

- `heaps = [[] for _ in range(10)]` Initialize 10 empty priority queues.
- `heapq.heappush(heaps[i], len(diskmap))` Push the index of the free space to
  the heap of the size of the free space. The index is just the length of the
  disk map at that point.
- `return diskmap, heaps` Return the disk map _and_ the priority queues.

The new `make_contiguous2` function has a bit more changes:

```python
def make_contiguous2_heap(diskmap: list, heaps: list) -> list:
    index = len(diskmap) - 1
    while index >= 0:
        if diskmap[index] == FREE_SPACE:
            index -= 1
            continue

        id = diskmap[index]
        file_width = 0
        # Get the width of the file
        while index >= 0 and diskmap[index] == id:
            file_width += 1
            index -= 1

        best_width = -1
        smallest_index = len(diskmap)
        # Find the leftmost index of free space that can fit the file
        for width in range(file_width, 10):
            if heaps[width]:
                if smallest_index > heaps[width][0]:
                    smallest_index = heaps[width][0]
                    best_width = width

        if smallest_index == len(diskmap):
            continue
        if smallest_index > index:
            continue

        # Remove the smallest index from the heap
        # In-place swap the file with the free space
        heapq.heappop(heaps[best_width])
        for j in range(file_width):
            diskmap[smallest_index + j] = id
            diskmap[index + j + 1] = FREE_SPACE
        # Push the new smaller free space to the heap
        heapq.heappush(heaps[best_width - file_width], smallest_index + file_width)

    return diskmap
```

Again this is more similar to my part 1 `make_contiguous` function, but with a
few changes:

- `index = len(diskmap) - 1` Start from the end of the disk map.
- `while index >= 0:` Loop until we reach the start of the disk map.
  - `if diskmap[index] == FREE_SPACE:` If the element is a free space, nothing
    to swap, skip it.
  - `id = diskmap[index]` Get the file ID of current element.
  - `file_width = 0` Initialize the width of the file to `0`.
  - `while index >= 0 and ...: index -= 1` Get the width of the file, and move
    the index to the left.
  - `best_width = -1` Initialize the best width of free space to `-1`.
  - `smallest_index = len(diskmap)` Initialize the smallest index of free space
    to the end of the disk map.
  - `for width in range(file_width, 10):` Iterate through the priority queues
    from the width of the file to `10`.
    - `if heaps[width]:` If the heap of the width is not empty:
      - `if smallest_index > heaps[width][0]:` If the smallest index in the heap
        is less than the current smallest index, i.e. it is more to the left:
        - `smallest_index = heaps[width][0]` Update the smallest index.
        - `best_width = width` Update the best width.
  - `if smallest_index == len(diskmap): continue` No free space found that can
    fit the file, file can't be moved, skip it.
  - `if smallest_index > index: continue` The free space is to the right of the
    file, file can't be moved, skip it.
  - `heapq.heappop(heaps[best_width])` Remove the leftmost free space index from
    its heap.
  - `for j in range(file_width): ...` In-place swap the file with the free
    space.
  - `heapq.heappush(heaps[best_width - file_width], smallest_index + file_width)`
    Push the new free space to the heap of the new size.

The rest of the code is the same as Part 1, but now I call the new functions:

```python
def main3():
    with open("input.txt", "r") as f:
        line = f.read().strip()
        int_line = [int(x) for x in line]
    diskmap, heaps = convert_with_heaps(int_line)
    contiguous_diskmap = make_contiguous2_heap(diskmap, heaps)
    checksum = sum(
        i * id for i, id in enumerate(contiguous_diskmap) if id != FREE_SPACE
    )
    print(f"LOGF: {checksum = }")
```

This code is more optimized than my original solution, and runs much faster than
my original solution.

```
LOG: checksum = files 6415666220005
Function 'main2' executed in 3.6767s
LOG: checksum = 6415666220005
Function 'main3' executed in 0.0288s
```

---

I took a lot of time to get to my Part 2 solution today. The problems are only
going to get harder from here, so we'll see how long I last. I hope you enjoyed
reading my solution and that's it for Day 9!
