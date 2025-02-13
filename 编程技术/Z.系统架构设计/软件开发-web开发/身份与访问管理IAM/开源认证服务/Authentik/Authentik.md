## Authentik

Authentik 是一个开源的身份验证和授权框架，用于保护和授权对应用程序的访问

Authentik 的主要特点包括：

- 身份验证：Authentik 支持多种身份验证机制，包括用户名 / 密码、JWT（JSON Web Tokens）、OAuth 2.0 等，可以满足不同场景下的需求。
- 授权：Authentik 提供基于策略的授权功能，可以定义各种访问规则和限制，以确保只有具有足够权限的用户才能访问应用程序。
- 多租户支持：Authentik 可以支持多个租户（Tenants），这意味着多个客户可以在同一个部署中使用该框架，并且每个客户可以独立管理其用户和权限。
- 可扩展性：Authentik 提供了丰富的插件和扩展机制，可以轻松地与其他身份验证和授权解决方案集成，例如 LDAP、Active Directory 等。
- 安全性和隐私保护：Authentik 在处理敏感信息时确保了足够的安全性和隐私保护，例如加密密码、防止跨站点请求伪造（CSRF）攻击等。

### 界面

- 管理界面：用于创建和管理用户和组、令牌和凭证、应用程序集成、事件以及定义标准和可自定义登录和身份验证流程的流程的可视化工具。易于阅读的可视化仪表板显示系统状态、最近的登录和身份验证事件以及应用程序使用情况。
- 用户界面：authentik 中的此控制台视图显示您在其中实施了 authentik 的所有应用程序和集成。单击要访问的应用程序以将其打开，或向下钻取以在管理界面中编辑其配置。
- 流：流是登录和身份验证过程的各个阶段发生的步骤。阶段表示登录过程中的单个验证或逻辑步骤。Authentik 允许自定义和精确定义这些流。

### 术语

![image-20250213145927033](./.assets/Authentik/image-20250213145927033.png)

- Application

应用程序将 Policy 与 Provider 链接在一起，从而允许控制访问。它还包含 UI 名称、图标等信息

- Source

源是可以将用户添加到 authentik 的位置。例如，用于从 Active Directory 导入用户的 LDAP 连接，或用于允许社交登录的 OAuth2 连接

- Provider

Provider 是其他应用程序对 authentik 进行身份验证的一种方式。常见的提供商是 OpenID Connect （OIDC） 和 SAML

LDAP、Proxy 见 Outpost

- Policy

策略是 yes/no。它将计算为 True 或 False，具体取决于 Policy Kind （策略类型） 和设置。

例如，如果用户是指定组的成员，则“组成员身份策略”的计算结果为 True，否则为 False。

这可用于有条件地应用 Stages、授予/拒绝对各种对象的访问权限，以及用于其他自定义逻辑

- Flows & Stages

流是有序的阶段序列。这些流可用于定义用户如何进行身份验证、注册等。

阶段表示单个验证或逻辑步骤。它们用于对用户进行身份验证、注册用户等。可以选择通过策略将这些阶段应用于流。

- Property Mappings

属性映射允许为外部应用程序提供信息，并修改来自源的信息在 authentik 中的存储方式

- Outpost

Outpost 是 authentik 的一个单独组件，无论 authentik 部署如何，都可以部署在任何地方。Outpost 提供的服务未直接实施到 authentik 核心中，例如反向代理

### 架构

![image-20250213150509372](./.assets/Authentik/image-20250213150509372.png)

authentik 由少量组件组成，其中大多数都是正常运行所需要的

- Server 容器由两个子组件组成，即实际服务器本身和嵌入式前哨。轻量级路由器将传入服务器容器的请求路由到核心服务器或嵌入式前哨。此路由器还处理对任何静态文件（例如 JavaScript 和 CSS 文件）的请求。

  - Core 子组件处理 authentik 的大部分逻辑，例如 API 请求、流执行、任何类型的 SSO 请求等。
  - Embedded outpost 与其他 Outpost 类似，此 Outpost 允许使用代理提供商，而无需部署单独的 Outpost。

  - `/media` 用于存储 icon 等，但不是必需的，如果未挂载，authentik 将允许设置图标的 URL 来代替文件上传

