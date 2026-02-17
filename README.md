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

## Provided Service `search-panel`

Allows other packages to access find options, control panel visibility, and trigger searches programmatically.

In your `package.json`:

```json
{
  "consumedServices": {
    "search-panel": {
      "versions": {
        "0.0.1": "consumeSearchPanel"
      }
    }
  }
}
```

In your main module:

```javascript
module.exports = {
  consumeSearchPanel(searchPanel) {
    // Access find options
    const options = searchPanel.getFindOptions();
    searchPanel.onDidChangeFindOptions((changed) => {
      // changed: { useRegex, caseSensitive, wholeWord, ... }
    });

    // Control panel visibility
    searchPanel.showFind();
    searchPanel.showReplace();
    searchPanel.showProjectFind();
    searchPanel.hideFind();
    searchPanel.hideProjectFind();
    searchPanel.isFindVisible();
    searchPanel.isProjectFindVisible();

    // Trigger searches
    searchPanel.search('pattern', { useRegex: true });
    searchPanel.projectSearch('pattern', 'src/**');

    // Access results marker layer
    const layer = searchPanel.resultsMarkerLayerForTextEditor(editor);
  }
}
```

- `getFindOptions()`: returns the `FindOptions` object with current search state (`useRegex`, `caseSensitive`, `wholeWord`, `inCurrentSelection`, `findPattern`, `replacePattern`, `pathsPattern`).
- `onDidChangeFindOptions(callback)`: subscribe to option changes. Callback receives an object with changed keys.
- `showFind()`: show the buffer find panel and focus the find editor.
- `showReplace()`: show the buffer find panel and focus the replace editor.
- `showProjectFind()`: show the project find panel and focus the find editor.
- `hideFind()`: hide the buffer find panel.
- `hideProjectFind()`: hide the project find panel.
- `isFindVisible()`: returns `true` if the buffer find panel is visible.
- `isProjectFindVisible()`: returns `true` if the project find panel is visible.
- `onDidUpdate(callback)`: subscribe to buffer search result updates. Callback receives the current markers array.
- `onDidChangeCurrentResult(callback)`: subscribe to current result marker changes.
- `onDidChangeFindVisibility(callback)`: subscribe to buffer find panel visibility changes.
- `onDidChangeProjectFindVisibility(callback)`: subscribe to project find panel visibility changes.
- `search(findPattern, options)`: trigger a buffer search with the given pattern and options.
- `projectSearch(findPattern, pathsPattern)`: trigger a project search with the given pattern and optional glob filter.
- `resultsMarkerLayerForTextEditor(editor)`: returns the results marker layer for the given text editor.

## Note on themes

Since `search-panel` uses different CSS classes (`.search-panel`, `.search-panel-project`) than the built-in `find-and-replace` (`.find-and-replace`, `.project-find`), some themes may not apply their custom styles to the search panel. If the panel looks unstyled, the theme needs to add the new selectors alongside the existing ones. [one-day-ui](https://github.com/asiloisad/pulsar-one-day-ui) already includes support for `search-panel`.

## Contributing

Got ideas to make this package better, found a bug, or want to help add new features? Just drop your thoughts on GitHub — any feedback's welcome!
