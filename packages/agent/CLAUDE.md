[根目录](../CLAUDE.md) > [packages](../) > **agent**

# @mariozechner/pi-agent-core - General-Purpose Agent Runtime

> **Package Version:** 0.72.1
> **Last Updated:** 2026-05-04 08:50:13 CST
> **Documentation Status:** Phase 2 Complete - Module Level Detail
> **Coverage:** Core agent framework | Tool calling | State management | Transport abstraction

---

## Module Responsibilities

The `@mariozechner/pi-agent-core` package provides the foundational agent framework that powers the entire pi ecosystem. It handles:

- **Agent Loop:** Core request/response loop with streaming support
- **Tool Calling:** Automatic tool execution and result integration
- **State Management:** Agent state tracking and transitions
- **Transport Abstraction:** Pluggable transport layer (HTTP, WebSocket, SSE)
- **Message Handling:** Unified message format and processing
- **Attachment Support:** File and image attachments in messages

---

## Entry Points and Startup

### Main Entry Point

**File:** `dist/index.js` (TypeScript source: `src/index.ts`)

**Primary Exports:**
```typescript
// Core Agent class and utilities
export {
  Agent,
  type AgentConfig,
  type AgentMessage,
  type AgentState,
  type AgentEvent,
  type Tool,
  type ThinkingLevel
} from "./agent.js";

// Agent loop functions
export {
  runAgentLoop,
  runAgentLoopWithEvents,
  type AgentLoopConfig,
  type AgentLoopResult
} from "./agent-loop.js";

// Proxy utilities
export {
  createAgentProxy,
  type AgentProxy
} from "./proxy.js";

// Core types
export type {
  Message,
  ToolCall,
  ToolResult,
  Context,
  Model,
  AssistantMessage,
  UserMessage,
  SystemMessage
} from "./types.js";
```

---

## External Interfaces

### Core Agent API

**Creating an Agent:**

```typescript
import { Agent } from "@mariozechner/pi-agent-core";
import { stream } from "@mariozechner/pi-ai";

const agent = new Agent({
  model: {
    id: "claude-sonnet-4-20250514",
    api: "anthropic-messages",
    provider: "anthropic"
  },
  tools: [
    {
      name: "read_file",
      description: "Read a file",
      inputSchema: {
        type: "object",
        properties: {
          path: { type: "string" }
        },
        required: ["path"]
      },
      execute: async (input) => {
        // Tool implementation
        return { content: "File contents..." };
      }
    }
  ],
  system: "You are a helpful coding assistant.",
  thinking: "medium"
});

// Run agent
const result = await agent.run({
  messages: [
    { role: "user", content: "Read package.json" }
  ]
});
```

**Agent Configuration:**

```typescript
interface AgentConfig {
  model: Model;
  tools?: Tool[];
  system?: string;
  thinking?: ThinkingLevel;
  temperature?: number;
  maxTokens?: number;
  transport?: "http" | "websocket" | "sse";
}

interface Tool {
  name: string;
  description: string;
  inputSchema: {
    type: "object";
    properties: Record<string, unknown>;
    required?: string[];
  };
  execute: (input: unknown, context: ToolContext) => Promise<ToolResult>;
}
```

### Agent Loop API

**Running the Agent Loop:**

```typescript
import { runAgentLoop, type AgentLoopConfig } from "@mariozechner/pi-agent-core";

const config: AgentLoopConfig = {
  agent,
  messages: [
    { role: "user", content: "Help me write code" }
  ],
  onEvent: (event) => {
    switch (event.type) {
      case "message_start":
        console.log("Message started");
        break;
      case "text":
        console.log("Text delta:", event.delta);
        break;
      case "tool_call":
        console.log("Tool called:", event.toolCall.name);
        break;
      case "tool_result":
        console.log("Tool result:", event.result);
        break;
      case "message_end":
        console.log("Message completed");
        break;
    }
  }
};

const result = await runAgentLoop(config);
```

**Agent Events:**

```typescript
type AgentEvent =
  | { type: "message_start"; message: AssistantMessage }
  | { type: "text"; delta: string }
  | { type: "tool_call"; toolCall: ToolCall }
  | { type: "tool_result"; toolCallId: string; result: ToolResult }
  | { type: "thinking"; text: string }
  | { type: "usage"; usage: TokenUsage }
  | { type: "message_end"; message: AssistantMessage }
  | { type: "error"; error: Error };
```

