# search-panel

Find and replace in buffers and across the project.

Fork of [find-and-replace](https://github.com/pulsar-edit/pulsar/tree/master/packages/find-and-replace).

## Features

- **Replace in-place**: Replace matches without full buffer refresh.
- **Regex options**: Ripgrep and PCRE2 regex buttons.
- **Center scroll**: Navigate from result view centers the match.
- **Progress display**: Distinct indicators for ripgrep and scandal.
- **Scrollmap**: Shows search results in the scrollbar via [scrollmap](https://github.com/asiloisad/pulsar-scrollmap).

## Installation

To install `search-panel` run `ppm install asiloisad/pulsar-search-panel` to install a package directly from the GitHub repository.

## Changes from find-and-replace

- Replace in-place functionality.
- Ripgrep and PCRE2 regex option buttons.
- Center scroll when navigating from result view.
- Distinct progress display for ripgrep and scandal searchers.
- Project search and ripgrep on Windows.
- Line ending normalization in buffer search.
- Race condition when starting new search while previous is in progress.
- Ripgrep cancellation handling.
- Windows process termination reliability.
- Converted CoffeeScript to JavaScript.
- Reworked result view component.
- Reduced dependencies.
- Option to clean editors on panel hide.
- Explicit glob patterns in path filter (no magic auto-expansion).
- "Search in Folder" escapes special characters and appends `/**`.
- "Use selection as replace pattern" no longer escapes regex characters.

## Note on themes

Since `search-panel` uses different CSS classes (`.search-panel`, `.search-panel-project`) than the built-in `find-and-replace` (`.find-and-replace`, `.project-find`), some themes may not apply their custom styles to the search panel. If the panel looks unstyled, the theme needs to add the new selectors alongside the existing ones.

## Contributing

Got ideas to make this package better, found a bug, or want to help add new features? Just drop your thoughts on GitHub — any feedback's welcome!
