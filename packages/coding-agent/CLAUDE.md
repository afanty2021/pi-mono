[根目录](../CLAUDE.md) > [packages](../) > **coding-agent**

# @mariozechner/pi-coding-agent - Interactive AI Coding Agent

> **Package Version:** 0.72.1
> **Last Updated:** 2026-05-04 08:50:13 CST
> **Documentation Status:** Phase 2 Complete - Module Level Detail
> **Coverage:** Full CLI | Interactive TUI | Session Management | Extension System

---

## Module Responsibilities

The `@mariozechner/pi-coding-agent` package is the main product - an interactive AI coding agent that runs in terminal mode. It provides:

- **Interactive CLI:** Terminal-based AI coding assistant with TUI
- **Session Management:** Persistent sessions with branching and compaction
- **Tool System:** Built-in tools (read, bash, edit, write, grep, find, ls)
- **Extension Framework:** TypeScript-based extension system for customization
- **Multi-Mode Operation:** Interactive, print, JSON, RPC, and SDK modes
- **Skill System:** Agent Skills standard for on-demand capabilities
- **Provider Integration:** Unified access to 35+ LLM providers via pi-ai

---

## Entry Points and Startup

### Main CLI Entry Point

**File:** `dist/cli.js` (TypeScript source: `src/cli.ts`)

**Binary:** `pi` (installed via `npm install -g @mariozechner/pi-coding-agent`)

**Usage:**
```bash
# Interactive mode (default)
pi

# With initial prompt
pi "Help me refactor this code"

# Non-interactive print mode
pi -p "Summarize this codebase"

# Continue last session
pi -c

# Resume specific session
pi -r

# Different model
pi --provider openai --model gpt-4o

# Read-only mode
pi --tools read,grep,find,ls -p "Review the code"
```

### SDK Entry Point

**File:** `dist/index.js` (TypeScript source: `src/index.ts`)

**Primary Exports:**
```typescript
// Core session management
export { AgentSession, type AgentSessionConfig } from "./core/agent-session.js";

// Authentication and models
export {
  AuthStorage,
  type ApiKeyCredential,
  type OAuthCredential,
  type AuthStatus
} from "./core/auth-storage.js";

// Model registry
export { ModelRegistry } from "./model-registry.js";

// Session management
export {
  SessionManager,
  type SessionHeader,
  type SessionEntry,
  type BranchSummaryEntry
} from "./core/session-manager.js";

// Compaction
export {
  compact,
  generateSummary,
  shouldCompact,
  type CompactionResult,
  type CompactionSettings
} from "./core/compaction/index.js";

// Extension system
export type {
  Extension,
  ExtensionAPI,
  ExtensionFactory,
  ToolDefinition,
  ExtensionCommandContext
} from "./core/extensions/index.js";

// Event bus
export { createEventBus, type EventBus } from "./core/event-bus.js";
```

---

## External Interfaces

### CLI Interface

**Primary Modes:**

1. **Interactive Mode (default):**
```bash
pi [options] [@files...] [messages...]
```

2. **Print Mode:**
```bash
pi -p "One-time prompt"
pi --print @file.txt "Process this file"
```

3. **JSON Mode:**
```bash
pi --mode json "Output all events as JSON"
```

4. **RPC Mode:**
```bash
pi --mode rpc  # For process integration
```

5. **Export Mode:**
```bash
pi --export <session-file> [output.html]
```

**Key CLI Options:**

```bash
# Model selection
--provider <name>              # Provider (anthropic, openai, google, etc.)
--model <pattern>              # Model pattern or ID
--thinking <level>             # off, minimal, low, medium, high, xhigh
--list-models [search]         # List available models

# Session management
-c, --continue                 # Continue most recent session
-r, --resume                   # Browse and select session
--session <path|id>            # Use specific session file or ID
--fork <path|id>               # Fork specific session into new one
--no-session                   # Ephemeral mode (don't save)

# Tool control
--tools <list>, -t <list>      # Allowlist specific tools
--no-builtin-tools, -nbt       # Disable built-in tools
--no-tools, -nt                # Disable all tools

# Resources
-e, --extension <source>       # Load extension (repeatable)
--skill <path>                 # Load skill (repeatable)
--theme <path>                 # Load theme (repeatable)
--no-context-files, -nc        # Disable AGENTS.md/CLAUDE.md loading

# Other
--system-prompt <text>         # Replace default prompt
--append-system-prompt <text>  # Append to system prompt
--offline                      # Disable network operations
```

