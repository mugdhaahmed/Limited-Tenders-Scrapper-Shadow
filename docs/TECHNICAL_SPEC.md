# Technical Specification

Technical requirements, system specifications, and configuration details for the Limited Tenders Scraper.

---

## System Requirements

### Minimum Hardware

- **CPU:** 2 cores
- **RAM:** 2 GB minimum (4 GB recommended)
- **Storage:** 20 GB (for logs, screenshots, database)
- **Network:** Stable internet connection with 10+ Mbps

### Operating System

- **Linux:** Ubuntu 20.04 LTS or later
- **Docker-Compatible:** Any OS with Docker support

### Software Dependencies

- **Python:** 3.11+
- **Docker:** 20.10+
- **Docker Compose:** 2.0+ (optional)
- **MySQL:** 8.0+
- **Firefox:** Latest (headless mode)
- **GeckoDriver:** Latest compatible with Firefox

---

## Architecture Requirements

### Database

**MySQL 8.0+ (InnoDB Engine)**

- **Character Set:** UTF8MB4
- **Collation:** utf8mb4_general_ci
- **Max Connections:** 50+
- **Storage Engine:** InnoDB (for transactions)

### Connection Parameters

- **Host:** Configurable via environment variable
- **Port:** 3306 (standard MySQL)
- **User:** Dedicated database user
- **Database:** Two databases (live + archive)

### Browser Automation

**Firefox with Selenium WebDriver**

- **WebDriver:** GeckoDriver
- **Mode:** Headless (no GUI)
- **Memory Limit:** 512 MB per instance
- **Timeout:** 30 seconds for element waits

---

## Configuration

### Environment Variables

```
DATABASE_HOST=
DATABASE_USER=
DATABASE_PASSWORD=
DATABASE_NAME_LIVE=
DATABASE_NAME_ARCHIVE=
MAX_POOL_SIZE=3
ELEMENT_WAIT_TIMEOUT=30
EXTRACTION_BATCH_SIZE=100
GITHUB_ACTIONS_TOKEN=
```

### Database Configuration

**Live Database Schema:**

- `limited_tenders` - Core tender records
- `tender_details` - Supporting information
- `bidding_info` - Bidding-related data
- `contact_data` - Vendor/organization details
- `attachment_refs` - Document references

**Archive Database Schema:**

- Identical to live database
- Holds historical/expired records
- Same character set and collation

### System Configuration

**Connection Pool:**

- **Minimum Connections:** 2
- **Maximum Connections:** 3
- **Connection Timeout:** 10 seconds
- **Idle Timeout:** 300 seconds

**Extraction Settings:**

- **Element Wait Timeout:** 30 seconds
- **Element Retry Count:** 3
- **Page Load Timeout:** 60 seconds
- **Screenshot on Error:** Yes

**Pagination:**

- **Pagination Detection:** Automatic
- **Supported Methods:** Button-based, Direct page-jump
- **Fallback Strategy:** Yes

---

## Data Model

### Entity Relationships

```
Limited Tender (Primary)
    ├── Tender Details (1:1)
    ├── Bidding Info (1:N)
    ├── Contact Data (1:N)
    └── Attachment Refs (1:N)
```

### Foreign Key Constraints

- All relationships enforced at database level
- Cascade delete on parent record deletion
- Referential integrity checks enabled
- Orphan detection and repair capabilities

### Validation Rules

**Field Requirements:**

- All primary keys: Non-null, unique
- Foreign keys: Must reference existing parent
- Timestamps: Auto-set on insert/update
- Amounts: Decimal(15,2) precision
- Text fields: UTF8MB4 encoding

---

## API & Integration Points

### GitHub Actions Webhook

- **Frequency:** Every 3 hours
- **Payload:** Trigger execution with credentials
- **Response:** Exit code indicates success/failure
- **Logging:** Output to GitHub Actions log

### Database Connection

- **Protocol:** MySQL native protocol
- **Port:** 3306
- **Encryption:** Optional SSL/TLS
- **Connection Pooling:** Enabled

### Error Notifications

- **Method:** GitHub Actions email
- **Trigger:** On execution failure
- **Content:** Error logs, timestamp, execution details

---

## Performance Specifications

### Throughput

- **Records per Execution:** 100-500 (typical)
- **Extraction Speed:** 5-10 records per minute
- **Database Operations:** <100ms per transaction

### Resource Usage

- **CPU:** 30-60% during extraction
- **RAM:** 512 MB - 1.5 GB
- **Disk I/O:** 10-50 MB per execution
- **Network:** 10-50 Mbps during extraction

### Timeouts

- **Element Wait:** 30 seconds
- **Page Load:** 60 seconds
- **Connection:** 10 seconds
- **Query Execution:** 5 seconds

---

## Security Specifications

### Credential Management

- **Credentials:** Environment variables only
- **No Hardcoding:** Zero credentials in code
- **No Logging:** Credentials never logged
- **GitHub Secrets:** All sensitive data stored here

