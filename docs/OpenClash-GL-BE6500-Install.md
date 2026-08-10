# GL-BE6500 安装 OpenClash（Mihomo）教程

本文记录在 **GL.iNet GL-BE6500（Qualcomm IPQ5332 / OpenWrt 23.05-SNAPSHOT）** 上安装 OpenClash，并使用新版 Clash 订阅的完整过程。

## 一、适用场景

由于部分机场已经停止支持旧版 Clash 配置，而 GL-BE6500 原生固件中的旧代理方案兼容性有限，因此采用：

```
GL-BE6500
    ↓
OpenWrt / LuCI
    ↓
OpenClash
    ↓
Mihomo Meta Core
    ↓
Clash YAML订阅
```

实现整个家庭局域网透明代理。

## 二、设备环境

实测环境：

```
型号：GL-BE6500
SoC：Qualcomm IPQ5332
系统：OpenWrt 23.05-SNAPSHOT
架构：aarch64_cortex-a53_neon-vfpv4
```

确认：

```bash
cat /etc/openwrt_release
uname -a
```

## 三、安装前准备

SSH 登录：

```bash
ssh root@192.168.8.1
```

更新软件源：

```bash
opkg update
```

确认基础组件：

```bash
opkg list-installed | grep -E 'curl|ca-certificates'
```

如果缺少：

```bash
opkg install curl ca-certificates
```

## 四、安装 OpenClash

由于 GL-BE6500 软件源中没有直接提供 OpenClash，采用官方 GitHub Release 下载 ipk。

安装脚本示例：

```bash
cd /tmp
wget https://raw.githubusercontent.com/vernesong/OpenClash/master/dev/openclash.sh
chmod +x openclash.sh
./openclash.sh
```

或者手动下载：

```bash
wget https://github.com/vernesong/OpenClash/releases/latest/download/luci-app-openclash.ipk
opkg install luci-app-openclash.ipk
```

安装完成后：

```text
LuCI → 服务 → OpenClash
```

如果菜单没有出现：

```bash
/etc/init.d/uhttpd restart
```

或刷新浏览器缓存。

## 五、安装 Mihomo Meta Core

OpenClash 首次运行会提示安装内核。

选择：

```
Mihomo Meta Core
```

不要使用旧 Clash Core。

原因：

- 支持新版 Clash YAML
- 支持 SS/Vmess/Trojan 等协议
- 支持 Fake-IP
- 支持 TUN 模式

## 六、导入订阅

进入：

```
OpenClash
 → 配置订阅
```

填写机场提供的 Clash/Mihomo 订阅地址。

下载成功后，在：

```
配置文件
```
可以看到生成的 yaml 文件。

## 七、推荐初始配置

### 运行模式

推荐：

```
Fake-IP 增强模式
```

如果第一次启动较慢属于正常现象，需要等待：

- DNS 初始化
- Fake-IP 地址池建立
- 规则加载
- 防火墙规则生成

### 代理模式

推荐：

```
规则模式
```

效果：

```
国内网站 → DIRECT
国外网站 → Proxy
```

## 八、DNS 设置建议

推荐保持订阅默认 DNS 配置。

不要初期同时开启：

- DNS 劫持
- TUN
- 自定义复杂覆写

待基础代理稳定后再调整。

## 九、验证代理

### 路由器查看

OpenClash 首页：

检查：

- 活动连接
- 上传/下载流量
- CPU 使用率
- 内存占用

### 局域网设备测试

手机关闭自身 VPN，只连接 Wi-Fi。

测试：

- 国内网站
- Google
- YouTube

如果手机可以直接访问，说明路由器透明代理成功。

## 十、常见问题

### 1. 国内外网站刚启动打不开

原因：

OpenClash 初始化需要时间。

等待约几十秒到几分钟即可。

### 2. Clash Verge 可以，路由器不行

检查：

- YAML 是否为新版格式
- Mihomo Core 是否安装
- DNS 是否正常
- Fake-IP 是否启动

### 3. curl 测试 SSL 失败

例如：

```text
SSL/TLS connection failed
```

先确认：

```bash
curl -I https://www.google.com
```

再确认 OpenClash 节点状态。

### 4. 不建议立即开启 TUN

GL-BE6500 性能较强，但透明代理 + Fake-IP 已满足家庭使用需求。

## 十一、最终推荐方案

```
GL-BE6500

OpenClash
 ├── Mihomo Meta Core
 ├── Fake-IP 增强模式
 ├── 规则模式
 ├── Clash YAML订阅
 └── DNS保持默认

客户端：无需安装代理软件
```

该方案适合：

- 手机
- 平板
- PC
- 游戏设备
- 智能家居设备

实现整个家庭网络统一代理管理。
