# Code Samples & Design Patterns

Architectural patterns and pseudocode examples demonstrating key design concepts.

---

## Pattern 1: Multi-Strategy Selector Fallback

### Problem

Web page selectors become invalid when page structure changes, causing extraction failures.

### Solution

Try multiple selector strategies sequentially until one succeeds.

### Pseudocode

```
function locateElement(strategies):
    for strategy in strategies:
        try:
            element = applyStrategy(strategy)
            if element exists:
                return element
        catch ElementNotFound:
            logFailure(strategy)
            continue
    
    raise ElementNotFoundError("All strategies exhausted")

// Usage
strategies = [
    CSS("#tender-id"),
    XPath("//table[@class='tenders']/tr[1]"),
    Index("document.querySelectorAll('tr')[0]"),
    JavaScript("$('.tender-row').first()"),
    jQuery("$('tbody').find('tr').eq(0)")
]

element = locateElement(strategies)
```

### Benefits

- Survives page layout changes
- No manual updates on each change
- Graceful fallback chain
- Production-ready reliability

---

## Pattern 2: Pagination Detection & Adaptation

### Problem

Portal may support multiple pagination methods (button-based, direct page-jump), and may switch between them.

### Solution

Auto-detect available pagination methods and select appropriate one.

### Pseudocode

```
function detectPaginationMethod():
    // Try button-based first
    if hasElement("#btnNext"):
        return "BUTTON_BASED"
    
    // Try direct page-jump
    if hasElement("#dispPage") and hasElement("#btnGoto"):
        return "PAGE_JUMP"
    
    // No pagination found
    return "NONE"

function iteratePages(pagination_method):
    current_page = 1
    
    while hasMorePages():
        data = extractDataFromPage()
        process(data)
        
        if pagination_method == "BUTTON_BASED":
            clickElement("#btnNext")
            waitForPageLoad()
        
        else if pagination_method == "PAGE_JUMP":
            setInputValue("#dispPage", current_page + 1)
            clickElement("#btnGoto")
            waitForPageLoad()
        
        current_page++

// Usage
method = detectPaginationMethod()
iteratePages(method)
```

### Benefits

- Works with different portal versions
- Automatic adaptation to UI changes
- No hardcoded assumptions
- Flexible and maintainable

---

## Pattern 3: Atomic Transaction Wrapper

### Problem

Partial updates could leave database in inconsistent state with orphaned records.

### Solution

Wrap all related inserts in single transaction that either succeeds completely or fails completely.

### Pseudocode

```
function insertTenderWithRelatedData(tender_data):
    transaction = startTransaction()
    
    try:
        // Insert primary entity
        tender_id = insertRow(
            table="limited_tenders",
            data=tender_data,
            transaction=transaction
        )
        
        // Insert related entities
        insertRow(
            table="tender_details",
            data={tender_id: tender_id, ...details},
            transaction=transaction
        )
        
        insertRow(
            table="bidding_info",
            data={tender_id: tender_id, ...bidding},
            transaction=transaction
        )
        
        insertRow(
            table="contact_data",
            data={tender_id: tender_id, ...contacts},
            transaction=transaction
        )
        
        insertRow(
            table="attachment_refs",
            data={tender_id: tender_id, ...attachments},
            transaction=transaction
        )
        
        // All succeeded, commit
        transaction.commit()
        return tender_id
        
    catch DatabaseError as e:
        // Any failure: rollback everything
        transaction.rollback()
        logError("Transaction failed: " + e)
        raise

// Usage
try:
    tender_id = insertTenderWithRelatedData(data)
    log("Tender inserted: " + tender_id)
except TransactionError:
    log("Tender insertion failed, rolled back")
```

### Benefits

- Never partial data
- Guaranteed consistency
- Automatic rollback on error
- Production-safe operation

---

## Pattern 4: Connection Pooling

### Problem

Creating new database connections for each operation is expensive and wasteful.

### Solution

Maintain small pool of persistent connections, reuse them for all operations.

### Pseudocode

