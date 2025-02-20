## Docker Compose 部署

```bash
docker compose exec -it vault vault operator init
```



- <https://ambar-thecloudgarage.medium.com/hashicorp-vault-with-docker-compose-0ea2ce1ca5ab>

## 其他

HashiCorp 推荐使用 Consul 作为 Vault 的数据存储 Backend

首先要创建 secret 需要线开启 kv secret 引擎，并将用户名和密码放在指定的路径中

```bash
vault secrets enable -path=internal kv-v2
```

然后在 `internal/exampleapp/config` 路径下面添加一个用户名和密码的秘钥：

```bash
vault kv put internal/database/config username="db-readonly-username" password="db-secret-password"
```

创建完成后可以通过如下命令校验上面创建的 secret：

```bash
vault kv get internal/database/config
```



Vault 提供了一个 Kubernetes 认证的方法可以让客户端通过使用 Kubernetes ServiceAccount 进行身份认证。

- <https://developer.hashicorp.com/vault/docs/auth/kubernetes>

开启 Kubernetes 认证方式：

```bash
vault auth enable kubernetes
```

Vault 会接受来自于 Kubernetes 集群中的任何客户端的服务 Token。在身份验证的时候，Vault 通过配置的 Kubernetes 地址来验证 ServiceAccount 的 Token 信息。通过 ServiceAccount 的 Token、Kubernetes 地址和 CA 证书信息配置 Kubernetes 认证方式：

```bash
vault write auth/kubernetes/config \
         token_reviewer_jwt="$(cat /var/run/secrets/kubernetes.io/serviceaccount/token)" \
         kubernetes_host="https://$KUBERNETES_PORT_443_TCP_ADDR:443" \
         kubernetes_ca_cert=@/var/run/secrets/kubernetes.io/serviceaccount/ca.crt
```

其中 `token_reviewer_jwt` 和 `kubernetes_ca_cert` 都是 Kubernetes 默认注入到 Pod 中的，而环境变量 `KUBERNETES_PORT_443_TCP_ADDR` 也是内置的表示 Kubernetes APIServer 的内网地址。为了让客户端读取上一步定义在 `internal/database/config` 路径下面的 secret 数据，还需要为该路径授予 `read` 的权限。

这里创建一个名为 `internal-app` 的策略名称，该策略会启用对路径 `internal/database/config` 中的 secret 的读取权限：

```bash
vault policy write internal-app - <<EOH
path "internal/data/database/config" {
  capabilities = ["read"]
}
EOH
```

然后创建一个名为 `internal-app` 的 Kubernetes 认证角色：

## 参考资料

- <https://www.qikqiak.com/post/deploy-vault-on-k8s/>

