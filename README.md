# GL-BE6500 区域识别码修改指南

> 本文记录在 GL.iNet GL-BE6500（Qualcomm IPQ5332）上，将设备区域识别码从 `CN` 修改为 `US` 的完整过程。

## 重要警告

此操作会写入路由器的 ART 分区。

ART 分区通常包含：

- 无线校准数据
- 设备序列号
- MAC 地址
- 区域识别码
- 其他出厂参数

操作失误可能导致：

- Wi-Fi 无法启动
- 无线功率异常
- MAC 地址丢失
- 路由器无法正常启动
- 需要串口、U-Boot 或硬件编程器恢复

请务必：

1. 使用有线网络连接路由器。
2. 保证路由器供电稳定。
3. 在写入前完整备份 ART 分区。
4. 将备份下载到电脑并校验。
5. 确认修改后的镜像仅有两个字节发生变化。
6. 不要直接照搬其他型号的分区号或偏移量。

本文实测环境：

```text
型号：GL-BE6500
SoC：Qualcomm IPQ5332
board_name：qcom,ipq5332-ap-mi01.2
OpenWrt：23.05-SNAPSHOT
Kernel：5.4.213
```

本方法仅适用于设备树中明确显示：

```text
分区：0:ART
偏移：0x88
```

的设备。

---

## 一、区域识别码与无线法规域的区别

### 1. 区域识别码

设备区域识别码用于决定固件将路由器识别为哪个销售区域版本。

读取命令：

```bash
cat /proc/gl-hw-info/country_code
```

可能输出：

```text
CN
```

区域识别码可能影响：

- 国际版或中国版固件功能
- 管理后台菜单
- 某些服务或插件
- 固件初始化逻辑
- 区域相关脚本分支

### 2. 无线法规域

无线法规域用于控制 Wi-Fi：

- 可用信道
- 发射功率
- DFS 规则
- 频宽
- 2.4 GHz、5 GHz、6 GHz 的使用限制

查看命令：

```bash
iw reg get
```

UCI 配置示例：

```bash
uci show wireless | grep country
```

修改法规域：

```bash
uci set wireless.wifi0.country='US'
uci set wireless.wifi1.country='US'
uci commit wireless
wifi reload
```

修改无线法规域并不一定会改变 GL.iNet 固件的设备区域识别。

---

## 二、通过 SSH 登录

默认管理地址通常为：

```text
192.168.8.1
```

SSH 用户名：

```text
root
```

密码为路由器后台管理员密码。

登录命令：

```bash
ssh root@192.168.8.1
```

---

## 三、确认硬件平台

执行：

```bash
ubus call system board
```

实测输出关键字段：

```json
{
  "kernel": "5.4.213",
  "hostname": "GL-BE6500",
  "system": "ARMv8 Processor rev 4",
  "model": "Qualcomm Technologies, Inc. IPQ5332/AP-MI01.2",
  "board_name": "qcom,ipq5332-ap-mi01.2",
  "rootfs_type": "squashfs",
  "release": {
    "distribution": "OpenWrt",
    "version": "23.05-SNAPSHOT",
    "target": "ipq53xx/generic"
  }
}
```

---

## 四、确认设备区域识别码

执行：

```bash
cat /proc/gl-hw-info/country_code
```

修改前输出：

```text
CN
```

查看硬件信息模块：

```bash
lsmod | grep -iE 'gl|hw|info'
```

关键模块：

```text
gl_sdk4_hw_info
gl_sdk4_mtd_rw
```

---

## 五、定位区域识别码所在分区和偏移

查找设备树节点：

```bash
find /sys/firmware/devicetree/base -type d -name 'factory_data'
```

实测结果：

```text
/sys/firmware/devicetree/base/gl-hw/factory_data
```

读取区域码定义：

```bash
hexdump -C /sys/firmware/devicetree/base/gl-hw/factory_data/country_code
```

输出：

```text
00000000  30 3a 41 52 54 00 30 78  38 38 00  |0:ART.0x88.|
```

转换为字符串：

```bash
tr '\0' '\n' < /sys/firmware/devicetree/base/gl-hw/factory_data/country_code
```

输出：

```text
0:ART
0x88
```

说明：

```text
分区：0:ART
偏移：0x88
```

---

## 六、确认 MTD 分区

执行：

```bash
cat /proc/mtd
```

实测 ART 分区：

```text
mtd11: 00200000 00040000 "0:ART"
```

其中：

