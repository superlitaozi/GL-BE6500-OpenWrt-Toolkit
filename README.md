# GL-BE6500 OpenWrt Toolkit

> GL.iNet GL-BE6500 深度定制与优化工具集。
>
> 本项目记录 GL-BE6500 在 OpenWrt 环境下的区域识别码修改、OpenClash 部署以及家庭网络优化过程。

---

## 项目简介

GL-BE6500 是基于 Qualcomm IPQ5332 平台的 Wi-Fi 7 路由器。本项目基于实际设备测试，整理：

- 区域识别码修改与 ART 分区备份
- OpenWrt 环境研究
- OpenClash + Mihomo Meta Core 部署
- Clash/Mihomo 订阅接入
- 家庭网络透明代理优化
- 常见问题排查

## 已验证环境

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

```text
docs/GL-BE6500区域识别码修改指南.md
```

包含：

- country_code 定位
- ART 分区备份
- CN/US 区域修改
- 写入验证
- 恢复方法

⚠️ 修改 ART 分区前必须备份。

---

## 2. OpenClash 部署

```text
docs/OpenClash-GL-BE6500-Install.md
```

方案：

```text
家庭设备
    |
GL-BE6500
    |
OpenClash
    |
Mihomo Meta Core
    |
Clash/Mihomo订阅
```

特点：

- 路由器统一代理
- 手机、电脑、电视、IoT无需单独配置
- 支持新版 Clash 订阅格式
- 支持 Fake-IP

---

## 3. OpenClash 高级优化

```text
docs/OpenClash-Advanced-Optimization.md
```

包含：

- DNS 优化
- Fake-IP 模式
- TUN 模式说明
- 游戏设备分流
- 节点优化
- 故障排查

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

---

# 常用命令

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

1. ART 分区包含设备关键校准信息，修改前必须备份。
2. 不同 GL-BE6500 固件版本可能存在差异。
3. OpenClash 首次启动需要初始化 DNS、规则和透明代理规则。
4. 不建议同时启用多个 DNS 接管服务。

---

# License

本文档用于技术研究与设备配置记录。