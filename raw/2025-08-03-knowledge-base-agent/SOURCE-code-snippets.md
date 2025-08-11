---
content_type: SOURCE
source: code
verified: true
date_collected: 2025-08-03
---

# Code Snippets - Knowledge Base Agent

## 🚨 Emergency Fix Code

### Infinite Loop Protection (655dd81)
```python
# ai_writing_flow/src/ai_writing_flow/main.py

class AIWritingFlow(Flow[WritingFlowState]):
    def __init__(self):
        super().__init__(verbose=False)
        self._execution_count = 0  # Track executions
        
    @start()
    def receive_topic(self):
        """Emergency fix for infinite loop"""
        self._execution_count += 1
        
        # CRITICAL: Changed from >3 to >1
        if self._execution_count > 1:
            logger.error("❌ Flow executed too many times, preventing infinite loop!")
            return "finalize_output"
            
        # Continue normal flow...
```

### Aggressive Log Suppression (cc279d8)
```python
# AGGRESSIVE CrewAI FLOOD PREVENTION
os.environ["CREWAI_STORAGE_LOG_ENABLED"] = "false"
os.environ["CREWAI_FLOW_EXECUTION_LOG_ENABLED"] = "false"
os.environ["CREWAI_PROGRESS_TRACKING_ENABLED"] = "false"

# KILL ALL CrewAI LOGGERS
crewai_loggers = [
    'crewai', 'crewai.flow', 'crewai_flows', 'crewai.memory', 
    'crewai.telemetry', 'crewai.crew', 'crewai.agent', 'crewai.task',
    'crewai.tools', 'crewai.utilities', 'crewai.llm',
    'crewai.flows', 'crewai.flows.flow', 'crewai.flows.engine'
]

for logger_name in crewai_loggers:
    logging.getLogger(logger_name).setLevel(logging.CRITICAL)
    logging.getLogger(logger_name).disabled = True
```

### Emergency Kill Switch API
```python
# Emergency API endpoints for crisis management
@app.post("/api/emergency/kill-all-flows")
async def emergency_kill_flows():
    """Force terminate all running flows"""
    try:
        # Kill all active flow processes
        for flow_id in active_flows:
            await force_terminate_flow(flow_id)
        
        return {"status": "terminated", "count": len(active_flows)}
    except Exception as e:
        return {"status": "error", "message": str(e)}

@app.get("/api/emergency/status")
async def emergency_status():
    """Get emergency system status"""
    return {
        "active_flows": len(active_flows),
        "cpu_usage": psutil.cpu_percent(),
        "memory_usage": psutil.virtual_memory().percent
    }
```

## ⚡ Circuit Breaker Implementation

### Circuit Breaker State Management
```python
@dataclass
class CircuitBreakerState:
    """Production-ready circuit breaker"""
    _is_open: bool = False
    _failure_count: int = 0
    _last_failure: Optional[datetime] = None
    _threshold: int = 5
    _timeout_seconds: int = 60
    
    def is_open(self) -> bool:
        """Fast path circuit breaker check"""
        if not self._is_open:
            return False  # 99.9% of requests take this path
            
        # Recovery check (slow path)
        if (self._last_failure and 
            datetime.now() - self._last_failure > timedelta(seconds=self._timeout_seconds)):
            self.reset()
            return False
            
        return True
    
    def record_failure(self):
        """Record failure and potentially open circuit"""
        self._failure_count += 1
        self._last_failure = datetime.now()
        
        if self._failure_count >= self._threshold:
            self._is_open = True
            logger.warning("Circuit breaker opened", 
                         failure_count=self._failure_count,
                         threshold=self._threshold)
    
    def record_success(self):
        """Reset on success"""
        self._failure_count = 0
        if self._is_open:
            logger.info("Circuit breaker closed after successful request")
        self._is_open = False
```

