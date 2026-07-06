# Shell Scripting Troubleshooting Guide

---

## Table of Contents

1. [General Troubleshooting Methodology](#module-1-general-troubleshooting-methodology)
2. [Syntax Errors](#module-2-syntax-errors)
3. [Variable & Quoting Issues](#module-3-variable--quoting-issues)
4. [Conditional & Test Expression Issues](#module-4-conditional--test-expression-issues)
5. [Loop Issues](#module-5-loop-issues)
6. [Command Substitution & Pipeline Issues](#module-6-command-substitution--pipeline-issues)
7. [Permission & Execution Issues](#module-7-permission--execution-issues)
8. [Environment & PATH Issues](#module-8-environment--path-issues)
9. [Text Processing (grep/sed/awk) Issues](#module-9-text-processing-grepsedawk-issues)
10. [Performance Issues](#module-10-performance-issues)
11. [Cron & Scheduled Script Issues](#module-11-cron--scheduled-script-issues)
12. [Debugging Techniques Reference](#module-12-debugging-techniques-reference)

---

## MODULE 1: General Troubleshooting Methodology

### 1.1 The Layered Troubleshooting Approach

```
Script fails / behaves unexpectedly
        │
1. Check syntax first (bash -n script.sh)
        │
2. Run with tracing enabled (bash -x script.sh)
        │
3. Isolate the failing section (comment out / run in pieces)
        │
4. Check variable values at the point of failure (echo/printf debugging)
        │
5. Check environment differences if it works in one context but not another
        │
6. Run ShellCheck for static analysis
        │
7. Fix, re-test, confirm
```

---

### 1.2 Essential Diagnostic Commands

```bash
bash -n script.sh              # syntax check only, no execution
bash -x script.sh               # execution trace
shellcheck script.sh             # static analysis / linting
echo $?                            # exit code of last command
set -o                               # show currently active shell options
type command_name                    # check if something is an alias/function/builtin/binary
which command_name                    # locate a binary in PATH
```

---

## MODULE 2: Syntax Errors

### 2.1 "unexpected end of file" or "unexpected token"

```bash
bash -n script.sh
```
Common causes: unmatched `if`/`fi`, `do`/`done`, `case`/`esac`, or quotes/parentheses — count opening and closing pairs carefully, especially in scripts with deep nesting; an editor with bracket-matching/syntax highlighting catches most of these before running the script at all.

---

### 2.2 "command not found" for a Line That Looks Like a Valid Assignment

```bash
# WRONG
name = "value"

# CORRECT — no spaces around =
name="value"
```
Bash interprets `name = "value"` as attempting to run a command named `name` with arguments `=` and `"value"` — spaces around the `=` are never allowed in variable assignment.

---

### 2.3 Script Works When Pasted Into Terminal but Fails When Run as a File

```bash
file script.sh
cat -A script.sh | head -5
```
Common cause: Windows-style line endings (`\r\n` / CRLF) introduced by editing on Windows or via a Git checkout without proper `.gitattributes` line-ending normalization — `cat -A` reveals `^M` characters at line ends if this is the issue.

```bash
sed -i 's/\r$//' script.sh
# or
dos2unix script.sh
```

---

### 2.4 "bad interpreter: No such file or directory" When Running the Script

```bash
head -1 script.sh
which bash
```
Common causes: the shebang path doesn't match where the interpreter actually exists on this system (e.g., `#!/bin/bash` but bash is actually at `/usr/local/bin/bash` on this particular system), or (again) CRLF line endings corrupting the shebang line itself.

```bash
#!/usr/bin/env bash    # more portable — searches PATH rather than hardcoding a path
```

---

### 2.5 Heredoc Not Terminating Properly

```bash
cat << EOF
some content
EOF
```
Common cause: trailing whitespace after the closing `EOF` delimiter, or the delimiter word appearing (even accidentally, e.g., as part of a variable's expanded value) within the heredoc body itself — the closing delimiter must appear alone on its own line with no leading/trailing characters (unless intentionally using `<<-EOF` to allow leading tabs).

---

## MODULE 3: Variable & Quoting Issues

### 3.1 Variable Not Expanding as Expected Inside Single Quotes

```bash
echo 'Value is $var'    # WRONG — prints literally "Value is $var", no expansion
echo "Value is $var"    # CORRECT — double quotes allow variable expansion
```
Single quotes suppress ALL expansion (variables, command substitution); double quotes allow variable/command substitution while still preventing word-splitting and globbing.

---

### 3.2 "unbound variable" Error with `set -u`

```bash
set -u
echo "$undefined_var"    # errors out under set -u
echo "${undefined_var:-}"  # provide an empty default to avoid the error safely
```
`set -u` is a valuable safety net (catches typos in variable names), but requires deliberately providing defaults (`${var:-default}`) for variables that are legitimately optional/conditionally set.

---

### 3.3 Word-Splitting Breaking a Script on Filenames with Spaces

```bash
# WRONG — breaks on filenames containing spaces
for file in $(ls *.txt); do
    process "$file"
done

# CORRECT
for file in *.txt; do
    process "$file"
done
```
Always quote variable expansions (`"$file"` not `$file`) and prefer globbing over parsing command output for file lists — unquoted variable expansion undergoes word-splitting on whitespace, silently breaking on any filename containing spaces.

---

### 3.4 Piped `while` Loop Not Preserving Variable Changes

```bash
# WRONG — subshell created by the pipe means count doesn't persist
count=0
cat file.txt | while read line; do count=$((count+1)); done
echo $count   # prints 0

# CORRECT
count=0
while read line; do count=$((count+1)); done < file.txt
echo $count   # correct count
```
A pipe (`|`) creates a subshell for the command on its right side in Bash — any variables modified inside that subshell are lost once it exits; redirecting input (`< file.txt`) instead of piping avoids the subshell entirely.

---

### 3.5 Array Elements Splitting Incorrectly When Passed to a Function

```bash
# WRONG — loses array structure if elements contain spaces
my_func $my_array

# CORRECT
my_func "${my_array[@]}"
```
`"${array[@]}"` (quoted, with `@`) preserves each element as a distinct word even if individual elements contain spaces; unquoted or using `*` collapses everything into word-split tokens or a single string respectively.

---

### 3.6 Variable Holds a Value with Trailing Newline/Whitespace Unexpectedly

```bash
value=$(command)
echo "[$value]"    # check for unexpected trailing whitespace/newlines visually
```
Command substitution `$(...)` strips trailing newlines by default, but embedded/internal newlines within multi-line output are preserved — if downstream logic assumes a single-line value, explicitly handle or strip this with `tr -d '\n'` or similar if the source command's output format is unpredictable.

---

## MODULE 4: Conditional & Test Expression Issues

### 4.1 "[: too many arguments" or "[: unary operator expected"

```bash
# WRONG — if $var is empty or contains spaces, this breaks
if [ $var = "value" ]; then

# CORRECT — always quote
if [ "$var" = "value" ]; then
```
An unquoted variable that's empty or contains spaces/multiple words causes `[ ]` to receive an unexpected number of arguments — always quote variables inside `[ ]` (this specific issue is one reason `[[ ]]` is generally preferred, since it's more forgiving of unquoted variables).

---

### 4.2 String Comparison Using `==` Failing in POSIX `sh`

```bash
[ "$a" == "$b" ]    # works in bash's [ ], but NOT strictly POSIX
[ "$a" = "$b" ]      # POSIX-correct, single = for string equality
```
Bash tolerates `==` inside `[ ]` as a non-standard extension, but a script run under a strict POSIX `sh` (like `dash`) may fail — use `=` for POSIX portability, or `[[ ]]` (Bash-only) where `==` is fully supported natively.

---

### 4.3 Numeric Comparison Accidentally Using String Comparison Operators

```bash
# WRONG — string comparison of numbers doesn't work as expected for numeric ordering
if [ "$a" > "$b" ]; then    # this is actually a REDIRECTION in [ ], not comparison!

# CORRECT
if [ "$a" -gt "$b" ]; then
if (( a > b )); then
```
Inside single `[ ]`, `>` and `<` are interpreted as shell redirection operators, not comparison — this silently creates/overwrites a file named after `$b`'s value rather than producing a comparison error, making it a particularly dangerous, easy-to-miss bug.

---

### 4.4 `if` Condition Always Evaluates True/False Regardless of Actual Logic

```bash
if [ "$var" ]; then     # true if $var is a non-empty string — often unintentional
if [ -n "$var" ]; then    # explicit, clearer equivalent
if [ "$var" = "true" ]; then  # if you actually meant to compare against the literal string "true"
```
Check exactly what condition is being tested — `[ "$var" ]` alone tests for non-empty string truthiness, which is easy to confuse with an intended boolean-style comparison against a specific expected value.

---

### 4.5 Regex Match with `=~` Not Working as Expected

```bash
pattern="^[0-9]+$"
if [[ "$str" =~ $pattern ]]; then    # store pattern in a variable, don't quote it in the test
```
Quoting the pattern directly inside `[[ ... =~ "pattern" ]]` can cause it to be treated as a literal string match rather than a regex in some Bash versions — storing the pattern in an unquoted variable reference is the more reliable, portable approach.

---

## MODULE 5: Loop Issues

### 5.1 `for` Loop Iterating Over Characters Instead of Words

```bash
# WRONG — iterates over the whole string as one "word" if quoted improperly, or splits unexpectedly
for word in "$sentence"; do echo "$word"; done   # treats the whole sentence as ONE item

# CORRECT — split explicitly if you want word-by-word iteration
for word in $sentence; do echo "$word"; done      # unquoted, relies on word-splitting (only safe if content is trusted/simple)
```
Be deliberate about whether you want word-splitting to occur — quoting fully preserves the string as a single unit; leaving it unquoted invokes word-splitting, which is exactly what you want in some cases but a bug in others.

---

### 5.2 Infinite Loop Due to a Never-Updated Loop Variable

```bash
count=0
while [ $count -lt 5 ]; do
    echo $count
    # BUG: forgot to increment count — infinite loop!
done
```
Always double-check that the loop's exit condition variable is actually being modified somewhere inside the loop body — a surprisingly common oversight, especially after refactoring loop bodies.

---

### 5.3 `break`/`continue` Not Working as Expected in Nested Loops

```bash
for i in 1 2 3; do
    for j in a b c; do
        if [ "$j" = "b" ]; then
            break 2    # breaks out of BOTH loops, not just the inner one
        fi
    done
done
```
`break`/`continue` accept an optional numeric argument specifying how many nested loop levels to break out of/continue — without it, they only affect the innermost loop, which is often not what's intended when trying to exit an entire nested structure early.

---

### 5.4 Loop Reading a File Skips Every Other Line or Behaves Oddly

```bash
# Potential issue: a command inside the loop that ALSO reads from stdin can consume lines meant for the read loop
while read -r line; do
    ssh remote_host "some_command" < /dev/null    # redirect stdin explicitly to avoid consuming the loop's input
done < file.txt
```
If a command inside a `while read` loop (like `ssh` without input redirected away) also reads from stdin, it can inadvertently consume lines intended for the outer loop's `read`, causing lines to appear skipped — explicitly redirect such commands' stdin from `/dev/null` to prevent this interference.

---

## MODULE 6: Command Substitution & Pipeline Issues

### 6.1 `$(command)` vs Backticks — Nesting Issues

```bash
# Backticks require awkward escaping when nested
result=`echo \`date\``

# $() nests cleanly
result=$(echo $(date))
```
Prefer `$(...)` over legacy backtick syntax — it nests naturally without escaping and is generally considered more readable/maintainable.

---

### 6.2 Pipeline Reports Success Even Though an Earlier Command Failed

```bash
false | true
echo $?    # prints 0 (success) by default — only the LAST command's exit status counts

set -o pipefail
false | true
echo $?    # now correctly reports failure
```
Enable `set -o pipefail` in any script relying on pipeline exit status for error detection — without it, a failure anywhere except the very last command in a pipe chain is silently invisible to `$?`.

---

### 6.3 Command Substitution Stripping Needed Trailing Newlines

```bash
content=$(printf "line1\nline2\n")
echo "$content"    # trailing newline from printf is stripped by $()
```
This is expected, standard Bash behavior (`$()` always strips trailing newlines) — if you need to preserve an exact trailing newline, append it back explicitly (`content="${content}"$'\n'`) rather than assuming command substitution preserves it.

---

### 6.4 Large Pipeline Performance Degradation

```bash
# Potentially slow: multiple separate process invocations chained together
cat file.txt | grep "pattern" | awk '{print $1}' | sort | uniq -c

# Often faster: fewer processes, awk itself can filter and process
awk '/pattern/ {print $1}' file.txt | sort | uniq -c
```
Every pipe segment spawns a new process — for very large files or performance-critical scripts, consolidating logic into fewer tools (e.g., doing filtering directly within `awk` rather than piping from `grep` first) can meaningfully reduce overhead, though for most everyday scripts this is a micro-optimization not worth obsessing over.

---

## MODULE 7: Permission & Execution Issues

### 7.1 "Permission denied" When Running `./script.sh`

```bash
chmod +x script.sh
ls -l script.sh    # verify execute bit is actually set afterward
```
The script file itself needs the execute permission bit set — this is separate from being readable, and a very common first-run gap for newly created scripts.

---

### 7.2 Script Runs Fine as Root but Fails for a Regular User (or Vice Versa)

```bash
sudo -u someuser ./script.sh    # test running as a specific different user directly
```
Check for hardcoded paths only accessible to certain users, operations requiring elevated privileges (writing to system directories) without an explicit `sudo` call inside the script, or environment variable differences between the root/user context (e.g., a different `PATH`).

---

### 7.3 "Text file busy" Error When Trying to Execute/Modify a Script

Occurs when trying to overwrite/modify a script file while it's currently being executed by a running process — wait for the running instance to complete, or (if genuinely needed) rename rather than directly overwrite the file to avoid this OS-level file-locking behavior.

---

### 7.4 Script Works Locally but Fails When Run via SSH/Automation (e.g., CI, cron)

```bash
env    # compare the environment when run interactively vs via the automated context
```
Non-interactive/non-login shell contexts (SSH commands, cron jobs, many CI runners) often have a significantly reduced `PATH` and don't source the same profile files (`.bashrc`, `.bash_profile`) as an interactive login shell — explicitly set required environment variables/PATH within the script itself rather than assuming they'll be inherited from an interactive session's configuration.

---

## MODULE 8: Environment & PATH Issues

### 8.1 "command not found" for a Command That Works Interactively

```bash
echo $PATH        # compare PATH between contexts
which mycommand     # confirm where the command actually lives
```
Non-login/non-interactive shells (common in cron, CI, and some automation contexts) typically don't source `.bashrc`/`.bash_profile`, resulting in a minimal default `PATH` that doesn't include directories (like `/usr/local/bin` or a custom tool's install location) that your interactive shell's profile adds.

```bash
# Fix by using the absolute path, or explicitly setting PATH in the script
export PATH="/usr/local/bin:$PATH"
```

---

### 8.2 Alias Defined in `.bashrc` Doesn't Work Inside a Script

Aliases are generally not expanded in non-interactive shell scripts by default — use a function instead of an alias for any reusable logic you need available both interactively and within scripts, since functions work consistently in both contexts.

---

### 8.3 Environment Variable Set in the Parent Shell Isn't Visible Inside the Script

```bash
export MY_VAR="value"    # must be exported to be inherited by child processes/scripts
./script.sh
```
A variable must be explicitly `export`ed in the parent shell to be inherited by any script/subprocess it launches — a plain (non-exported) variable assignment is only visible within that exact shell instance, not passed down to children.

---

### 8.4 Script Behaves Differently Depending on Which Shell Actually Executes It

```bash
readlink -f /bin/sh    # on many systems, /bin/sh is symlinked to dash, NOT bash
```
Running a script with Bash-specific syntax via `sh script.sh` (rather than `bash script.sh` or `./script.sh` with a proper `#!/bin/bash` shebang) can invoke a different shell entirely (often `dash` on Debian/Ubuntu systems) that doesn't support Bash-specific features, causing confusing syntax errors or silently different behavior.

---

## MODULE 9: Text Processing (grep/sed/awk) Issues

### 9.1 `grep` Pattern Not Matching Expected Lines

```bash
grep -E "pattern" file.txt    # confirm whether you need Extended regex (-E) for the pattern used
cat -A file.txt | head         # check for hidden characters (CRLF, non-printable) that might prevent matching
```
Common causes: needing `-E` for extended regex features (`+`, `?`, `|`) that basic `grep` doesn't support by default, or hidden characters (like Windows line endings) in the file preventing an otherwise-correct pattern from matching as expected.

---

### 9.2 `sed` In-Place Edit Fails on macOS but Works on Linux (or Vice Versa)

```bash
# GNU sed (Linux)
sed -i 's/old/new/' file.txt

# BSD sed (macOS) requires an explicit (even if empty) backup extension argument
sed -i '' 's/old/new/' file.txt
```
GNU sed and BSD sed (the default on macOS) have subtly different `-i` flag syntax — scripts intended to be portable across both should account for this difference, often by detecting the OS or using a more portable tool (like `perl -i -pe`) instead.

---

### 9.3 `awk` Field Separator Not Behaving as Expected

```bash
awk -F',' '{print $2}' file.csv    # explicitly set the field separator for non-whitespace-delimited data
awk -F'\t' '{print $2}' file.tsv     # tab-delimited
```
By default, `awk` splits on any whitespace (including collapsing multiple consecutive spaces/tabs) — for CSV or other explicitly-delimited data, always set `-F` explicitly rather than relying on the default whitespace behavior, which can miscount fields if the delimiter differs.

---

### 9.4 `sed`/`awk` Script Breaks on Special Characters in the Replacement String

```bash
# If the replacement contains a literal `/`, it conflicts with sed's default delimiter
sed 's/path/new\/path/' file.txt      # escape the conflicting character
sed 's|path|new/path|' file.txt        # or use an alternative delimiter entirely
```
When the text you're searching for or replacing with contains the same character used as `sed`'s delimiter (commonly `/`), either escape every occurrence or (more readable) choose a different delimiter character that doesn't appear in your pattern/replacement.

---

## MODULE 10: Performance Issues

### 10.1 Script Processing a Large File Is Extremely Slow

```bash
# Slow: spawning a new process (grep) for every single line in a loop
while read -r line; do
    echo "$line" | grep "pattern"
done < largefile.txt

# Fast: let grep process the whole file directly, no per-line subprocess spawning
grep "pattern" largefile.txt
```
Avoid spawning external processes inside a tight loop over many lines — this is one of the single biggest, most common shell scripting performance anti-patterns; whenever possible, let a single invocation of `grep`/`awk`/`sed` process the entire file/stream directly instead of looping and re-invoking it per line.

---

### 10.2 Excessive Use of `cat` Piped Into Another Command ("Useless Use of cat")

```bash
# Unnecessary extra process
cat file.txt | grep "pattern"

# Simpler, faster (grep reads files directly)
grep "pattern" file.txt
```
While the performance difference is often negligible for small files, this is flagged by ShellCheck and considered a stylistic/minor efficiency anti-pattern worth avoiding as a matter of habit for larger-scale scripts.

---

### 10.3 Script with Many Small `echo`/File-Append Operations Is Slow

```bash
# Slow: opens and writes to the file on every single iteration
for i in {1..10000}; do
    echo "$i" >> output.txt
done

# Faster: batch output, write once
for i in {1..10000}; do
    echo "$i"
done > output.txt
```
Redirecting the entire loop's output once (rather than appending inside every iteration) avoids repeatedly opening/closing the file descriptor for every single write, meaningfully improving performance for high-iteration-count loops.

---

### 10.4 Script Using External Tools for Simple Arithmetic Unnecessarily

```bash
# Slower: spawns bc as an external process for simple integer math
result=$(echo "5 + 3" | bc)

# Faster: Bash's native arithmetic, no external process needed
result=$((5 + 3))
```
Use Bash's built-in arithmetic (`$(( ))`) for integer math whenever possible — reserve external tools like `bc`/`awk` specifically for floating-point operations that Bash's native arithmetic genuinely can't handle.

---

## MODULE 11: Cron & Scheduled Script Issues

### 11.1 Cron Job Doesn't Run at All / Silently Fails

```bash
grep CRON /var/log/syslog     # confirm cron actually attempted to trigger the job
crontab -l                     # verify the entry exists with correct syntax
```
Check the crontab syntax carefully (5 time fields, then the command), and check system logs to confirm cron even attempted execution — if it's not even attempting, the crontab entry itself likely has a syntax error or wasn't saved correctly.

---

### 11.2 Script Works Manually but Fails Silently Under Cron

```bash
* * * * * /path/to/script.sh >> /var/log/script.log 2>&1
```
Always redirect both stdout and stderr to a log file for cron jobs — cron's default behavior for command output (mailing it, if mail is even configured) makes debugging silent failures nearly impossible without explicit logging; this single change resolves the vast majority of "works manually, fails under cron" mysteries by revealing the actual error.

---

### 11.3 "command not found" Specifically Under Cron

Cron runs jobs with an extremely minimal environment/PATH (often just `/usr/bin:/bin`) — always use absolute paths to commands/scripts within cron-executed scripts, or explicitly set `PATH` at the top of the script itself, rather than assuming the same `PATH` as your interactive login shell.

```bash
#!/bin/bash
export PATH="/usr/local/bin:/usr/bin:/bin"
```

---

### 11.4 Cron Job Runs at the Wrong Time (Timezone Confusion)

```bash
timedatectl    # check the system's configured timezone
crontab -l
```
Cron typically uses the system's configured timezone, not necessarily the timezone the person writing the crontab entry assumed — verify the actual system timezone, especially on cloud instances that may default to UTC regardless of the team's local timezone expectations.

---

### 11.5 Multiple Overlapping Cron Executions of a Long-Running Script

```bash
#!/bin/bash
LOCKFILE="/tmp/myscript.lock"
if [ -e "$LOCKFILE" ]; then
    echo "Script already running, exiting"
    exit 1
fi
trap 'rm -f "$LOCKFILE"' EXIT
touch "$LOCKFILE"

# ... actual script logic ...
```
If a script sometimes takes longer to run than the cron interval between scheduled executions, implement a simple lock-file mechanism (or use `flock` for a more robust approach) to prevent overlapping concurrent executions from interfering with each other.

```bash
flock -n /tmp/myscript.lock ./actual_script.sh
```

---

## MODULE 12: Debugging Techniques Reference

### 12.1 Core Debugging Toolkit

```bash
bash -n script.sh                    # syntax check only
bash -x script.sh                     # full execution trace
bash -v script.sh                      # print each line as read (before substitution)
set -x / set +x                         # toggle tracing within a specific section
PS4='+ ${BASH_SOURCE}:${LINENO}: '        # enrich trace output with file/line info
shellcheck script.sh                        # static analysis
```

---

### 12.2 Selective Tracing (Debugging Only Part of a Script)

```bash
#!/bin/bash
echo "Normal execution"

set -x    # start tracing here
suspicious_function
set +x    # stop tracing here

echo "Back to normal"
```

---

### 12.3 Print-Statement Debugging Pattern

```bash
debug() {
    if [ "${DEBUG:-0}" = "1" ]; then
        echo "[DEBUG] $*" >&2
    fi
}

debug "Variable value: $my_var"
```
```bash
DEBUG=1 ./script.sh    # enable debug output only when explicitly requested
```

---

### 12.4 Checking Variable State at a Specific Point

```bash
set | grep "^my_var="     # inspect a specific variable's current value among all shell variables
declare -p my_array         # inspect an array's full structure/contents in detail
```

---

## Quick Reference: Most Useful Shell Troubleshooting Commands

```bash
# Syntax & Static Analysis
bash -n script.sh
shellcheck script.sh

# Execution Tracing
bash -x script.sh
set -x / set +x

# Exit Codes
echo $?
echo "${PIPESTATUS[@]}"

# Environment
echo $PATH
env
type command_name
which command_name

# File/Line-ending Issues
cat -A file.sh | head
dos2unix file.sh

# Cron Debugging
grep CRON /var/log/syslog
* * * * * /path/script.sh >> /var/log/script.log 2>&1

# Locking (prevent overlapping runs)
flock -n /tmp/script.lock ./script.sh
```

---
