# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [2.0.0] - 2025-09-26

### Changed
- **Breaking**: Migrated to new zk-chat service provider architecture
- Constructor now accepts `service_provider` parameter instead of `vault` and `llm` parameters
- Renamed package from `zk-rag-wikipedia` to `zk-chat-wikipedia`
- Updated mojentic dependency to `>=0.6.1` for compatibility with new plugin architecture
- Plugin now inherits directly from `LLMTool` using service provider pattern
- Removed unused vault parameter from constructor as Wikipedia lookup doesn't require file system access

### Migration Notes
- Existing installations should uninstall the old `zk-rag-wikipedia` package and install `zk-chat-wikipedia`
- Plugin functionality remains the same but uses the new extensible service provider interface
- Wikipedia lookup capabilities are unchanged and maintain full backward compatibility

## [1.1.0] - 2024-01-24

### Changed
- **Breaking**: Enhanced LLM integration by requiring explicit LLMBroker instance in tool initialization
- Improved architecture to better handle language model interactions

## [1.0.0] - 2024-01-24

### Added
- Initial release of the zk-rag-wikipedia plugin
- Wikipedia content lookup functionality
- Support for article search and retrieval
- Handling of disambiguation pages
- Comprehensive error handling
- Full test coverage
- Documentation in README.md

### Dependencies
- Requires Python >= 3.11
- Depends on mojentic >= 0.2.5
- Depends on wikipedia >= 1.4.0
