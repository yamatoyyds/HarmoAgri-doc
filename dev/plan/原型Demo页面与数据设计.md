# 原型 Demo 页面与数据设计

| 项目 | 内容 |
|---|---|
| 文档版本 | V1.0 |
| 创建日期 | 2026-03-16 |
| 对齐依据 | `doc/product/PRD_智慧农业控制系统.md`、`doc/design/页面详细规格.md`、`doc/design/系统总体设计.md` |

## 1. 设计原则

本设计文档仅为原型实现服务，但必须严格服从现有正式文档：

- 不改变既有 MVP 边界。
- 不降低首页、灌溉、补光、告警、扩展设备的优先级。
- 不引入与正式文档冲突的新业务概念。
- 原型新增“开屏页”仅作为辅助展示页，不替代任何正式业务页面。

## 2. 原型页面结构

### 2.1 页面层级

```text
开屏页 Splash
  -> 主控制台 MainShell
      -> 首页总览 Dashboard
      -> 设备页 Device
      -> 灌溉页 Irrigation
      -> 光照页 Lighting
      -> 告警页 Alert
      -> 设置页 Settings
```

### 2.2 与正式文档的关系

- 开屏页：辅助展示品牌、Logo 和系统名称，不进入 FR 范围统计。
- 首页总览：对应 `FR-01`、`FR-02`、`FR-03`。
- 设备页：对应 `FR-03`、`FR-12`。
- 灌溉页：对应 `FR-04`、`FR-05`。
- 光照页：对应 `FR-06`，并为 `FR-07` 预留占位。
- 告警页：对应 `FR-08`、`FR-09`。
- 设置页：承接网关配置、阈值规则、联动规则和扩展设备配置占位。

## 3. 组件规划

### 3.1 布局组件

- `SplashView`：品牌展示、启动过渡。
- `MainShell`：顶部状态栏 + 左侧导航 + 内容区。
- `SectionPanel`：统一面板容器。
- `StatusBadge`：在线、离线、告警、处理中等状态标签。

### 3.2 业务展示组件

- `MetricCard`：温度、湿度、光照、二氧化碳、土壤湿度卡片。
- `DeviceCard`：设备列表项。
- `DeviceDetailPanel`：设备详情与控制区。
- `AlertRow`：告警列表项。
- `RecordRow`：控制记录项。

### 3.3 交互组件

- `NavButton`：左侧导航按钮。
- `ScenarioChip`：演示场景切换按钮。
- `PrimaryActionButton`：主控制按钮。
- `OptionChip`：亮度、时长等离散选择。

## 4. 页面字段映射

### 4.1 首页总览

- 顶部状态栏：网关状态、更新时间、当前场景。
- 环境指标：温度、湿度、光照、二氧化碳、土壤湿度。
- 设备摘要：在线设备数、离线设备数、扩展设备数。
- 告警摘要：当前告警数量、最高等级告警。
- 快捷入口：灌溉、补光、告警、设置。

### 4.2 设备页

- 筛选维度：全部、在线、离线。
- 设备字段：名称、类型、在线状态、当前状态、位置。
- 详情字段：最近状态、可执行动作、最近操作记录。

### 4.3 灌溉页

- 土壤湿度值。
- 灌溉建议说明。
- 手动灌溉时长。
- 自动灌溉开关占位。
- 最近灌溉记录。

### 4.4 光照页

- 当前光照值。
- LED 开关状态。
- 当前亮度等级。
- 自动补光开关占位。
- 最近补光记录。

### 4.5 告警页

- 告警时间、等级、类型、内容、状态。
- 告警详情：触发值、关联设备、处理状态。

### 4.6 设置页

- host、port、protocol、wsPath。
- 阈值规则配置占位。
- 联动规则配置占位。
- 扩展设备配置占位。

## 5. Mock 数据设计

### 5.1 设计目标

- 字段命名尽量贴近正式接口与设计文档。
- 保持后续可平滑替换为真实 `service/repository`。
- 支持快速切换演示场景。

### 5.2 核心数据对象

```text
GatewayStatus
- online
- lastHeartbeat
- connectedDeviceCount

RealtimeTelemetry
- temperature
- humidity
- light
- co2
- soilMoisture
- timestamp

DeviceItem
- id
- name
- type
- category
- online
- status
- level
- location

AlertItem
- id
- level
- type
- message
- status
- triggeredValue
- deviceName
- time

CommandRecord
- id
- deviceId
- commandType
- resultStatus
- resultMessage
- source
- time
```

### 5.3 场景模型

建议内置以下场景：

- `NORMAL`：正常运行。
- `LOW_SOIL`：土壤湿度过低。
- `LOW_LIGHT`：光照过低。
- `GATEWAY_OFFLINE`：网关离线。
- `DEVICE_OFFLINE`：关键设备离线。
- `CRITICAL_ALERT`：高温或高 CO2 告警。
- `PARTIAL_ERROR`：部分指标缺失。

## 6. 开屏页设计说明

由于用户新增要求，原型增加开屏页，但处理原则如下：

- 开屏页只负责展示 `logo + 名称 + 系统定位`。
- 停留时间应短，默认自动进入主页面。
- 用户可点击跳过。
- 不承载业务配置、登录或权限流程。

此设计不与现有正式文档冲突，因为其属于展示层辅助页面，不改变业务页面结构和 MVP 范围。

## 7. 原型实现约束

- 当前实现优先采用单 `UIAbility` + 单入口页内多视图切换方式快速落地。
- 代码结构上需为后续分拆 `pages/components/services/models/state` 预留空间。
- 原型代码中 mock 逻辑与页面展示逻辑应尽量分离。

## 8. 验收检查点

- 开屏页能正确过渡到主控制台。
- 首页能在首屏展示 5 项核心环境指标。
- 灌溉和补光页均可演示提交中、成功、失败、离线禁用。
- 告警页可展示 `NEW/ACK/RESOLVED` 状态。
- 设置页可展示网关与规则配置占位。
- 至少 1 类扩展设备在设备页中可见。
