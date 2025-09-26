# AI Coding Agent Instructions for zk-chat-wikipedia

## Project Architecture

This is a **plugin for the zk-chat ecosystem** that provides Wikipedia content lookup functionality. The plugin follows the **Mojentic LLMTool pattern** with service provider architecture introduced in v2.0.0.

### Core Components

- **`wikipedia_content.py`**: Single-file plugin implementation containing:
  - `WikipediaContentResult` - Pydantic model for structured responses
  - `LookUpTopicOnWikipedia` - Main tool class inheriting from `mojentic.llm.tools.llm_tool.LLMTool`
- **Plugin Discovery**: Uses `pyproject.toml` entry points: `zk_rag_plugins = { wikipedia = "wikipedia_content:LookUpTopicOnWikipedia" }`

### Key Patterns

1. **Service Provider Architecture** (v2.0.0+): Constructor accepts `service_provider` parameter instead of deprecated `vault`/`llm` parameters
2. **Structured Error Handling**: Always returns `WikipediaContentResult.model_dump()` even for errors - never raises exceptions
3. **Disambiguation Handling**: Automatically selects first option from `wikipedia.DisambiguationError.options[]`
4. **Tool Descriptor**: Implements `@property descriptor` returning OpenAI function call schema

## Development Workflow

### Local Setup
```bash
pip install -e .[dev]  # Editable install with dev dependencies
```

### Testing Strategy
- **BDD Style**: Uses `Describe*` classes with `should_*` methods (see `wikipedia_content_spec.py`)
- **pytest-spec**: Configured for specification-style output with custom formatting
- **Mock Heavy**: Patches `wikipedia.search` and `wikipedia.page` for all external API calls
- **Error Scenarios**: Tests handle no results, disambiguation, and general exceptions

### Code Quality
- **Flake8**: Max line length 127, complexity 10, ignores unused imports (F401)
- **Python 3.13**: Target version with backwards compatibility to 3.11+
- **Pydantic Models**: For type safety and JSON serialization

## Integration Points

### Plugin Registration
The tool is auto-discovered via `pyproject.toml` entry points when zk-chat scans for plugins. The `wikipedia` key maps to the class.

### External Dependencies
- **`wikipedia`** library: Direct API wrapper, no authentication needed
- **`mojentic`**: Plugin framework providing `LLMTool` base class

### Response Format
Tools must return `dict` from `.model_dump()` - this is consumed by the zk-chat framework for LLM function calls.

## Common Tasks

### Adding New Features
1. Extend `WikipediaContentResult` model if response structure changes
2. Modify `run()` method for new Wikipedia API interactions
3. Update `descriptor` property for new function parameters
4. Add corresponding test methods in `DescribeWikipediaContentTool`

### Version Bumps
Update `pyproject.toml` version and document breaking changes in `CHANGELOG.md` following Keep a Changelog format.

### Publishing
GitHub Actions automatically publishes to PyPI on release creation using trusted publishing (no API keys needed).

## Testing Commands
```bash
pytest                    # Run all tests with spec output
python -m build          # Build wheel and source dist
flake8 .                 # Run linting
```