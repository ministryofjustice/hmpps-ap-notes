## Store payload as JSONB in PostgreSQL and map as String in the JPA entity

We will store payload data in PostgreSQL using the `jsonb` column type, while mapping the entity field as a `String` in JPA.

### Entity mapping

```kotlin
@Column(columnDefinition = "jsonb")
@JdbcTypeCode(SqlTypes.JSON)
var payload: String
```

This allows PostgreSQL to store the data in an efficient, queryable JSONB format, while keeping the application-side representation lightweight.

---

## Rationale

### Database storage (JSONB)

Using `jsonb` in PostgreSQL gives us:

* Efficient binary storage of JSON
* Ability to index and query inside JSON when required

So we keep full DB-side capability without constraining payload structure.

---

### Application representation options

There are three practical representations in the application:

#### 1) JsonNode

Loading into `JsonNode` parses into tree structure.

Pros:
* Can query unknown structures (we don't do this)
* `jsonMapper.readTree` is faster when coverting to a pojo

Cons:
* Requires full JSON parsing
* Creates many objects in memory
* Wasteful if ww don't use the full payload.

This is in my opinion way to expensive

---

#### 2) POJO mapping

Mapping directly to typed objects parses JSON into domain classes.

Pros:

* Strong typing and validation
* Convenient when payload structure is stable (Which ours will be)
* Good when fields are always used

Cons:

* Still requires full parsing
* Produces object graphs we may not need

Probably the best if the structure is heavily used.

---

#### 3) String representation (chosen approach)

Keeping the payload as a raw JSON string avoids parsing unless needed.

Pros:

* Minimal memory overhead
* Fast entity loading
* No parsing cost unless explicitly required

Cons:

* No compile-time typing
* Parsing required when payload inspection is needed

However, parsing only occurs when necessary, not on every load, and we can decide when to parse.

---

## Performance considerations

* Retrieving a string from the database is cheaper than constructing JSON trees or object graphs.
* JSON parsing cost is paid only when needed.
* Memory usage lower since we avoid building temporary object trees unnecessarily.
* For small payloads, performance differences are megligable, but avoiding unnecessary parsing improves throughput under load.

---

## Conclusion

i think
JSONB in the database
String in the entity

gives us the best form both worlds as payloads are small


