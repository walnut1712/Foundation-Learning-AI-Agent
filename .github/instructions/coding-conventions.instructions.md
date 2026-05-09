---
description: "Use when writing Python code. Enforces naming conventions, error-handling patterns, logging standards, async/await best practices, and testing conventions for consistency and maintainability."
name: "Coding Conventions"
applyTo: "**/*.py"
---

# Coding Conventions & Standards

This guide enforces consistent naming, error-handling, logging, async patterns, and testing practices across the project.

---

## Naming Conventions

### Variables & Functions

- **snake_case** for variables, functions, and method names
  - ✓ `process_data()`, `user_profile`, `max_retries`
  - ✗ `ProcessData()`, `userProfile`, `maxRetries`

- **UPPER_SNAKE_CASE** for constants
  - ✓ `MAX_RETRIES = 3`, `DEFAULT_TIMEOUT = 30`
  - ✗ `MaxRetries`, `defaultTimeout`

- **PascalCase** for classes and custom exceptions
  - ✓ `class DataProcessor:`, `class ValidationError(Exception):`
  - ✗ `class data_processor:`, `class validation_error:`

### Booleans & Flags

- **Prefix with `is_`, `has_`, `should_`, `can_`** for clarity
  - ✓ `is_valid`, `has_error`, `should_retry`, `can_proceed`
  - ✗ `valid`, `error`, `retry`, `proceed`

- **Avoid negations in names** (use positive form)
  - ✓ `is_enabled` instead of `is_not_disabled`
  - ✗ `is_disabled` when you need to check "if enabled"

### Files & Modules

- **snake_case** for filenames
  - ✓ `data_processor.py`, `test_api_handler.py`
  - ✗ `DataProcessor.py`, `testApiHandler.py`

- **Descriptive, single responsibility** per module
  - ✓ `api_client.py`, `database_models.py`
  - ✗ `utilities.py`, `helpers.py` (too vague)

---

## Error Handling

### Exception Strategy

- **Catch specific exceptions**, not bare `except:`
  - ✓ `except ValueError:`, `except (KeyError, TypeError):`
  - ✗ `except:`, `except Exception:`

- **Create custom exceptions** for application-level errors

  ```python
  class DataValidationError(ValueError):
      """Raised when data fails validation."""
      pass

  class RetryExhaustedError(RuntimeError):
      """Raised when max retries exceeded."""
      pass
  ```

- **Raise with context** using `raise ... from e` to preserve tracebacks
  ```python
  try:
      result = api_call()
  except requests.Timeout as e:
      raise APITimeoutError(f"Request timed out after {timeout}s") from e
  ```

### Error Messages

- **Include context** in error messages: what, why, actionable next step
  - ✓ `"Failed to parse CSV: missing 'date' column at row 5. Ensure CSV has required headers."`
  - ✗ `"Error"`, `"Invalid data"`

- **Log before raising** critical errors (exception details logged, user sees sanitized message)
  ```python
  logger.error(f"Database connection failed: {db_error}", exc_info=True)
  raise ServiceUnavailableError("Database temporarily unavailable. Try again later.")
  ```

### Try/Except Blocks

- **Keep try blocks small** — wrap only the operation that might fail

  ```python
  try:
      data = json.loads(response_text)
  except json.JSONDecodeError as e:
      logger.error(f"Invalid JSON: {e}")
      raise
  ```

- **Always use finally for cleanup** if needed (file handles, connections)
  ```python
  try:
      result = process_file(filepath)
  except IOError as e:
      logger.error(f"File error: {e}")
  finally:
      file_handle.close()  # Always runs
  ```

---

## Logging

### Log Levels & Usage

- **DEBUG**: Detailed diagnostic info (variable values, function entry/exit)

  ```python
  logger.debug(f"Processing dataset with {len(data)} rows")
  ```

- **INFO**: General informational messages (startup, major milestones)

  ```python
  logger.info("Data analysis complete. Generated 5 charts.")
  ```

- **WARNING**: Something unexpected, but not an error (deprecated usage, unusual values)

  ```python
  logger.warning(f"Inflation value {inflation} exceeds typical range [0%, 10%]")
  ```

- **ERROR**: An error occurred, but execution continues

  ```python
  logger.error(f"Failed to save results: {io_error}", exc_info=True)
  ```

- **CRITICAL**: System in severe distress, likely to fail
  ```python
  logger.critical("Database unreachable. Cannot proceed with analysis.")
  ```

### Logging Setup

- **Initialize logger per module** at the top

  ```python
  import logging
  logger = logging.getLogger(__name__)
  ```

- **Include context** in log messages (who, what, why)

  ```python
  # Good
  logger.info(f"User '{user_id}' uploaded file '{filename}' ({file_size} bytes)")

  # Poor
  logger.info("File uploaded")
  ```

- **Use exc_info=True for exceptions**
  ```python
  except Exception as e:
      logger.error(f"Processing failed: {e}", exc_info=True)
  ```

### What NOT to Log

- ✗ Sensitive data (passwords, API keys, PII)
- ✗ Entire request/response bodies (log summaries instead)
- ✗ Redundant info (if logged elsewhere in the call chain)
- ✓ Always sanitize external input before logging

---

## Async & Concurrency

