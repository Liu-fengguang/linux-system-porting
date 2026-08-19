# 正点原子官方 U-Boot 编译

## 一、编译流程

```
配置（选板级配置）
  → 编译
  → 生成 u-boot.bin
  → 添加头部信息（mkimage）
  → 生成 u-boot.imx ← 烧录这个
```

| 步骤 | 命令 | 产物 |
|------|------|------|
| ① 配置 | `make ARCH=arm CROSS_COMPILE=... mx6ull_alientek_emmc_defconfig` | `.config` 配置文件 |
| ② 编译 | `make -j4` | `u-boot.bin` |
| ③ 加头部 | 编译时自动调用 `tools/mkimage` | `u-boot.imx` |

---

## 二、为什么要添加头部信息

和裸机实验的 `led.bin → load.imx` 逻辑一样：

```
u-boot.bin（纯代码，Boot ROM 不认识）
    │
  tools/mkimage 添加 IVT/DCD 头部
    │
u-boot.imx（带启动头的完整固件）
```

> i.MX6ULL 的 Boot ROM 需要从 SD 卡第 1024 字节处读到 IVT 头部，才能初始化 DDR 并加载代码。U-Boot 编译流程自动完成这一步，产物是 `u-boot.imx` 而非 `u-boot.bin`。

---

## 三、⚠️ 配置丢失陷阱

**执行清理命令会连配置一起删掉。**

| 命令 | 清什么 | 后果 |
|------|--------|------|
| `make clean` | 编译产物（.o、.bin） | 配置还在 |
| `make distclean` | **一切**，包括 `.config` | 配置全没，要重新 `make xxx_defconfig` |

> 教程提供的 shell 脚本如果执行了 `distclean`，你的板级配置（`mx6ull_alientek_emmc_defconfig` 生成的 `.config`）会被删除，必须重新配置才能编译。

---

## 四、配置 ARCH 和 CROSS_COMPILE

每次敲 `make` 都带一长串变量很麻烦。直接写进顶层 Makefile：

```makefile
# U-Boot 顶层 Makefile 开头
ARCH = arm
CROSS_COMPILE = arm-linux-gnueabihf-
```

之后编译只需要：

```bash
make mx6ull_alientek_emmc_defconfig   # 配置
make -j4                              # 编译，自动带上 ARCH 和交叉编译器
```

| 变量 | 含义 |
|------|------|
| `ARCH` | 目标架构（arm） |
| `CROSS_COMPILE` | 交叉编译器前缀 |

---

## 五、完整实操流程

```bash
# 1. 解压源码
tar -vxjf uboot-imx-rel_imx_4.1.15_2.1.0_ga_alientek.tar.bz2

# 2. 清理（可选，首次不用）
make distclean

# 3. 配置板级
make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- \
     mx6ull_alientek_emmc_defconfig

# 4. 编译（4 线程加速）
make -j4

# 5. 检查产物
ls u-boot.bin u-boot.imx
```

烧录到 SD 卡：

```bash
sudo dd if=u-boot.imx of=/dev/rdiskX bs=512 seek=2 conv=sync
```

> 和裸机实验烧 load.imx 一模一样：跳过前 1024 字节（2 扇区），从 Boot ROM 约定位置写入。