### SDK Interface

**Creating a Session:**

```typescript
import {
  createAgentSession,
  AuthStorage,
  ModelRegistry,
  SessionManager
} from "@mariozechner/pi-coding-agent";

// Setup
const authStorage = AuthStorage.create();
const modelRegistry = ModelRegistry.create(authStorage);

// Create session
const { session } = await createAgentSession({
  sessionManager: SessionManager.inMemory(),
  authStorage,
  modelRegistry,
});

// Use session
await session.prompt("What files are in current directory?");

// Listen to events
session.on("message_start", (event) => {
  console.log("Message started:", event);
});
```

**Advanced Runtime:**

```typescript
import {
  createAgentSessionRuntime,
  AgentSessionRuntime
} from "@mariozechner/pi-coding-agent";

const runtime = await createAgentSessionRuntime({
  sessionManager: SessionManager.inMemory(),
  authStorage,
  modelRegistry,
});

// Multi-session management
const session1 = await runtime.createSession();
const session2 = await runtime.createSession();
```

### Extension API

**Extension Factory:**

```typescript
export default function (pi: ExtensionAPI) {
  // Register custom tool
  pi.registerTool({
    name: "deploy",
    description: "Deploy to production",
    inputSchema: {
      type: "object",
      properties: {
        environment: { type: "string" }
      }
    },
    async execute(input, context) {
      return { success: true, url: "..." };
    }
  });

  // Register command
  pi.registerCommand("stats", {
    description: "Show statistics",
    handler: async (ctx) => {
      pi.ui.renderMessage({ type: "text", text: "Stats..." });
    }
  });

  // Event handler
  pi.on("tool_call", async (event, ctx) => {
    console.log("Tool called:", event.toolCall.name);
  });

  // Keyboard shortcut
  pi.registerShortcut({ key: "ctrl+d" }, async (ctx) => {
    pi.ui.renderMessage({ type: "text", text: "Deployed!" });
  });
}
```

**Extension API Capabilities:**

```typescript
interface ExtensionAPI {
  // Tools
  registerTool(definition: ToolDefinition): void;
  unregisterTool(name: string): void;

  // Commands
  registerCommand(name: string, command: Command): void;
  unregisterCommand(name: string): void;

  // Shortcuts
  registerShortcut(keybinding: Keybinding, handler: Handler): void;

  // Events
  on(event: string, handler: EventHandler): void;
  off(event: string, handler: EventHandler): void;

  // Providers
  registerProvider(provider: Provider): void;

  // UI
  ui: {
    renderMessage(message: Message): void;
    replaceEditor(component: Component): void;
    addWidget(options: WidgetOptions): void;
    showNotification(message: string): void;
  };

  // Context
  getContext(): ExtensionContext;
}
```

---

## Key Dependencies and Configuration

### Runtime Dependencies

**Core Dependencies:**
```json
{
  "@mariozechner/jiti": "^2.6.2",
  "@mariozechner/pi-agent-core": "^0.72.1",
  "@mariozechner/pi-ai": "^0.72.1",
  "@mariozechner/pi-tui": "^0.72.1"
}
```

**Tool Support:**
```json
{
  "chalk": "^5.5.0",
  "cli-highlight": "^2.1.11",
  "diff": "^8.0.2",
  "glob": "^13.0.1",
  "ignore": "^7.0.5",
  "minimatch": "^10.2.3",
  "yaml": "^2.8.2"
}
```

**File Operations:**
```json
{
  "file-type": "^21.1.1",
  "extract-zip": "^2.0.1",
  "uuid": "^14.0.0"
}
```

**Optional Dependencies:**
```json
{
  "@mariozechner/clipboard": "^0.3.5",
  "koffi": "^2.9.0"
}
```

