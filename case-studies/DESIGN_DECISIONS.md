# Design Decisions

Key architectural decisions and their rationale.

---

## Decision 1: Firefox WebDriver vs Chromium

### Decision

Use Firefox WebDriver instead of Chromium for browser automation.

### Rationale

- **Flexibility:** Firefox often has better JavaScript execution isolation
- **Memory:** Firefox typically uses less memory than Chromium
- **Compatibility:** Handles certain portal implementations better
- **Licensing:** Mozilla's licensing more favorable for automation
- **Tradeoff:** Slightly slower than Chrome → Better resource utilization


### Alternative Considered

- Use Chromium/Chrome (faster, more common)
- Industry standard choice
- Faster rendering in many cases
- More memory consumption
- Rejected for this use case


### Implementation

- Selenium WebDriver with Firefox headless mode
- GeckoDriver for Firefox management
- Configured for Docker deployment

---

## Decision 2: Multi-Strategy Selector Fallback Chain

### Decision

Implement five-tier selector fallback instead of single hardcoded selector.

### Rationale

- **Problem:** Web page selectors break when layout changes
- **Single Selector:** Breaks on any page structure change
- **Fallback Chain:** Survives most layout variations automatically
- **Benefit:** Reduces manual maintenance, increases robustness
- **Tradeoff:** Slightly more complex code → Dramatic reliability increase


### Strategy Order

1. CSS Selector (fastest, most specific)
2. XPath (semantic, flexible)
3. Index-based (positional fallback)
4. JavaScript query (DOM manipulation)
5. jQuery chaining (cross-browser)


### Alternative Considered

- Single CSS selector with regular updates (simpler)
- Breaks on page changes
- Requires code modifications
- Rejected


### Implementation

- Try each strategy sequentially
- Stop on first success
- Log which strategy worked
- Helps identify page changes

---

## Decision 3: Adaptive Pagination Detection

### Decision

Detect and adapt to multiple pagination strategies (button-based AND direct page-jump).

### Rationale

- **Problem:** Portal may update pagination mechanism over time
- **Single Method:** Would break if portal changes approach
- **Adaptive:** Automatically switches based on what's available
- **Benefit:** Survives major UI changes
- **Tradeoff:** More code → Much more resilient


### Pagination Methods Supported