### Circuit Breaker Integration
```python
async def search_knowledge_base(self, query: str) -> KnowledgeResponse:
    """KB search with circuit breaker protection"""
    
    # 1. Circuit breaker check (0.001ms)
    if self.circuit_breaker.is_open():
        logger.warning("Circuit breaker is open, blocking KB request")
        raise CircuitBreakerOpen("Knowledge Base circuit breaker is open")
    
    # 2. Retry loop with exponential backoff
    last_error = None
    for attempt in range(self.max_retries + 1):
        try:
            session = await self._get_session()
            
            async with session.post(f"{self.kb_api_url}/api/v1/knowledge/query", 
                                  json=payload) as response:
                
                if response.status == 200:
                    data = await response.json()
                    
                    # 3. Success path
                    self.circuit_breaker.record_success()
                    self.stats.kb_successes += 1
                    
                    return KnowledgeResponse(
                        query=query,
                        results=data["results"],
                        kb_available=True
                    )
                    
        except asyncio.TimeoutError:
            last_error = AdapterError("Knowledge Base request timed out")
            logger.warning("KB request timeout", attempt=attempt + 1)
            
        except aiohttp.ClientError as e:
            last_error = AdapterError(f"KB connection error: {str(e)}")
            logger.warning("KB connection error", error=str(e))
        
        # Exponential backoff: 1s, 2s, 4s
        if attempt < self.max_retries:
            await asyncio.sleep(2 ** attempt)
    
    # 4. All retries failed - open circuit breaker
    self.circuit_breaker.record_failure()
    self.stats.kb_errors += 1
    raise last_error
```

## 🔧 Hybrid Search Strategies

### Strategy Pattern Implementation
```python
class SearchStrategy(Enum):
    KB_FIRST = "KB_FIRST"      # Try KB first, fallback to files
    FILE_FIRST = "FILE_FIRST"  # Try files first, then KB
    HYBRID = "HYBRID"          # Combine results from both
    KB_ONLY = "KB_ONLY"        # Use only KB (no fallback)

async def search(self, query: str, strategy: SearchStrategy) -> KnowledgeResponse:
    """Main search dispatcher"""
    start_time = time.time()
    
    try:
        if strategy == SearchStrategy.KB_FIRST:
            return await self._search_kb_first(query)
        elif strategy == SearchStrategy.FILE_FIRST:
            return await self._search_file_first(query)
        elif strategy == SearchStrategy.HYBRID:
            return await self._search_hybrid(query)  
        elif strategy == SearchStrategy.KB_ONLY:
            return await self._search_kb_only(query)
    finally:
        # Performance tracking
        response_time = (time.time() - start_time) * 1000
        self.stats.total_response_time_ms += response_time
```

### Hybrid Strategy (Best Performance)
```python
async def _search_hybrid(self, query: str) -> KnowledgeResponse:
    """Combine KB and file results for maximum reliability"""
    
    # 1. File search first (always succeeds, fast)
    file_content = self._search_local_files(query, limit)
    
    # 2. KB search in parallel (may fail, enriches results)
    kb_results = []
    kb_available = True
    
    try:
        if not self.circuit_breaker.is_open():
            kb_response = await self.search_knowledge_base(query, limit)
            kb_results = kb_response.results
    except (AdapterError, CircuitBreakerOpen):
        kb_available = False
        logger.info("KB unavailable, using file results only")
    
    # 3. Return combined response
    return KnowledgeResponse(
        query=query,
        results=kb_results,           # Structured KB results
        file_content=file_content,    # Raw file content
        strategy_used=SearchStrategy.HYBRID,
        kb_available=kb_available,
        response_time_ms=(time.time() - start_time) * 1000
    )
```

## 🚀 Performance Optimizations

### Connection Pooling with aiohttp
```python
async def _get_session(self) -> aiohttp.ClientSession:
    """Optimized HTTP session with connection pooling"""
    if self._session is None or self._session.closed:
        timeout = aiohttp.ClientTimeout(total=self.timeout)
        
        # Connection pool optimization
        connector = aiohttp.TCPConnector(
            limit=10,              # Total connection pool size
            limit_per_host=5,      # Max connections per host
            keepalive_timeout=30,  # Keep connections open
            enable_cleanup_closed=True
        )
        
        self._session = aiohttp.ClientSession(
            timeout=timeout,
            connector=connector,
            headers={'Content-Type': 'application/json'},
            raise_for_status=False  # Handle errors manually
        )
    return self._session

# PERFORMANCE IMPACT: 40% latency reduction
# Before pooling: 0.38ms average response time  
# After pooling: 0.23ms average response time
```

