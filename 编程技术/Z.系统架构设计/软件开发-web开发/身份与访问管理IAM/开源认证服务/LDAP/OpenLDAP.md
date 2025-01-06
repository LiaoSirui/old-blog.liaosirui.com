## OpenLDAP 简介

OpenLDAP 是一个开源的用户系统实现，主要支持 LDAP 协议，可以给其他系统提供用户认证

- <https://www.openldap.org/>
- <https://git.openldap.org/openldap/openldap>

## 搭建 OpenLDAP

拉取镜像

```bash
docker-pull() {
  skopeo copy docker://${1} docker-daemon:${1}
}
docker-pull "docker.io/bitnami/openldap:2.6.9-debian-12-r1"
```

给 OpenLDAP 配置 TLS。首先用 OpenSSL 生成 CA 和证书。

创建数据目录

```bash
mkdir ./openldap-data
chown -R 1001:root ./openldap-data

chown -R 1001:root certs
```

启动一个 OpenLDAP 服务端

```yaml
services:
  openldap:
    image: docker.io/bitnami/openldap:${IMAGE_VERSION}
    ports:
      - '389:389' # LDAP
      - '636:636' # LDAPS
    environment:
      - LDAP_PORT_NUMBER=389
      - LDAP_LDAPS_PORT_NUMBER=636
      - LDAP_ADMIN_USERNAME=${LDAP_ADMIN_USERNAME}
      - LDAP_ADMIN_PASSWORD=${LDAP_ADMIN_PASSWORD}
      - LDAP_ROOT=DC=alpha-quant,DC=tech
      - LDAP_ADMIN_DN=CN=admin,DC=alpha-quant,DC=tech
      - LDAP_ALLOW_ANON_BINDING=no
      - LDAP_ENABLE_TLS=yes
      - LDAP_TLS_CERT_FILE=/opt/bitnami/openldap/certs/alpha-quant.tech.crt
      - LDAP_TLS_KEY_FILE=/opt/bitnami/openldap/certs/alpha-quant.tech.key
      - LDAP_TLS_CA_FILE=/opt/bitnami/openldap/certs/alpha-quant.tech.CA.crt
    volumes:
      - ./certs:/opt/bitnami/openldap/certs
      - ./openldap:/bitnami/openldap
    networks:
      - openldap
networks:
  openldap:
    driver: bridge
    ipam:
      driver: default
      config:
      - subnet: 172.30.1.0/24
        gateway: 172.30.1.254

```

admin 密码建议单独保存，例如写在 `.env` 中：

```bash
IMAGE_VERSION=2.6.9-debian-12-r1

LDAP_ADMIN_USERNAME=admin
LDAP_ADMIN_PASSWORD=12345678REDACTED
```

指定最小 TLS 证书版本

```bash
docker compose exec -T openldap ldapmodify -Y EXTERNAL -H "ldapi:///" << __EOF__
dn: cn=config
changetype: modify
add: olcTLSProtocolMin
olcTLSProtocolMin: 3.1

__EOF__
```

访问方式：

- LDAP

```bash
ldapwhoami \
    -H ldap://ldap.alpha-quant.tech:389/ \
    -x \
    -D "cn=admin,dc=alpha-quant,dc=tech" \
    -W
```

- LDAP Over SSL

```bash
ldapwhoami \
    -H ldaps://ldap.alpha-quant.tech:636/ \
    -x \
    -D "cn=admin,dc=alpha-quant,dc=tech" \
    -W \
    -o TLS_CACERT="$PWD/certs/alpha-quant.tech.CA.crt"
```

- LDAP Over TLS

```bash
ldapwhoami \
    -H ldap://ldap.alpha-quant.tech:389/ \
    -x \
    -Z \
    -D "cn=admin,dc=alpha-quant,dc=tech" \
    -W \
    -o TLS_CACERT="$PWD/certs/alpha-quant.tech.CA.crt"
```

可以通过 ldapsearch 列出所有对象，默认情况下不需要登录（Bind DN），可以只读访问：

```bash
ldapsearch \
    -H ldap://localhost:389/ \
    -x \
    -D "cn=admin,dc=alpha-quant,dc=tech" \
    -b "dc=alpha-quant,dc=tech" \
    -W
```

管理员修改用户的密码，使用 ldappasswd 修改：

默认情况下，用户没有权限修改自己的密码。可以进入 Docker 容器，修改数据库的权限：

```bash
docker compose exec -T openldap ldapmodify -Y EXTERNAL -H "ldapi:///" << __EOF__

# Paste the following lines
# Allow user to change its own password
dn: olcDatabase={2}mdb,cn=config
changetype: modify
replace: olcAccess
olcAccess: {0}to attrs=userPassword
  by anonymous auth
  by self write
  by * none
olcAccess: {1}to *
  by * read

__EOF__

```

通过管理员修改密码

```bash
LDAPTLS_CACERT=$PWD/certs/openldap.crt ldappasswd \
    -H ldaps://ldap.alpha-quant.tech:636/ \
    -x \
    -S \
    -D "cn=admin,dc=alpha-quant,dc=tech" \
    -W \
    cn=user01,ou=users,dc=alpha-quant,dc=tech
```

用户修改密码

```bash
LDAPTLS_CACERT=$PWD/certs/openldap.crt ldappasswd \
    -H ldaps://ldap.alpha-quant.tech:636/ \
    -x \
    -S \
    -D "cn=user01,ou=users,dc=alpha-quant,dc=tech" \
    -W
```

并且 userPassword 也对非 admin 用户被隐藏

## LDAP 管理面板

```bash
services:
  ldap-account-manager:
    image: ghcr.io/ldapaccountmanager/lam:9.0
    restart: unless-stopped
    ports:
      - "8080:80"
    environment:
      - LAM_PASSWORD=${LAM_PASSWORD}
      - LAM_LANG=en_US
      - LDAP_SERVER=${LDAP_SERVER}
      - LDAP_DOMAIN=${LDAP_DOMAIN}
      - LDAP_BASE_DN=${LDAP_BASE_DN}
      - LDAP_USER=cn=admin,${LDAP_BASE_DN}
      - VIRTUAL_HOST=ldap-admin.alpha-quant.tech
      - VIRTUAL_PORT=80
    hostname: ldap-account-manager

```

## 自助修改密码系统

问题解决，添加命令如下：

``` 
ldapmodify -Y EXTERNAL -H ldapi:/// -f updatepass.ldif
```

"AD 自助修改密码" 是 AD 服务中的一个重要功能，它允许用户通过 Web 界面自主更改自己的密码，无需管理员介入，提高了效率并降低了支持成本

- <https://github.com/ltb-project/self-service-password>
- <https://github.com/alvinsiew/ldap-self-service>

## 参考资料

- <https://blog.csdn.net/achi010/article/details/130655238>

- <http://114.242.246.250:8036/basetool/ldap.html>



