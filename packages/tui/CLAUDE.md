[根目录](../CLAUDE.md) > [packages](../) > **tui**

# @mariozechner/pi-tui - Terminal UI Library with Differential Rendering

> **Package Version:** 0.72.1
> **Last Updated:** 2026-05-04 08:50:13 CST
> **Documentation Status:** Phase 2 Complete - Module Level Detail
> **Coverage:** Terminal UI components | Differential rendering | Keyboard handling | Fuzzy matching

---

## Module Responsibilities

The `@mariozechner/pi-tui` package provides a comprehensive Terminal User Interface (TUI) library with efficient differential rendering. It handles:

- **Differential Rendering:** Only redraw changed parts of the screen for performance
- **UI Components:** Reusable terminal components (Editor, Markdown, SelectList, etc.)
- **Keyboard Handling:** Cross-platform keyboard input with keybinding support
- **Fuzzy Matching:** Fast fuzzy search for autocomplete and filtering
- **Theme System:** Customizable themes for components
- **Image Rendering:** Terminal image display with sixel support (optional)

---

## Entry Points and Startup

### Main Entry Point

**File:** `dist/index.js` (TypeScript source: `src/index.ts`)

**Primary Exports:**
```typescript
// Core TUI interfaces
export {
  TUI,
  type TUIConfig,
  type TUIEvent,
  type TUIEventHandler
} from "./tui.js";

// Components
export { Box } from "./components/box.js";
export { Editor, type EditorOptions, type EditorTheme } from "./components/editor.js";
export { Markdown, type MarkdownTheme } from "./components/markdown.js";
export { SelectList, type SelectListTheme } from "./components/select-list.js";
export { SettingsList, type SettingsListTheme } from "./components/settings-list.js";
export { Input } from "./components/input.js";
export { Text } from "./components/text.js";
export { Image, type ImageOptions, type ImageTheme } from "./components/image.js";
export { Loader, type LoaderIndicatorOptions } from "./components/loader.js";
export { CancellableLoader } from "./components/cancellable-loader.js";
export { Spacer } from "./components/spacer.js";
export { TruncatedText } from "./components/truncated-text.js";

// Autocomplete support
export {
  type AutocompleteProvider,
  type AutocompleteSuggestions,
  CombinedAutocompleteProvider
} from "./autocomplete.js";

// Fuzzy matching
export {
  fuzzyMatch,
  fuzzyFilter,
  type FuzzyMatch
} from "./fuzzy.js";

// Keybindings
export {
  KeybindingsManager,
  getKeybindings,
  setKeybindings,
  type Keybinding,
  type Keybindings,
  type KeybindingsConfig,
  TUI_KEYBINDINGS
} from "./keybindings.js";

// Keyboard input handling
export {
  decodeKittyPrintable,
  isKeyRelease,
  createKeyReader
} from "./input.js";
```

---

## External Interfaces

### Core TUI API

**Creating a TUI Instance:**

```typescript
import { TUI } from "@mariozechner/pi-tui";

const tui = new TUI({
  stdin: process.stdin,
  stdout: process.stdout,
  // Optional: custom key reader
  keyReader: createKeyReader(process.stdin)
});

// Start TUI
await tui.start();

// Render content
tui.render(
  <Box>
    <Text>Hello World</Text>
  </Box>
);

// Handle input
tui.on("key", (event) => {
  if (event.key === "q") {
    tui.stop();
  }
});

// Stop TUI
await tui.stop();
```

### Component API

**Editor Component:**

```typescript
import { Editor } from "@mariozechner/pi-tui";

const editor = new Editor({
  value: "Initial text",
  placeholder: "Type here...",
  multiline: true,
  onChange: (value) => {
    console.log("Value changed:", value);
  },
  onSubmit: (value) => {
    console.log("Submitted:", value);
  },
  theme: {
    border: "cyan",
    text: "white",
    placeholder: "gray"
  }
});

// Get value
const value = editor.getValue();

// Set value
editor.setValue("New text");

// Focus editor
editor.focus();
```

