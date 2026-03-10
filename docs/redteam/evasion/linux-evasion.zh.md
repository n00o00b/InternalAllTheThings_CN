# Linux - 规避 (Evasion)

## 目录 (Summary)

- [文件名 (File Names)](#file-names)
- [命令历史 (Command History)](#command-history)
- [隐藏文本 (Hiding Text)](#hiding-text)
- [时间戳篡改 (Timestomping)](#timestomping)
- [对非 Root 用户隐藏 PID 列表 (Hiding PID Listings From Non-Root Users)](#hiding-pid-listings-from-non-root-users)

## 文件名 (File Names)

可以在文件名中插入 Unicode 零宽空格，使文件名在视觉上难以区分：

```bash
# 一个没有特殊字符的诱饵文件
touch 'index.php'

# 一个视觉上完全相同但包含特殊字符的文件
touch $'index\u200D.php'
```

## 命令历史 (Command History)

大多数 shell 都会保存命令历史记录，以便用户以后再次调用。可以使用 `history` 命令查看命令记录，或者手动检查 `$HISTFILE` 指向的文件内容（例如 `~/.bash_history`）。
可以通过多种方式阻止这种情况。

```bash
# 完全阻止写入历史文件
unset HISTFILE

# 不在内存中保存此会话的命令历史
export HISTSIZE=0
```

匹配 `HISTIGNORE` 中模式的单个命令将从命令历史中被排除，无论 `HISTFILE` 或 `HISTSIZE` 的设置如何。
默认情况下，`HISTIGNORE` 将忽略所有以空格开头的命令：

```bash
# 注意起始部分的空格字符：
 my-sneaky-command
```

如果命令意外地被添加到历史记录中，可以使用 `history -d` 删除单个条目：

```bash
# 删除最近一次记录的命令。
# 注意我们实际上必须一次删除两条历史记录，
# 否则 `history -d` 命令本身也会被记录。
history -d -2 && history -d -1
```

也可以清除整个命令历史，尽管这种方法不那么隐蔽，而且非常容易被注意到：

```bash
# 清除内存中的历史记录并将空历史写入磁盘。
history -c && history -w
```

若要采取更具破坏性的方法，你可以删除 `.bash_history` 文件的内容，或者将其链接到 `/dev/null` 以防止未来的历史记录日志记录。

```ps1
# 通过将其链接到 /dev/null 来永久禁用 bash 历史记录
ln /dev/null ~/.bash_history -sf

# 清除现有的 bash 历史记录
echo "" > .bash_history
```

## 隐藏文本 (Hiding Text)

在某些情况下，可以滥用 ANSI 转义序列来隐藏文本。
如果文件内容被打印到终端（例如 `cat`、`head`、`tail`），则文本会被隐藏。
如果使用编辑器（例如 `vim`、`nano`、`emacs`）查看文件，则转义序列将可见。

```bash
echo "sneaky-payload-command" > script.sh
echo "# $(clear)" >> script.sh
echo "# Do not remove. Generated from /etc/issue.conf by configure." >> script.sh

# 打印时，终端将被清除，只有最后一行可见：
cat script.sh
```

## 时间戳篡改 (Timestomping)

时间戳篡改 (Timestomping) 是指通过修改文件或目录的修改/访问时间戳，以掩盖其被修改过的事实。
实现这一目标最简单的方法是使用 `touch` 命令：

```bash
# 使用 YYYYMMDDhhmm 格式更改访问时间 (-a) 和修改时间 (-m)。
touch -a -m -t 202210312359 "example"

# 使用 Unix 时间戳 (Epoch) 更改时间。
touch -a -m -d @1667275140 "example"

# 将一个文件的时间戳复制到另一个文件。
touch -a -m -r "other_file" "example"

# 获取文件的修改时间戳，修改文件，然后恢复时间戳。
MODIFIED_TS=$(stat --format="%Y" "example")
echo "backdoor" >> "example"
touch -a -m -d @$MODIFIED_TS "example"
```

需要注意的是，`touch` 只能修改访问时间和修改时间。它不能用于更新文件的“状态更改时间 (Change)”或“创建时间 (Birth)”。如果文件系统支持，创建时间会跟踪文件的创建时间。状态更改时间则会在文件的元数据发生更改（包括访问时间和修改时间的更新）时进行更新。

如果攻击者拥有 root 权限，他们可以通过修改系统时钟、创建或修改文件，然后恢复系统时钟来绕过这一限制：

```bash
ORIG_TIME=$(date)
date -s "2022-10-31 23:59:59"
touch -a -m "example"
date -s "${ORIG_TIME}"
```

别忘了，创建一个文件也会更新父目录的修改时间戳！

## 对非 Root 用户隐藏 PID 列表 (Hiding PID Listings From Non-Root Users)

默认情况下，`/proc` 文件系统向所有用户公开进程信息。你可以通过修改 `/proc` 挂载选项将此访问权限限制为仅限 root 用户。

```ps1
sudo mount -o remount,rw,nosuid,nodev,noexec,relatime,hidepid=2 /proc
```

- `hidepid=2`：隐藏不属于该用户的所有进程。
- `hidepid=1`：仅隐藏进程详情（命令行、环境变量），但仍显示 PID。

## 参考资料 (References)

- [ATT&CK - Impair Defenses: Impair Command History Logging](https://attack.mitre.org/techniques/T1562/003/)
- [ATT&CK - Indicator Removal: Timestomp](https://attack.mitre.org/techniques/T1070/006/)
- [ATT&CK - Indicator Removal on Host: Clear Command History](https://attack.mitre.org/techniques/T1070/003/)
- [ATT&CK - Masquerading: Match Legitimate Name or Location](https://attack.mitre.org/techniques/T1036/005/)
- [Wikipedia - ANSI escape codes](https://en.wikipedia.org/wiki/ANSI_escape_code)
- [InverseCos - Detecting Linux Anti-Forensics: Timestomping](https://www.inversecos.com/2022/08/detecting-linux-anti-forensics.html)
