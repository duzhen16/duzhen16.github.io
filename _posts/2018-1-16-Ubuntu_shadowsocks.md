---
layout:     post                    # layout, do not alter
title:         Ubuntu 安装 shadowsocks      # title
subtitle:   增加新的加密方式 # subtitle
date:       2018-1-19              # time
author:     xSun                    # author
header-img: img/post-bg-unix-linux.jpg    #bg image
catalog: true                       # catalog or not
tags:                               #tags
    - Ubuntu
---

>有幸生在种花家，上网从此抓了瞎。作为墙内科学上网的必需品——shadowsocks客户端已成了生活中必不可少的一部分，Windows上可以直接安装最新版的客户端即可，但是在Ubuntu上，还是需要花一点功夫的。本文记录下曾经踩过的雷。

##  chacha20-ietf-poly1305

过去，Ubuntu上安装shadowsocks也比较方便，支持图形界面的方式可以直接通过ppa 的方式。但是，十九大后我们进入了新时代，VPN加密方式也不得不进行了调整，原因大家都懂的。新的加密方式就不再被支持了，如chacha20-ietf-poly1305。这种情况给笔者自己也带来了极大的不便，本着钻研的态度，势必要找到新的办法。

因此，综合网上的资料，将安装方式记录在此。
## install 
总的来说，编译图形化的客户端需要3个源：Botan，libQtShadowsocks，shadowsocks-qt5。当然，肯定会有依赖包的问题。如下。

1.安装依赖包
``` stylus
sudo apt-get install libsodium-dev
sudo apt-get install qt5-qmake qtbase5-dev libqrencode-dev libappindicator-dev libzbar-dev 
```
2.编译Botan
``` stylus
# Install libbotan-2.x from source
wget https://botan.randombit.net/releases/Botan-2.3.0.tgz
tar xvf Botan-2.3.0.tgz
cd Botan-2.3.0
./configure.py
make
sudo make install
sudo ldconfig
```
3.编译libQtShadowsocks
``` stylus
# Install libQtShadowsocks
git clone https://github.com/shadowsocks/libQtShadowsocks.git
cd libQtShadowsocks
vim CMakeLists.txt
# Change Line 8 as "On"
mkdir build
cd build
cmake ..
make
sudo make install
```
4.编译shadowsocks-qt5
``` stylus
# Install 
git clone https://github.com/shadowsocks/shadowsocks-qt5
cd shadowsocks-qt5
mkdir build
cd build
cmake ..
make
sudo make install
```
5.网络设置
最后一步，我们需要进行网络代理设置，将流量转发到1080端口。如下图：
![](http://p194hb5ge.bkt.clouddn.com/net.png)

## 结束
最终打开shadowsocks-Qt5即可，填入账户信息。我们发现，我们自己编译的客户端已经可以支持更多的加密方式了。终于又可以摆脱百度的束缚，投入Google的怀抱！
