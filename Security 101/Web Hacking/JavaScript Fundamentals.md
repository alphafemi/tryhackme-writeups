# JavaScript Fundamentals for Web Security
 

 
---
 
JavaScript (JS) is the primary language for adding interactivity to web pages. It works alongside HTML (structure) and CSS (styling) and runs directly in the browser — making it highly visible and a key attack surface in web security.
 
---
 
### Core Language Concepts
 
#### Variables
 
Three declaration keywords, each with different scoping behavior:
 
| Keyword | Scope | Mutable? | Notes |
|---|---|---|---|
| `var` | Function-scoped | Yes | Older style — avoid in modern JS |
| `let` | Block-scoped | Yes | Preferred for values that change |
| `const` | Block-scoped | No (reference) | Preferred for values that shouldn't change |
 
```javascript
let age = 25;         // can be reassigned
const PI = 3.14159;   // cannot be reassigned
var name = "Alice";   // function-scoped (older)
```
 
#### Data Types
 
| Type | Example | Notes |
|---|---|---|
| `string` | `"hello"` | Text |
| `number` | `42`, `3.14` | Integers and floats combined |
| `boolean` | `true`, `false` | |
| `null` | `null` | Intentional absence of value |
| `undefined` | `undefined` | Variable declared but not assigned |
| `object` | `{name: "Alice"}` | Complex data — arrays, objects |
 
#### Functions
 
Reusable blocks of code that perform a specific task:
 
```javascript
function PrintResult(rollNum) {
    alert("Roll number " + rollNum + " has passed");
}
 
PrintResult(101);
```
 
#### Loops
 
Repeat a block of code while a condition is true:
 
```javascript
const rollNumbers = [101, 102, 103];
 
for (let i = 0; i < rollNumbers.length; i++) {
    PrintResult(rollNumbers[i]);
}
```
 
Common loop types: `for`, `while`, `do...while`
 
#### Conditional Statements
 
```javascript
let age = prompt("What is your age?");
 
if (age >= 18) {
    document.getElementById("message").innerHTML = "You are an adult.";
} else {
    document.getElementById("message").innerHTML = "You are a minor.";
}
```
 
---
 
### Integrating JavaScript into HTML
 
#### Internal JS
 
Script embedded directly in the HTML file using `<script>` tags:
 
```html
<!DOCTYPE html>
<html>
<body>
    <p id="result"></p>
    <script>
        let x = 5;
        let y = 10;
        document.getElementById("result").innerHTML = "Result: " + (x + y);
    </script>
</body>
</html>
```
 
> `<script>` can go in `<head>` (loaded before content) or `<body>` (loaded as content renders).
 
#### External JS
 
Script stored in a separate `.js` file, referenced via the `src` attribute:
 
```html
<!-- external.html -->
<script src="script.js"></script>
```
 
```javascript
// script.js
let x = 5;
let y = 10;
document.getElementById("result").innerHTML = "Result: " + (x + y);
```
 
**Benefits of external JS:** Keeps HTML clean, encourages code reuse, easier maintenance.
 
#### Identifying Internal vs. External JS (Recon)
 
**View Page Source** → Right-click → "View Page Source"
 
| What you see | Meaning |
|---|---|
| `<script>` with code inside | Internal JS |
| `<script src="file.js">` | External JS loaded from same server |
| `<script src="https://cdn.example.com/lib.js">` | External JS from a third-party CDN |
 
> When pen-testing, checking for internal JS often reveals hardcoded credentials, API keys, internal endpoints, or logic that should be server-side only.
 
---
 
### User Interaction Functions
 
| Function | Behavior | Returns |
|---|---|---|
| `alert("msg")` | Displays a message box with OK | Nothing |
| `prompt("msg")` | Asks for user input | String input (or `null` if cancelled) |
| `confirm("msg")` | Asks yes/no — OK or Cancel | `true` or `false` |
 
```javascript
// Alert
alert("Hello THM");
 
// Prompt
let name = prompt("What is your name?");
alert("Hello " + name);
 
// Confirm
let proceed = confirm("Do you want to continue?");
if (proceed) { /* user clicked OK */ }
```
 
#### How Attackers Exploit These
 
Malicious HTML files distributed via email or download can weaponize these functions:
 
```html
<script>
for (let i = 0; i < 500; i++) {
    alert("Hacked");   // Forces user to click OK 500 times
}
</script>
```
 
