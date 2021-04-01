title: linux 导入证书
permalink: linux_import_cert/
tags: [linux]
date: 2016-12-21 09:29:42
---

把`xx.crt` 文件拷贝到 `/usr/local/share/ca-certificates/`
然后运行
```sh
sudo update-ca-certificates
```
然后就可以了