### State Management API

**Agent States:**

```typescript
type AgentState =
  | "idle"           // Not running
  | "starting"       // Initializing
  | "running"        // Processing request
  | "thinking"       // Generating response
  | "tool_executing" // Running tool
  | "stopping"       // Shutting down
  | "error";         // Error occurred

// Get current state
const state = agent.getState();
console.log("Agent state:", state);

// Listen to state changes
agent.on("state_change", (event) => {
  console.log("State changed:", event.from, "->", event.to);
});
```

### Transport Layer API

**Transport Abstraction:**

```typescript
interface AgentConfig {
  transport?: "http" | "websocket" | "sse" | "auto";
}

// HTTP transport (default)
const agent = new Agent({
  model,
  transport: "http"
});

// WebSocket transport (for streaming)
const agent = new Agent({
  model,
  transport: "websocket"
});

// Server-Sent Events (SSE)
const agent = new Agent({
  model,
  transport: "sse"
});
```

---

## Key Dependencies and Configuration

### Runtime Dependencies

**Core Dependencies:**
```json
{
  "@mariozechner/pi-ai": "^0.72.1",
  "typebox": "^1.1.24"
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
    "strict": true,
    "esModuleInterop": true
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
    "test": "vitest --run"
  }
}
```

---

## Data Models

### Core Types

**Message Types:**
```typescript
type Message =
  | UserMessage
  | AssistantMessage
  | SystemMessage;

interface UserMessage {
  role: "user";
  content: Content[];
}

interface AssistantMessage {
  role: "assistant";
  content: Content[];
  toolCalls?: ToolCall[];
}

interface SystemMessage {
  role: "system";
  content: string;
}

type Content =
  | { type: "text"; text: string }
  | { type: "image"; image: ImageContent };
```

**Tool Call Types:**
```typescript
interface ToolCall {
  id: string;
  name: string;
  input: unknown;
}

interface ToolResult {
  output?: string;
  error?: string;
  metadata?: Record<string, unknown>;
}
```

**Agent Configuration:**
```typescript
interface AgentConfig {
  model: Model;
  tools?: Tool[];
  system?: string;
  thinking?: ThinkingLevel;
  temperature?: number;
  maxTokens?: number;
  topP?: number;
  transport?: Transport;
}

type ThinkingLevel =
  | "off"
  | "minimal"
  | "low"
  | "medium"
  | "high"
  | "xhigh";
```

### State Models

**Agent State:**
```typescript
interface AgentStateData {
  state: AgentState;
  currentMessage?: AssistantMessage;
  currentToolCall?: ToolCall;
  error?: Error;
  metadata: Record<string, unknown>;
}
```

---

## Testing and Quality

### Test Organization

**Test Structure:**
```
packages/agent/test/
├── agent.test.ts              # Core agent functionality
├── agent-loop.test.ts         # Agent loop tests
├── tools.test.ts              # Tool execution tests
├── state.test.ts              # State management tests
└── transport.test.ts          # Transport layer tests
```

### Test Guidelines

**From AGENTS.md:**
- Use mock providers - NO real API calls
- Run tests from package root
- Always run tests after creating/modifying them

**Test Commands:**
```bash
cd packages/agent

# Run all tests
npx tsx ../../node_modules/vitest/dist/cli.js --run

# Run specific test
npx tsx ../../node_modules/vitest/dist/cli.js --run test/agent.test.ts
```

### Quality Tools

**Type Checking:**
```bash
# From repo root
npm run check

# From package root
cd packages/agent
npx tsc --noEmit
```

---

## Common Issues and Solutions

### Tool Execution Issues

**Issue:** Tool not being called by agent

**Solution:**
```typescript
// Ensure tool is properly registered
const tool = {
  name: "my_tool",
  description: "Clear description of what tool does",
  inputSchema: {
    type: "object",
    properties: {
      param: {
        type: "string",
        description: "Clear parameter description"
      }
    },
    required: ["param"]
  },
  execute: async (input, context) => {
    return { output: "Result" };
  }
};

const agent = new Agent({
  model,
  tools: [tool]  // Tool must be in tools array
});
```

**Issue:** Tool execution timing out

