[根目录](../CLAUDE.md) > [packages](../) > **web-ui**

# @mariozechner/pi-web-ui - Web Components for AI Chat Interfaces

> **Package Version:** 0.72.1
> **Last Updated:** 2026-05-04 08:50:13 CST
> **Documentation Status:** Phase 2 Complete - Module Level Detail
**Coverage:** Web components | AI chat UI | Attachment support | Sandbox runtime

---

## Module Responsibilities

The `@mariozechner/pi-web-ui` package provides reusable web components for building AI chat interfaces. It handles:

- **Chat Interface:** Complete chat UI with message list and input
- **Message Rendering:** Support for text, images, tools, artifacts, and custom message types
- **Attachment Support:** File upload and rendering (PDF, DOCX, images, etc.)
- **Sandbox Runtime:** Secure iframe-based execution environment
- **Provider Integration:** UI for API key configuration and provider selection
- **Web Components:** Lit-based reusable components

---

## Entry Points and Startup

### Main Entry Point

**File:** `dist/index.js` (TypeScript source: `src/index.ts`)

**Primary Exports:**
```typescript
// Main chat interface
export { ChatPanel } from "./ChatPanel.js";

// Core components
export { AgentInterface } from "./components/AgentInterface.js";
export { MessageList } from "./components/MessageList.js";
export { MessageEditor } from "./components/MessageEditor.js";
export { Input } from "./components/Input.js";
export { AttachmentTile } from "./components/AttachmentTile.js";

// Message components
export type {
  ArtifactMessage,
  UserMessageWithAttachments
} from "./components/Messages.js";

export {
  AssistantMessage,
  UserMessage,
  ToolMessage,
  AbortedMessage,
  ToolMessageDebugView,
  convertAttachments,
  defaultConvertToLlm,
  isArtifactMessage,
  isUserMessageWithAttachments
} from "./components/Messages.js";

// Message renderer registry
export {
  getMessageRenderer,
  registerMessageRenderer,
  renderMessage,
  type MessageRenderer,
  type MessageRole
} from "./components/message-renderer-registry.js";

// Utility components
export { ConsoleBlock } from "./components/ConsoleBlock.js";
export { ExpandableSection } from "./components/ExpandableSection.js";
export { StreamingMessageContainer } from "./components/StreamingMessageContainer.js";
export { ProviderKeyInput } from "./components/ProviderKeyInput.js";
export { CustomProviderCard } from "./components/CustomProviderCard.js";

// Sandbox components
export {
  SandboxIframe,
  type SandboxResult,
  type SandboxUrlProvider,
  type SandboxFile
} from "./components/SandboxedIframe.js";

// Sandbox runtime providers
export { ArtifactsRuntimeProvider } from "./components/sandbox/ArtifactsRuntimeProvider.js";
export { AttachmentsRuntimeProvider } from "./components/sandbox/AttachmentsRuntimeProvider.js";
export { ConsoleRuntimeProvider } from "./components/sandbox/ConsoleRuntimeProvider.js";
export { FileDownloadRuntimeProvider } from "./components/sandbox/FileDownloadRuntimeProvider.js";
```

---

## External Interfaces

### Chat Panel API

**Creating a Chat Interface:**

```typescript
import { ChatPanel } from "@mariozechner/pi-web-ui";

const chatPanel = new ChatPanel({
  messages: [],
  onSendMessage: async (message) => {
    // Handle message send
    return { response: "Response" };
  },
  onAttachmentUpload: async (files) => {
    // Handle file uploads
    return attachments;
  },
  theme: "dark"
});

document.body.appendChild(chatPanel);
```

**Chat Panel Configuration:**

```typescript
interface ChatPanelOptions {
  messages?: Message[];
  onSendMessage?: (message: string, attachments?: Attachment[]) => Promise<void>;
  onAttachmentUpload?: (files: File[]) => Promise<Attachment[]>;
  onToolCall?: (toolCall: ToolCall) => Promise<void>;
  theme?: "light" | "dark" | Theme;
  streaming?: boolean;
  showThinking?: boolean;
  allowedMessageTypes?: MessageType[];
}
```

### Message Renderer API

**Custom Message Renderers:**

