 Linux Shells & Bash Scripting
 
### What is a Shell?
 
A **shell** is the command-line interface through which users interact with the Linux operating system. Most Linux distributions default to **Bash** (Bourne Again Shell), though others are available and switchable.
 
---
 
### Core Shell Commands (Recap)
 
| Command | Purpose |
|---|---|
| `pwd` | Print Working Directory — show current location |
| `cd <directory>` | Change directory |
| `ls` | List directory contents |
| `cat <file>` | Display file contents |
| `grep <pattern> <file>` | Search for a pattern within a file |
 
**Example — `grep` usage:**
```bash
user@tryhackme:~$ grep THM dictionary.txt
The flag is THM
```
 
---
 
### Identifying and Switching Shells
 
#### Check Your Current Shell
 
```bash
echo $SHELL
```
```
/bin/bash
```
 
#### List Available Shells
 
```bash
cat /etc/shells
```
```
/bin/sh
/bin/bash
/bin/dash
/bin/zsh
/usr/bin/zsh
/usr/bin/fish
/usr/bin/tmux
```
 
#### Switch Shell (Temporary)
 
```bash
zsh      # opens Zsh in the current session
```
 
#### Change Default Shell (Permanent)
 
```bash
chsh -s /usr/bin/zsh
```
 
---
 
### Shell Comparison
 
| Feature | Bash | Fish | Zsh |
|---|---|---|---|
| **Full Name** | Bourne Again Shell | Friendly Interactive Shell | Z Shell |
| **Default on most distros** | Yes | No | No |
| **Scripting** | Widely compatible, extensive docs | Limited | Excellent; extends Bash capabilities |
| **Tab completion** | Basic | Advanced (history-aware suggestions) | Highly extensible via plugins |
| **Spell correction** | No | Yes (built-in) | Yes (built-in) |
| **Syntax highlighting** | No | Yes (built-in) | Via plugins |
| **Customization** | Basic | Good (interactive tools) | Advanced (oh-my-zsh framework) |
| **User-friendliness** | Familiar to most users | Most beginner-friendly | Highly configurable |
 
> The best shell depends on your priorities — **Bash** for maximum compatibility and documentation, **Fish** for approachability, **Zsh** for a balance of power and extensibility.
 
---
 
### Bash Scripting
 
A **shell script** is a file containing a sequence of shell commands that can be executed together, automating repetitive or complex tasks.
 
#### Creating a Script File
 
Scripts use the `.sh` extension and are created with any text editor:
 
```bash
nano first_script.sh
```
 
#### The Shebang Line
 
Every script must begin with a **shebang** (`#!`) followed by the path to the interpreter — this tells the OS which shell to use to execute the file:
 
```bash
#!/bin/bash
```
 
> Always the **first line** of a script. Without it, the OS may default to a different interpreter, causing unexpected behavior.
 
#### Making a Script Executable
 
Before running, grant execution permission with `chmod`:
 
```bash
chmod +x first_script.sh
```
 
#### Running a Script
 
```bash
./first_script.sh
```
 
> The `./` prefix tells the shell to execute the file **in the current directory** specifically. Without it, the shell searches only directories listed in `$PATH` and will not find local scripts.
 
---
 
### Script Building Blocks
 
#### Variables
 
Store and reuse values — referenced with a `$` prefix:
 
```bash
#!/bin/bash
echo "Hey, what's your name?"
read name
echo "Welcome, $name"
```
 
```
Hey, what's your name?
John
Welcome, John
```
 
| Element | Purpose |
|---|---|
| `echo "text"` | Print text to the terminal |
| `read <variable>` | Capture user input and store in a variable |
| `$name` | Reference the value stored in `name` |
 
---
 
#### Loops
 
Repeat a block of code for each item in a range or list:
 
```bash
#!/bin/bash
for i in {1..10};
do
    echo $i
done
```
 
```
1
2
3
...
10
```
 
| Element | Purpose |
|---|---|
| `for i in {1..10}` | Iterates `i` through values 1 to 10 |
| `do` | Marks the **start** of the loop body |
| `done` | Marks the **end** of the loop body |
| `$i` | Current iteration's value |
 
