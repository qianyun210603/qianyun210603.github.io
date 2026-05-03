---
title: Airflow 升级与配置问题处理记录
date: 2026-05-03T19:20:44+08:00
categories:
  - Linux
  - 软件
---

具体升级流程客参见[Apache-Airflow的官方文档](https://airflow.apache.org/docs/apache-airflow/3.1.8/installation/upgrading_to_airflow3.html)

## 以下是过程中遇到的问题

1. [Q1. 升级 Airflow 建议先清理数据库 `airflow db clean`，参数如何设置？](#q1)
2. [Q2. airflow.cfg中`logging/log_filename_template`是做什么的？为何在 Airflow 3 中被移除？](#q2)
3. [Q3. airflow db migrate 报错 `No module named 'airflow.auth'`](#q3)
4. [Q4. FAB Provider 安装选择：哪个兼容旧版用户密码](#q4)
5. [Q5. webserver_config.py 不适配 Airflow 3.1.8，如何修改？](#q5)
6. [Q6. jwt_secret 必须设置的错误修复](#q6)
7. [Q7. 使用 systemctl 查看完整日志的方法](#q7)
8. [Q8. 升级后运行发现很多 DeprecationWarning 如何将自动处理未覆盖的选项升级到3.1.18兼容版本？](#q8)
9. [Q9. 使用`systemctl`将 `airflow` 注册为服务，运行服务后发现未正常启动，如何用 journalctl 精简并查看日志？](#q9)
10. [Q10. 升级 Airflow 3 后出现 msgspec.DecodeError](#q10)
11. [Q11. Airflow 3 任务卡在 queued 状态](#q11)
12. [Q12. Airflow 3 中 scheduler 与 dag-processor 的关系](#q12)

<a id="q1"></a>
## Q1. 升级 Airflow 建议先清理数据库 `airflow db clean`，参数如何设置？

升级前使用 `airflow db clean` 可减小数据库体积、提升架构变更效率。

### 常用参数说明

- `--clean-before-timestamp`：必需，清除该时间点之前的数据（如 `"2025-01-25 00:00:00+08:00"`）。
- `--dry-run`：建议先执行，仅列出 SQL 不实际删除。
- `--skip-archive`：推荐，直接永久删除，不归档。
- `--yes` / `-y`：自动确认，用于自动化。
- `--tables`：指定要清理的表，逗号分隔。
- `--dag-ids` / `--exclude-dag-ids`：按 DAG ID 精准清理或排除。
- `--batch-size`：控制每批次删除行数，防止锁死。
- `--verbose` / `-v`：详细日志。

### 推荐操作步骤

1. **停机并备份**：完全停止 Airflow 服务，完整备份数据库。
2. **先预览**：

   ```bash
   airflow db clean --clean-before-timestamp "2025-12-25 00:00:00+00:00" --dry-run
   ```

3. **执行清理**：

   ```bash
   airflow db clean --clean-before-timestamp "2025-12-25 00:00:00+00:00" --skip-archive --yes
   ```

4. **超大表分批清理**：可设置 `--batch-size 10000` 重复执行。

### 常见问题与避坑

- 任务超时：改为 cron 作业或延长超时时间。
- 未知归档表：未使用 `--skip-archive` 时会产生，可运行 `airflow db drop-archived` 清理。
- 数据库无法连接：检查环境变量 `AIRFLOW__DATABASE__SQL_ALCHEMY_CONN`。

---

<a id="q2"></a>
## Q2. airflow.cfg中`logging/log_filename_template`是做什么的？为何在 Airflow 3 中被移除？

`log_filename_template` 曾用于自定义任务日志的文件名和路径模板，如：

```ini
[logging]
log_filename_template = {{ ti.dag_id }}/{{ ti.task_id }}/{{ ts }}/{{ try_number }}.log
```

### 移除原因

- **动态文件名需求**：需包含 `run_id`、`map_index` 等动态字段，旧模板机制维护成本高。
- **清理历史遗留设计**：统一内部架构，降低复杂性。
- **简化配置**：Airflow 3 目标，减少用户自定义负担。

### 迁移指南

- 调整本地日志目录：在 `[logging]` 下使用 `base_log_folder`。
- 远程日志存储（S3/GCS）：路径由各自的 Handler 管理，不受影响。
- 自定义 Handler：升级或寻找替代处理器，或通过 `airflow_local_settings.py` 覆盖（不推荐）。

---

<a id="q3"></a>
## Q3. airflow db migrate 报错 `No module named 'airflow.auth'`

**错误信息**：

```error
ModuleNotFoundError: No module named 'airflow.auth'
...
airflow.exceptions.AirflowConfigException: The object could not be loaded. 
Please check "auth_manager" key in "core" section. 
Current value: "airflow.auth.managers.fab.fab_auth_manager.FabAuthManager".
```

**原因**：`airflow.cfg` 中的 `auth_manager` 配置指向 Airflow 2 的旧路径，Airflow 3 已移除该模块。

### 解决方案

#### 方法一：保留 FAB 认证管理器（生产环境推荐）

1. **安装 FAB provider**：
   - 基础安装：`pip install apache-airflow-providers-fab`
   - 如需 LDAP：`pip install "apache-airflow-providers-fab[ldap]"`
   - 如需 OAuth2：`pip install "apache-airflow-providers-fab[oauth]"`
2. **更新配置文件** `airflow.cfg`：

   ```ini
   [core]
   auth_manager = airflow.providers.fab.auth_manager.fab_auth_manager.FabAuthManager
   ```

3. **重新运行迁移**：`airflow db migrate`

#### 方法二：切换到 SimpleAuthManager（测试环境）

修改 `[core]` 部分为：

```ini
auth_manager = airflow.providers.fab.auth_manager.fab_auth_manager.SimpleAuthManager
```

### 验证与后续

- 运行 `airflow config get-value core auth_manager` 确认新路径。
- 重启 Webserver，检查日志。
- 运行 `airflow config list --deprecated` 清理其他弃用配置。

---

<a id="q4"></a>
## Q4. FAB Provider 安装选择：哪个兼容旧版用户密码

**问题**：基础安装、LDAP 安装、OAuth2 安装，哪个能沿用 Airflow 2 原有的用户和密码？

**回答摘要**：
**基础安装**（`pip install apache-airflow-providers-fab`）即可完全兼容原有用户密码，因为用户数据仍保存在同一个数据库中，FAB Auth Manager 只是被分离成独立 Provider，表结构不变。

| 安装命令 | 用途 | 兼容旧用户密码 |
| :--- | :--- | :--- |
| `pip install apache-airflow-providers-fab` | 核心认证功能 | **是，完全兼容** |
| `pip install "apache-airflow-providers-fab[ldap]"` | 增加 LDAP 集成 | 是，用于外部账号 |
| `pip install "apache-airflow-providers-fab[oauth]"` | 增加 OAuth2 集成 | 是，用于外部账号 |

**后续步骤**：安装后重启 Webserver，并运行 `airflow fab-db migrate` 确保 FAB 表结构最新。

---

<a id="q5"></a>
## Q5. webserver_config.py 不适配 Airflow 3.1.8，如何修改？

### 推荐配置方式

- **`airflow.cfg`** 中指定认证管理器：

  ```ini
  [core]
  auth_manager = airflow.providers.fab.auth_manager.fab_auth_manager.FabAuthManager
  ```

- **`webserver_config.py`** 精简为仅微调行为：

  ```python
  from __future__ import annotations
  import os
  from flask_appbuilder.const import AUTH_OAUTH
  
  basedir = os.path.abspath(os.path.dirname(__file__))
  AUTH_TYPE = AUTH_OAUTH
  # 其他微调项如 AUTH_USER_REGISTRATION = True 可继续保留
  ```

---

<a id="q6"></a>
## Q6. jwt_secret 必须设置的错误修复

**错误信息**：

```error
ValueError: The value api_auth/jwt_secret must be set!
```

**原因**：Airflow 3.x 强制要求为 API 认证设置 JWT 密钥，用于签名和验证令牌。

### 解决方案

#### 方案一：环境变量（推荐）

1. 生成强随机密钥：

   ```bash
   openssl rand -base64 48
   ```

2. 设置环境变量：

   ```bash
   export AIRFLOW__API_AUTH__JWT_SECRET='<生成的密钥>'
   ```

#### 方案二：修改 airflow.cfg

```ini
[api_auth]
jwt_secret = z1Sd+ZC3cK9vW8xH/3mYpA2rF5tG7uJwN0qX4bV6yE=
```

#### 方案三：Docker 环境

通过 `-e AIRFLOW__API_AUTH__JWT_SECRET=...` 传递。

### 注意事项

- 所有需要 JWT 的组件（Scheduler、Worker、Triggerer）必须使用**相同**的密钥。
- 不要将密钥提交到版本控制，建议使用密钥管理服务。
- 密钥轮换会立即使现有令牌失效，需妥善规划。
- 若仍有 JWT 错误，可尝试升级 `apache-airflow-providers-fab`。

---

<a id="q7"></a>
## Q7. 使用 systemctl 查看完整日志的方法

**问题**：如何用 `systemctl` 查看服务完整日志？

**回答摘要**：
使用 `journalctl` 命令查看 systemd 服务的完整日志。

### 基本命令

```bash
journalctl -u 服务名.service
```

例如：

```bash
journalctl -u airflow-webserver.service
```

### 常用选项组合

| 选项 | 作用 | 示例 |
| :--- | :--- | :--- |
| `-e` | 跳转到日志末尾 | `journalctl -u airflow-webserver.service -e` |
| `-f` | 实时跟踪（类似 `tail -f`） | `journalctl -u airflow-webserver.service -f` |
| `-b` | 仅显示本次启动后的日志 | `journalctl -u airflow-webserver.service -b` |
| `--no-pager` | 禁用分页，一次性输出 | `journalctl -u airflow-webserver.service --no-pager` |
| `--since` / `--until` | 按时间范围过滤 | `--since "2026-05-03 13:39:00"` |
| `-o cat` | 简洁输出，无元信息 | `journalctl -u airflow-webserver.service -o cat` |

### 查找服务名

```bash
systemctl list-unit-files | grep airflow
```

### 应用示例（结合 JWT 错误排查）

```bash
sudo systemctl restart airflow-webserver
journalctl -u airflow-webserver.service -f -e --since "2026-05-03 13:39:00"
```

<a id="q8"></a>
## Q8. 升级后运行发现很多 DeprecationWarning 如何将自动处理未覆盖的选项升级到3.1.18兼容版本？

从日志中提取所有 `DeprecationWarning`，去重后共 **10 项**，归纳为**配置项迁移**和**常量弃用**两类。

### 对应关系表

| 旧配置/常量 | 新配置/常量 |
| ---------- | ---------- |
| `[webserver] grid_view_sorting_order` | `[api] grid_view_sorting_order` |
| `[scheduler] dag_dir_list_interval` | `[dag_processor] refresh_interval` |
| `[webserver] secret_key` | `[api] secret_key` |
| `[webserver] base_url` | `[api] base_url` |
| `[webserver] web_server_port` | `[api] port` |
| `[webserver] workers` | `[api] workers` |
| `[webserver] web_server_host` | `[api] host` |
| `[webserver] expose_hostname` | `[fab] expose_hostname` |
| `[webserver] page_size` | `[api] page_size` |
| `HTTP_422_UNPROCESSABLE_ENTITY` | `HTTP_422_UNPROCESSABLE_CONTENT` |

### 变更说明

- `[scheduler] dag_dir_list_interval` → `[dag_processor] refresh_interval`
- `[webserver]` 下多个选项迁移至 `[api]`，部分键重命名：
  - `web_server_port` → `port`
  - `web_server_host` → `host`
- `[webserver] expose_hostname` → `[fab] expose_hostname`
- `[core] dag_file_processor_timeout` 建议迁移至 `[dag_processor]`
- `HTTP_422_UNPROCESSABLE_ENTITY` 需在 Python 代码中手动替换为 `HTTP_422_UNPROCESSABLE_CONTENT`

### 从 [webserver] 迁移至 [api] 的完整列表

|新配置项 (API)|说明|默认值|
|---------------|------|--------|
|`base_url`|API 基础 URL|`None`|
|`default_wrap`|文本换行默认开关|`False`|
|`enable_swagger_ui`|运行 Swagger UI|`True`|
|`expose_config`|暴露配置信息|`False`|
|`expose_stacktrace`|暴露堆栈信息|`False`|
|`grid_view_sorting_order`|网格视图排序规则|`topological`|
|`hide_paused_dags_by_default`|默认隐藏暂停的 DAG|`False`|
|`host` (原 `web_server_host`)|监听地址|`0.0.0.0`|
|`log_fetch_timeout_sec`|日志获取超时|`5`|
|`page_size`|API 页面大小|`50`|
|`port` (原 `web_server_port`)|监听端口|`8080`|
|`secret_key`|签名密钥|`{SECRET_KEY}`|
|`ssl_cert` / `ssl_key`|SSL 证书/密钥路径|空|
|`worker_timeout` (原 `web_server_worker_timeout`)|Worker 超时|`120`|
|`workers`|Worker 进程数|`1`|
|`access_logfile`|已弃用，迁移至 `[logging]`|—|
|`auto_refresh_interval`|自动刷新间隔|`3`|

---

<a id="q9"></a>
## Q9. 使用`systemctl`将 `airflow` 注册为服务，运行服务后发现未正常启动，如何用 journalctl 精简并查看日志？

### 清理前检查

```bash
journalctl --disk-usage
```

### 清理方法

| 策略 | 命令示例 | 说明 |
|------|----------|------|
| 按时间清理 | `sudo journalctl --vacuum-time=2weeks` | 删除指定时间之前的日志 |
| 按大小清理 | `sudo journalctl --vacuum-size=500M` | 保留日志直到总大小低于指定值 |
| 按文件数清理 | `sudo journalctl --vacuum-files=20` | 只保留指定数量的日志文件 |

### 删除特定服务的日志

```bash
sudo journalctl -u <服务名> --rotate
sudo journalctl -u <服务名> --vacuum-time=2weeks
```

---

<a id="q10"></a>
## Q10. 升级 Airflow 3 后出现 msgspec.DecodeError

### 错误信息

```error
msgspec.DecodeError: MessagePack data is malformed: trailing characters (byte 1)
```

### 原因

Airflow 3 将 Flask session 序列化改为 MessagePack，数据库中残留旧格式 session 数据。

### 解决方法

清空 session 表：

```sql
DELETE FROM session;
-- 或 TRUNCATE TABLE session;
```

重启 Airflow Webserver 即可。

---

<a id="q11"></a>
## Q11. Airflow 3 任务卡在 queued 状态

### 原因分析

- 升级后 `dag_file_processor_timeout` 可能需迁移至 `[dag_processor]`
- 调度器与 DAG 处理器协同异常
- 资源池槽位满、执行器配置错误、僵尸任务等

### 解决方案（按优先级排序）

1. **清理状态**：`airflow tasks clear <dag_id> --dag-run-state running --task-state running --upstream --downstream`
2. **重启调度器**：`pkill -f "airflow scheduler" && airflow scheduler -D`
3. **检查资源池**：UI 中 `Admin -> Pools`，必要时增加槽位
4. **检查 CeleryExecutor**（若使用）：确认 worker 在线、broker 和 backend 连接正常
5. **调整并发/心跳参数**：增大 `parallelism`、`parsing_processes`、心跳时间等
6. **手动标记失败**：UI 中将 queued 实例标记为 failed

---

<a id="q12"></a>
## Q12. Airflow 3 中 scheduler 与 dag-processor 的关系

### 核心概念

- **Scheduler**：决定何时运行哪些任务，不直接解析 DAG 文件
- **DAG Processor**：解析 Python DAG 文件并写入数据库

### 部署模式

| 模式 | 配置 | 启动方式 |
| ------ | ------ | ---------- |
| 进程内运行（默认） | `standalone_dag_processor = False` | 只需 `airflow scheduler` |
| 分开部署 | `standalone_dag_processor = True` | 需分别启动 `airflow dag-processor` 和 `airflow scheduler` |

### 建议

开发/测试推荐默认进程内模式；生产环境推荐分开部署以确保安全隔离。
