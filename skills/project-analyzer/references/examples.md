# Examples

Use these examples as style guidance, not rigid templates.

## Example Invocation Patterns

```text
/project-analyzer
/project-analyzer .
/project-analyzer /path/to/repo
```

## Example Program Architecture Diagram

```text
User
  │
  ├──── CLI
  │      ▼
  │    main.py
  │      ├── parse_args()
  │      └── runner.py
  │             └── service.py
  │
  └──── API
         ▼
       app.py
         ├── routes.py
         └── handlers.py
                └── storage.py
```

## Example Rich Program Architecture Diagram

```text
用户 / 调用方
  │
  ├──── HTTP API ─────────────────────────────────────────────
  │       │
  │       ▼
  │     api/app.py                    （Flask /submit, /status）
  │       │
  │       ├── before_request()        ← 生成 request_id / 注入日志上下文 / 追踪本次请求
  │       ├── authenticate()          ← 调用 api/utils.py 做鉴权 / 决定是否允许进入主流程
  │       ├── POST /submit            ← 接收任务提交请求 / 读取原始 body / 分流到任务入口
  │       └── GET  /status/<taskid>   ← 查询任务执行状态 / 读取 Celery backend 结果
  │
  │     api/utils.py                  ← 响应封装 / 鉴权 / 参数读取
  │       ├── require_auth()          ← 校验调用方身份 / 失败时直接中断请求
  │       └── response()              ← 统一输出 JSON 响应 / 约束 status-msg-data 结构
  │
  ├──── Worker ───────────────────────────────────────────────
  │       │
  │       ▼
  │     worker/__init__.py            ← Celery app 与任务注册入口
  │       ├── submit_job(payload)     ← Celery 任务入口
  │       └── celeryconfig.py         ← 队列超时 / 序列化配置
  │
  └──── 核心处理层 ────────────────────────────────────────────
          │
          ▼
        service/processor.py          ← 业务主流程协调层
          ├── normalize_payload()     ← 归一化输入结构 / 补齐默认值 / 准备后续计算上下文
          ├── run_pipeline()          ← 执行业务主流程 / 串起查询-计算-落库主链路
          ├── write_result()          ← 写回存储层 / 返回持久化结果
          └── notify_downstream()     ← 触发下游处理 / 推送结果或刷新依赖系统

外部依赖
  ├── Redis                          ← 队列 / 锁 / 状态存储
  ├── Database                       ← 结果持久化
  └── supervisor.conf                ← 进程托管与拉起
```

## Example Code Logic Diagram

```text
┌──────────────┐
│  request in  │
└──────┬───────┘
       ▼
  validate input
       ▼
 normalize payload
       ▼
 execute service
       ▼
 write / respond
```

## Example Rich Code Logic Diagram

