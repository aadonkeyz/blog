---
title: volume的备份及还原
abbrlink: e113cc4a
date: 2019-05-09 09:17:23
categories:
    - Docker
---

现在Docker中有一个名为`registry-data`的volume，我想备份它。

{% note info %}
思路如下：
1. 创建一个文件夹，路径如下`/d/Docker/backup`，`backup`就是我保存备份的文件夹；
2. 运行一个临时的容器，它需要做以下工作：
    - 将想要备份的`registry-data`挂载到容器上；
    - 将第一步创建的文件夹，挂载到容器上；
    - 通过命令压缩`registry-data`，而后保存到第一步创建的文件夹中。
{% endnote %}

运行如下命令，将会生成`registry-data20190509.tar`压缩文件，并将其保存在`backup`文件夹中。

```md
docker run --rm -v registry-data:/source -v /d/Docker/backup:/backup alpine sh -c "cd /source && tar cvf /backup/registry-data20190509.tar ."
```

如果想将备份还原成volume，只需运行下面的命令即可：

```md
docker run --rm -v another-registry-data:/source -v /d/Docker/backup:/backup alpine sh -c "cd /source && tar xvf /backup/registry-data20190509.tar ."
```

这样就会利用备份的压缩文件，为我们生成一个名为`another-registry-data`的volume。
