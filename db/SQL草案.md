# SQL 草案

## 1. 说明

本文档提供用于 OpenHarmony RDB 设计参考的 SQL 草案，主要用于文档设计阶段，不代表最终工程代码。

## 2. 建表 SQL

### 2.1 `device`

```sql
CREATE TABLE IF NOT EXISTS device (
  id TEXT PRIMARY KEY,
  name TEXT NOT NULL,
  type TEXT NOT NULL,
  category TEXT NOT NULL,
  online_status INTEGER NOT NULL,
  current_status TEXT,
  location TEXT,
  extra_json TEXT,
  created_at INTEGER NOT NULL,
  updated_at INTEGER NOT NULL
);
```

### 2.2 `sensor_reading`

```sql
CREATE TABLE IF NOT EXISTS sensor_reading (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  device_id TEXT NOT NULL,
  metric_type TEXT NOT NULL,
  value REAL NOT NULL,
  unit TEXT,
  collected_at INTEGER NOT NULL
);
```

### 2.3 `alert_rule`

```sql
CREATE TABLE IF NOT EXISTS alert_rule (
  id TEXT PRIMARY KEY,
  name TEXT NOT NULL,
  metric_type TEXT NOT NULL,
  operator TEXT NOT NULL,
  threshold_value REAL NOT NULL,
  level TEXT NOT NULL,
  enabled INTEGER NOT NULL,
  created_at INTEGER NOT NULL,
  updated_at INTEGER NOT NULL
);
```

### 2.4 `alert_event`

```sql
CREATE TABLE IF NOT EXISTS alert_event (
  id TEXT PRIMARY KEY,
  rule_id TEXT,
  device_id TEXT,
  metric_type TEXT NOT NULL,
  level TEXT NOT NULL,
  message TEXT NOT NULL,
  status TEXT NOT NULL,
  triggered_value REAL,
  created_at INTEGER NOT NULL,
  resolved_at INTEGER
);
```

### 2.5 `automation_rule`

```sql
CREATE TABLE IF NOT EXISTS automation_rule (
  id TEXT PRIMARY KEY,
  name TEXT NOT NULL,
  condition_json TEXT NOT NULL,
  action_json TEXT NOT NULL,
  enabled INTEGER NOT NULL,
  created_at INTEGER NOT NULL,
  updated_at INTEGER NOT NULL
);
```

### 2.6 `command_record`

```sql
CREATE TABLE IF NOT EXISTS command_record (
  id TEXT PRIMARY KEY,
  device_id TEXT NOT NULL,
  command_type TEXT NOT NULL,
  request_payload TEXT,
  result_status TEXT NOT NULL,
  result_message TEXT,
  source TEXT NOT NULL,
  created_at INTEGER NOT NULL
);
```

## 3. 索引 SQL

```sql
CREATE INDEX IF NOT EXISTS idx_device_type ON device(type);
CREATE INDEX IF NOT EXISTS idx_device_online_status ON device(online_status);

CREATE INDEX IF NOT EXISTS idx_sensor_device_metric_time
ON sensor_reading(device_id, metric_type, collected_at);

CREATE INDEX IF NOT EXISTS idx_alert_level_status_time
ON alert_event(level, status, created_at);

CREATE INDEX IF NOT EXISTS idx_command_device_time
ON command_record(device_id, created_at);
```

## 4. 初始化数据草案

### 4.1 初始化设备

```sql
INSERT INTO device (id, name, type, category, online_status, current_status, location, created_at, updated_at)
VALUES
('sensor-temp-001', '温湿度传感器1', 'TEMP_HUMIDITY_SENSOR', 'SENSOR', 1, 'NORMAL', 'A区', 1710000000000, 1710000000000),
('sensor-light-001', '光照传感器1', 'LIGHT_SENSOR', 'SENSOR', 1, 'NORMAL', 'A区', 1710000000000, 1710000000000),
('sensor-co2-001', '二氧化碳传感器1', 'CO2_SENSOR', 'SENSOR', 1, 'NORMAL', 'A区', 1710000000000, 1710000000000),
('sensor-soil-001', '土壤湿度传感器1', 'SOIL_SENSOR', 'SENSOR', 1, 'NORMAL', 'A区', 1710000000000, 1710000000000),
('pump-001', '主灌溉水泵', 'PUMP', 'ACTUATOR', 1, 'OFF', 'A区', 1710000000000, 1710000000000),
('valve-001', '主灌溉水阀', 'VALVE', 'ACTUATOR', 1, 'CLOSED', 'A区', 1710000000000, 1710000000000),
('led-001', '补光灯1', 'LED', 'ACTUATOR', 1, 'OFF', 'A区', 1710000000000, 1710000000000),
('fan-001', '风扇1', 'FAN', 'ACTUATOR', 1, 'OFF', 'A区', 1710000000000, 1710000000000);
```

### 4.2 初始化告警规则

```sql
INSERT INTO alert_rule (id, name, metric_type, operator, threshold_value, level, enabled, created_at, updated_at)
VALUES
('alert-rule-001', '温度过高预警', 'TEMPERATURE', '>', 30, 'WARNING', 1, 1710000000000, 1710000000000),
('alert-rule-002', '土壤湿度过低预警', 'SOIL_MOISTURE', '<', 35, 'WARNING', 1, 1710000000000, 1710000000000),
('alert-rule-003', '光照不足预警', 'LIGHT', '<', 200, 'WARNING', 1, 1710000000000, 1710000000000);
```

### 4.3 初始化联动规则

```sql
INSERT INTO automation_rule (id, name, condition_json, action_json, enabled, created_at, updated_at)
VALUES
('auto-rule-001', '土壤湿度低自动灌溉', '{"metricType":"SOIL_MOISTURE","operator":"<","thresholdValue":35}', '{"deviceId":"pump-001","command":"TURN_ON"}', 1, 1710000000000, 1710000000000),
('auto-rule-002', '光照不足自动补光', '{"metricType":"LIGHT","operator":"<","thresholdValue":200}', '{"deviceId":"led-001","command":"TURN_ON","params":{"level":3}}', 1, 1710000000000, 1710000000000);
```

## 5. 推荐查询 SQL

### 5.1 查询首页最新环境数据

```sql
SELECT metric_type, value, collected_at
FROM sensor_reading
WHERE collected_at = (
  SELECT MAX(collected_at) FROM sensor_reading
);
```

### 5.2 查询最新告警

```sql
SELECT *
FROM alert_event
ORDER BY created_at DESC
LIMIT 10;
```

### 5.3 查询某设备最近控制记录

```sql
SELECT *
FROM command_record
WHERE device_id = ?
ORDER BY created_at DESC
LIMIT 20;
```
