# Edita - Feature Reference & Implementation Guide

This document describes all features and their implementations. **Reference this before making changes** to avoid breaking existing functionality.

---

## 🔍 Find & Replace System

### Find Dialog Behavior

**Opening:**
- Keyboard: `Ctrl+F`
- Button: Click Find button in toolbar
- Menu: Search → Find

**Key Implementation Details:**

1. **Enter Key Handling (CRITICAL - Often Breaks!)**
   - When find dialog is open AND editor has selected text:
     - Enter key is intercepted by global keyboard handler
     - `e.preventDefault()` prevents text deletion
     - Calls `findNext()` instead
   - When typing in find input:
     - Enter triggers `findNext()` via dedicated event listener
   - When editor has focus but NO selection:
     - Enter works normally (creates new line)

2. **Text Selection & Highlighting**
   - After finding text: `this.editor.setSelectionRange(index, index + length)`
   - Editor receives focus to make selection visible
   - Selection stays visible because editor keeps focus
   - CSS styling: `::selection` for blue highlight

3. **Focus Management**
   - After finding: Editor receives focus (to show selection)
   - User presses Enter: Intercepted to find next (doesn't delete)
   - This allows text to be visible AND prevents accidental deletion

### Find Functions

**`findNext()`**
- Searches from current cursor position + 1
- Wraps to beginning if not found
- Sets selection range on editor
- Keeps editor focused
- Shows "Wrapped to beginning" message when wrapping

**`findPrevious()`**
- Searches backward from current cursor position - 1
- Wraps to end if not found
- Sets selection range on editor
- Keeps editor focused
- Shows "Wrapped to end" message when wrapping

**`searchAllFilesNow()`**
- Searches across all open tabs
- Shows results in bottom panel
- Results are collapsible by file
- Clicking result switches to that tab and line
- Does NOT show line numbers in results (removed per user request)

**`countOccurrences()`**
- Counts in current file only
- Respects case sensitive and whole word options
- Shows count in find results area

**`countInAllFiles()`**
- Counts across all open tabs
- Shows summary: "Found X matches in Y files"
- Displays toast notification

### Replace Functions

**`replaceOne()`**
- Replaces current selection only
- If no selection, finds next match first

**`replaceAll()`**
- Replaces ALL occurrences in current file only
- Uses regex with global flag for efficiency
- Respects case sensitive and whole word options
- Button labeled "Replace All in File" to be clear

### Dialog Closing Behavior

**When closing dialog:**
- Clear find input: `document.getElementById('findInput').value = ''`
- Clear replace input: `document.getElementById('replaceInput').value = ''`
- Remove 'active' class from dialog
- This prevents old search terms from appearing on next open

---

## ⌨️ Keyboard Event Handlers

### Global Document Keyboard Handler

**Location:** `document.addEventListener('keydown', ...)`

**Order of checks (IMPORTANT):**

1. **Enter key when find dialog is open with selection:**
   ```javascript
   if (e.key === 'Enter' && findDialog.classList.contains('active')) {
       if (document.activeElement === this.editor && 
           this.editor.selectionStart !== this.editor.selectionEnd) {
           e.preventDefault();
           this.findNext();
       }
   }
   ```

2. **F3 key (Find Next):**
   - Works when find dialog is open
   - Alternative to Enter key

3. **Escape key:**
   - Closes find dialog if open

4. **Ctrl/Cmd shortcuts:**
   - `Ctrl+S`: Save
   - `Ctrl+O`: Open
   - `Ctrl+N`: New File
   - `Ctrl+F`: Find
   - `Ctrl+H`: Replace
   - `Ctrl+T`: Toggle Theme

### Editor-Specific Handler

**Location:** `this.editor.addEventListener('keydown', ...)`

**Handles:**
- Tab key: Inserts 4 spaces instead of tabbing away

### Find Input Handler

**Location:** `document.getElementById('findInput').addEventListener('keydown', ...)`

**Handles:**
- Enter key: Calls `findNext()` with `e.preventDefault()`

---

## 📋 Dialog Layout

**Structure:**
```
Find Dialog
├── Header (draggable)
├── Find input
├── Replace input
├── Options (checkboxes)
│   ├── Case sensitive
│   └── Whole word
├── Button Row 1 (right-aligned)
│   ├── Find Next
│   ├── Find Previous
│   ├── Count
│   ├── Replace
│   └── Replace All in File
└── Button Row 2 (centered)
    ├── Search All Files
    ├── Count in All Files
    └── Close
```

**Spacing:**
- Gap between button rows: 4px (reduced for compactness)
- Bottom padding: 8px (reduced from 20px)
- Find results margin-top: 4px

**Second row centering:**
- Uses `.dialog-buttons + .dialog-buttons { justify-content: center; }`

---

## 📁 File Operations

### Opening Files

**Methods:**
1. File picker API: `openFileWithPicker()` - modern approach
2. File input: `openFile(e)` - fallback

**File Handles (IndexedDB):**
- Store `fileHandle` for direct save capability
- Saved to IndexedDB per tab
- Restored on page reload
- Allows saving without re-picking file

### Tab Management

**Tab state includes:**
- `id`: Unique identifier
- `name`: File name
- `content`: File contents
- `language`: Syntax highlighting mode
- `modified`: Unsaved changes flag
- `fileHandle`: File system handle (if available)

**Saving tabs to localStorage:**
- Content limited to 50MB per file
- Total state limited to 50MB
- Maximum 20 tabs restored
- File handles saved separately to IndexedDB

---

## 🎨 UI Customization

### Theme Toggle

**States:** Dark (default) / Light
**Storage:** `localStorage.getItem('edita_theme')`
**Implementation:** Toggle `.light-theme` class on body

### Vertical Tabs

**Default:** Vertical (true)
**Storage:** `localStorage.getItem('edita_vertical_tabs')`
**Classes:** 
- `#tabsContainer.vertical`
- `#editorContainer.vertical-tabs`

---

## ⚠️ Common Breaking Points (CHECK THESE!)

### 1. Enter Key Behavior
**Most fragile area - breaks often when adding features**

**What to verify:**
- Enter in find input → Find Next (works)
- Enter in editor with find dialog open + text selected → Find Next (doesn't delete)
- Enter in editor with find dialog closed → New line (works)
- Enter in editor with find dialog open + no selection → New line (works)

**How it breaks:**
- Adding new keyboard handlers without proper checks
- Changing focus management
- Modifying event handler order

**How to fix:**
- Global handler must intercept Enter FIRST when conditions are met
- Check: find dialog active + editor has focus + text is selected
- Always use `e.preventDefault()` before calling `findNext()`

### 2. Focus Management
**Second most fragile**

**Current implementation:**
- After finding text: Editor gets focus (makes selection visible)
- Enter key is intercepted before it can delete text
- Works because global handler catches it first

**What NOT to do:**
- Don't move focus to find input after finding (selection won't be visible)
- Don't move focus to buttons (creates confusing UX)
- Don't remove `editor.focus()` from find functions

### 3. Dialog State
**Third most fragile**

**What to verify:**
- Dialog clears search/replace inputs when closed
- Dialog uses `.active` class (not `style.display`)
- Previous search terms don't appear when reopening

### 4. Event Handler Conflicts

**Multiple handlers exist for Enter:**
1. Global document handler (intercepts for find)
2. Find input handler (triggers find next)
3. Editor handler (tab key only)

**Order matters!** Global handler must run first to intercept.

---

## 🔧 Making Changes Safely

### Before Editing Code:

1. **Read this file** - Understand what might break
2. **Check affected features** - Review related functions
3. **Read full function context** - Don't edit in isolation

### After Editing Code:

1. **Run syntax check:** Use `get_errors` tool
2. **Read modified functions completely** - Verify logic intact
3. **Test critical paths** - Use TEST_CHECKLIST.md
4. **Verify no regression** - Test Enter key, find, replace

### When Adding Keyboard Shortcuts:

1. Check global keyboard handler first
2. Ensure proper event ordering
3. Test interaction with find dialog
4. Verify Enter key still works correctly

### When Modifying Find/Replace:

1. Test with find dialog open
2. Test Enter key in all scenarios
3. Verify text highlighting works
4. Check focus management

---

## 📊 Cache & Versioning

**Current Version:** `v=3`

**Files with cache busting:**
- `styles.css?v=3`
- `script.js?v=3`

**When to increment:**
- Any change to CSS or JS files
- Before pushing to GitHub
- To force browser reload

**Format:** `?v=X` where X is integer

---

## 🔄 Service Worker

**Status:** Unregistered
**Reason:** Prevent caching issues during development
**Location:** Unregistration code in `init()`

---

## 📝 Notes for Future Development

1. **Enter key handling is the #1 source of bugs** - Always test thoroughly
2. **Focus management affects selection visibility** - Editor must keep focus
3. **Event handler order matters** - Global handler must intercept first
4. **Always increment cache version** when pushing changes
5. **Test with TEST_CHECKLIST.md** before every push
6. **Reference this file** before making changes to keyboard handlers or find/replace

---

**Last Updated:** January 5, 2026
**Version:** 1.0
