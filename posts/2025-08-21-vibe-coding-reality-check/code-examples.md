# Code Examples - Vibe Coding Reality Check

## The Problem: Slow CrewAI Tools

### Before Optimization (25-35s response times)

```python
# Recipe Search Tool with hidden performance killers
class RecipeSearchTool(BaseTool):
    name: str = "recipe_search"
    description: str = "Search for recipes based on ingredients and preferences"
    
    def _run(self, search_params) -> str:
        # Mock data generation
        mock_results = {
            "recipes": [
                {
                    "id": "recipe_001",
                    "name": "Quick Pasta Carbonara",
                    "ingredients": ["pasta", "eggs", "bacon", "parmesan"],
                    "prep_time": 20,
                    "difficulty": "easy"
                }
            ]
        }
        
        # PERFORMANCE KILLER #1: Artificial delay
        time.sleep(2.5)  # Simulating API call
        
        # PERFORMANCE KILLER #2: Pretty printing
        return json.dumps(mock_results, indent=2)


# Price Comparison Tool with O(n³) complexity
class PriceComparisonTool(BaseTool):
    name: str = "price_comparison"
    description: str = "Compare prices across different stores"
    
    def calculate_total_cost(self, ingredients):
        total_costs = {}
        
        # PERFORMANCE KILLER: Triple nested loops
        for ingredient in ingredients:
            for store in ["walmart", "kroger", "whole_foods", "target"]:
                for category in ["produce", "dairy", "meat", "pantry"]:
                    # Simulating database lookup
                    time.sleep(0.1)
                    
                    # Complex price calculation
                    base_price = self.get_base_price(ingredient, store, category)
                    tax = base_price * 0.08
                    total_costs[f"{store}_{ingredient}"] = base_price + tax
        
        return total_costs
```
## The Solution: Optimized Implementation

### After Optimization (29ms response times)

```python
# Optimized Recipe Search Tool
class RecipeSearchTool(BaseTool):
    name: str = "recipe_search"
    description: str = "Search for recipes using real API with timeout protection"
    
    def _run(self, search_params) -> str:
        recipe_service_url = os.getenv("RECIPE_SERVICE_URL", "http://recipe-service:8000")
        
        # Real API call with timeout protection
        timeout = httpx.Timeout(connect=5.0, read=10.0, write=5.0, pool=5.0)
        
        try:
            with httpx.Client(timeout=timeout) as client:
                response = client.get(
                    f"{recipe_service_url}/recipes/search",
                    params=search_params
                )
                # Compact JSON, no pretty printing
                return json.dumps(response.json(), separators=(',', ':'))
        except httpx.TimeoutException:
            # Fast fallback
            return json.dumps({"error": "timeout", "fallback": True})


# Hash-based O(1) Price Calculation
import hashlib

def get_consistent_price(ingredient_name: str, store: str) -> float:
    """Generate consistent prices using hash function - O(1) complexity"""
    # Create unique hash for ingredient+store combination
    hash_input = f"{ingredient_name}_{store}".encode()
    hash_value = int(hashlib.md5(hash_input).hexdigest()[:8], 16)
    
    # Generate price between 0.50 and 9.99
    base_price = (hash_value % 950) / 100.0 + 0.50
    
    # Store-specific multipliers
    store_multipliers = {
        "walmart": 0.9,
        "kroger": 0.95,
        "whole_foods": 1.3,
        "target": 1.0
    }
    
    return round(base_price * store_multipliers.get(store, 1.0), 2)
```
## Circuit Breaker Pattern

```python
import time
import asyncio
from typing import Optional, Callable

class CircuitBreaker:
    """Prevent cascading failures in distributed AI systems"""
    
    def __init__(self, failure_threshold: int = 5, timeout_seconds: int = 60):
        self.failure_threshold = failure_threshold
        self.timeout_seconds = timeout_seconds
        self.failures = 0
        self.opened_at: Optional[float] = None
        self.success_count = 0
    
    async def protected(self, func: Callable):
        """Execute function with circuit breaker protection"""
        
        # Check if circuit is open
        if self.opened_at:
            if time.time() - self.opened_at < self.timeout_seconds:
                raise Exception(f"Circuit breaker is open (failures: {self.failures})")
            else:
                # Try to close circuit after timeout
                self.opened_at = None
                self.failures = 0
        
        try:
            result = await func()
            self.success_count += 1
            
            # Reset failures on success
            if self.success_count > 2:
                self.failures = 0
            
            return result
            
        except Exception as e:
            self.failures += 1
            self.success_count = 0
            
            if self.failures >= self.failure_threshold:
                self.opened_at = time.time()
                print(f"Circuit breaker opened after {self.failures} failures")
            
            raise e


# Usage with CrewAI agents
circuit_breaker = CircuitBreaker(failure_threshold=3, timeout_seconds=30)

async def execute_agent_with_protection(agent, task):
    """Execute CrewAI agent with circuit breaker protection"""
    
    async def agent_execution():
        return await agent.execute(task)
    
    try:
        return await circuit_breaker.protected(agent_execution)
    except Exception as e:
        # Return fallback response
        return {
            "status": "circuit_open",
            "message": "Service temporarily unavailable",
            "fallback": True,
            "suggestions": ["Try simpler query", "Wait 30 seconds"]
        }
```
## Docker Networking Issues

### The Problem Configuration

