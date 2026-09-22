# LibWrt-JDC-RE-SS-01-CI

[![Build LiBwrt OpenWrt](https://github.com/EmptyGhost/LibWrt-JDC-RE-SS-01-CI/actions/workflows/build-openwrt.yml/badge.svg)](https://github.com/EmptyGhost/LibWrt-JDC-RE-SS-01-CI/actions/workflows/build-openwrt.yml)

京东云 RE-SS-01 路由器的 LiBwrt 自动编译仓库。GitHub Actions 以仓库根目录的 `.config` 作为固定配置，从上游拉取最新源码编译，并将生成的固件发布到 Release。默认每两周自动编译一次，每月约 2–3 次。

## 工作方式

| 触发方式 | 条件 | 说明 |
| --- | --- | --- |
| 定时触发（自动编译） | 每两周一次，按周一 00:00 UTC 执行 | 每月约 2–3 次，避免高频编译耗尽 Actions 额度 |
| Push | 修改根目录 `.config` 或本工作流并推送到 `main` | 立即额外编译一次，不改变自动编译周期 |
| 手动触发 | Actions → Build LiBwrt OpenWrt → Run workflow | 立即额外编译一次，可勾选 SSH 进入调试会话 |

自动编译采用“每周一检查、隔周执行”的方式实现：GitHub Actions 只支持 cron 表达式，无法直接表达“每 14 天”，因此工作流在 `prepare` job 中判断当前属于单周还是双周，仅在一个周期内实际编译。

当前编译目标：

- 上游源码：`https://github.com/LiBwrt/LibWrt.git`，分支 `25.12-nss`
- 目标平台：`qualcommax` / `ipq60xx`
- 设备配置：`DEVICE_jdcloud_re-ss-01`
- Release 标签：`LiBwrt-OpenWrt`

## Release 行为（预期）

项目只占用一条 Release 条目。第一次编译成功时创建 `LiBwrt-OpenWrt`；之后每次编译成功都更新同一条目的名称、说明和固件文件，不会新增条目。下载固件时始终查看这个条目即可。

## 使用方法

1. 在项目根目录上传或修改 `.config`。
2. 提交并推送到 `main`，或在 Actions 页面手动运行工作流。
3. 编译完成后在 Actions 查看日志，在 Releases 下载固件。

建议在上游 LiBwrt 源码目录中生成配置，再复制回本仓库：

```sh
make menuconfig
make defconfig
cp .config /path/to/LibWrt-JDC-RE-SS-01-CI/.config
```

## 常用调整

在 `.github/workflows/build-openwrt.yml` 中修改：

| 变量 | 作用 |
| --- | --- |
| `REPO_URL`、`REPO_BRANCH` | 上游源码地址与分支 |
| `FIRMWARE_TAG` | Release 标签；改成新值会创建新的 Release 条目 |
| `FIRMWARE_RELEASE` | 设为 `false` 时只上传 Actions artifact |
| `CACHE_TOOLCHAIN` | 是否缓存工具链和编译输出 |
| `cron` | 定时检查周期；当前为每周一检查，配合隔周判断实现每两周自动编译 |

改为每天编译：把 `cron` 改成 `0 0 * * *`，并移除 `prepare` job 中的隔周判断。修改自动编译频率前，请先确认 runner 额度充足；完整编译通常需要数小时。

## 注意事项

- `.config` 必须能被上游 Kconfig 解析；工作流会执行 `make defconfig` 自动补全依赖。
- Release 说明中的默认地址为 `192.168.1.1`，默认密码为空。
- 定时任务可能因 GitHub 负载而延迟；仓库连续 60 天无活动时 schedule 会被自动停用，需要手动运行一次或重新启用。
- 不要提交 token、私钥或其他凭据；发布使用内置的 `GITHUB_TOKEN`。
