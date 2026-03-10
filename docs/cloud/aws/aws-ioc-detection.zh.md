# AWS - IOC 和检测 (IOC & Detections)

## CloudTrail

### 禁用 (Disable) CloudTrail

```powershell
aws cloudtrail delete-trail --name cloudgoat_trail --profile administrator
```

禁用全局服务事件监控

```powershell
aws cloudtrail update-trail --name cloudgoat_trail --no-include-global-service-event 
```

在指定区域禁用 Cloud Trail

```powershell
aws cloudtrail update-trail --name cloudgoat_trail --no-include-global-service-event --no-is-multi-region --region=eu-west
```

## GuardDuty

### 操作系统 User Agent (OS User Agent)

:warning: 在 Kali Linux、Pentoo 和 Parrot Linux 上使用 awscli 时，会生成基于 user-agent 的日志。

Pacu 通过定义自定义 User-Agent 来绕过此问题：[pacu.py#L1473](https://web.archive.org/web/20201111195614/https://github.com/RhinoSecurityLabs/pacu/blob/master/pacu.py#L1303)

```python
boto3_session = boto3.session.Session()
ua = boto3_session._session.user_agent()
if 'kali' in ua.lower() or 'parrot' in ua.lower() or 'pentoo' in ua.lower():  # 如果本地操作系统是 Kali/Parrot/Pentoo Linux
    # GuardDuty 会触发与来自 Kali Linux 的 API 调用相关的发现，因此让我们避免这种情况...
    self.print('Detected environment as one of Kali/Parrot/Pentoo Linux. Modifying user agent to hide that from GuardDuty...')
```
