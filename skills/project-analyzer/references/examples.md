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
  │     api/app.py   （Flask /submit, /status）
  │       │
  │       ├── 参数校验 / 鉴权
  │       │     └── api/utils.py
  │       │           ├── require_auth()
  │       │           └── response()
  │       │
  │       └── 异步投递任务
  │             └── worker.submit_job  →  Queue: jobs:default
  │
  ├──── Worker ───────────────────────────────────────────────
  │       │
  │       ▼
  │     worker/__init__.py
  │       ├── submit_job(payload)
  │       └── celeryconfig.py
  │
  └──── 核心处理层 ────────────────────────────────────────────
          │
          ▼
        service/processor.py
          ├── normalize_payload()
          ├── run_pipeline()
          ├── write_result()
          └── notify_downstream()
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
                         ┌──────────────────────┐
                         │     调用方 / 用户      │
                         │   POST /submit        │
                         └──────────┬───────────┘
                                    │
                                    ▼
                         ┌──────────────────────┐
                         │      api/app.py      │
                         │      submit()        │
                         └──────────┬───────────┘
                                    │
             ┌──────────────────────┼──────────────────────┐
             │                      │                      │
             ▼                      ▼                      ▼
   ┌─────────────────┐   ┌──────────────────┐   ┌────────────────────┐
   │ 参数校验         │   │ 鉴权/响应封装      │   │ 队列投递             │
   │ validate()      │   │ api/utils.py     │   │ apply_async()      │
   └────────┬────────┘   └────────┬─────────┘   └─────────┬──────────┘
            │                     │                       │
            └─────────────────────┴───────────────────────┘
                                    │
                                    ▼
                        ┌─────────────────────────┐
                        │      worker task        │
                        │    submit_job()         │
                        └────────────┬────────────┘
                                     │
                                     ▼
                        ┌─────────────────────────┐
                        │   service/processor.py  │
                        │    run_pipeline()       │
                        └────────────┬────────────┘
                                     │
                     ┌───────────────┼────────────────┐
                     │               │                │
                     ▼               ▼                ▼
             ┌──────────────┐ ┌──────────────┐ ┌──────────────┐
             │ normalize()  │ │ write_result │ │ notify()     │
             └──────────────┘ └──────────────┘ └──────────────┘
```

## Example Mock Stage Pattern

```markdown
## Stage 2: Normalized Payload

### Code owner

`service/build_payload.py`

### Input shape

Raw request args

### Output shape

```json
{
  "user_id": "string",
  "items": [],
  "options": {}
}
```

### Mock data

```json
{
  "user_id": "u_123",
  "items": [
    {"sku": "sku-1", "qty": 2}
  ],
  "options": {
    "dry_run": true
  }
}
```
```

## Tone

- Clear and concrete
- Technical but readable
- No filler
- Honest about inferred parts
- When a repository already has a stronger visual/documentation style, follow that style instead of these minimal examples
