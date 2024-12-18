---
title: "Advent of Code 2024 Day 5 – Print Queue"
layout: post
date: 2024-12-05 20:53
image: /assets/images/2024-12-14/christmas_tree_scaled.png
headerImage: true
tag:
    - python
    - advent-of-code
star: false
projects: true
category: blog
author: vineshbenny
description: Vinesh Benny's Advent of Code 2024 Day 5 solution in Python.

excerpt: "Vinesh Benny's Advent of Code 2024 Day 5 solution in Python."
seo_title: "Advent of Code 2024 Day 5 -- Print Queue by Vinesh Benny"
seo_description: "Vinesh Benny's Advent of Code 2024 Day 5 solution in Python."
---

# Day 5: [Print Queue](https://adventofcode.com/2024/day/5)

## Part 1 {#Part1}

Today's puzzle is called "Print Queue". The input for today's puzzle is a file
containing two paragraphs of text, with the first paragraph being ordering
rules, and the second paragraph being sequences of numbers. This is the example
input that was given.

```plaintext
47|53
97|13
97|61
97|47
75|29
61|13
75|53
29|13
97|29
53|29
61|53
97|53
61|29
47|13
75|47
97|75
47|61
75|61
47|29
75|13
53|13

75,47,61,53,29
97,61,53,29,13
75,29,13
75,97,47,61,53
61,13,29
97,13,75,29,47
```

For the first paragraph, each line corresponds to an ordering rule such that the
first number must always be before the second number in a sequence. For example
`47|53` means that `47` must come before `53` in the sequence. This doesn't
necessarily mean that `47` must be directly before `53`, but it must be before
it in the sequence.

