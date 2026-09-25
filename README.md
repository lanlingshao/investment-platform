# investment-platform

A personal investment research platform for systematic financial data collection, factor computation, valuation analysis, stock screening, alerts, and investment research workflows.

The platform is designed for fundamental and value-oriented equity research. It connects financial data, factor computation, valuation metrics, screening rules, alerts, and research workflows into a unified system.

> **Note**: investment-platform is a private project. The core application source code is not open-sourced. Several reusable infrastructure components extracted from the platform are available as independent open-source projects.

---

## Overview

The platform covers the complete research data flow:

```
Financial Data 
        │
        ▼ 
Data Collection 
        │
        ▼ 
Data Storage 
        │
        ▼ 
Factor Engine 
        │
        ▼ 
        ├── Valuation Factors 
        ├── Profitability Factors 
        ├── Financial Factors 
        └── Derived Factors 
        │
        ▼ 
 Event & Alert System 
        │
        ▼ 
 Investment Research Console 
        ├── Stock Screening 
        ├── Stock Scoring 
        ├── Valuation Analysis 
        ├── Stock Comparison 
        ├── Alerts 
        └── Research Tags
```

The system is intended to make investment research more systematic and reproducible by separating:
- financial data acquisition
- data storage
- factor computation
- investment rules
- screening and scoring
- alerts
- research workflows

## System Architecture

The platform is organized into several major subsystems:

![](diagrams/system-architecture.png)

The architecture separates infrastructure concerns from investment-specific business logic.

## Core Components
### 1. Financial Data Collection

