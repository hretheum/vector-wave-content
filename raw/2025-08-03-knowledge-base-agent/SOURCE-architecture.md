---
content_type: SOURCE
source: architecture
verified: true
date_collected: 2025-08-03
---

# Architecture Knowledge Base Agent

## 🏗️ System Architecture Overview

### Core Components

```
┌─────────────────────────────────────────────────────────────┐
│                    Knowledge Base Agent                     │
├─────────────────────────────────────────────────────────────┤
│  ┌─────────────────┐    ┌──────────────────────────────────┐ │
│  │ Enhanced        │    │     Knowledge Adapter           │ │
│  │ Knowledge Tools │◄───┤                                  │ │
│  │                 │    │  ┌─────────────┐ ┌─────────────┐ │ │
│  │ - search_crewai │    │  │Circuit      │ │Search       │ │ │
│  │ - get_examples  │    │  │Breaker      │ │Strategies   │ │ │
│  │ - troubleshoot  │    │  │             │ │             │ │ │
│  └─────────────────┘    │  └─────────────┘ └─────────────┘ │ │
│                         │                                  │ │
│                         │  ┌──────────────────────────────┐ │ │
│                         │  │   Hybrid Search Engine       │ │ │
│                         │  │                              │ │ │
│                         │  │ KB_FIRST    FILE_FIRST      │ │ │
│                         │  │ HYBRID      KB_ONLY         │ │ │
│                         │  └──────────────────────────────┘ │ │
│                         └──────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────┘
                                    │
                 ┌──────────────────┼──────────────────┐
                 │                  │                  │
         ┌───────▼────────┐ ┌───────▼────────┐ ┌──────▼────────┐
         │  Knowledge     │ │ Local File     │ │  Circuit      │
         │  Base API      │ │ System         │ │  Breaker      │
         │                │ │                │ │  Monitor      │
         │ - Vector DB    │ │ - CrewAI docs  │ │               │
         │ - Cache        │ │ - MDX files    │ │ - Health      │
         │ - Semantic     │ │ - Local search │ │ - Recovery    │
         │   Search       │ │                │ │ - Stats       │
         └────────────────┘ └────────────────┘ └───────────────┘
```

## 🔧 Hybrid Search Strategies

### 1. KB_FIRST Strategy
```python
async def _search_kb_first(self, query: str) -> KnowledgeResponse:
    try:
        # 1. Try Knowledge Base first
        kb_response = await self.search_knowledge_base(query)
        if kb_response.results:
            return kb_response
        
        # 2. Fallback to local files if no results
        file_content = self._search_local_files(query)
        return KnowledgeResponse(...)
        
    except (AdapterError, CircuitBreakerOpen):
        # 3. Emergency fallback on failures
        return self._search_local_files(query)
```

**Use Cases**: Production environments z reliable internet

### 2. FILE_FIRST Strategy  
```python
async def _search_file_first(self, query: str) -> KnowledgeResponse:
    # 1. Search local files first (always available)
    file_content = self._search_local_files(query)
    
    # 2. Enrich with KB if available
    try:
        if not self.circuit_breaker.is_open():
            kb_response = await self.search_knowledge_base(query)
            # Combine results
    except:
        # KB unavailable, proceed with files only
        pass
```

**Use Cases**: Development environments, unreliable connectivity

### 3. HYBRID Strategy
```python
async def _search_hybrid(self, query: str) -> KnowledgeResponse:
    # 1. Search files first (guaranteed results)
    file_content = self._search_local_files(query)
    
    # 2. Simultaneously try KB for enrichment
    kb_results = []
    try:
        kb_response = await self.search_knowledge_base(query)
        kb_results = kb_response.results
    except:
        pass  # Not critical, we have file results
    
    # 3. Return combined results
    return KnowledgeResponse(
        results=kb_results,
        file_content=file_content,
        strategy_used=SearchStrategy.HYBRID
    )
```

**Use Cases**: Production z maximum reliability requirements

### 4. KB_ONLY Strategy
```python
async def _search_kb_only(self, query: str) -> KnowledgeResponse:
    try:
        return await self.search_knowledge_base(query)
    except (AdapterError, CircuitBreakerOpen) as e:
        raise AdapterError(f"KB unavailable and no fallback allowed: {e}")
```

**Use Cases**: Testing, environments z guaranteed KB availability

## ⚡ Circuit Breaker Implementation

### State Machine
```
    ┌──────────────┐     failure_count >= threshold    ┌──────────────┐
    │   CLOSED     │────────────────────────────────────►    OPEN      │
    │  (Normal)    │                                    │  (Failing)   │
    └──────┬───────┘                                    └──────┬───────┘
           │                                                   │
           │ success                              timeout elapsed │
           │                                                   │
           ▼                                                   ▼
  ┌──────────────┐                                    ┌──────────────┐
  │              │◄───────── success ─────────────────│  HALF_OPEN   │
  │   CLOSED     │                                    │  (Testing)   │
  │              │─────────── failure ─────────────────►    OPEN      │
  └──────────────┘                                    └──────────────┘
```

### Configuration
```python
@dataclass
class CircuitBreakerState:
    _threshold: int = 5              # Failures before opening
    _timeout_seconds: int = 60       # Recovery timeout
    _failure_count: int = 0
    _last_failure: Optional[datetime] = None
    _is_open: bool = False
```

