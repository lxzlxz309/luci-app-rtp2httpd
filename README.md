# luci-app-rtp2httpd（LuCI 界面，r3 修复版）

rtp2httpd 的 LuCI 管理界面（基于上游 3.17.2 UI 的定制修复版），
提供 **服务 → rtp2httpd** 配置页：实例管理、监听地址、组播上游、rejoin 间隔、
access_log 等开关。

> ✅ 已在 ImmortalWrt 25.x（aarch64_cortex-a53）实测装机验收，
> 产物 `luci-app-rtp2httpd-3.17.2-r3.apk`。

## 与上游的差异

| 修改 | 说明 |
| --- | --- |
| TypedSection 实例类型修复 | `TypedSection,"instance"` + `anonymous=false` + `addremove=false`，修复"尚无任何配置"——类型不匹配导致命名 section 被过滤 |
| openPage 通配 host 修复 | 监听地址为 `0.0.0.0` / `::` / `[::]` 时置 null 回退到当前 hostname，修复打开的播放器/状态页指向 `http://0.0.0.0:56309/...` 的问题 |

主程序本体（含 010 会话结束日志补丁）见独立仓库
[lxzlxz309/rtp2httpd](https://github.com/lxzlxz309/rtp2httpd)（`LUCI_DEPENDS` 依赖它，需一并安装）。

## 仓库结构

```
luci-app-rtp2httpd/
├── Makefile                 # OpenWrt 包定义（LUCI_TITLE / PKG_RELEASE r3 / DEPENDS +rtp2httpd）
├── Makefile.versioned
├── htdocs/luci-static/resources/view/rtp2httpd.js   # 页面视图（r3 修复版）
├── po/                      # 翻译模板与 zh_Hans
└── root/usr/share/
    ├── luci/menu.d/luci-app-rtp2httpd.json   # 菜单：服务 → rtp2httpd
    └── rpcd/acl.d/luci-app-rtp2httpd.json    # 访问控制
```

## 使用方法（换环境克隆即编译）

### 1. 前提

- 已就绪的 OpenWrt / ImmortalWrt 编译环境（feeds 已初始化）。
- 主程序包 [lxzlxz309/rtp2httpd](https://github.com/lxzlxz309/rtp2httpd) 需先放入 `feeds/packages/net/`
  （本包依赖 `rtp2httpd`，编译时由构建系统自动链接）。

### 2. 克隆到 luci feed 的 applications 目录

```bash
git clone git@github.com:lxzlxz309/luci-app-rtp2httpd.git <openwrt-src>/feeds/luci/applications/luci-app-rtp2httpd
```

> ⚠️ ImmortalWrt / OpenWrt 的 luci feed **自带** luci-app-rtp2httpd，
> 克隆会**覆盖**该目录以替换官方包。

### 3. 注册到 feeds

```bash
cd <openwrt-src>
./scripts/feeds update -i        # 刷新 feed 索引
./scripts/feeds install luci-app-rtp2httpd
```

### 4. 配置（menuconfig）

```bash
make menuconfig
# 进入 LuCI → Applications → luci-app-rtp2httpd
#   按 Y：编入固件 / 按 M：编成独立安装包（推荐）
```

> 只想单独出包可跳过 menuconfig，直接执行第 5 步。

### 5. 单独编译

```bash
make package/luci-app-rtp2httpd/compile V=s
```

产物路径：

```
bin/packages/<目标架构>/luci/luci-app-rtp2httpd-3.17.2-r3.apk
# 例：bin/packages/aarch64_cortex-a53/luci/luci-app-rtp2httpd-3.17.2-r3.apk
```

（传统 ipk 固件输出在 `bin/packages/<架构>/luci/luci-app-rtp2httpd_3.17.2-r3_all.ipk`。）

### 6. 编入固件（可选）

menuconfig 按 `Y` 后整体 `make V=s`。

## 说明

- 基于官方 luci-app-rtp2httpd（stackia/rtp2httpd 附带的 LuCI 支持包）定制。
- 装机时需与 `rtp2httpd` 主程序包一同安装；配置项说明见主程序仓库 `docs/`。
