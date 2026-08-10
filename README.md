# GL-BE6500 Region Changer

> GL.iNet GL-BE6500 深度定制与配置指南。
>
> 本项目记录 GL-BE6500 在 OpenWrt 环境下的区域识别码修改、OpenClash 部署以及家庭网络优化过程。

---

## 项目简介

GL-BE6500 是一款基于 Qualcomm IPQ5332 平台的高性能 Wi-Fi 7 路由器。本项目基于实际设备测试，整理以下内容：

- 设备区域识别码修改
- ART 分区备份与恢复
- OpenClash + Mihomo Meta Core 部署
- Clash/Mihomo 订阅接入
- 家庭网络透明代理配置
- 常见问题排查

实测环境：

```text
型号：GL-BE6500
SoC：Qualcomm IPQ5332
系统：OpenWrt 23.05-SNAPSHOT
Kernel：5.4.213
架构：aarch64
```

---

# 功能文档

## 1. 区域识别码修改

文档：

```
docs/GL-BE6500区域识别码修改指南.md
```

内容包括：

- 读取设备区域码
- 定位 ART 分区
- 备份 ART 数据
- 修改 CN / US 区域识别码
- 写入验证
- 恢复方法

⚠️ 修改 ART 分区存在风险，请务必备份。

---

## 2. OpenClash 安装

文档：

```
docs/OpenClash-GL-BE6500-Install.md
```

实现方案：

```text
家庭设备
    |
    |
GL-BE6500
    |
OpenClash
    |
Mihomo Meta Core
    |
Clash / Mihomo 订阅
```

特点：

- 无需每台设备安装代理软件
- 支持手机、电脑、电视、IoT设备
- 使用新版 Clash 订阅格式
- 支持 Fake-IP

---

## 3. OpenClash 高级优化

文档：

```
docs/OpenClash-Advanced-Optimization.md
```

包含：

- 稳定运行配置
- DNS 调整建议
- TUN 模式说明
- 游戏设备分流
- 节点优化建议
- 故障排查方法

---

# 快速开始

## 第一步：连接设备

SSH：

```bash
ssh root@192.168.8.1
```

## 第二步：备份重要数据

修改区域前：

- 备份 ART 分区
- 保存 SHA256 校验值
- 下载备份到电脑

## 第三步：安装 OpenClash

安装：

```text
LuCI → 服务 → OpenClash
```

然后安装：

```text
Mihomo Meta Core
```

## 第四步：导入订阅

添加 Clash/Mihomo 订阅地址。

推荐：

```text
Rule 模式
增强模式（Fake-IP）
IPv6 根据实际情况调整
```

---

# 推荐网络结构

```text
Internet
   |
GL-BE6500
   |
OpenClash
   |
Mihomo
   |
代理节点
   |
家庭全部设备
```

客户端无需单独配置代理。

---

# 常用排查命令

查看设备信息：

```bash
ubus call system board
```

查看区域码：

```bash
cat /proc/gl-hw-info/country_code
```

查看 OpenClash 日志：

```bash
tail -n 100 /tmp/openclash.log
```

查看 Mihomo：

```bash
/etc/openclash/core/clash_meta -v
```

---

# 注意事项

1. 不同 GL-BE6500 固件版本可能存在差异。
2. ART 分区包含设备关键校准信息，修改前必须备份。
3. OpenClash 首次启动需要初始化 DNS、规则和透明代理规则，可能需要等待。
4. 不建议同时启用多个 DNS 接管服务。

---

# License

本文档用于技术研究与设备配置记录。