> Only execute JS files from trusted sources. Even a simple `.html` attachment can run arbitrary JS in the browser.
 
---
 
### Minification & Obfuscation
 
#### Minification
 
Removes whitespace, comments, and shortens variable names to reduce file size and improve load times. Minified code is functionally identical but unreadable to humans.
 
#### Obfuscation
 
Deliberately transforms code to make it harder to understand — renaming variables to meaningless names, adding dummy code, encoding strings. Often used to protect proprietary logic, but also used by attackers to hide malicious code.
 
**Original:**
```javascript
function hi() {
    alert("Welcome to THM");
}
hi();
```
 
**Obfuscated (same function, unreadable):**
```javascript
(function(_0x114713,_0x2246f2){var _0x51a830=_0x33bf ...
```
 
> Obfuscation is **not encryption** — it can be reversed. Tools like [deobfuscate.io](https://deobfuscate.io) or browser DevTools → Sources can recover readable code.
 
#### Deobfuscating in Practice
 
1. Open browser DevTools (`F12`) → **Sources** tab
2. Find and open the JS file
3. Use **Pretty Print** (`{}` button) to format minified code
4. For heavily obfuscated code, use online deobfuscators or manually trace execution
---
 
### Security Best Practices
 
| Practice | Why It Matters |
|---|---|
| **Never rely solely on client-side validation** | Users can disable/manipulate JS in the browser — always validate server-side too |
| **Only include trusted libraries** | Malicious packages with similar names to legitimate ones exist ("typosquatting") — verify sources |
| **Never hardcode secrets in JS** | Source code is visible to anyone — API keys, tokens, and credentials in JS are exposed |
| **Minify and obfuscate production JS** | Reduces readability for attackers — not a security guarantee, but raises the effort required |
| **Sanitize all user input** | Unvalidated input in JS enables XSS attacks — never trust data from `prompt()`, query strings, or form fields |
 
#### Example of a Hardcoded Secret (Bad Practice)
 
```javascript
// BAD — anyone viewing source code sees this
const privateAPIKey = 'pk_TryHackMe-1337';
```
 
> Move secrets to the server side where they cannot be retrieved by inspecting client-side source code.
 
---
 
### JavaScript & the Browser Console
 
The browser console (`Ctrl+Shift+I` → Console tab) executes JS directly in the page context — useful for both development and security testing:
 
```javascript
// Run directly in browser console
let x = 5;
let y = 10;
console.log("Result: " + (x + y));   // → Result: 15
```
 
> During a web application assessment, the browser console can be used to test input handling, inspect variables, manipulate DOM elements, and experiment with application logic.
 
---
 
### Quick Reference
 
| Task | Code |
|---|---|
| Declare a variable | `let name = "Alice";` |
| Declare a constant | `const MAX = 100;` |
| Display a message | `alert("msg");` |
| Get user input | `let val = prompt("Enter value:");` |
| Print to console | `console.log("msg");` |
| Update page element | `document.getElementById("id").innerHTML = "text";` |
| For loop | `for (let i = 0; i < n; i++) { }` |
| If-else | `if (condition) { } else { }` |
| Link external JS | `<script src="file.js"></script>` |
| Embed internal JS | `<script> /* code */ </script>` |
 
---
 
## Summary
 
- JavaScript runs directly in the browser alongside HTML and CSS, making it a highly visible and frequently targeted attack surface.
- Variables are declared with `var` (function-scoped, legacy), `let` (block-scoped, mutable), or `const` (block-scoped, immutable reference) — core data types are string, number, boolean, null, undefined, and object.
- JS can be embedded **internally** (inline `<script>` tags) or loaded **externally** (`<script src="...">`) — reviewing page source for internal JS during recon can reveal hardcoded credentials, API keys, or internal endpoints.
- User interaction functions (`alert`, `prompt`, `confirm`) can be weaponized in malicious HTML files to disrupt or manipulate a victim's browser session.
- **Minification** shrinks code for performance; **obfuscation** deliberately hides logic but is reversible, not encryption — browser DevTools' Pretty Print and online deobfuscators can recover readable code.
- Key security practices: never rely solely on client-side validation, only use trusted/verified libraries, never hardcode secrets in JS, minify/obfuscate production code, and sanitize all user input to prevent XSS.