### Configuration Files

**Settings Structure:**
```bash
# Global settings
~/.pi/agent/settings.json

# Project settings (overrides global)
.pi/settings.json
```

**Key Settings:**
```json
{
  "thinkingLevel": "medium",
  "theme": "dark",
  "steeringMode": "one-at-a-time",
  "followUpMode": "one-at-a-time",
  "transport": "auto",
  "autoCompact": {
    "enabled": true,
    "thresholdRatio": 0.8
  },
  "enableInstallTelemetry": true,
  "npmCommand": ["npm"]
}
```

**Keybindings:**
```bash
~/.pi/agent/keybindings.json  # Global keybindings
.pi/keybindings.json          # Project keybindings
```

**Models Configuration:**
```bash
~/.pi/agent/models.json  # Custom models and providers
```

### Environment Variables

**Path Overrides:**
```bash
PI_CODING_AGENT_DIR=/path/to/config    # Config directory (default: ~/.pi/agent)
PI_CODING_AGENT_SESSION_DIR=/path/sessions  # Session directory
PI_PACKAGE_DIR=/path/to/packages       # Package directory
```

**Feature Flags:**
```bash
PI_OFFLINE=1                    # Disable all network operations
PI_SKIP_VERSION_CHECK=1         # Skip version check
PI_TELEMETRY=0                  # Disable telemetry
PI_CACHE_RETENTION=long         # Extended prompt cache
PI_NO_LOCAL_LLM=1               # Skip local LLM tests
```

---

## Data Models

### Session Format

**JSONL Structure:**
```typescript
interface SessionEntry {
  id: string;
  parentId: string | null;
  timestamp: number;
  type: "user" | "assistant" | "tool" | "system" | "custom";
  content: MessageContent;
  usage?: TokenUsage;
  model?: ModelInfo;
}

interface SessionHeader {
  version: number;
  created: number;
  updated: number;
  name?: string;
  workingDirectory: string;
  branchId: string;
}
```

**Session Storage:**
```bash
# Organized by working directory
~/.pi/agent/sessions/
  ├── /path/to/project1/
  │   ├── session-abc123.jsonl
  │   └── session-def456.jsonl
  └── /path/to/project2/
      └── session-ghi789.jsonl
```

### Message Types

**Core Message Structure:**
```typescript
type AgentMessage =
  | { type: "text"; text: string }
  | { type: "tool_call"; toolCall: ToolCallMessage }
  | { type: "tool_result"; toolResult: ToolResultMessage }
  | { type: "thinking"; text: string }
  | { type: "custom"; custom: CustomMessage };

interface ToolCallMessage {
  id: string;
  name: string;
  input: unknown;
}

interface ToolResultMessage {
  toolCallId: string;
  output: string;
  error?: boolean;
}
```

### Tool Definitions

**Built-in Tools:**
```typescript
interface ToolDefinition {
  name: string;
  description: string;
  inputSchema: {
    type: "object";
    properties: Record<string, unknown>;
    required?: string[];
  };
  execute: (input: unknown, context: ToolContext) => Promise<ToolResult>;
}

// Built-in tools:
// - read: Read file contents
// - write: Write/create files
// - edit: Edit files with search/replace
// - bash: Execute shell commands
// - grep: Search files with regex
// - find: Find files by pattern
// - ls: List directory contents
```

---

## Testing and Quality

### Test Organization

**Test Structure:**
```
packages/coding-agent/test/
├── suite/
│   ├── harness.ts              # Test harness with faux provider
│   ├── agent-session.test.ts   # Session management tests
│   ├── compaction.test.ts      # Compaction algorithm tests
│   ├── tools.test.ts           # Built-in tool tests
│   ├── extensions.test.ts      # Extension system tests
│   └── regressions/            # Issue-specific regression tests
│       ├── 1234-bug-fix.test.ts
│       └── 5678-feature.test.ts
└── unit/
    ├── auth-storage.test.ts
    ├── model-registry.test.ts
    └── session-manager.test.ts
```

### Test Guidelines

