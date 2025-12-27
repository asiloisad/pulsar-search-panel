# search-panel

Find and replace in buffers and across the project. Fork of [find-and-replace](https://github.com/pulsar-edit/pulsar/tree/master/packages/find-and-replace).

## Changes from original find-and-replace

### Features

- Replace in-place functionality
- Ripgrep and PCRE2 regex option buttons
- Center scroll when navigating from result view
- Distinct progress display for ripgrep and scandal searchers

### Fixes

- Project search and ripgrep on Windows
- Line ending normalization in buffer search
- Race condition when starting new search while previous is in progress
- Ripgrep cancellation handling
- Windows process termination reliability

### Code

- Converted CoffeeScript to JavaScript
- Reworked result view component
- Reduced dependencies

## Installation

To install run `ppm install asiloisad/pulsar-search-panel` to install a package directly from the GitHub repository.

# Contributing

Got ideas to make this package better, found a bug, or want to help add new features? Just drop your thoughts on GitHub — any feedback’s welcome!
