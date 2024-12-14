---
title: "Advent of Code 2024 Day 7 – Bridge Repair"
layout: post
date: 2024-12-07 10:46
image: /assets/images/2024-12-14/christmas_tree_scaled.png
headerImage: true
tag:
    - python
    - advent-of-code
star: false
projects: true
category: blog
author: vineshbenny
description: Vinesh Benny's Advent of Code 2024 Day 7 solution in Python.

excerpt: "Vinesh Benny's Advent of Code 2024 Day 7 solution in Python."
seo_title: "Advent of Code 2024 Day 7 -- Bridge Repair by Vinesh Benny"
seo_description: "Vinesh Benny's Advent of Code 2024 Day 7 solution in Python."
---

# Day 7: [Bridge Repair](https://adventofcode.com/2024/day/7)

## Part 1

Today's input is a list of equations where the operators are missing. An
equation is of the form `X: Y ... Z`, where `X` is the value and `Y ... Z` could
be combined in some way to get `X`. The example input is:

```
190: 10 19
3267: 81 40 27
83: 17 5
156: 15 6
7290: 6 8 6 15
161011: 16 10 13
192: 17 8 14
21037: 9 7 18 13
292: 11 6 16 20
```

Operators are _always_ evaluated left-to-right, _not_ according to precedence
rules, and parentheses are _not_ allowed. The operators we need to consider are
only `+` and `*`. Some of the equations are wrong, i.e., they don't evaluate to
the value on the left side of the colon, no matter what combination of operators
are used. As each equation is missing the operators, we need to figure out how
to combine the numbers to get the value on the left side of the colon, if
possible.

For example:

- The first equation, `190: 10 19`, can be combined as `10 * 19 = 190`.
- `3267: 81 40 27` has two spots for operators, and of the four possible
  combinations, `81 + 40 * 27` and `81 * 40 + 27` both evaluate to `3267`.
- `292: 11 6 16 20` can be solved in only one way: `11 + 6 * 16 + 20`.

We need to sum the values on the left side of the colon for the equations that
are actually solvable. So for the sample input, the answer is
`190 + 3267 + 292 = 3749`.

### My Solution

My solution used a forward recursion approach at first, and then once I had a
working solution I did a backward recursion approach as well.

#### Forward Recursion

Let's start with the pseudocode first:

- Read the input, split the equation into the value and the numbers, and store
  them in a list as integers.
- For each equation, recursively solve the equation by adding and multiplying
  the numbers in all possible ways.
  - For the base case, if there's only two numbers left, check if they can be
    added or multiplied to get the desired value.
  - If the first number is greater than the desired value, there's no way to get
    the desired value since there's no operator to decrease the value of a
    number, so return `False`.
  - If there are more than two numbers left:
    - Take the first two numbers and apply both operators to them.
    - Recursively call the function with the result from the previous step and
      the rest of the numbers.
- If the equation is solvable, add the value to the total sum.

I can implement this in Python as follows:

```python
def is_valid_equation(equation: Equation, operators) -> bool:
    desired_value, numbers = equation
    if numbers[0] > desired_value:
        return False
    if len(numbers) == 2:
        return any(desired_value == op(*numbers) for op in operators)
    first, second, *remaining_numbers = numbers
    return any(
        is_valid_equation(
            (desired_value, [op(first, second)] + remaining_numbers), operators
        )
        for op in operators
    )
```

This function takes an equation and a list of operators to apply to the numbers.
Breaking down the function:

- `desired_value, numbers = equation` Unpack the equation into the desired value
  and the numbers to combine.
- `if len(numbers) == 2:` If there are only two numbers left, check if they can
  be combined to get the desired value. This is the base case.
- `if numbers[0] > desired_value` If the first value is greater than the desired
  value, no need to check the rest of numbers, early return `False`.
- `first, second, *remaining_numbers = numbers` Take the first two numbers and
  the rest of the numbers.
- `return any(...)` Check if the equation is solvable by applying the operators
  to the first two numbers and recursively calling the function with the result
  and the rest of the numbers.

I can now use this function to find all the solvable equations and sum the
values on the left side of the colon.