### Database Access

- **User Privileges:** SELECT, INSERT, UPDATE, DELETE only
- **No DDL:** No CREATE, ALTER, DROP permissions
- **Connection Limit:** Per-user connection quota
- **Audit:** Log important operations

### Data Protection

- **Encryption in Transit:** Optional SSL/TLS
- **Encryption at Rest:** Database-level (optional)
- **Access Control:** Username/password authentication
- **Backup:** Regular automated backups

---

## Monitoring & Logging

### Log Locations

- **Application Logs:** `/data/logs/`
- **Error Logs:** `/data/logs/errors/`
- **Screenshots:** `/data/screenshots/`
- **GitHub Actions:** GitHub Actions UI

### Log Format

```
[TIMESTAMP] [LEVEL] [COMPONENT] Message
2026-06-15 14:30:45 INFO [Pagination] Detected button-based navigation
2026-06-15 14:30:46 ERROR [Locator] CSS selector failed, trying XPath
```

### Log Retention

- **Application Logs:** 30 days
- **Error Logs:** 60 days
- **Screenshots:** 7 days
- **Database:** Full historical retention

---

## Disaster Recovery

### Backup Strategy

- **Frequency:** Daily automated backups
- **Location:** Separate storage system
- **Retention:** 30 days of full backups
- **Verification:** Weekly backup integrity checks

### Recovery Procedures

- **Database Corruption:** Restore from backup
- **Connection Loss:** Automatic reconnection with exponential backoff
- **Failed Extraction:** Partial records rolled back, next cycle resumes
- **GitHub Actions Failure:** Automatic retry in next scheduled window

---

## Scalability Considerations

### Current Limits

- **Concurrent Executions:** 1 (sequential via GitHub Actions)
- **Database Connections:** 3 (pooled)
- **Records per Cycle:** 500 (configurable)
- **Storage Growth:** ~100 MB per month

### Future Scaling

- **Horizontal:** Multiple scheduled windows (different regions)
- **Vertical:** Increase pool size, batch sizes
- **Database:** Archive old records to separate system
- **Caching:** In-memory deduplication already implemented

---

## Compliance & Standards

### Code Standards

- **Language:** Python 3.11+ (PEP 8)
- **Database:** MySQL 8.0+ (InnoDB)
- **Encoding:** UTF8MB4 throughout
- **Transactions:** ACID compliance enforced

### Data Standards

- **Character Encoding:** UTF8MB4 (supports all languages)
- **Date Format:** ISO 8601 (YYYY-MM-DD)
- **Timezone:** UTC stored, local display on demand
- **Numeric Precision:** Decimal(15,2) for monetary values

### Security Standards

- **Credentials:** Environment-based, no hardcoding
- **Connections:** Connection pooling, automatic cleanup
- **Transactions:** All-or-nothing atomicity
- **Backups:** Daily automated, verified regularly

---

## Testing Requirements

### Unit Testing

- **Framework:** pytest
- **Coverage:** >80% code coverage
- **Database Mocks:** Yes, for integration tests
- **CI Integration:** GitHub Actions runs tests

### Integration Testing

- **Database:** Test database for integration tests
- **Firefox:** Headless mode testing
- **Element Locators:** Test with sample pages
- **Error Handling:** Test failure scenarios

### Production Testing

- **Pre-deployment:** Test run on staging database
- **Validation:** Verify data integrity
- **Performance:** Monitor first execution
- **Rollback:** Ability to revert if needed

---

## Maintenance Schedule

### Daily Tasks

- Monitor GitHub Actions execution logs
- Verify database connectivity
- Check available disk space

### Weekly Tasks

- Review error logs for patterns
- Verify backup completion
- Test disaster recovery procedures

### Monthly Tasks

- Archive old log files
- Archive old screenshots
- Analyze performance trends
- Update dependencies if needed

### Quarterly Tasks

- Full database backup verification
- Security audit of credentials
- Performance optimization review
- Capacity planning assessment

---

## Support & Troubleshooting

### Common Issues

1. **Element Not Found**
   - Check if multi-strategy locator worked
   - Review page screenshots
   - Verify selector is in fallback chain

2. **Pagination Failed**
   - Verify both button and page-jump methods
   - Check for JavaScript errors
   - Review extraction logs

3. **Database Connection Lost**
   - Check connection pool status
   - Verify database is online
   - Review network connectivity

4. **GitHub Actions Timeout**
   - Review execution logs
   - Check for slow page loads
   - Increase timeout if needed

---

## Version & Change Log

**Current Version:** 1.0.0  
**Last Updated:** June 2026  
**Status:** Production Ready

### Notable Features

- Multi-strategy element locators
- Adaptive pagination detection
- GitHub Actions automation
- Atomic transaction support
- Archive database system

---

**For questions or clarifications, refer to ARCHITECTURE.md and DESIGN_DECISIONS.md**