### Async/Await Conventions

- **Use async/await** for I/O-bound operations (HTTP, file, database)

  ```python
  async def fetch_data(url: str) -> dict:
      async with aiohttp.ClientSession() as session:
          async with session.get(url) as response:
              return await response.json()
  ```

- **Never block event loop** with sync operations in async code
  - ✗ `time.sleep(1)` inside async function
  - ✓ `await asyncio.sleep(1)`

- **Name async functions clearly** — prefix with `async_` if ambiguity
  ```python
  async def async_process_data(data):
      """Async variant. Use when you need concurrent processing."""
      pass
  ```

### Running Async Code

- **Use asyncio.run()** in main entry point (Python 3.7+)

  ```python
  if __name__ == "__main__":
      result = asyncio.run(main())
  ```

- **Gather multiple tasks** with `asyncio.gather()`

  ```python
  results = await asyncio.gather(
      fetch_data(url1),
      fetch_data(url2),
      fetch_data(url3),
      return_exceptions=True
  )
  ```

- **Set timeouts** for async operations
  ```python
  try:
      result = await asyncio.wait_for(fetch_data(url), timeout=10)
  except asyncio.TimeoutError:
      logger.error(f"Request to {url} timed out after 10s")
  ```

### Threading (When Async Unavailable)

- **Use ThreadPoolExecutor** for sync functions in async context (rare)

  ```python
  loop = asyncio.get_event_loop()
  result = await loop.run_in_executor(None, blocking_function, arg)
  ```

- **Never use bare threading** for new code—use asyncio instead

---

## Testing Conventions

### Test Structure

- **Test file naming**: `test_<module>.py` (e.g., `test_api_client.py`)

  ```
  src/
    api_client.py
  tests/
    test_api_client.py
  ```

- **Test function naming**: `test_<function>_<scenario>` (descriptive, no "test\_" prefix overload)
  - ✓ `test_process_data_with_valid_input()`
  - ✓ `test_process_data_raises_on_invalid_input()`
  - ✗ `test_process()`, `test1()`, `test_bad()`

### Test Organization

- **One assert per test or use multiple asserts for related conditions**

  ```python
  def test_calculate_growth_returns_correct_value():
      result = calculate_growth(gdp_2022=8.0, gdp_2021=2.6)
      assert result == pytest.approx(5.4, rel=0.01)
  ```

- **Use descriptive test names** as documentation
  ```python
  def test_api_client_retries_on_timeout():
      """Verifies that client retries up to 3 times on timeout."""
      pass
  ```

### Setup & Fixtures

- **Use pytest fixtures** for reusable setup (not setUp methods)

  ```python
  @pytest.fixture
  def sample_dataset():
      """Provides cleaned economic dataset for tests."""
      return {
          'years': [2019, 2020, 2021, 2022, 2023],
          'gdp_growth': [7.0, 2.9, 2.6, 8.0, 5.1]
      }

  def test_analyze_with_sample_data(sample_dataset):
      result = analyze(sample_dataset)
      assert result is not None
  ```

- **Keep fixtures focused** — one concern per fixture
- **Use autouse sparingly** — explicit is better than implicit

### Assertions & Matchers

- **Be specific** with assertions
  - ✓ `assert result == expected_value`
  - ✗ `assert result` (ambiguous)

- **Use pytest.approx()** for floating-point comparisons

  ```python
  assert inflation == pytest.approx(3.2, abs=0.1)
  ```

- **Use pytest.raises()** for exception testing
  ```python
  with pytest.raises(ValueError, match="Missing data"):
      process_data({})
  ```

### Test Categories

- **Unit tests**: Test single function in isolation (mocking external deps)

  ```python
  def test_calculate_average_with_positive_numbers():
      assert calculate_average([1, 2, 3]) == 2.0
  ```

- **Integration tests**: Test multiple components together

  ```python
  def test_full_analysis_pipeline(sample_csv_file):
      """Tests data loading → cleaning → analysis → report generation."""
      pass
  ```

- **Mark integration tests** to run separately
  ```python
  @pytest.mark.integration
  def test_database_connection():
      pass
  ```

### Coverage & CI

- **Target 80%+ code coverage** for production code
  - Use `pytest --cov=src` to measure
  - Skip obvious patterns: `pragma: no cover` for hard-to-reach code

- **Run tests in CI** before merge (GitHub Actions, etc.)
  ```yaml
  - name: Run tests
    run: pytest tests/ -v --cov=src
  ```

---

## Summary

| Area        | Key Rules                                                                                                |
| ----------- | -------------------------------------------------------------------------------------------------------- |
| **Naming**  | snake*case for functions/vars, PascalCase for classes, prefix bools with is*/has*/should*                |
| **Errors**  | Specific exceptions, custom error classes, include context in messages, raise from original exception    |
| **Logging** | Use appropriate levels (DEBUG/INFO/WARNING/ERROR/CRITICAL), include context, sanitize sensitive data     |
| **Async**   | Use async/await for I/O, never block event loop, gather tasks, set timeouts                              |
| **Testing** | test\_<module>.py naming, descriptive test names, fixtures for setup, specific assertions, 80%+ coverage |

---

**Apply these conventions to all Python files in the project for consistency and maintainability.**