```text
                     ┌────────────────────────────┐
                     │        调用方 / 用户         │
                     │   POST /submit 或查询状态    │
                     └──────────────┬─────────────┘
                                    │
                                    ▼
                     ┌────────────────────────────┐
                     │         api/app.py          │
                     │       Flask 路由与入口层      │
                     ├────────────────────────────┤
                     │ api/app.py:12 before_request() │
                     │   └── 生成 request_id / 记录日志 / 准备 trace 上下文 │
                     │ api/app.py:20 authenticate()   │
                     │   └── 调 api/utils.require_auth() / 决定是否继续分派 │
                     │ api/app.py:34 submit()         │
                     │   └── 读取 body / 校验参数 / 进入任务提交流程         │
                     │ api/app.py:52 fetch(taskid)    │
                     │   └── 读取 taskid / 查询 backend / 进入状态查询流程   │
                     └──────────────┬─────────────┘
                                    │
                    公共前置处理完成：日志 / 鉴权 / 路由分派
                                    │
                 ┌──────────────────┴──────────────────┐
                 │                                     │
                 ▼                                     ▼
   ┌────────────────────────────┐         ┌────────────────────────────┐
   │ api/app.py:34 submit()      │         │ api/app.py:52 fetch(taskid)│
   │ 任务提交分支入口              │         │ 状态查询分支入口              │
   └──────────────┬─────────────┘         └──────────────┬─────────────┘
                  │                                      │
                  ▼                                      ▼
   ┌────────────────────────────┐         ┌────────────────────────────┐
   │     api/data_process.py     │         │   Celery Result Backend    │
   │    请求归一化与任务分发层      │         │    状态查询与结果读取层      │
   ├────────────────────────────┤         ├────────────────────────────┤
   │ api/data_process.py:18      │         │ api/app.py:52 fetch()      │
   │ DataProcessor.submit()      │         │ 1. AsyncResult(taskid)     │
   │ 1. _normalize_payload()     │         │ 2. 读取 task.state         │
   │ 2. _select_worker()         │         │ 3. 提取 task.info / result │
   │ 3. submit_job.delay()       │         │ 4. api/utils.response()    │
   │ 4. 返回 task_id / queue 信息 │         └──────────────┬─────────────┘
   └──────────────┬─────────────┘                        │
                  │                                      │
                  ▼                                      ▼
   ┌────────────────────────────┐         ┌────────────────────────────┐
   │       worker/__init__.py    │         │      api/utils.py:30       │
   │       Celery 任务执行层       │         │ response()                 │
   ├────────────────────────────┤         │ 统一封装 JSON 响应          │
   │ worker/__init__.py:15       │         │ 返回 status / msg / data    │
   │ submit_job(payload)         │         └────────────────────────────┘
   │ 1. 初始化 Processor         │
   │ 2. 调 normalize_payload()   │
   │ 3. 调 run_pipeline()        │
   │ 4. 返回任务结果              │
   └──────────────┬─────────────┘
                  │
                  ▼
┌──────────────────────────────────────────────────────┐
│                service/processor.py                  │
│                  业务主流程执行层                     │
├──────────────────────────────────────────────────────┤
│ service/processor.py:18 normalize_payload()         │
│   ├── _extract_fields()      ← 提取输入字段          │
│   ├── _normalize_shape()     ← 归一化结构            │
│   └── _fill_defaults()       ← 补齐默认值            │
│ service/processor.py:42 run_pipeline()              │
│   ├── _build_query()         ← 生成查询条件          │
│   ├── _fetch_origin()        ← 查询原始数据          │
│   ├── _transform_result()    ← 计算核心结果          │
│   ├── write_result()         ← 使用结果写入存储      │
│   └── notify_downstream()    ← 使用结果触发通知      │
└───────────────────────┬──────────────────────────────┘
                        │
        ┌───────────────┴───────────────┐
        │                               │
        ▼                               ▼
┌────────────────────────────┐ ┌────────────────────────────┐
│ service/processor.py:70    │ │ service/processor.py:82    │
│ write_result()             │ │ notify_downstream()        │
│ 1. 写数据库                 │ │ 1. 组装通知 payload         │
│ 2. 刷新缓存                 │ │ 2. 调 HTTP Client          │
│ 3. 返回持久化结果            │ │ 3. 返回通知状态             │
└────────────────────────────┘ └────────────────────────────┘
```

## Example Code Call Graph

```markdown
# Code Call Graph

## Overview Call Graph

```text
External entrypoints                          ← Collect all upstream triggers
  │
  ├── HTTP POST /submit                       ← Start async task submission
  │   └── api/app.py:34 submit()              ← Parse request and enter submission path
  │       └── api/data_process.py:18 DataProcessor.submit()
  │           └── worker.submit_job.delay()   ← Hand off normalized payload to async worker
  │
  ├── HTTP GET /submit/<taskid>               ← Read task state or final result
  │   └── api/app.py:52 fetch(taskid)         ← Query result backend with task id
  │       └── AsyncResult(taskid)             ← Return state/result used by API response
  │
  └── Worker / Scheduler                      ← Background execution or timed trigger
      └── worker/__init__.py:15 submit_job()  ← Execute queued payload in worker context
          └── service/processor.py:42 run_pipeline()
              ├── normalize_payload()         ← Prepare stable internal data before branching
              ├── _build_query()              ← Build source lookup parameters
              ├── _fetch_origin()             ← Load source data used by later computation
              ├── _transform_result()         ← Produce business result consumed downstream
              ├── write_result()              ← Persist output and capture write status
              └── notify_downstream()         ← Trigger dependent systems with final result
```

## Call Chain 1: API Submission Flow

```text
Caller                                       ← External system or end user
  │
  ▼