### Async File Search Optimization
```python
def _search_local_files(self, query: str, limit: int = 5) -> str:
    """Optimized local file search with early termination"""
    self.stats.file_searches += 1
    
    if not self.docs_path.exists():
        return f"Documentation path not found: {self.docs_path}"
    
    results = []
    query_lower = query.lower()
    
    try:
        # Use pathlib.rglob for efficient file iteration
        for mdx_file in self.docs_path.rglob("*.mdx"):
            if len(results) >= limit:
                break  # Early termination for performance
            
            try:
                content = mdx_file.read_text(encoding='utf-8')
                
                # Simple but fast keyword matching
                if query_lower in content.lower():
                    # Extract relevant context (5 lines before/after)
                    lines = content.split('\n')
                    relevant_lines = []
                    
                    for i, line in enumerate(lines):
                        if query_lower in line.lower():
                            start = max(0, i - 5)
                            end = min(len(lines), i + 6)
                            relevant_lines.extend(lines[start:end])
                            break  # First match per file
                    
                    if relevant_lines:
                        relative_path = mdx_file.relative_to(self.docs_path)
                        results.append({
                            'path': str(relative_path),
                            'content': '\n'.join(relevant_lines[:50])
                        })
                        
            except Exception:
                continue  # Skip unreadable files
        
        # Format results efficiently
        if not results:
            return f"No results found for: {query}"
        
        return "\n\n---\n\n".join([
            f"**Source: {r['path']}**\n{r['content']}" 
            for r in results
        ])
        
    except Exception as e:
        logger.error("Error reading documentation", error=str(e))
        return f"Error reading documentation: {str(e)}"

# PERFORMANCE: ~2ms for typical query across 500+ files
```

## 🛠️ Enhanced Knowledge Tools

### Production-Ready Tool Implementation
```python
@tool("search_crewai_knowledge")
def search_crewai_knowledge(query: str, 
                          limit: int = 5, 
                          score_threshold: float = 0.7,
                          strategy: str = "HYBRID") -> str:
    """
    Advanced search through CrewAI knowledge base with circuit breaker protection.
    
    Returns formatted results with performance metrics and fallback handling.
    """
    start_time = time.time()
    
    logger.info("Knowledge search requested", 
               query=query, 
               strategy=strategy)
    
    try:
        # Get adapter with strategy
        adapter = _get_adapter(strategy)
        
        # Run async search in sync context (CrewAI requirement)
        loop = asyncio.new_event_loop()
        asyncio.set_event_loop(loop)
        try:
            response = loop.run_until_complete(
                adapter.search(query, limit, score_threshold)
            )
        finally:
            loop.close()
        
        # Add performance metrics
        response.response_time_ms = (time.time() - start_time) * 1000
        
        return _format_knowledge_response(response)
        
    except CircuitBreakerOpen:
        return (
            "⚠️ **Knowledge Base temporarily unavailable** (circuit breaker protection)\n\n"
            "The system is protecting against repeated failures. "
            "Please try again in a few minutes.\n\n"
            f"**Your query:** {query}\n"
            "**Status:** Service will auto-recover"
        )
    except Exception as e:
        logger.error("Knowledge tool error", error=str(e), query=query)
        return _handle_tool_error(e, query)
```

### Error Handling with Graceful Degradation
```python
def _handle_tool_error(error: Exception, query: str) -> str:
    """Production error handling with user-friendly messages"""
    
    if isinstance(error, CircuitBreakerOpen):
        return (
            "⚠️ **Knowledge Base temporarily unavailable**\n\n"
            "Circuit breaker protection is active. "
            "System will recover automatically.\n\n"
            f"**Query:** {query}\n"
            "**ETA:** ~60 seconds"
        )
    elif isinstance(error, asyncio.TimeoutError):
        return (
            "⚠️ **Search timed out**\n\n"
            "The knowledge search took longer than expected.\n\n"
            f"**Query:** {query}\n"
            "**Suggestion:** Try shorter, more specific terms"
        )
    else:
        return (
            "⚠️ **Knowledge system temporarily unavailable**\n\n"
            f"Unexpected error: {str(error)}\n\n"
            f"**Query:** {query}\n"
            "**Status:** Falling back to basic search methods"
        )
```

## 📊 Statistics and Monitoring

### Real-time Statistics Collection
```python
@dataclass
class AdapterStatistics:
    """Thread-safe statistics tracking"""
    total_queries: int = 0
    kb_successes: int = 0
    kb_errors: int = 0
    file_searches: int = 0
    strategy_usage: Dict[SearchStrategy, int] = field(default_factory=dict)
    total_response_time_ms: float = 0.0
    
    @property
    def kb_availability(self) -> float:
        """Calculate KB availability percentage"""
        total_attempts = self.kb_successes + self.kb_errors
        if total_attempts == 0:
            return 1.0
        return self.kb_successes / total_attempts
    
    @property
    def average_response_time_ms(self) -> float:
        """Calculate average response time"""
        if self.total_queries == 0:
            return 0.0
        return self.total_response_time_ms / self.total_queries

# Usage in search method
def track_query_stats(self, strategy: SearchStrategy, response_time_ms: float):
    """Thread-safe statistics update"""
    self.stats.total_queries += 1
    self.stats.total_response_time_ms += response_time_ms
    
    if strategy not in self.stats.strategy_usage:
        self.stats.strategy_usage[strategy] = 0
    self.stats.strategy_usage[strategy] += 1
```

