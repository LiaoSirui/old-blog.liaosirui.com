## 配置最基础的邮件服务器

docker-mailserver

- <https://docker-mailserver.github.io/docker-mailserver/latest/>
- <https://github.com/docker-mailserver/docker-mailserver>

`docker-mailserver` 使用 `docker compose` 来管理

```bash
mkdir ~/mailserver && cd ~/mailserver
 
DMS_GITHUB_URL='https://raw.githubusercontent.com/docker-mailserver/docker-mailserver/master'
wget "${DMS_GITHUB_URL}/docker-compose.yml"
wget "${DMS_GITHUB_URL}/mailserver.env"
wget "${DMS_GITHUB_URL}/setup.sh"
chmod a+x ./setup.sh
```

配置 mailserver 本身。首先需要添加一个邮件账户并设置密码，在未添加账户的情况下 mailserver 会反复重启

```bash
docker compose exec -ti mailserver setup email add postmaster@alpha-quant.tech
```

## 个性化设置

- `Catch-All`

```bash
docker compose exec -ti mailserver setup email add info@alpha-quant.tech
echo "@alpha-quant.tech info@alpha-quant.tech" >> docker-data/dms/config/postfix-virtual.cf
```

将所有未匹配到收件人的邮件转投给这个账户

尝试给存在的账户发信，也会被转发。这是由于 `postfix` 具有虚拟高于真实的查找优先级，发向真实账户的邮件没来及匹配真实账户就被 Catch All 规则匹配走了。

解决方法也很简单：既然被优先级抢走了，就用更高的优先级抢回来。对于任何不想被 Catch All 捕获的地址，添加一条指向自己的别名。至于文件内规则的顺序并不重要，别名拥有高于 Catch All 的优先级。例如，我希望 `admin@alpha-quant.tech` 不被捕获，可以添加别名：

```bash
echo "admin@alpha-quant.tech admin@alpha-quant.tech" >> docker-data/dms/config/postfix-virtual.cf
```

- `no-reply`

```bash
docker compose exec -ti mailserver setup email add no-reply@alpha-quant.tech 
echo "devnull: /dev/null" >> docker-data/dms/config/postfix-aliases.cf
echo "no-reply@alpha-quant.tech devnull\ndevnull@alpha-quant.tech devnull" >> docker-data/dms/config/postfix-virtual.cf
```

注册一个别名 `devnull` 指向 `/dev/null`，然后将 `no-reply` 账户的邮件全部转到 `devnull`。此时此刻我们又要感谢虚拟的高优先级，否则我们无法将发往 `no-reply` 这个真实账户的信件转发

需要特别注意的是，如果设置了任何 Catch-All 规则，则需要额外添加一条 `devnull@alpha-quant.tech devnull` 规则

- `config/postfix-main.cf`

```bash
smtpd_helo_required=no
smtpd_helo_restrictions=
```

