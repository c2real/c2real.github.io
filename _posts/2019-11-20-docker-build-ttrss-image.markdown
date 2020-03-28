---
layout: post
title:  "Docker build TTRSS Image"
date:   2019-11-20 6:45:00 +0800
categories: TTRSS Docker
---

### Docker Build Process
基于Awesome-TTRSS的Dockerfile, 可以加入自己想要的配置，那么 https://c2real.github.io/2019/11/15/setup-ttrss-by-docker/ 里面提到的修改，可以在生成镜像阶段直接修改，得到自己想要的镜像，上传到Docker Hub里。

- 在Dockerfile中修改对应的TTRSS的路径，比如/var/www/ttrss
- 修改config-db.php文件，保持子路径一直，/var/www/ttrss
- 修改后台守护进程启动文件，src/s6/update-daemon/run 子目录地址
- 以及docker-compose.yml文件对应地址

