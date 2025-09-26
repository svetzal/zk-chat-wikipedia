````instructions
# Project Guidelines for zk-chat-wikipedia

## Project Architecture

This is a **plugin for the zk-chat ecosystem** that provides Wikipedia content lookup functionality. The plugin follows the **Mojentic LLMTool pattern** with service provider architecture introduced in v2.0.0.

### Core Components

- **`wikipedia_content.py`**: Single-file plugin implementation containing:
  - `WikipediaContentResult` - Pydantic model for structured responses
  - `LookUpTopicOnWikipedia` - Main tool class inheriting from `mojentic.llm.tools.llm_tool.LLMTool`
- **Plugin Discovery**: Uses `pyproject.toml` entry points: `zk_rag_plugins = { wikipedia = "wikipedia_content:LookUpTopicOnWikipedia" }`

### Key Plugin Patterns

1. **Service Provider Architecture** (v2.0.0+): Constructor accepts `service_provider` parameter instead of deprecated `vault`/`llm` parameters
2. **Structured Error Handling**: Always returns `WikipediaContentResult.model_dump()` even for errors - never raises exceptions
3. **Disambiguation Handling**: Automatically selects first option from `wikipedia.DisambiguationError.options[]`
4. **Tool Descriptor**: Implements `@property descriptor` returning OpenAI function call schema

## Code Organization

### Import Structure
1. Imports should be grouped in the following order, with one blank line between groups:
   - Standard library imports
   - Third-party library imports
   - Local application imports
2. Within each group, imports should be sorted alphabetically

### Naming Conventions
1. Use descriptive variable names that indicate the purpose or content
2. Prefix test mock objects with 'mock_' (e.g., mock_wikipedia_page)
3. Prefix test data variables with 'test_' (e.g., test_topic)
4. Use '_' for unused variables or return values

### Type Hints and Documentation
1. Use type hints for method parameters and class dependencies
2. Include return type hints when the return type isn't obvious
3. Use docstrings for methods that aren't self-explanatory
4. Class docstrings should describe the purpose and behavior of the component
5. Follow numpy docstring style

### Logging Conventions
1. Use structlog for all logging
2. Initialize logger at module level using `logger = structlog.get_logger()`
3. Include relevant context data in log messages
4. Use appropriate log levels:
   - INFO for normal operations
   - DEBUG for detailed information
   - WARNING for concerning but non-critical issues
   - ERROR for critical issues
5. Use print statements only for direct user feedback

### Code Conventions
1. Do not write comments that just restate what the code does
2. Use pydantic BaseModel classes, do not use @dataclass

## Testing Guidelines

### General Rules
1. Use pytest for all testing
2. Test files:
   - Named with `_spec.py` suffix
   - Co-located with implementation files (same folder as the test subject)
3. Code style:
   - Max line length: 127
   - Max complexity: 10
4. Run tests with: `pytest`
5. Run linting with: `flake8 .`

### BDD-Style Tests
We follow a Behavior-Driven Development (BDD) style using the "Describe/should" pattern to make tests readable and focused on component behavior.

#### Test Structure
1. Tests are organized in classes that start with "Describe" followed by the component name
2. Test methods:
   - Start with "should_"
   - Describe the expected behavior in plain English
   - Follow the Arrange/Act/Assert pattern (separated by blank lines)
3. Do not use comments (eg Arrange, Act, Assert) to delineate test sections - just use a blank line
4. No conditional statements in tests - each test should fail for only one clear reason
5. Do not test private methods directly (those starting with '_') - test through the public API

#### Fixtures and Mocking
1. Use pytest @fixture for test prerequisites:
   - Break large fixtures into smaller, reusable ones
   - Place fixtures in module scope for sharing between classes
   - Place module-level fixtures at the top of the file
2. Mocking:
   - Use pytest's `mocker` for dependencies
   - Use Mock's spec parameter for type safety (e.g., `Mock(spec=ServiceProvider)`)
   - Mock external libraries like `wikipedia.search` and `wikipedia.page`
   - Do not mock library internals or private functions
   - Do not use unittest or MagicMock directly

