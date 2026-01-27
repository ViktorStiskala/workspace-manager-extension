---
name: Add Vitest Unit Tests
overview: Add Vitest for unit testing with parametrized tests covering the core logic in PatternMatcher and SettingsMerger, plus extracting and testing helper functions from services.
todos:
  - id: setup-vitest
    content: Install vitest, create vitest.config.ts, add test scripts to package.json
    status: pending
  - id: export-helpers
    content: Export detectIndentationStyle, removeWorkspaceManagerKeys, offsetToPosition for testing
    status: pending
  - id: test-pattern-matcher
    content: Create tests/patternMatcher.test.ts with parametrized tests for isExcluded and filterSettings
    status: pending
  - id: test-settings-merger
    content: Create tests/settingsMerger.test.ts with parametrized tests for merge and diff methods
    status: pending
  - id: test-helpers
    content: Create tests/helpers.test.ts for extracted helper functions
    status: pending
isProject: false
---

# Add Vitest Unit Tests for Core Logic

## Setup

Install Vitest and configure it to work with the existing TypeScript/esbuild setup:

```bash
npm install -D vitest
```

Add test script to [package.json](package.json):

```json
"scripts": {
  "test": "vitest run",
  "test:watch": "vitest"
}
```

Create `vitest.config.ts` at project root with TypeScript support.

---

## Test Files to Create

### 1. PatternMatcher Tests (`tests/patternMatcher.test.ts`)

Test [src/utils/patternMatcher.ts](src/utils/patternMatcher.ts) with parametrized tests covering:

`**isExcluded()` method:**

- Basic exclusion: `"editor.*"` excludes `editor.fontSize`, `editor.tabSize`
- Non-matching: `"editor.*"` does NOT exclude `terminal.fontSize`
- Negation override: `["editor.*", "!editor.fontSize"]` - `editor.fontSize` NOT excluded
- Multiple patterns: `["editor.*", "terminal.*"]` excludes both
- Exact match: `"editor.fontSize"` only excludes exact key
- Wildcard edge cases: `"*"` excludes everything, `"**"` behavior
- Empty patterns: no exclusions

`**filterSettings()` method:**

- Filters object correctly based on patterns
- Returns partial object with non-excluded keys
- Handles empty settings object
- Preserves value types (nested objects, arrays, primitives)

### 2. SettingsMerger Tests (`tests/settingsMerger.test.ts`)

Test [src/services/settingsMerger.ts](src/services/settingsMerger.ts) with parametrized tests covering:

`**merge()` method - critical behavior:**

- Deep merge: `{a: {b: 1}}` + `{a: {c: 2}}` = `{a: {b: 1, c: 2}}`
- Override values: `{a: 1}` + `{a: 2}` = `{a: 2}`
- **null removes keys**: `{a: 1, b: 2}` + `{a: null}` = `{b: 2}` (CRITICAL)
- Arrays are replaced, not merged: `{a: [1,2]}` + `{a: [3]}` = `{a: [3]}`
- Nested null removal: `{a: {b: 1}}` + `{a: null}` = `{}`
- Override object with primitive: `{a: {b: 1}}` + `{a: 1}` = `{a: 1}`
- Override primitive with object: `{a: 1}` + `{a: {b: 1}}` = `{a: {b: 1}}`
- Empty override: `{a: 1}` + `{}` = `{a: 1}`
- Deep clone (no mutation): verify original objects unchanged

`**diff()` method - critical behavior:**

- Changed value: expected `{a: 1}`, current `{a: 2}` → `{a: 2}`
- Added key: expected `{}`, current `{a: 1}` → `{a: 1}`
- **Removed key returns null**: expected `{a: 1}`, current `{}` → `{a: null}` (CRITICAL)
- Nested changes detected
- No changes: expected `{a: 1}`, current `{a: 1}` → `{}`
- Array differences (replacement semantics)
- Complex nested diff scenarios

### 3. Helper Functions Tests (`tests/helpers.test.ts`)

Extract and test pure helper functions:

**From [src/services/codeActions.ts](src/services/codeActions.ts) - `detectIndentationStyle()`:**

- Tab indentation: `"\t{..."` → `{tabSize: 1, insertSpaces: false}`
- Space indentation (2 spaces): `"  {..."` → `{tabSize: 2, insertSpaces: true}`
- Space indentation (4 spaces): `"    {..."` → `{tabSize: 4, insertSpaces: true}`
- No indentation (default to tabs)
- Mixed content (first indented line wins)

**From [src/services/forwardSync.ts](src/services/forwardSync.ts) - `removeWorkspaceManagerKeys()`:**

- Removes all `workspaceManager.*` keys
- Preserves other keys
- Handles nested objects (only top-level filtering)

**From [src/services/diagnostics.ts](src/services/diagnostics.ts) - `offsetToPosition()`:**

- Single line: offset 5 → line 0, col 5
- Multiple lines: offset after newlines → correct line/col
- Edge cases: offset 0, offset at newline

---

## Refactoring Required

To make helper functions testable, export them or extract to utils:

1. `**detectIndentationStyle()**` - Move to `src/utils/indentation.ts` or export from codeActions
2. `**removeWorkspaceManagerKeys()**` - Move to `src/utils/settingsFilter.ts` or export from forwardSync
3. `**offsetToPosition()**` - Move to `src/utils/textPosition.ts` or export from diagnostics

Alternatively, test through the public API where the logic is exercised.

---

## Test Structure

```
tests/
├── patternMatcher.test.ts    # PatternMatcher class
├── settingsMerger.test.ts    # SettingsMerger class
└── helpers.test.ts           # Extracted helper functions
```

Use Vitest's `describe.each` and `it.each` for parametrized tests:

```typescript
describe.each([
  { patterns: ['editor.*'], key: 'editor.fontSize', expected: true },
  { patterns: ['editor.*'], key: 'terminal.fontSize', expected: false },
  { patterns: ['editor.*', '!editor.fontSize'], key: 'editor.fontSize', expected: false },
])('isExcluded($key) with patterns $patterns', ({ patterns, key, expected }) => {
  it(`returns ${expected}`, () => {
    const matcher = new PatternMatcher(patterns);
    expect(matcher.isExcluded(key)).toBe(expected);
  });
});
```

---

## Files to Modify

- [package.json](package.json) - Add vitest dependency and test scripts
- [src/services/codeActions.ts](src/services/codeActions.ts) - Export `detectIndentationStyle`
- [src/services/forwardSync.ts](src/services/forwardSync.ts) - Export `removeWorkspaceManagerKeys`
- [src/services/diagnostics.ts](src/services/diagnostics.ts) - Export `offsetToPosition`

## Files to Create

- `vitest.config.ts` - Vitest configuration
- `tests/patternMatcher.test.ts` - PatternMatcher tests
- `tests/settingsMerger.test.ts` - SettingsMerger tests
- `tests/helpers.test.ts` - Helper function tests