```typescript
import { registerMessageRenderer } from "@mariozechner/pi-web-ui";

// Register custom renderer
registerMessageRenderer("custom", {
  render: (message, context) => {
    const element = document.createElement("div");
    element.className = "custom-message";
    element.textContent = message.content;
    return element;
  },
  canRender: (message) => {
    return message.type === "custom";
  }
});

// Use custom message type
chatPanel.addMessage({
  role: "assistant",
  type: "custom",
  content: "Custom content"
});
```

**Message Renderer Registry:**

```typescript
interface MessageRenderer {
  role: MessageRole;
  render: (message: Message, context: RenderContext) => HTMLElement;
  canRender?: (message: Message) => boolean;
}

type MessageRole = "user" | "assistant" | "system" | "tool" | "custom";

// Get renderer for message type
const renderer = getMessageRenderer("assistant");

// Register additional renderer
registerMessageRenderer("my-type", renderer);
```

### Sandbox Runtime API

**Creating Sandbox Runtime:**

```typescript
import {
  SandboxIframe,
  ArtifactsRuntimeProvider,
  ConsoleRuntimeProvider
} from "@mariozechner/pi-web-ui";

const sandbox = new SandboxIframe({
  runtimeProviders: [
    new ArtifactsRuntimeProvider(),
    new ConsoleRuntimeProvider(),
    new FileDownloadRuntimeProvider()
  ],
  onConsoleMessage: (message) => {
    console.log("Sandbox console:", message);
  },
  onError: (error) => {
    console.error("Sandbox error:", error);
  }
});

document.body.appendChild(sandbox);

// Execute code in sandbox
const result = await sandbox.execute(`
  console.log("Hello from sandbox");
  const div = document.createElement("div");
  div.textContent = "Sandbox content";
  document.body.appendChild(div);
`);
```

**Sandbox URL Provider:**

```typescript
interface SandboxUrlProvider {
  getUrl(file: SandboxFile): Promise<string>;
}

class CustomUrlProvider implements SandboxUrlProvider {
  async getUrl(file: SandboxFile): Promise<string> {
    // Convert file to URL
    return URL.createObjectURL(file.blob);
  }
}

const sandbox = new SandboxIframe({
  runtimeProviders: [
    new ArtifactsRuntimeProvider(new CustomUrlProvider())
  ]
});
```

### Attachment API

**Attachment Types:**

```typescript
interface Attachment {
  id: string;
  name: string;
  type: string;
  size: number;
  url?: string;
  content?: string;
}

// Convert attachment to LLM format
import { convertAttachments } from "@mariozechner/pi-web-ui";

const llmAttachments = convertAttachments(attachments, {
  maxSize: 10 * 1024 * 1024,  // 10MB
  allowedTypes: ["image/*", "application/pdf"]
});
```

---

## Key Dependencies and Configuration

### Runtime Dependencies

**Core Dependencies:**
```json
{
  "@mariozechner/pi-ai": "^0.72.1",
  "@mariozechner/pi-tui": "^0.72.1",
  "typebox": "^1.1.24",
  "lit": "^3.3.1"
}
```

**File Processing:**
```json
{
  "docx-preview": "^0.3.7",
  "jszip": "^3.10.1",
  "pdfjs-dist": "^5.4.394",
  "xlsx": "https://cdn.sheetjs.com/xlsx-0.20.3/xlsx-0.20.3.tgz"
}
```

**Icons:**
```json
{
  "lucide": "^0.544.0"
}
```

**Provider Integration:**
```json
{
  "@lmstudio/sdk": "^1.5.0",
  "ollama": "^0.6.0"
}
```

### Configuration

**TypeScript Configuration:**
```json
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "ESNext",
    "moduleResolution": "bundler",
    "strict": true,
    "experimentalDecorators": true
  }
}
```

**Build Scripts:**
```json
{
  "scripts": {
    "clean": "shx rm -rf dist",
    "build": "tsc -p tsconfig.build.json && tailwindcss -i ./src/app.css -o ./dist/app.css --minify",
    "dev": "concurrently --names \"build,example\" \"tsc -p tsconfig.build.json --watch\" \"tailwindcss -i ./src/app.css -o ./dist/app.css --watch\" \"npm run dev --prefix example\"",
    "check": "biome check --write --error-on-warnings . && tsc --noEmit"
  }
}
```

---

## Data Models

### Message Types

