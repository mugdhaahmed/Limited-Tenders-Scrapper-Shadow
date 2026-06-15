# Deployment Guide

Deployment of the system on Hostinger VPS with Docker and GitHub Actions CI/CD integration.

---

## Overview

The system is deployed as a containerized application on Hostinger VPS with GitHub Actions providing scheduled execution:

- **Container Runtime:** Docker
- **Host:** Hostinger Virtual Private Server (VPS)
- **Database:** MySQL on Hostinger
- **Execution:** GitHub Actions (every 3 hours)
- **Execution Mode:** Automated, no manual intervention

---

## Architecture

```
GitHub Actions
├── Scheduled Workflow (every 3 hours)
└── Triggers Docker Container
        ↓
Hostinger VPS
├── Docker Engine
│   └── App Container
│       ├── Python 3.11+
│       ├── Selenium WebDriver (Firefox)
│       ├── Data volumes (/data)
│       └── .env configuration
│
└── Hostinger MySQL
    ├── Live Database
    └── Archive Database

```

---

## Prerequisites

### VPS Requirements

- [ ] Hostinger VPS account
- [ ] SSH access to VPS
- [ ] Docker installed on VPS
- [ ] Sufficient disk space (15GB+)


### Database Setup

- [ ] MySQL database created on Hostinger
- [ ] Database user with appropriate privileges
- [ ] Database connection tested from VPS
- [ ] Network firewall rules configured


### GitHub Setup

- [ ] Repository created on GitHub
- [ ] GitHub Actions enabled
- [ ] Secrets configured (DB credentials, etc.)
- [ ] Workflow file (.github/workflows/runs-scraper.yml)

---

## Production Checklist

Before going live on Hostinger:

- [ ] Hostinger VPS created and SSH access configured
- [ ] Docker installed on VPS
- [ ] MySQL database created on Hostinger
- [ ] Database credentials verified
- [ ] .env file configured with Hostinger credentials
- [ ] Docker image built and tested locally
- [ ] Container runs successfully once (test run)
- [ ] GitHub Actions workflow configured
- [ ] Secrets added to GitHub (DB_HOST, DB_USER, DB_PASSWORD)
- [ ] Workflow runs successfully (test execution)
- [ ] Logs verified for normal operation
- [ ] Database integrity verified
- [ ] Error notifications configured
- [ ] Auto-restart policy configured

---

## System Configuration Summary

| Component        | Location            | Status                     |
| ---------------- | ------------------- | -------------------------- |
| **VPS**          | Hostinger           | ✅ Deployed                 |
| **Docker**       | Hostinger VPS       | ✅ Running                  |
| **Container**    | Hostinger VPS       | ✅ app-scraper             |
| **Database**     | Hostinger MySQL     | ✅ app\_db + app\_archive   |
| **Data**         | /home/app/scraper   | ✅ Logs, screenshots        |
| **Config**       | /home/app/.env      | ✅ Hostinger credentials    |
| **Execution**    | GitHub Actions      | ✅ Every 3 hours (automated) |
| **Monitoring**   | GitHub Actions UI   | ✅ Build status & logs      |

---

## Conclusion

The system is deployed on Hostinger VPS with GitHub Actions automation:

1. **Containerized** - Docker image simplifies deployment
2. **Database on Hostinger** - MySQL with live and archive databases
3. **GitHub-Driven** - Actions scheduler provides reliable execution
4. **Automated** - No manual intervention needed
5. **Monitored** - GitHub Actions logs available for troubleshooting
6. **Scalable** - Can adjust container resources as needed


This setup provides reliable, fully-automated, production-grade operation on Hostinger infrastructure.
