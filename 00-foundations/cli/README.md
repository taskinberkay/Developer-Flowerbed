# Command line cheat sheet

I'm on Windows, so I keep PowerShell and Bash (Git Bash / WSL) side by side. The ideas are the same, the syntax isn't.

## The big difference

- **Bash** pipes **text**. Tools like `grep`, `awk` and `sed` slice strings.
- **PowerShell** pipes **objects**. You filter by property, no text parsing needed.

```bash
# Bash: biggest files, by parsing text
ls -l | sort -k5 -n -r | head -5
```
```powershell
# PowerShell: same idea, with properties
Get-ChildItem | Sort-Object Length -Descending | Select-Object -First 5
```

## Navigation and files

| I want to... | Bash | PowerShell |
|---|---|---|
| Where am I? | `pwd` | `pwd` (`Get-Location`) |
| List files | `ls -la` | `ls` (`Get-ChildItem -Force`) |
| Change directory | `cd dir`, `cd ..`, `cd ~`, `cd -` | `cd dir`, `cd ..`, `cd ~` |
| Make a folder | `mkdir -p a/b/c` | `mkdir a\b\c` (`New-Item -ItemType Directory`) |
| Create empty file | `touch file` | `New-Item file` |
| Copy | `cp -r src dst` | `cp -Recurse src dst` |
| Move / rename | `mv a b` | `mv a b` |
| Delete file | `rm file` | `rm file` |
| Delete folder | `rm -rf dir` ⚠️ | `rm -Recurse -Force dir` ⚠️ |
| Show file | `cat file` | `cat file` (`Get-Content`) |
| First / last lines | `head -n 10`, `tail -n 10` | `Get-Content f -TotalCount 10`, `-Tail 10` |
| Follow a log | `tail -f app.log` | `Get-Content app.log -Wait` |

## Searching

| I want to... | Bash | PowerShell |
|---|---|---|
| Find files by name | `find . -name "*.java"` | `Get-ChildItem -Recurse -Filter *.java` |
| Search inside files | `grep -rn "TODO" .` | `Select-String -Path *.java -Pattern "TODO" -Recurse` |
| Case-insensitive | `grep -i` | `Select-String` is case-insensitive by default |
| Count lines | `wc -l file` | `(Get-Content file).Count` |
| Find a command | `which java` | `Get-Command java` |

`ripgrep` (`rg`) is faster than both for code search, worth installing.

## Pipes and redirection

```bash
cmd > out.txt        # overwrite
cmd >> out.txt       # append
cmd 2> err.txt       # stderr only
cmd > all.txt 2>&1   # both
cmd1 | cmd2          # pipe
cmd1 && cmd2         # run cmd2 only if cmd1 succeeded
cmd1 || cmd2         # run cmd2 only if cmd1 failed
```

PowerShell 5.1 has no `&&` / `||`. Use `cmd1; if ($?) { cmd2 }`.

## Environment and processes

| I want to... | Bash | PowerShell |
|---|---|---|
| Read a variable | `echo $PATH` | `$env:PATH` |
| Set for this session | `export JAVA_HOME=/path` | `$env:JAVA_HOME = "C:\path"` |
| List processes | `ps aux` | `Get-Process` |
| Kill by name / PID | `kill 1234`, `pkill name` | `Stop-Process -Id 1234`, `-Name name` |
| What uses port 8080? | `lsof -i :8080` | `Get-NetTCPConnection -LocalPort 8080` |
| Run a previous command | `history`, `!!`, Ctrl+R | `Get-History`, Ctrl+R, ↑ |

Setting a variable permanently on Windows: `[Environment]::SetEnvironmentVariable("JAVA_HOME","C:\path","User")`, then restart the terminal.

## Web and data

```bash
curl -i https://example.com                  # headers + body
curl -X POST -H "Content-Type: application/json" -d '{"a":1}' https://api.example.com
```
```powershell
Invoke-RestMethod https://api.example.com                       # parses JSON for me
Invoke-RestMethod -Method Post -Uri $url -Body '{"a":1}' -ContentType 'application/json'
```

`curl` in Windows PowerShell 5.1 is an alias for `Invoke-WebRequest`, so use `curl.exe` for the real one.

## Permissions (Bash only)

```bash
chmod +x script.sh        # make executable
chmod 644 file            # rw-r--r--
chmod 755 dir             # rwxr-xr-x
```
Numbers: read = 4, write = 2, execute = 1. Add them per owner / group / others.

## Shortcuts that save time

- `Tab` completes, `Ctrl+R` searches history, `Ctrl+C` stops a command
- `Ctrl+L` clears the screen, `Ctrl+A` / `Ctrl+E` jump to line start / end (Bash)
- `!!` repeats the last command (Bash), handy as `sudo !!`
- Wrap paths with spaces in quotes

## Tiny scripts

```bash
#!/usr/bin/env bash
set -euo pipefail          # stop on errors, undefined vars, pipe failures
for f in *.java; do
  echo "Compiling $f"
  javac "$f"
done
```
```powershell
$ErrorActionPreference = "Stop"
Get-ChildItem *.java | ForEach-Object {
    Write-Host "Compiling $($_.Name)"
    javac $_.FullName
}
```

## Gotchas

- `rm -rf` has no recycle bin. Check with `ls` first, and never run it with an unchecked variable (`rm -rf $DIR/`).
- Windows paths use `\`, Bash uses `/`. Git Bash accepts `C:/Projects/...`.
- `echo $VAR` and `echo "$VAR"` differ when the value has spaces. Quote it.
- PowerShell prints objects, so `>` redirection may write the formatted table, not the raw data. Use `Out-File -Encoding utf8` when encoding matters.
- Case sensitivity: Linux paths are case-sensitive, Windows ones aren't. Code that works on my machine can break in CI.

## Further reading

- The Missing Semester (MIT): https://missing.csail.mit.edu
- Bash manual: `man bash`, or https://explainshell.com to decode a command
- Microsoft Learn: PowerShell 101
