# Integrated Order Management System

**2021 – 2025 · PHP (Phalcon), MySQL, JavaScript, Ajax**

A business system for managing order fulfillment, local inventory, 
and returns/claims, integrated with an external order-management API. 
Built end-to-end: architecture, implementation, deployment, and 
ongoing feature development.

> **Source code:** Not publicly available. This repository documents 
> the architecture and selected implementation details, based on 
> internal project documentation.

## Overview

The system synchronizes orders and inventory data from a third-party 
order-management platform, maintains a local inventory ("local 
warehouse"), and automates order status transitions based on 
inventory availability and incoming deliveries.

```text
                  ┌─────────────────────┐
                  │  External Order API  │
                  └──────────┬───────────┘
                             │ sync (cron)
                             ▼
                  ┌─────────────────────┐
                  │   Local database    │
                  │  (orders, inventory,│
                  │   invoices, logs)   │
                  └──────────┬───────────┘
                             │
              ┌──────────────┼──────────────┐
              ▼              ▼              ▼
        Order status    Inventory      Analytics &
        automation      management     reporting
```

## API Integration & Rate Limiting

The system polls an external order-management API on a scheduled 
basis, syncing orders, products, shipments, and invoices into a 
local database.

To stay within the API's request-rate limits, the system tracks its 
own request history and throttles further calls once approaching 
the limit — rather than relying on trial and error or hard failures:

```php
// simplified illustration — not the original implementation
if (getRecentRequestCount() >= SAFE_REQUEST_LIMIT) {
    waitUntilWindowClears();
}
recordRequest();
callExternalApi();
```

## Distributing Long-Running Tasks Across a Cron Window

Some synchronization tasks are too long to run within a single 
scheduled interval. Rather than risk overlapping executions, the 
system distributes these tasks across a configurable time window, 
sized based on observed execution times:

```php
// simplified illustration — not the original implementation
if (timeSinceLastRun() >= DISTRIBUTION_INTERVAL) {
    runLongTask();
}
```

## Multi-Format Invoice Parsing

The system ingests supplier invoices in multiple formats (PDF and 
two XML variants), each requiring a dedicated extraction approach — 
including regex-based text extraction for PDF invoices — normalizing 
all of them into a common internal structure (recipient, invoice 
number, dates, totals, and itemized lines).

## Fuzzy Product Matching

Products arriving from different data sources don't always share a 
consistent identifier. A matching routine reconciles these using a 
tiered strategy: exact structural matching first, falling back to a 
frequency-informed search when the fast path fails — with results 
cached for reuse rather than recomputed on every lookup.

## Performance Tuning

Processing large inventory datasets required deliberate performance 
work beyond default framework behavior:

* using plain array structures instead of full ORM objects for 
  hot-path data access
* limiting queried columns to only what's needed
* measuring actual impact of server resources on processing time 
  (observed ~20x speedup on a higher-spec configuration) to inform 
  hosting decisions

## Internal API

An internal API layer (~15 methods) mediates between the frontend, 
scheduled jobs, and the external API — handling data synchronization, 
status transitions, analytics calculation, and reporting exports 
through a single, consistent interface.

## Logging & Diagnostics

The system maintains separate logs for general activity, API 
interactions, and inventory operations, with automatic monthly log 
rotation and periodic cleanup of time-bound tracking data.

## Historical Context

Originally specified in **2020** and in active use through **2025**, 
with ongoing feature development across that period.

## What the Project Demonstrates

* Integration with a rate-limited external API under production load
* Custom scheduling strategy for long-running tasks within cron constraints
* Multi-format document parsing and data normalization
* Heuristic data-matching algorithm with tiered fallback strategy
* Performance profiling and optimization based on measurement, not guesswork
* Internal API design mediating multiple system consumers

**Status:** Historical/ongoing production project · source code not 
publicly available 
