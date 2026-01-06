# DynamoDB

## Overview

Amazon DynamoDB is a fully managed, serverless, key-value NoSQL database designed to run high-performance applications at any scale. It offers built-in security, continuous backups, automated multi-region replication, in-memory caching, and data export tools.

## Study Checklist

### Core Concepts
- [ ] Partition Key vs Sort Key
- [ ] Read/Write Capacity Units (RCU/WCU)
- [ ] Consistency Models (Eventual vs Strong)
- [ ] Item Size Limits (400KB)

### Indexing & Querying
- [ ] Global Secondary Indexes (GSI)
- [ ] Local Secondary Indexes (LSI)
- [ ] Scan vs Query
- [ ] Filter Expressions vs Key Conditions

### Advanced Design
- [ ] Single Table Design (Adjacency Lists)
- [ ] Sparse Indexes
- [ ] DynamoDB Streams & Triggers
- [ ] Time To Live (TTL)

### Performance & Scaling
- [ ] On-Demand vs Provisioned Mode
- [ ] DAX (DynamoDB Accelerator)
- [ ] Handling Hot Partitions
- [ ] Batch Operations

## Quick Reference

| Topic | Key Concept |
|-------|-------------|
| Partition Key | Distributes data across physical storage nodes |
| Sort Key | Sorts data within a partition |
| GSI | Index with new Partition Key & Sort Key (Eventual Consistency) |
| LSI | Index with same Partition Key but new Sort Key |
| Strong Consistency | Read latest write (consumes 2x RCU) |
| Eventual Consistency | Fast, cheap, might be <1s stale (Default) |
| Single Table Design | Storing multiple entities in one table for performance |
| TTL | Automatically delete items after a timestamp |
| DAX | In-memory cache for microsecond latency |

## Resources

- [Questions & Answers](./questions-and-answers.md)

## Progress Tracker

- [ ] Master Partition/Sort Key design
- [ ] Implement a Single Table Design schema
- [ ] Understand GSI vs LSI trade-offs
- [ ] Use DynamoDB Streams with Lambda
- [ ] Handle throttling with exponential backoff
