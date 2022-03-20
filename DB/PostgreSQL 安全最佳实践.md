# PostgreSQL 安全最佳实践

数据库是黑客眼中的“圣杯”，需要我们像对待“花朵”一样精心呵护它。本文主要介绍数据库保护的最佳实践。首先，从最常用的开源数据库 PostgreSQL 开始，我们将一一介绍诸位需要考虑的几个安全层级：

- 网络层安全
- 传输层安全
- 数据库层安全

## PostgreSQL 中网络层的安全

#### 防火墙

理想情况下，PostgreSQL 服务器应当是完全隔离，不允许任何入站申请、SSH 或 psql 的。然而，PostgreSQL 没有对这类网闸设置提供开箱即用的支持。

我们最多也只能通过设置防火墙，锁定数据库所在节点的端口级访问来提升数据库服务器的安全性。默认情况下，PostgreSQL 监听 TCP 端口 5432。而根据操作系统的不同，锁定其他端口的方式也会有所不同。以 Linux 最常用的防火墙`iptables`为例，下面这几行代码就可轻松完成任务：

```
# 确保已有连接不被dropiptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT
# 允许SSH.iptables -A INPUT -p tcp -m state --state NEW --dport 22 -j ACCEPT
# 允许PostgreSQL.iptables -A INPUT -p tcp -m state --state NEW --dport 5432 -j ACCEPT
# 允许所有出站，drop所有入站iptables -A OUTPUT -j ACCEPTiptables -A INPUT -j DROPiptables -A FORWARD -j DROP
```

