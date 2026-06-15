# Automated Limited Tenders Scraper - Architecture Showcase

> A production-grade web scraper demonstrating resilient automation patterns. Built to reliably extract limited tender data with adaptive element locators, intelligent pagination handling, and CI/CD-integrated continuous execution.

[![Python 3.11+](https://img.shields.io/badge/python-3.11+-blue.svg)](https://www.python.org/downloads/) [![MySQL 8.0+](https://img.shields.io/badge/mysql-8.0+-blue.svg)](https://www.mysql.com/) [![Docker](https://img.shields.io/badge/docker-%230db7ed.svg?logo=docker&logoColor=white)](https://www.docker.com/)

---

## 🎯 Overview

[#-overview](#-overview)

This project demonstrates **production-grade systems design** for automated data collection with dynamic element handling:

- **Problem Domain:** Automated extraction of limited tender data from dynamic web portals
- **Key Challenge:** Handle fragile page selectors and variable pagination patterns
- **Solution:** Multi-strategy element locators, adaptive pagination detection, GitHub Actions automation
- **Scale:** Processes high-volume tender records with comprehensive data validation

---

## ✨ Core Architectural Principles

[#-core-architectural-principles](#-core-architectural-principles)

### 1. **Resilient Element Location**

[#1-resilient-element-location](#1-resilient-element-location)

Multi-strategy selector fallback chain:

- **CSS Selector** - First attempt (fastest)
- **XPath** - Semantic element paths
- **Index-based** - Positional fallback
- **JavaScript** - DOM queries
- **jQuery** - Chained selections

**Result:** Handles selector breakage and dynamic content gracefully

### 2. **Adaptive Pagination Strategy**

[#2-adaptive-pagination-strategy](#2-adaptive-pagination-strategy)

Flexible pagination handling:

- Button-based navigation (click next)
- Direct page jump (numeric input + submit)
- Automatic strategy detection
- Fallback between approaches
- Handles both old and new UI patterns

**Key principle:** Never assume a single pagination method

### 3. **Relational Data Integrity**

[#3-relational-data-integrity](#3-relational-data-integrity)

Normalized multi-table schema:

- Separate entity tables for different concerns
- Foreign key constraints ensuring consistency
- Atomic transactions (all-or-nothing updates)
- Built-in validation across relationships

### 4. **CI/CD Integrated Automation**

[#4-cicd-integrated-automation](#4-cicd-integrated-automation)

GitHub Actions-driven continuous execution:

- Scheduled runs (every 3 hours)
- Automatic error notifications
- Build status tracking
- No manual intervention required
- Integrated version control

---

## 🏗️ Architecture

[#️-architecture](#️-architecture)

```
GitHub Actions Scheduler
      ↓
Adaptive Pagination Layer
      ↓
Multi-Strategy Locator | Data Layer | Validation
      ↓
Data Processing (Parse, Normalize, Deduplicate)
      ↓
Database (Connection Pool + Atomic Transactions)

```

---

## 🔑 Core Design Patterns

[#-core-design-patterns](#-core-design-patterns)

### **Pattern 1: Selector Fallback Chain**

[#pattern-1-selector-fallback-chain](#pattern-1-selector-fallback-chain)

Try multiple strategies sequentially until one succeeds, handling page structure variations.

### **Pattern 2: Pagination Detection**

[#pattern-2-pagination-detection](#pattern-2-pagination-detection)

Detect available pagination methods and choose appropriate strategy, adapting to UI changes.

### **Pattern 3: Atomic Transactions**

[#pattern-3-atomic-transactions](#pattern-3-atomic-transactions)

All related data updates grouped into single transaction for guaranteed consistency.

### **Pattern 4: Connection Pooling**

[#pattern-4-connection-pooling](#pattern-4-connection-pooling)

Maintain reusable database connections to respect external quotas and improve efficiency.

### **Pattern 5: Scheduled Automation**

[#pattern-5-scheduled-automation](#pattern-5-scheduled-automation)

GitHub Actions orchestration provides reliable, version-controlled execution scheduling.

---

## 📊 Technical Architecture

[#-technical-architecture](#-technical-architecture)

### **Browser Automation**

[#browser-automation](#browser-automation)

- **Browser:** Firefox WebDriver (headless)
- **Locator Strategy:** CSS → XPath → Index → JavaScript → jQuery
- **Waits:** Explicit element waits, smart retries
- **Screenshots:** Captured on failures for debugging

### **Technology Stack**

[#technology-stack](#technology-stack)

- **Language:** Python 3.11+
- **Automation:** Selenium WebDriver (Firefox)
- **Database:** MySQL 8.0+ (InnoDB)
- **Connection Management:** Connection pooling
- **CI/CD:** GitHub Actions
- **Deployment:** Docker

### **Performance Characteristics**

[#performance-characteristics](#performance-characteristics)

- **Throughput:** High-volume records per execution
- **Reliability:** 99%+ uptime through scheduled automation
- **Error Recovery:** Automatic retries with smart backoff
- **Data Safety:** Zero orphaned records through atomic transactions
- **Flexibility:** Adapts to page structure changes

---

## ✨ Key Features

[#-key-features](#-key-features)

### **Smart Element Location**

[#smart-element-location](#smart-element-location)

✅ Multi-strategy selector fallback  
✅ Automatic retry on stale elements  
✅ JavaScript-based element queries  
✅ Semantic XPath expressions

### **Adaptive Pagination**

[#adaptive-pagination](#adaptive-pagination)

✅ Button-based navigation support  
✅ Direct page-jump capability  
✅ Automatic method detection  
✅ Handles UI variations

### **Data Management**

[#data-management](#data-management)

✅ Normalized multi-table schema  
✅ Atomic transactions  
✅ Integrity validation  
✅ Automatic archival

### **Continuous Execution**

[#continuous-execution](#continuous-execution)

✅ GitHub Actions automation  
✅ Scheduled every 3 hours  
✅ Error notifications  
✅ No manual intervention needed

---

## 🎓 Engineering Principles

[#-engineering-principles](#-engineering-principles)

### **Resilience Over Fragility**

[#resilience-over-fragility](#resilience-over-fragility)

Fragile selectors and layout changes aren't failures—they're design challenges that drive robust solutions.

### **Data Integrity Priority**

[#data-integrity-priority](#data-integrity-priority)

Consistency across related data matters most. Atomic transactions are non-negotiable.

### **Observable Systems**

[#observable-systems](#observable-systems)

Structured logging, error tracking, CI/CD integration for complete production visibility.

### **Graceful Degradation**

[#graceful-degradation](#graceful-degradation)

Adapt to page changes automatically. Fail clearly and recover without data corruption.

---

## 📚 Documentation

[#-documentation](#-documentation)

- **[ARCHITECTURE.md](https://github.com/mugdhaahmed/Limited-Tenders-Scrapper-Shadow/blob/main/architecture/ARCHITECTURE.md)** - System design with 5 components and 5 patterns
- **[DEPLOYMENT.md](https://github.com/mugdhaahmed/Limited-Tenders-Scrapper-Shadow/blob/main/docs/DEPLOYMENT.md)** - Production deployment procedures
- **[TECHNICAL_SPEC.md](https://github.com/mugdhaahmed/Limited-Tenders-Scrapper-Shadow/blob/main/docs/TECHNICAL_SPEC.md)** - System requirements and configuration
- **[DESIGN_DECISIONS.md](https://github.com/mugdhaahmed/Limited-Tenders-Scrapper-Shadow/blob/main/case-studies/DESIGN_DECISIONS.md)** - 10 key architectural decisions
- **[CODE_SAMPLES.md](https://github.com/mugdhaahmed/Limited-Tenders-Scrapper-Shadow/blob/main/examples/CODE_SAMPLES.md)** - Design patterns and pseudocode

---

## 📄 License

[#-license](#-license)

Proprietary. See LICENSE.md

---

**Engineered for resilience, adaptability, and production-grade data integrity.**
