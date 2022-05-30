# APT源服务搭建

## 0，环境

- apache2
  
- ubuntu-16.04-server
  

## 1，安装HTTP服务器
``` shell
$ sudo apt install apache2
```

安装完成在浏览器中输入服务器IP地址，出现**Apache2 Ubuntu Default Page**说明安装成功，服务器的根目录位于`/var/www/html`

## 2，创建APT源所需目录结构
``` shell
$ tree ubuntu
ubuntu/
├── dists
│   └── test
│       ├── InRelease
│       ├── main
│       │   └── binary-amd64
│       │       ├── Packages
│       │       └── Packages.gz
│       ├── Release
│       └── Release.gpg
└── pool
    └── main
        └── test.deb # deb软件包

6 directories, 6 files
```

## 3，相关文件解析

**以下相关命令都是在`cd /var/www/html/ubuntu`目录下进行**

**dists/**: 存放不同发行版的`deb`信息

**test/**: `test`发行版本

**main/**: 存放不用架构`deb`包相关信息

**binary-amd64**: `amd64`架构

**Packages**: `deb`包信息，使用以下命令生成，`pool`为存放真正`deb`包的位置

``` shell
$ dpkg-scanpackages pool > dists/test/main/binary-amd64/Packages
```

**Packages.gz**
``` shell
$ cat dists/test/main/binary-amd64/Packages | gzip -9 > dists/test/main/binary-amd64/Packages.gz
```

**Release**
``` shell
$ cd dists/test/
$ apt-ftparchive release . > Release
$ cd -
```

**InRelease**
``` shell
$ cd dists/test/
$ gpg --clearsign -o InRelease Release
$ cd -
```

**Release.gpg**
``` shell
$ cd dists/test/
$ gpg -abs -o Release.gpg Release
$ cd -
```

**pool/** 存放不同`component`的`deb`包

**main**: 存放所有`deb`信息列表

## 4，GPG使用(可选)

`gpg`主要用于签名认证

`注意`: 当服务没有进行签名认证的时候，在使用`apt`命令进行安装软件时，需要加上`--allow-unauthenticated`参数

- 安装
``` shell
$ sudo apt install gpg
```

- 生成密钥
``` shell
$ gpg --gen-key # 根据提示信息填写相关信息即可
```

- 列出密钥
``` shell
$ gpg --list-keys
```

- 导出公钥
``` shell    
$ sudo gpg -a -o public.key --export xxxxx # xxxxx表示gpg生成的公钥
```

# 服务器的使用

- 添加`apt`源
``` shell
$ sudo sh -c 'echo "deb http://{服务器ip}/ubuntu/ test main" > /etc/apt/sources.list.d/test.list'
```
- 添加公钥
需要在服务器上将生成的``public.key`存放到`/var/www/html/key`目录下
``` shell
$ wget -qO - http://{服务器ip}/key/public.key | sudo apt-key add -
```

- 更新缓存
``` shell
$ sudo apt update
```

- 安装所需的软件包
``` shell
$ sudo apt install <pkg-name>
```

# 参考文档

* [creating-and-hosting-your-own-deb-packages-and-apt-repo](https://earthly.dev/blog/creating-and-hosting-your-own-deb-packages-and-apt-repo/)
* [create-simple-apt-repo](https://www.cnblogs.com/bamanzi/p/create-simple-apt-repo.html)
* [gpg](http://ruanyifeng.com/blog/2013/07/gpg.html)
