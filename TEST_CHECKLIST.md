# Edita - Manual Test Checklist

Run through these tests before pushing changes to ensure core functionality works.

## 🔍 Find & Search
- [ ] Open Find dialog (Ctrl+F)
- [ ] Type search term and press Enter → should find first occurrence
- [ ] Press Enter again → should find NEXT occurrence (not delete text!)
- [ ] Click "Find Next" button → should find next occurrence
- [ ] Click "Find Previous" button → should find previous occurrence
- [ ] Search wraps around at end/beginning of file
- [ ] "Count" button shows correct number of occurrences
- [ ] "Search All Files" shows results in bottom panel
- [ ] "Count in All Files" shows correct summary

## ⌨️ Keyboard Input
- [ ] Typing in editor works normally
- [ ] Press Enter in editor → creates new line (doesn't trigger find!)
- [ ] Press Tab in editor → inserts 4 spaces
- [ ] Press Enter in Find input → triggers Find Next
- [ ] Ctrl+F → opens find dialog
- [ ] Ctrl+S → saves file
- [ ] Ctrl+N → creates new file

## 🔄 Replace
- [ ] "Replace" button replaces current selection
- [ ] "Replace All in File" replaces all occurrences in current file only
- [ ] Case sensitive option works
- [ ] Whole word option works

## 📋 Dialog Behavior
- [ ] Close dialog (X button or Close button) → clears search/replace text boxes
- [ ] Dialog can be dragged by header
- [ ] Opening find dialog doesn't block editor input

## 📁 File Operations
- [ ] Click "Open" button → file picker opens
- [ ] Can open files successfully
- [ ] Can create new files
- [ ] Can save files
- [ ] Can close tabs
- [ ] Can switch between tabs

## 🎨 UI
- [ ] Find dialog layout is compact (no excessive spacing)
- [ ] Second row of buttons is centered
- [ ] No line numbers shown in search results

## ⚠️ Common Regression Points
- **Enter key behavior** - Most fragile area, check both in editor and find input
- **Event handler conflicts** - Make sure keyboard events don't interfere with each other
- **Dialog state** - Verify dialogs clear properly when closed

---

**Before Pushing:** Test at minimum the "⚠️ Common Regression Points" section