```
class ConnectionPool:
    function __init__(size=3):
        this.available = Queue(size)
        this.in_use = Set()
        
        for i in range(size):
            conn = createConnection()
            this.available.enqueue(conn)
    
    function getConnection():
        conn = this.available.dequeue()  // Wait if none available
        this.in_use.add(conn)
        return conn
    
    function releaseConnection(conn):
        this.in_use.remove(conn)
        this.available.enqueue(conn)
    
    function cleanup():
        for conn in this.in_use:
            closeConnection(conn)
        while not this.available.isEmpty():
            closeConnection(this.available.dequeue())

// Usage
pool = ConnectionPool(size=3)

for record in records:
    conn = pool.getConnection()
    try:
        executeQuery(conn, "INSERT INTO tenders ...")
    finally:
        pool.releaseConnection(conn)

pool.cleanup()
```

### Benefits

- Reduced connection overhead
- Better resource utilization
- Respects external quotas
- Automatic cleanup

---

## Pattern 5: In-Memory Deduplication

### Problem

Querying database for each record during extraction is slow (N+1 queries).

### Solution

Load all existing IDs into memory set on startup, use O(1) lookup.

### Pseudocode

```
function loadExistingIds():
    existing_ids = Set()
    
    query = "SELECT tender_id FROM limited_tenders"
    results = executeQuery(query)
    
    for row in results:
        existing_ids.add(row.tender_id)
    
    return existing_ids

function isDuplicate(tender_id, existing_ids):
    return tender_id in existing_ids  // O(1) lookup

function extractAndStore(existing_ids):
    new_records = 0
    duplicate_records = 0
    
    for page in paginate():
        for record in page:
            if isDuplicate(record.id, existing_ids):
                duplicate_records++
                continue
            
            storeRecord(record)
            existing_ids.add(record.id)  // Update cache
            new_records++
    
    return {new: new_records, duplicates: duplicate_records}

// Usage
ids = loadExistingIds()
stats = extractAndStore(ids)
log("Inserted: " + stats.new + ", Skipped: " + stats.duplicates)
```

### Benefits

- Very fast deduplication (O(1))
- Eliminates redundant queries
- Trades memory for speed
- Significant performance improvement

---

## Pattern 6: Selector Chain with Logging

### Problem

When selector fails, we need to know which strategies were tried for debugging.

### Solution

Log each attempt, track which succeeded.

### Pseudocode

```
function locateElementWithLogging(element_name, strategies):
    log("Locating element: " + element_name)
    
    for i, strategy in enumerate(strategies):
        log("  Attempt " + i + ": " + strategy.name)
        
        try:
            element = applyStrategy(strategy)
            log("  ✓ SUCCESS with " + strategy.name)
            return element
        
        catch ElementNotFound:
            log("  ✗ FAILED: " + strategy.name)
            continue
    
    log("ERROR: All strategies failed for " + element_name)
    captureScreenshot("error_" + element_name)
    raise ElementNotFoundError(element_name)

// Usage
strategies = [CSS("#id"), XPath("//div"), Index("...")]
element = locateElementWithLogging("tender-row", strategies)
```

### Benefits

- Clear debugging information
- Track selector reliability
- Identify page structure changes
- Support troubleshooting

---

## Pattern 7: Graceful Error Recovery

### Problem

Single errors shouldn't stop entire extraction.

### Solution

Catch errors, log them, continue with next item.

### Pseudocode

```
function extractAllRecords():
    extracted = 0
    failed = 0
    errors = List()
    
    for page in paginate():
        try:
            for record in page:
                try:
                    data = extractRecord(record)
                    storeRecord(data)
                    extracted++
                
                catch ExtractionError as e:
                    failed++
                    errors.add({
                        record_id: record.id,
                        error: e.message,
                        timestamp: now()
                    })
                    logError("Failed to extract: " + record.id)
                    captureScreenshot("error_" + record.id)
                    continue  // Skip this record
        
        catch PageLoadError as e:
            logError("Page failed: " + e)
            break  // Exit extraction
    
    logSummary({
        extracted: extracted,
        failed: failed,
        errors: errors
    })
    
    return {success: extracted, failed: failed}

// Usage
stats = extractAllRecords()
log("Extraction complete: " + stats.success + " succeeded, " + 
    stats.failed + " failed")
```