**Core Message Structure:**
```typescript
type Message =
  | UserMessage
  | AssistantMessage
  | SystemMessage
  | ToolMessage
  | ArtifactMessage
  | CustomMessage;

interface UserMessage {
  role: "user";
  content: string;
  attachments?: Attachment[];
}

interface AssistantMessage {
  role: "assistant";
  content: string;
  toolCalls?: ToolCall[];
  thinking?: string;
  usage?: TokenUsage;
}

interface ToolMessage {
  role: "tool";
  toolCallId: string;
  content: string;
  error?: boolean;
}

interface ArtifactMessage {
  role: "assistant";
  type: "artifact";
  content: string;
  language?: string;
  metadata?: Record<string, unknown>;
}
```

### Attachment Types

**Attachment Structure:**
```typescript
interface Attachment {
  id: string;
  name: string;
  type: string;
  size: number;
  url?: string;
  content?: string;
  thumbnail?: string;
  metadata?: Record<string, unknown>;
}

interface AttachmentTileOptions {
  attachment: Attachment;
  onRemove?: () => void;
  onPreview?: () => void;
  showSize?: boolean;
  showType?: boolean;
}
```

### Sandbox Types

**Sandbox Configuration:**
```typescript
interface SandboxIframeOptions {
  runtimeProviders?: RuntimeProvider[];
  onConsoleMessage?: (message: ConsoleMessage) => void;
  onError?: (error: Error) => void;
  sandboxAttributes?: {
    allowScripts?: boolean;
    allowForms?: boolean;
    allowSameOrigin?: boolean;
    allowModals?: boolean;
  };
}

interface ConsoleMessage {
  type: "log" | "error" | "warn" | "info";
  args: unknown[];
  timestamp?: number;
}

interface SandboxFile {
  name: string;
  blob: Blob;
  type: string;
}
```

---

## Testing and Quality

### Test Organization

**Test Structure:**
```
packages/web-ui/test/
├── components/
│   ├── MessageList.test.ts
│   ├── MessageEditor.test.ts
│   └── AttachmentTile.test.ts
├── sandbox/
│   ├── SandboxIframe.test.ts
│   └── RuntimeProviders.test.ts
└── utils/
    ├── message-renderer.test.ts
    └── attachment-converter.test.ts
```

### Quality Tools

**Type Checking:**
```bash
# From repo root
npm run check

# From package root
cd packages/web-ui
npx tsc --noEmit
```

**Linting:**
```bash
# Biome for linting and formatting
biome check --write --error-on-warnings .
```

---

## Common Issues and Solutions

### Rendering Issues

**Issue:** Custom message renderer not working

**Solution:**
```typescript
// Ensure renderer is registered before use
registerMessageRenderer("custom", {
  role: "assistant",
  render: (message, context) => {
    // Return HTMLElement
  },
  canRender: (message) => {
    return message.type === "custom";
  }
});

// Check message type matches
const message = {
  role: "assistant",
  type: "custom",  // Must match canRender check
  content: "..."
};
```

**Issue:** Message list not updating

**Solution:**
```typescript
// Use reactive properties
const messageList = document.querySelector("message-list");
messageList.messages = [...messageList.messages, newMessage];

// Or use addMessage method
messageList.addMessage(newMessage);

// Ensure Lit is properly configured
import { LitElement } from "lit";
class MyComponent extends LitElement {
  static properties = {
    messages: { type: Array }
  };
}
```

### Attachment Issues

**Issue:** File upload not working

**Solution:**
```typescript
// Check file type is supported
const allowedTypes = ["image/*", "application/pdf", "text/*"];
const file = files[0];

if (!allowedTypes.some(type => mimematch(type, file.type))) {
  throw new Error("File type not supported");
}

// Check file size
const maxSize = 10 * 1024 * 1024;  // 10MB
if (file.size > maxSize) {
  throw new Error("File too large");
}

// Convert to LLM format
const attachment = {
  id: generateId(),
  name: file.name,
  type: file.type,
  size: file.size,
  content: await fileToBase64(file)
};
```

### Sandbox Issues

**Issue:** Sandbox iframe not loading

**Solution:**
```typescript
// Check CSP headers
const sandbox = new SandboxIframe({
  sandboxAttributes: {
    allowScripts: true,
    allowSameOrigin: true,
    allowModals: true
  }
});

// Check runtime providers are registered
const sandbox = new SandboxIframe({
  runtimeProviders: [
    new ArtifactsRuntimeProvider(),
    new ConsoleRuntimeProvider()
  ]
});

// Check for console errors
sandbox.onError((error) => {
  console.error("Sandbox error:", error);
});
```