The data collection subsystem is built on top of [fin-data-kit](https://github.com/lanlingshao/fin-data-kit).

It collects and normalizes financial market data from multiple sources, including:
- Daily market data
- Financial statements
- Share capital data
- Stock lists and security metadata
- Trading calendars
- Other fundamental and market data

The application declares the required data capabilities rather than directly depending on a specific data provider.

For example:
```
Application 
    │ 
    │ request: daily_kline 
    ▼ fin-data-kit 
    │ 
    ├── Primary Source 
    │ 
    ├── Fallback Source 
    │ 
    ├── Rate Limiting 
    │ 
    └── Retry 
    │ 
    ▼ Financial Data
```

This allows data sources to be changed or extended without coupling the research application to a specific provider.

**Key characteristics**
- Multiple data sources
- Capability-oriented data access
- Configurable source priority
- Rate limiting
- Automatic retry
- Source fallback
- Unified data interfaces
- Asynchronous data collection

### 2. Factor Computation Engine
The factor engine is the core quantitative research component of the platform.

It computes fundamental and valuation indicators such as:
- PE
- PB
- ROE
- Dividend Yield
- Earnings-related indicators
- Valuation percentiles
- Other derived investment factors

The engine uses a DAG-based dependency model to represent relationships between factors.

For example:

```
Financial Statements 
        │ 
        ├───────────────┐ 
        ▼               ▼ 
     Net Income       Equity 
        │               │ 
        └───────┬───────┘ 
                ▼ ROE 
                │ 
                ▼ 
        Historical ROE Analysis 
        
Market Data ──────────────┐ 
                          ▼ 
                  Valuation Factors 
                          │ 
              ┌───────────┴───────────┐ 
              ▼                       ▼ 
              PE                     PB 
              │                       │ 
              └───────────┬───────────┘ 
                          ▼ 
                 Valuation Percentile
```

The DAG allows dependent factors to be calculated in the correct order while avoiding unnecessary duplicate computation.

The factor engine is intentionally kept inside the private project because it contains investment-specific research logic.

### 3. Event & Alert System

The alert subsystem connects the data pipeline and factor engine with investment rules.

The event flow is approximately:
```
Data Collection 
        │ 
        │ Kafka Event 
        ▼ 
┌───────────────┐ 
│ Event System  │ 
└───────┬───────┘ 
        │ 
        ▼ 
┌───────────────┐ 
│ Rule Engine   │ 
└───────┬───────┘ 
        │ 
        ├── Stock Rules 
        │ 
        ├── Tag Rules 
        │ 
        └── Indicator Conditions 
        │ 
        ▼ Alert
```

The event processing layer is built on top of the open-source [eventflow](https://github.com/lanlingshao/eventflow) framework.

The application-specific rule engine evaluates investment conditions based on incoming events.

Examples include:
- valuation threshold alerts
- factor threshold alerts
- indicator trend conditions
- stock-specific rules
- tag-based rules

The separation between event transport and investment rules allows the event infrastructure to remain reusable while investment logic stays within the application.

### 4. Investment Research Console

The research console provides the main interface for interacting with the investment research system.

**Research Tags**

Stocks can be organized using research-oriented tags.

Examples include:

- Industry leaders
- Companies with strong competitive advantages
- Pricing power
- High-ROE Companies
- Watchlist
- Holdings
- Intended purchase list

Tags can also be used as inputs to screening and alert rules.

---

**Stock Scoring**

The platform supports stock scoring and research indicators.

The scoring system allows qualitative investment research to be combined with quantitative indicators.

For example:
```
Company 
    │ 
    ├── Financial Indicators 
    ├── Valuation Indicators 
    ├── Research Tags 
    └── Manual Score 
              │ 
              ▼ 
       Investment View
```

---

**Indicators**

The indicator system supports both system-defined and user-defined research indicators.

System indicators are generated from the factor engine, while user-defined indicators can be constructed from existing data and mathematical expressions.

This allows the research console to expose investment concepts without tightly coupling the UI to the underlying factor implementation.

--- 

**Stock Screener**

The stock screener supports filtering stocks using combinations of:

- Industry
- Research Tags
- Scores
- Valuation indicators
- Financial indicators
- User-defined indicators

Example:
```
Industry 
    AND 
ROE > threshold 
    AND 
PB < threshold 
    AND 
PE percentile < threshold 
    AND 
Dividend Yield > threshold
```

The purpose is to transform investment ideas into explicit, repeatable screening conditions.

---

**Stock Comparison**

The platform provides a stock comparison interface for comparing companies across multiple indicators.

Typical comparison dimensions include:

- Valuation
- Profitability
- Growth
- Dividends
- Financial quality
- Historical indicators

This is intended to support comparative research rather than replacing fundamental investment analysis.

---

**Alerts**

The alert interface allows investment rules to be configured and monitored.

Rules can reference:

- stocks
- tags
- indicators
- indicator trends
- valuation conditions

The system evaluates these rules against events generated by the data and factor pipelines.

---

## Task Scheduling

Data collection and factor computation tasks are orchestrated using [Prefect](https://www.prefect.io/).

Typical workflows include:
```
Trading Calendar 
        │ 
        ▼ 
Data Collection 
        │ 
        ▼ 
Financial Data Update 
        │ 
        ▼ 
Factor Computation 
        │ 
        ▼ 
Factor Snapshot 
        │ 
        ▼ 
Event / Alert Processing
```

The scheduling layer is responsible for:
- periodic data collection
- factor computation
- task dependencies
- retries
- workflow execution
- historical backfills
- scheduled and manual execution

This separates workflow orchestration from the business logic implemented by the individual services.

---

## Open-source Infrastructure

Several reusable infrastructure components were extracted from the platform and developed as independent open-source projects.

### daokit

**Asynchronous data-access toolkit for MySQL and ClickHouse**

*daokit* is an asynchronous data-access toolkit built on SQLAlchemy, asyncmy, asynch, and clickhouse-connect.

It provides reusable DAO base classes and database clients while leaving application-specific query rules in small, explicit DAO subclasses.

Repository:

[https://github.com/lanlingshao/daokit](https://github.com/lanlingshao/daokit)

---

### fin-data-kit

**Asynchronous financial data collection framework.**

*fin-data-kit* is an asynchronous framework for collecting financial data.

Callers declare the capabilities they require, while the framework handles:

- data source selection
- source priority
- rate limiting
- retries
- source fallback
- asynchronous execution

Repository:

[https://github.com/lanlingshao/fin-data-kit](https://github.com/lanlingshao/fin-data-kit)

---

### eventflow

**Asynchronous event-dispatching framework for Python and Kafka.**

*eventflow* separates:

- event production
- broker consumption
- batch processing
- failure handling

Applications can therefore focus on business event handlers without embedding Kafka-specific infrastructure throughout the application.

Repository:

[https://github.com/lanlingshao/eventflow](https://github.com/lanlingshao/eventflow)

---

## Data & Event Flow

A simplified end-to-end workflow:

```
            ┌──────────────────┐ 
            │ Financial Sources│ 
            └────────┬─────────┘ 
                     │ 
                     ▼ 
            ┌──────────────────┐ 
            │ Data Collector   │ 
            │ fin-data-kit     │ 
            └────────┬─────────┘ 
                     │ 
                     ▼ 
            ┌──────────────────┐ 
            │ MySQL/ClickHouse │ 
            └────────┬─────────┘ 
                     │ 
                     ▼ 
            ┌──────────────────┐ 
            │ Factor Engine    │ 
            │                  │ 
            │ Factor DAG       │ 
            └────────┬─────────┘ 
                     │ 
                     │ Kafka 
                     ▼ 
            ┌──────────────────┐ 
            │ eventflow        │ 
            │ Event Processing │ 
            └────────┬─────────┘ 
                     │
                     ▼ 
            ┌──────────────────┐ 
            │ Rule Engine      │ 
            └────────┬─────────┘ 
                     │ 
        ┌────────────┴────────────┐ 
        ▼                         ▼ 
Stock Screening                 Alerts 
        │                         │ 
        └────────────┬────────────┘ 
                     ▼ 
             Investment Research 
                  Console
```

---

## Research Workflow

The platform is designed around a repeatable research workflow:

```
Market / Financial Data 
          ↓ 
    Data Cleaning 
          ↓ 
  Factor Computation 
          ↓ 
Historical Analysis 
          ↓ 
   Stock Screening 
          ↓ 
 Company Comparison 
          ↓ 
    Research Tags / Scores 
          ↓ 
      Monitoring 
          ↓ 
        Alerts

```

The objective is not to automate investment decisions, but to provide infrastructure for systematic research and monitoring.

---

## Screenshots


