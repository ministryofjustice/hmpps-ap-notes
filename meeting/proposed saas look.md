# System Architecture Diagram

```mermaid
graph TD
    TS[TypeScript Frontend] --> KotlinAPI[Kotlin API]
    
    KotlinAPI --> PROBATION_SERVICE[PROBATION_SERVICE<br/>Returns CRNs & staffProfileIds]
    
    KotlinAPI --> CAS1[CAS1]
    KotlinAPI --> CAS2[CAS2]
    KotlinAPI --> CPR[CPR<br/>Batch CRNS]
    KotlinAPI --> ADDRESS_SERVICE[address-service<br/>Batch CRNS ??]
    KotlinAPI --> ARNS[ARNS<br/>Batch CRNS ??]
    
    ADDRESS_SERVICE --> Phase1[Phase 1: Shim to existing system]
    ADDRESS_SERVICE --> Phase2[Phase 2: Address Data]
```ADDRESS_SERVICE --> Phase2[Phase 2: Address Data]
```

PROBATION_SERVICE return say a list of crns, and staffProfileIds, from a teamId

Option 1. have PROBATION_DIGITAL be a sync call that is fed to the async aggreagator (easier)
Option 2. have some form of topological ordering in the APIS (harder)
