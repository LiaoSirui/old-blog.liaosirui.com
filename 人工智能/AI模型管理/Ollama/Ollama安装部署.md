## Linux 下安装

安装方式：<https://ollama.com/download/linux>

```bash
curl -fsSL https://ollama.com/install.sh | sh
```

安装后使用 systemd 管理进程，模块文件：`/etc/systemd/system/ollama.service`

更换服务端监听的地址

```bash
Environment="OLLAMA_HOST=0.0.0.0"
```

更换存储模型的位置（一定要 `chown -R /app/models/ollama`）


```bash
Environment="OLLAMA_MODELS=/app/models/ollama"
```

设置代理地址（不设置代理拉取模型也很快）

```bash
Environment="HTTP_PROXY=http://192.168.248.11:18899"
Environment="HTTPS_PROXY=http://192.168.248.11:18899"
Environment="ALL_PROXY=socks5://192.168.248.11:18899"
Environment="NO_PROXY=127.0.0.1,localhost,10.0.0.0/8,172.16.0.0/12,192.168.0.0/16"
```



## K8s 部署

```yaml
customTolerations: &customTolerations
  - key: node.kubernetes.io/not-ready
    operator: Exists
    effect: NoExecute
    tolerationSeconds: 60
  - key: node.kubernetes.io/unreachable
    operator: Exists
    effect: NoExecute
    tolerationSeconds: 60
  - key: node-role.kubernetes.io/control-plane
    operator: Exists

customNodeAffinity: &customNodeAffinity
  requiredDuringSchedulingIgnoredDuringExecution:
    nodeSelectorTerms:
      - matchExpressions:
          - key: node-role.kubernetes.io/worker
            operator: In
            values:
              - ""

customNodeSelector: &customNodeSelector
  kubernetes.io/arch: amd64
  kubernetes.io/os: linux

replicaCount: 1

image:
  repository: harbor.alpha-quant.tech/3rd/docker.io/ollama/ollama
  tag: "0.5.11"

imagePullSecrets:
  - name: platform-oci-image-pull-secrets

ollama:
  gpu:
    enabled: true
    type: "nvidia"
    nvidiaResource: "nvidia.com/gpu"
    number: 4
  models:
    run:
      - deepseek-r1:70b
      - qwen2.5-coder:32b

ingress:
  enabled: true
  className: "nginx"
  hosts:
    - host: ollama.alpha-quant.tech
      paths:
        - path: /
          pathType: Prefix
  tls:
    - secretName: https-self-hosted-certs
      hosts:
        - ollama.alpha-quant.tech

resources:
  requests:
    memory: 300Gi
    cpu: "48"
  limits:
    memory: 300Gi
    cpu: "48"

nodeSelector: *customNodeSelector
tolerations: *customTolerations
affinity:
  nodeAffinity: *customNodeAffinity

extraEnv:
  - name: NVIDIA_VISIBLE_DEVICES
    value: GPU-08453bc4-c2c3-4b97-42a3-b1047533264d,GPU-324badda-57ba-468e-2e93-ab1f0ae06be1,GPU-c24351e3-a9ac-2437-7ad6-1fb903a621d9,GPU-d91f98df-5f46-4103-ff1f-55b9422ead34
  - name: OLLAMA_SCHED_SPREAD
    value: "1"

persistentVolume:
  enabled: true
  storageClass: "hostpath-local-data"

```

