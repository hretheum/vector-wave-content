# Prompt for Self-Debugging Agent POC Code Generation

## Task

Create a minimal proof-of-concept for a self-debugging AI agent in Python. This should be a complete, runnable implementation that demonstrates agents fixing their own code errors.

## Requirements

- Python 3.10+
- OpenAI API (user will provide key)
- Docker for sandboxing (with fallback to subprocess)
- SQLite for metrics tracking
- Total code: ~250 lines

## File Structure

### 1. agent.py (150 lines)

```python
class MinimalDebugger:
    def fix_code(self, code, error, max_tries=3):
        # GPT-4 with temperature=0.2
        # Track tokens: completion.usage.total_tokens
        # Handle rate limits with 2^n backoff (max 60s)
        
    def test_in_sandbox(self, code):
        # Try Docker first, fallback to subprocess
        # Return (success: bool, output: str)
```

### 2. sandbox.py (50 lines)

```python
class DockerSandbox:
    def run_code(self, code, timeout=5):
        # docker run --rm -m 100m python:3.10-slim
        # Write to /tmp/test.py, execute
        # Fallback: subprocess.run with timeout
```

### 3. test_bugs.py (30 lines)

Use these EXACT bugs:

```python
bugs = [
    # Syntax errors
    ("print('hello'", "SyntaxError: unexpected EOF while parsing"),
    ("def func()\n  return 42", "SyntaxError: invalid syntax"),
    ("for i in range(10) print(i)", "SyntaxError: invalid syntax"),
    
    # Runtime errors  
    ("x = [1,2,3]\nprint(x[5])", "IndexError: list index out of range"),
    ("print(undefined_var)", "NameError: name 'undefined_var' is not defined"),
    ("5 / 0", "ZeroDivisionError: division by zero"),
    
    # Logic errors
    ("def add(a,b):\n  return a - b", "Logic: should add not subtract"),
    ("if x = 5:\n  print('yes')", "SyntaxError: invalid syntax"),
    ("while True\n  print('loop')", "SyntaxError: invalid syntax"),
    ("list = [1,2,3]\nlist.append[4]", "TypeError: 'builtin_function_or_method'")
]
```

### 4. run_benchmark.py

```python
# Initialize DB with schema
# Run each bug through agent
# Print: "Bug 1/10: Fixed in 2 attempts (0.003 tokens)"
# Final: "Success: 7/10 (70%), Avg attempts: 2.1, Cost: $0.03"
```

### 5. Database schema

```sql
CREATE TABLE IF NOT EXISTS attempts (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    timestamp DATETIME DEFAULT CURRENT_TIMESTAMP,
    error_type TEXT,
    original_code TEXT,
    fixed_code TEXT,
    attempts INTEGER,
    success BOOLEAN,
    tokens_used INTEGER,
    cost_usd REAL
);
```

### 6. docker-compose.yml

```yaml
version: '3'
services:
  sandbox:
    image: python:3.10-slim
    command: tail -f /dev/null
    mem_limit: 100m
    cpus: 0.5
```

### 7. requirements.txt

```
openai>=1.0.0
docker>=6.0.0
python-dotenv>=1.0.0
```

## Implementation Details

### OpenAI prompt template

```
Fix this Python code. Return ONLY the corrected code, no explanations.

Code:
{code}

Error:
{error}

Fixed code:
```

### Rate limiting

```python
def call_openai_with_retry(prompt, max_retries=3):
    for i in range(max_retries):
        try:
            return openai.chat.completions.create(...)
        except RateLimitError:
            time.sleep(2 ** i)  # 1s, 2s, 4s
    return None
```

### Cost calculation

```python
# GPT-4: $0.03/1K input, $0.06/1K output tokens
cost = (prompt_tokens * 0.03 + completion_tokens * 0.06) / 1000
```

### Docker fallback

```python
try:
    result = docker_client.containers.run(...)
except:
    # Fallback to subprocess
    result = subprocess.run(['python', '-c', code], 
                          capture_output=True, 
                          timeout=5)
```

## Expected output format

```
Starting benchmark...
Bug 1/10: SyntaxError - Fixed in 1 attempt (247 tokens, $0.008)
Bug 2/10: SyntaxError - Fixed in 2 attempts (412 tokens, $0.014)
...
Results:
- Success rate: 70% (7/10)
- Average attempts: 2.1
- Total tokens: 3,847
- Total cost: $0.127
```

## Important Notes

- Make it immediately runnable with clear error messages if Docker/OpenAI unavailable
- Include proper error handling for all external dependencies
- No placeholders or TODO comments - fully functional code only