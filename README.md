## pandora fork

这是 [`lolipuru/android_device_xiaomi_sm8850-common`](https://github.com/lolipuru/android_device_xiaomi_sm8850-common)
（分支 `lineage-24.0`）的下游副本，为 **Xiaomi 17 Pro（`pandora` / `canoe` / SM8850）** 适配。

| 项 | 值 |
|---|---|
| 上游 | `https://github.com/lolipuru/android_device_xiaomi_sm8850-common` |
| 上游分支 | `lineage-24.0` |
| 取证时上游 HEAD | `6b1fa621c5daabf5be3ac02c25576462b1ba8f43`（2026-10-31，168 commits） |
| 本副本快照时间 | 2026-10-30 23:36（本地） |
| 与上游的差异 | **未逐条比对**。本副本可能落后上游 1 个提交，且下方代码无改动痕迹可逐条呈现 |
| 上游支持的机型 | Pudding / Popsicle / Myron / Athens / Songyuan —— **不含 pandora** |

### 本副本的定位

- pandora 是**下游新增机型**，上游不支持，不要把本仓的问题报给上游。
- 文件头（`SPDX-License-Identifier: Apache-2.0` / `Copyright (C) 2026 The LineageOS Project`）
  是上游自己的，**未被修改**。仓库级 `LICENSE` 与 `NOTICE` 为新增，用于满足 Apache-2.0 §4(a)/(d)。
- 已知遗留：`interfaces/citsensors/Android.bp.orig` 是快照中自带的备份文件，未删除。

### 已知问题（照此构建不会得到能开机的包）

1. **pandora 机型 blobs 尚不存在** —— 机型仓 `LiKgDg/android_device_xiaomi_pandora` 里的
   `vendor/xiaomi/pandora` 从未生成，构建时会被 `inherit-product-if-exists` / `-include` 静默跳过。
2. **本仓 `proprietary-files.txt` 有 211/1758（12.0%）条目在 blobs 实体中找不到对应文件**
   （vendor 118 / odm 89 / system_ext 2 / system 1 / mi_ext 1）；其中 108 条是"父目录在、文件缺"。
   上游清单覆盖 5 台机，部分条目本就不属于 pandora，但 108 条说明抽取过程本身有遗漏。
3. **裸 clone 编不过** —— 需要 6 个树外补丁（`fix_ipa_headers` / `fix_blob_deps` /
   `fix_wfd_elfcheck` / `fix_qcwcn_common` / `fix_rildb_filegroups` / `fix_sepolicy_compat`），
   都在主仓 `lineage-os-pandora` 的 `scripts/maintain/` 下。其中最典型的是预编译内核没有
   `kernel-headers/`，导致 `linux/msm_ipa.h` 找不到。
4. **`configs/vintf/manifest_canoe.xml` 是空壳**（2 行、0 个 HAL），本设备实际 HAL 声明来自
   blobs 自带的 `vendor/etc/vintf/manifest/*.xml`（100 个）与 `odm/etc/vintf/manifest/*.xml`（6 个）。
5. **`BOARD_DTBOIMG_PARTITION_SIZE := 23068672`** 取的是 stock `dtbo.img` 文件大小，
   而 stock `dtbo` 分区是 32 MiB —— dtbo 重新生成超 22 MiB 会直接构建失败（零余量）。
6. **`configs/modules/modules.{load,load.recovery,load.vendor_boot,blocklist}` 在预编译内核路线下未生效**
   （`BoardConfigCommon.mk` 里这段在 `ifeq ($(TARGET_OPT_IN_OSS_KERNEL),true)` 内），
   真正生效的是 `pandora-kernel` 仓里的 `modules.load`。改配置时别改错文件。
7. **AVB 使用 AOSP 测试 key**（`external/avb/test/data/testkey_rsa4096.pem`）—— 只能自签自用，不能与 stock 互刷。

启动状态：**未验证**（最近一次刷入停在开机动画，根因定位为内核模块 KMI 世代错配，
修复为二进制级整目录替换，替换后未再验证）。

---

#  Common device configuration for Xiaomi 8850 family

Currently supported devices
- Xiaomi 17 (Pudding)
- Xiaomi 17 Pro Max (Popsicle)
- Redmi K90 Pro Max / Poco F8 Ultra (Myron)
- Redmi K100 Pro (Athens)
- Redmi K100 Pro Max (Songyuan)
