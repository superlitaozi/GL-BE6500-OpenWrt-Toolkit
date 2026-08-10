# GL-BE6500 OpenClash 高级优化指南

本文基于 GL-BE6500 + OpenClash + Mihomo 的家庭网络方案。

## 一、推荐稳定配置

推荐长期运行：

```
代理模式：Rule
运行模式：增强模式（Fake-IP）
TUN：关闭
IPv6：关闭或根据实际测试开启
DNS：优先使用订阅配置
```

原因：

- Clash Verge 使用的也是 Mihomo 生态
- 服务商提供的 YAML 通常已经包含 DNS、规则和策略组
- 路由器端不需要额外覆盖配置

## 二、DNS 调整原则

不要同时启用多个 DNS 接管：

避免：

```
GL.iNet DNS
+
AdGuard Home
+
OpenClash DNS
+
浏览器安全 DNS
```

否则容易出现：

- 国内网站打不开
- Fake-IP异常
- 节点域名无法解析

## 三、TUN 模式建议

家庭路由场景通常不需要 TUN。

OpenClash 透明代理已经可以处理：

- HTTP
- HTTPS
- TCP

只有以下情况考虑 TUN：

- 某些应用绕过代理
- UDP代理需求
- 游戏特殊需求

## 四、游戏设备建议

建议：

```
游戏主机/IoT
→ DIRECT
```

避免：

- 延迟增加
- NAT异常
- 登录服务器失败

可通过规则增加：

```
IP-CIDR
DOMAIN-SUFFIX
PROCESS-NAME
```

## 五、节点优化

建议保留：

- 香港节点（日常）
- 日本节点（亚洲服务）
- 美国节点（Google/GitHub等）

不要长期使用自动选择作为唯一节点。

## 六、故障排查

查看核心：

```bash
/etc/openclash/core/clash_meta -v
```

查看日志：

```bash
tail -n 100 /tmp/openclash.log
```

检查端口：

```bash
netstat -lnp | grep clash
```

## 七、最终家庭网络结构

```
Internet
 |
GL-BE6500
 |
OpenClash
 |
Mihomo
 |
VPN节点
 |
家庭全部设备
```

客户端无需安装代理软件。