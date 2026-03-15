# Person Event Listener -> Domain Events Flow

## New functionality (these are the changes)

The following components were added to bridge `PersonEventListener` with the domain events topic:

- **PublishPerson\*** events (`PublishPersonCreated`, `PublishPersonUpdated`, `PublishPersonDeleted`) - new Spring events published by `PersonEventListener`
- **PersonRecordPublisherListener** - new listener that consumes `PublishPerson*` events and delegates to `PersonRecordPublisher`
- **PersonRecordPublisher** – publishes `PersonDomainEvent` payloads to the `domainevents` topic

These are highlighted in the diagram below.

---

## Mermaid Diagram

```mermaid
flowchart TB
    subgraph Publishers["Event Publishers (Spring ApplicationEventPublisher)"]
        PersonService["PersonService"]
        DeletionService["DeletionService"]
    end

    subgraph SpringEvents["Spring Events Published"]
        PersonCreated["PersonCreated"]
        PersonUpdated["PersonUpdated"]
        PersonDeleted["PersonDeleted"]
    end

    subgraph PersonEventListener["PersonEventListener"]
        onPersonCreated["onPersonCreated()"]
        onPersonUpdated["onPersonUpdated()"]
        onPersonDeleted["onPersonDeleted()"]
    end

    subgraph NewEvents["New Spring Events (Publish*)"]
        PublishPersonCreated["PublishPersonCreated"]
        PublishPersonUpdated["PublishPersonUpdated"]
        PublishPersonDeleted["PublishPersonDeleted"]
    end

    subgraph PersonRecordPublisherListener["PersonRecordPublisherListener"]
        onPersonCreatedForDomainEvent["onPersonCreatedForDomainEvent()"]
        onPersonUpdatedForDomainEvent["onPersonUpdatedForDomainEvent()"]
        onPersonDeletedForDomainEvent["onPersonDeletedForDomainEvent()"]
    end

    subgraph PersonRecordPublisher["PersonRecordPublisher"]
        publishPersonCreated["publishPersonCreated()"]
        publishPersonUpdated["publishPersonUpdated()"]
        publishPersonDeleted["publishPersonDeleted()"]
    end

    subgraph DomainEventsTopic["domainevents topic (SNS)"]
        Topic["Topic"]
    end

    PersonService -->|"publishEvent()"| PersonCreated
    PersonService -->|"publishEvent()"| PersonUpdated
    DeletionService -->|"publishEvent()"| PersonDeleted

    PersonCreated --> onPersonCreated
    PersonUpdated --> onPersonUpdated
    PersonDeleted --> onPersonDeleted

    onPersonCreated -->|"publishEvent()"| PublishPersonCreated
    onPersonUpdated -->|"publishEvent()"| PublishPersonUpdated
    onPersonDeleted -->|"publishEvent()"| PublishPersonDeleted

    PublishPersonCreated --> onPersonCreatedForDomainEvent
    PublishPersonUpdated --> onPersonUpdatedForDomainEvent
    PublishPersonDeleted --> onPersonDeletedForDomainEvent

    onPersonCreatedForDomainEvent --> publishPersonCreated
    onPersonUpdatedForDomainEvent --> publishPersonUpdated
    onPersonDeletedForDomainEvent --> publishPersonDeleted

    publishPersonCreated -->|"publish()"| Topic
    publishPersonUpdated -->|"publish()"| Topic
    publishPersonDeleted -->|"publish()"| Topic

    %% Highlight new components
    style NewEvents fill:#e8f5e9,stroke:#2e7d32,stroke-width:2px
    style PersonRecordPublisherListener fill:#e8f5e9,stroke:#2e7d32,stroke-width:2px
    style PersonRecordPublisher fill:#e8f5e9,stroke:#2e7d32,stroke-width:2px
```

## Class References

| Class | Role |
|-------|------|
| `PersonService` | Publishes `PersonCreated` and `PersonUpdated` when processing person create/update |
| `DeletionService` | Publishes `PersonDeleted` when processing person deletion |
| `PersonEventListener` | `@EventListener` for PersonCreated/Updated/Deleted; publishes `RecordPersonTelemetry`, `RecordEventLog`, and `PublishPerson*` events |
| `PersonRecordPublisherListener` | `@EventListener` for `PublishPerson*` events; delegates to `PersonRecordPublisher` |
| `PersonRecordPublisher` | Publishes `PersonDomainEvent` to `domainevents` topic with event types: (See below) |

## Event Types (Domain Events Topic)

- `core-person-record.person.created`
- `core-person-record.person.updated`
- `core-person-record.person.deleted`