api/app.py:12 before_request()               ← Create request context used by logging and tracing
  │
  ▼
api/app.py:20 authenticate()                 ← Decide whether the caller may enter business flow
  │
  ├── auth failed
  │   └── api/utils.py:30 response()         ← Return error response and stop further calls
  │
  └── auth passed
      ▼
api/app.py:34 submit()                       ← Main HTTP submission entry
  ├── request.get_data()                     ← Read raw request payload from HTTP body
  ├── _validate_payload()                    ← Check required fields before dispatch
  ├── api/data_process.py:18 DataProcessor.submit()
  │   ├── _normalize_payload()               ← Convert request data into a stable internal shape
  │   ├── _select_worker()                   ← Choose queue/worker based on request mode
  │   ├── worker.submit_job.delay()          ← Enqueue async execution and return task handle
  │   └── task.id                            ← Result handoff consumed by API layer
  └── api/utils.py:30 response()             ← Return `task_id` so caller can poll later
```

## Call Chain 2: Status Lookup Flow

```text
Caller                                       ← Polling client or UI
  │
  ▼
api/app.py:52 fetch(taskid)                  ← Enter result lookup path with known task id
  ├── AsyncResult(taskid)                    ← Read state/result from the backend store
  ├── task.state                             ← Branch on current execution state
  ├── task.info / task.result                ← Extract backend payload used in response
  ├── if state in {"PENDING", "STARTED"}     ← Return progress-style response
  ├── if state == "SUCCESS"                  ← Return final result payload
  └── api/utils.py:30 response()             ← Normalize backend state into API JSON shape
```

## Call Chain 3: Worker Execution Flow

```text
worker/__init__.py:15 submit_job(payload)      ← Celery or worker task entry
  ├── service/processor.py:18 normalize_payload()
  │   ├── _extract_fields()                    ← Pull out fields needed by later steps
  │   ├── _normalize_shape()                   ← Standardize the payload structure
  │   └── _fill_defaults()                     ← Fill optional values before execution
  ├── service/processor.py:42 run_pipeline()   ← Main worker orchestration point
  │   ├── _build_query()                       ← Construct source lookup request
  │   ├── _fetch_origin()                      ← Return source records consumed by transform step
  │   ├── _transform_result()                  ← Produce computed output and branch-specific result
  │   ├── write_result()                       ← Save computed output for storage/result reading
  │   └── notify_downstream()                  ← Trigger dependent service using final output
  └── return result                            ← Store task result for later `fetch(taskid)` calls
```

## Runtime Dependency Chain

```text
service/processor.py:42 run_pipeline()                ← Main execution chain
  ├── lib/conf_manager.py :: ConfManager.get_config() ← Read runtime configuration used by later calls
  ├── lib/utils.py :: http_post()                     ← Call an external HTTP dependency when downstream sync is required
  ├── DatabaseClient.execute()                        ← Persist data into storage for durable result tracking
  └── CacheClient.set()                               ← Refresh cache or derived state after write completes
```

## Notes On Inference

- The example above uses generic file names and functions to demonstrate the output style rather than describe any one repository.
- In real generation, prefer statically confirmed call edges from the target project whenever possible.
```

## Example Mock Stage Pattern