### Benefits

- Robust error handling
- Detailed error tracking
- Graceful degradation
- No catastrophic failures

---

## Pattern 8: Resource Cleanup Guarantee

### Problem

If exception occurs, resources might not be released (connections leak).

### Solution

Use try-finally or context manager pattern.

### Pseudocode

```
function executeWithCleanup():
    conn = null
    
    try:
        conn = getConnection()
        data = executeQuery(conn, "SELECT ...")
        processData(data)
        return result
    
    catch Error as e:
        logError(e)
        throw  // Re-raise
    
    finally:
        // ALWAYS runs, even on exception
        if conn != null:
            releaseConnection(conn)

// Alternative: Context manager pattern
function withConnection():
    return ConnectionContext()

// Usage
with withConnection() as conn:
    data = executeQuery(conn, "SELECT ...")
    processData(data)
// Connection auto-released even if error occurs
```

### Benefits

- No resource leaks
- Guaranteed cleanup
- Exception-safe
- Clean code

---

## Pattern 9: Validation Chain

### Problem

Data could be invalid (missing fields, wrong format).

### Solution

Chain validations, stop on first failure.

### Pseudocode

```
function validateRecord(record):
    validators = [
        validateRequiredFields,
        validateFieldFormats,
        validateSchemaCompliance,
        validateBusinessRules
    ]
    
    for validator in validators:
        try:
            validator(record)
        catch ValidationError as e:
            return {valid: false, error: e.message}
    
    return {valid: true}

function validateRequiredFields(record):
    required = ["tender_id", "tender_title", "deadline"]
    for field in required:
        if field not in record or record[field] is null:
            raise ValidationError("Missing required field: " + field)

function validateFieldFormats(record):
    if not isValidDate(record.deadline):
        raise ValidationError("Invalid date format: " + record.deadline)
    
    if not isValidAmount(record.budget):
        raise ValidationError("Invalid amount: " + record.budget)

// Usage
result = validateRecord(data)
if result.valid:
    storeRecord(data)
else:
    logError("Validation failed: " + result.error)
```

### Benefits

- Comprehensive validation
- Clear error messages
- Prevents bad data
- Easy to extend

---

## Pattern 10: GitHub Actions Integration

### Problem

Need reliable, scheduled execution without manual intervention.

### Solution

Use GitHub Actions workflow with environment-based configuration.

### YAML Example

```yaml
name: Limited Tenders Scraper
on:
  schedule:
    - cron: '0 */3 * * *'  # Every 3 hours
  workflow_dispatch:        # Manual trigger

jobs:
  scrape:
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v3
      
      - name: Set up Python
        uses: actions/setup-python@v4
        with:
          python-version: '3.11'
      
      - name: Install dependencies
        run: |
          pip install -r requirements.txt
      
      - name: Run scraper
        env:
          DATABASE_HOST: ${{ secrets.DB_HOST }}
          DATABASE_USER: ${{ secrets.DB_USER }}
          DATABASE_PASSWORD: ${{ secrets.DB_PASSWORD }}
        run: python main.py
      
      - name: Notify on failure
        if: failure()
        uses: actions/github-script@v6
        with:
          script: |
            github.rest.issues.createComment({
              issue_number: context.issue.number,
              owner: context.repo.owner,
              repo: context.repo.repo,
              body: 'Scraper failed in run ${{ github.run_id }}'
            })
```

### Benefits

- Reliable scheduling
- Version-controlled
- Transparent logs
- Automatic notifications

---

## Summary

These patterns demonstrate:

✅ **Resilience** - Multi-strategy locators, error recovery  
✅ **Flexibility** - Adaptive pagination, configurable validators  
✅ **Reliability** - Atomic transactions, resource cleanup  
✅ **Efficiency** - Connection pooling, in-memory caching  
✅ **Observability** - Comprehensive logging, clear errors  
✅ **Automation** - GitHub Actions integration  

Each pattern balances **robustness** with **maintainability**, prioritizing production-grade reliability.

---

**For implementation details, refer to ARCHITECTURE.md and DESIGN_DECISIONS.md**
