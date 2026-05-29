# 智能应急救援平台 — 开发手册

> 配套文档：[rescue_platform_mvp_plan.md](./rescue_platform_mvp_plan.md)（产品规划与技术方案）  
> 更新日期：2026-05-29  
> GitHub 账号：[liuchao0739](https://github.com/liuchao0739)

---

## 文档说明

| 文档 | 用途 |
|------|------|
| `rescue_platform_mvp_plan.md` | 需求范围、技术选型、分期任务、KPI、MQTT 规范 |
| `rescue_platform_dev_guide.md`（本文） | 仓库、本地工程、Monorepo、IDE、运行调试、协作流程 |

两份文档均已同步到各 Git 仓库的 `docs/` 目录。

---

## 一、产品与仓库总览

### 1.1 五个端 + 后端

| 部分 | 命名 | 仓库 | 技术 |
|------|------|------|------|
| 用户端 App | `rescue_user_app` | [rescue_mobile_suite](https://github.com/liuchao0739/rescue_mobile_suite) | Flutter |
| 救援人员 App | `rescue_worker_app` | 同上（Monorepo） | Flutter |
| 运营平台 Web | `rescue_ops_web` | [rescue_ops_web](https://github.com/liuchao0739/rescue_ops_web) | React + Ant Design Pro |
| 调度大屏 | `rescue_dispatch_screen` | 同上（子路由） | React |
| 物联网固件 | `rescue_iot_firmware` | [rescue_iot_firmware](https://github.com/liuchao0739/rescue_iot_firmware) | C/C++ 嵌入式 |
| 后端 API | `rescue_platform_api` | [rescue_platform_api](https://github.com/liuchao0739/rescue_platform_api) | Go / Java |
| 基础设施 | `rescue_platform_infra` | [rescue_platform_infra](https://github.com/liuchao0739/rescue_platform_infra) | Docker / K8s / Terraform |

后端逻辑服务（可同仓多进程部署）：`rescue_mqtt_worker`、`rescue_ws_server`、`rescue_notify_worker`。

### 1.2 P0 核心闭环（必跑通）

```
设备激活/绑定 SIM → 用户绑定设备 → 触发 SOS → 平台秒级收到
→ 后台地图弹出 → 调度员派单 → 救援人员接单前往 → 用户看进度 → 完成归档
```

---

## 二、本地工程目录

建议在用户目录下用父文件夹聚合（**非 Git 仓库**，仅本地分组）：

```
~/rescue_platform/                    # 本地项目组目录（可选）
├── rescue_mobile_suite/              # git clone
├── rescue_ops_web/
├── rescue_platform_api/
├── rescue_iot_firmware/
└── rescue_platform_infra/
```

当前本机路径示例：`/Users/liuchao/rescue_platform/`

---

## 三、克隆与 SourceTree

### 3.1 一次性克隆

```bash
mkdir -p ~/rescue_platform && cd ~/rescue_platform

git clone https://github.com/liuchao0739/rescue_mobile_suite.git
git clone https://github.com/liuchao0739/rescue_ops_web.git
git clone https://github.com/liuchao0739/rescue_platform_api.git
git clone https://github.com/liuchao0739/rescue_iot_firmware.git
git clone https://github.com/liuchao0739/rescue_platform_infra.git
```

### 3.2 SourceTree 使用说明

- **5 个独立 Git 仓库**，需分别 **File → Open** 添加，不能只对父目录 `rescue_platform` 点一次（父目录不是 git 根）。
- 每个子目录有自己的提交、分支、Push 操作。
- 批量推送（终端）：

```bash
for dir in rescue_mobile_suite rescue_platform_api rescue_ops_web rescue_iot_firmware rescue_platform_infra; do
  git -C ~/rescue_platform/$dir push origin main
done
```

### 3.3 批量创建仓库脚本

`~/Downloads/create_rescue_repos.sh` — 在 GitHub 上创建仓库（已执行过，重复运行会跳过已存在仓库）。

---

## 四、各仓库当前状态（2026-05-29）

| 仓库 | README | docs/MVP 规划 | 代码脚手架 |
|------|--------|---------------|------------|
| rescue_mobile_suite | ✅ | ✅ | ✅ Flutter Monorepo 已初始化 |
| rescue_platform_api | ✅ | ✅ | ✅ `cmd/`、`internal/` 目录骨架 |
| rescue_ops_web | ✅ | ✅ | ✅ `src/` 目录骨架 |
| rescue_iot_firmware | ✅ | ✅ | ✅ `src/` 目录骨架 |
| rescue_platform_infra | ✅ | ✅ | ✅ `docker/`、`k8s/`、`terraform/` 骨架 |

各仓库 `docs/` 内文件：

- `rescue_platform_mvp_plan.md`
- `rescue_platform_dev_guide.md`（同步后）

---

## 五、Flutter Monorepo（rescue_mobile_suite）

### 5.1 目录结构

```
rescue_mobile_suite/
├── apps/
│   ├── rescue_user_app/          # 用户端 com.rescue.user
│   └── rescue_worker_app/        # 救援端 com.rescue.worker
├── packages/
│   ├── rescue_mobile_core/       # 配置、鉴权、API（规划）
│   ├── rescue_mobile_models/     # SosStatus 等共享模型
│   ├── rescue_mobile_map/        # 地图占位组件
│   └── rescue_mobile_ui/         # 共享 UI（如 SOS 按钮）
├── docs/
├── melos.yaml
├── pubspec.yaml                  # Dart workspace 根
└── pubspec.lock
```

### 5.2 环境要求

- Flutter SDK >= 3.9（stable）
- Melos 7.x：`dart pub global activate melos`

### 5.3 安装依赖

```bash
cd ~/rescue_platform/rescue_mobile_suite
export PATH="$PATH:$HOME/.pub-cache/bin"
melos bootstrap
```

### 5.4 命令行运行

```bash
# 查看已连接设备
flutter devices

# 用户端（示例：vivo V2049A）
cd apps/rescue_user_app
flutter run -d 3042791426007LI

# 救援人员端
cd apps/rescue_worker_app
flutter run -d 3042791426007LI
```

热重载：终端内 `r` 热重载，`R` 热重启，`q` 退出。

### 5.5 已验证设备（本机）

| 设备名 | Device ID | 系统 |
|--------|-----------|------|
| V2049A | `3042791426007LI` | Android 14 |
| OCE AN10 | `XWN0220B28001347` | Android 12 |

`rescue_user_app` 已在 V2049A 上成功 `flutter run`（debug）。

---

## 六、Android Studio 打开路径

| 场景 | 打开路径 |
|------|----------|
| **跑用户端（推荐）** | `~/rescue_platform/rescue_mobile_suite/apps/rescue_user_app` |
| **跑救援人员端** | `~/rescue_platform/rescue_mobile_suite/apps/rescue_worker_app` |
| **改共享 packages** | 用 Cursor / VS Code 打开 `~/rescue_platform/rescue_mobile_suite` 根目录 |

Android Studio 步骤：

1. **File → Open** → 选择上述 App 目录  
2. 等待 Gradle / `pub get` 完成  
3. 顶部设备选择 **V2049A**（或你的真机）  
4. 点击 Run ▶️  

说明：IDE 里若显示「Flutter (staging)」等，可能来自其它项目配置；本仓库当前为默认 **debug**，尚未配置 staging flavor。

---

## 七、其他仓库（待开发）

### rescue_platform_api

规划目录：`cmd/api`、`cmd/mqtt_worker`、`cmd/ws_server`、`cmd/notify_worker`、`internal/`、`migrations/`。

本地开发（待实现）：`docker compose up -d` → `go run ./cmd/api`。

### rescue_ops_web

规划：React 18 + TypeScript + Ant Design Pro；调度大屏路由 `/dispatch-screen`。

### rescue_iot_firmware

MQTT Topic 前缀：`rescue/iot/{device_id}/...`，详见 MVP 规划附录 A。

### rescue_platform_infra

EMQX、PostgreSQL、Redis、Prometheus/Grafana 等，见仓库 README。

---

## 八、Git 协作与推送

- 远程：`https://github.com/liuchao0739/<repo>.git`，默认分支 `main`。
- 使用 `gh` 登录：`gh auth login` → `gh auth setup-git`。
- 各仓库独立提交；Monorepo 改动范围大时注意只提交相关文件。
- 2026-05-29 已完成首轮推送：README、docs、mobile 脚手架均已上 GitHub。

---

## 九、推荐下一步

| 优先级 | 事项 | 负责仓库 |
|--------|------|----------|
| P0 | MQTT Topic 协议定稿（硬件 ↔ 后端） | iot_firmware + platform_api |
| P0 | 数据库 ER / migrations 初版 | platform_api |
| P0 | SOS API + EMQX 联调 | platform_api + infra |
| P1 | 用户端 SOS 流程对接 API | rescue_mobile_suite |
| P1 | 运营平台 SOS 中心 + 地图 | rescue_ops_web |

---

## 十、相关链接

- GitHub 仓库列表：https://github.com/liuchao0739?tab=repositories  
- MVP 规划文档：[rescue_platform_mvp_plan.md](./rescue_platform_mvp_plan.md)  
- 原始需求：`MVP产品功能清单_中文需求文档_英文界面版.docx`（Downloads）

---

*文档版本：v1.0 | 2026-05-29*
