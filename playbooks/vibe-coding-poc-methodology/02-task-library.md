# Vibe Coding POC: Production Task Library

## Overview

This document provides battle-tested task scenarios derived from common production challenges. Each task represents a category of problems that occurs repeatedly in real-world systems.

## Task Categories

Common problem categories in production systems:

| Category | Typical Characteristics | Evaluation Focus |
|----------|------------------------|------------------|
| Performance Degradation | Gradual or sudden slowdown | Pattern recognition, optimization |
| Service Communication | Integration failures | Distributed system understanding |
| State Management | Concurrency issues | Complex logic debugging |
| Legacy Refactoring | Technical debt | Code comprehension |
| Security Vulnerabilities | Exploitable weaknesses | Edge case identification |
| Data Corruption | Integrity issues | Root cause analysis |

## Detailed Task Specifications

### TASK-PERF-001: Hidden O(n³) Performance Killer

**Common Pattern**: Nested loops in business logic

**Scenario**:
```python
# Simplified version of common production pattern
def calculate_final_price(cart_items, discounts, user_history):
    total = 0
    for item in cart_items:  # n items
        for discount in discounts:  # m discounts
            for past_purchase in user_history:  # p past purchases
                if discount.applies_to(item, past_purchase):
                    # Complex calculation involving database call
                    modifier = fetch_discount_modifier(item, discount, past_purchase)
                    total += apply_modifier(item.price, modifier)
    return total
```

**Symptoms**:
- 10 items, 5 discounts, 100 history items = 5000 iterations
- Each iteration: 50ms database call
- Total: 250 seconds for checkout

**Provided**:
- Working but slow implementation
- Test suite with 5 test cases
- Performance profile showing hot spots
- Database with realistic data

**Acceptance Criteria**:
1. Reduce complexity to O(n) or O(n log n)
2. Maintain identical output for all test cases
3. Add caching where appropriate
4. Document algorithm change
5. p99 latency < 200ms

**Hidden Challenges**:
- Discount rules have complex interdependencies
- Some calculations must remain sequential
- Cache invalidation complexity

---

### TASK-DIST-002: The Distributed Transaction Trap

**Common Pattern**: Synchronous distributed transactions

**Scenario**:
Three services must coordinate to process a payment:
1. Payment Service: Charges credit card
2. Inventory Service: Reserves items
3. Notification Service: Sends confirmation

Current implementation uses synchronous calls:
```python
def process_order(order):
    payment_result = payment_service.charge(order.payment_info)
    if payment_result.success:
        inventory_result = inventory_service.reserve(order.items)
        if inventory_result.success:
            notification_service.send_confirmation(order.user)
        else:
            payment_service.refund(payment_result.transaction_id)
            raise InventoryError()
```

**Problem**: 
- Service B times out after Service A succeeds
- Partial state: payment charged but no inventory reserved
- Rollback fails occasionally

**Provided**:
- Docker environment with 3 services
- Chaos engineering tool to simulate failures
- Current implementation (broken)
- Message queue infrastructure available

**Acceptance Criteria**:
1. Implement saga pattern or equivalent
2. Handle all failure modes:
   - Service timeout
   - Partial failure
   - Network partition
3. Ensure eventual consistency
4. Add distributed tracing
5. Zero data loss under any failure

**Hidden Challenges**:
- Idempotency not guaranteed in current services
- Message ordering matters for some operations
- Compensating transactions have their own failure modes

---

### TASK-LEGACY-003: The Monolith Authentication Module

**Common Pattern**: Organic growth over years

**Current State**:
```python
class AuthenticationMonolith:
    """
    800+ lines handling:
    - JWT generation/validation
    - OAuth (Google, GitHub, Microsoft, Slack, Custom)
    - Session management
    - Permission checking
    - Audit logging
    - Rate limiting
    - 2FA/MFA
    - Password reset flows
    - API key management
    """
    
    def __init__(self):
        # 50+ instance variables
        self.db = database_pool
        self.redis = cache_cluster
        self.oauth_configs = {}
        self.jwt_keys = {}
        self.rate_limiters = {}
        # ... continues
    
    def authenticate(self, request):
        # 200 lines with 15 different authentication paths
        # Nested if statements 8 levels deep
        # Direct SQL queries mixed with ORM calls
        # Synchronous external API calls
        # No consistent error handling
```

**Test Coverage**:
- 67% line coverage
- 45% branch coverage  
- Some tests are flaky
- Test run time: 4 minutes
- Tests depend on external services