**Solution:**
```typescript
// Set appropriate timeout
const tool = {
  name: "slow_tool",
  // ...
  execute: async (input, context) => {
    // Use AbortSignal for timeout
    const controller = new AbortController();
    const timeout = setTimeout(() => controller.abort(), 30000);

    try {
      const result = await slowOperation(controller.signal);
      clearTimeout(timeout);
      return { output: result };
    } catch (error) {
      clearTimeout(timeout);
      return { error: "Operation timed out" };
    }
  }
};
```

### State Management Issues

**Issue:** Agent state not updating correctly

**Solution:**
```typescript
// Listen to state changes
agent.on("state_change", (event) => {
  console.log(`State: ${event.from} -> ${event.to}`);
});

// Check current state
const state = agent.getState();
console.log("Current state:", state);

// Wait for specific state
await agent.waitForState("idle");
```

### Transport Issues

**Issue:** WebSocket connection failing

**Solution:**
```typescript
// Check provider supports WebSocket
const model = {
  id: "model-id",
  api: "openai-responses",  // Must support WebSocket
  provider: "openai"
};

const agent = new Agent({
  model,
  transport: "websocket"  // Explicitly set
});

// Fallback to HTTP if WebSocket fails
const agent = new Agent({
  model,
  transport: "auto"  // Automatic fallback
});
```

---

## File Structure

### Core Files

```
packages/agent/src/
├── index.ts                    # Main entry point
├── agent.ts                    # Core Agent class
├── agent-loop.ts               # Agent loop implementation
├── proxy.ts                    # Agent proxy utilities
└── types.ts                    # Core type definitions
```

### Architecture

```
Agent Loop
├── Initialize agent
├── Start request
├── Stream response
│   ├── Handle text deltas
│   ├── Handle tool calls
│   │   ├── Execute tool
│   │   └── Return result
│   └── Handle thinking blocks
├── End request
└── Update state
```

---

## Related Files

### Dependencies
- `packages/ai/src/index.ts` - LLM provider integration
- `packages/coding-agent/src/core/agent-session.ts` - Session management uses agent

### Configuration
- `tsconfig.build.json` - TypeScript build configuration
- `vitest.config.ts` - Test configuration

### Documentation
- `README.md` - Package overview and usage

---

## Development Notes

### Creating Custom Tools

**Tool Template:**
```typescript
import type { Tool } from "@mariozechner/pi-agent-core";

const myTool: Tool = {
  name: "tool_name",
  description: "Clear description of what the tool does",
  inputSchema: {
    type: "object",
    properties: {
      parameter: {
        type: "string",
        description: "Clear parameter description"
      }
    },
    required: ["parameter"]
  },
  execute: async (input, context) => {
    // Extract parameters
    const { parameter } = input as { parameter: string };

    // Perform operation
    try {
      const result = await performOperation(parameter);
      return { output: result };
    } catch (error) {
      return { error: error.message };
    }
  }
};
```

### Agent Lifecycle

**Lifecycle States:**
```
idle -> starting -> running -> thinking -> tool_executing -> running -> thinking -> ... -> idle
```

**Event Flow:**
```
1. message_start
2. text (multiple)
3. thinking (optional)
4. tool_call (multiple)
5. tool_result (multiple)
6. text (multiple)
7. message_end
```

### Error Handling

**Error Recovery:**
```typescript
agent.on("error", (event) => {
  console.error("Agent error:", event.error);

  // Check error type
  if (event.error instanceof ToolExecutionError) {
    // Tool execution failed
  } else if (event.error instanceof ProviderError) {
    // LLM provider error
  }
});

// Retry on error
const result = await agent.run(config, {
  maxRetries: 3,
  retryDelay: 1000
});
```

---

## Change Log

### 2026-05-04 08:50:13 CST

**Initial Documentation:**
- Created comprehensive package-level documentation
- Documented core Agent API and configuration
- Added agent loop event flow examples
- Included tool calling interface
- Documented state management and transport abstraction

**Key Features Documented:**
- Core agent framework with streaming support
- Tool calling and execution
- State management and transitions
- Transport layer abstraction (HTTP, WebSocket, SSE)
- Attachment support for files and images

---

*For project-level documentation, see [root CLAUDE.md](../CLAUDE.md)*
