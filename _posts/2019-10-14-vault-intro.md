---
layout: single
title:  "Intro to Vault"
date:   2019-10-14 20:00:00 +0800
categories: software
tags: [vault]
toc: true
toc_sticky: true
toc_icon: "fish"
---

# 简介
Vault 是 HashCorp 公司出品的用于管理和存储各种密码、令牌、证书等的软件。

Vault 会把 secret 先加密再存储起来。 存储系统无法获取未加密的值。 存储形式可以是文件、consul、etcd 等。

Vault 提供了 Web UI，CLI 和 HTTP API 的交互形式。

# 安装
` brew install vault`， 或者下载 vault 二进制包，这就是一个可执行文件，然后将其移入 PATH 中即可。


# 试用：

1. 启动 dev 模式的server：`vault server --dev`

2. 设置环境变量：

   * `export VAULT_ADDR='http://127.0.0.1:8200'`
   * `export VAULT_DEV_ROOT_TOKEN_ID=""`

3. 检查服务状态：`vault status`

4. secrets 的 CRUD

* 写：`vault kv put secret/hello foo=world` 将 `foo=world` 写入 `secret/hello` 路径。 注意：`secret/` 前缀是必须的。
  ```
  $ vault kv put secret/hello foo=world
  Key              Value
  ---              -----
  created_time     2019-10-14T07:43:09.224068Z
  deletion_time    n/a
  destroyed        false
  version          1
  ```

  也可以一次写入多个 kv 对，例如 `vault kv put secret/hello foo=world excited=yes`

* 读：`vault kv get secret/hello`

  ```
  $ vault kv get secret/hello
  ====== Metadata ======
  Key              Value
  ---              -----
  created_time     2019-10-14T07:44:43.209507Z
  deletion_time    n/a
  destroyed        false
  version          3

  ===== Data =====
  Key        Value
  ---        -----
  excited    yes
  foo        world
  ```

  只获取其中一个值：`vault kv get -field=foo secret/hello`

  输出为 json 格式：
  ```
  $ vault kv get -format=json secret/hello
  {
    "request_id": "1792d9f7-3103-213b-3c17-f4e39bf74a1d",
    "lease_id": "",
    "lease_duration": 0,
    "renewable": false,
    "data": {
      "data": {
        "excited": "yes",
        "foo": "world"
      },
      "metadata": {
        "created_time": "2019-10-14T07:44:43.209507Z",
        "deletion_time": "",
        "destroyed": false,
        "version": 3
      }
    },
    "warnings": null
  }
  ```

* 删除：

  ```
  $ vault kv delete secret/hello
  Success! Data deleted (if it existed) at: secret/hello
  ```

  删除后再执行 get 命令，就没有数据了：

  ```
  $ vault kv get secret/hello
  ====== Metadata ======
  Key              Value
  ---              -----
  created_time     2019-10-14T07:44:43.209507Z
  deletion_time    2019-10-14T07:51:58.242085Z
  destroyed        false
  version          3
  ``` 

# secrets engine

上面的例子中，`secret/hello` 中的 secret 是 secretes engine 的名字。

Vault 默认开启了一个类型为 `kv` 的 secrets engine， PATH 是 `secret/`。

除了 kv 外，Vault 还支持多种类型的 secrets engine，包括 Cubbyhole（默认开启且不能关闭），AWS， Axure，Consul，Databases 等等，见下图。

![vault-secrets-engines](/assets/images/vault-secrets-engines.png)

### 开启新的 secrets engine

```
$ vault secrets enable -path=test1 kv
Success! Enabled the kv secrets engine at: test1/
```

表示开启了新的 kv 类型的engine，路径是 test1。



### 查看所有 secrets engine

```
$ vault secrets list
Path          Type         Accessor              Description
----          ----         --------              -----------
Helix/        kv           kv_def19680           n/a
cubbyhole/    cubbyhole    cubbyhole_568ac986    per-token private secret storage
identity/     identity     identity_6ab7a61a     identity store
kv/           kv           kv_1d865a45           n/a
secret/       kv           kv_d91ea0c5           key/value secret storage
sys/          system       system_a74d3a4e       system endpoints used for control, policy and debugging
test1/        kv           kv_4725061e           n/a
```

### 关闭某个 secrets engine
```
$ vault secrets disable test1/
Success! Disabled the secrets engine (if it existed) at: test1/

# 验证
$ vault secrets list
Path          Type         Accessor              Description
----          ----         --------              -----------
Helix/        kv           kv_def19680           n/a
cubbyhole/    cubbyhole    cubbyhole_568ac986    per-token private secret storage
identity/     identity     identity_6ab7a61a     identity store
kv/           kv           kv_1d865a45           n/a
secret/       kv           kv_d91ea0c5           key/value secret storage
sys/          system       system_a74d3a4e       system endpoints used for control, policy and debugging
```
> 1. 命令中接收的参数是该 engine 的 PATH，而不是 TYPE
> 2. 关闭  secrets engine 会删除其中所有的 secrets （包括存储层面的删除）


### secrets engine 是什么

Vault 要处理各种各样的 secrets 的 CRUD 操作，但是对于不同类型的 secrets 处理方式肯定也是不同的。
因此就需要抽象出一层 secrets engine，由它们来决定具体如何操作。
换句话说， CRUD 操作是接口，各个 secrets engine 是具体实现。

# 部署

生产级别的部署参考：https://learn.hashicorp.com/vault/getting-started/deploy。
这里建议使用 consul 作为后端存储。

web ui：

- dev 模式下 web ui 是默认开启的， 地址是： localhost:8200/ui。 登录时填写保存下的 root token。
- 非 dev 模式下 web ui 默认没有激活，需要修改配置文件来激活，具体见：https://learn.hashicorp.com/vault/getting-started/ui
 
# 开发

* [HTTP API Doc](https://www.vaultproject.io/api/overview.html)
* [客户端库](https://www.vaultproject.io/api/libraries.html)






