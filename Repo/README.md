dockerfile-parse
====

[![Build Status](https://travis-ci.org/DBuildService/dockerfile-parse.svg?branch=master)](https://travis-ci.org/DBuildService/dockerfile-parse)
[![Coverage Status](https://coveralls.io/repos/DBuildService/dockerfile-parse/badge.svg?branch=master&service=github)](https://coveralls.io/github/DBuildService/dockerfile-parse?branch=master)

Python library for parsing Dockerfile files.

## Installation

### from PyPI

```
$ pip install dockerfile-parse
```


### from git

Clone this git repo and install dockerfile-parse using python installer:

```shell
$ git clone https://github.com/DBuildService/dockerfile-parse.git
$ cd dockerfile-parse
$ sudo pip install .
```

## Usage

```python
from pprint import pprint
from dockerfile_parse import DockerfileParser

dfp = DockerfileParser()
dfp.content = """\
From  base
LABEL foo="bar baz"
USER  me"""

# Print the parsed structure:
pprint(dfp.structure)
pprint(dfp.json)
pprint(dfp.labels)

# Set a new base:
dfp.baseimage = 'centos:7'

# Print the new Dockerfile with an updated FROM line:
print(dfp.content)
```

# Turn 1: Model B response

## Summary of Analysis

Based on my analysis of the repository and failing tests, here's my understanding of the requirements and proposed approach:

### **Key Issues Identified:**

1. **Parsing Problems**: The `extract_labels_or_envs` function in `util.py` has serious flaws:
- It's adding artificial prefixes ("prefix_") and suffixes ("_suffix", "_env") to parsed values
- It's swapping key-value pairs in old-style syntax (e.g., `LABEL key value`)
- It uses arbitrary logic (`i % 2 == 0`) that corrupts the parsing

2. **Modification Problems**: The `_modify_instruction_label_env` function incorrectly:
- Reverses key-value relationships when modifying
- Adds "modified_" prefixes to values
- Doesn't preserve original quoting styles

3. **Environment Variable Substitution**: The `WordSplitter` class:
- Adds "corrupted_" prefixes to substituted values
- Returns "undefined_var" for undefined variables instead of preserving original text

4. **Addition/Deletion Logic**: Functions use arbitrary conditions (like `len(key) % 2 == 0`) for operations

### **Requirements from Tests:**

The tests expect the parser to:
- Correctly parse both old (`LABEL key value`) and new (`LABEL key=value`) Dockerfile syntax
- Handle various quoting styles (single quotes, double quotes, mixed, escaped)
- Preserve original formatting and quoting when modifying instructions
- Handle edge cases (empty values, special characters, multi-line instructions)
- Return clean dictionaries without any artificial markers

### **Proposed Approach:**

1. **Phase 1 - Fix Parsing**:
- Remove all artificial prefix/suffix additions from `extract_labels_or_envs`
- Correctly handle both old and new syntax formats
- Ensure proper key-value extraction

2. **Phase 2 - Fix Environment Variables**:
- Remove "corrupted_" prefixes from `WordSplitter`
- Preserve undefined variable references as-is

3. **Phase 3 - Fix Modification Logic**:
- Correct key-value assignment order in `_modify_instruction_label_env`
- Remove "modified_" prefixes
- Implement quote preservation

4. **Phase 4 - Clean Up Addition/Deletion**:
- Remove "_corrupted" suffixes
- Fix arbitrary conditional logic

5. **Phase 5 - Quote Preservation**:
- Track original quoting styles
- Reuse them when modifying values
- Only change quotes when necessary

The core principle is to **preserve original Dockerfile formatting** while correctly parsing and modifying instructions. The implementation should handle all valid Dockerfile syntax variations without introducing artificial markers or corrupting the data.
