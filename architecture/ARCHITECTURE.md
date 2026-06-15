# System Architecture

High-level design of the limited tender scraper system.

---

## Table of Contents

1. [Overview](#overview)
2. [System Components](#system-components)
3. [Data Flow](#data-flow)
4. [Design Patterns](#design-patterns)
5. [Scalability](#scalability)

---

## Overview

The system extracts limited tender records from dynamic web portals, handling variable element selectors and pagination patterns. It uses multi-strategy element location with adaptive pagination detection to maintain reliability as the target website changes.

**Key characteristics:**

- Multi-strategy element locator (CSS → XPath → Index → JS → jQuery)
- Adaptive pagination detection (button-based and direct page-jump)
- Atomic database transactions
- GitHub Actions scheduled execution
- Connection pooling for efficiency

---

## System Components

### 1. Pagination Detector

Analyzes page structure to determine available pagination methods:

- Detects button-based pagination (click next/previous)
- Detects direct page-jump capability (#dispPage + #btnGoto)
- Adapts to available methods
- Handles both approaches seamlessly


**Timing strategy:**

- All pagination approaches respect extraction speed
- Waits for element availability before interaction
- Automatic fallback if method fails


### 2. Adaptive Locator Layer

Multi-strategy element location with automatic fallback:

- **Strategy 1:** CSS Selector (fastest)
- **Strategy 2:** XPath expression (semantic)
- **Strategy 3:** Index-based selection (positional)
- **Strategy 4:** JavaScript query (DOM manipulation)
- **Strategy 5:** jQuery chaining (cross-browser)


Stops on first successful match.

### 3. Browser Automation

Handles web interaction:

- Firefox WebDriver in headless mode
- Explicit waits for element availability
- JavaScript execution for dynamic content
- Screenshot capture for failed operations
- Graceful error recovery


### 4. Data Processing Layer

Transforms extracted data:

- Field parsing and validation
- Format normalization
- Deduplication checks
- Missing data detection
- Structure validation


### 5. Database Layer

Manages persistent storage:

- Connection pooling
- Transaction management
- Multi-table inserts
- Data integrity maintenance
- Archive database support


---

## Data Flow

```
GitHub Actions Trigger
        ↓ (every 3 hours)
  Pagination Detector
        ↓
  Adaptive Locators
        ↓ (Extract)
  Processing Layer
        ↓ (Parse & Validate)
  Database Layer
        ↓
  Live Database (Active Records)
        ↓ (Archive)
  Archive Database (Historical)

```

### Detailed Flow

1. **Initialization**

  - Load existing record IDs (for deduplication)
  - Verify database connectivity
  - Check page structure for pagination method
  - Select appropriate pagination strategy

2. **Pagination Detection**

  - Test for button-based navigation
  - Test for direct page-jump method
  - Select available strategy
  - Prepare for iteration

3. **Extraction**

  - Iterate through pages (using detected method)
  - For each record, apply multi-strategy locators
  - Extract all required fields
  - Handle extraction failures gracefully

4. **Validation**

  - Parse extracted fields
  - Validate field values
  - Check for required fields
  - Compare against schema

5. **Storage**

  - Begin atomic transaction
  - Insert into main entity table
  - Insert related entity records
  - Commit (all or nothing)

6. **Post-Processing**

  - Archive expired records
  - Run integrity checks
  - Generate status report
  - Log operation details

---

## Design Patterns

### Pattern 1: Selector Fallback Chain

**Problem:** Web selectors break when page structure changes.

**Solution:** Try multiple selector strategies sequentially, using first that succeeds.

**Benefits:**

- Survives page layout changes
- Handles different page versions
- No manual updates needed
- Graceful degradation


### Pattern 2: Pagination Adaptation

**Problem:** Different pages use different pagination approaches.

**Solution:** Detect available pagination methods and choose appropriate one.

**Benefits:**

- Works with button-based navigation
- Works with direct page-jump
- Automatically adapts to page changes
- No hardcoded assumptions


### Pattern 3: Atomic Transactions

**Problem:** Partial updates could leave data inconsistent.

**Solution:** Group all related updates in single transaction.

**Benefits:**

- No orphaned records
- Guaranteed consistency
- No partial states
- Rollback on any error


### Pattern 4: Connection Pooling

**Problem:** Creating new connections is expensive.

**Solution:** Maintain pool of reusable persistent connections.

**Benefits:**

- Reduced connection overhead
- Better resource utilization
- Respects external quotas


### Pattern 5: CI/CD Integration

**Problem:** Manual scheduling and execution is error-prone.

**Solution:** GitHub Actions provides reliable scheduled execution.

**Benefits:**

- Automatic execution every 3 hours
- Version-controlled workflow
- Error notifications built-in
- No manual intervention


---

## Database Design

### Data Model

Normalized relational schema with multiple related entities:

**Live Database:**

- Limited Tender: Core tender records
- Tender Details: Supporting information
- Bidding Info: Bidding-related data
- Contact Data: Vendor/organization details
- Attachment Refs: Document references


**Archive Database:**

- Same schema as live database
- Holds historical/expired records
- Keeps live database optimized


### Integrity Constraints

- Foreign key constraints between all related entities
- Referential integrity enforced at database level
- Atomic transactions ensure consistency
- Automatic validation layer catches edge cases


### Design Rationale

**Normalization:** Separate tables reduce duplication and improve consistency.

**Relationships:** Foreign keys prevent orphaned data.

**Atomicity:** All updates in single transaction prevent partial states.

**Archival:** Separate database keeps live DB optimized.

---

## Error Handling

### Error Categories

1. **Selector Errors** (Element not found)

  - Try next selector strategy
  - Log failure pattern
  - Capture screenshot
  - Skip record if all fail

2. **Pagination Errors** (Navigation failure)

  - Attempt fallback pagination method
  - Log which method failed
  - Continue with working method
  - Alert if both methods fail

3. **Data Errors** (Invalid format, missing field)

  - Log with context
  - Mark record for manual review
  - Continue extraction
  - Don't stop overall process

4. **Database Errors** (Connection, transaction)

  - Retry with exponential backoff
  - Log error details
  - Rollback failed transactions
  - Notify on persistent failures

### Recovery Strategy

- Try multiple selector strategies before giving up
- Adapt pagination method if primary fails
- Skip bad records, continue processing
- Automatic retry for transient failures
- GitHub Actions handles overall scheduling

---

## Performance Considerations

### Locator Efficiency

- CSS Selectors: Fastest (~1ms)
- XPath: Slower (~5ms)
- Index: Fast (~1ms)
- JavaScript: Variable (5-20ms)
- jQuery: Variable (5-20ms)

### Pagination Performance

- Button-based: Wait time varies
- Direct page-jump: Generally faster
- Adapts to choose efficient method

### Data Integrity

- Sub-second validation for consistency checks
- Efficient deduplication with in-memory set
- Connection pooling reduces overhead

### Scalability

- Connection pooling scales better than 1:1
- Archive database prevents live DB bloat
- Normalized design minimizes redundancy
- GitHub Actions handles scheduling

---

## Security & Best Practices

- Credentials in environment variables only
- Database users have minimal required privileges
- No sensitive data in logs
- Secure error messages
- Regular dependency updates

---

## Monitoring & Observability

### Structured Logging

- Operation start/end with timestamps
- Record counts and statistics
- Error details with context
- Selector strategy choices
- Pagination method used

### Status Indicators

- GitHub Actions build status
- Workflow run logs
- Error notifications
- Exit codes for monitoring

### Health Checks

- Database connectivity verification
- Pagination detection success
- Selector availability verification
- Extraction completion rate

---

## Conclusion

The architecture balances reliability, flexibility, and efficiency through:

1. **Multi-strategy locators** to survive page changes
2. **Adaptive pagination** for flexibility
3. **Atomic transactions** to maintain consistency
4. **Connection pooling** for efficiency
5. **CI/CD integration** for reliability

This approach enables reliable, high-volume tender extraction despite dynamic page structures and variable UI patterns.