```python
def main1_forward():
    with open("input.txt", "r") as f:
        equations = [
            (int(desired_value), list(map(int, remaining_numbers.split())))
            for line in f
            for desired_value, remaining_numbers in [line.split(":")]
        ]

    print(
        f"LOGF: Sum of true equations: {sum(equation[0] for equation in equations if is_valid_equation(equation, {operator.add, operator.mul}))}"
    )
```

This function reads the input, splits the equation into the desired value and
the numbers, and stores them in a list. It then prints the sum of the values on
the left side of the colon for the solvable equations. The operators are passed
in as a set of `operator.add` and `operator.mul`.

This does work, but I'm brute-forcing all the possible combinations of the
operators to solve the equations. I can find a better way to pick my operators.

#### Backward Recursion

Instead of looking at the first two numbers and applying all possible operators
to them, I can instead look at the last number and the desired value to pick the
operator in a smarter way.

Let's explain the logic with an example, using an equation from the sample
input, `292: 11 6 16 20`.

- The desired value is `292`, and the last number is `20`.
- Since the operations are left-to-right, if the operator applied to the last
  number is `*`, then this means that the last number would have to be a factor
  of the desired value. In this case, `20` isn't a factor of `292`, so the
  operator for the last number can't be `*`.
- If the operator applied to the last number is `+`, then the last number would
  have to be smaller than the desired value. In this case, `20` is smaller than
  `292`, so the operator for the last number can be `+`.
- Now that we know that the operator for the last number is `+`, we can remove
  the last number from the equation and the desired value, and recursively call
  the function with the new equation: `272: 11 6 16`.
- `272` _is_ divisible by `16`, so the operator for `16` can be `*`.
- `272` is also greater than `16`, so the operator for `16` can be `+`.
- Now we make two recursive calls, one with the equation `17: 11 6` and the
  other with the equation `256: 11 6`.
- For the purpose of the example (and we already know the answer), let's focus
  on the first recursive call with the equation `17: 11 6`.
- There are only two numbers left, so check if they can be combined to get the
  desired value. In this case, `11 + 6 = 17`, so the equation is solvable.

This approach is more efficient than the forward recursion approach, as we don't
always check all possible combinations of operators, pruning the ones that are
not possible.

I can implement this in Python as follows:

```python
def is_valid_equation_p1(equation: Equation) -> bool:
    desired_value, numbers = equation
    if len(numbers) == 2:
        return (
            desired_value == numbers[0] + numbers[1]
            or desired_value == numbers[0] * numbers[1]
        )
    last = numbers[-1]
    mult, add = False, False
    if desired_value % last == 0:
        mult = is_valid_equation_p1((int(desired_value / last), numbers[:-1]))
    if desired_value - last >= 0:
        add = is_valid_equation_p1((desired_value - last, numbers[:-1]))
    return any([mult, add])
```

Breaking down the function:

- `desired_value, numbers = equation` Unpack the equation into the desired value
  and the numbers to combine.
- `if len(numbers) == 2:` If there are only two numbers left, check if they can
  be combined to get the desired value. This is the base case.
- `last = numbers[-1]` Get the last number in the equation.
- `mult, add = False, False` Initialize variables to check if the equation is
  solvable by multiplying or adding the last number.
- `if desired_value % last == 0:` If the last number is a factor of the desired
  value, recursively call the function with the last number removed and the
  desired value divided by the last number.
- `if desired_value - last >= 0:` If the last number is smaller than the desired
  value, recursively call the function with the last number removed and the
  desired value minus the last number.
- `return any([mult, add])` Return `True` if the equation is solvable by either
  multiplying or adding the last number.

I can now use this function to find all the solvable equations and sum the
values of the left side of the colon, just like before.

```python
def main1_backward():
    with open("input.txt", "r") as f:
        equations = [
            (int(desired_value), list(map(int, remaining_numbers.split())))
            for line in f
            for desired_value, remaining_numbers in [line.split(":")]
        ]

    print(
        f"LOGF: Sum of true equations: {sum(equation[0] for equation in equations if is_valid_equation_p1(equation))}"
    )
```

Looking at the performance of both the forward and backward recursion
approaches:

```
LOG: Sum of true equations: 1153997401072
Function 'main1_forward' executed in 0.1072s
LOG: Sum of true equations: 1153997401072
Function 'main1_backward' executed in 0.0058s
```

