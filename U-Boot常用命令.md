# U-Boot 常用命令

> 对应课程：第3讲 Uboot命令使用。U-Boot 的 `bootdelay` 倒计时结束前按回车进入命令行。

## 一、帮助

| 命令 | 作用 | 示例 |
|------|------|------|
| `help` / `?` | 查看命令帮助 | `help mmc`、`? mmc` |

## 二、信息查询

| 命令 | 作用 |
|------|------|
| `bdinfo` | 查看板子信息（DRAM 起始地址 0x80000000、大小等） |
| `printenv` | 查看当前所有环境变量（**重要**） |
| `version` | 查看 U-Boot 版本、编译器信息 |

## 三、环境变量操作

| 命令 | 作用 | 示例 |
|------|------|------|
| `setenv` | 设置 / 新建 / 删除环境变量 | `setenv bootdelay 5` |
| `saveenv` | 保存环境变量（写入 EMMC/NAND，掉电不丢） | `saveenv` |

```sh
setenv myvar 123        # 新建 myvar = 123
setenv myvar            # 什么都不跟 → 删除 myvar
setenv bootdelay 5      # 修改 bootdelay
saveenv                 # 保存，否则重启就没了
```

## 四、内存操作

> 单位后缀：`.b`=1字节 `.w`=2字节 `.l`=4字节（默认 `.l`）

| 命令 | 作用 | 示例 |
|------|------|------|
| `md` | 显示内存值 | `md.b 80000000 10` |
| `nm` | 修改内存（地址**不**自增） | `nm.l 80000000` |
| `mm` | 修改内存（地址自增） | `mm.l 80000000` |
| `mw` | 填充内存 | `mw.b 80000000 FF 10` |
| `cp` | 拷贝内存 | `cp.b 80000000 80010000 100` |
| `cmp` | 比较两段内存 | `cmp.b 80000000 80010000 10` |

## 五、网络操作

**前提**：网线插到板子 **ENET2**，与电脑（Ubuntu）在同一网段。

```sh
setenv ipaddr    192.168.1.50      # 板子 IP
setenv ethaddr   b8:ae:1d:01:02:03 # MAC 地址（第一次必须设置！）
setenv gatewayip 192.168.1.1
setenv netmask   255.255.255.0
setenv serverip  192.168.1.100     # Ubuntu 的 IP
saveenv
```

| 命令 | 作用 | 示例 |
|------|------|------|
| `ping` | 测试与 Ubuntu 是否连通（**重点**） | `ping 192.168.1.100` |
| `dhcp` | 从路由器自动获取 IP | `dhcp` |
| `nfs` | 从 Ubuntu 的 NFS 服务器下载文件（**重点**，调试用） | `nfs 80800000 192.168.1.100:/home/xxx/zImage` |
| `tftp` | 从 Ubuntu 的 TFTP 服务器下载文件（**重点**） | `tftp 80800000 zImage` |

## 六、EMMC / SD 卡操作

| 命令 | 作用 | 示例 |
|------|------|------|
| `mmc list` | 列出所有 MMC 设备 | `mmc list` |
| `mmc dev` | 切换设备（0=SD 卡，1=EMMC） | `mmc dev 1` |
| `mmc info` | 显示当前设备信息（容量等） | `mmc info` |
| `mmc part` | 显示分区表 | `mmc part` |
| `mmc read` | 读（blk 单位 512 字节） | `mmc read 80800000 600 10` |
| `mmc write` | 写 | `mmc write 80800000 600 10` |
| `mmc erase` | 擦除 —— **千万不要用**（会误擦数据） | ✗ |

## 七、文件系统操作

> I.MX6U 的 SD/EMMC 通常分三个区：① U-Boot ② zImage + dtb（**FAT**）③ 根文件系统（**EXT4**）

| 命令 | 作用 | 示例 |
|------|------|------|
| `fatinfo` | 查看 FAT 分区信息 | `fatinfo mmc 1:1` |
| `fatls` | 列出 FAT 分区目录 | `fatls mmc 1:1` |
| `fstype` | 查看分区文件系统类型 | `fstype mmc 1:1` |
| `fatload` | FAT 文件 → DRAM | `fatload mmc 1:1 80800000 zImage` |
| `fatwrite` | DRAM → FAT 文件 | `fatwrite mmc 1:1 80800000 zImage $filesize` |
| `ext4ls` | 列出 EXT4 分区目录 | `ext4ls mmc 1:2` |
| `ext4load` | EXT4 文件 → DRAM | `ext4load mmc 1:2 80800000 xxx` |

`mmc 1:1` 指 mmc 设备 1（EMMC）的**分区 1**；`mmc 1:2` 即分区 2。

## 八、NAND 操作

| 命令 | 作用 | 示例 |
|------|------|------|
| `nand info` | 查看 NAND 信息 | `nand info` |
| `nand erase` | 擦除（**写之前必须先擦**） | `nand erase 0x200000 0x200000` |
| `nand write` | 写 | `nand write 80800000 0x200000 0x200000` |
| `nand read` | 读 | `nand read 80800000 0x200000 0x200000` |

## 九、BOOT 操作

**启动 Linux 三要素**：zImage + 设备树 dtb + 根文件系统；zImage 和 dtb 必须先放到 DRAM（如 80800000）。

| 命令 | 作用 | 示例 |
|------|------|------|
| `bootz` | 启动 zImage | `bootz 80800000 - 83000000` |
| `bootm` | 启动 uImage | `bootm 80800000` |
| `boot` | 执行 `bootcmd`（自动启动） | `boot` |

关键环境变量：

```sh
# 内核传参：串口控制台 + 根文件系统位置
setenv bootargs 'console=ttymxc0,115200 root=/dev/mmcblk1p2 rootwait rw'

# 自动启动：从 EMMC 分区1加载 zImage 到 80800000、dtb 到 83000000，然后 bootz
setenv bootcmd 'fatload mmc 1:1 80800000 zImage; fatload mmc 1:1 83000000 imx6ull-alientek-emmc.dtb; bootz 80800000 - 83000000'
saveenv
```

## 十、其他命令

| 命令 | 作用 | 示例 |
|------|------|------|
| `reset` | 复位（重启） | `reset` |
| `go` | 跳转到指定地址执行 | `go 87800000` |
| `run` | 执行环境变量里的命令序列 | `run bootcmd` |
| `mtest` | 内存读写测试 | `mtest 80000000 800fffff` |

## 相关笔记

- [U-Boot介绍](U-Boot介绍.md) ｜ [U-Boot编译](U-Boot编译.md) ｜ [系统烧录](系统烧录.md)
