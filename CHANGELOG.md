# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

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
