## Docker Compose 部署

简单部署

```yaml
services:
  vault:
    image: docker.io/hashicorp/vault:1.18.4
    ports:
      - "16443:443"
    environment:
      VAULT_ADDR: "https://vault.alpha-quant.tech"
      VAULT_SKIP_VERIFY: "true"
    cap_add:
      - IPC_LOCK
    entrypoint: |
      vault server -config /vault/config/config.hcl
    volumes:
      - ./vault/logs:/vault/logs/:rw
      - ./vault/config:/vault/config/:rw
      - ./vault/file:/vault/file/:rw
      - ./certs:/vault/certs/:rw
    networks:
      - vault
networks:
  vault:
    ipam:
      config:
        - subnet: 172.28.6.0/24
          ip_range: 172.28.6.0/24
          gateway: 172.28.6.254

```

`config.hcl` 如下

```
// Enable UI
ui = true

disable_mlock = true

// Storage
storage "file" {
  path    = "/vault/file"
}

// TCP Listener
listener "tcp" {
  address = "0.0.0.0:443"
  tls_disable = "false"
  tls_cert_file = "/vault/certs/alpha-quant.tech.crt"
  tls_key_file  = "/vault/certs/alpha-quant.tech.key"
}

```

然后执行初始化

```bash
docker compose exec -it vault vault operator init
```

保存输出的`Unseal Key`和`Root Token`备用

解封 Vault，输入上面保存的 Key

```bash
vault operator unseal
```

解封后，使用上面的 Root Token 登录

```bash
vault login <Initial_Root_Token>
```

## Cert 认证

生成证书，使用如下配置

```ini
[req]
default_bits = 4096
default_md = sha256
distinguished_name = req_distinguished_name
x509_extensions = v3_req
prompt = no

[req_distinguished_name]
C = FR
ST = VA
L = SomeCity
O = MyCompany
OU = MyDivision
CN = xxx.xxx.xxx.xxx

[v3_req]
keyUsage = digitalSignature, keyEncipherment
extendedKeyUsage = serverAuth,clientAuth
subjectAltName = @alt_names

[alt_names]
IP.1 = xxx.xxx.xxx.xxx
```

 生成自签名证书

```bash
openssl req -nodes -x509 -days 365 -keyout vault.key -out vault.crt -config cert.conf
```

登录成功后，开启 cert 认证模式

```bash
# 开启 cert 认证模式
vault auth enable cert

# 开启名为 alpha-quant 的 v1 版本的密钥引擎
vault secrets enable -path="alpha-quant" -version=1 kv

# 写入证书
vault write auth/cert/certs/alpha-quant policies=alpha-quant certificate=@/vault/cert/vault.crt ttl=1h
# 写入 policy
vault policy write alpha-quant /vault/config/alpha-quant-policy.hcl
```

`policy.hcl` 如下

```bash
path "alpha-quant/*" {
  capabilities = ["create", "read", "update", "delete", "list"]
}
```

验证一下是否成功开启

```bash
vault login -method=cert -client-cert=/vault/cert/vault.crt -client-key=/vault/cert/vault.key name=alpha-quant

# 写入数据
vault kv put alpha-quant/test bar=baz

# 读取数据
vault kv get -version=1 alpha-quant/test
```

## Consul 作为存储后台

```hcl
storage "consul"{
  address = "127.0.0.1:8500"
  path    = "vault/"
}

listener "tcp"{
  address     = "127.0.0.1:8200"
  tls_disable = 1
}

```



## 其他

- <https://developer.hashicorp.com/vault/docs/auth/kubernetes>

## 参考资料

- <https://www.qikqiak.com/post/deploy-vault-on-k8s/>

