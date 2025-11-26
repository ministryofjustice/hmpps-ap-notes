# Future Manage


```mermaid
graph LR
    TS[TypeScript App] -->|API Calls| KAPI[Kotlin API]
    KAPI -->|Queries| PG[(PostgreSQL DB<br/>Old Person Data)]
```

## Proposed Solution: Redis Caching

#### Shared Redis

```mermaid
graph TB
    TS[TypeScript App] -->|1. Check Cache| Redis[(Shared Redis Cache)]
    TS -->|2. Cache Miss| KAPI[Kotlin API]
    KAPI -->|3. Query| CPR[CPR API<br/>New Person Data]
    KAPI -->|4. Set Cache| Redis
    KAPI -->|5. Return Data| TS
```

#### Non-Shared Redis

```mermaid
graph TB
    TS[TypeScript App] -->|1. Request| KAPI[Kotlin API]
    KAPI -->|2. Check Cache| Redis[(Redis Cache<br/>API Only)]
    KAPI -->|3. Query| CPR[CPR API<br/>New Person Data]
    KAPI -->|4. Set Cache| Redis
    KAPI -->|5. Return Data| TS
```

## Notes

- PostgreSQL = old person data
- CPR API = new person data
- Cache needs "live-ish" data
- Post V1: Integrate with CPR
