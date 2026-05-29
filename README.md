# rescue_platform_api

智能应急救援平台（Rescue Platform）**后端服务**仓库，承载 REST API 及异步 Worker，支撑 SOS 救援闭环与高并发设备接入。

| 属性 | 说明 |
|------|------|
| 平台 | [rescue_platform](https://github.com/liuchao0739) |
| 目标市场 | 美国（LTE-M 主网、MQTT over TLS） |
| 推荐语言 | Go (Gin/Fiber) 或 Java (Spring Boot) |

## 服务组成（规划）

MVP 阶段建议 **3~4 个可部署单元**，逻辑分层、可合并部署：

| 服务名 | 职责 |
|--------|------|
| `rescue_platform_api` | 主 REST API：认证、用户、设备、SIM、SOS、调度、OTA、地图 |
| `rescue_mqtt_worker` | 消费 EMQX 消息，SOS 高优先级处理，GPS/心跳入库 |
| `rescue_ws_server` | WebSocket 实时推送到运营平台 / 调度大屏 |
| `rescue_notify_worker` | Push / SMS / Email 异步通知 |

## 规划目录结构

```
rescue_platform_api/
├── cmd/
│   ├── api/              # 主 API 入口
│   ├── mqtt_worker/
│   ├── ws_server/
│   └── notify_worker/
├── internal/
│   ├── auth/
│   ├── user/
│   ├── device/
│   ├── sim/
│   ├── sos/
│   ├── dispatch/
│   ├── ota/
│   └── notify/
├── api/                  # OpenAPI / proto 定义
├── migrations/           # 数据库迁移
├── docs/
│   ├── rescue_platform_mvp_plan.md
│   └── mqtt_topics.md    # 待补充
└── docker-compose.yml    # 本地 dev：PG + Redis + EMQX
```

## P0 核心服务

- 认证、用户、设备、SIM、SOS、调度、消息通知、地图、OTA、日志审计

## P0 数据表

`users`、`emergency_contacts`、`vehicles`、`devices`、`sim_cards`、`device_sim_bindings`、`sos_events`、`rescue_orders`、`rescue_resources`、`notifications`、`ota_tasks`、`operation_logs`

## 架构原则

```
设备 → EMQX → MQ → SOS/设备/调度服务 → Redis → PostgreSQL
                                              ↓
                                    WebSocket → ops_web / dispatch_screen
```

- **SOS 优先**：不被 GPS / 心跳阻塞  
- **实时状态** → Redis  
- **业务记录** → PostgreSQL  
- **水平扩展**：API / Worker 多实例  

## MQTT Topic 前缀

设备上行统一：`rescue/iot/{device_id}/{message_type}`  

详见 [MVP 规划文档 - 附录 A](docs/rescue_platform_mvp_plan.md#附录amqtt-topic-规范草案)

## 本地开发（待实现）

```bash
# 启动依赖
docker compose up -d

# 运行 API
go run ./cmd/api
```

## 验收 KPI（节选）

| 指标 | 目标 |
|------|------|
| SOS 送达率 | > 99.9% |
| SOS 平台接收延迟 | < 3 秒 |
| WebSocket 推送延迟 | < 1 秒 |

## 相关仓库

| 仓库 | 说明 |
|------|------|
| [rescue_mobile_suite](https://github.com/liuchao0739/rescue_mobile_suite) | Flutter 双端 App |
| [rescue_ops_web](https://github.com/liuchao0739/rescue_ops_web) | 运营平台 + 调度大屏 |
| [rescue_iot_firmware](https://github.com/liuchao0739/rescue_iot_firmware) | 设备固件 |
| [rescue_platform_infra](https://github.com/liuchao0739/rescue_platform_infra) | EMQX / K8s / 监控 |

## 文档

- [MVP 产品规划与技术方案](docs/rescue_platform_mvp_plan.md)

## License

Proprietary — Rescue Platform MVP
