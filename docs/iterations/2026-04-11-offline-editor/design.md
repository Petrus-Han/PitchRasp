# Offline HTML Editor — Design Document

## Goal

Build a fully offline, single-file HTML editor that lets users visually edit HTML documents through mouse and keyboard interactions — no code editing, no server required. Users load an HTML file, select elements by clicking, and edit properties (text, color, position) through a visual panel.

## Approach

### Architecture

Single `index.html` file containing all CSS and JS inline. Opens directly in any modern browser.

### Layout (3-panel)

```
+------------------+------------------------------+-------------------+
|   Element Tree   |       Canvas (iframe)        |  Property Panel   |
|   (left sidebar) |       (center)               |  (right sidebar)  |
+------------------+------------------------------+-------------------+
|                        Toolbar (top)                                |
+--------------------------------------------------------------------|
```

- **Toolbar (top)**: File open/save, undo/redo, language toggle (EN/中文), theme toggle (light/dark)
- **Element Tree (left)**: Collapsible DOM tree for element lookup and selection
- **Canvas (center)**: `<iframe>` rendering the loaded HTML; click to select elements
- **Property Panel (right)**: Edit selected element's text content, colors, and actions (delete/copy)

### Core Features

#### 1. File I/O
- **Open**: `<input type="file" accept=".html,.htm">` reads file via FileReader API
- **Save/Export**: Serialize iframe's `document.documentElement.outerHTML`, download as `.html`

#### 2. Element Selection
- Click an element in the iframe → highlight with dashed border overlay
- Corresponding node highlights in the element tree
- Click in element tree → scrolls to and highlights element in canvas
- Selection overlay is a positioned `<div>` on top of iframe, not modifying the source DOM

#### 3. Element Operations (no "Add")
- **Delete**: Remove selected element (`element.remove()`)
- **Copy**: `cloneNode(true)` → `insertAdjacentElement('afterend', clone)` (same level, right after original)
- **Find**: Element tree panel with search/filter input
- **Modify**: Via property panel (text, color)

#### 4. Text Editing
- When a text-containing element is selected, property panel shows an editable text field
- Changes sync to `element.textContent` in real-time

#### 5. Color Editing
- For text elements: modify `style.color`
- For SVG elements: modify `fill` and `stroke` attributes
- Three preset color buttons:
  - Black: `#000000`
  - White: `#FFFFFF`
  - Blue: `#0033FF`
- Also allow custom hex input

#### 6. Undo/Redo
- Command pattern: each action (delete, copy, text change, color change) pushes a command onto a stack
- Each command stores: `{ type, target (CSS selector path), before, after }`
- Undo pops from undo stack → pushes to redo stack (and vice versa)
- Keyboard: `Ctrl+Z` / `Ctrl+Shift+Z` (or `Cmd` on Mac)

#### 7. i18n (English default + Chinese)
- All UI strings in a `translations` object keyed by language code
- Toggle button in toolbar switches `en` ↔ `zh`
- Persist language preference in `localStorage`

#### 8. Light/Dark Theme
- CSS custom properties for theme colors
- Toggle button in toolbar switches themes
- Persist in `localStorage`
- Dark mode: dark backgrounds, light text for editor UI (does NOT affect the loaded HTML content)

### Keyboard Shortcuts
| Action | Shortcut |
|--------|----------|
| Undo | Ctrl+Z |
| Redo | Ctrl+Shift+Z |
| Delete element | Delete / Backspace |
| Copy element | Ctrl+D |
| Save file | Ctrl+S |

### Technical Details

- **iframe isolation**: Editor styles/scripts do not leak into user content
- **Selection overlay**: Absolute-positioned highlight div on top of iframe, recalculated on scroll/resize
- **Unique element identification**: Generate a temporary `data-editor-id` attribute on each element for reliable targeting in undo/redo (stripped on export)
- **No external dependencies**: Pure vanilla HTML/CSS/JS

## Scope

### In scope
- Single-file offline HTML editor
- Open/save HTML files
- Element tree navigation
- Click-to-select elements in canvas
- Delete, copy (same-level), find elements
- Edit text content
- Edit text color and SVG fill/stroke
- 3 preset colors (black, white, blue #0033FF)
- Undo/redo with keyboard shortcuts
- English/Chinese UI toggle
- Light/dark theme toggle
- All mouse/keyboard driven, no code editing

### NOT in scope
- Adding new elements
- Drag-to-reorder elements
- CSS property editor beyond color
- JavaScript execution in loaded HTML
- Multi-file projects
- Collaborative editing
- Image upload or media management

## Test Plan

Since this is a single HTML file with no build step, testing is manual browser-based:

1. **File loading**: Open a sample HTML file with text, images, SVG elements
2. **Element selection**: Click elements in canvas and tree, verify highlighting syncs
3. **Text editing**: Select a `<p>`, change text in panel, verify canvas updates
4. **Color editing**: Change text color with preset buttons and custom input
5. **SVG color**: Select an SVG path, change fill color
6. **Delete**: Select and delete an element, verify it disappears from canvas and tree
7. **Copy**: Copy an element, verify duplicate appears after original at same level
8. **Undo/Redo**: Perform actions, undo each, redo each, verify state consistency
9. **Theme toggle**: Switch light/dark, verify editor UI changes but canvas content unchanged
10. **Language toggle**: Switch EN/ZH, verify all UI labels update
11. **Export**: Save edited HTML, open in new tab, verify changes persisted and no editor artifacts (`data-editor-id` stripped)
12. **Keyboard shortcuts**: Test Ctrl+Z, Ctrl+Shift+Z, Delete, Ctrl+D, Ctrl+S