```markdown
# Mock Data Stages

This example uses demo values shaped from code structure rather than production data.

## Flow Chosen

`POST /submit` -> request normalization -> worker dispatch -> processor execution

## Supported Operation Types

- `STANDARD_SUBMIT`: the default path that creates or updates a task with the incoming payload
- `REWRITE_FIELDS`: an alternate path that rewrites part of the payload before execution

Both operation types are first-class branches in this example, so both are expanded below instead of reducing the second one to a short difference note.

## Stage 1: API Request Payload

### Code owner

`api/app.py`

### What happens here

The HTTP layer accepts the raw request body before validation or enrichment.

### Code-confirmed structure

```text
{
  "user_id": string,
  "items": list[object],
  "options": object,
  "request_meta"?: object
}
```

### Mock data

```json
{
  "user_id": "u_123",
  "items": [
    {"sku": "sku-1", "qty": 2},
    {"sku": "sku-2", "qty": 1}
  ],
  "options": {
    "dry_run": true,
    "priority": "high"
  },
  "request_meta": {
    "source": "dashboard"
  }
}
```

### Notes

- This stage should match the incoming request as closely as possible.
- If authentication fields or signatures exist, show them here before they are stripped or transformed.

## Stage 2: Validated Internal Params

### Code owner

`api/data_process.py`

### What changed

Validation removes HTTP-only wrapper data and keeps only the fields needed for internal processing.

### Input shape

Raw request payload from Stage 1.

### Output shape

```text
params = {
  "user_id": str,
  "items": list[dict],
  "options": dict,
  "request_id": str
}
```

### Mock data

```json
{
  "user_id": "u_123",
  "items": [
    {"sku": "sku-1", "qty": 2},
    {"sku": "sku-2", "qty": 1}
  ],
  "options": {
    "dry_run": true,
    "priority": "high"
  },
  "request_id": "a9b2d1a8-6df7-4d33-a3cc-4f9d2b6a0e51"
}
```

### Notes

- Show removed fields if that helps explain the boundary between the API layer and worker layer.
- If the code renames keys or expands aliases, call that out explicitly here.

## Stage 3: Worker Dispatch Payload

### Code owner

`worker/__init__.py` or `api/data_process.py`

### What changed

The validated params are converted into the async task payload passed to the worker backend.

### Code-confirmed structure

```text
task_kwargs = {
  "user_id": str,
  "items": list[dict],
  "options": dict,
  "request_id": str,
  "submitted_at": string
}
```

### Mock data

```json
{
  "user_id": "u_123",
  "items": [
    {"sku": "sku-1", "qty": 2},
    {"sku": "sku-2", "qty": 1}
  ],
  "options": {
    "dry_run": true,
    "priority": "high"
  },
  "request_id": "a9b2d1a8-6df7-4d33-a3cc-4f9d2b6a0e51",
  "submitted_at": "2026-04-08T10:30:00Z"
}
```

### Notes

- If queue name, route key, or countdown settings are important, document them beside this stage.
- If some values come from runtime config, label them as illustrative when the exact value is not statically proven.

## Stage 4: Processor-Enriched Payload

### Code owner

`service/processor.py`

### What changed

The worker enriches the task payload with normalized fields and execution context needed by downstream steps.

### Input shape

Worker dispatch payload from Stage 3.

### Output shape

```text
execution_payload = {
  "user_id": str,
  "items": list[dict],
  "normalized_items": list[dict],
  "options": dict,
  "request_id": str,
  "query_filters": dict,
  "execution_mode": str
}
```

### Mock data

```json
{
  "user_id": "u_123",
  "items": [
    {"sku": "sku-1", "qty": 2},
    {"sku": "sku-2", "qty": 1}
  ],
  "normalized_items": [
    {"sku": "sku-1", "qty": 2, "unit_price": 15.5},
    {"sku": "sku-2", "qty": 1, "unit_price": 28.0}
  ],
  "options": {
    "dry_run": true,
    "priority": "high"
  },
  "request_id": "a9b2d1a8-6df7-4d33-a3cc-4f9d2b6a0e51",
  "query_filters": {
    "user_id": ["u_123"],
    "status": ["active"]
  },
  "execution_mode": "async"
}
```

### Notes

- This is a good place to show fields added by helper functions such as normalization, filter-building, or runtime config loading.
- If the code preserves both original and normalized forms, show both rather than collapsing them into one object.

## `REWRITE_FIELDS` Stage 1: API Request Payload

### Code owner

`api/app.py`

### What happens here

The HTTP layer accepts a request that includes an explicit field-remapping intent.

### Code-confirmed structure

```text
{
  "user_id": string,
  "items": list[object],
  "options": object,
  "field_mapping": object,
  "mode": "REWRITE_FIELDS"
}
```

### Mock data

```json
{
  "user_id": "u_123",
  "items": [
    {"sku": "sku-1", "qty": 2},
    {"sku": "sku-2", "qty": 1}
  ],
  "options": {
    "dry_run": true,
    "priority": "high"
  },
  "field_mapping": {
    "unit_price": "display_price"
  },
  "mode": "REWRITE_FIELDS"
}
```

## `REWRITE_FIELDS` Stage 2: Validated Internal Params

### Code owner

`api/data_process.py`

### What changed

The API layer preserves the remapping directive while removing HTTP-only wrapper fields.

### Mock data

```json
{
  "user_id": "u_123",
  "items": [
    {"sku": "sku-1", "qty": 2},
    {"sku": "sku-2", "qty": 1}
  ],
  "options": {
    "dry_run": true,
    "priority": "high"
  },
  "field_mapping": {
    "unit_price": "display_price"
  },
  "mode": "REWRITE_FIELDS",
  "request_id": "a9b2d1a8-6df7-4d33-a3cc-4f9d2b6a0e51"
}
```

## `REWRITE_FIELDS` Stage 3: Worker Dispatch Payload

### Code owner

`worker/__init__.py` or `api/data_process.py`

### What changed

The task payload carries both the normalized request fields and the remapping plan into async execution.

### Mock data

```json
{
  "user_id": "u_123",
  "items": [
    {"sku": "sku-1", "qty": 2},
    {"sku": "sku-2", "qty": 1}
  ],
  "options": {
    "dry_run": true,
    "priority": "high"
  },
  "field_mapping": {
    "unit_price": "display_price"
  },
  "mode": "REWRITE_FIELDS",
  "request_id": "a9b2d1a8-6df7-4d33-a3cc-4f9d2b6a0e51",
  "submitted_at": "2026-04-08T10:30:00Z"
}
```

## `REWRITE_FIELDS` Stage 4: Processor Rewrite Payload

### Code owner

`service/processor.py`

### What changed

`run_pipeline()` chooses the rewrite helper and builds a payload that contains both the original normalized data and the rewritten preview.

### Output shape

```text
rewrite_payload = {
  "user_id": str,
  "items": list[dict],
  "normalized_items": list[dict],
  "field_mapping": dict[str, str],
  "rewrite_preview": list[dict],
  "request_id": str
}
```

### Mock data

```json
{
  "user_id": "u_123",
  "items": [
    {"sku": "sku-1", "qty": 2},
    {"sku": "sku-2", "qty": 1}
  ],
  "normalized_items": [
    {"sku": "sku-1", "qty": 2, "unit_price": 15.5},
    {"sku": "sku-2", "qty": 1, "unit_price": 28.0}
  ],
  "field_mapping": {
    "unit_price": "display_price"
  },
  "rewrite_preview": [
    {"sku": "sku-1", "display_price": 15.5},
    {"sku": "sku-2", "display_price": 28.0}
  ],
  "request_id": "a9b2d1a8-6df7-4d33-a3cc-4f9d2b6a0e51"
}
```

## `REWRITE_FIELDS` Stage 5: Final Result Payload

### Code owner

`service/processor.py`

### What changed

The processor returns a result that emphasizes rewritten fields rather than only the default computed summary.

### Mock data

```json
{
  "request_id": "a9b2d1a8-6df7-4d33-a3cc-4f9d2b6a0e51",
  "status": "success",
  "rewritten_fields": ["display_price"],
  "result_preview": [
    {"sku": "sku-1", "display_price": 15.5},
    {"sku": "sku-2", "display_price": 28.0}
  ]
}
```

### Notes

- When a repository supports two or more main modes, generated docs should expand each major mode at roughly this level instead of leaving one mode as a short appendix.

## Stage 5: Final Result Or Write Payload

### Code owner

`service/processor.py`

### What changed

The processor turns the enriched payload into the final result returned to storage, a downstream service, or the API caller.

### Mock data

```json
{
  "request_id": "a9b2d1a8-6df7-4d33-a3cc-4f9d2b6a0e51",
  "status": "success",
  "summary": {
    "item_count": 2,
    "computed_total": 59.0
  },
  "write_payload": {
    "user_id": "u_123",
    "total": 59.0,
    "processed_at": "2026-04-08T10:30:03Z"
  }
}
```

### Notes

- If the project has both a storage payload and an API response payload, show them as separate sub-stages.
- When some downstream fields are derived from runtime config or third-party responses, mark those values as illustrative.
```

## Tone

- Clear and concrete
- Technical but readable
- No filler
- Honest about inferred parts
- When a repository already has a stronger visual/documentation style, follow that style instead of these minimal examples