1. Button-based (Next button, Previous button)
2. Direct page-jump (#dispPage input + #btnGoto button)
3. Automatic fallback between methods


### Alternative Considered

- Hardcode one pagination method (simpler)
- Works until portal updates
- Requires code change after update
- Rejected


### Implementation

- Test for button-based navigation first
- If unavailable, test for page-jump
- Store detected method for iteration
- Continue extraction with available method

---

## Decision 4: GitHub Actions vs Bash Supervisor Loop

### Decision

Use GitHub Actions scheduled workflows instead of custom bash supervisor loop.

### Rationale

- **Reliability:** GitHub's infrastructure is more reliable than bash scripts
- **Monitoring:** Built-in GitHub Actions UI shows all executions
- **Notifications:** Error notifications integrated
- **Version Control:** Workflow definition in repository
- **Scalability:** Can trigger from multiple events
- **Tradeoff:** Less control over timing → Better reliability and visibility


### Alternative Considered

- Custom bash loop with cron (more control)
- Can implement complex backoff logic
- Harder to monitor
- Logs buried in system files
- Doesn't integrate with GitHub
- Rejected for production


### Implementation

- YAML workflow file in .github/workflows/
- Triggers every 3 hours
- Passes credentials via GitHub Secrets
- Logs to GitHub Actions UI
- Error notifications via GitHub

---

## Decision 5: Atomic Transactions vs Individual Inserts

### Decision

All related data updates happen in single atomic transaction.

### Rationale

- **Goal:** Never allow partial updates
- **Problem:** Individual inserts can leave inconsistent data
- **Solution:** Transaction wraps all related inserts
- **Benefit:** No orphaned records, guaranteed consistency
- **Tradeoff:** Slightly more complex code → Rock-solid data integrity


### Alternative Considered

- Insert related data individually
- Simpler per-insert logic
- Can result in inconsistent data
- Rejected for production


### Implementation

- BEGIN TRANSACTION at start
- Multiple INSERT operations
- COMMIT if all succeed
- ROLLBACK if any fails

---

## Decision 6: Connection Pooling vs On-Demand Connections

### Decision

Maintain connection pool instead of creating connections on demand.

### Rationale

- **Problem:** Creating new connections is expensive
- **Solution:** Maintain 2-3 persistent connections, reuse them
- **Benefit:** Reduced overhead, better quota management
- **Tradeoff:** Slightly more complex code → Significant performance improvement


### Alternative Considered

- Create connection for each operation (simpler)
- Works but wasteful
- Higher connection overhead
- Rejected


### Implementation

- Pool initialized on startup
- Connections checked out/returned explicitly
- Automatic cleanup on errors

---

## Decision 7: In-Memory Deduplication vs Database Queries

### Decision

Load existing record IDs into memory set for O(1) deduplication checks.

### Rationale

- **Problem:** Querying database for each record is slow
- **Solution:** Load IDs on startup, use in-memory set lookup
- **Benefit:** Very fast deduplication (O(1) lookup)
- **Tradeoff:** Uses memory → Eliminates redundant queries


### Alternative Considered

- Query database for each record
- Always accurate
- N+1 query problem
- Very slow
- Rejected


### Implementation

- Load IDs from database once on startup
- Store in Python set (hash table)
- O(1) lookup during extraction

---

## Decision 8: Separate Archive Database vs Single Database

### Decision

Maintain separate archive database for historical records.

### Rationale

- **Goal:** Keep live database optimized
- **Problem:** Old records bloat live database
- **Solution:** Move expired records to archive
- **Benefit:** Live DB stays small and fast
- **Tradeoff:** Slightly more operational complexity → Better performance


### Alternative Considered

- Single database (simpler)
- Gets bloated over time
- Queries slow down
- Rejected


### Implementation

- Archive database with identical schema
- Periodic job moves old records
- Archive available for historical queries

---

## Decision 9: UTF8MB4 vs UTF8 Encoding

### Decision

Use UTF8MB4 (4-byte UTF-8) instead of UTF8 (3-byte).

### Rationale

- **Data:** Contains multi-language text (Bengali, English, etc.)
- **Problem:** UTF8 only supports 3-byte characters
- **Solution:** UTF8MB4 supports full 4-byte Unicode
- **Benefit:** Supports all Unicode characters, all languages


### Alternative Considered

- UTF8 (MySQL's limited version)
- Works for basic Latin
- Fails for CJK, Arabic, etc.
- Data corruption risk
- Rejected


### Implementation

- Database: DEFAULT CHARACTER SET utf8mb4
- Connection: UTF8MB4 charset

---

## Decision 10: Foreign Key Constraints vs Application Validation

### Decision

Enforce referential integrity at database level using foreign keys.

### Rationale

- **Goal:** Prevent orphaned records
- **Database-Level:** Can't violate constraint
- **Application-Level:** Can be bypassed
- **Benefit:** Integrity guaranteed by database, not code
- **Tradeoff:** Less flexibility → Guaranteed consistency


### Alternative Considered

- Application-level validation
- More flexible
- Can accidentally bypass validation
- Data can become corrupted
- Rejected


### Implementation

- FK constraints in schema
- Database rejects invalid operations
- Application respects constraints

---

## Summary

These 10 decisions balance competing concerns:

| Decision           | Tradeoff                                  | Winner              |
| ------------------ | ----------------------------------------- | ------------------- |
| Firefox            | Memory vs compatibility                   | Firefox             |
| Selector Fallback  | Complexity vs page change resilience      | Selector Fallback   |
| Pagination Adapt   | Code complexity vs flexibility            | Pagination Adapt    |
| GitHub Actions     | Control vs reliability                    | GitHub Actions      |
| Atomic Trans       | Code simplicity vs consistency            | Atomic Trans        |
| Pooling            | Complexity vs performance                 | Pooling             |
| In-Memory Dedup    | Memory vs query speed                     | In-Memory Dedup     |
| Archive DB         | Operational complexity vs live DB perf    | Archive DB          |
| UTF8MB4            | Compatibility vs character support        | UTF8MB4             |
| FK Constraints     | Flexibility vs guaranteed integrity       | FK Constraints      |


Each decision prioritizes **reliability** and **data integrity** over simplicity.