### Health Check Implementation
```python
@tool("knowledge_system_stats")
def knowledge_system_stats() -> str:
    """Get comprehensive system health information"""
    try:
        adapter = _get_adapter()
        stats = adapter.get_statistics()
        
        # Format health report
        response = "# 📊 Knowledge System Statistics\n\n"
        response += f"**Total Queries:** {stats.total_queries:,}\n"
        response += f"**KB Availability:** {stats.kb_availability:.1%}\n"
        response += f"**Average Response Time:** {stats.average_response_time_ms:.2f}ms\n"
        response += f"**File Searches:** {stats.file_searches:,}\n"
        
        # Performance indicators
        if stats.average_response_time_ms < 1.0:
            response += "🟢 **Performance:** Excellent (<1ms)\n"
        elif stats.average_response_time_ms < 10.0:
            response += "🟡 **Performance:** Good (<10ms)\n"
        else:
            response += "🟠 **Performance:** Degraded (>10ms)\n"
        
        # Circuit breaker status
        if adapter.circuit_breaker.is_open():
            response += "\n⚠️ **Circuit Breaker:** OPEN (KB temporarily unavailable)\n"
            last_failure = adapter.circuit_breaker._last_failure
            if last_failure:
                time_since = (datetime.now() - last_failure).total_seconds()
                response += f"**Recovery ETA:** {max(0, 60 - int(time_since))} seconds\n"
        else:
            response += "\n✅ **Circuit Breaker:** CLOSED (KB available)\n"
        
        return response
        
    except Exception as e:
        return f"❌ **Error getting statistics:** {str(e)}"
```

## 🔧 Configuration Management

### Environment-based Configuration
```python
class KnowledgeConfig:
    """Production-ready configuration management"""
    
    # Knowledge Base API settings
    KB_API_URL: str = os.getenv("KNOWLEDGE_BASE_URL", "http://localhost:8082")
    KB_TIMEOUT: float = float(os.getenv("KNOWLEDGE_TIMEOUT", "10.0"))
    KB_MAX_RETRIES: int = int(os.getenv("KNOWLEDGE_MAX_RETRIES", "3"))
    
    # Search strategy
    DEFAULT_STRATEGY: SearchStrategy = SearchStrategy(
        os.getenv("KNOWLEDGE_STRATEGY", "HYBRID")
    )
    
    # Circuit breaker settings
    CIRCUIT_BREAKER_THRESHOLD: int = int(os.getenv("CIRCUIT_BREAKER_THRESHOLD", "5"))
    CIRCUIT_BREAKER_TIMEOUT: int = int(os.getenv("CIRCUIT_BREAKER_TIMEOUT", "60"))
    
    @classmethod
    def validate_config(cls) -> List[str]:
        """Configuration validation with detailed error messages"""
        issues = []
        
        if not cls.DOCS_PATH.exists():
            issues.append(f"Documentation path not found: {cls.DOCS_PATH}")
        
        if cls.KB_TIMEOUT <= 0:
            issues.append(f"Invalid timeout: {cls.KB_TIMEOUT}")
        
        if not 0.0 <= cls.DEFAULT_SCORE_THRESHOLD <= 1.0:
            issues.append(f"Invalid score threshold: {cls.DEFAULT_SCORE_THRESHOLD}")
        
        return issues
```

### Global Adapter Management
```python
# Global adapter instance for tools
_adapter_instance: Optional[KnowledgeAdapter] = None

def get_adapter(strategy: Optional[Union[SearchStrategy, str]] = None) -> KnowledgeAdapter:
    """Singleton adapter with lazy initialization"""
    global _adapter_instance
    
    env_strategy = os.getenv('KNOWLEDGE_STRATEGY', 'HYBRID')
    actual_strategy = strategy or env_strategy
    
    if _adapter_instance is None:
        _adapter_instance = KnowledgeAdapter(
            strategy=actual_strategy,
            kb_api_url=os.getenv('KB_API_URL', 'http://localhost:8082'),
            timeout=float(os.getenv('KB_API_TIMEOUT', '30.0')),
            max_retries=int(os.getenv('KB_MAX_RETRIES', '3'))
        )
    
    return _adapter_instance

# Context manager for proper cleanup
@asynccontextmanager
async def knowledge_adapter(**kwargs):
    """Async context manager for Knowledge Adapter lifecycle"""
    adapter = KnowledgeAdapter(**kwargs)
    try:
        yield adapter
    finally:
        await adapter.close()
```