The second paragraph contains sequences of numbers (in the context of the
question they're also called updates), where each sequence needs to be checked
to see if it follows the ordering rules.

Looking at the example input, the first sequence is `75,47,61,53,29`. This
sequence follows the ordering rules, because:

- `75` is correctly first because there are rules that put each other page after
  it: `75|47`, `75|61`, `75|53`, and `75|29`.
- `47` is correctly second because `75` must be before it (`75|47`) and every
  other page must be after it according to `47|61`, `47|53`, and `47|29`.
- `61` is correctly in the middle because `75` and `47` are before it (`75|61`
  and `47|61`) and `53` and `29` are after it (`61|53` and `61|29`).
- `53` is correctly fourth because it is before page number `29` (`53|29`).
- `29` is the only page left and so is correctly last.

The fourth update, `75,97,47,61,53`, is not in the correct order: it would print
`75` before `97`, which violates the rule `97|75`. The fifth update, `61,13,29`,
is also not in the correct order, since it breaks the rule `29|13`. The last
update, `97,13,75,29,47`, is not in the correct order due to breaking several
rules.

Our task is to find all the correct sequences and sum all the middle numbers in
the correct sequences. So for the example input, the correct sequences are
`75,47,61,53,29`, `97,61,53,29,13` and `75,29,13`, and the sum of the middle
numbers is `61 + 53 29 = 143`.

### My Solution {#Part1Solution}

First, I split the input file into two files, one for each paragraph.

<div>
	<figcaption class="caption">Save the rules into a rules file, orderings into an updates file.</figcaption>
	<script src="https://asciinema.org/a/VvBSkia5sN71MfbuXK6eIbrga.js" id="asciicast-VvBSkia5sN71MfbuXK6eIbrga" async="true"></script>
</div>

Now that I have the rules and updates in separate files, I can start reading the
files and processing the data.

```python
with open("input-rules.txt", "r") as f:
    rules = (
        [int(value) for value in line.strip().split("|")] for line in f.readlines()
    )
with open("input-updates.txt", "r") as f:
    updates = (
        tuple(int(value) for value in line.strip().split(","))
        for line in f.readlines()
    )
```

Let's break down the code:

- `rules = ([int(value) for value in line.strip().split("|")] for line in f.readlines())`
  Create a generator expression that reads each line from the rules file, splits
  it by the `|` character, converts the values to integers, and yields them as a
  list.
- `updates = (tuple(int(value) for value in line.strip().split(",")) for line in f.readlines())`
  Create a generator expression that reads each line from the updates file,
  splits it by the `,` character, converts the values to integers, and yields
  them as a tuple.

Now that I have all the pairwise rules, I can create a dictionary that maps each
number to a set of numbers that must come after it.

```python
ruleset = DefaultDict(set)
for rule in rules:
    ruleset[rule[0]].add(rule[1])
return ruleset
```

I create a `defaultdict` with a `set` as the default value, so that I can add
values to the set without having to check if the key exists.

Now that I have the ruleset, I can check if each update follows the rules.

```python
def is_valid1(update: tuple[int, ...], ruleset: dict[int, set[int]]) -> bool:
    for i in range(len(update)):
        before = update[:i]
        page = update[i]
        if page in ruleset:
            if any(dep in before for dep in ruleset[page]):
                return False
    return True
```

Let's break down the code:

- `for i in range(len(update)):` Loop through each page in the update.
- `before = update[:i]` Get the pages that come before the current page.
- `page = update[i]` Get the current page.
- `if page in ruleset:` Check if the current page has any rules associated with
  it.
- `if any(dep in before for dep in ruleset[page]):` Check if any of the pages
  that must come after the current page are already in the `before` list. If
  that's the case, the update is invalid, so return `False`.
- `return True` If the loop completes without finding any violations, return
  `True`.

Now that I have a way to determine valid updates, I can calculate the sum of the
middle numbers in the valid updates.

```python
sum = 0
for update in updates:
    if is_valid1(update, ruleset):
        sum += update[(len(update) - 1) // 2]
print(f"LOG: {sum = }")
```

This code loops through each update, checks if it's valid, and if it is, adds
the middle number to the sum. This was my first solution to the problem, but
after working on part 2, I found another way as well that I'll explain in the
next section.

I've only included the relevant parts of the code here, but to see my full
solution, you can check out my
[Advent of Code GitHub repository](https://github.com/VBenny42/AoC/blob/main/2024/day05/solution.py).

## Part 2 {#Part2}

For part 2, we instead need to find the sum of the middle numbers of the
_invalid_ updates.

Revisiting the example:

```plaintext
47|53
97|13
97|61
97|47
75|29
61|13
75|53
29|13
97|29
53|29
61|53
97|53
61|29
47|13
75|47
97|75
47|61
75|61
47|29
75|13
53|13

75,47,61,53,29
97,61,53,29,13
75,29,13
75,97,47,61,53
61,13,29
97,13,75,29,47
```

The incorrect sequences when ordered by the rules are:

- `75,97,47,61,53` becomes `97,75,47,61,53`.
- `61,13,29` becomes `61,29,13`.
- `97,13,75,29,47` becomes `97,75,47,29,13`.

Adding the middle numbers of these sequences gives `47 + 29 + 47 = 123`.

### My Solution {#Part2Solution}

So for part 2, I can reuse the `is_valid1` function from part 1, but instead of
checking if the update is valid, I can check if it's invalid. Once I find an
invalid update, I need to find the valid ordering of the update according to the
rules and sum the middle numbers.

To reorder the update according to the rules, I can make use of a comparator
function that I can pass to the `sorted` function. A comparator function takes
two values and returns `-1` if the first value should come before the second,
`0` if they're equal, and `1` if the second value should come before the first.
For our purposes, we only care if the first value should come before the second,
so we can return `-1` if the first value should come before the second, and `0`
otherwise.

```python
def compare(ruleset, a, b):
    if a in ruleset:
        if b in ruleset[a]:
            return -1
    return 0
```

This function checks if `a` has any rules associated with it, and if `b` is in
the set of pages that must come after `a`. If that's the case, it returns `-1`,
since `a` should come before `b`. Otherwise, it returns `0`.

Now I can use this comparator function to sort the update according to the
rules.

```python
def reordering(
    update: tuple[int, ...], ruleset: dict[int, set[int]]
) -> tuple[int, ...]:
    compare_with_ruleset = lambda a, b: compare(ruleset, a, b)
    return tuple(sorted(update, key=cmp_to_key(compare_with_ruleset)))
```

The `sorted` function takes a `key` argument that specifies a function to use to
extract a comparison key from each element, but since we need to compare two
elements at a time, we need to use the
[`cmp_to_key`](https://docs.python.org/3/library/functools.html#functools.cmp_to_key)
function from the `functools` module to convert the comparator function to a key
function. Also, the `cmp_to_key` function only accepts functions that take two
arguments, but since our `compare` function takes three arguments, I've created
a lambda function that only takes two arguments and passes the ruleset as the
first argument. Now I just have to let `sorted` figure out the valid ordering
according to the rules.

Now that I have a way to reorder the update according to the rules, I can
calculate the sum of the middle numbers in the invalid updates.

```python
sum = 0
for update in updates:
    if not is_valid2(update, ruleset):
        valid_reordering = reordering(update, ruleset)
        sum += valid_reordering[(len(valid_reordering) - 1) // 2]
print(f"LOG: {sum = }")
```

This code loops through each update, checks if it's invalid, and if it is,
reorders the update according to the rules and adds the middle number to the
sum.

#### Part 1 revisited {#Part1Revisited}

Once I started using `sorted` to get the valid ordering of the updates, I
realized that I could use the same approach to solve part 1 as well. Instead of
checking if the update is valid, I can sort the update according to the rules
and check if the sorted update is the same as the original update. If it is, the
update is valid.

```python
def is_valid2(update: tuple[int, ...], ruleset: dict[int, set[int]]) -> bool:
    return reordering(update, ruleset) == update
```

All I had to do was check if the valid reordering from part 2 was the same as
the original update. Both ways of solving part 1 are valid, but I found the
second way to be more elegant.

Again, I've only included the relevant parts of the code here, check out my
[repository](https://github.com/VBenny42/AoC/blob/main/2024/day05/solution.py)
for the full solution.

---

That's it for day 5 of Advent of Code 2024! I hope you enjoyed reading my
solution and let's see how the rest of the month goes!