```yaml
# WRONG: Using external ports for internal communication
version: '3.8'

services:
  api-gateway:
    build: ./api-gateway
    ports:
      - "8080:8000"  # External:Internal
    environment:
      # WRONG: Using external port 8001
      - RECIPE_SERVICE_URL=http://recipe-service:8001
      - AGENT_SERVICE_URL=http://agent-service:8004
    depends_on:
      - recipe-service
      - agent-service
  
  recipe-service:
    build: ./recipe-service
    ports:
      - "8001:8000"  # External:Internal
    
  agent-service:
    build: ./agent-service
    ports:
      - "8004:8000"  # External:Internal
```

### The Fixed Configuration

```yaml
# CORRECT: Using internal Docker network ports
version: '3.8'

services:
  api-gateway:
    build: ./api-gateway
    ports:
      - "8080:8000"
    environment:
      # CORRECT: Using internal port 8000
      - RECIPE_SERVICE_URL=http://recipe-service:8000
      - AGENT_SERVICE_URL=http://agent-service:8000
    depends_on:
      - recipe-service
      - agent-service
    networks:
      - cookbook-network
  
  recipe-service:
    build: ./recipe-service
    ports:
      - "8001:8000"
    networks:
      - cookbook-network
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:8000/health"]
      interval: 30s      # Reasonable interval
      timeout: 10s       # Enough time for AI services
      retries: 3
      start_period: 60s  # Time to start up
    
  agent-service:
    build: ./agent-service
    ports:
      - "8004:8000"
    networks:
      - cookbook-network
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:8000/health"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 90s  # More time for AI model loading

networks:
  cookbook-network:
    driver: bridge
```
## AI-Powered Debugging Conversation

### The Actual Conversation That Saved Us

```markdown
Me: The system takes 25-35 seconds to respond. Users think it's broken. 
    Health checks pass, services are running, but browser shows timeout errors.

Claude: Let me analyze this systematically. The symptoms suggest the backend is 
        working but taking too long. Let's trace the execution path:
        1. Can you show me the CrewAI tool implementations?
        2. What do the OpenTelemetry traces show for a typical request?

Me: [Pastes RecipeSearchTool code with time.sleep(2.5)]

Claude: I found a critical issue - there's a `time.sleep(2.5)` in your _run method. 
        This adds 2.5 seconds of artificial delay per tool call. With 3 tools 
        running sequentially, that's 7.5 seconds of unnecessary waiting.
        
        Also, you're using `json.dumps(mock_results, indent=2)` which pretty-prints 
        the JSON. For large responses, this can add 100-200ms.

Me: Oh my god. But even removing that, we're still at 15-20 seconds.

Claude: Let's look at the PriceComparisonTool next. Can you share that code?

Me: [Pastes the triple-nested loop code]

Claude: This is O(n³) complexity. With 20 ingredients, 4 stores, and 4 categories, 
        you're doing 320 iterations with a 0.1s sleep in each. That's 32 seconds 
        alone! 
        
        Replace this with hash-based pricing:
        ```python
        def get_consistent_price(ingredient_name: str) -> float:
            hash_value = int(hashlib.md5(ingredient_name.encode()).hexdigest()[:8], 16)
            base_price = (hash_value % 1000) / 100.0
            return max(0.50, base_price)
        ```

Me: Applied. Down to 8 seconds. Still too slow.

Claude: Now let's check service communication. Are your services communicating 
        using Docker internal ports or external ports?

Me: ...what's the difference?

Claude: In Docker Compose, services should communicate using internal ports 
        (usually 8000) not the external mapped ports (8001, 8004, etc). 
        Check your environment variables.

Me: RECIPE_SERVICE_URL=http://recipe-service:8001

Claude: That's your problem. Change it to :8000 (the internal port). The external 
        port 8001 only works from outside the Docker network, causing routing 
        delays and potential failures.

Me: Changed all services to internal ports. Response time is now... 29ms. 
    TWENTY-NINE MILLISECONDS.

Claude: Excellent! That's a 1000x improvement. The combination of removing 
        artificial delays, optimizing algorithms from O(n³) to O(1), and fixing 
        network configuration brought you from 25,000ms to 29ms.
        
        Now add circuit breakers to prevent future cascading failures:
        [Provides circuit breaker code]

Me: This is insane. We literally used AI to debug AI agents.

Claude: Meta, isn't it? Sometimes the best debugger is another intelligence that 
        can pattern-match across different failure modes simultaneously.
```

### The Debugging Agent Chain

```python
# How we automated this debugging process
async def emergency_diagnosis():
    """Use AI agents to diagnose system failures"""
    
    # Phase 1: System health analysis
    health_report = await debugger_agent.analyze_system_health()
    
    # Phase 2: Crisis coordination
    recovery_plan = await emergency_controller_agent.create_recovery_plan(
        health_report
    )
    
    # Phase 3: Architecture review
    architecture_issues = await architecture_advisor.identify_antipatterns()
    
    # Phase 4: Generate fixes
    fixes = await code_generator_agent.create_patches(
        issues=architecture_issues,
        constraints=["maintain backward compatibility", "< 100ms response time"]
    )
    
    return {
        "diagnosis": health_report,
        "recovery_plan": recovery_plan,
        "fixes": fixes,
        "estimated_improvement": "1000x based on pattern analysis"
    }
```