**SelectList Component:**

```typescript
import { SelectList } from "@mariozechner/pi-tui";

const list = new SelectList({
  items: [
    { id: "1", label: "Option 1" },
    { id: "2", label: "Option 2" },
    { id: "3", label: "Option 3" }
  ],
  onSelect: (item) => {
    console.log("Selected:", item);
  },
  onCancel: () => {
    console.log("Cancelled");
  },
  searchable: true,
  theme: {
    selected: { bg: "blue", fg: "white" },
    border: "cyan"
  }
});

// Open list
await list.open();
```

**Markdown Component:**

```typescript
import { Markdown } from "@mariozechner/pi-tui";

const markdown = new Markdown({
  content: "# Hello\n\nThis is **bold** text.",
  theme: {
    h1: { fg: "cyan", bold: true },
    h2: { fg: "blue", bold: true },
    bold: { bold: true },
    code: { bg: "gray", fg: "white" }
  }
});
```

### Keyboard API

**Keybinding Manager:**

```typescript
import { KeybindingsManager } from "@mariozechner/pi-tui";

const manager = new KeybindingsManager({
  "ctrl+c": "quit",
  "ctrl+l": "open_model_selector",
  "escape": "cancel"
});

// Register handler
manager.on("quit", () => {
  tui.stop();
});

// Match key
const action = manager.match({ key: "c", ctrl: true });
console.log("Action:", action); // "quit"
```

**Key Reading:**

```typescript
import { createKeyReader } from "@mariozechner/pi-tui";

const keyReader = createKeyReader(process.stdin);

for await (const key of keyReader) {
  console.log("Key pressed:", key);
  // key: { key: string, ctrl?: boolean, shift?: boolean, meta?: boolean }
}
```

### Fuzzy Matching API

**Fuzzy Filter:**

```typescript
import { fuzzyFilter } from "@mariozechner/pi-tui";

const items = [
  "apple",
  "application",
  "banana",
  "apricot"
];

const results = fuzzyFilter(items, "app");
// [
//   { value: "apple", score: 3, matches: [0, 1, 2] },
//   { value: "application", score: 3, matches: [0, 1, 2] },
//   { value: "apricot", score: 2, matches: [0, 1] }
// ]
```

---

## Key Dependencies and Configuration

### Runtime Dependencies

**Core Dependencies:**
```json
{
  "chalk": "^5.5.0",
  "marked": "^15.0.12",
  "get-east-asian-width": "^1.3.0"
}
```

**Optional Dependencies:**
```json
{
  "koffi": "^2.9.0"
}
```

**Dev Dependencies:**
```json
{
  "@xterm/headless": "^5.5.0",
  "@xterm/xterm": "^5.5.0"
}
```

### Configuration

**TypeScript Configuration:**
```json
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "NodeNext",
    "moduleResolution": "NodeNext",
    "strict": true
  }
}
```

**Build Scripts:**
```json
{
  "scripts": {
    "clean": "shx rm -rf dist",
    "build": "tsgo -p tsconfig.build.json",
    "dev": "tsgo -p tsconfig.build.json --watch --preserveWatchOutput",
    "test": "node --test --import tsx test/*.test.ts"
  }
}
```

---

## Data Models

### Component Types

**Base Component:**
```typescript
interface Component {
  render(): string;
  handleEvent(event: TUIEvent): void;
  focus(): void;
  blur(): void;
}

interface BoxProps {
  border?: boolean;
  padding?: number;
  width?: number;
  height?: number;
  children?: Component[];
}
```

**Editor Types:**
```typescript
interface EditorOptions {
  value?: string;
  placeholder?: string;
  multiline?: boolean;
  password?: boolean;
  maxLength?: number;
  onChange?: (value: string) => void;
  onSubmit?: (value: string) => void;
  onCancel?: () => void;
  theme?: EditorTheme;
}

interface EditorTheme {
  border?: string;
  text?: string;
  placeholder?: string;
  cursor?: string;
  focusedBorder?: string;
}
```

