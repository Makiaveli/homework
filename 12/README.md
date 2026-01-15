
## Управление процессами 
### написать свою реализацию lsof

```bash
#!/usr/bin/env bash
# my_lsof.sh - простая реализация lsof через /proc
# Параметры: (необязательно) PATH_TO_MATCH -> показать только дескрипторы, где путь содержит строку
# Пример: ./my_lsof.sh /var/log
set -euo pipefail

match="${1:-}"

# Header
printf "%-7s %-10s %-6s %-6s %s\n" "PID" "USER" "FD" "TYPE" "NAME"

for pid_dir in /proc/[0-9]*; do
  pid=$(basename "$pid_dir")
  # skip if not accessible
  if [ ! -r "$pid_dir/status" ]; then
    continue
  fi
  uid=$(awk '/^Uid:/ {print $2; exit}' "$pid_dir/status" 2>/dev/null || echo "")
  user=$(getent passwd "$uid" | cut -d: -f1 || echo "$uid")
  # iterate fds
  fd_dir="$pid_dir/fd"
  if [ ! -d "$fd_dir" ]; then
    continue
  fi
  for fd in "$fd_dir"/*; do
    fdname=$(basename "$fd")
    # readlink target
    target=$(readlink -f "$fd" 2>/dev/null || true)
    if [ -z "$target" ]; then
      # Could be socket:[12345] or anon_inode or deleted
      target=$(readlink "$fd" 2>/dev/null || echo "")
    fi
    # get type: if target looks like socket:, pipe, anon_inode, or file: use stat
    type="unknown"
    if [[ "$target" =~ ^socket:\[ ]]; then
      type="SOCKET"
    elif [[ "$target" =~ ^pipe:\[ ]]; then
      type="PIPE"
    elif [[ "$target" =~ ^anon_inode: ]]; then
      type="ANON"
    elif [ -e "$fd" ]; then
      # stat the fd to see if it's a regular file, chr, dir...
      st=$(stat -L -c %F "$fd" 2>/dev/null || true)
      type=$(echo "$st" | tr '[:lower:]' '[:upper:]' | awk '{print $1}')
    fi

    # Filter by match if provided
    if [ -n "$match" ] && [[ "$target" != *"$match"* ]]; then
      continue
    fi

    printf "%-7s %-10s %-6s %-6s %s\n" "$pid" "$user" "$fdname" "$type" "$target"
  done
done | sort -k1,1n -k3,3

```
