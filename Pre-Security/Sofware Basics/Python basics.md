Python Programming Fundamentals
 
This section walks through core programming concepts — variables, user input, conditionals, and loops — by incrementally building a "Guess the Number" game in Python.
 
### Core Concepts Introduced
 
| Concept | Purpose |
|---|---|
| **Variables** | Store and track values that change during execution |
| **Random number generation** | Introduce unpredictability/randomness into a program |
| **User input** | Accept and process data from the person running the program |
| **Conditional statements** (`if` / `elif` / `else`) | Execute different code depending on a condition |
| **Loops** (`while`) | Repeat a block of code until a condition is no longer true |
 
---
 
### Version 1: Setup & Initial Input
 
**Goal:** Pick a random secret number, accept one guess from the user, and track attempts.
 
```python
import random  # gives us tools for picking random numbers
 
secret = random.randint(1, 20)  # a <= secret <= b
tries = 0
guess = 0  # start with a value that cannot be the secret (since secret is 1..20)
 
print("I'm thinking of a number between 1 and 20")
 
text = input("Take a guess: ")  # input() returns text (a string)
guess = int(text)               # convert the text to a number
 
tries = tries + 1  # add 1 try
```
 
#### Key Functions & Concepts
 
| Element | Description |
|---|---|
| `random.randint(a, b)` | Returns a random integer between `a` and `b` (inclusive) |
| `input(prompt)` | Displays `prompt` and returns the user's typed response **as text (a string)** |
| `int(text)` | Converts a string to an integer — required since `input()` always returns text |
| `print(message)` | Displays output to the screen |
 
> **Why convert with `int()`?** `input()` always returns a string, even if the user types digits. Comparing a string `"10"` to an integer `10` would not behave as expected, so explicit conversion is required before numeric comparison.
 
**Limitation:** This version reads a guess but never compares it to the secret number — functionally incomplete as a game.
 
---
 
### Version 2: Adding Conditional Logic
 
**Goal:** Compare the user's guess to the secret number and provide feedback.
 
#### Decision Logic (Pseudocode)
 
```
If guess is less than 1 OR greater than 20:
    → "Out of range"
Else if guess is less than secret:
    → "Too low"
Else if guess is greater than secret:
    → "Too high"
Else:
    → "You got it"
```
 
#### Python Implementation
 
```python
import random
 
secret = random.randint(1, 20)
tries = 0
guess = 0
 
print("I'm thinking of a number between 1 and 20")
 
text = input("Take a guess: ")
guess = int(text)
 
tries = tries + 1
 
# Give a hint using if / elif / else.
if guess < 1 or guess > 20:
    print("That number is out of range. Try again.")
elif guess < secret:
    print("Too low, try again.")
elif guess > secret:
    print("Too high, try again.")
else:
    print("You got it in", tries, "tries!")
```
 
#### Conditional Flow Control
 
| Keyword | Python Syntax | Behavior |
|---|---|---|
| **if** | `if CONDITION:` | Checked first; runs if true |
| **elif** | `elif CONDITION:` | Checked only if the prior `if`/`elif` was false |
| **else** | `else:` | Runs only if every preceding condition was false |
 
```
if condition_1:
    → run this
elif condition_2:
    → only checked if condition_1 was False
elif condition_3:
    → only checked if condition_1 AND condition_2 were False
else:
    → runs only if ALL above conditions were False
```
 
#### Worked Examples (`secret = 10`)
 
| User Input | Evaluation | Output |
|---|---|---|
| `30` | `30 > 20` → True | "That number is out of range. Try again." |
| `5` | Not out of range; `5 < 10` → True | "Too low, try again." |
| `15` | Not out of range; not `< 10`; `15 > 10` → True | "Too high, try again." |
| `10` | Not out of range; not `<`; not `>` → falls to `else` | "You got it in 1 tries!" |
 
**Limitation:** The user only gets **one** attempt — the program ends regardless of the outcome.
 
---
 
### Version 3: Adding a Loop for Repeated Guessing
 
**Goal:** Allow unlimited attempts until the correct number is guessed.
 
#### The Concept of Iteration
 
A **loop** repeats a block of code as long as a specified condition remains true. Real-world analogies:
 
| Scenario | Repeated Action | Stopping Condition |
|---|---|---|
| Shopping for a t-shirt | Visit shop after shop | Suitable t-shirt found |
| Finding a parking space | Check row after row | Empty space found |
| Guessing game | Prompt for another guess | Correct number entered |
 
#### The `while` Loop
 
```python
while CONDITION:
    # repeated code block
```
 
In Python, **"not equal to"** is written as `!=`. The loop in this program runs **while the guess does not equal the secret number**:
 
```python
while guess != secret:
```
 
#### Complete Implementation
 
```python
import random  # gives us tools for picking random numbers
 
# ----------------------------
# Guess the Number (Beginner Demo)
# ----------------------------
# The computer picks a secret number.
# The player keeps guessing until they find it.
 
secret = random.randint(1, 20)
tries = 0
guess = 0  # start with a value that cannot be the secret (since secret is 1..20)
 
print("I'm thinking of a number between 1 and 20")
 
# Repeat until the user guesses the secret number.
while guess != secret:
    text = input("Take a guess: ")
    guess = int(text)
 
    tries = tries + 1
 
    # Give a hint using if / elif / else.
    if guess < 1 or guess > 20:
        print("That number is out of range. Try again.")
    elif guess < secret:
        print("Too low, try again.")
    elif guess > secret:
        print("Too high, try again.")
    else:
        print("You got it in", tries, "tries!")
```
 
#### How the Loop Evaluates Each Pass
 
| Iteration | `guess` value | `guess != secret`? | Action |
|---|---|---|---|
| Before loop starts | `0` (initial) | True (`10 != 0`) | Loop body executes |
| After incorrect guess | e.g., `5` | True (`10 != 5`) | Loop body executes again |
| After correct guess | `10` | **False** (`10 != 10` is False) | Loop terminates |
 
> The loop's condition is re-evaluated **every time** before running the code block again. Once the guess matches the secret, the condition becomes false and execution exits the loop naturally — no separate "stop" instruction is needed.
 
---