### Performance Characteristics
- **Closed State**: Normal operation, <0.5ms overhead
- **Open State**: Immediate failure, <10ms response  
- **Half-Open State**: Single test request, recovery logic

## 🚀 Performance Architecture

### Connection Pooling
```python
async def _get_session(self) -> aiohttp.ClientSession:
    if self._session is None or self._session.closed:
        timeout = aiohttp.ClientTimeout(total=self.timeout)
        connector = aiohttp.TCPConnector(
            limit=10,           # Max total connections
            limit_per_host=5    # Max per host
        )
        self._session = aiohttp.ClientSession(
            timeout=timeout,
            connector=connector
        )
    return self._session
```

### Async Request Pipeline
```python
async def search_knowledge_base(self, query: str) -> KnowledgeResponse:
    # 1. Circuit breaker check (<1ms)
    if self.circuit_breaker.is_open():
        raise CircuitBreakerOpen("KB circuit breaker open")
    
    # 2. Retry loop with exponential backoff
    for attempt in range(self.max_retries + 1):
        try:
            # 3. Async HTTP request
            async with session.post(url, json=payload) as response:
                if response.status == 200:
                    # 4. Success path
                    self.circuit_breaker.record_success()
                    return KnowledgeResponse(...)
        except asyncio.TimeoutError:
            # 5. Retry with backoff
            await asyncio.sleep(2 ** attempt)
    
    # 6. All retries failed
    self.circuit_breaker.record_failure()
    raise AdapterError("KB request failed")
```

## 📊 Statistics & Observability

### Real-time Metrics
```python
@dataclass
class AdapterStatistics:
    total_queries: int = 0
    kb_successes: int = 0
    kb_errors: int = 0
    file_searches: int = 0
    strategy_usage: Dict[SearchStrategy, int]
    total_response_time_ms: float = 0.0
    
    @property
    def kb_availability(self) -> float:
        """KB availability percentage"""
        total = self.kb_successes + self.kb_errors
        return self.kb_successes / total if total > 0 else 1.0
    
    @property  
    def average_response_time_ms(self) -> float:
        """Average response time"""
        return self.total_response_time_ms / self.total_queries
```

### Health Check API
```
GET /api/v1/knowledge/health
{
  "status": "healthy",
  "components": {
    "vector_store": {"status": "healthy"},
    "cache": {"status": "healthy"},
    "circuit_breaker": {"status": "closed"}
  },
  "metrics": {
    "total_queries": 1247,
    "kb_availability": 0.985,
    "avg_response_time_ms": 0.23
  }
}
```

## 🔒 Error Handling & Recovery

### Exception Hierarchy
```python
class AdapterError(Exception):
    """Base exception for Knowledge Adapter"""

class CircuitBreakerOpen(AdapterError):
    """Circuit breaker protection active"""

class KnowledgeUnavailable(AdapterError):
    """Knowledge Base temporarily unavailable"""
```

### Recovery Strategies
1. **Graceful Degradation**: Fall back to local files
2. **Automatic Retry**: Exponential backoff (1s, 2s, 4s)
3. **Circuit Recovery**: Test requests after timeout
4. **Emergency Procedures**: Kill switch API endpoints

## 🧩 Integration Patterns

### CrewAI Tool Integration
```python
@tool("search_crewai_knowledge")
def search_crewai_knowledge(query: str, strategy: str = "HYBRID") -> str:
    """Enhanced search with circuit breaker protection"""
    try:
        adapter = get_adapter(strategy)
        response = await adapter.search(query)
        return _format_knowledge_response(response)
    except CircuitBreakerOpen:
        return "⚠️ Knowledge Base temporarily unavailable"
    except Exception as e:
        return _handle_tool_error(e, query)
```

### Backward Compatibility
```python
@tool("search_crewai_docs")  # Legacy tool name
def search_crewai_docs(query: str) -> str:
    """Backward compatible wrapper"""
    return search_crewai_knowledge(query, strategy="FILE_FIRST")
```

### Configuration Management
```python
class KnowledgeConfig:
    KB_API_URL = os.getenv("KNOWLEDGE_BASE_URL", "http://localhost:8082")
    DEFAULT_STRATEGY = SearchStrategy(os.getenv("KNOWLEDGE_STRATEGY", "HYBRID"))
    CIRCUIT_BREAKER_THRESHOLD = int(os.getenv("CIRCUIT_BREAKER_THRESHOLD", "5"))
    
    @classmethod
    def get_adapter_config(cls) -> dict:
        return {
            "strategy": cls.DEFAULT_STRATEGY,
            "kb_api_url": cls.KB_API_URL,
            "circuit_breaker_threshold": cls.CIRCUIT_BREAKER_THRESHOLD
        }
```

## 🎯 Architecture Benefits

### 1. Reliability
- **Circuit breaker** prevents cascading failures
- **Multiple fallbacks** ensure always-available knowledge
- **Automatic recovery** without manual intervention

### 2. Performance  
- **Connection pooling** reduces latency
- **Async operations** enable high concurrency
- **Local caching** via file system fallback

### 3. Flexibility
- **Strategy pattern** allows runtime configuration
- **Adapter pattern** enables easy extension
- **Configuration-driven** behavior

### 4. Observability
- **Structured logging** with correlation IDs
- **Real-time metrics** for monitoring
- **Health checks** for automated operations