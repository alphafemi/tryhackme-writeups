JavaScript Programming Fundamentals (Node.js)
 
This section mirrors [Section 25](#25-python-programming-fundamentals), implementing the same "Guess the Number" game in **JavaScript**, executed via **Node.js**.
 
### Running JavaScript
 
| Environment | How to Run |
|---|---|
| **Web Browser** | Open Developer Tools (`F12` in most browsers) → Console tab |
| **Node.js** | `node filename.js` from the command line |
 
> This section uses Node.js exclusively, since browser-based JavaScript is oriented around web pages rather than command-line interaction.
 
---
 
### Version 1: Variables, Constants & User Input
 
#### Declaring Variables vs. Constants
 
| Keyword | Mutability | Use Case |
|---|---|---|
| `let` | Value **can** change | `tries`, `guess` — values that update during execution |
| `const` | Value **cannot** change after assignment | `secret` — fixed for the duration of a single game |
 
```javascript
let tries = 0;
let guess = 0;
```
 
#### Generating a Random Secret Number
 
```javascript
const secret = Math.floor(Math.random() * (20)) + 1; // 1 <= secret <= 20
```
 
| Expression Component | Effect | Example |
|---|---|---|
| `Math.random()` | Random decimal between `0` (inclusive) and `1` (exclusive) | `0.372` |
| `* 20` | Stretches the range to `0`–~`20` | `7.44` |
| `Math.floor()` | Rounds down, removing the decimal | `7` |
| `+ 1` | Shifts the range from `0–19` to `1–20` | `8` |
 
#### Displaying Output
 
```javascript
console.log("I'm thinking of a number between 1 and 20");
```
 
#### Capturing User Input
 
Unlike browser JavaScript, Node.js does not pause execution to wait for typed input by default — additional setup is required:
 
```javascript
import * as readline from "node:readline/promises";
import { stdin as input, stdout as output } from "node:process";
 
const rl = readline.createInterface({ input, output });
```
 
| Import | Purpose |
|---|---|
| `readline/promises` | Enables asking questions and *awaiting* a typed response without freezing the program |
| `stdin` (renamed `input`) | The keyboard / standard input stream |
| `stdout` (renamed `output`) | The screen / standard output stream |
| `readline.createInterface()` | Builds the input/output channel (`rl`) used to prompt the user |
 
#### Reading and Converting Input
 
```javascript
const text = await rl.question("Take a guess: "); // returns text (a string)
guess = parseInt(text, 10);                        // convert text to a number
```
 
| Function | Purpose |
|---|---|
| `rl.question(prompt)` | Displays `prompt`, waits (`await`) for input, returns the response as a **string** |
| `parseInt(text, 10)` | Converts a string to an integer, explicitly specifying base 10 |
| `await` | Pauses execution until the asynchronous operation (waiting for user input) completes |
 
> **Why `await`?** Node.js is designed as a non-blocking runtime — it normally does *not* wait for anything. `await` (combined with the `/promises` readline module) explicitly overrides this default so the program pauses until the user responds.
 
#### Resource Cleanup with `try` / `finally`
 
```javascript
const rl = readline.createInterface({ input, output });
 
try {
  // program logic, including rl.question() calls
} finally {
  rl.close();
}
```
 
| Block | Purpose |
|---|---|
| `try { }` | Runs the program logic; isolates errors so they don't crash the entire script |
| `finally { }` | **Always** executes afterward — used here to close (`rl.close()`) the input interface, regardless of success or failure |
 
#### Complete Version 1
 
```javascript
import * as readline from "node:readline/promises";
import { stdin as input, stdout as output } from "node:process";
 
const rl = readline.createInterface({ input, output });
 
try {
  const secret = Math.floor(Math.random() * (20)) + 1; // 1 <= secret <= 20
  let tries = 0;
  let guess = 0; // start with a value that cannot be the secret (since secret is 1..20)
 
  console.log("I'm thinking of a number between 1 and 20");
 
  const text = await rl.question("Take a guess: "); // returns text (a string)
  guess = parseInt(text, 10); // convert the text to a number
 
  tries = tries + 1; // add 1 try
 
} finally {
  rl.close();
}
```
 
**Limitation:** Accepts a guess but never evaluates it — no feedback is given.
 
---
 
### Version 2: Conditional Feedback
 
#### Decision Logic
 
```
If guess < 1 OR guess > 20  → "Out of range"
Else if guess < secret      → "Too low"
Else if guess > secret      → "Too high"
Else                        → "You got it"
```
 
#### JavaScript Implementation
 
```javascript
// Give a hint using if / else if / else.
if (guess < 1 || guess > 20) {
  console.log("That number is out of range. Try again.");
} else if (guess < secret) {
  console.log("Too low, try again.");
} else if (guess > secret) {
  console.log("Too high, try again.");
} else {
  console.log("You got it in", tries, "tries!");
}
```
 
| Operator | Meaning |
|---|---|
| `\|\|` | Logical **OR** |
| `<` / `>` | Less than / greater than |
| `if` / `else if` / `else` | Conditional branching — evaluated top to bottom; the first true condition runs, and the rest are skipped |
 
#### Complete Version 2
 
```javascript
import * as readline from "node:readline/promises";
import { stdin as input, stdout as output } from "node:process";
 
const rl = readline.createInterface({ input, output });
 
try {
  const secret = Math.floor(Math.random() * (20)) + 1; // 1 <= secret <= 20
  let tries = 0;
  let guess = 0;
 
  console.log("I'm thinking of a number between 1 and 20");
 
  const text = await rl.question("Take a guess: ");
  guess = parseInt(text, 10);
 
  tries = tries + 1;
 
  // Give a hint using if / else if / else.
  if (guess < 1 || guess > 20) {
    console.log("That number is out of range. Try again.");
  } else if (guess < secret) {
    console.log("Too low, try again.");
  } else if (guess > secret) {
    console.log("Too high, try again.");
  } else {
    console.log("You got it in", tries, "tries!");
  }
} finally {
  rl.close();
}
```
 
#### Sample Run
 
```
$ node guess_v2.js
I'm thinking of a number between 1 and 20
Take a guess: 10
Too low, try again.
```
 
**Limitation:** The program ends after a single guess, regardless of the outcome.
 
---
 
### Version 3: Looping for Unlimited Attempts
 
#### The `while` Loop
 
```javascript
// Repeat until the user guesses the secret number.
while (guess !== secret) {
  // Loop body: instructions to be repeated
}
```
 
| Operator | Meaning |
|---|---|
| `!==` | "Not equal to" (strict inequality) |
 
The loop condition is checked **before each repetition**; once `guess` equals `secret`, the condition becomes false and the loop exits automatically.
 
#### Complete Version 3
 
```javascript
import * as readline from "node:readline/promises";
import { stdin as input, stdout as output } from "node:process";
 
const rl = readline.createInterface({ input, output });
 
try {
  const secret = Math.floor(Math.random() * (20)) + 1; // 1 <= secret <= 20
  let tries = 0;
  let guess = 0; // start with a value that cannot be the secret (since secret is 1..20)
 
  console.log("I'm thinking of a number between 1 and 20");
 
  // Repeat until the user guesses the secret number.
  while (guess !== secret) {
    const text = await rl.question("Take a guess: ");
    guess = parseInt(text, 10);
 
    tries = tries + 1;
 
    // Give a hint using if / else if / else.
    if (guess < 1 || guess > 20) {
      console.log("That number is out of range. Try again.");
    } else if (guess < secret) {
      console.log("Too low, try again.");
    } else if (guess > secret) {
      console.log("Too high, try again.");
    } else {
      console.log("You got it in", tries, "tries!");
    }
  }
} finally {
  rl.close();
}
```
 
#### Sample Run
 
```
$ node guess_v3.js
I'm thinking of a number between 1 and 20
Take a guess: 10
Too low, try again.
Take a guess: 15
Too high, try again.
Take a guess: 13
Too low, try again.
Take a guess: 14
You got it in 4 tries!
```
 
> A `guess_v4.js` exists with further refinements beyond the scope of this introductory walkthrough.
 
---
 
### Program Evolution Summary
 
| Version | File | Capability Added | Limitation |
|---|---|---|---|
| **v1** | `guess_v1.js` | Random secret + single input capture via `readline` | No comparison logic |
| **v2** | `guess_v2.js` | Conditional feedback (`if`/`else if`/`else`) | Only one attempt allowed |
| **v3** | `guess_v3.js` | Repeated attempts via `while` loop | Fully functional |
 
---
 