- 分区大小：2 MB
- 擦除块大小：256 KB
- 设备节点：`/dev/mtd11`
- 块设备：`/dev/mtdblock11`

---

## 七、备份 ART 分区

执行：

```bash
dd if=/dev/mtd11 of=/tmp/ART-backup.bin
```

确认大小：

```bash
ls -lh /tmp/ART-backup.bin
```

应为：

```text
2.0M
```

计算 SHA-256：

```bash
sha256sum /tmp/ART-backup.bin
```

实测原始备份校验值：

```text
f51ab096c7288b87e2f4ba9e29a66eb3357ecc97473f78437bb12274a24820d4
```

请通过 MobaXterm 左侧文件浏览器，将：

```text
/tmp/ART-backup.bin
```

下载到电脑。

在电脑端再次执行：

```bash
sha256sum ART-backup.bin
```

必须与路由器端校验值完全一致。

---

## 八、查看区域码原始字节

执行：

```bash
hexdump -C -s 0x70 -n 64 /tmp/ART-backup.bin
```

实测输出：

```text
00000070  47 4c 2d 32 36 30 33 34  33 31 30 30 30 36 31 ff
00000080  ff ff ff ff ff ff ff ff  43 4e ff ff ff ff ff ff
00000090  ff ff ff ff ff ff ff ff  ff ff ff ff ff ff ff ff
```

其中：

```text
0x88 = 43
0x89 = 4e
```

ASCII：

```text
43 4e = CN
```

---

## 九、生成修改后的 ART 镜像

先复制备份：

```bash
cp /tmp/ART-backup.bin /tmp/ART-US.bin
```

将偏移 `0x88` 的两个字节修改为 `US`：

```bash
printf 'US' | dd of=/tmp/ART-US.bin bs=1 seek=$((0x88)) conv=notrunc
```

检查结果：

```bash
hexdump -C -s 0x80 -n 32 /tmp/ART-US.bin
```

应显示：

```text
00000080  ff ff ff ff ff ff ff ff  55 53 ff ff ff ff ff ff
```

ASCII：

```text
55 53 = US
```

---

## 十、确认仅修改两个字节

执行：

```bash
cmp -l /tmp/ART-backup.bin /tmp/ART-US.bin
```

实测输出：

```text
137 103 125
138 116 123
```

说明：

- `cmp` 的位置从 1 开始
- 第 137、138 字节对应文件偏移 `0x88`、`0x89`
- 八进制 `103 116` 对应 `CN`
- 八进制 `125 123` 对应 `US`

除此之外不应出现任何其他差异。

检查镜像大小：

```bash
ls -l /tmp/ART-US.bin
```

必须仍为：

```text
2097152
```

---

## 十一、写入前校验原始分区

确认 `mtd` 工具存在：

```bash
which mtd
```

实测路径：

```text
/sbin/mtd
```

先验证原始备份与当前 ART 分区一致：

```bash
mtd verify /tmp/ART-backup.bin /dev/mtd11
```

成功时显示：

```text
Success
```

---

## 十二、正式写入修改后的 ART 镜像

确保：

- 路由器使用稳定电源
- 电脑通过网线连接
- 原始 ART 已下载到电脑
- 原始备份 SHA-256 已校验
- 修改镜像只改变两个字节

执行：

```bash
mtd write /tmp/ART-US.bin /dev/mtd11
```

正常输出类似：

```text
Unlocking /dev/mtd11 ...

Writing from /tmp/ART-US.bin to /dev/mtd11 ...
```

不要在写入过程中断电或重启。

---

## 十三、写入后验证

执行：

```bash
mtd verify /tmp/ART-US.bin /dev/mtd11
```

应显示：

```text
Success
```

再读取真实分区：

```bash
hexdump -C -s 0x80 -n 32 /dev/mtd11
```

应看到：

```text
00000080  ff ff ff ff ff ff ff ff  55 53 ff ff ff ff ff ff
```

---

## 十四、保存持久备份

`/tmp` 会在重启后清空，因此建议将原始备份复制到 `/root`：

```bash
cp /tmp/ART-backup.bin /root/ART-backup.bin
```

校验：

```bash
sha256sum /root/ART-backup.bin
```

应与电脑保存的原始备份一致。

---

## 十五、重启并确认

执行：

```bash
reboot
```

重新登录后：

```bash
cat /proc/gl-hw-info/country_code
```

成功后输出：

```text
US
```

---

## 十六、恢复为 CN

如果设备仍可正常启动并进入 SSH：

```bash
mtd write /root/ART-backup.bin /dev/mtd11
```

