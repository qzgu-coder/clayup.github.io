---
layout: post # 使用post模版
title:  "docker 安装 Ubuntu桌面版 并搭建 jekyII 环境"
date:   2021-03-26
categories: jekyII 
tags:   jekyII docker Ubuntu

---

# docker 安装 Ubuntu桌面版 并搭建 jekyII 环境

6080是VNC Web 端口，5900是VNC桌面客户端端口，8080是`jekyI` 博客的端口

```cmd
docker pull dorowu/ubuntu-desktop-lxde-vnc
docker run -itd --name=my_ubuntu -p 6080:80 -p 8080:8080 -p 5900:5900  -e PRESOLUTION=1920x1080 -e VNC_PASSWORD=000 -v /dev/shm:/dev/shm dorowu/ubuntu-desktop-lxde-vnc
```

安装`jekyll`

```bash
echo '# Install Ruby Gems to ~/gems' >> ~/.bashrc
echo 'export GEM_HOME="$HOME/gems"' >> ~/.bashrc
echo 'export PATH="$HOME/gems/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc
# 关闭SSH窗口，再重新链接
apt install ruby-full build-essential zlib1g-dev
gem install bundler jekyll
```

克隆GitHub源代码

```cmd
git clone https://github.com/clayup/clayup.github.io/
```

```cmd
jekyll serve -H 0.0.0.0 -P 80
```

