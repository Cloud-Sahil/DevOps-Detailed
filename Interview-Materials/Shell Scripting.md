# Shell Scripting Interview Questions 
---

## Table of Contents

1. [Shell Scripting Fundamentals (Simple)](#module-1-shell-scripting-fundamentals-simple) — 14
2. [Variables & Data Types](#module-2-variables--data-types) — 14
3. [Input/Output & Redirection](#module-3-inputoutput--redirection) — 12
4. [Conditionals & Test Expressions](#module-4-conditionals--test-expressions) — 14
5. [Loops](#module-5-loops) — 14
6. [Functions](#module-6-functions) — 10
7. [Arrays](#module-7-arrays) — 10
8. [String Manipulation](#module-8-string-manipulation) — 10
9. [Text Processing: grep, sed, awk](#module-9-text-processing-grep-sed-awk) — 16
10. [Process Management & Job Control](#module-10-process-management--job-control) — 10
11. [Error Handling & Debugging](#module-11-error-handling--debugging) — 12
12. [File Operations & Permissions](#module-12-file-operations--permissions) — 8
13. [Advanced Real-World Scenarios](#module-13-advanced-real-world-scenarios) — 20

**Total: 164 Questions**

---

## MODULE 1: Shell Scripting Fundamentals (Simple)

**1. What is a shebang line, and what does it do?**
The `#!` line at the top of a script (e.g., `#!/bin/bash`) telling the OS which interpreter to use when the script is executed directly.

**2. How do you make a shell script executable?**
`chmod +x script.sh`, then run it with `./script.sh`.

**3. What's the difference between running `./script.sh` and `bash script.sh`?**
`./script.sh` requires the execute permission bit and uses the shebang to determine the interpreter; `bash script.sh` explicitly invokes bash regardless of the shebang or execute permission.

**4. What does `#!/usr/bin/env bash` do differently from `#!/bin/bash`?**
It searches the user's `PATH` to find `bash` wherever it's installed, rather than assuming it's always at the fixed path `/bin/bash` — more portable across systems where bash might be installed elsewhere.

**5. What does `$?` represent?**
The exit status/code of the most recently executed command.

**6. What is considered a "successful" exit code by convention?**
`0` — any non-zero value conventionally indicates failure or a specific error condition.

**7. What is `$0` in a script?**
The name of the script itself as it was invoked.

**8. What's the difference between `$@` and `$*`?**
`$@` treats each positional argument as a separate word when quoted (`"$@"`); `$*` treats all arguments as a single combined string.

**9. What does `$#` represent?**
The number of positional arguments passed to the script or function.

**10. How do you add a comment in a shell script?**
Using `#` at the start of a line for a single-line comment.

**11. What is the difference between a login shell and a non-login shell?**
A login shell reads profile files (`.bash_profile`, `.profile`) typically when you log in; a non-login shell (like most subshells or scripts) typically doesn't, often resulting in a different/reduced environment.

**12. What command checks a script's syntax without actually running it?**
`bash -n script.sh`.

**13. What is ShellCheck, and why is it recommended?**
A static analysis tool for shell scripts that catches common mistakes (unquoted variables, incorrect test operators, subshell scope issues) before they cause runtime bugs.

**14. What's the difference between `sh` and `bash`?**
`sh` refers to the POSIX-standard shell (often symlinked to `dash` on many Linux distributions, a smaller/faster but less feature-rich shell); `bash` is a specific, more feature-rich shell that's a superset of POSIX `sh` with additional extensions.

---

## MODULE 2: Variables & Data Types

**15. How do you declare a variable in Bash, and what's a critical syntax rule?**
`name="value"` — critically, there must be NO spaces around the `=` sign, or Bash interprets it as a command invocation instead of an assignment.

**16. What is the difference between `local` and a regular variable assignment inside a function?**
`local` scopes the variable to that function only; without it, the variable becomes global and persists/leaks outside the function after it returns.

**17. Scenario: A function modifies a variable that was supposed to be temporary, but the change unexpectedly persists after the function returns and affects the rest of the script. What's the likely cause?**
The variable inside the function was declared without `local`, causing it to be treated as a global variable — any assignment inside the function directly modified the global scope rather than a function-local copy.

**18. What does `export` do to a variable?**
Makes the variable part of the environment passed down to any child processes spawned from that shell — without `export`, a variable is only visible within the current shell/script itself.

**19. Scenario: A script sets an environment variable and then calls another script, but the second script can't see that variable. What's the fix?**
`export` the variable in the first script before invoking the second — a plain (non-exported) assignment is not inherited by child processes.

**20. What does `readonly` do to a variable?**
Prevents the variable's value from being changed or unset after it's declared — attempting to reassign it produces an error.

**21. What is `$$` used for?**
The process ID (PID) of the currently running script/shell — commonly used to generate unique temporary filenames.

**22. What is `$!` used for?**
The PID of the most recently launched background process.

**23. Scenario: You need to perform floating-point division in a Bash script, but `$(( 10 / 3 ))` only gives you `3`. How do you get a decimal result?**
Bash's native arithmetic only supports integers — use an external tool like `bc` (`echo "10/3" | bc -l`) or `awk` (`awk "BEGIN {print 10/3}"`) for floating-point math.

**24. What's the difference between `${var:-default}` and `${var:=default}`?**
`${var:-default}` uses the default value if `var` is unset/empty but doesn't change `var` itself; `${var:=default}` does the same but also assigns the default value to `var` for subsequent use.

**25. Scenario: How would you write a script that fails loudly with a clear error message if a required variable was never set, rather than silently proceeding with an empty value?**
Use `${var:?Error message}` — this prints the error message and exits the script immediately if `var` is unset or empty, rather than silently continuing.

**26. What does `unset` do?**
Removes a variable (or function) entirely from the current shell's scope.

**27. Scenario: You need a variable's default value to be used only if it's currently unset, but an intentionally empty string should be treated as a valid value (not triggering the default). How do you distinguish this?**
Use `${var-default}` (without the colon) rather than `${var:-default}` — the colon version treats empty-and-unset identically, while omitting the colon only triggers the default for a genuinely unset variable, treating an explicitly-set empty string as a valid (non-default) value.

**28. What is the difference between a Bash variable and a Bash array in terms of declaration?**
A regular variable holds a single scalar value (`var="value"`); an array holds multiple values accessed by index (`arr=(one two three)`), declared with parentheses.

---

## MODULE 3: Input/Output & Redirection

**29. What are the three standard streams in a shell, and their file descriptor numbers?**
stdin (0), stdout (1), stderr (2).

**30. What's the difference between `>` and `>>`?**
`>` redirects output to a file, overwriting any existing content; `>>` appends to the end of the file instead.

**31. Scenario: You need to capture both a command's normal output AND its error messages into the same single log file. What's the correct redirection syntax, and what's a common mistake?**
`command > logfile.txt 2>&1` is correct (redirect stdout to the file, then make stderr follow stdout to that same destination); the common mistake is writing `command 2>&1 > logfile.txt` — the order matters, and this incorrect version only sends stdout to the file while stderr still goes to the terminal.

**32. What does `2>/dev/null` do?**
Discards stderr output entirely, sending it to the null device instead of displaying it or saving it anywhere.

**33. How do you read user input into a variable interactively?**
`read -p "Prompt: " variable_name`.

**34. Scenario: You need to prompt for a password without it being visible as the user types. What `read` option helps?**
`read -s` suppresses the terminal echo of typed characters.

**35. What is a "here document" (heredoc), and give a basic example.**
A way to provide multi-line input to a command directly within the script, using `<<`:
```bash
cat << EOF
multi-line content here
EOF
```

**36. Scenario: You want a heredoc's content to be output literally, with `$variable` references NOT expanded. How do you achieve this?**
Quote the heredoc delimiter: `cat << 'EOF' ... EOF` — quoting the delimiter word disables variable/command substitution within the heredoc body.

**37. What does the `tee` command do, and why is it useful?**
Writes output to a file while simultaneously passing it through to stdout (or the next command in a pipe) — useful when you need to both save output to a log AND continue processing/viewing it live.

**38. Scenario: How would you view a long-running command's output live in the terminal while also saving it to a log file, and append (not overwrite) that log file across multiple runs?**
`command | tee -a logfile.txt`.

**39. What does `command < input.txt` do?**
Redirects the file's content to be used as the command's stdin, as if it were typed interactively.

**40. What is a "here string" (`<<<`), and how does it differ from a heredoc?**
Feeds a single string directly as a command's stdin in one line (`grep "pattern" <<< "$string"`) — simpler than a full heredoc block for feeding a single existing variable's content as input.

---

## MODULE 4: Conditionals & Test Expressions

**41. What's the difference between `[ ]` and `[[ ]]`?**
`[ ]` is the POSIX-standard test command (more portable across shells but less forgiving of unquoted variables); `[[ ]]` is a Bash-specific extended test construct that supports pattern matching and regex, and is generally safer with unquoted variables.

**42. Scenario: A script using `if [ $var = "value" ]` fails with "too many arguments" when `$var` is empty. Why, and what's the fix?**
When `$var` is empty and unquoted, `[ = "value" ]` is missing its first operand entirely, causing a syntax error from `[`'s perspective; fix by quoting: `[ "$var" = "value" ]`.

**43. What does `-z` test for, and `-n`?**
`-z` tests whether a string is empty (zero length); `-n` tests whether a string is non-empty.

**44. Scenario: How would you check if a file exists AND is readable before attempting to process it?**
```bash
if [ -f "$file" ] && [ -r "$file" ]; then
```

**45. What's the difference between `-e` and `-f` as file test operators?**
`-e` checks if the path exists at all (regardless of type — file, directory, symlink); `-f` checks specifically that it exists AND is a regular file (not a directory or special file).

**46. Scenario: You need to compare two numbers for "greater than" inside `[ ]`. Why would using `>` directly cause unexpected behavior, and what's the correct operator?**
Inside single `[ ]`, `>` is interpreted as shell output redirection (creating/overwriting a file), not numeric comparison — the correct operator is `-gt` (e.g., `[ "$a" -gt "$b" ]`), or use `(( a > b ))` in an arithmetic context where `>` works as expected.

**47. What is the arithmetic evaluation context `(( ))` used for?**
Performing and evaluating arithmetic expressions/comparisons directly (e.g., `(( a > b ))`, `(( count++ ))`) without needing the `-gt`/`-lt` style operators required inside `[ ]`.

**48. Scenario: How would you write a condition checking multiple criteria — a variable greater than 5 AND a file existing — using `[[ ]]`?**
```bash
if [[ "$count" -gt 5 && -f "$file" ]]; then
```

**49. What does the `case` statement provide that a long chain of `if`/`elif` doesn't as cleanly?**
Cleaner, more readable pattern-matching against a single value with multiple possible cases, including glob-style pattern matching and combining multiple patterns per case (e.g., `start|begin)`).

**50. Scenario: Write a `case` statement handling a script argument that could be "start", "stop", "restart" or "reload" (treating restart and reload identically), with a usage message for anything else.**
```bash
case "$1" in
    start) echo "Starting" ;;
    stop) echo "Stopping" ;;
    restart|reload) echo "Restarting" ;;
    *) echo "Usage: $0 {start|stop|restart|reload}"; exit 1 ;;
esac
```

**51. What does `[[ "$str" =~ $pattern ]]` do?**
Performs Bash's native regular expression matching, testing whether `$str` matches the regex stored in `$pattern`.

**52. Scenario: How would you check if a string contains ONLY digits using an `[[ ]]` regex test?**
```bash
if [[ "$str" =~ ^[0-9]+$ ]]; then
```

**53. What is the difference between testing `[ "$var" ]` and `[ -n "$var" ]`?**
They're functionally equivalent — both test for non-empty string truthiness — but `-n` is more explicit/readable about the actual intent being tested.

**54. Scenario: A conditional that should only match an exact string is instead matching partial/similar strings unexpectedly. What might be happening, and what test construct fixes it?**
The condition might be using `[[ "$str" == $pattern ]]` with an unquoted pattern containing glob-wildcard characters (like `*`) being interpreted as pattern matching rather than literal string comparison — quote the pattern (`"$pattern"`) if literal exact-match comparison is intended, since unquoted patterns in `[[ ]]` enable glob-style matching.

---

## MODULE 5: Loops

**55. What are the three main loop constructs in Bash?**
`for`, `while`, and `until`.

**56. What's the difference between `while` and `until`?**
`while` continues looping as long as its condition is TRUE; `until` continues looping as long as its condition is FALSE (i.e., until it becomes true) — logical inverses of each other.

**57. Scenario: You need to loop through a range of numbers from 1 to 100, but only the even numbers. How would you do this efficiently?**
```bash
for i in $(seq 2 2 100); do echo $i; done
# or
for ((i=2; i<=100; i+=2)); do echo $i; done
```

**58. What's the correct, safe way to read a file line by line in Bash?**
```bash
while IFS= read -r line; do
    echo "$line"
done < file.txt
```

**59. Scenario: Why is `IFS= read -r` specifically recommended over just `read` when reading lines from a file?**
`IFS=` prevents leading/trailing whitespace from being stripped from each line; `-r` prevents backslash escape sequences in the input from being interpreted/altered, preserving the line's content exactly as it appears in the file.

**60. Scenario: A script counts lines using a `while` loop piped from `cat`, but the counter variable shows 0 after the loop completes even though lines were clearly processed inside it. What's happening, and how do you fix it?**
The pipe (`cat file.txt | while read ...`) creates a subshell for the `while` loop, so any variable changes inside it don't persist to the parent shell after the loop ends; fix by redirecting input directly instead of piping: `while read ...; done < file.txt`.

**61. What does `break 2` do inside a nested loop?**
Breaks out of two levels of nested loops at once, rather than just the innermost loop that a plain `break` would exit.

**62. Scenario: How would you skip processing empty lines while looping through a file's content?**
```bash
while IFS= read -r line; do
    [ -z "$line" ] && continue
    echo "$line"
done < file.txt
```

**63. What's the syntax for a C-style `for` loop in Bash?**
`for ((i=0; i<10; i++)); do ... done`.

**64. Scenario: You need to iterate over all `.log` files in a directory, but there might be zero matching files — a naive glob loop would then iterate once with the literal unmatched pattern as the "filename." How do you guard against this?**
Enable `nullglob` (`shopt -s nullglob`) before the loop, which causes a glob pattern matching no files to expand to nothing (rather than the literal pattern string) — so the loop simply doesn't execute at all if no files match.

**65. What does `for i in {1..10..2}; do ... done` produce?**
Numbers 1, 3, 5, 7, 9 — brace expansion with a step value of 2.

**66. Scenario: How would you process a list of items in parallel (not sequentially) using a shell loop, limiting to a maximum number of concurrent background jobs?**
```bash
for item in "${items[@]}"; do
    process "$item" &
    while [ $(jobs -r | wc -l) -ge 4 ]; do wait -n; done   # cap at 4 concurrent jobs
done
wait   # wait for all remaining background jobs to finish
```

**67. What's a key difference between a `for` loop over `"$@"` versus `$*` when processing script arguments?**
`for arg in "$@"` correctly treats each argument as a separate item even if an argument contains spaces; `for arg in $*` (or even `"$*"`) collapses/word-splits differently and can incorrectly merge or mis-split arguments containing spaces.

**68. Scenario: How would you write a retry loop that attempts a command up to 3 times with a delay between attempts, succeeding early if the command succeeds?**
```bash
for attempt in 1 2 3; do
    if my_command; then
        echo "Success on attempt $attempt"
        break
    fi
    echo "Attempt $attempt failed, retrying..."
    sleep 5
done
```

---

## MODULE 6: Functions

**69. How do you define a function in Bash?**
```bash
my_function() {
    echo "Hello"
}
```

**70. How do you access arguments passed to a function?**
Via `$1`, `$2`, etc. (identical positional parameter syntax to script-level arguments), and `$#`/`$@` for count/all-arguments within the function's own scope.

**71. Scenario: How does a Bash function "return" a computed value, given that `return` only supports exit-status integers (0-255)?**
Echo the value to stdout and capture it via command substitution from the caller: `result=$(my_function args)` — `return` is reserved for signaling success/failure status, not for returning arbitrary data.

**72. What does `return 1` inside a function actually do?**
Sets the function's own exit status to 1 (failure, by convention), checkable via `$?` immediately after calling the function, or directly usable in a conditional (`if my_function; then`).

**73. Scenario: A function modifies what was intended to be a local counter variable, but the change unexpectedly affects a same-named variable used elsewhere in the script. What's the fix?**
Declare the variable with `local` inside the function — without it, the variable is global by default and any modification leaks outside the function's intended scope.

**74. How would you pass an entire array to a function and have it correctly received as separate elements?**
Call it with `my_function "${my_array[@]}"`, and inside the function, capture with `local arr=("$@")`.

**75. Scenario: How would you write a function that validates its own required arguments were provided, exiting with a usage message if not?**
```bash
my_function() {
    if [ $# -lt 2 ]; then
        echo "Usage: my_function <arg1> <arg2>" >&2
        return 1
    fi
    # ... function logic ...
}
```

**76. Can a Bash function be recursive? Give a simple example.**
Yes:
```bash
factorial() {
    if [ "$1" -le 1 ]; then
        echo 1
    else
        echo $(( $1 * $(factorial $(( $1 - 1 ))) ))
    fi
}
```

**77. Scenario: How would you make a utility function's logic available across multiple separate scripts without duplicating the code in each one?**
Define the functions in a separate file (e.g., `lib.sh`) and `source`/`.` it at the top of each script that needs it: `source ./lib.sh` or `. ./lib.sh` — making all functions/variables in `lib.sh` available in the sourcing script's context.

**78. What's the difference between calling a script normally versus `source`-ing it?**
Running a script normally (`./script.sh`) executes it in a new subshell/process, so any variable/function definitions inside it don't persist in your current shell; `source`-ing (or `. script.sh`) executes it within the CURRENT shell's context, so its variables/functions/`cd` commands do persist afterward.

---

## MODULE 7: Arrays

**79. How do you declare a simple indexed array in Bash?**
`arr=(one two three)`.

**80. How do you access all elements of an array?**
`"${arr[@]}"`.

**81. Scenario: How would you get the total number of elements in an array?**
`"${#arr[@]}"`.

**82. What's the difference between `${arr[@]}` and `${arr[*]}` when quoted?**
`"${arr[@]}"` expands to each element as a separate quoted word (preserving elements with internal spaces correctly); `"${arr[*]}"` expands to all elements joined into a single string (using the first character of `$IFS` as separator).

**83. How do you declare an associative array (key-value pairs) in Bash?**
`declare -A my_map` followed by `my_map[key]="value"` — requires Bash 4.0 or later.

**84. Scenario: How would you iterate over both the keys and values of an associative array?**
```bash
for key in "${!my_map[@]}"; do
    echo "$key -> ${my_map[$key]}"
done
```

**85. What does `unset arr[1]` do to an indexed array, and what's a common gotcha afterward?**
Removes that specific element, but leaves a "gap" in the index sequence (the array becomes non-contiguous) — a common gotcha is that a subsequent `for ((i=0; i<${#arr[@]}; i++))` loop using sequential indices will skip or miss elements because the index count no longer matches contiguous positions; iterating with `"${arr[@]}"` directly avoids this issue.

**86. Scenario: How would you get the last element of an array without knowing its exact length in advance?**
`"${arr[-1]}"` (Bash 4.3+), or `"${arr[@]: -1}"` (note the required space before the minus sign) for broader compatibility.

**87. How would you append a new element to the end of an existing array?**
`arr+=("new_element")`.

**88. Scenario: How would you convert a comma-separated string into a Bash array?**
```bash
IFS=',' read -ra my_array <<< "a,b,c"
```

---

## MODULE 8: String Manipulation

**89. How do you get the length of a string variable?**
`"${#str}"`.

**90. How do you extract a substring starting at a specific position?**
`"${str:position}"` or `"${str:position:length}"` for a bounded-length substring.

**91. Scenario: How would you replace ALL occurrences of a character in a string, not just the first?**
`"${str//old/new}"` — double slash for global replacement, versus a single slash (`${str/old/new}`) which only replaces the first occurrence.

**92. How do you convert a string to uppercase in native Bash (without an external tool)?**
`"${str^^}"` (Bash 4+); use `"${str,,}"` for lowercase.

**93. Scenario: How would you check if a string starts with a specific prefix?**
```bash
if [[ "$str" == prefix* ]]; then
```

**94. How would you remove a specific prefix from a string using parameter expansion?**
`"${str#prefix}"` (removes shortest matching prefix) or `"${str##prefix}"` (removes longest matching prefix, relevant for patterns with wildcards).

**95. Scenario: How would you extract just the filename from a full path, and separately just the file extension?**
```bash
filename="${path##*/}"       # remove everything up to the last /
extension="${filename##*.}"   # remove everything up to the last .
```

**96. What's the difference between `${str#pattern}` and `${str%pattern}`?**
`#` removes a matching pattern from the BEGINNING (front) of the string; `%` removes a matching pattern from the END (back) of the string.

**97. Scenario: How would you split a sentence into an array of individual words?**
```bash
read -ra words <<< "$sentence"
```

**98. How would you check if one string contains another as a substring (not just prefix/suffix)?**
```bash
if [[ "$str" == *substring* ]]; then
```

---

## MODULE 9: Text Processing: grep, sed, awk

**99. What does `grep -i` do?**
Performs a case-insensitive search.

**100. What does `grep -v` do?**
Inverts the match, showing lines that do NOT match the given pattern.

**101. Scenario: How would you count how many lines in a log file contain the word "ERROR"?**
`grep -c "ERROR" logfile.txt`.

**102. What's the difference between basic and extended regular expressions in `grep`, and how do you enable extended?**
Basic regex (default) requires escaping characters like `+`, `?`, `|` to use their special meaning; Extended regex (`-E` flag, or using `egrep`) treats these as special characters natively without escaping.

**103. Scenario: How would you search recursively through an entire directory tree for files containing a specific string?**
`grep -r "search_string" /path/to/directory`.

**104. What does `sed 's/old/new/'` do differently from `sed 's/old/new/g'`?**
Without `g`, only the FIRST occurrence per line is replaced; with `g` (global), ALL occurrences on each line are replaced.

**105. Scenario: How would you edit a file in place using sed, while keeping a backup of the original?**
`sed -i.bak 's/old/new/g' file.txt` — creates `file.txt.bak` as a backup before applying changes in place.

**106. What does `sed -n '5,10p' file.txt` do?**
Prints only lines 5 through 10 of the file (the `-n` suppresses sed's default behavior of printing every line, and `p` explicitly prints the specified range).

**107. Scenario: How would you delete all lines matching a specific pattern from a file using sed?**
`sed '/pattern/d' file.txt` (or `sed -i '/pattern/d' file.txt` to edit in place).

**108. What does `awk '{print $2}'` do?**
Prints the second whitespace-delimited field/column of each input line.

**109. Scenario: How would you sum up the values in the third column of a space-delimited file using awk?**
`awk '{sum += $3} END {print sum}' file.txt`.

**110. What does `-F` do in an awk command?**
Sets a custom field delimiter/separator, needed for non-whitespace-delimited data like CSV files (`awk -F',' '{print $2}' file.csv`).

**111. Scenario: How would you print only the lines where the third column's value exceeds 100, using awk?**
`awk '$3 > 100' file.txt` (or explicitly `awk '$3 > 100 {print}' file.txt`).

**112. What do `NR` and `NF` represent in awk?**
`NR` is the current record (line) number being processed; `NF` is the number of fields in the current record.

**113. Scenario: You need to find and kill all processes matching "nginx" by name, using a pipeline of common Unix tools. Write the command.**
`ps aux | grep nginx | grep -v grep | awk '{print $2}' | xargs kill -9`.

**114. How would you find the most frequently occurring lines in a file, sorted by frequency?**
`sort file.txt | uniq -c | sort -rn`.

---

## MODULE 10: Process Management & Job Control

**115. How do you run a command in the background?**
Append `&` to the command.

**116. What does the `jobs` command show?**
A list of background/stopped jobs currently associated with the current shell session.

**117. Scenario: You started a long-running script in the foreground but need to move it to the background without stopping it. What key sequence and command achieve this?**
Press `Ctrl+Z` to suspend the process (stopping it), then run `bg` to resume it running in the background.

**118. What does `nohup` do, and why would you use it?**
Prevents a process from receiving the SIGHUP signal (which is normally sent when the terminal/session closes), allowing the command to keep running even after you log out or close the terminal.

**119. Scenario: How would you make a background job survive even if the shell itself exits, beyond what `nohup` alone provides?**
`disown -h %1` — detaches the specified job from the shell's job table so it's not affected by the shell's own exit/closure.

**120. What's the difference between `kill -15` and `kill -9`?**
`kill -15` (SIGTERM) requests a graceful shutdown, allowing the process to clean up/handle the signal if it chooses to; `kill -9` (SIGKILL) forcibly terminates the process immediately, bypassing any cleanup and cannot be caught or ignored by the process.

**121. Scenario: Why is it generally considered bad practice to always use `kill -9` as a first resort, rather than `kill -15`?**
`SIGKILL` gives the process no opportunity to clean up open file handles, flush buffered writes, or gracefully close database transactions/connections — potentially leaving data in an inconsistent state; `SIGTERM` should generally be tried first, escalating to `SIGKILL` only if the process doesn't respond within a reasonable time.

**122. What does the `trap` command do?**
Registers a function/command to be executed when the script receives a specified signal (e.g., SIGINT, SIGTERM) or reaches a specific event (like EXIT).

**123. Scenario: How would you ensure a temporary file created by a script is always cleaned up, even if the script is interrupted (e.g., via Ctrl+C) or exits due to an error?**
```bash
tmpfile=$(mktemp)
trap 'rm -f "$tmpfile"' EXIT
```
`trap ... EXIT` guarantees the cleanup runs regardless of how/why the script terminates.

**124. How would you check whether a specific process (by name) is currently running, from within a script?**
`if pgrep -x "processname" > /dev/null; then ... fi`.

**125. What does `timeout 30s ./script.sh` do, and what does exit code `124` specifically indicate afterward?**
Runs the command but forcibly kills it if it hasn't completed within 30 seconds; exit code 124 specifically indicates the command was terminated due to the timeout being reached (as opposed to the command's own natural exit code).

---

## MODULE 11: Error Handling & Debugging

**126. What does `set -e` do?**
Causes the script to exit immediately if any command returns a non-zero exit status (rather than continuing execution despite the failure).

**127. What does `set -u` do?**
Causes the script to error out immediately if it tries to use an undefined/unset variable, rather than silently treating it as an empty string.

**128. Scenario: A pipeline `false | true` reports overall success (`$?` is 0) even though the first command failed. Why, and what setting fixes this?**
By default, a pipeline's exit status is only that of its LAST command; `set -o pipefail` changes this so the pipeline reports failure if ANY command within it fails, not just the final one.

**129. What is considered "Bash strict mode," and what three options does it typically combine?**
`set -euo pipefail` — combining `-e` (exit on error), `-u` (error on undefined variables), and `-o pipefail` (pipeline fails if any command in it fails) as a common defensive scripting preamble.

**130. What does `bash -x script.sh` do?**
Runs the script with execution tracing enabled, printing each command (with variable substitutions already applied) as it's executed — invaluable for debugging.

**131. Scenario: How would you enable tracing for only a specific suspicious section of a script, rather than the entire script's execution?**
Wrap just that section with `set -x` before it and `set +x` immediately after, rather than passing `-x` for the whole script invocation.

**132. What is `PS4`, and how can customizing it improve debugging output?**
`PS4` is the prompt string shown before each traced command line when using `set -x`/`bash -x`; customizing it (e.g., `PS4='+ Line ${LINENO}: '`) adds useful context like line numbers directly into the trace output, making it far easier to correlate trace lines with actual script locations.

**133. Scenario: How would you check the individual exit status of EACH command in a multi-stage pipeline, not just the overall pipeline result?**
Check the `PIPESTATUS` array immediately after the pipeline runs: `echo "${PIPESTATUS[@]}"` — gives the exit code of every command in the last executed pipeline, in order.

**134. What does `trap cleanup EXIT` guarantee, compared to just calling `cleanup` at the end of the script normally?**
It guarantees `cleanup` runs regardless of HOW the script terminates — whether it completes normally, exits early due to an error (especially relevant combined with `set -e`), or is interrupted by a signal — whereas code placed simply at "the end" of a script wouldn't run if the script exits early for any of those reasons.

**135. Scenario: How would you write custom, more descriptive error handling for a specific command's failure, rather than relying solely on `set -e`'s generic behavior?**
```bash
mkdir /some/path || { echo "ERROR: Failed to create directory" >&2; exit 1; }
```

**136. What's the difference between `bash -n script.sh` and `shellcheck script.sh`?**
`bash -n` only checks for actual syntax errors (would the script even parse); `shellcheck` performs deeper static analysis, catching a much broader range of logical mistakes, unsafe patterns, and portability issues that are syntactically valid but likely bugs.

**137. Scenario: How would you build a simple debug-logging mechanism into a script that only prints extra diagnostic output when explicitly enabled via an environment variable, without cluttering normal output otherwise?**
```bash
debug() { [ "${DEBUG:-0}" = "1" ] && echo "[DEBUG] $*" >&2; }
debug "Current value: $var"
```
Run with `DEBUG=1 ./script.sh` to enable, normal execution otherwise stays quiet.

---

## MODULE 12: File Operations & Permissions

**138. How do you check if a file exists in a shell script?**
`if [ -f "$file" ]; then`.

**139. What's the difference between `-d` and `-f` file test operators?**
`-d` tests if the path is a directory; `-f` tests if it's a regular file.

**140. Scenario: How would you create a nested directory structure in one command, without erroring if some of the parent directories already exist?**
`mkdir -p /some/nested/directory` — the `-p` flag creates all necessary parent directories and doesn't error if the target already exists.

**141. What does `find . -type f -mtime +7` do?**
Finds regular files modified more than 7 days ago within the current directory tree.

**142. Scenario: How would you safely find and delete all `.tmp` files older than 7 days across a directory tree, in one command?**
`find /path -name "*.tmp" -type f -mtime +7 -delete`.

**143. What does `mktemp` provide over manually constructing a temporary filename yourself (e.g., `"/tmp/myfile$$"`)?**
Guarantees a uniquely-named file created atomically and safely, avoiding race conditions or predictable-filename security vulnerabilities that manually constructed temp filenames can introduce (e.g., a malicious actor pre-creating a symlink at a predictable path).

**144. Scenario: Why is looping over `$(ls *.txt)` considered an anti-pattern compared to looping over `*.txt` directly?**
Parsing `ls` output breaks on filenames containing spaces or special characters (since the shell word-splits the command substitution's output); using the glob pattern directly (`for f in *.txt`) lets the shell expand it properly, correctly handling filenames with spaces without any parsing issues.

**145. What is the risk of running `rm -rf` with a variable that might be unset or empty, e.g., `rm -rf $dir/*`?**
If `$dir` is unset/empty, the command could expand to something unintended and dangerous (like effectively `rm -rf /*` in the worst case if further path construction goes wrong) — always quote and validate that such variables are non-empty before using them in destructive commands, and consider `set -u` as an additional safety net.

---

## MODULE 13: Advanced Real-World Scenarios

**146. Scenario: Design a shell script that safely deploys a new application version with automatic rollback if a health check fails after deployment.**
```bash
#!/bin/bash
set -euo pipefail

CURRENT_VERSION=$(readlink /opt/app/current)
NEW_VERSION="/opt/app/releases/$1"

ln -sfn "$NEW_VERSION" /opt/app/current
systemctl restart myapp

sleep 5
if ! curl -sf http://localhost:8080/health; then
    echo "Health check failed! Rolling back..." >&2
    ln -sfn "$CURRENT_VERSION" /opt/app/current
    systemctl restart myapp
    exit 1
fi
echo "Deployment successful"
```
This uses a symlink-swap pattern for near-instant rollback, a health check gate, and `set -euo pipefail` for safety.

**147. Scenario: How would you write a script to monitor disk usage across multiple mount points and send an alert only when usage crosses a threshold, avoiding duplicate alerts for the same ongoing issue?**
```bash
THRESHOLD=90
STATE_FILE="/tmp/disk_alert_state"

df -h --output=pcent,target | tail -n +2 | while read -r pcent mount; do
    usage=${pcent%\%}
    if [ "$usage" -ge "$THRESHOLD" ]; then
        if ! grep -q "^${mount}$" "$STATE_FILE" 2>/dev/null; then
            echo "ALERT: $mount at ${usage}%"
            echo "$mount" >> "$STATE_FILE"
        fi
    else
        sed -i "\|^${mount}\$|d" "$STATE_FILE" 2>/dev/null || true
    fi
done
```
Uses a simple state file to track already-alerted mounts, only re-alerting once the condition clears and re-triggers, avoiding alert spam for a persistent ongoing issue.

**148. Scenario: You need to process a 50GB log file to extract and count specific error patterns, but the naive approach (loading it all into a variable) crashes the script with an out-of-memory error. How do you redesign this?**
Never read an entire large file into a shell variable — instead, stream-process it directly with tools designed for this (`grep`/`awk`/`sed` operate on streams, not loading the whole file into memory at once): `grep -c "ERROR" hugefile.log` or `awk '/ERROR/{count++} END{print count}' hugefile.log` process the file line-by-line without ever holding the entire content in memory simultaneously.

**149. Scenario: How would you write a script that safely handles being run multiple times concurrently (e.g., accidentally triggered by overlapping cron executions), ensuring only one instance actually does the work?**
```bash
#!/bin/bash
exec 200>/tmp/myscript.lock
flock -n 200 || { echo "Already running, exiting"; exit 1; }

# ... actual script logic here ...
```
Using `flock` on a dedicated file descriptor provides a robust, race-condition-free locking mechanism (more reliable than a simple "check if a lock file exists" pattern, which has a race condition between the check and the file creation).

**150. Scenario: How would you design a script that reads a configuration file (key=value pairs) and safely loads it as shell variables, without allowing arbitrary code execution if the config file were ever maliciously modified?**
Avoid simply `source`-ing an arbitrary config file (which would execute any code within it, including malicious commands, not just variable assignments) — instead, parse it explicitly and validate each line matches a strict `KEY=value` pattern before assignment:
```bash
while IFS='=' read -r key value; do
    [[ "$key" =~ ^[A-Za-z_][A-Za-z0-9_]*$ ]] || continue
    declare "$key=$value"
done < config.txt
```

**151. Scenario: How would you build a shell script-based CI/CD deployment gate that only proceeds if ALL of several independent health checks (API, database connectivity, cache) pass, clearly reporting which one(s) failed if not?**
```bash
#!/bin/bash
failures=()

curl -sf http://localhost/api/health > /dev/null || failures+=("API")
pg_isready -h localhost > /dev/null || failures+=("Database")
redis-cli ping > /dev/null 2>&1 || failures+=("Cache")

if [ ${#failures[@]} -gt 0 ]; then
    echo "FAILED checks: ${failures[*]}" >&2
    exit 1
fi
echo "All health checks passed"
```
Collecting all failures into an array (rather than exiting on the first failure) provides more complete diagnostic information in a single run.

**152. Scenario: A script that worked reliably for months suddenly starts failing intermittently in production, but only under high load. How would you approach diagnosing a timing/race-condition-related bug in a shell script?**
Look for patterns involving concurrent file access without locking (multiple invocations writing to the same file simultaneously), reliance on `mktemp`/PID-based unique naming that might collide under specific circumstances, or race conditions between a "check" and subsequent "act" (e.g., checking a file doesn't exist, then creating it, with a gap where another process could interleave) — add `flock`-based locking around any genuinely shared-resource operations, and add detailed timestamped logging to narrow down exactly which operation is failing under load.

**153. Scenario: How would you refactor a very long, monolithic 2000-line shell script (that's become hard to maintain) into a more maintainable structure without switching to a different language entirely?**
Break it into logical function libraries sourced from a main orchestrating script (`source ./lib/deployment.sh`, `source ./lib/validation.sh`, etc.), extract configuration into a separate variables file, add ShellCheck to CI to enforce consistent quality going forward, and add proper `set -euo pipefail` plus structured logging/error handling if not already present — while also honestly evaluating whether certain complex logic (heavy data manipulation, complex business logic) would genuinely be better served by a proper scripting language like Python, since shell scripting has real limits for very complex logic.

**154. Scenario: How would you write a script that safely rotates and compresses log files older than a certain size or age, without risking data loss if interrupted partway through?**
```bash
#!/bin/bash
set -euo pipefail
LOGFILE="/var/log/myapp.log"
MAXSIZE=$((100*1024*1024))  # 100MB

size=$(stat -c%s "$LOGFILE")
if [ "$size" -gt "$MAXSIZE" ]; then
    mv "$LOGFILE" "${LOGFILE}.$(date +%Y%m%d%H%M%S)"
    touch "$LOGFILE"
    chmod 644 "$LOGFILE"
    # Signal the application to reopen its log file handle (varies by app)
    systemctl kill -s HUP myapp
    gzip "${LOGFILE}".*[0-9]
fi
```
Using `mv` (an atomic operation on the same filesystem) to rename the log before compressing avoids a window where log data could be lost or corrupted mid-write, and signaling the application to reopen its file handle ensures it doesn't keep writing to the now-renamed (and soon compressed) old file descriptor.

**155. Scenario: How would you write a portable script intended to run identically on both a full Bash environment and a minimal `sh`/`dash`-based container (e.g., Alpine Linux's default shell), avoiding Bash-specific syntax pitfalls?**
Use `#!/bin/sh` explicitly, avoid Bash-only constructs (`[[ ]]`, arrays, `${var,,}` case conversion, `(( ))` outside of POSIX-compatible contexts), use `[ ]` with careful quoting throughout, and validate portability using `checkbashisms` or by actually testing the script under `dash` directly rather than assuming compatibility — since subtle Bash-isms can silently behave differently (or simply error) under a strict POSIX shell.

**156. Scenario: How would you design a shell script-based secrets-handling pattern that avoids ever writing a sensitive value (like a database password) to disk, shell history, or process listing (visible via `ps`)?**
Avoid passing secrets as command-line arguments (visible in `ps aux` output to any other user on the system) — instead, pass them via environment variables (set just before the command, `SECRET=value command`) or better, via stdin/a file descriptor the command reads directly, and avoid `echo`-ing secrets to logs; also disable history recording for the specific commands involved (`set +o history` temporarily, or prefix commands with a space if `HISTCONTROL=ignorespace` is configured) to prevent secrets from persisting in shell history files.

**157. Scenario: A script that processes user-supplied filenames as arguments is discovered to be vulnerable to command injection if a filename contains something like `; rm -rf /`. How did this likely happen, and how do you fix it?**
Likely caused by using `eval` on constructed strings containing the untrusted filename, or unquoted variable expansion allowing the shell to reinterpret special characters within the filename as command syntax; fix by always quoting variables (`"$filename"`), never using `eval` on anything derived from untrusted input, and validating/sanitizing filenames against an expected pattern before use if they come from an external, untrusted source.

**158. Scenario: How would you design comprehensive automated testing for a critical shell script (e.g., a production deployment script) beyond just manually running it and observing the output?**
Use a shell-testing framework (like BATS — Bash Automated Testing System) to write actual unit/integration tests asserting on specific function behaviors and edge cases, mock/stub external commands the script depends on (so tests don't require the actual production infrastructure), integrate ShellCheck as a mandatory CI gate, and test explicitly for edge cases (empty inputs, missing files, network failures) rather than only testing the happy path.

**159. Scenario: How would you handle a scenario where a shell script needs to process a very large number of files (millions) efficiently, where a simple `for file in *` loop would exceed the shell's argument list limits or be prohibitively slow?**
Use `find ... -print0` piped to `xargs -0` (null-delimited to safely handle filenames with spaces/special characters) which properly batches arguments to avoid "argument list too long" errors and can parallelize processing with `xargs -P`:
```bash
find /path -type f -name "*.log" -print0 | xargs -0 -P 4 -I {} process_file {}
```

**160. Scenario: How would you design a shell script wrapper around a complex multi-step infrastructure operation (e.g., a database migration) so that partial failures leave the system in a clearly identifiable, recoverable state rather than an ambiguous "somewhere in the middle" condition?**
Implement explicit state tracking at each major step (writing a status marker to a state file after each successful stage completes), design each step to be idempotent/safely re-runnable (checking current state before acting rather than assuming a clean starting point), use `trap` to catch failures and clearly log exactly which step failed along with guidance on manual recovery steps if automatic rollback isn't feasible for that specific stage, and ensure the script's exit codes and logging are detailed enough that an on-call engineer picking this up at 3 AM can quickly understand exactly what state the system is actually in.

**161. Scenario: How would you build a shell script that generates a dynamic, environment-specific configuration file from a template, without hardcoding environment-specific values directly in the script itself?**
Use `envsubst` (part of gettext) combined with a template file containing `${VAR_NAME}` placeholders, and environment variables sourced from an environment-specific `.env` file:
```bash
set -a
source ".env.${ENVIRONMENT}"
set +a
envsubst < config.template.yml > config.yml
```

**162. Scenario: How would you handle a situation where a shell script needs to interact with a REST API, requiring proper JSON parsing rather than fragile regex/text-manipulation of JSON responses?**
Use `jq`, a purpose-built JSON processor, rather than attempting to parse JSON with `grep`/`sed`/`awk` (which is fragile and breaks easily on nested structures, escaped characters, or formatting variations):
```bash
response=$(curl -s http://api.example.com/status)
status=$(echo "$response" | jq -r '.status')
if [ "$status" = "healthy" ]; then
    echo "API is healthy"
fi
```

**163. Scenario: How would you approach converting a critical, complex shell script (with lots of business logic, string processing, and data structures) to a "better" language, while minimizing risk during the transition?**
Rather than a risky full rewrite-and-cutover, incrementally extract the most complex/error-prone logic into a properly-tested component in a more suitable language (Python is common), calling it from the existing shell script as a subprocess for just that piece, validating output parity between old and new logic in parallel for a period before fully removing the old shell-based logic — treating the migration as an incremental strangler-pattern replacement rather than a single risky cutover.

**164. Scenario: How would you write a shell script health-check wrapper suitable for use as a Docker `HEALTHCHECK` instruction, ensuring it behaves correctly with Docker's expected exit-code semantics?**
```bash
#!/bin/sh
set -e
if curl -sf http://localhost:8080/health > /dev/null; then
    exit 0
else
    exit 1
fi
```
Docker's `HEALTHCHECK` specifically interprets exit code 0 as healthy and any non-zero exit code as unhealthy — keep the script simple, fast (Docker healthchecks have a timeout), and ensure it uses `#!/bin/sh` for maximum compatibility with minimal container base images that may not include full Bash.

---