- Background Worker 此容器执行后台任务，例如发送电子邮件、事件通知系统以及可以在前端的 System Tasks 页面上看到的所有内容。
  - `/certs` 用于 authentik 导入外部证书，在大多数情况下，这些证书不应用于 SAML，但如果使用 authentik 而不使用反向代理，则可以将其用于 Let's Encrypt 集成
  - `/templates` 用于自定义电子邮件模板，并且与其他模板一样，完全可选

中间件

- PostgreSQL 数据库 `/var/lib/postgresql/data` 用于存储 PostgreSQL 数据库
- Redis，authentik 使用 Redis 作为消息队列和缓存。Redis 中的数据不需要持久化，重新启动 Redis 将导致所有会话丢失。`/data` 用于存储 Redis 数据

### 安装 Authentik

官方是推荐使用 Docker Compose 进行安装

文档：<https://docs.goauthentik.io/docs/install-config/install/docker-compose>

```bash
wget https://goauthentik.io/docker-compose.yml
```

`.env` 文件会存储 PostgreSQL 数据库的密码，以及 Authentik 的一个私钥

```bash
echo "PG_PASS=$(openssl rand -base64 36 | tr -d '\n')" >> .env
echo "AUTHENTIK_SECRET_KEY=$(openssl rand -base64 60 | tr -d '\n')" >> .env
```

Authentik 实际上会启动一个 Server 跟一个 worker 的一个两个容器 Server 的话呢一般是处理前端的界面，具体的业务流程；worker 一般是作为计划任务的这个执行者 它们之间呢并不会直接的进行沟通，而是通过连接，同一个 PostgreSQL 的数据库和 Redis 数据库来进行这个间接的信息交流

禁止出站点网络请求

```bash
AUTHENTIK_DISABLE_STARTUP_ANALYTICS=true
AUTHENTIK_DISABLE_UPDATE_CHECK=true
AUTHENTIK_ERROR_REPORTING__ENABLED=false
```

设置自动安装信息

```bash
AUTHENTIK_BOOTSTRAP_EMAIL=
AUTHENTIK_BOOTSTRAP_PASSWORD=
AUTHENTIK_BOOTSTRAP_TOKEN=
```

配置电子邮件功能

这些设置定义了 SMTP 设置，确保电子邮件功能正常工作

```bash
AUTHENTIK_EMAIL__HOST=localhost
AUTHENTIK_EMAIL__PORT=25
AUTHENTIK_EMAIL__USERNAME=
AUTHENTIK_EMAIL__PASSWORD=
AUTHENTIK_EMAIL__USE_TLS=false
AUTHENTIK_EMAIL__USE_SSL=false
AUTHENTIK_EMAIL__TIMEOUT=10
AUTHENTIK_EMAIL__FROM=authentik@localhost
```

启动后进入系统

进入到右上角管理员界面，在左侧栏有个系统 -> 设置，建议把头像中的 gravatar 给删掉，只留 initials。然后允许用户去修改自己的昵称，但是不允许修改电子邮箱和用户名

## 应用接入

Authentik 支持 5 种协议接入，分别是 SAML 2.0， OAuth 2.0/OpenID Connect (OIDC)， LDAP， SCIM，和 RADIUS

### 创建应用

![image-20250213153441824](./.assets/Authentik/image-20250213153441824.png)

authentik 中定义的应用程序用于配置和分隔授权/访问控制以及 My applications 页面中特定软件应用程序的外观

当用户登录 authentik 时，会看到 authentik 配置为为其提供身份验证和授权的应用程序（他们有权使用的应用程序）的列表

