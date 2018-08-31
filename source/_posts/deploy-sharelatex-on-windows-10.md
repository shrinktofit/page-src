---
title: 在Windows10上部署ShareLatex
date: 2018-08-31 11:18:00
tags:
---

# 步骤概述

* 安装Docker For Windows
* 拉取ShareLatex的Docker镜像并创建其容器
* 在ShareLate容器中，安装最新版TexLive
* 设置ShareLatex的访问端口、TexLive路径
* 安装中文字体

# 安装Docker

在Docker官网下载安装Windows版Docker。

# 拉取ShareLatex的Docker镜像并创建其容器

在Powershell命令行运行：
```powershell
docker pull sharelatex/sharelatex
```

并下载[docker-compose.yml](https://github.com/sharelatex/sharelatex/blob/master/docker-compose.yml)到任意路径下。
`docker-compose.yml`可以看作是Docker容器的启动配置文件。

编辑`docker-compose.yml`文件：

修改
```yml
ports:
    - 80:80
```
为
```yml
ports:
    - 你期望的端口号:80
```

修改
```yml
    volumes:
        - /sharelatex_data:/var/lib/sharelatex
        - /var/run/docker.sock:/var/run/docker.sock
```
为
```yml
    volumes:
        - ShareLatex存放数据的目录/sharelatex_data:/var/lib/sharelatex
        - ShareLatex与Docker通讯的目录/var/run/docker.sock:/var/run/docker.sock
        - Windows主机向ShareLatex容器传递的数据目录:ShareLatex容器接收数据的目录
```
其中“ShareLatex存放数据的目录”、“ShareLatex与Docker通讯的目录”、“Windows主机向ShareLatex容器传递的数据目录”可以是Windows主机上任意的目录，例如：“E:\Docker”。
“ShareLatex容器接收数据的目录”可以是ShareLatex容器中任意的目录，例如：“/home/host”。

`environment`项中增加一项：
```yml
environment:
        PATH: /usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/local/texlive/2018/bin/x86_64-linux:/usr/local/texlive/2017/bin/x86_64-linux
```

最后，在`docker-compose.yml`文件所在路径下运行Powershell命令行：
```powershell
docker-compose up -d
```

至此，ShareLatex容器创建完毕。

# 在ShareLatex容器中安装TexLive2018

将提前下载好的texlive.iso拷贝至E:\Host-Docker-Transfer并挂载至`/mount/iso`。

在挂载后的路径下执行
```bash
./ tl-install
```

要注意这里会提示你是否选择已有TexLive的配置，选择否，然后按照默认安装即可。

这一步应该会很久。

# 设置字体