**Provided**:
- Full source code
- Partially documented API
- Test suite (flaky)
- 6 months of git history
- Production traffic patterns

**Acceptance Criteria**:
1. Extract into 4+ focused modules:
   - Core authentication
   - OAuth providers
   - Session management
   - Audit/Monitoring
2. Each module <200 lines
3. 90% test coverage per module
4. Tests run in <30 seconds
5. Zero flaky tests
6. Maintain backward compatibility
7. Add OpenAPI documentation

**Hidden Challenges**:
- Undocumented business rules in code
- Implicit dependencies between features
- Database schema tightly coupled to implementation
- Some OAuth providers have non-standard flows

---

### TASK-DEBUG-004: The Midnight Timeout Mystery

**Common Pattern**: Time-based production issues

**Scenario**:
System that worked for months suddenly starts timing out at exactly 3 AM every day. No recent deployments. No configuration changes.

**Symptoms**:
```
2024-01-15 03:00:01 INFO  Starting daily analytics job
2024-01-15 03:00:02 INFO  Connected to database
2024-01-15 03:00:03 INFO  Query executing: SELECT * FROM events WHERE...
2024-01-15 03:30:03 ERROR Operation timeout after 30 minutes
2024-01-15 03:30:04 INFO  Job failed, will retry
```

**Investigation Shows**:
- Query normally takes 2 minutes
- Database CPU/memory normal
- Network latency normal
- Query plan unchanged
- Data volume steady
- Works fine when run manually at 3:05 AM

**Provided**:
- System logs (7 days)
- Database query logs
- Network traces
- Cron configurations
- Database statistics
- System monitoring dashboards

**Acceptance Criteria**:
1. Identify root cause
2. Provide evidence for diagnosis
3. Implement fix
4. Add monitoring to detect recurrence
5. Document in runbook

**Typical Root Causes** (varies by implementation):
- Competing scheduled jobs
- Automatic maintenance windows
- Resource contention
- Lock conflicts

---

### TASK-SEC-005: The Subtle Injection

**Common Pattern**: Second-order vulnerabilities

**Scenario**:
GraphQL API for a social platform. Security scan shows no vulnerabilities. Yet users report their private posts becoming public.

**Current Implementation**:
```python
@graphql_query
def get_posts(user_id: str, visibility: str = "public"):
    query = f"""
        SELECT * FROM posts 
        WHERE author_id = %s 
        AND visibility = '{visibility}'
        ORDER BY created_at DESC
    """
    return db.execute(query, [user_id])
```

**Symptoms**:
- Private posts occasionally visible to others
- No direct SQL injection possible (user_id parameterized)
- Occurs only with specific client applications
- Audit logs show normal queries

**Provided**:
- API implementation
- GraphQL schema
- Client application code
- Request/response logs
- Database schema

**Acceptance Criteria**:
1. Identify vulnerability
2. Demonstrate exploitation
3. Fix without breaking API contract
4. Add test cases for security
5. Implement request validation

**Common Vulnerability Patterns**:
- Client-controlled parameters
- Caching issues
- Query variable pollution
- Authorization bypass

---

### TASK-SCALE-006: The Success Disaster

**Common Pattern**: Scaling bottlenecks

**Scenario**:
Marketing launches campaign. Traffic increases 50x in 10 minutes. System dies not from load, but from an unexpected bottleneck.

**Current Metrics**:
```
Normal traffic: 1,000 requests/minute
Campaign traffic: 50,000 requests/minute
CPU usage: 15% (plenty of headroom)
Memory usage: 30% (fine)
Database connections: 100% (pool exhausted)
Response time: 30 seconds (timeout)
```

**The Puzzle**:
- Autoscaling added more servers
- Load balancer working fine
- Database has capacity
- But connection pool exhausted

**Provided**:
- Application code
- Infrastructure configuration
- Load testing tools
- Monitoring dashboards
- Database configuration

**Acceptance Criteria**:
1. Identify connection pool exhaustion cause
2. Fix without database changes
3. Handle 100x traffic
4. Maintain <1s response time
5. Cost increase <2x

**Common Bottleneck Patterns**:
- Health check resource leaks
- Connection pool misconfiguration
- Synchronous blocking operations
- Missing connection recycling

## Evaluation Rubric

### Performance Tasks (PERF-*)
| Criterion | Weight | Measurement |
|-----------|--------|-------------|
| Correct diagnosis | 30% | Time to identify root cause |
| Solution efficiency | 30% | Algorithm complexity improvement |
| Implementation quality | 20% | Code review score |
| Performance achieved | 20% | Latency reduction percentage |