应用程序是提供商的 “另一半”。它们通常以 1 对 1 的关系存在；每个应用程序都需要一个提供程序，并且每个提供程序都可以与一个应用程序一起使用。但是，应用程序可以使用特定的其他提供程序来增强主提供程序的功能。

在以下情况下，将向用户显示应用程序：

- 用户通过策略定义了访问权限（或者应用程序没有绑定策略）
- 配置了有效的 Launch URL

可以配置以下选项：

- 名称
- 启动 URL：用户单击应用程序时打开的 URL。如果留空，authentik 会尝试根据提供程序进行推测。
  - 可以在启动 URL 中使用占位符，以便根据登录用户动态构建它们。例如，可以将 Launch URL 设置为 `https://goauthentik.io/%(username)s `，该 URL 将替换为当前登录用户的用户名。
  - 只有启动 URL 以 `http://` 或 `https://` 开头或相对 URL 的应用程序才会显示在用户的 My applications 页面上。这还可用于隐藏不应在 My applications （我的应用程序） 页面上可见但仍可供用户访问的应用程序，方法是将 Launch URL 设置为 `blank://blank`。
- 图标 （URL）：（可选）为应用程序配置图标。如果 authentik 服务器没有在 `/media` 下挂载卷，将获得文本输入。这接受绝对 URL。如果您已将单个文件挂载到容器中，则可以使用 <https://authentik.company/media/my-file.png>。如果 `/media` 下有挂载，或者配置了 S3 存储，将看到一个用于上传文件的字段。
- 发布者
- 描述

### 应用授权

要使用策略来控制哪些用户或组可以访问应用程序，请单击应用程序列表中的应用程序，然后选择 Policy/Group/User Bindings 选项卡。在那里，您可以绑定用户/组/策略以授予他们访问权限。当没有绑定任何内容时，每个人都可以访问。绑定策略会限制对特定用户或组的访问，或者通过其他自定义策略（例如限制到设定的时间或地理区域）进行访问。

默认情况下，所有用户在未绑定策略时都可以访问应用程序。

附加多个策略/组/用户时，您可以将策略引擎模式配置为：

- 要求用户传递所有绑定/成为所有组的成员 （ALL），或
- 要求用户传递任一组的 binding/be 成员 （ANY

![image-20250213171743911](./.assets/Authentik/image-20250213171743911.png)

## Providers

Provider 是一种身份验证方法，是 authentik 用来对关联应用程序的用户进行身份验证的服务。常见的提供商包括 OpenID Connect （OIDC）/OAuth2、LDAP、SAML 和通用代理提供商

提供程序是应用程序的 “另一半”。它们通常以 1 对 1 的关系存在;每个应用程序都需要一个提供程序，并且每个提供程序都可以与一个应用程序一起使用。

应用程序可以使用其他提供程序来增强主提供程序的功能。有关更多信息，请参阅 反向通道提供程序。

- LDAP：<https://docs.goauthentik.io/docs/add-secure-apps/providers/ldap/

- Oauth2：<https://docs.goauthentik.io/docs/add-secure-apps/providers/oauth2/>

### LDAP

用户位于 `ou=users，<base DN>` 下，组位于 `ou=groups，<base DN>` 下。为了帮助兼容性，每个用户都属于自己的“虚拟”组，这是大多数类 Unix 系统的标准。此组不存在于 authentik 数据库中，并且是动态生成的。这些虚拟组位于 `ou=virtual-groups，<base DN>` DN 下。

当前向用户发送以下字段：

- `cn`： 用户的用户名
- `uid`：唯一用户标识符
- `uidNumber`：用户的唯一数字标识符
- `name`：用户名
- `displayName`：用户名
- `mail`：用户的电子邮件地址
- `objectClass`：这些字符串的列表：
  - `user`
  - `organizationalPerson`
  - `goauthentik.io/ldap/user`
