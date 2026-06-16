# search-panel

Find and replace in buffers and across the project.

Fork of [find-and-replace](https://github.com/pulsar-edit/pulsar/tree/master/packages/find-and-replace).

## Features

- **Replace in-place**: Replace matches without full buffer refresh.
- **Regex options**: Ripgrep and PCRE2 regex buttons.
- **Center scroll**: Navigate from result view centers the match.
- **Progress display**: Distinct indicators for ripgrep and scandal.
- **Scrollmap**: Shows search results in the scrollbar via [scrollmap](https://github.com/asiloisad/pulsar-scrollmap).
- **Search adapters**: The buffer find panel can search custom pane items such as data grids and notebooks when they expose a search adapter.

## Installation

To install `search-panel` search for [search-panel](https://web.pulsar-edit.dev/packages/search-panel) in the Install pane of the Pulsar settings or run `ppm install search-panel`. Alternatively, you can run `ppm install asiloisad/pulsar-search-panel` to install a package directly from the GitHub repository.

**Note**: This package automatically disables the built-in `find-and-replace` package to avoid conflicts.

## Commands

- `search-panel:project-show`: show project search,
- `search-panel:show`: show buffer search,
- `search-panel:show-replace`: show replace controls,
- `search-panel:find-next`: find next match,
- `search-panel:find-previous`: find previous match,
- `search-panel:find-next-selected`: find next selected text,
- `search-panel:find-previous-selected`: find previous selected text,
- `search-panel:select-all`: select all matches,
- `search-panel:select-next`: select next match,
- `search-panel:select-skip`: skip current match,
- `search-panel:confirm`: confirm search,
- `search-panel:replace-all`: replace all from the replace editor,
- `search-panel:clear`: clear results.

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
- Buffer search can target adapter-backed pane items, not only text editors.

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
    searchPanel.search("pattern", { useRegex: true });
    searchPanel.projectSearch("pattern", "src/**");

    // Access results marker layer
    const layer = searchPanel.resultsMarkerLayerForTextEditor(editor);
  },
};
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

## Consumed Service `search-adapter`

Allows other packages to make non-TextEditor pane items searchable through the normal buffer find panel. When the active center pane item is handled by a `search-adapter` provider, search-panel routes find, navigation, replacement, result counts, and wrap indicators to that adapter instead of a text editor.

Provider packages register the service in `package.json`:

```json
{
  "providedServices": {
    "search-adapter": {
      "versions": {
        "1.0.0": "provideSearchAdapter"
      }
    }
  }
}
```

In the provider package's main module:

```javascript
module.exports = {
  provideSearchAdapter() {
    return {
      handlesItem(item) {
        return item?.constructor?.name === "MyPaneItem";
      },

      getAdapterForItem(item) {
        return this.handlesItem(item) ? item.searchAdapter : null;
      },
    };
  },
};
```

The service may also provide `getActiveAdapter()` when the active searchable object is not represented directly by the active center pane item.

The returned adapter owns matching, navigation, highlighting, selection, lifecycle cleanup, and replacing inside its own view. It is handed the shared `FindOptions` on each search, so it can honor the regex / case / whole-word / selection options and build the search regex via `findOptions.getFindPatternRegex()` without reimplementing option handling.

Adapter interface (methods marked optional may be omitted):

- `search(findOptions)`: scan the view, build an ordered match list, then emit the update event. Use `findOptions.findPattern` and `findOptions.getFindPatternRegex()`.
- `onDidUpdate(callback)`: emitted when the match list changes.
- `onDidChangeCurrentResult(callback)`: emitted when the current match changes.
- `onDidError(callback)` (optional): emitted on a search error.
- `getResultCount()`: total number of matches.
- `getCurrentResultIndex()`: index of the current match, or `-1`.
- `selectNext()` / `selectPrevious()`: move to and reveal the next / previous match. Return `{ found, wrapped }`, where `wrapped` is `"up"`, `"down"`, or `null`.
- `selectFirstFromCursor()` (optional): like `selectNext` but may include a match at the current position. Falls back to `selectNext`.
- `selectAll()` (optional): reveal / select all matches.
- `canReplace`: `true` if the view supports replacing.
- `replaceCurrentMatch(replaceText, direction)` (optional): replace the current match; `direction` is `"next"` or `"previous"`.
- `replaceAll(replaceText)` (optional): replace every match.
- `hasSelectionMatchingResult()` (optional): whether the view's selection sits on a match.
- `isSelectionEmpty()` (optional): whether the view has no active selection.
- `getSelectedText()` / `getWordUnderCursor()` (optional): used by "use selection as find pattern".
- `getWrapIconHost()` (optional): element to host the wrap-around indicator.
- `deactivate()` (optional): clear highlights, selections, timers, or other transient state when the pane item stops being the active search target.

Adapter notes:

- Keep the adapter instance stable for the lifetime of the pane item. Recreating it on every `getAdapterForItem()` call will lose result state and subscriptions.
- Emit `onDidUpdate` after searches and data changes that affect the match list. The find panel reads `getResultCount()` after this event.
- Emit `onDidChangeCurrentResult` whenever navigation changes the current match. The find panel reads `getCurrentResultIndex()` after this event.
- For read-only views, set `canReplace = false`; replace commands will be disabled and will beep if invoked directly.
- `deactivate()` should remove visible search UI so matches do not linger after switching back to a text editor or another pane item.

## Provided Service `find-and-replace`

Backward-compatible service for packages that still consume the built-in `find-and-replace` service. It exposes `resultsMarkerLayerForTextEditor(editor)`.

## Note on themes

Since `search-panel` uses different CSS classes (`.search-panel`, `.search-panel-project`) than the built-in `find-and-replace` (`.find-and-replace`, `.project-find`), some themes may not apply their custom styles to the search panel. If the panel looks unstyled, the theme needs to add the new selectors alongside the existing ones. [one-day-ui](https://github.com/asiloisad/pulsar-one-day-ui) already includes support for `search-panel`.

## Contributing

Got ideas to make this package better, found a bug, or want to help add new features? Just drop your thoughts on GitHub. Any feedback is welcome!