**From AGENTS.md:**
- Use `test/suite/harness.ts` plus the faux provider
- Do NOT use real provider APIs, real API keys, or paid tokens
- Run tests from package root, not repo root
- Name regression tests: `<issue-number>-<short-slug>.test.ts`
- Always run tests after creating/modifying them

**Test Commands:**
```bash
cd packages/coding-agent

# Run all tests
npx tsx ../../node_modules/vitest/dist/cli.js --run

# Run specific test
npx tsx ../../node_modules/vitest/dist/cli.js --run test/suite/agent-session.test.ts

# Run with coverage
npx tsx ../../node_modules/vitest/dist/cli.js --run --coverage
```

### Quality Tools

**Pre-commit Checks:**
```bash
# From repo root
npm run check  # Runs Biome, tsc, and browser-smoke check

# From package root
cd packages/coding-agent
npm run check  # Package-specific checks
```

---

## Common Issues and Solutions

### Session Issues

**Issue:** Session not saving

**Solution:**
```bash
# Check session directory
ls -la ~/.pi/agent/sessions/

# Check file permissions
chmod 755 ~/.pi/agent/sessions/

# Use ephemeral mode to test
pi --no-session
```

**Issue:** Cannot resume session

**Solution:**
```bash
# List available sessions
pi -r

# Use specific session ID
pi --session abc123

# Fork existing session
pi --fork session-abc123.jsonl
```

### Extension Issues

**Issue:** Extension not loading

**Solution:**
```bash
# Check extension syntax
cd ~/.pi/agent/extensions/my-ext/
npx tsc --noEmit my-extension.ts

# Reload extensions
pi
/reload

# Check loaded resources
/session  # Shows loaded extensions in header
```

**Issue:** Extension tool not available

**Solution:**
```typescript
// Ensure tool is registered
pi.registerTool({
  name: "my-tool",
  // ... must have name, description, inputSchema, execute
});

// Check tool is not disabled
pi --no-builtin-tools  # Disables built-in tools only
pi --tools my-tool      # Enables only this tool
```

### Model Issues

**Issue:** Model not found

**Solution:**
```bash
# List available models
pi --list-models anthropic

# Check provider authentication
pi
/login  # Select provider and authenticate

# Check models.json
cat ~/.pi/agent/models.json
```

**Issue:** API key not detected

**Solution:**
```bash
# Set environment variable
export ANTHROPIC_API_KEY=sk-ant-...

# Or use OAuth
pi
/login

# Check auth storage
cat ~/.pi/agent/auth.json
```

---

## File Structure

### Core Files

```
packages/coding-agent/src/
├── cli.ts                      # CLI entry point
├── index.ts                    # SDK entry point
├── config.ts                   # Path and version constants
├── modes/
│   ├── interactive/
│   │   ├── index.ts            # Interactive mode entry
│   │   ├── ui.ts               # Main TUI component
│   │   ├── theme/              # Theme definitions
│   │   └── keybindings/        # Keybinding handlers
│   ├── print.ts                # Print mode
│   ├── json.ts                 # JSON mode
│   ├── rpc.ts                  # RPC mode
│   └── export.ts               # HTML export
├── core/
│   ├── agent-session.ts        # Core session management
│   ├── auth-storage.ts         # Authentication storage
│   ├── session-manager.ts      # Session file management
│   ├── event-bus.ts            # Event system
│   ├── compaction/             # Context compaction
│   │   ├── index.ts
│   │   ├── algorithm.ts
│   │   └── summary.ts
│   ├── extensions/             # Extension system
│   │   ├── index.ts
│   │   ├── runner.ts
│   │   ├── api.ts
│   │   └── types.ts
│   ├── bash-executor.ts        # Bash execution
│   ├── export-html/            # HTML export
│   ├── model-registry.ts       # Model management
│   ├── prompt-templates.ts     # Template expansion
│   ├── resource-loader.ts      # Resource discovery
│   ├── settings-manager.ts     # Settings management
│   ├── slash-commands.ts       # Built-in commands
│   ├── system-prompt.ts        # System prompt builder
│   └── tools/                  # Built-in tools
│       ├── bash.ts
│       ├── edit.ts
│       ├── find.ts
│       ├── grep.ts
│       ├── ls.ts
│       ├── read.ts
│       └── write.ts
├── utils/
│   ├── frontmatter.ts          # Frontmatter parsing
│   ├── sleep.ts                # Async utilities
│   └── ...
└── messages.ts                 # Message type definitions
```