---
 
#### Conditional Statements
 
Execute different code depending on whether a condition is true or false:
 
```bash
#!/bin/bash
echo "Please enter your name first:"
read name
 
if [ "$name" = "Stewart" ]; then
    echo "Welcome Stewart! Here is the secret: THM_Script"
else
    echo "Sorry! You are not authorized to access the secret."
fi
```
 
| Element | Purpose |
|---|---|
| `if [ condition ]; then` | Evaluates condition; runs body if true |
| `else` | Code to run if the `if` condition is false |
| `fi` | Ends the conditional block (reverse of `if`) |
| `[ "$name" = "Stewart" ]` | Tests whether variable equals a string |
| `-eq` | Numeric equality (use instead of `=` for integers) |
| `&&` | Logical AND — both conditions must be true |
 
**Execution examples:**
```bash
$ ./conditional_script.sh
Please enter your name first:
Stewart
Welcome, Stewart! Here is the secret: THM_Script
 
$ ./conditional_script.sh
Please enter your name first:
Alex
Sorry! You are not authorized to access the secret.
```
 
---
 
#### Comments
 
Improve readability and maintainability without affecting execution — prefixed with `#`:
 
```bash
#!/bin/bash
 
# Asking the user to enter a value.
echo "Please enter your name first:"
 
# Storing the user input value in a variable.
read name
 
# Checking if the name matches the authorized user.
if [ "$name" = "Stewart" ]; then
    echo "Welcome, Stewart!"
else
    echo "Access denied."
fi
```
 
> Comments don't affect script behavior. Best practice: comment **major and complex areas** rather than every single line. The shebang line itself (`#!/bin/bash`) is technically a special comment.
 
---
 
### Worked Example — Locker Authentication Script
 
Combines all four building blocks (variables, loops, conditionals, comments) into a practical script.
 
**Requirements:** Prompt for username, company name, and PIN. Grant access only if all three match.
 
```bash
#!/bin/bash
 
# Defining the variables
username=""
companyname=""
pin=""
 
# Loop to collect three inputs sequentially
for i in {1..3}; do
 
    if [ "$i" -eq 1 ]; then
        echo "Enter your Username:"
        read username
 
    elif [ "$i" -eq 2 ]; then
        echo "Enter your Company name:"
        read companyname
 
    else
        echo "Enter your PIN:"
        read pin
    fi
 
done
 
# Verify all three credentials match
if [ "$username" = "John" ] && [ "$companyname" = "Tryhackme" ] && [ "$pin" = "7385" ]; then
    echo "Authentication Successful. You can now access your locker, John."
else
    echo "Authentication Denied!!"
fi
```
 
**Sample runs:**
 
```bash
# Correct credentials
$ ./locker_script.sh
Enter your Username:
John
Enter your Company name:
Tryhackme
Enter your PIN:
7385
Authentication Successful. You can now access your locker, John.
 
# Incorrect PIN
$ ./locker_script.sh
Enter your Username:
John
Enter your Company name:
Tryhackme
Enter your PIN:
1349
Authentication Denied!!
```
 
---
 
### Scripting Quick Reference
 
| Element | Syntax | Notes |
|---|---|---|
| Shebang | `#!/bin/bash` | First line of every script |
| Print output | `echo "text"` | Quotes recommended for strings with spaces |
| Read user input | `read variable` | Stores input in `variable` |
| Reference variable | `$variable` | Use double quotes: `"$variable"` for safety |
| String equality | `[ "$a" = "$b" ]` | Use `=` for strings |
| Numeric equality | `[ "$a" -eq "$b" ]` | Use `-eq` for integers |
| Logical AND | `&&` | Both conditions must be true |
| For loop | `for i in {1..N}; do ... done` | Iterates N times |
| Conditional | `if [ cond ]; then ... elif ... else ... fi` | `fi` closes the block |
| Comment | `# comment text` | Ignored by the interpreter |
| Make executable | `chmod +x script.sh` | Required before first run |
| Execute script | `./script.sh` | `./` specifies current directory |
 
---
