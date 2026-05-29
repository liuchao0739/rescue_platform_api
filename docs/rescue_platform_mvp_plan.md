# 智能应急救援平台 MVP — 产品命名规范 & 技术方案

> 基于《MVP产品功能清单_中文需求文档_英文界面版.docx》整理  
> 文档日期：2026-05-29  
> 界面语言：English (en-US) | 需求文档语言：中文

**配套开发手册**：[rescue_platform_dev_guide.md](./rescue_platform_dev_guide.md)（仓库克隆、Monorepo、Android Studio、真机运行、SourceTree 等实操汇总）

---

## 目录

1. [产品命名规范](#一产品命名规范)
2. [产品定位与架构总览](#二产品定位与架构总览)
3. [需求范围梳理](#三需求范围梳理)
4. [技术选型方案](#四技术选型方案)
5. [模块分工建议](#五模块分工建议)
6. [分阶段开发任务清单](#六分阶段开发任务清单)
7. [整体时间线估算](#七整体时间线估算)
8. [验收 KPI](#八验收-kpi)
9. [风险与建议](#九风险与建议)
10. [项目落地进展](#十七项目落地进展)
11. [附录 A：MQTT Topic 规范草案](#附录amqtt-topic-规范草案)
12. [附录 B：各端 P0 功能清单速查](#附录b各端-p0-功能清单速查)

---

## 一、产品命名规范

### 1.1 命名规则

| 规则 | 说明 |
|------|------|
| 格式 | `xxx_xxx_xxx`（全小写，下划线分隔，三段式） |
| 前缀 | 统一使用 `rescue` 作为平台前缀 |
| 中段 | 表示产品类型或角色（`user` / `worker` / `ops` / `iot` / `platform`） |
| 末段 | 表示具体形态（`app` / `web` / `screen` / `api` / `worker` / `core`） |
| 用途 | 代码仓库名、服务名、包名、数据库前缀、MQTT Topic 前缀 |

### 1.2 产品命名一览

| 序号 | 产品 | 中文名称 | 命名（Code Name） | 说明 |
|------|------|----------|-------------------|------|
| 0 | 平台总称 | 智能应急救援平台 | `rescue_platform` | 整个 MVP 平台的统称 |
| 1 | 用户端 App | 用户端应用 | `rescue_user_app` | Flutter，iOS App Store + Google Play |
| 2 | 救援人员 App | 救援人员应用 | `rescue_worker_app` | Flutter，iOS + Android |
| 3 | 移动端 Monorepo | 移动端代码库 | `rescue_mobile_suite` | 两个 App 共用代码库 |
| 4 | 移动端共享包 | 共享核心包 | `rescue_mobile_core` | 网络、模型、地图、UI 组件 |
| 5 | 运营平台 Web | 运营平台 | `rescue_ops_web` | React 后台管理系统 |
| 6 | 调度大屏 | 调度大屏 | `rescue_dispatch_screen` | 运营平台的大屏视图（独立路由） |
| 7 | 物联网设备固件 | 物联网警示设备 | `rescue_iot_firmware` | 嵌入式固件，LTE-M / NB-IoT |
| 8 | 后端 API 服务 | 平台 API | `rescue_platform_api` | 主 REST API 服务 |
| 9 | MQTT 消费服务 | MQTT Worker | `rescue_mqtt_worker` | 设备消息消费、SOS 优先级处理 |
| 10 | WebSocket 服务 | 实时推送服务 | `rescue_ws_server` | 运营平台 / 大屏实时事件推送 |
| 11 | 通知异步服务 | 通知 Worker | `rescue_notify_worker` | Push / SMS / Email 异步发送 |

### 1.3 基础设施命名

| 资源 | 命名 | 说明 |
|------|------|------|
| 主数据库 | `rescue_platform_db` | PostgreSQL |
| 缓存 | `rescue_platform_redis` | Redis 7 |
| 消息队列 | `rescue_platform_mq` | RabbitMQ / Kafka |
| MQTT Broker | `rescue_platform_emqx` | EMQX 5.x 集群 |
| 对象存储 Bucket | `rescue-platform-assets` | S3 / MinIO，固件包、照片、语音 |
| Docker 镜像前缀 | `rescue/` | 如 `rescue/platform-api:latest` |

### 1.4 移动端包名 / Bundle ID

| 产品 | Android Package | iOS Bundle ID |
|------|-----------------|---------------|
| `rescue_user_app` | `com.rescue.user` | `com.rescue.user` |
| `rescue_worker_app` | `com.rescue.worker` | `com.rescue.worker` |

### 1.5 Git 仓库规划

**GitHub 组织/账号**： [liuchao0739](https://github.com/liuchao0739)

| 仓库 | 地址 | 状态 |
|------|------|------|
| rescue_mobile_suite | https://github.com/liuchao0739/rescue_mobile_suite | ✅ 已创建，Flutter Monorepo 已初始化 |
| rescue_ops_web | https://github.com/liuchao0739/rescue_ops_web | ✅ 已创建，目录骨架 + README |
| rescue_platform_api | https://github.com/liuchao0739/rescue_platform_api | ✅ 已创建，目录骨架 + README |
| rescue_iot_firmware | https://github.com/liuchao0739/rescue_iot_firmware | ✅ 已创建，目录骨架 + README |
| rescue_platform_infra | https://github.com/liuchao0739/rescue_platform_infra | ✅ 已创建，目录骨架 + README |

**本地项目组目录**（非 Git 仓库，仅用于聚合 clone）：

```
~/rescue_platform/
├── rescue_mobile_suite/          # Flutter Monorepo（user + worker App）
├── rescue_ops_web/               # 运营平台 + 调度大屏
├── rescue_platform_api/          # 后端 API + 各 Worker 服务
├── rescue_iot_firmware/          # 设备固件
└── rescue_platform_infra/        # DevOps、K8s、Terraform、监控配置
```

克隆、SourceTree、运行调试详见 [开发手册](./rescue_platform_dev_guide.md)。

### 1.6 MQTT Topic 前缀

所有设备 MQTT Topic 统一以 `rescue/iot/` 开头，详见 [附录 A](#附录amqtt-topic-规范草案)。

---

## 二、产品定位与架构总览

### 2.1 MVP 核心闭环

```
用户遇险
  → 设备 / 应用发起 SOS
  → 平台接收定位与设备状态
  → 调度员确认并派单
  → 用户查看救援进度
  → 救援完成并沉淀记录
```

**MVP 核心价值**：稳定 SOS + 精准定位 + 可靠调度 + 实时反馈

### 2.2 P0 核心闭环（必须跑通）

```
设备激活 / 绑定 SIM
  → 用户绑定设备
  → 用户或设备触发 SOS
  → 平台秒级收到 SOS
  → 后台地图弹出事件
  → 调度员联系用户并派单
  → 救援人员接单前往
  → 用户看到进度
  → 救援完成归档
```

### 2.3 产品组成（5 + 1）

| 部分 | 命名 | 说明 |
|------|------|------|
| 物联网警示设备 | `rescue_iot_firmware` | 硬件，SOS 按钮 + 警示灯 + GPS + 蜂窝通信 |
| 用户端应用 | `rescue_user_app` | 普通用户，Flutter 双端 |
| 救援人员应用 | `rescue_worker_app` | 救援人员 / 服务商，Flutter 双端 |
| 运营平台 | `rescue_ops_web` | Web 后台管理系统 |
| 调度大屏 | `rescue_dispatch_screen` | 运营平台的大屏视图，非独立产品 |
| 后端服务集群 | `rescue_platform_api` 等 | MQTT + API + WebSocket |

> **关于「PC 端」**：文档未单独列出 PC 客户端。PC 浏览器访问 `rescue_ops_web`（运营平台）和 `rescue_dispatch_screen`（调度大屏）即为 PC 端形态。

### 2.4 系统架构图

```
┌─────────────────────────────────────────────────────────────────┐
│                          客户端层                                │
│  rescue_user_app   rescue_worker_app   rescue_ops_web           │
│  (Flutter iOS/Android)                  rescue_dispatch_screen  │
└────────────┬──────────────────┬─────────────────┬───────────────┘
             │ REST API          │ REST API        │ REST + WebSocket
             ▼                   ▼                 ▼
┌─────────────────────────────────────────────────────────────────┐
│                     rescue_platform_api                          │
│  认证 | 用户 | 设备 | SIM | SOS | 调度 | OTA | 地图 | 审计      │
└──────┬──────────────────────────────────────────────────────────┘
       │                          ▲
       ▼                          │ WebSocket 推送
┌──────────────┐    ┌─────────────┴──────┐    ┌──────────────────┐
│rescue_mqtt   │    │  rescue_ws_server   │    │rescue_notify     │
│   _worker    │    │  (实时事件推送)      │    │   _worker        │
└──────┬───────┘    └─────────────────────┘    └──────────────────┘
       │
       ▼
┌──────────────┐    ┌──────────────┐    ┌──────────────┐
│rescue_platform│   │rescue_platform│   │rescue_platform│
│    _emqx     │───▶│     _mq      │───▶│    _redis    │
│ (MQTT Broker)│    │ (消息队列)    │    │   (缓存)     │
└──────┬───────┘    └──────────────┘    └──────────────┘
       │ MQTT over TLS
       ▼
┌──────────────┐
│rescue_iot    │
│  _firmware   │
│ (警示设备)    │
└──────────────┘
```

---

## 三、需求范围梳理

### 3.1 MVP 纳入范围

| 产品 | 是否纳入 | 说明 |
|------|----------|------|
| 物联网警示设备 | ✅ 是 | SOS 按钮、警示灯、GPS、LTE-M 主网、NB-IoT 辅助、MQTT、OTA |
| 用户端应用 | ✅ 是 | SOS、定位、设备、救援跟踪、异常上报、消息、在线升级 |
| 救援人员应用 | ✅ 轻量纳入 | 接单、导航、状态更新、在线升级 |
| 运营平台 | ✅ 是 | SOS、调度、设备、地图、用户、SIM、OTA、权限 |
| 调度大屏 | ✅ 是 | 运营平台的大屏视图 |
| AI 调度 | ❌ 非 P0 | 仅规则风险等级 + 最近资源推荐 |
| 911 / CAD 联动 | ⏳ P1/P2 | MVP 先人工升级与记录 |
| 保险 / 商城 / 会员 | ❌ 不纳入 | 后续商业化 |

### 3.2 界面语言要求

| 端 | 界面语言 | 说明 |
|----|----------|------|
| 全部客户端 | English (en-US) | 默认界面语言 |
| 需求文档 | 中文 | 功能描述用中文，标注英文界面名 |
| 后端模板 | i18n | Push、SMS、Email、错误码支持多语言 |
| P1 扩展语言 | zh-CN、es-ES | 后续支持 |

**推荐英文状态值**：`CREATED` / `CONFIRMED` / `DISPATCHING` / `ON_THE_WAY` / `ARRIVED` / `COMPLETED` / `CANCELLED` / `ESCALATED`

### 3.3 美国市场物联网网络策略

| 项目 | 要求 |
|------|------|
| 主网络 | LTE-M，基于美国主流 4G 蜂窝物联网 |
| 辅助网络 | NB-IoT，补充覆盖和低功耗场景 |
| 网络优先级 | 优先 LTE-M，不可用时尝试 NB-IoT |
| 通信协议 | MQTT over TLS |
| 核心上报 | SOS、GPS、Heartbeat、Battery、Signal、OTA Status |
| 弱网策略 | SOS 优先发送；失败后重试、缓存并补发 |
| 平台字段 | network_type、signal_strength、carrier、cell_id |

---

## 四、技术选型方案

### 4.1 各端技术栈

| 产品（命名） | 推荐技术 | 理由 |
|-------------|----------|------|
| `rescue_user_app` | Flutter 3.x + Dart | 一套代码 iOS/Android 双端上架 |
| `rescue_worker_app` | Flutter 3.x（Monorepo 共用） | 与用户端共享 core/models/map |
| `rescue_ops_web` | React 18 + TypeScript + Ant Design Pro | 后台表格/表单/权限成熟 |
| `rescue_dispatch_screen` | React + Mapbox GL + WebSocket | 运营平台子路由，复用地图和推送 |
| `rescue_platform_api` | Go (Gin/Fiber) 或 Java (Spring Boot) | Go 更适合高并发 MQTT 消息处理 |
| `rescue_mqtt_worker` | Go | 与 API 同语言，独立进程可水平扩展 |
| `rescue_ws_server` | Go / Node.js | WebSocket 实时推送 |
| `rescue_notify_worker` | Go | Push/SMS/Email 异步发送 |
| `rescue_iot_firmware` | C/C++ + FreeRTOS | 嵌入式，MQTT 客户端 |
| MQTT Broker | EMQX 5.x 集群 | 文档明确要求 |
| 消息队列 | RabbitMQ（MVP）/ Kafka（扩展） | SOS 削峰、GPS/心跳异步 |
| 数据库 | PostgreSQL 15+ + PostGIS | JSONB 事件时间线、GIS 支持 |
| 缓存 | Redis 7 | 设备在线、最新 GPS、SOS 状态 |
| 对象存储 | AWS S3 / MinIO | 固件包、照片、语音附件 |
| 地图 | Mapbox（美国市场）/ Google Maps SDK | 用户/救援端导航；后台 GIS |
| 推送 | FCM + APNs | 跨平台 Push；SMS 用 Twilio |
| 监控 | Prometheus + Grafana | API/MQTT/Redis/WS 可观测 |
| CI/CD | GitHub Actions + Fastlane | Flutter 双端打包上架自动化 |

### 4.2 Flutter Monorepo 结构（`rescue_mobile_suite`）

```
rescue_mobile_suite/
├── apps/
│   ├── rescue_user_app/          # 用户端
│   └── rescue_worker_app/        # 救援人员端
├── packages/
│   ├── rescue_mobile_core/       # 网络、鉴权、错误码、i18n
│   ├── rescue_mobile_models/     # 共享数据模型
│   ├── rescue_mobile_map/        # 地图组件封装
│   └── rescue_mobile_ui/         # 共享 UI 组件
└── melos.yaml
```

### 4.3 后端服务拆分（MVP 3~4 个部署单元）

| 部署单元 | 包含服务 | 说明 |
|----------|----------|------|
| `rescue_platform_api` | 认证、用户、设备、SIM、SOS、调度、OTA、地图 | 主 REST API |
| `rescue_mqtt_worker` | MQTT 消息消费、SOS 优先级、GPS/心跳入库 | 独立进程，可水平扩展 |
| `rescue_notify_worker` | Push、SMS、Email 异步发送 | 解耦通知 |
| `rescue_ws_server` | WebSocket 推送到运营平台/大屏 | 实时事件流 |

### 4.4 高并发架构原则

```
设备 → EMQX 集群 → 消息队列/异步处理
     → SOS/设备/调度服务 → Redis 缓存 → PostgreSQL
     → WebSocket 推送到 rescue_ops_web / rescue_dispatch_screen
```

| 原则 | 说明 |
|------|------|
| SOS 优先 | SOS 消息最高优先级，不被 GPS/心跳阻塞 |
| 实时状态走缓存 | Redis 存设备在线、最新 GPS、SOS 状态 |
| 业务记录走 DB | PostgreSQL 持久化 |
| 高频消息异步 | GPS/心跳/OTA 走 MQ 异步处理 |
| 水平扩展 | 核心服务支持多实例部署 |

### 4.5 P0 后端核心服务

| 服务 | P0 要求 |
|------|---------|
| 认证服务 | 登录、Token、权限校验 |
| 用户服务 | 用户资料、紧急联系人、车辆 |
| 设备服务 | 设备档案、绑定、状态、激活 |
| SIM 服务 | SIM 档案、绑定、状态、异常 |
| SOS 服务 | SOS 创建、状态流转、优先级、归档 |
| 调度服务 | 工单创建、人工派单、状态更新、完成 |
| 消息通知服务 | Push、短信、站内消息 |
| 地图服务 | SOS/设备/救援资源位置、路线 ETA |
| OTA 服务 | 设备固件 + 应用版本检测 |
| 日志审计服务 | 操作日志、API 日志 |

### 4.6 P0 核心数据库表

| 数据表 | 说明 |
|--------|------|
| users | 用户账号、联系方式、状态 |
| emergency_contacts | 紧急联系人 |
| vehicles | 车辆资料 |
| devices | SN、IMEI、型号、状态、固件 |
| sim_cards | ICCID、运营商、套餐、激活状态 |
| device_sim_bindings | 设备与 SIM 绑定关系 |
| sos_events | SOS 位置、类型、状态、时间线 |
| rescue_orders | 调度、接单、到达、完成 |
| rescue_resources | 救援人员、车辆、在线状态 |
| notifications | Push、短信、站内消息记录 |
| ota_tasks | 固件升级和应用版本记录 |
| operation_logs | 后台操作和关键业务变更 |

---

## 五、模块分工建议

### 5.1 团队配置（8~12 人）

| 角色 | 人数 | 负责范围 |
|------|------|----------|
| Tech Lead / 架构师 | 1 | 整体架构、EMQX/MQ 设计、SOS 优先级链路 |
| 后端工程师 A | 1~2 | 认证、用户、SOS、调度、WebSocket |
| 后端工程师 B | 1 | 设备、SIM、MQTT Worker、OTA |
| Flutter 工程师 A | 1~2 | `rescue_user_app`（SOS 核心、地图、设备） |
| Flutter 工程师 B | 1 | `rescue_worker_app` + 共享 packages |
| 前端工程师 | 1~2 | `rescue_ops_web` + `rescue_dispatch_screen` |
| 硬件/嵌入式工程师 | 1 | `rescue_iot_firmware`、MQTT 协议、OTA |
| QA / 测试 | 1 | SOS 闭环 E2E、弱网/高峰压测、上架验收 |
| DevOps | 0.5~1 | 部署、CI/CD、监控告警 |
| 产品经理 | 1 | 需求澄清、验收 KPI、试点协调 |

### 5.2 模块依赖关系

```
rescue_iot_firmware (MQTT 协议)
        ↓
rescue_mqtt_worker → rescue_platform_api (SOS API)
        ↓                        ↓
rescue_ops_web (SOS 中心)    rescue_user_app (SOS 发送)
        ↓
    调度派单
        ↓
rescue_worker_app (接单) → rescue_user_app (救援跟踪)

rescue_platform_api (设备/SIM) → rescue_user_app (设备绑定)
rescue_ws_server → rescue_ops_web / rescue_dispatch_screen
```

**关键路径**：硬件 MQTT → 后端 SOS 链路 → 运营平台 → 调度 → 救援 App → 用户端跟踪

### 5.3 各模块交付物

| 模块 | 主要交付物 | 对接方 |
|------|------------|--------|
| `rescue_iot_firmware` | MQTT Topic 规范、设备鉴权方案、OTA 协议 | 后端 B |
| `rescue_platform_api` | OpenAPI 文档、WebSocket 事件协议 | 全部前端 |
| `rescue_user_app` | TestFlight / 内测 APK、App Store 素材 | 后端、硬件 |
| `rescue_worker_app` | 同上 | 后端 |
| `rescue_ops_web` | Web 部署 URL、RBAC 权限矩阵 | 后端 |
| `rescue_dispatch_screen` | 大屏页面 + 投屏方案 | 运营平台 |
| `rescue_platform_infra` | dev/staging/prod 环境、监控面板 | 全员 |

---

## 六、分阶段开发任务清单

### 第一阶段：设备 + MQTT + SOS API（4~6 周）

**目标**：设备按下 SOS，平台 3 秒内收到并落库

| # | 任务 | 负责 | 优先级 |
|---|------|------|--------|
| 1.1 | 确定 MQTT Topic 规范（SOS/GPS/Heartbeat/Battery/Signal/OTA） | 架构 + 硬件 | P0 |
| 1.2 | EMQX 集群部署 + 设备 TLS 鉴权 | DevOps + 后端 B | P0 |
| 1.3 | `rescue_iot_firmware`：SOS 按钮 + GPS + LTE-M 联调 | 硬件 | P0 |
| 1.4 | `rescue_mqtt_worker`：SOS 高优先级消费 + Redis 缓存 | 后端 B | P0 |
| 1.5 | SOS 服务：创建事件、状态机 | 后端 A | P0 |
| 1.6 | 数据库：devices、sos_events、sim_cards 表 | 后端 A/B | P0 |
| 1.7 | SOS REST API：创建/查询/状态更新 | 后端 A | P0 |
| 1.8 | 验证：SOS 送达率 > 99.9%，延迟 < 3s | QA | P0 |

**阶段验收**：设备触发 SOS → API 可查询 → Redis 有实时状态

---

### 第二阶段：用户端 App + 运营平台（6~8 周）

**目标**：用户发 SOS 能看到状态；调度员能在后台看到事件和地图

| # | 任务 | 负责 | 优先级 |
|---|------|------|--------|
| 2.1 | 认证服务（手机/邮箱/Apple/Google 登录） | 后端 A | P0 |
| 2.2 | 用户服务、紧急联系人、设备绑定 API | 后端 A | P0 |
| 2.3 | 设备服务、SIM 绑定、激活校验 | 后端 B | P0 |
| 2.4 | `rescue_mobile_suite` 初始化、Monorepo、网络层、i18n | Flutter A | P0 |
| 2.5 | `rescue_user_app`：登录/注册/首页/SOS 长按确认/事故类型 | Flutter A | P0 |
| 2.6 | `rescue_user_app`：SOS 状态页、救援跟踪 | Flutter A | P0 |
| 2.7 | `rescue_user_app`：设备列表/扫码绑定/设备详情 | Flutter A | P0 |
| 2.8 | `rescue_ops_web`：登录 + RBAC 权限框架 | 前端 | P0 |
| 2.9 | `rescue_ops_web`：Dashboard + SOS 中心 + SOS 列表/详情 | 前端 | P0 |
| 2.10 | `rescue_ops_web`：GIS 地图 + SOS 图层 + 设备图层 | 前端 | P0 |
| 2.11 | `rescue_ops_web`：设备管理 + SIM 管理 | 前端 + 后端 B | P0 |
| 2.12 | `rescue_ws_server`：SOS 事件实时推送到运营平台 | 后端 A + 前端 | P0 |

**阶段验收**：用户 App 发 SOS → 后台 1 秒内地图出现点位 → 可查看详情

---

### 第三阶段：调度 + 救援人员 App（4~6 周）

**目标**：调度员派单，救援人员接单并更新状态，用户看到进度

| # | 任务 | 负责 | 优先级 |
|---|------|------|--------|
| 3.1 | 调度服务（工单创建、人工派单、状态流转） | 后端 A | P0 |
| 3.2 | 救援资源管理（人员/车辆/在线状态） | 后端 A | P0 |
| 3.3 | `rescue_ops_web`：调度中心 + 工单看板 + 人工派单 | 前端 | P0 |
| 3.4 | `rescue_ops_web`：调度时间线 + 风险等级 L1-L5 | 前端 + 后端 A | P0 |
| 3.5 | `rescue_worker_app`：登录 + 可接工单 + 我的工单 | Flutter B | P0 |
| 3.6 | `rescue_worker_app`：接单 + 导航 + 状态更新 | Flutter B | P0 |
| 3.7 | `rescue_user_app`：救援跟踪（救援人员位置、ETA、路线） | Flutter A | P0 |
| 3.8 | 通知服务：SOS 状态 Push 通知用户 | 后端 A + Flutter A | P0 |
| 3.9 | `rescue_dispatch_screen` v1：SOS 队列 + 地图 + 事件流 | 前端 | P0 |

**阶段验收**：完整 P0 闭环跑通

---

### 第四阶段：OTA + 应用升级 + 通知 + 监控（3~4 周）

**目标**：设备固件和应用可持续升级；运营稳定性可观测

| # | 任务 | 负责 | 优先级 |
|---|------|------|--------|
| 4.1 | OTA 服务（固件包管理、任务创建、状态跟踪） | 后端 B | P0 |
| 4.2 | `rescue_iot_firmware` OTA 升级联调 | 硬件 + 后端 B | P0 |
| 4.3 | `rescue_ops_web`：OTA 管理页面 | 前端 | P0 |
| 4.4 | Flutter 双端：应用版本检测 + 升级提示 + Release Notes | Flutter A/B | P0 |
| 4.5 | `rescue_user_app`：设备 OTA 状态展示 | Flutter A | P0 |
| 4.6 | 通知完善：SMS（Twilio）、站内消息、设备/SIM 异常告警 | 后端 A | P0 |
| 4.7 | 监控：Prometheus + Grafana 面板 | DevOps | P0 |
| 4.8 | `rescue_ops_web`：系统健康页 + 操作日志 | 前端 + 后端 | P0 |

---

### 第五阶段：高并发架构验证（2~3 周）

**目标**：压测验证高并发要求

| # | 任务 | 负责 | 优先级 |
|---|------|------|--------|
| 5.1 | EMQX 集群扩展验证（模拟 1 万+ 设备连接） | DevOps + 后端 B | P0 |
| 5.2 | SOS 高峰压测：SOS 不被 GPS/心跳阻塞 | 后端 B + QA | P0 |
| 5.3 | API 多实例部署 + 限流熔断验证 | DevOps + 后端 A | P0 |
| 5.4 | WebSocket 推送延迟 < 1s 验证 | QA | P0 |
| 5.5 | Redis 缓存命中率、DB 读写分离（如需要） | 后端 A | P1 |

---

### 第六阶段：试点上线（2~4 周）

| # | 任务 | 负责 | 优先级 |
|---|------|------|--------|
| 6.1 | App Store + Google Play 上架（user + worker） | Flutter + QA | P0 |
| 6.2 | 生产环境部署 + 域名/SSL/备份策略 | DevOps | P0 |
| 6.3 | 1~2 个城市/区域试点设备投放 | 产品 + 硬件 | P0 |
| 6.4 | KPI 验收 | QA + 产品 | P0 |
| 6.5 | P1 功能评估：异常上报、道路风险、聊天等 | 产品 | P1 |

---

## 七、整体时间线估算

| 阶段 | 周期 | 累计 |
|------|------|------|
| 第一阶段：设备 + MQTT + SOS API | 4~6 周 | 6 周 |
| 第二阶段：用户端 App + 运营平台 | 6~8 周 | 14 周 |
| 第三阶段：调度 + 救援人员 App | 4~6 周 | 20 周 |
| 第四阶段：OTA + 升级 + 通知 + 监控 | 3~4 周 | 24 周 |
| 第五阶段：高并发架构验证 | 2~3 周 | 27 周 |
| 第六阶段：试点上线 | 2~4 周 | **~30 周（约 7~8 个月）** |

> 部分阶段可并行（硬件联调与后端 API、运营平台与用户端 App），实际可压缩至 **5~6 个月**。

---

## 八、验收 KPI

| 指标 | 目标 |
|------|------|
| SOS 送达率 | > 99.9% |
| SOS 平台接收延迟 | < 3 秒 |
| GPS 定位精度 | 信号允许时 < 10 米 |
| 调度员响应时间 | < 60 秒 |
| 救援完成率 | > 95% |
| 设备在线状态准确率 | > 99% |
| 地图刷新延迟 | < 1 秒 |
| SOS 高峰消息处理 | SOS 不被 GPS/心跳阻塞 |
| WebSocket 推送延迟 | < 1 秒 |
| 核心 API 横向扩展 | 支持多实例部署 |
| 用户端应用在线升级 | 版本检测、升级提示、Release Notes |
| 救援人员应用在线升级 | 版本检测、升级提示、Release Notes |

---

## 九、风险与建议

| 风险 | 影响 | 建议 |
|------|------|------|
| 硬件 LTE-M 美国运营商对接慢 | 阻塞第一阶段 | 先用 4G 模块 + 模拟器开发，硬件并行 |
| 地图 SDK 成本（Mapbox/Google） | 预算 | MVP 用 Mapbox 免费额度；上线前评估 |
| App Store 审核（SOS 紧急功能） | 上架延迟 | 提前准备隐私政策、定位权限说明、SOS 误触机制 |
| 高并发架构过度设计 | 工期 | MVP 用 3~4 部署单元，表结构预留扩展 |
| 两个 App 是否独立上架 | 商店策略 | 独立 Bundle ID，救援端可企业分发或 TestFlight |

---

## 附录 A：MQTT Topic 规范草案

### Topic 命名规则

格式：`rescue/iot/{device_id}/{message_type}`

### 设备 → 平台（Uplink）

| Topic | 说明 | QoS | 优先级 |
|-------|------|-----|--------|
| `rescue/iot/{device_id}/sos` | SOS 紧急事件 | 2 | **最高** |
| `rescue/iot/{device_id}/gps` | GPS 位置上报 | 0 | 普通 |
| `rescue/iot/{device_id}/heartbeat` | 心跳 / 在线状态 | 0 | 普通 |
| `rescue/iot/{device_id}/battery` | 电量状态 | 0 | 普通 |
| `rescue/iot/{device_id}/signal` | 信号状态 | 0 | 普通 |
| `rescue/iot/{device_id}/ota/status` | OTA 升级状态回报 | 1 | 高 |

### 平台 → 设备（Downlink）

| Topic | 说明 | QoS |
|-------|------|-----|
| `rescue/iot/{device_id}/cmd/light_test` | 远程灯光测试 | 1 |
| `rescue/iot/{device_id}/cmd/ota` | OTA 升级指令 | 1 |
| `rescue/iot/{device_id}/cmd/config` | 配置更新 | 1 |

### SOS 消息 Payload 示例

```json
{
  "device_id": "DEV-20260001",
  "event_type": "SOS",
  "timestamp": "2026-05-29T10:30:00Z",
  "location": {
    "latitude": 37.7749,
    "longitude": -122.4194,
    "accuracy": 8.5
  },
  "battery_level": 85,
  "signal_strength": -72,
  "network_type": "LTE-M",
  "carrier": "AT&T",
  "cell_id": "310410-12345",
  "incident_type": "ROADside_ASSISTANCE"
}
```

---

## 附录 B：各端 P0 功能清单速查

### `rescue_user_app` P0

登录/注册 → 首页 SOS 长按 → SOS 确认/事故类型 → SOS 状态/救援跟踪 → 设备绑定/详情 → 消息通知 → 联系客服 → 在线升级

### `rescue_worker_app` P0

登录 → 可接工单/我的工单 → 工单详情 → 接单 → 导航 → 状态更新（On The Way / Arrived / Completed）→ 联系用户 → 在线升级

### `rescue_ops_web` P0

Dashboard → SOS 中心/列表/详情 → 调度中心/工单看板/人工派单 → GIS 地图（SOS/设备/救援资源图层）→ 设备管理 → SIM 管理 → OTA 管理 → 用户管理 → RBAC 权限 → 操作日志 → 系统健康

### `rescue_dispatch_screen` P0

全局状态栏 → SOS 事件队列 → 实时 GIS 地图 → 调度资源 → 事件流 → 关键告警浮层

### `rescue_iot_firmware` P0

物理 SOS 按钮 → 高亮警示灯 → GPS 定位 → LTE-M 主连接 → MQTT 通信 → 心跳/电量/信号上报 → OTA 升级 → 防误触

---

## 十七、项目落地进展

> 汇总截至 2026-05-29 的工程落地情况；操作细节见 [开发手册](./rescue_platform_dev_guide.md)。

### 17.1 已完成

| 事项 | 说明 |
|------|------|
| 需求梳理 | 基于 docx 整理 MVP 范围、五端架构、分期与 KPI |
| 产品命名 | `rescue_xxx_xxx` 规范，见第一章 |
| GitHub 五仓 | 全部 Public，含 README 与 `docs/rescue_platform_mvp_plan.md` |
| 本地 clone | `~/rescue_platform/` 下 5 仓已克隆，可用 SourceTree 管理 |
| Flutter Monorepo | `rescue_mobile_suite`：双 App + 4 共享包 + Melos workspace |
| 真机验证 | `rescue_user_app` 已在 Android 真机 V2049A 上 `flutter run` 通过 |
| 文档双册 | `rescue_platform_mvp_plan.md` + `rescue_platform_dev_guide.md` |

### 17.2 Monorepo 脚手架摘要

- **用户端**：`apps/rescue_user_app`（`com.rescue.user`）— 首页 + Emergency SOS 占位 UI  
- **救援端**：`apps/rescue_worker_app`（`com.rescue.worker`）— 工单首页占位 UI  
- **共享包**：`rescue_mobile_core`、`rescue_mobile_models`（含 `SosStatus`）、`rescue_mobile_map`、`rescue_mobile_ui`  
- **工具**：`melos bootstrap` 安装全仓依赖  

### 17.3 Android Studio 打开路径

| 目的 | 路径 |
|------|------|
| 跑用户端 | `rescue_mobile_suite/apps/rescue_user_app` |
| 跑救援端 | `rescue_mobile_suite/apps/rescue_worker_app` |
| 改共享包 | 用 Cursor 打开 `rescue_mobile_suite` 根目录 |

### 17.4 待推进（按 MVP 第一阶段）

1. MQTT Topic 协议定稿（`rescue_iot_firmware` ↔ `rescue_platform_api`）  
2. PostgreSQL 表结构与 migrations  
3. SOS API + EMQX 联调（SOS 延迟 < 3s）  
4. `rescue_ops_web` SOS 中心与地图  
5. 用户端 SOS 流程对接后端  

---

*文档版本：v1.1 | 更新日期：2026-05-29*
