---
description: 更改用户权限的一些操作
---

# User Privileges

将文件的所有者更改为自己

```bash
sudo chown -R jiuyou2020:jiuyou2020 home
```

给文件添加指定用户的权限

```bash
sudo setfacl -m u:jiuyou2020:rwx test/
```

* r：read
* w：write
* x：execute
