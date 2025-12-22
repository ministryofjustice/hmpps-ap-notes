# State of Play

This document outlines the development progress up to the March release. Development will continue through summer, with a value assessment planned for March.

## Christmas 2025

![Markdown Logo](christmas-2025.png "Christmas 2025")

**Current State:**
- Macro Tracker: Renders data
- Micro Tracker: Renders minimal data, including referral histories (excluding DTR and CRS)

### Aggregator Phase 1

![Markdown Logo](agg1-christmas-2025.png "Aggregator phase 1")

**Status:** In place and pulling real data (not yet filtered by logged-in user).

**Purpose:** Enables multiple asynchronous API calls simultaneously. Supports concurrency but lacks validation, error handling, and resilience. Works only in happy-path scenarios.

### Eligibility Rules Engine Phase 1

![Markdown Logo](rules1-christmas-2025.png "Eligibility Rules Engine Phase 1")

**Status:** Successfully processing CAS1 rules. Viable for micro tracker, not macro tracker.

**Purpose:** Basic rules engine that applies rules to datasets to provide indicators and guide business processes.

### Probation Integration

**Status:** Initial work identifying requirements and changes needed to access data from the probation integration team. Collaboration begins in January.

### Addresses

**Status:** Address data shape still being defined. Expected finalization in early to mid-January. Can run entirely parallel to single accommodation development work.

## January 2026

![Markdown Logo](arch-january-2026.png "January 31st")

**Target State:** Infrastructure should be taking shape. UI functional with mix of real and mock data. May be missing proposed accommodations and/or DTR entry. Accommodation service added.

### Aggregator Phase 1.5

![Markdown Logo](agg1-6-january-2026.png "January 31st")

**Target:** Support graceful error handling and partial completions.

### Eligibility Rules Engine

**Target:** Processing CAS3 rules, potentially CRS (depending on probation integration). Defining more business behavior using pull technique (not viable for go-live). Work on better solution started.

### Probation Integration

**Target:** Endpoints providing CRN data for logged-in users. Integration with probation team ongoing.

### Addresses

**Target:** Ready to start as a work item. Can run parallel to other development.

## February 2026

![Markdown Logo](arch-February.png "February 28th")

**Target State:** 
- Application retrieves CRNs from probation integration service
- All infrastructure in place
- UI functional (only address data mocked)
- DTR backend investigation underway
- Proposed accommodation flow taking shape (address work excluded)

### Aggregator Phase 2

![Markdown Logo](agg-february.png "February 28th")

**Status:** MVP complete.

**Features:**
- Concurrent API calls
- Caching support
- Error handling with retry and timeout functionality

### Eligibility Rules Engine Phase 1.5

**Status:** Supports CAS1 and CAS3 rules. CRS support depends on probation integration. All other data mocked.

**Limitations:** Still runs on-demand (pull technique). Phase 2 development started.

### Accommodation Service

**Status:** Service added with support for:
- Address data (from Delius)
- Proposed accommodation
- Next accommodation

## Post February

![Markdown Logo](rules future.png "Post February")

### Eligibility Rules Engine Phase 2

**Target:** Event-driven architecture replacing on-demand pull technique.

**Key Features:**
- **Event Processing:** Events from MOJEvents queue passed to Rules Data Context
- **Reference Data Generation:** Rules Data Context generates reference data for the Rules Engine
- **Fresh Data Handling:** When events require fresh data, aggregator fetches and updates records
- **Automatic Rule Execution:** Updated data triggers rules engine to run and store results
- **Data Storage:** Operational data and rules results stored in ReferenceDB

**Limitations:**
- DTR (Non-standard interventions) not supported
- CRS not supported

**Architecture:**
- Event-driven flow: MOJEvents → SAS Service → Rules Data Context
- Integration with Aggregator for fresh data retrieval
- Results persisted in ReferenceDB (ReferenceData table with CRN and rules_results)


### Address 

The address phases represent a migration strategy from using Delius as the source of truth for address data to becoming the master data source ourselves. Each phase builds upon the previous one, gradually reducing dependency on Delius while improving data ownership and quality.

## Phase 1: Reference-Only Model

![Markdown Logo](address1.png "Address Phase 1")

**Key Characteristics:**
- **Data Storage**: Only stores the `delius_address_key` (UUID) in the Accommodation Table
- **External Dependency**: Calls Delius Address (search) API to retrieve address data
- **Data Ownership**: No local copy of address data; always fetches from Delius

**Architecture:**
- Proposed Accommodation component calls Delius Address (search) via HTTP
- Database stores only the reference key, not the actual address data
- Address data is always retrieved on-demand from Delius

**Benefits:**
- Minimal data storage requirements
- Always uses the latest data from Delius
- Simple implementation