#### Best Practices
1. Test organization:
   - Place instantiation/initialization tests first
   - Group related scenarios together (success and failure cases)
   - Keep tests focused on single behaviors
2. Assertions:
   - One assertion per line for better error identification
   - Use 'in' operator for partial string matches
   - Use '==' for exact matches
3. Test data:
   - Use fixtures for reusable prerequisites
   - Define complex test data structures within test methods

### Plugin-Specific Testing Patterns
- **Mock Heavy**: Patches `wikipedia.search` and `wikipedia.page` for all external API calls
- **Error Scenarios**: Tests handle no results, disambiguation, and general exceptions
- **Response Format**: Always test that methods return properly formatted `.model_dump()` dictionaries

## Plugin Development Best Practices

### 1. Error Handling
Always handle errors gracefully and return meaningful error messages:

```python
def run(self, input_data: str) -> str:
    try:
        # Plugin logic here
        return result
    except Exception as e:
        logger.error("Plugin error", error=str(e))
        return f"Error in plugin: {str(e)}"
```

### 2. Input Validation
Validate inputs and provide clear feedback:

```python
def run(self, topic: str) -> str:
    if not topic:
        return WikipediaContentResult(
            title="Error",
            content="Error: topic parameter is required",
            url=None
        ).model_dump()

    if not topic.strip():
        return WikipediaContentResult(
            title="Error",
            content="Error: topic cannot be empty",
            url=None
        ).model_dump()

    # Continue with plugin logic...
```

### 3. Descriptive Function Descriptors
Make your tool easy for the LLM to understand and use:

```python
@property
def descriptor(self) -> dict:
    return {
        "type": "function",
        "function": {
            "name": "lookup_topic_on_wikipedia",
            "description": "Retrieves information about a given topic from Wikipedia. Use this when the user asks for factual information about people, places, concepts, or events that would be found in an encyclopedia.",
            "parameters": {
                "type": "object",
                "properties": {
                    "topic": {
                        "type": "string",
                        "description": "The topic to search for on Wikipedia (e.g., 'Albert Einstein', 'Machine Learning', 'Paris France')"
                    }
                },
                "required": ["topic"]
            }
        }
    }
```

### 4. Use Structured Logging
Include relevant context in your log messages:

```python
logger.info(
    "Wikipedia lookup completed",
    topic=topic,
    results_found=len(search_results),
    page_title=page.title if page else None
)
```

### 5. Service Availability Checking
Check service availability when using optional services:

```python
def check_services(self):
    """Check what services are available."""
    provider = self.service_provider

    # Most services should be available, but check optional ones
    if provider.has_service(ServiceType.GIT_GATEWAY):
        git = provider.get_git_gateway()
        # Git operations are available

    # For required services, use require_service
    try:
        config = provider.require_service(ServiceType.CONFIG)
    except RuntimeError as e:
        return f"Required service not available: {e}"
```

## Available Runtime Services

### Service Provider Access Pattern
Use the service provider to access zk-chat services:

```python
from zk_chat.services import ServiceProvider, ServiceType

def __init__(self, service_provider: ServiceProvider):
    super().__init__()
    self.service_provider = service_provider

    # Access services through convenient methods
    config = service_provider.get_config()
    llm = service_provider.get_llm_broker()
    # Note: Wikipedia plugin typically doesn't need vault access
```

### Available Services for Plugins

#### 1. Configuration Service
- **Purpose**: Access to application configuration (vault path, model settings)
- **Usage**: Get vault path, model configuration
- **Access**: `service_provider.get_config()`

#### 2. LLM Broker (Optional for Wikipedia)
- **Purpose**: Make requests to the configured LLM if needed
- **Usage**: Could be used for content analysis or summarization
- **Access**: `service_provider.get_llm_broker()`

#### 3. Smart Memory (Optional)
- **Purpose**: Store information across chat sessions
- **Usage**: Remember frequently searched topics or user preferences
- **Access**: `service_provider.get_smart_memory()`

Note: Wikipedia plugin primarily uses external Wikipedia API and doesn't typically need vault or filesystem access.

### Example