在更新 iptables 的规则时，建议使用[iptables-apply](https://man7.org/linux/man-pages/man8/iptables-apply.8.html?fileGuid=jWHTCry9KtVKqy3D)工具。这样，哪怕你不小心把自己也锁在外面了，它也能自动回滚更改。

这条 PostgreSQL 规则会允许所有人连接到端口 5432 上。当然，你也可以将其修改为只接受特定 IP 地址或子网让限制更加严格：

```
# 仅允许本地子网访问PostgreSQL端口iptables -A INPUT -p tcp -m state --state NEW --dport 5432 -s 192.168.1.0/24 -j ACCEPT
```

继续讨论我们的理想情况。想要完全限制到 5432 端口的所有入站连接需要一个某种类型的本地代理，由它维持一个到客户端节点的持久出站连接，并且能将流量代理到本地 PostgreSQL 实例。

这种代理被称作是“反向通道”。具体使用方法可以通过 SSH 远程端口转发的功能进行演示。运行下列指令可以在 PostgreSQL 数据库运行的节点上打开一个反向通道：

```
ssh -f -N -T -R 5432:localhost:5432 user@<client-host>
```

PostgreSQL 的节点需要能访问<client-host>，其上的 SSH 守护进程（daemon）也要处于运行状态。下列指令会将数据库服务器上端口 5432 转发到客户端机器的端口 5432 上，这样你就可以通过这个通道连接数据库了：

```
psql "host=localhost port=5432 user=postgres dbname=postgres"
```

#### PostgreSQL 监听地址

通过配置文件指令`listen_addresses`来限制服务器监听客户端连接的地址是个好习惯。如果运行 PostgreSQL 的节点上有多个网络接口，`listen_addresses`可以确保服务器只会监听客户端所连接的一个或多个接口：

```
listen_addresses = 'localhost, 192.168.0.1'
```

如果连接到数据库的客户端总是驻留在同一节点上，或是与数据库共同驻留在同一个 Kubernetes pod 上，PostgreSQL 作为 sidecar 容器运行，禁用套接字监听（socket）可以完全消除网络的影响。将监听地址设置为空字符串可以使服务器只接受 Unix 域的套接字连接：

```
listen_addresses = ''
```

## PostgreSQL 中传输层级的安全

当世界上大部分的网络都转投向 HTTP 的怀抱时，为数据库连接选择强传输加密也变成了一个必备项目。PostgreSQL 本身即支持 TLS（因为历史遗留问题，在文档、配置文件，以及 CLI 中仍被称作是 SSL），我们也可以使用它进行服务器端和客户端认证。

#### 服务器端 TLS

对于服务器的认证，我们首先需要为服务器准备一份用于和相连接的客户端认证的证书。在 Let's Encrypt 上，我们可以找到免费提供的 X.509 证书，具体使用方法以[certbot](https://certbot.eff.org/?fileGuid=jWHTCry9KtVKqy3D)的命令行工具为例：

```
certbot certonly --standalone -d postgres.example.com
```

需要注意的是，certbot 默认使用 ACME 规范的 HTTP-01 挑战来验证证书请求，这里我们就需要确保请求域指向节点的 DNS 有效，并且端口 80 处于开放状态。

除了 Let's Encrypt 之外，如果想在本地生成所有的信息，我们还可以选择 openssl 命令行工具：

```bash
# 生成一个自签名的服务器CA
openssl req -sha256 -new -x509 -days 365 -nodes \
    -out server-ca.crt \
    -keyout server-ca.key
    
# 生成服务器CSR，CN里填需要连接到数据库的主机名 
openssl req -sha256 -new -nodes \
    -subj "/CN=postgres.example.com" \
    -out server.csr \
    -keyout server.key
    
# 签证书
openssl x509 -req -sha256 -days 365 \
    -in server.csr \
    -CA server-ca.crt \
    -CAkey server-ca.key \
    -CAcreateserial \
    -out server.crt
```

在生产环境中记得在证书过期前更新。

#### 客户端 TLS

通过验证客户端提供的 X.509 证书是由可信的证书颁发机构（CA）签名，服务器便可以验证连接客户端的身份。

建议使用不同的 CA 来分别给客户端和服务器端发证书。下列代码创建了一个客户端 CA，并用它来给客户签证书：

```bash
# 生成一个自签名的客户端CA
openssl req -sha256 -new -x509 -days 365 -nodes \
    -out client-ca.crt \
    -keyout client-ca.key
    
# 生成客户端CSR。CN必须填用于连接的数据库角色名
openssl req -sha256 -new -nodes \
    -subj "/CN=alice" \
    -out client.csr \
    -keyout server.key
    
# 签证书
openssl x509 -req -sha256 -days 365 \
    -in client.csr \
    -CA client-ca.crt \
    -CAkey client-ca.key \
    -CAcreateserial \
    -out client.crt
```

这里需要注意，客户端证书里的 CommonName（CN）字段必须要包含用于连接的数据库账号。PostgreSQL 服务器将用 CN 来创建客户端的身份。

#### TLS 配置

小结一下，我们现在可以配置 PostgreSQL 服务器来接收 TLS 连接了：

```
ssl = on
ssl_cert_file = '/path/to/server.crt'
ssl_key_file = '/path/to/server.key'
ssl_ca_file = '/path/to/client-ca.crt'
# 这里默认是on，但出于安全考虑，特意写出来总是没错的 
ssl_prefer_server_ciphers = on
# TLS 1.3能提供最强的安全保护。在控制服务器和客户端时建议使用ssl_min_protocol_version = 'TLSv1.3'
```

我们还需要再配置的就只剩下用于更新 PostgreSQL 服务器的基于主机的认证文件（`pg_hba.conf`）了。它可以要求所有的连接使用 TLS，并通过 X.509 证书对客户端进行认证。

```bash
# TYPE  DATABASE        USER            ADDRESS                 METHOD
hostssl all             all             ::/0                    cert
hostssl all             all             0.0.0.0/0               cert
```

现在，连接到数据库服务器的客户端就需要提供由客户端 CA 签名的有效证书了：

```bash
psql "host=postgres.example.com \
      user=alice \
      dbname=postgres \
      sslmode=verify-full \
      sslrootcert=/path/to/server-ca.crt \
      sslcert=/path/to/client.crt \
      sslkey=/path/to/client.key"
```

需要注意的是，默认情况下，psql 不会进行服务器证书认证，所以我们需要将“sslmode”设置为“`verify-full`”或者“`verify-ca`”。具体设置要看你是否使用了编码后的 X.509 中 CN 字段的主机名对服务器进行连接。

为了简化指令，并且避免在每次连接到数据库时都要重新输入一遍到 TLS 的路径，可以通过使用 PostgreSQL 的连接服务文件来解决。它可以将连接参数分组到“services”中，通过连接字符串中的“service”参数进行引用。

用下列代码创建“`~/.pg_service.conf`”文件：

```ini
[example]
host=postgres.example.com
user=alice
sslmode=verify-full
sslrootcert=/path/to/server-ca.crt
sslcert=/path/to/client.crt
sslkey=/path/to/client.key
```

现在，当连接到数据库时，我们只需要指定服务名和想要连接的数据库名即可：

```shell
psql "service=example dbname=postgres"
```

## PostgreSQL 中数据库层级安全

#### 角色

目前为止，我们已经探讨的内容有以下几点：如何从未认证的网络连接中保护 PostgreSQL 数据库，如何使用强加密进行数据传输，以及如何通过共同的 TLS 认证让服务器与客户互相信任对方身份。接下来，我们将继续分析用户在这里能做什么，连接到数据库后他们可以访问什么，以及如何验证他们的身份。这一步通常被称作是“授权”。

PostgreSQL 有一套完整的、依据角色（role）建立的用户权限系统。在现代的 PostgreSQL（8.1 及以上版本）中，“角色”是“用户”的同义词。无论你用什么样的数据库账户名，比如 psql 中的"user=alice"，它其实都是一个可以连接数据库的、拥有 LOGIN 属性的角色。也就是说，下面两条指令其实效果一样：

```sql
CREATE USER alice;
CREATE ROLE alice LOGIN;
```

除了登入的权限外，角色还可以拥有其他属性：可以通过所有的权限检查的`SUPERUSER`，可以创建数据库的`CREATEDB`，可以创建其他角色的`CREATEROLE`，等等。

除了属性外，角色被授予的权限可以分为两大类：一是其他角色的成员身份（membership），二是数据库对象的权限。下面我们将介绍这两类都是如何工作的。

#### 授予角色权限

假设，我们需要跟踪服务器清单：

```sql
CREATE TABLE server_inventory (
    id            int PRIMARY KEY,
    description   text,
    ip_address    text,
    environment   text,
    owner         text,
);
```

默认情况下，PostgreSQL 安装会包含一个用于引导数据库的超级用户角色，通常被称作是“postgres”。使用这一角色进行所有的数据库操作相当于在 Linux 系统中经常使用“root”登入，永远不是个好主意。所以，我们要创建一个无权限角色，根据需要为其授予最小权限。

通过创建一个“组角色”并授权其他角色（角色与用户一一对应）为组内成员，可以避免为每一个用户或角色单独分配权限的麻烦。假如说，我们想要授予开发者 Alice 和 Bob 查看服务器清单，但不允许其修改的权限：

```sql
-- 创建一个自身没有登入能力的组角色，授予其在服务器清单表中进行SELECT的权限
GRANT SELECT ON server_inventory TO developer;
-- 创建两个用户账号，在登入权限的基础上继承“developer”权限 
CREATE ROLE alice LOGIN INHERIT;
CREATE ROLE bob LOGIN INHERIT;
-- 分配给两个用户账号“developer”组角色
GRANT developer TO alice, bob;
```

现在，当连接到数据库时，Alice 和 Bob 都会继承“developer”组角色中的权限，并且可以查询数据库清单表。

`SELECT`权限默认对表的所有列有效，但这一点也是可以更改的。假设我们只想让实习生查看服务器的大致信息但又不想让他们连接到服务器，那么就可以选择隐藏 IP 地址：

```sql
CREATE ROLE intern;
GRANT SELECT(id, description) ON server_inventory TO intern;
CREATE ROLE charlie LOGIN INHERIT;
GRANT intern TO charlie;
```

其他常用数据库对象权限还有“`INSERT`”、“`UPDATE`”、“`DELETE`”，以及“`TRUNCATE`”，分别对应它们各自的 SQL 语句。我们还可以为连接到特定数据库、新建模式或模式中新建对象、执行函数等等分配权限。PostgreSQL 文档中的[权限](https://www.postgresql.org/docs/13/ddl-priv.html?fileGuid=jWHTCry9KtVKqy3D)章节提供了完整列表。

#### 行级安全策略

PostgreSQL 权限系统的更高级的玩法是行级安全策略（RLS），它允许你为表中的部分行分配权限。这既包括可以用`SELECT`查询的行，也包括可以`INSERT`、`UPDATE`以及`DELETE`的行。

想要使用行级安全，我们需要准备两件事情：在表中启用 RLS，为其定义一个用于控制行级访问的策略。

继续之前的例子，假设我们只想允许用户更新他们自己的服务器。那么，第一步，启用表中的 RLS：

```
ALTER TABLE server_inventory ENABLE ROW LEVEL SECURITY;
```

如果没有定义任何策略的话，PostgreSQL 会默认到“拒绝”策略上，这就意味着除了表的创建者/所有者之外，没有任何角色能够访问。

行安全策略是一个布尔表达式，是 PostgreSQL 用于判定所有需要返回或更新的行的条件。SELECT 语句返回的行将对照 USING 子语句所指定的表达式进行检查，而通过 INSERT，UPDATE 或 DELETE 语句更新的行将对照 WITH CHECK 表达式进行检查。

首先，让我们定义几个策略，允许用户查看所有服务器，但只能更新他们自己服务器，这个“自己”是由表中的“owner”字段决定的。

```sql
CREATE POLICY select_all_servers
    ON server_inventory FOR SELECT
    USING (true);
CREATE POLICY update_own_servers
    ON server_inventory FOR UPDATE
    USING (current_user = owner)
    WITH CHECK (current_user = owner);
```

注，只有表的所有者可以创建或更新 RLS。

#### 审计

到目前为止，我们主要都在讨论先手防御的安全措施。根据其中一项基本的安全原则——深度防御，我们分析了这些措施是如何通过互相叠加，帮助减缓（假想中的）攻击者进攻系统的进程。

留存准确且详细的审计追踪记录是系统安全属性中常常被忽视的一点。监控数据库服务器端的网络层或节点层访问并不在本文的讨论范围内，但当涉及到 PostgreSQL 服务器本身时，不妨先来看看我们有什么可选项。

想要更清晰地观测数据库内情况时，最常用的方法是启用详细日志记录。在服务器配置文件中添加以下指令，开启对所有连接尝试以及已执行 SQL 的日志记录。

```ini
; 记录成功与非成功连接尝试 
log_connections = on
; 记录终止对话
log_disconnections = on
; 记录所有执行过的SQL语句
log_statement = all
```

不幸的是，对于标准的自托管 PostgreSQL，这几乎是在不安装其他插件的情况下你能做到的全部了。虽然总比没有强，但在少数数据库服务器和简单的“grep”之外，它的延展性并不好。

如果想要更高级的 PostgreSQL 审计解决方案，诸如[pgAudit](https://github.com/pgaudit/pgaudit?fileGuid=jWHTCry9KtVKqy3D)这样的第三方插件是个不错的选择。自托管的 PostgreSQL 需要手动安装，但对于一些托管版本的 PostgreSQL，比如 AWS RDS 这些，本身就有，我们只需启用即可。

对于记录的语句，pgAudit 提供了更多的结构和粒度。但要记住，它并没有脱离日志的范围，也就是说，如果需要将审计日志以结构化的格式传输到外部 SIEM 系统以作更详细的分析时，恐怕会很难。

## PostgreSQL 中基于证书的访问

[Teleport数据库访问](https://goteleport.com/blog/introducing-database-access/?fileGuid=jWHTCry9KtVKqy3D)（Teleport for Database Access）是一款开源项目，在它的帮助下，我们可以实现本文中介绍的所有用于保护 PostgreSQL 和其他数据库的最佳实践。

- 用户可以通过单点登录流程访问数据库，使用短期 X.509 证书代替常规凭证进行认证。
- 数据库不需暴露在公网中，可以使用 Teleport 内置的反向通道子系统在网闸环境中安全运行。
- 管理员和审计人员可以在审计日志中查看数据库活动，例如与特定用户标识相关联的会话和 SQL 语句，并可以选择将其传送至外部系统。

## 结语

与任何以安全为前提设计的系统一样，正确地保护对数据库实例的访问需要在网络协议栈的多个层次上采取保护措施。在这篇文章中，我们从网络和传输安全开始，探讨了如何使用 PostgreSQL 灵活的用户权限系统。

原文链接：

https://goteleport.com/blog/securing-postgres-postgresql?fileGuid=jWHTCry9KtVKqy3D