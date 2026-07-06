# Shell Scripting (Bash) Complete Study Guide
---

## Table of Contents

1. [Shell Scripting Fundamentals](#module-1-shell-scripting-fundamentals)
2. [Variables & Data Types](#module-2-variables--data-types)
3. [Input/Output & Redirection](#module-3-inputoutput--redirection)
4. [Conditionals & Test Expressions](#module-4-conditionals--test-expressions)
5. [Loops](#module-5-loops)
6. [Functions](#module-6-functions)
7. [Arrays](#module-7-arrays)
8. [String Manipulation](#module-8-string-manipulation)
9. [Text Processing: grep, sed, awk](#module-9-text-processing-grep-sed-awk)
10. [Process Management & Job Control](#module-10-process-management--job-control)
11. [Error Handling & Debugging](#module-11-error-handling--debugging)
12. [File Operations & Permissions](#module-12-file-operations--permissions)
13. [Regular Expressions in Shell](#module-13-regular-expressions-in-shell)
14. [Advanced Scripting & Best Practices](#module-14-advanced-scripting--best-practices)

---

## MODULE 1: Shell Scripting Fundamentals

### 1.1 What is a Shell Script?

A shell script is a text file containing a sequence of commands executed by a shell interpreter (bash, sh, zsh) — used to automate repetitive tasks, orchestrate other programs, and build system administration tooling.

```
User writes script.sh
        │
Shell interpreter (#!/bin/bash) reads and executes line by line
        │
Commands run sequentially, with control flow (if/loops/functions) as directed
```

---

### 1.2 The Shebang Line

```bash
#!/bin/bash
```

The shebang (`#!`) on the first line tells the OS which interpreter to use when the script is executed directly (`./script.sh`) — without it, the script relies on however it's invoked (`bash script.sh` explicitly specifies the interpreter regardless of shebang).

```bash
#!/usr/bin/env bash    # more portable — finds bash wherever it is in PATH
```

---

### 1.3 Making a Script Executable

```bash
chmod +x script.sh
./script.sh

# Alternative without execute permission:
bash script.sh
sh script.sh
```

---

### 1.4 Comments

```bash
# This is a single-line comment

: '
This is a multi-line comment
using a no-op colon command and a heredoc-like string
'
```

---

### 1.5 Exit Codes

```bash
exit 0     # success
exit 1     # generic failure
echo $?    # check the exit code of the last executed command
```

By convention, `0` means success and any non-zero value indicates failure — scripts should exit with meaningful, distinct codes for different failure scenarios when building tooling other scripts/CI pipelines will check.

---

## MODULE 2: Variables & Data Types

### 2.1 Declaring and Using Variables

```bash
name="DevOps"
echo "Hello, $name"
echo "Hello, ${name}"     # braces avoid ambiguity, e.g., "${name}_suffix"
```

**No spaces around `=`** — `name = "value"` is a syntax error in Bash (it's interpreted as running a command called `name` with arguments `=` and `"value"`).

---

### 2.2 Variable Scope: Local vs Global vs Environment

```bash
GLOBAL_VAR="visible everywhere in this script"

my_function() {
    local LOCAL_VAR="only visible inside this function"
    echo "$LOCAL_VAR"
}

export ENV_VAR="visible to child processes too"
```

Without `export`, a variable is only available within the current shell/script — `export` makes it part of the environment passed to any child processes spawned from that shell.

---

### 2.3 Special/Built-in Variables

| Variable | Meaning |
|---|---|
| `$0` | Script name |
| `$1`, `$2`, ... | Positional arguments |
| `$#` | Number of arguments passed |
| `$@` | All arguments as separate words |
| `$*` | All arguments as a single string |
| `$?` | Exit code of the last command |
| `$$` | Process ID of the current script |
| `$!` | PID of the last background process |

```bash
#!/bin/bash
echo "Script: $0"
echo "First arg: $1"
echo "Total args: $#"
```

---

### 2.4 Read-only and Unsetting Variables

```bash
readonly MAX_RETRIES=5
unset my_variable
```

---

### 2.5 Arithmetic with Variables

```bash
count=5
count=$((count + 1))
echo $count            # 6

let count++
((count += 10))

result=$(( 10 / 3 ))   # integer division only — result is 3
```

For floating-point math, Bash's native arithmetic doesn't support it — use `bc` or `awk`:

```bash
result=$(echo "10 / 3" | bc -l)
result=$(awk "BEGIN {print 10/3}")
```

---

## MODULE 3: Input/Output & Redirection

### 3.1 Reading User Input

```bash
read -p "Enter your name: " name
echo "Hello, $name"

read -s -p "Enter password: " password    # -s suppresses echo (for secrets)
read -a my_array                          # reads input into an array
```

---

### 3.2 Standard Streams and Redirection

```
stdin  (0) — input to a command
stdout (1) — normal output
stderr (2) — error output
```

```bash
command > output.txt          # redirect stdout, overwrite
command >> output.txt         # redirect stdout, append
command 2> errors.txt         # redirect stderr only
command > all.txt 2>&1        # redirect both stdout AND stderr to the same file
command &> all.txt            # shorthand for the above (bash-specific)
command < input.txt           # redirect stdin from a file
command 2>/dev/null           # discard stderr entirely
```

**Order matters:** `command > file 2>&1` works correctly (stdout to file, then stderr follows stdout to the same place); `command 2>&1 > file` does NOT achieve the same result (stderr is duplicated to the terminal first, then only stdout goes to the file) — a very common interview trap.

---

### 3.3 Pipes

```bash
ps aux | grep nginx | awk '{print $2}'
```

Pipes connect one command's stdout directly to the next command's stdin, enabling composable command chains without intermediate temp files.

---

### 3.4 Here Documents and Here Strings

```bash
cat << EOF
This is a multi-line
block of text with $variable substitution
EOF

cat << 'EOF'
This text has $variable NOT substituted because EOF is quoted
EOF

grep "pattern" <<< "$my_string"    # here-string, feeds a string directly as stdin
```

---

### 3.5 `tee` — Writing to a File and stdout Simultaneously

```bash
command | tee output.txt              # show output AND save to file
command | tee -a output.txt            # append instead of overwrite
command | tee output.txt | grep error  # save output, but also continue processing it
```

---

## MODULE 4: Conditionals & Test Expressions

### 4.1 Basic `if` Statement

```bash
if [ "$count" -gt 10 ]; then
    echo "Count is greater than 10"
elif [ "$count" -eq 10 ]; then
    echo "Count is exactly 10"
else
    echo "Count is less than 10"
fi
```

---

### 4.2 Test Operators

**Numeric comparisons (inside `[ ]`):**
```bash
-eq   # equal
-ne   # not equal
-gt   # greater than
-lt   # less than
-ge   # greater than or equal
-le   # less than or equal
```

**String comparisons:**
```bash
[ "$a" = "$b" ]     # equal
[ "$a" != "$b" ]    # not equal
[ -z "$a" ]         # string is empty
[ -n "$a" ]         # string is non-empty
```

**File test operators:**
```bash
[ -f "$file" ]      # is a regular file
[ -d "$dir" ]       # is a directory
[ -e "$path" ]      # exists (any type)
[ -r "$file" ]      # is readable
[ -w "$file" ]      # is writable
[ -x "$file" ]      # is executable
[ -s "$file" ]      # exists and is non-empty
```

---

### 4.3 `[ ]` vs `[[ ]]` vs `(( ))`

```bash
[ "$a" = "$b" ]        # POSIX test — always quote variables, no pattern matching
[[ "$a" == $pattern ]]  # bash extended test — supports pattern matching, safer with unquoted vars
(( a > b ))             # arithmetic context — for numeric comparisons specifically
```

**Interview point:** `[[ ]]` is generally preferred in Bash scripts (not POSIX `sh`) because it doesn't perform word-splitting/globbing on unquoted variables the way `[ ]` does, making it less error-prone — but `[ ]` remains necessary for POSIX-compliant `/bin/sh` scripts.

---

### 4.4 Combining Conditions

```bash
if [ "$a" -gt 5 ] && [ "$b" -lt 10 ]; then
    echo "Both conditions true"
fi

if [[ "$a" -gt 5 && "$b" -lt 10 ]]; then
    echo "Same logic, single [[ ]] block"
fi
```

---

### 4.5 `case` Statements

```bash
case "$1" in
    start)
        echo "Starting service"
        ;;
    stop)
        echo "Stopping service"
        ;;
    restart|reload)
        echo "Restarting service"
        ;;
    *)
        echo "Usage: $0 {start|stop|restart}"
        exit 1
        ;;
esac
```

---

## MODULE 5: Loops

### 5.1 `for` Loop

```bash
for i in 1 2 3 4 5; do
    echo "Number: $i"
done

for i in {1..10}; do echo $i; done          # brace expansion range
for i in {1..10..2}; do echo $i; done        # step by 2

for file in /var/log/*.log; do
    echo "Processing: $file"
done

for ((i=0; i<10; i++)); do
    echo "C-style loop: $i"
done
```

---

### 5.2 `while` Loop

```bash
count=0
while [ $count -lt 5 ]; do
    echo "Count: $count"
    count=$((count + 1))
done

# Reading a file line by line
while IFS= read -r line; do
    echo "Line: $line"
done < file.txt
```

**Interview trap:** Always use `IFS= read -r` when reading lines — without `IFS=`, leading/trailing whitespace gets stripped; without `-r`, backslash escape sequences in the input get interpreted, potentially corrupting data with literal backslashes.

---

### 5.3 `until` Loop

```bash
count=0
until [ $count -ge 5 ]; do
    echo "Count: $count"
    count=$((count + 1))
done
```

`until` runs while the condition is **false** — the logical inverse of `while`.

---

### 5.4 Loop Control: `break` and `continue`

```bash
for i in {1..10}; do
    if [ $i -eq 5 ]; then
        break            # exit the loop entirely
    fi
    echo $i
done

for i in {1..10}; do
    if [ $((i % 2)) -eq 0 ]; then
        continue         # skip to the next iteration
    fi
    echo $i
done
```

---

### 5.5 Common Pitfall: Piping into a `while` Loop and Variable Scope

```bash
# WRONG — count doesn't persist after the loop because the pipe creates a subshell
count=0
cat file.txt | while read line; do
    count=$((count + 1))
done
echo $count    # prints 0!

# CORRECT — use process substitution or input redirection instead
count=0
while read line; do
    count=$((count + 1))
done < file.txt
echo $count    # correctly prints the actual count
```

This is one of the most commonly-tested Bash "gotchas" in interviews — a pipe creates a subshell for the command on its right, so variables modified inside a piped `while` loop don't persist in the parent shell.

---

## MODULE 6: Functions

### 6.1 Defining and Calling Functions

```bash
greet() {
    echo "Hello, $1!"
}

greet "World"    # calls the function, "World" becomes $1 inside it
```

```bash
function greet {    # alternative syntax (less portable, avoid in POSIX sh scripts)
    echo "Hello, $1!"
}
```

---

### 6.2 Return Values

```bash
add() {
    local result=$(( $1 + $2 ))
    echo $result          # "return" a value via stdout, captured with command substitution
}

sum=$(add 5 3)
echo "Sum: $sum"

is_even() {
    if [ $(( $1 % 2 )) -eq 0 ]; then
        return 0    # success/true
    else
        return 1    # failure/false
    fi
}

if is_even 4; then
    echo "Even"
fi
```

**Key distinction:** `return` only sets the function's exit status (0-255, checked via `$?` or directly in a conditional) — it cannot return arbitrary data types. To "return" an actual value (like a computed number or string), echo it and capture the output via command substitution.

---

### 6.3 Local Variables in Functions

```bash
counter() {
    local count=0    # scoped to this function only
    count=$((count + 1))
    echo $count
}
```

Always use `local` for variables inside functions unless you specifically intend to modify a global variable — omitting `local` causes the variable to leak into the global scope, a common source of subtle bugs in larger scripts.

---

### 6.4 Passing Arrays to Functions

```bash
print_array() {
    local arr=("$@")
    for item in "${arr[@]}"; do
        echo "$item"
    done
}

my_array=(one two three)
print_array "${my_array[@]}"
```

---

## MODULE 7: Arrays

### 7.1 Declaring and Accessing Arrays

```bash
fruits=("apple" "banana" "cherry")
echo "${fruits[0]}"          # apple
echo "${fruits[@]}"          # all elements
echo "${#fruits[@]}"         # array length (3)

fruits[3]="date"             # add a new element
unset fruits[1]              # remove an element (leaves a gap in indices!)
```

---

### 7.2 Iterating Over Arrays

```bash
for fruit in "${fruits[@]}"; do
    echo "$fruit"
done

for i in "${!fruits[@]}"; do    # iterate over indices
    echo "Index $i: ${fruits[$i]}"
done
```

---

### 7.3 Associative Arrays (Bash 4+)

```bash
declare -A capitals
capitals["India"]="New Delhi"
capitals["USA"]="Washington DC"

echo "${capitals[India]}"

for country in "${!capitals[@]}"; do
    echo "$country -> ${capitals[$country]}"
done
```

---

### 7.4 Array Slicing

```bash
arr=(a b c d e f g)
echo "${arr[@]:2:3}"     # elements from index 2, length 3 → c d e
echo "${arr[@]: -2}"      # last 2 elements → f g (note the space before the minus)
```

---

## MODULE 8: String Manipulation

### 8.1 String Length and Substring

```bash
str="Hello, World"
echo "${#str}"              # length: 12
echo "${str:7}"              # substring from index 7: World
echo "${str:7:3}"             # substring from index 7, length 3: Wor
```

---

### 8.2 String Replacement

```bash
str="Hello, World"
echo "${str/World/Bash}"      # replace first match: Hello, Bash
echo "${str//o/0}"             # replace ALL matches: Hell0, W0rld
echo "${str^^}"                 # uppercase entire string
echo "${str,,}"                  # lowercase entire string
```

---

### 8.3 Default Values and Parameter Expansion

```bash
echo "${name:-default_value}"     # use default_value if name is unset OR empty
echo "${name:=default_value}"     # same as above, but also ASSIGNS the default to name
echo "${name:?Error message}"     # print error and exit if name is unset/empty
echo "${name:+alt_value}"          # use alt_value ONLY if name IS set (inverse logic)
```

**Interview scenario:** `${name:-default}` vs `${name-default}` (without the colon) — the colon version treats an empty string the same as unset; without the colon, an empty (but set) variable is treated as "set" and the default is NOT used.

---

### 8.4 Splitting Strings

```bash
IFS=',' read -ra parts <<< "one,two,three"
for part in "${parts[@]}"; do
    echo "$part"
done
```

---

### 8.5 Trimming Whitespace

```bash
str="  hello  "
trimmed="${str#"${str%%[![:space:]]*}"}"    # trim leading whitespace
trimmed="${trimmed%"${trimmed##*[![:space:]]}"}"  # trim trailing whitespace

# Simpler alternative using xargs (has some caveats with special characters)
trimmed=$(echo "$str" | xargs)
```

---

## MODULE 9: Text Processing: grep, sed, awk

### 9.1 `grep` — Searching Text

```bash
grep "error" logfile.txt
grep -i "error" logfile.txt         # case-insensitive
grep -v "debug" logfile.txt          # invert match (show non-matching lines)
grep -r "TODO" ./src                  # recursive search
grep -c "error" logfile.txt           # count matching lines
grep -n "error" logfile.txt           # show line numbers
grep -E "error|warning" logfile.txt   # extended regex, OR pattern
grep -A 3 -B 2 "error" logfile.txt    # 3 lines After, 2 lines Before context
```

---

### 9.2 `sed` — Stream Editor

```bash
sed 's/old/new/' file.txt              # replace first occurrence per line
sed 's/old/new/g' file.txt              # replace ALL occurrences per line
sed -i 's/old/new/g' file.txt            # edit the file IN PLACE
sed -i.bak 's/old/new/g' file.txt         # in-place, keeping a .bak backup
sed -n '5,10p' file.txt                    # print only lines 5-10
sed '/pattern/d' file.txt                   # delete lines matching pattern
```

---

### 9.3 `awk` — Pattern Scanning and Processing

```bash
awk '{print $1}' file.txt                  # print first column (whitespace-delimited)
awk -F',' '{print $2}' file.csv             # specify a custom field delimiter
awk '{sum += $3} END {print sum}' file.txt  # sum a column
awk '$3 > 100 {print $1, $3}' file.txt      # conditional filtering
awk 'NR==5' file.txt                         # print only line 5
awk 'BEGIN {print "Start"} {print} END {print "End"}' file.txt
```

**Interview point:** `awk` operates on a record-by-record (typically line-by-line) basis, splitting each record into fields — it's essentially a mini programming language purpose-built for column-oriented text processing, far more powerful than `grep`/`sed` for anything beyond simple search/replace.

---

### 9.4 Combining grep, sed, and awk in a Pipeline

```bash
ps aux | grep nginx | grep -v grep | awk '{print $2}' | xargs kill -9
```

This finds nginx processes, excludes the `grep` command itself from matching, extracts the PID column, and kills each matching process — a very common real-world administrative one-liner pattern.

---

### 9.5 `cut`, `sort`, `uniq`, `wc`

```bash
cut -d',' -f2 file.csv          # extract the 2nd field, comma-delimited
sort file.txt                    # alphabetical sort
sort -n file.txt                  # numeric sort
sort -k2 -n file.txt                # sort by 2nd column, numerically
uniq file.txt                        # remove adjacent duplicate lines (sort first!)
sort file.txt | uniq -c | sort -rn    # count occurrences, sorted by frequency
wc -l file.txt                         # count lines
```

---

## MODULE 10: Process Management & Job Control

### 10.1 Running Processes in Background/Foreground

```bash
long_command &        # run in background, prompt returns immediately
jobs                   # list background jobs
fg %1                    # bring job 1 to foreground
bg %1                     # resume a stopped job in background
wait                       # wait for all background jobs to complete
wait $PID                   # wait for a specific PID
```

---

### 10.2 `nohup` and `disown`

```bash
nohup ./long_script.sh &     # survives terminal/session closure
disown -h %1                  # detach a job so it's not affected by shell exit
```

---

### 10.3 Signals

```bash
kill -15 $PID    # SIGTERM — graceful shutdown request
kill -9 $PID      # SIGKILL — force kill, cannot be caught/ignored
kill -1 $PID       # SIGHUP — often used to trigger config reload
trap 'echo "Caught SIGINT, cleaning up..."; exit 1' SIGINT
```

`trap` lets a script intercept signals and run custom cleanup logic (e.g., removing temp files) before exiting — critical for scripts that create temporary resources needing guaranteed cleanup even if interrupted.

---

### 10.4 Checking if a Process Is Running

```bash
if pgrep -x "nginx" > /dev/null; then
    echo "nginx is running"
fi

if ps -p $PID > /dev/null; then
    echo "Process $PID is still running"
fi
```

---

### 10.5 Timeout for Long-Running Commands

```bash
timeout 30s ./potentially_hanging_command.sh
echo $?    # 124 specifically indicates the command was killed due to timeout
```

---

## MODULE 11: Error Handling & Debugging

### 11.1 `set` Options for Safer Scripts

```bash
set -e            # exit immediately if any command fails
set -u             # error on use of undefined variables
set -o pipefail     # a pipeline fails if ANY command in it fails, not just the last one
set -x              # print each command before executing it (debugging)

set -euo pipefail   # combined — a very common "strict mode" preamble
```

**Interview point on `pipefail`:** Without it, `false | true` reports success (exit code 0) because only the *last* command's exit status matters by default — `set -o pipefail` fixes this so a failure anywhere in the pipeline causes the whole pipeline to report failure.

---

### 11.2 Trapping Errors for Cleanup

```bash
#!/bin/bash
set -e

cleanup() {
    echo "Cleaning up temp files..."
    rm -f /tmp/myscript.$$
}
trap cleanup EXIT

echo "data" > /tmp/myscript.$$
# ... rest of script ...
```

`trap cleanup EXIT` ensures the cleanup function runs no matter how the script exits — success, failure, or being interrupted — making it a reliable pattern for guaranteed resource cleanup.

---

### 11.3 Custom Error Handling with `||` and `&&`

```bash
mkdir /some/dir || { echo "Failed to create directory" >&2; exit 1; }

command1 && command2    # run command2 only if command1 succeeds
command1 || command2     # run command2 only if command1 fails
```

---

### 11.4 Debugging Techniques

```bash
bash -x script.sh              # trace every command as it executes
bash -n script.sh                # syntax check only, don't execute
set -x / set +x                   # enable/disable tracing within the script
PS4='+ ${LINENO}: '                # customize trace output to include line numbers
```

```bash
#!/bin/bash
set -x
PS4='+ Line ${LINENO}: '
```

---

### 11.5 Checking Exit Codes of Individual Commands in a Pipeline

```bash
command1 | command2 | command3
echo "${PIPESTATUS[@]}"    # exit codes of EACH command in the last pipeline, in order
```

`$?` alone only reflects the last command's exit code in a pipeline — `PIPESTATUS` (Bash-specific) is needed to check whether an earlier command in the chain actually failed.

---

## MODULE 12: File Operations & Permissions

### 12.1 Checking File Existence and Type

```bash
if [ -f "$file" ]; then echo "Regular file exists"; fi
if [ -d "$dir" ]; then echo "Directory exists"; fi
if [ -L "$link" ]; then echo "Is a symbolic link"; fi
```

---

### 12.2 Creating, Copying, Moving, Deleting

```bash
touch newfile.txt
mkdir -p /some/nested/directory      # -p creates parent directories as needed, no error if exists
cp -r source_dir dest_dir
mv oldname.txt newname.txt
rm -f file.txt                         # force delete, no error if missing
rm -rf directory/                       # recursive force delete — use with extreme caution
```

---

### 12.3 Finding Files

```bash
find /var/log -name "*.log"
find . -type f -mtime +7                 # files modified more than 7 days ago
find . -type f -size +100M                # files larger than 100MB
find . -name "*.tmp" -exec rm {} \;        # find and delete matching files
find . -name "*.log" -delete                # simpler alternative delete syntax
```

---

### 12.4 File Permissions in Scripts

```bash
chmod 755 script.sh
chmod +x script.sh
chown user:group file.txt
stat -c "%a" file.txt    # show permission mode numerically
```

---

### 12.5 Working with Temp Files Safely

```bash
tmpfile=$(mktemp)
trap 'rm -f "$tmpfile"' EXIT
echo "data" > "$tmpfile"
```

`mktemp` generates a uniquely-named, safely-created temp file (avoiding race conditions/predictable-filename vulnerabilities that manually constructing a temp filename could introduce) — always pair with a `trap` to guarantee cleanup.

---

## MODULE 13: Regular Expressions in Shell

### 13.1 Basic vs Extended Regular Expressions (BRE vs ERE)

```bash
grep "a\+b" file.txt      # BRE requires escaping + for "one or more"
grep -E "a+b" file.txt     # ERE (extended) treats + as a special character natively
```

Tools differ in default regex flavor: `grep` defaults to BRE (needs `-E` for ERE), `sed` defaults to BRE (needs `-E` or `-r` for ERE depending on the sed implementation), `awk` uses ERE-like syntax by default.

---

### 13.2 Bash's `[[ ]]` with `=~` for Regex Matching

```bash
if [[ "$string" =~ ^[0-9]+$ ]]; then
    echo "String is entirely numeric"
fi

if [[ "$email" =~ ^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$ ]]; then
    echo "Looks like a valid email format"
fi
```

`=~` inside `[[ ]]` is Bash's native regex matching operator — the pattern on the right should generally be unquoted (or stored in a variable) since quoting can cause it to be treated as a literal string rather than a regex pattern in some Bash versions.

---

### 13.3 Capturing Regex Groups with `BASH_REMATCH`

```bash
if [[ "server-01.example.com" =~ ^([a-z]+)-([0-9]+)\. ]]; then
    echo "Full match: ${BASH_REMATCH[0]}"
    echo "Group 1: ${BASH_REMATCH[1]}"    # server
    echo "Group 2: ${BASH_REMATCH[2]}"     # 01
fi
```

---

## MODULE 14: Advanced Scripting & Best Practices

### 14.1 Script Structure and Argument Parsing

```bash
#!/bin/bash
set -euo pipefail

usage() {
    echo "Usage: $0 -f <file> [-v]"
    exit 1
}

verbose=false
while getopts "f:v" opt; do
    case $opt in
        f) file="$OPTARG" ;;
        v) verbose=true ;;
        *) usage ;;
    esac
done

if [ -z "${file:-}" ]; then
    usage
fi
```

`getopts` is the standard Bash built-in for parsing short-form command-line flags (`-f value`, `-v`) — for long-form flags (`--file value`), you typically need to write manual parsing logic or use an external tool.

---

### 14.2 Idempotent Script Design

```bash
if [ ! -f /etc/myapp/config.yml ]; then
    cp default_config.yml /etc/myapp/config.yml
fi

mkdir -p /var/log/myapp    # -p makes mkdir idempotent — no error if it already exists
```

Well-designed shell scripts should be safely re-runnable without causing errors or unintended duplicate side effects — checking state before acting, rather than assuming a clean-slate environment every time.

---

### 14.3 Logging in Shell Scripts

```bash
LOG_FILE="/var/log/myscript.log"

log() {
    echo "$(date '+%Y-%m-%d %H:%M:%S') - $1" | tee -a "$LOG_FILE"
}

log "Starting deployment"
```

---

### 14.4 Portability: Bash-isms vs POSIX `sh`

```bash
# Bash-specific (won't work in dash/POSIX sh):
[[ "$a" == "$b" ]]
arr=(a b c)
${var,,}

# POSIX-portable equivalents:
[ "$a" = "$b" ]
```

Scripts intended to run under `#!/bin/sh` (which on many systems is `dash`, not `bash`) must avoid Bash-specific syntax — check with `checkbashisms` or explicitly test under `dash` if POSIX portability is a genuine requirement (e.g., for scripts embedded in Docker `RUN` instructions using a minimal shell).

---

### 14.5 Shellcheck — Static Analysis for Shell Scripts

```bash
shellcheck script.sh
```

ShellCheck catches a huge range of common shell scripting mistakes (unquoted variables, incorrect test operators, subshell scope issues) before they cause runtime bugs — treat it as a mandatory pre-commit/CI check for any non-trivial shell script, similar to a linter for any other language.

---

### 14.6 Security Best Practices

- Always quote variables (`"$var"` not `$var`) to prevent word-splitting and globbing issues, especially with filenames containing spaces.
- Never `eval` untrusted input — it's a direct code-injection vector.
- Use `mktemp` for temp files rather than hardcoded/predictable paths.
- Validate/sanitize any user-supplied input before using it in a command, especially if it's ever passed to `sh -c`, `eval`, or used to construct file paths.
- Avoid parsing `ls` output programmatically (`for f in $(ls)`) — use globbing (`for f in *`) instead, since `ls` output isn't designed for reliable machine parsing and breaks on filenames with spaces/special characters.

```bash
# Fragile and unsafe:
for f in $(ls *.txt); do ...

# Correct:
for f in *.txt; do ...
```

---

## Summary Cheat-Sheet: Most-Used Shell Scripting Constructs

```bash
#!/bin/bash
set -euo pipefail

# Variables
name="value"
readonly CONST=5

# Conditionals
if [[ "$a" -gt 5 ]]; then ...; fi

# Loops
for i in {1..5}; do ...; done
while IFS= read -r line; do ...; done < file.txt

# Functions
my_func() { local x=$1; echo $x; }

# Arrays
arr=(a b c)
for item in "${arr[@]}"; do ...; done

# Text processing
grep -E "pattern" file | awk '{print $1}' | sort | uniq -c

# Error handling
trap 'cleanup' EXIT
command || { echo "failed" >&2; exit 1; }

# Debugging
bash -x script.sh
shellcheck script.sh
```

---