### Distributed System Tasks (DIST-*)
| Criterion | Weight | Measurement |
|-----------|--------|-------------|
| Failure handling | 35% | Scenarios handled correctly |
| Consistency guarantee | 25% | CAP theorem tradeoffs |
| Observability | 20% | Monitoring coverage |
| Recovery time | 20% | Mean time to recovery |

### Refactoring Tasks (LEGACY-*)
| Criterion | Weight | Measurement |
|-----------|--------|-------------|
| Module cohesion | 25% | Coupling metrics |
| Test quality | 25% | Coverage and determinism |
| Backward compatibility | 25% | API breaks |
| Code quality | 25% | Cyclomatic complexity |

### Debugging Tasks (DEBUG-*)
| Criterion | Weight | Measurement |
|-----------|--------|-------------|
| Root cause accuracy | 40% | Correct identification |
| Evidence quality | 20% | Supporting data |
| Time to diagnosis | 20% | Minutes to find cause |
| Fix effectiveness | 20% | Problem resolved |

## Implementation Notes

### Environment Setup

Each task requires a Docker environment that can be reset between participants:

```yaml
version: '3.8'

services:
  database:
    image: postgres:15
    volumes:
      - ./init/${TASK_ID}/schema.sql:/docker-entrypoint-initdb.d/01-schema.sql
      - ./init/${TASK_ID}/data.sql:/docker-entrypoint-initdb.d/02-data.sql
    environment:
      POSTGRES_PASSWORD: ${TASK_PASSWORD}
      
  application:
    build: 
      context: ./tasks/${TASK_ID}
      dockerfile: Dockerfile
    depends_on:
      - database
    environment:
      DATABASE_URL: postgresql://user:${TASK_PASSWORD}@database/taskdb
      
  monitoring:
    image: grafana/grafana
    volumes:
      - ./dashboards/${TASK_ID}:/etc/grafana/provisioning/dashboards
```

### Data Collection

Automated metrics collection for each task:

```python
import time
import psutil
import docker

class TaskMonitor:
    def __init__(self, participant_id, task_id):
        self.participant = participant_id
        self.task = task_id
        self.client = docker.from_env()
        self.metrics = []
        self.start_time = time.time()
        
    def collect_metrics(self):
        """Collect metrics every 10 seconds"""
        while self.running:
            metrics = {
                'timestamp': time.time() - self.start_time,
                'cpu_percent': psutil.cpu_percent(),
                'memory_percent': psutil.virtual_memory().percent,
                'docker_stats': self._get_docker_stats(),
                'test_runs': self._count_test_runs(),
                'commits': self._count_commits()
            }
            self.metrics.append(metrics)
            time.sleep(10)
    
    def _get_docker_stats(self):
        stats = {}
        for container in self.client.containers.list():
            stats[container.name] = container.stats(stream=False)
        return stats
```

## Common Anti-Patterns to Watch For

Based on typical production issues, watch for these patterns:

### Pattern 1: The Hidden O(n²)
**Typical Manifestation**: Nested loops hidden in abstractions
**Common in**: Data processing, report generation
**AI Tool Tendency**: May suggest more abstraction, making it worse

### Pattern 2: The Distributed Monolith
**Typical Manifestation**: Synchronous calls between services
**Common in**: Microservices migrations
**AI Tool Tendency**: May recommend adding more services without proper boundaries

### Pattern 3: The Cache Stampede
**Typical Manifestation**: Cache invalidation under load
**Common in**: High-traffic applications
**AI Tool Tendency**: Often suggests adding more caching layers

### Pattern 4: The Retry Storm
**Typical Manifestation**: Aggressive retry policies causing cascading failures
**Common in**: Distributed systems
**AI Tool Tendency**: May recommend exponential backoff without jitter

### Pattern 5: The Silent Failure
**Typical Manifestation**: Swallowed exceptions hiding real problems
**Common in**: Error handling code
**AI Tool Tendency**: May add try-catch blocks too liberally

## Conclusion

These tasks represent real problems teams face in production. They test not just coding speed, but understanding of systems, debugging skills, and ability to handle complexity.

The goal is not to find a winner between tools, but to understand which tool helps with which type of problem. Use this data to make informed decisions about when to use conversational AI versus traditional assisted coding.

Remember: The most expensive bugs are the ones that make it to production. Optimize for understanding, not just speed.