**Limitations:**
- Requires Delius API to be available for every address lookup
- No offline capability
- Performance depends on Delius API response times

---

## Phase 2: Local Copy with Event-Driven Updates

![Markdown Logo](address2.png "Address Phase 2")

**Key Characteristics:**
- **Data Storage**: Stores both `delius_address_key` and `address_data` (VARCHAR) locally
- **External Dependency**: Still calls Delius Address (search) API
- **Event-Driven Updates**: Listens to `ReadEvent` from MOJEvents to update local copy when Delius data changes

**Architecture:**
- Proposed Accommodation component still calls Delius Address (search) via HTTP
- Database now stores a local copy of address data
- Accommodation Service listens to `ReadEvent` messages from MOJEvents
- When address data changes in Delius, events trigger updates to the local copy

**Benefits:**
- Local copy provides faster access to address data
- Can operate with cached data even if Delius API is temporarily unavailable
- Event-driven updates keep local copy synchronized

**Limitations:**
- Still dependent on Delius as the source of truth
- Requires event infrastructure (MOJEvents) to maintain synchronization
- Potential for data inconsistency if events are missed

---

## Phase 3: Master Data Source

![Markdown Logo](address3.png "Address Phase 3")

**Key Characteristics:**
- **Data Ownership**: Now the master for address data; no longer calls Delius
- **Data Storage**: Changed from `delius_address_key` to `accommodation_guid` as the identifier
- **Event Publishing**: Publishes `AddressEvent` to MOJEvents when addresses are created or updated
- **Event Consumption**: Still listens to `ReadEvent` from MOJEvents

**Architecture:**
- Removed Delius Address (search) component and HTTP calls
- Proposed Accommodation component is now the source of truth
- Accommodation Service publishes `AddressEvent` messages to MOJEvents when addresses change
- Accommodation Service continues to listen to `ReadEvent` messages

**Benefits:**
- Complete independence from Delius
- Full control over address data quality and structure
- Other services can consume address events from MOJEvents
- Uses our own identifier (`accommodation_guid`) instead of Delius keys

**Changes from Phase 2:**
- Removed dependency on Delius Address (search) API
- Changed primary identifier from `delius_address_key` to `accommodation_guid`
- Added `AddressEvent` publishing capability
- No longer fetches data from external source

---

## Phase 4: Data Migration and Cleansing

![Markdown Logo](address4.png "Address Phase 4")

**Key Characteristics:**
- **Data Quality**: Acknowledges that migrated Delius data has poor quality
- **Migration Required**: Need to migrate and cleanse existing address data
- **Post-Release Work**: Data cleansing exercise can be fixed post-release
- **Same Architecture**: Maintains Phase 3 architecture (master data source)

**Architecture:**
- Same as Phase 3 (master data source)
- Accommodation Table contains migrated Delius data that needs cleansing
- Data quality improvement is identified as a post-release activity

**Benefits:**
- Maintains independence from Delius
- Clear acknowledgment of data quality issues
- Structured approach to data improvement

**Challenges:**
- Existing data quality issues from Delius migration
- Requires data cleansing and migration strategy
- Post-release work means system can go live with known data quality issues

**Next Steps:**
- Develop data cleansing rules and processes
- Plan migration strategy for improving data quality
- Implement validation and quality checks for new address data

---

## Phase Comparison Summary

| Aspect | Phase 1 | Phase 2 | Phase 3 | Phase 4 |
|--------|---------|---------|---------|---------|
| **Delius Dependency** | High (API calls) | High (API calls + events) | None | None |
| **Data Storage** | Key only | Key + local copy | Full data (own ID) | Full data (own ID) |
| **Data Ownership** | Delius | Delius | Ours | Ours |
| **Event Publishing** | No | No | Yes (AddressEvent) | Yes (AddressEvent) |
| **Event Consumption** | No | Yes (ReadEvent) | Yes (ReadEvent) | Yes (ReadEvent) |
| **Primary Identifier** | delius_address_key | delius_address_key | accommodation_guid | accommodation_guid |
| **Data Quality** | N/A | N/A | Assumed good | Poor (needs cleansing) |

## Migration Path

The phases represent a clear migration path:

1. **Phase 1 → Phase 2**: Add local caching to improve performance while maintaining Delius dependency
2. **Phase 2 → Phase 3**: Break dependency on Delius and become the master data source
3. **Phase 3 → Phase 4**: Acknowledge and plan for data quality improvements on migrated data

## Key Decisions

- **Phase 2**: Decision to store local copies while still depending on Delius
- **Phase 3**: Decision to break away from Delius and become the master
- **Phase 4**: Decision to accept data quality issues initially and address them post-release

## Diagrams

[Image URLs to be added later for each phase diagram]
