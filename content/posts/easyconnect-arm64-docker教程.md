---
title: easyconnect arm64 docker教程
date: 2022-12-10 10:28:01
tags:
---

基于[Hagb的develop分支](https://github.com/Hagb/docker-easyconnect/tree/develop)

在qemu中模拟arm64执行

修改easyconnect为最新配置文件

修改Dockerfile，在`CMD ["start.sh"]`前加入

```dockerfile
RUN busybox wget http://download.sangfor.com.cn/download/product/sslvpn/pkg/linux_767/EasyConnect_x64_7_6_7_3.deb && dpkg -x EasyConnect_x64_7_6_7_3.deb ec_7.6.3 && cp ec_7.6.3/usr/share/sangfor/EasyConnect/resources/conf/* /usr/share/sangfor/EasyConnect/resources/conf_backup
```

编译arm镜像

```bash
docker image build -f Dockerfile.build -t hagb/docker-easyconnect:build .
docker image build \
    --build-arg EC_URL=https://download.sangfor.com.cn/download/product/sslvpn/SSLVPN%E4%BF%A1%E5%88%9B%E5%AE%A2%E6%88%B7%E7%AB%AF/%E7%BB%9F%E4%BF%A1UOS%20%E6%B5%B7%E6%80%9D%E9%BA%92%E9%BA%9F990%20SSL%20VPN%E5%AE%A2%E6%88%B7%E7%AB%AF.zip \
    --build-arg EC_DEB_PATH=02-升级包及安装文件/EasyConnect_UOS_arm64-20220302.deb \
    --tag hagb/docker-easyconnect -f Dockerfile .
```

运行镜像

```bash
docker run --device /dev/net/tun --cap-add NET_ADMIN -ti -e PASSWORD=xxxx -v ecdata:/root -p 5901:5901 -p 1080:1080 -p 8888:8888 hagb/docker-easyconnect:latest
```
