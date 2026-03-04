## SSH 在 Cisco 上成立的两个必要条件

要让路由器发起 SSH，需要：

1️⃣ 必须有 **domain name**
 2️⃣ 必须生成 **RSA crypto key**

否则执行：

```
ssh -l user 10.1.1.1
```

会报错：

```
% Please create RSA keys to enable SSH
```

### 🔹 1️⃣ 需要 domain name

SSH key 生成时依赖：

```
ip domain-name example.com
```

否则：

```
crypto key generate rsa
```

会失败。

------

### 🔹 2️⃣ 需要 RSA key

```
crypto key generate rsa modulus 2048
```

SSH 依赖 RSA key。

没有 key = SSH 不工作。