---

## File Structure

### Core Files

```
packages/web-ui/src/
├── index.ts                    # Main entry point
├── ChatPanel.ts                # Main chat interface
├── app.css                     # Global styles
├── components/
│   ├── AgentInterface.ts       # Agent interface component
│   ├── MessageList.ts          # Message list component
│   ├── MessageEditor.ts        # Message editor component
│   ├── Input.ts                # Input component
│   ├── AttachmentTile.ts       # Attachment display
│   ├── Messages.ts             # Message type definitions
│   ├── message-renderer-registry.ts  # Message renderer registry
│   ├── ConsoleBlock.ts         # Console output display
│   ├── ExpandableSection.ts    # Expandable/collapsible section
│   ├── StreamingMessageContainer.ts  # Streaming message wrapper
│   ├── ProviderKeyInput.ts     # Provider key input
│   ├── CustomProviderCard.ts   # Custom provider card
│   ├── SandboxedIframe.ts      # Sandbox iframe
│   └── sandbox/
│       ├── ArtifactsRuntimeProvider.ts  # Artifacts runtime
│       ├── AttachmentsRuntimeProvider.ts # Attachments runtime
│       ├── ConsoleRuntimeProvider.ts     # Console runtime
│       └── FileDownloadRuntimeProvider.ts # File download runtime
└── utils/
    ├── message-converter.ts    # Message format conversion
    ├── attachment-converter.ts # Attachment conversion
    └── file-reader.ts          # File reading utilities
```

### Example Application

```
packages/web-ui/example/
├── index.html                  # Example HTML
├── main.ts                     # Example TypeScript
└── styles.css                  # Example styles
```

---

## Related Files

### Dependencies
- `packages/ai/src/index.ts` - LLM provider integration
- `packages/tui/src/index.ts` - Shared utility functions
- `packages/agent/src/index.ts` - Agent integration

### Configuration
- `tsconfig.build.json` - TypeScript build configuration
- `tailwind.config.js` - Tailwind CSS configuration

### Documentation
- `README.md` - Package overview and usage

---

## Development Notes

### Creating Custom Components

**Component Template:**
```typescript
import { LitElement, html, css } from "lit";
import { customElement, property } from "lit/decorators.js";

@customElement("my-component")
export class MyComponent extends LitElement {
  static styles = css`
    :host {
      display: block;
    }
  `;

  @property({ type: String })
  message = "";

  render() {
    return html`
      <div class="my-component">
        ${this.message}
      </div>
    `;
  }
}
```

### Creating Custom Message Renderers

**Renderer Template:**
```typescript
import {
  registerMessageRenderer,
  type MessageRenderer
} from "@mariozechner/pi-web-ui";

const renderer: MessageRenderer = {
  role: "assistant",
  render: (message, context) => {
    const container = document.createElement("div");
    container.className = "custom-message";

    const content = document.createElement("div");
    content.className = "message-content";
    content.textContent = message.content;

    container.appendChild(content);
    return container;
  },
  canRender: (message) => {
    return message.type === "custom";
  }
};

registerMessageRenderer("custom", renderer);
```

### Creating Runtime Providers

**Provider Template:**
```typescript
import { RuntimeProvider } from "@mariozechner/pi-web-ui";

class CustomRuntimeProvider implements RuntimeProvider {
  async init(iframe: HTMLIFrameElement): Promise<void> {
    // Initialize runtime
  }

  async execute(code: string): Promise<SandboxResult> {
    // Execute code
    return { success: true, result: "..." };
  }

  async cleanup(): Promise<void> {
    // Cleanup resources
  }
}
```

---

## Change Log

### 2026-05-04 08:50:13 CST

**Initial Documentation:**
- Created comprehensive package-level documentation
- Documented all web components and their APIs
- Added chat panel interface examples
- Included message renderer registry documentation
- Documented sandbox runtime system

**Key Features Documented:**
- Complete chat interface with message list and input
- Message rendering system with custom renderer support
- Attachment support for various file types
- Sandbox runtime with secure iframe execution
- Provider integration UI components
- Web Components based on Lit framework

---

*For project-level documentation, see [root CLAUDE.md](../CLAUDE.md)*
