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