校验：

```bash
mtd verify /root/ART-backup.bin /dev/mtd11
```

成功后重启：

```bash
reboot
```

确认：

```bash
cat /proc/gl-hw-info/country_code
```

应恢复为：

```text
CN
```

---

## 十七、建议的组合配置

如果修改区域识别码的目的只是解锁国际版固件功能，而设备实际在中国使用，可采用：

```text
设备区域识别码：US
无线法规域：CN
```

恢复无线法规域为中国：

```bash
uci set wireless.wifi0.country='CN'
uci set wireless.wifi1.country='CN'
uci commit wireless
wifi reload
```

检查：

```bash
cat /proc/gl-hw-info/country_code
iw reg get
```

这样可以使：

- 固件按国际版区域识别
- Wi-Fi 按实际所在地法规运行

---

## 十八、使用 Windows PowerShell 操作

Windows 10/11 通常已经内置 OpenSSH 客户端，可以直接通过 PowerShell 登录路由器、下载备份并校验文件。

### 1. 检查 SSH 和 SCP 是否可用

打开 PowerShell，执行：

```powershell
ssh -V
scp
```

如果 `ssh` 无法识别，可在 Windows 的“可选功能”中安装 **OpenSSH 客户端**。

### 2. SSH 登录路由器

执行：

```powershell
ssh root@192.168.8.1
```

首次连接时可能提示：

```text
Are you sure you want to continue connecting (yes/no/[fingerprint])?
```

输入：

```text
yes
```

然后输入 GL.iNet 后台管理员密码。输入密码时不会显示字符，这是正常现象。

### 3. 在路由器端生成 ART 备份

登录路由器后执行：

```bash
dd if=/dev/mtd11 of=/tmp/ART-backup.bin
sha256sum /tmp/ART-backup.bin
```

记录路由器端的 SHA-256 值。

### 4. 使用 SCP 下载 ART 备份

退出 SSH，或另开一个 PowerShell 窗口。

切换到希望保存备份的目录，例如：

```powershell
New-Item -ItemType Directory -Force "$HOME\Documents\GL-BE6500-Backup"
Set-Location "$HOME\Documents\GL-BE6500-Backup"
```

下载备份：

```powershell
scp root@192.168.8.1:/tmp/ART-backup.bin .
```

下载后确认文件存在：

```powershell
Get-Item .\ART-backup.bin
```

### 5. 使用 PowerShell 校验 SHA-256

执行：

```powershell
(Get-FileHash .\ART-backup.bin -Algorithm SHA256).Hash.ToLower()
```

输出必须与路由器端：

```bash
sha256sum /tmp/ART-backup.bin
```

得到的值完全一致。

本次实测原始备份值为：

```text
f51ab096c7288b87e2f4ba9e29a66eb3357ecc97473f78437bb12274a24820d4
```

> 不同设备的 ART 内容可能不同，不应把本文的校验值作为其他设备的标准值。应以自己路由器生成的备份为准。

### 6. 同时保存多份备份

建议复制两份并使用明确文件名：

```powershell
Copy-Item .\ART-backup.bin .\ART-backup-original.bin
Copy-Item .\ART-backup.bin .\ART-backup-restore.bin
```

再次校验：

```powershell
Get-FileHash .\ART-backup*.bin -Algorithm SHA256 |
    Select-Object Path, Hash
```

建议再将原始备份复制到 U 盘或其他电脑。

### 7. 从电脑重新上传 ART 备份

需要恢复时，可以把电脑中的原始镜像上传到路由器 `/tmp`：

```powershell
scp .\ART-backup-original.bin root@192.168.8.1:/tmp/ART-backup-original.bin
```

登录路由器：

```powershell
ssh root@192.168.8.1
```

在路由器中校验上传后的文件：

```bash
sha256sum /tmp/ART-backup-original.bin
```

必须与电脑端一致。

### 8. 从上传的备份恢复 ART

在路由器 SSH 终端中执行：

```bash
mtd verify /tmp/ART-backup-original.bin /dev/mtd11
```

确认显示：

```text
Success
```

然后写回：

```bash
mtd write /tmp/ART-backup-original.bin /dev/mtd11
```

写入后再次校验：

```bash
mtd verify /tmp/ART-backup-original.bin /dev/mtd11
```

显示 `Success` 后重启：

```bash
reboot
```

### 9. PowerShell 一次性下载与校验示例

下面的命令会创建备份目录、下载文件并显示校验值：