**SelectList Types:**
```typescript
interface SelectListOptions<T> {
  items: SelectItem<T>[];
  onSelect?: (item: SelectItem<T>) => void;
  onCancel?: () => void;
  searchable?: boolean;
  filterText?: string;
  layoutOptions?: SelectListLayoutOptions;
  theme?: SelectListTheme;
}

interface SelectItem<T> {
  id: string;
  label: string;
  secondaryText?: string;
  metadata?: T;
}
```

### Keyboard Types

**Keybinding Types:**
```typescript
interface Keybinding {
  key: string;
  ctrl?: boolean;
  shift?: boolean;
  meta?: boolean;
}

interface KeybindingsConfig {
  [keybinding: string]: string;  // action name
}

interface KeyEvent extends Keybinding {
  sequence: string;
  name: string;
}
```

### Theme Types

**Theme System:**
```typescript
interface Theme {
  colors: {
    primary?: string;
    secondary?: string;
    background?: string;
    text?: string;
    border?: string;
    [key: string]: string | undefined;
  };
  components?: {
    editor?: EditorTheme;
    markdown?: MarkdownTheme;
    selectList?: SelectListTheme;
    [key: string]: unknown;
  };
}
```

---

## Testing and Quality

### Test Organization

**Test Structure:**
```
packages/tui/test/
├── components/
│   ├── editor.test.ts
│   ├── select-list.test.ts
│   └── markdown.test.ts
├── fuzzy.test.ts              # Fuzzy matching tests
├── keybindings.test.ts        # Keybinding tests
└── input.test.ts              # Input handling tests
```

### Test Guidelines

**From AGENTS.md:**
- Run tests from package root
- Always run tests after creating/modifying them

**Test Commands:**
```bash
cd packages/tui

# Run all tests
npm test

# Run specific test
npm test test/components/editor.test.ts
```

### Quality Tools

**Type Checking:**
```bash
# From repo root
npm run check

# From package root
cd packages/tui
npx tsc --noEmit
```

---

## Common Issues and Solutions

### Rendering Issues

**Issue:** Screen not updating correctly

**Solution:**
```typescript
// Ensure differential rendering is enabled
const tui = new TUI({
  differentialRendering: true
});

// Force full redraw
tui.forceRedraw();

// Check terminal size
const size = tui.getScreenSize();
console.log("Screen size:", size.cols, "x", size.rows);
```

**Issue:** Text wrapping issues

**Solution:**
```typescript
// Use TruncatedText for long text
import { TruncatedText } from "@mariozechner/pi-tui";

const text = new TruncatedText({
  text: "Very long text...",
  maxLength: 50,
  ellipsis: "..."
});

// Or set explicit width
const box = new Box({ width: 80 });
```

### Keyboard Issues

**Issue:** Special keys not working

**Solution:**
```typescript
// Check key reader is properly configured
const keyReader = createKeyReader(process.stdin, {
  escapeSequenceTimeout: 500  // Increase timeout
});

// Debug key events
tui.on("key", (event) => {
  console.log("Key event:", event);
});
```

**Issue:** Keybindings not matching

**Solution:**
```typescript
// Use proper keybinding format
const keybindings = {
  "ctrl+c": "quit",      // Correct
  "C-c": "quit",         // Incorrect
  "ctrl+C": "quit"       // Incorrect
};

// Check for conflicts
const manager = new KeybindingsManager(keybindings);
const conflicts = manager.checkConflicts();
console.log("Conflicts:", conflicts);
```

### Performance Issues

**Issue:** Slow rendering

**Solution:**
```typescript
// Enable differential rendering
const tui = new TUI({
  differentialRendering: true,
  renderInterval: 16  // ~60fps
});

// Reduce render frequency
tui.setThrottle(50);  // 20fps

// Optimize component rendering
class OptimizedComponent extends Component {
  private cachedRender: string | null = null;

  render() {
    if (this.cachedRender && !this.shouldUpdate()) {
      return this.cachedRender;
    }
    this.cachedRender = this.doRender();
    return this.cachedRender;
  }

  shouldUpdate() {
    // Check if props changed
    return this.props !== this.lastProps;
  }
}
```