- `memberOf`：用户所属的所有 DN 的列表
- `homeDirectory`：用户的默认主目录路径，默认为 `/home/$username`。可以通过将 `homeDirectory` 设置为用户或组的属性来覆盖。
- `ak-active`： 如果帐户处于活动状态，则为 “true”，否则为 “false”
- `ak-superuser`： 如果账户属于具有超级用户权限的组，则为 “true”，否则为 “false”

以下字段是当前为组设置的字段：

- `cn`：组的名称
- `uid`：唯一组标识符
- `gidNumber`：组的唯一数字标识符
- `member`：组成员的所有 DN 的列表
- `objectClass`：这些字符串的列表：
  - `group`
  - `goauthentik.io/ldap/group`

还会为每个用户创建一个虚拟组，这些用户具有与组相同的字段，但具有额外的 `objectClass： goauthentik.io/ldap/virtual-group`。虚拟组 gidNumber 等于用户的 uidNumber

#### 分配 LDAP 权限

在 Directory -> Users -> Create 下创建一个要绑定的新用户帐户，在此示例中称为 ldapservice。请注意，此用户的 DN 将为 `cn=ldapservice,ou=users,dc=alpha-quant,dc=tech`

1. 导航到 *Applications* -*> Providers* -> `Provider for LDAP` 下的 LDAP 提供程序。
2. 切换到 *Permissions* 选项卡。
3. 单击 *Assign to new user* 按钮以选择要为其分配完整目录搜索权限的用户。
4. 通过键入用户名，在模式中选择 `ldapservice` 用户。选择 *Search full LDAP directory* 权限，然后单击 *Assign*

#### 创建 LDAP 应用程序和提供程序

在*应用程序* -> *应用程序* -> *使用向导创建*下创建 LDAP 应用程序，并将其命名为 `LDAP`

![img](./.assets/Authentik/general_setup14-3f263ead0aa142ee4a2465b57002ca93.png)

![img](./.assets/Authentik/general_setup15-f2d96f7d68f4ce034dc6c08ad49fff4b.png)

#### 创建 LDAP Outpost

在 *Applications* -> *Outposts* -> *Create* 下创建（或更新）LDAP Outpost。将 Type （类型） 设置为 `LDAP` 并选择在上一步中创建的 `LDAP` 应用程序

![img](./.assets/Authentik/general_setup16-1a8524a769f4f4d24e59522be62ea6cb.png)

#### ldapsearch 测试

```bash
#!/usr/bin/env bash

# no tls
ldapsearch \
    -H ldap://ldap.alpha-quant.tech:389/ \
    -x \
    -D "CN=akadmin,OU=users,DC=alpha-quant,DC=tech" \
    -b "OU=users,DC=alpha-quant,DC=tech" \
    -W
ldapsearch \
    -H ldap://ldap.alpha-quant.tech:389/ \
    -x \
    -D "CN=akadmin,OU=users,DC=alpha-quant,DC=tech" \
    -b "ou=groups,dc=alpha-quant,dc=tech" \
    -W

# tls
ldapsearch \
    -H ldap://ldap.alpha-quant.tech:389/ \
    -x \
    -ZZ \
    -D "CN=akadmin,OU=users,DC=alpha-quant,DC=tech" \
    -b "OU=users,DC=alpha-quant,DC=tech" \
    -W \
    -o TLS_CACERT="$PWD/certs/alpha-quant.tech.CA.crt"
ldapsearch \
    -H ldap://ldap.alpha-quant.tech:389/ \
    -x \
    -ZZ \
    -D "CN=akadmin,OU=users,DC=alpha-quant,DC=tech" \
    -b "ou=groups,dc=alpha-quant,dc=tech" \
    -W \
    -o TLS_CACERT="$PWD/certs/alpha-quant.tech.CA.crt"

```

## 参考资料

- <https://ecwuuuuu.com/post/authentik-tutorial-1-introduction-and-install/>