### Configuration Directories

```
~/.pi/agent/
├── settings.json               # Global settings
├── keybindings.json            # Global keybindings
├── models.json                 # Custom models
├── auth.json                   # API keys and OAuth tokens
├── AGENTS.md                   # Global context file
├── SYSTEM.md                   # Global system prompt
├── extensions/                 # Global extensions
├── skills/                     # Global skills
├── prompts/                    # Global prompt templates
├── themes/                     # Global themes
└── sessions/                   # Session storage
    └── /path/to/project/
        └── session-*.jsonl
```

---

## Related Files

### Dependencies
- `packages/ai/src/index.ts` - LLM provider integration
- `packages/agent/src/index.ts` - Core agent framework
- `packages/tui/src/index.ts` - Terminal UI components

### Configuration
- `tsconfig.build.json` - TypeScript build configuration
- `vitest.config.ts` - Test configuration

### Documentation
- `README.md` - Comprehensive user guide
- `docs/` - Detailed documentation
  - `extensions.md` - Extension development
  - `skills.md` - Skill system
  - `providers.md` - Provider setup
  - `sessions.md` - Session management
  - `compaction.md` - Compaction algorithm
  - `development.md` - Development setup

---

## Development Notes

### Creating Extensions

**Extension Template:**
```typescript
// my-extension.ts
import type { Extension } from "@mariozechner/pi-coding-agent/extensions";

export default function (api: ExtensionAPI) {
  // Register tool
  api.registerTool({
    name: "my-tool",
    description: "Does something useful",
    inputSchema: {
      type: "object",
      properties: {
        param: { type: "string" }
      }
    },
    async execute(input, context) {
      return { result: "success" };
    }
  });

  // Register command
  api.registerCommand("mycmd", {
    description: "My custom command",
    handler: async (ctx) => {
      api.ui.showNotification("Command executed!");
    }
  });

  // Event handler
  api.on("message_start", async (event, ctx) => {
    console.log("Message started");
  });
}
```

**Install Extension:**
```bash
# Copy to global extensions
cp my-extension.ts ~/.pi/agent/extensions/

# Or project-local
cp my-extension.ts .pi/extensions/

# Reload
pi
/reload
```

### Creating Skills

**Skill Template:**
```markdown
<!-- ~/.pi/agent/skills/my-skill/SKILL.md -->
# My Skill

Use this skill when the user asks about X.

## Steps
1. First, do this
2. Then, do that
3. Finally, verify

## Examples
Example: User says "help with X"
```

**Use Skill:**
```bash
pi
/skill:my-skill
```

### Creating Themes

**Theme Template:**
```json
{
  "name": "my-theme",
  "colors": {
    "primary": "#00ff00",
    "secondary": "#ff00ff",
    "background": "#1a1a1a",
    "text": "#ffffff"
  },
  "editor": {
    "border": "#00ff00",
    "text": "#ffffff"
  }
}
```

**Install Theme:**
```bash
cp my-theme.json ~/.pi/agent/themes/
pi
/settings  # Select theme
```

---

## Change Log

### 2026-05-04 08:50:13 CST

**Initial Documentation:**
- Created comprehensive package-level documentation
- Documented all CLI modes and options
- Added extension API examples
- Included session management details
- Documented testing strategies and quality standards

**Key Features Documented:**
- Interactive TUI mode with full command reference
- Session branching and compaction
- Extension system with TypeScript API
- Built-in tools (read, bash, edit, write, grep, find, ls)
- Multi-mode operation (interactive, print, JSON, RPC, SDK)
- Skill system and prompt templates

---

*For project-level documentation, see [root CLAUDE.md](../CLAUDE.md)*