---

## File Structure

### Core Files

```
packages/tui/src/
├── index.ts                    # Main entry point
├── tui.ts                      # Core TUI class
├── input.ts                    # Keyboard input handling
├── keybindings.ts              # Keybinding management
├── fuzzy.ts                    # Fuzzy matching
├── autocomplete.ts             # Autocomplete support
├── components/
│   ├── box.ts                  # Box container
│   ├── editor.ts               # Text editor
│   ├── input.ts                # Single-line input
│   ├── text.ts                 # Text display
│   ├── markdown.ts             # Markdown renderer
│   ├── select-list.ts          # Selection list
│   ├── settings-list.ts        # Settings editor
│   ├── image.ts                # Image display
│   ├── loader.ts               # Loading indicator
│   ├── cancellable-loader.ts   # Cancellable loader
│   ├── spacer.ts               # Spacer component
│   └── truncated-text.ts       # Truncated text
├── editor-component.ts         # Editor component interface
└── utils/
    ├── colors.ts               # Color utilities
    ├── measure.ts              # Text measurement
    └── wrap.ts                 # Text wrapping
```

---

## Related Files

### Dependencies
- `packages/coding-agent/src/modes/interactive/` - Main TUI usage
- `packages/web-ui/src/` - Web UI shares some concepts

### Configuration
- `tsconfig.build.json` - TypeScript build configuration

### Documentation
- `README.md` - Package overview and usage

---

## Development Notes

### Creating Custom Components

**Component Template:**
```typescript
import { Component } from "@mariozechner/pi-tui";

class MyComponent extends Component {
  constructor(private props: { text: string }) {
    super();
  }

  render(): string {
    return this.props.text;
  }

  handleEvent(event: TUIEvent): void {
    // Handle events
  }

  focus(): void {
    this.focused = true;
  }

  blur(): void {
    this.focused = false;
  }
}
```

### Differential Rendering

**How It Works:**
1. TUI maintains a virtual screen buffer
2. On each render, compare new output with buffer
3. Only send changed lines to terminal
4. Update buffer with new content

**Optimization Tips:**
```typescript
// Use stable component IDs
<Box id="stable-id">
  {/* Components with stable IDs diff better */}
</Box>

// Avoid frequent re-renders
const [value, setValue] = useState("");
const debouncedSetValue = debounce(setValue, 100);

// Use shouldUpdate for complex components
class ExpensiveComponent extends Component {
  shouldUpdate(newProps: Props): boolean {
    return newProps.data !== this.props.data;
  }
}
```

### Theme Customization

**Creating Themes:**
```typescript
const darkTheme = {
  colors: {
    primary: "#00ff00",
    secondary: "#ff00ff",
    background: "#1a1a1a",
    text: "#ffffff"
  },
  components: {
    editor: {
      border: "cyan",
      text: "white",
      placeholder: "gray"
    },
    markdown: {
      h1: { fg: "cyan", bold: true },
      h2: { fg: "blue", bold: true },
      code: { bg: "gray", fg: "white" }
    }
  }
};
```

---

## Change Log

### 2026-05-04 08:50:13 CST

**Initial Documentation:**
- Created comprehensive package-level documentation
- Documented all TUI components and their APIs
- Added keyboard handling and keybinding examples
- Included fuzzy matching API documentation
- Documented differential rendering optimization

**Key Features Documented:**
- Terminal UI components (Editor, SelectList, Markdown, etc.)
- Differential rendering for performance
- Cross-platform keyboard input handling
- Fuzzy matching for autocomplete
- Theme system for customization
- Image rendering with sixel support

---

*For project-level documentation, see [root CLAUDE.md](../CLAUDE.md)*
