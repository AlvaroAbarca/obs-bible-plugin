# AGENTS.md

This file provides guidelines for agentic coding assistants working on this OBS Bible Plugin repository.

## Build, Lint, and Test Commands

This project uses vanilla JavaScript with no build system or test framework.

```bash
# No build command - files are loaded directly by OBS as browser docks
# No linting configured - code should be manually reviewed for consistency
# No automated tests - test changes manually by opening files in web browser
```

**Manual Testing Approach:**
- Open `control_panel.html` in a web browser to test the control panel
- Open `browser_source.html` in a web browser to test the display source
- Ensure both pages communicate correctly via BroadcastChannel
- Test localStorage persistence by refreshing the browser

## Project Overview

OBS Bible Plugin is a vanilla JavaScript application that integrates with Open Broadcaster Software via browser docks and browser sources. The project has:

- **No TypeScript** - Plain JavaScript (ES5/ES6)
- **No build tools** - Scripts loaded directly via HTML script tags
- **No package manager** - No npm/yarn dependencies
- **No framework** - Vanilla DOM manipulation

## JavaScript Code Style Guidelines

### Variable Declarations

**Prefer `const` and `let` over `var`.**
- Use `const` for variables that won't be reassigned
- Use `let` for variables that need reassignment
- Avoid `var` unless working with legacy code

```javascript
// Good
const channel = new BroadcastChannel("myChannel");
let currentVerseIndex = -1;

// Avoid
var savedMessage = localStorage.getItem('savedMessage');
```

### Naming Conventions

- **Variables and functions:** camelCase (e.g., `searchBible`, `savedBibleVerse`)
- **Classes:** PascalCase (rarely used in this codebase)
- **Constants:** UPPER_SNAKE_CASE or camelCase for localStorage keys
- **DOM element IDs:** kebab-case or camelCase as found in HTML

### Functions

- Use arrow functions for callbacks and short anonymous functions
- Use function declarations for named functions at module level
- Be consistent within a file

```javascript
// Arrow function for callback
element.addEventListener("click", (event) => {
    sendMessage(channel, message);
});

// Function declaration for module-level function
function searchBible(query, bible_data) {
    // implementation
}
```

### Semicolons

Include semicolons at the end of statements for consistency, though the codebase has mixed usage. Follow existing patterns in the file you're editing.

### Comments

- Use `//` for single-line comments
- No JSDoc style comments currently used in the codebase
- Keep comments concise and explain "why" not "what"

## State Management

### localStorage

All localStorage keys should use the `obs-bible-` prefix for namespacing:

```javascript
localStorage.setItem('obs-bible-selectedSettingTab', tabName);
localStorage.setItem('obs-bible-saved-main-border', borderValue);
localStorage.setItem('saved-bible-version', version); // Exception: common settings
```

### Inter-Window Communication

Use BroadcastChannel for communication between control panel and browser source:

```javascript
const channel = new BroadcastChannel("myChannel");
channel.postMessage({ messageContent: text });
channel.close(); // Always close after use
```

For settings updates:
```javascript
const settingsChannel = new BroadcastChannel("settings");
settingsChannel.postMessage({ selectedFont: selectedValue });
settingsChannel.close();
```

### Global State

Some files use global variables for shared state (e.g., `bible_data`, `bblVerseDiv`). When adding new shared state, consider if it should use localStorage or be module-scoped.

## DOM Manipulation Guidelines

- Use `getElementById()` for elements with unique IDs (most common pattern)
- Use `querySelector()` and `querySelectorAll()` when needed
- Cache DOM references when used multiple times:

```javascript
const inputField = document.getElementById("bible-input");
const submitButton = document.getElementById("bible-submit");
```

- Use `classList` for adding/removing classes:

```javascript
button.classList.add("selected");
button.classList.remove("selected");
element.classList.toggle("active");
```

## Event Handling

- Use `addEventListener` for all event binding
- Event handlers can be named functions or arrow functions
- Clean up listeners when elements are replaced:

```javascript
// Clear existing listeners before adding new ones
document.getElementById("prev-line")?.replaceWith(
    document.getElementById("prev-line").cloneNode(true)
);
```

## Error Handling

- Use try-catch blocks for operations that may fail
- Log errors with `console.error()` for debugging:

```javascript
try {
    const { book, chapter, verse } = extractBookChapterVerse(verse.name);
} catch (error) {
    console.error(error.message);
}
```

- Validate localStorage values before use (check for null)

## CSS Guidelines

- Use kebab-case for class names
- Use CSS custom properties (variables) with `--` prefix
- Flexbox and grid for layout
- Color values: hex for user inputs, rgba/rgb for computed values

## File Organization

```
assets/
├── js/
│   ├── browser_source/    # Scripts loaded by browser_source.html
│   └── control_panel/     # Scripts loaded by control_panel.html
├── css/
│   ├── browser/           # Styles for browser source
│   └── control_panel/     # Styles for control panel
└── bibles/               # Bible translation data files
```

## HTML Structure

- Use semantic HTML elements
- IDs for elements accessed by JavaScript
- Classes for styling
- Load CSS in `<head>`, JavaScript at end of `<body>` or in appropriate sections

## Adding New Bible Translations

Bible translation files should be placed in `assets/bibles/[translation]/` directory. Each translation file exports a variable with the translation name (e.g., `kjv`, `nkjv`). Add the translation to the `bibleVersions` object in `select_bible_translation.js`.

## Working with Songs

Songs are loaded from `.txt` files. The format expects:
- Verses numbered (e.g., "1.", "2.")
- "CHORUS" line marks chorus sections
- Lines can be clicked to display

Two display modes:
- Line by line (checkbox: `obs-bible-display-song-line-by-line`)
- Verse by verse (uncheck the above)

## Key Files Reference

- `control_panel.html` - Main control panel interface
- `browser_source.html` - Display source for OBS
- `assets/js/control_panel/search_bible.js` - Bible search functionality
- `assets/js/control_panel/send_message.js` - Message broadcasting
- `assets/js/control_panel/settings.js` - Settings management
- `assets/js/control_panel/utils.js` - Utility functions including fuzzy search

## Code Modifications Tips

1. When editing existing files, match the existing code style within that file
2. Test changes in both browser source and control panel
3. Check localStorage persistence by refreshing the browser
4. Verify BroadcastChannel communication between windows
5. Consider both line-by-line and verse-by-verse modes for song features