```python
class DescribeLookUpTopicOnWikipedia:
    """
    Tests for the Wikipedia lookup tool plugin
    """
    def should_be_instantiated_with_service_provider(self):
        mock_service_provider = Mock(spec=ServiceProvider)

        tool = LookUpTopicOnWikipedia(mock_service_provider)

        assert isinstance(tool, LookUpTopicOnWikipedia)
        assert tool.service_provider == mock_service_provider
```

## Development Workflow

### Local Setup
```bash
pip install -e .[dev]  # Editable install with dev dependencies
```

### Testing Commands
```bash
pytest                    # Run all tests with spec output
python -m build          # Build wheel and source dist
flake8 .                 # Run linting
```

## Integration Points

### Plugin Registration
The tool is auto-discovered via `pyproject.toml` entry points when zk-chat scans for plugins. The `wikipedia` key maps to the class.

### External Dependencies
- **`wikipedia`** library: Direct API wrapper, no authentication needed
- **`mojentic`**: Plugin framework providing `LLMTool` base class

### Response Format
Tools must return `dict` from `.model_dump()` - this is consumed by the zk-chat framework for LLM function calls.

## Release Process

This project follows [Semantic Versioning](https://semver.org/) (SemVer) for version numbering. The version format is MAJOR.MINOR.PATCH, where:

1. MAJOR version increases for incompatible API changes
2. MINOR version increases for backward-compatible functionality additions
3. PATCH version increases for backward-compatible bug fixes

### Preparing a Release

When preparing a release, follow these steps:

1. **Update CHANGELOG.md**:
   - Move items from the "[Next]" section to a new version section
   - Add the new version number and release date: `## [x.y.z] - YYYY-MM-DD`
   - Ensure all changes are properly categorized under "Added", "Changed", "Deprecated", "Removed", "Fixed", or "Security"
   - Keep the empty "[Next]" section at the top for future changes

2. **Update Version Number**:
   - Update the version number in `pyproject.toml`
   - Ensure the version number follows semantic versioning principles based on the nature of changes:
     - **Major Release**: Breaking changes that require users to modify their code
     - **Minor Release**: New features that don't break backward compatibility
     - **Patch Release**: Bug fixes that don't add features or break compatibility

3. **Update Documentation**:
   - Review and update `README.md` to reflect any new features, changed behavior, or updated requirements
   - Update any other documentation files that reference features or behaviors that have changed
   - Ensure installation instructions and examples are up to date

4. **Synchronize Dependencies**:
   - Ensure that dependencies in `pyproject.toml` optional-dependencies match dev requirements
   - Update version constraints if necessary

5. **Final Verification**:
   - Run all tests to ensure they pass
   - Verify that the plugin works as expected with the updated version
   - Check that all documentation accurately reflects the current state of the project

### Release Types

#### Major Releases (x.0.0)

Major releases may include:
- Breaking API changes (eg service provider interface changes)
- Significant architectural changes
- Removal of deprecated features
- Changes that require users to modify their code or workflow

For major releases, consider:
- Providing migration guides
- Updating all documentation thoroughly
- Highlighting breaking changes prominently in the CHANGELOG

#### Minor Releases (0.x.0)

Minor releases may include:
- New Wikipedia lookup features
- Non-breaking enhancements
- Deprecation notices (but not removal of deprecated features)
- Performance improvements

For minor releases:
- Document all new features
- Update README to highlight new capabilities
- Ensure backward compatibility

#### Patch Releases (0.0.x)

Patch releases should be limited to:
- Bug fixes
- Security updates
- Performance improvements that don't change behavior
- Documentation corrections

For patch releases:
- Clearly describe the issues fixed
- Avoid introducing new features
- Maintain strict backward compatibility

## Common Tasks

### Adding New Features
1. Extend `WikipediaContentResult` model if response structure changes
2. Modify `run()` method for new Wikipedia API interactions
3. Update `descriptor` property for new function parameters
4. Add corresponding test methods in `DescribeLookUpTopicOnWikipedia`

### Publishing
GitHub Actions automatically publishes to PyPI on release creation using trusted publishing (no API keys needed).
````