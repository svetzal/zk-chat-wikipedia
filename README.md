# zk-rag-wikipedia Plugin

A plugin for zk-rag that enables Wikipedia content lookup functionality.

## Description

This plugin provides a tool to search and retrieve content from Wikipedia articles. It can be used to enhance your RAG (Retrieval-Augmented Generation) applications with Wikipedia knowledge.

## Features

- Search Wikipedia articles
- Retrieve article summaries
- Handle disambiguation pages
- Comprehensive error handling

## Installation

```bash
pip install zk-rag-wikipedia
```

## Usage

The plugin will be automatically discovered by zk-rag when installed. It provides the `LookUpTopicOnWikipedia` tool which can be used to retrieve information about specific topics from Wikipedia.

Example usage through zk-rag:

```python
from zk_rag import get_tool

wikipedia_tool = get_tool("wikipedia")
result = wikipedia_tool.run("Python programming language")
print(result.title)  # The article title
print(result.content)  # The article summary
print(result.url)  # The Wikipedia URL
```

## Requirements

- Python >= 3.11
- mojentic
- wikipedia

## License

MIT License

## Author

Stacey Vetzal (stacey@vetzal.com)