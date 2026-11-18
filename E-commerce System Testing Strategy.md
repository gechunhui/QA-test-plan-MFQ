# E-commerce System Testing Strategy (MFQ Framework)

## 1. Priorities

Testing priorities must align with business impact, financial risk, and revenue impact:

| Priority | Definition | Key Coverage Areas | Release Requirements |
|---|---|---|---|
| **P0 - Blocker** | Failures directly result in financial losses, overselling, or interruption of core transactions. | End-to-end golden path: Order Placement → Payment → Shipment → Delivery; inventory reservation, deduction, and release; payment callback idempotency; overselling prevention; inventory protection against duplicate and out-of-order messages. | Must pass. Release is prohibited if any P0 test fails. |
| **P1 - High** | May cause business interruptions, order backlogs, or subsequent data inconsistencies. | MQ message loss/backlog, payment timeouts, WMS/SC2P synchronization failures, retries, dead-letter queues, compensating mechanisms, and inventory reconciliation. | Must be executed, and a risk assessment must be completed. |
| **P2 - Medium** | Affects specific business scenarios or non-critical quality attributes. | Partial shipments, zero-inventory display, general boundary scenarios, sensitive data masking, and degraded performance of non-critical queries. | Execute based on project risks and release scope. |
| **P3 - Low** | User experience issues that do not affect transaction correctness. | UI interaction details, minor visual inconsistencies, and low-risk copywriting issues. | May be included in subsequent iterations but must be documented. |

## 3. Trade-offs

### 3.1 Test Coverage Completeness vs. Time to Market

It is unnecessary to exhaustively test every combination of Product × SKU × Warehouse × Order Status. Instead, adopt a risk-based testing approach and prioritize the following scenarios:

- Positive end-to-end golden path;
- Inventory boundary conditions, including zero inventory, exactly sufficient inventory, and inventory short by one unit;
- Concurrent overselling risks;
- Duplicate requests, duplicate messages, and out-of-order events;
- Payment timeouts, delayed callbacks, and external system failures;
- Inventory release upon order cancellation and inventory restoration after returns.

Improve test coverage efficiency through equivalence partitioning, boundary value analysis, state transition testing, and critical combination testing.

### 3.2 Performance Coverage vs. Testing Cost

Do not perform full-scale end-to-end stress testing indiscriminately across all modules. Prioritize isolated load testing of flash sale entry APIs and the inventory reservation/deduction layer, followed by necessary end-to-end capacity validation.

- **Specified Load Testing Scenario:** Simulate sudden traffic spikes of 10,000 QPS.
- **Key Metrics:** Throughput, P95/P99 latency, error rate, inventory accuracy, lock contention, CPU/memory utilization, and database connections.
- **Pass Criteria:** No overselling, negative inventory, or duplicate inventory deductions. System rate limiting, request queuing, and service degradation must behave as designed.

### 3.3 Consistency vs. Availability

Cross-system states between the e-commerce platform and WMS/SC2P are allowed to achieve eventual consistency within a short period. Distributed transactions (2PC) are not mandatory. Instead, prioritize reliable messaging, idempotent message consumption, limited retries, scheduled compensating operations, and reconciliation mechanisms.

**Degradation Boundaries Must Be Clearly Defined:**

- When WMS/SC2P is unavailable, product displays and non-critical queries may use caching or service degradation according to business policies.
- Local caches may only be used for non-authoritative displays or queries. They must not serve as the sole basis for confirming successful inventory reservations.
- If the inventory service cannot confirm the reservation result, the system must return an explicit failure or processing status. Orders must not be allowed to proceed based on stale cached data.
- After external systems recover, data consistency must be restored through message replay and inventory reconciliation.
"""