We can see that the backward recursion approach is orders of magnitude faster
than the forward recursion approach. I'm happy with the performance of my
solution, and I can move on to part 2.

I've only included the relevant parts of the code here, but to see my full
solution, you can check out my
[Advent of Code GitHub repository](https://github.com/VBenny42/AoC/blob/main/2024/day07/solution.py).

## Part 2

For part 2, a new operator is introduced: `||`. This is the concatenation
operator. It combines the digits of the numbers, so for example `12 || 34` would
be `1234`.

The input for part 2 is the same as part 1, but with the new operator, more
equations are now solvable.

- `156: 15 6` can be made true through a single concatenation: `15 || 6 = 156`.
- `7290: 6 8 6 15` can be made true using `6 * 8 || 6 * 15`.
- `192: 17 8 14` can be made true using `17 || 8 + 14`.

With these new solvable equations, the new sum of the values on the left side of
the colon is `11387`.

### My Solution

#### Forward Recursion

My forward recursion approach for part 1 is already set up to handle the new
operator. I just need to add the new operator to the set of operators.

I need to create the new operator function and add it to the set of operators:

```python
def concat_op(a, b):
    return int(str(a) + str(b))
```

All this function does is concatenate the two numbers as strings and converts
the result to an integer.

I can now add the new operator to the set of operators and call the
`is_valid_equation` function.

```python
{sum(equation[0] for equation in equations if is_valid_equation(equation, {operator.add, operator.mul, concat_op}))}
```

Other than this, the rest of the code remains the same.

#### Backward Recursion

The backward recursion approach for part 2 is also similar to part 1, but I need
to add the new operator within the function instead of passing it in.

I first make a helper function to check if the desired value ends with the last
number:

```python
def ends_with(a: int, b: int) -> int:
    str_a, str_b = str(a), str(b)
    if str_a.endswith(str_b):
        remaining = str_a[: -len(str_b)] if len(str_a) > len(str_b) else "0"
        return int(remaining)
    return 0
```

This function checks if the desired value ends with the last number. If it does,
it returns the remaining part of the desired value after removing the last
number. If it doesn't, it returns `0`. I can now use this function in my main
validation function.

```python
def is_valid_equation_p2(equation: Equation) -> bool:
    desired_value, numbers = equation
    if len(numbers) == 2:
        return (
            desired_value == numbers[0] + numbers[1]
            or desired_value == numbers[0] * numbers[1]
            or desired_value == concat_op(numbers[0], numbers[1])
        )
    last = numbers[-1]
    mult, add, concat = False, False, False
    if desired_value % last == 0:
        mult = is_valid_equation_p2((int(desired_value / last), numbers[:-1]))
    if desired_value - last >= 0:
        add = is_valid_equation_p2((desired_value - last, numbers[:-1]))
    if concat := ends_with(desired_value, last):
        concat = is_valid_equation_p2((concat, numbers[:-1]))
    return any([mult, add, concat])
```

Going over the additions from the part 1 validation function:

- `mult, add, concat = False, False, False` Initialize variables to check if the
  equation is solvable by multiplying, adding, or _concatenating_ the last
  number.
- `or desired_value == concat_op(numbers[0], numbers[1])` Check if the desired
  value can be made by concatenating the first two numbers.
- `concat := ends_with(desired_value, last)` Check if the desired value ends
  with the last number. If it does, recursively call the function with the last
  number removed and the remaining part of the desired value as the new desired
  value.
- return `any([mult, add, concat])` Return `True` if the equation is solvable by
  multiplying, adding, or concatenating the last number.

This function is called just like the part 1 version.

```python
print(f"LOG: Sum of true equations with concat: {sum(equation[0] for equation in equations if is_valid_equation_p2(equation))}")
```

Again the performance of the backward recursion approach is much better than the
forward recursion approach.

```
LOG: Sum of true equations with concat: 97902809384118
Function 'main2_forward' executed in 2.7434s
LOG: Sum of true equations with concat: 97902809384118
Function 'main2_backward' executed in 0.0161s
```

Again, I've only included the relevant parts of the code here, check out my
[repository](https://github.com/VBenny42/AoC/blob/main/2024/day07/solution.py)
for the full solution.

---

That's it for day 7 of Advent of Code 2024! I hope you enjoyed reading my
solution and let's see how the rest of the month goes!