```powershell
$BackupDir = "$HOME\Documents\GL-BE6500-Backup"
New-Item -ItemType Directory -Force $BackupDir | Out-Null
Set-Location $BackupDir

scp root@192.168.8.1:/tmp/ART-backup.bin .

if (-not (Test-Path .\ART-backup.bin)) {
    throw "ART-backup.bin 下载失败。"
}

$Hash = (Get-FileHash .\ART-backup.bin -Algorithm SHA256).Hash.ToLower()
Write-Host "ART 备份路径：$((Get-Item .\ART-backup.bin).FullName)"
Write-Host "SHA-256：$Hash"
```

### 10. 可选：在 PowerShell 中修改本地副本

更稳妥的做法仍然是在路由器 `/tmp` 中使用经过验证的 `dd` 命令生成修改镜像。如果需要在 Windows PowerShell 中修改副本，可使用 .NET 字节数组：

```powershell
$Source = ".\ART-backup-original.bin"
$Target = ".\ART-US.bin"

[byte[]]$Data = [System.IO.File]::ReadAllBytes($Source)

if ($Data.Length -ne 2097152) {
    throw "镜像大小不是 2 MB，停止操作。"
}

if ($Data[0x88] -ne 0x43 -or $Data[0x89] -ne 0x4E) {
    throw "偏移 0x88 处不是 CN，停止操作。"
}

$Data[0x88] = 0x55
$Data[0x89] = 0x53

[System.IO.File]::WriteAllBytes($Target, $Data)

Get-Item $Target
Get-FileHash $Target -Algorithm SHA256
```

该脚本会：

- 检查镜像是否为 2 MB
- 检查偏移 `0x88` 是否为 `CN`
- 将 `CN` 改为 `US`
- 保存为新的 `ART-US.bin`
- 不修改原始备份

比较两个文件有多少字节不同：

```powershell
[byte[]]$Original = [System.IO.File]::ReadAllBytes(".\ART-backup-original.bin")
[byte[]]$Modified = [System.IO.File]::ReadAllBytes(".\ART-US.bin")

$Diff = for ($i = 0; $i -lt $Original.Length; $i++) {
    if ($Original[$i] -ne $Modified[$i]) {
        [PSCustomObject]@{
            OffsetHex = ('0x{0:X}' -f $i)
            Original  = ('0x{0:X2}' -f $Original[$i])
            Modified  = ('0x{0:X2}' -f $Modified[$i])
        }
    }
}

$Diff
```

正常只能输出：

```text
OffsetHex Original Modified
--------- -------- --------
0x88      0x43     0x55
0x89      0x4E     0x53
```

### 11. 将修改镜像上传到路由器

```powershell
scp .\ART-US.bin root@192.168.8.1:/tmp/ART-US.bin
```

然后登录路由器，必须先检查文件大小和哈希：

```bash
ls -l /tmp/ART-US.bin
sha256sum /tmp/ART-US.bin
```

再与电脑端 PowerShell 结果比较：

```powershell
(Get-FileHash .\ART-US.bin -Algorithm SHA256).Hash.ToLower()
```

两端完全一致后，才能考虑执行 MTD 写入。

---

## 十九、恢复与救砖思路

### 1. 系统可以启动

通过 SSH 写回原始 ART：

```bash
mtd write /root/ART-backup.bin /dev/mtd11
mtd verify /root/ART-backup.bin /dev/mtd11
reboot
```

### 2. 系统可以启动但管理界面异常

尝试恢复出厂设置。

注意：恢复出厂设置通常不会恢复 ART 分区内容。

### 3. 系统固件损坏

尝试进入 GL.iNet U-Boot Web 恢复模式，重新刷写官方固件。

### 4. 无法进入系统但可进入 U-Boot

可通过：

- TTL 串口
- TFTP
- USB
- U-Boot NAND 命令

写回 ART 分区。

具体命令必须根据该设备 U-Boot 支持情况确定，不能照搬其他型号。

### 5. U-Boot 也无法进入

可能需要：

- Qualcomm 恢复接口
- 外置 NAND 编程器
- 拆机写入
- 官方售后

---

## 二十、免责声明

本文仅记录一次真实设备上的技术验证过程。

执行者需自行承担：

- 保修失效
- 设备损坏
- 数据丢失
- 无线异常
- 法规合规风险

不要在没有完整 ART 备份的情况下执行写入。

不要将其他型号的分区号、偏移量或写入命令直接用于 GL-BE6500。

区域识别码与无线法规域应根据实际用途和所在地法律法规谨慎配置。
