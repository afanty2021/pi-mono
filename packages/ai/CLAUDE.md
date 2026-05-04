[根目录](../CLAUDE.md) > [packages](../) > **ai**

# @mariozechner/pi-ai - Unified Multi-Provider LLM API

> **Package Version:** 0.72.1
> **Last Updated:** 2026-05-04 08:50:13 CST
> **Documentation Status:** Phase 2 Complete - Module Level Detail
> **Coverage:** Core LLM abstraction | 35+ providers | TypeScript strict mode

---

## Module Responsibilities

The `@mariozechner/pi-ai` package provides a unified, multi-provider LLM API that abstracts away the differences between various AI providers (Anthropic, OpenAI, Google, Amazon Bedrock, Mistral, etc.). It handles:

- **Provider Abstraction:** Unified interface for 35+ LLM providers
- **Model Discovery:** Automatic model discovery and capability detection
- **Streaming Support:** First-class streaming with standardized events
- **Type Safety:** Full TypeScript support with strict type checking
- **OAuth Integration:** Built-in OAuth support for subscription-based providers
- **Environment Detection:** Automatic API key detection from environment variables

---

## Entry Points and Startup

### Main Entry Point

**File:** `dist/index.js` (TypeScript source: `src/index.ts`)

**Primary Exports:**
```typescript
// Core types and utilities
export { Type } from "typebox";
export type { Static, TSchema } from "typebox";

// API registry and model management
export * from "./api-registry.js";        // registerApiProvider(), clearApiProviders()
export * from "./models.js";              // Model registry, getModels(), findModels()
export * from "./env-api-keys.js";        // getEnvApiKey()

// Provider implementations
export * from "./providers/register-builtins.js";  // Lazy provider registration
export type { AnthropicOptions, AnthropicEffort, AnthropicThinkingDisplay } from "./providers/anthropic.js";
export type { OpenAIResponsesOptions, OpenAICompletionsOptions } from "./providers/openai-*.js";
export type { GoogleOptions, GoogleThinkingLevel } from "./providers/google.js";
export type { BedrockOptions, BedrockThinkingDisplay } from "./providers/amazon-bedrock.js";
// ... (all provider option types)

// Streaming and events
export * from "./stream.js";              // stream(), streamSimple()
export * from "./types.js";               // Core type definitions
export * from "./utils/event-stream.js";  // AssistantMessageEventStream
export * from "./utils/json-parse.js";    // JSON parsing utilities

// OAuth support
export type {
  OAuthProvider, OAuthCredentials, OAuthAuthInfo,
  OAuthProviderInterface, OAuthLoginCallbacks
} from "./utils/oauth/types.js";

// Validation and utilities
export * from "./utils/validation.js";    // validate(), validateTypes()
export * from "./utils/overflow.js";      // isContextOverflow()
export * from "./utils/typebox-helpers.js"; // TypeBox utilities
```

### CLI Entry Point

**File:** `dist/cli.js` (TypeScript source: `src/cli.ts`)

**Binary:** `pi-ai` (installed via npm)

**Usage:**
```bash
pi-ai --help                    # Show CLI help
pi-ai --list-models             # List all available models
pi-ai --list-models anthropic   # List models for specific provider
pi-ai --providers               # List all registered providers
```

---

## External Interfaces

### Core Streaming API

**Primary Interface:** `stream()` and `streamSimple()`

```typescript
import { stream, type Model, type Context, type AssistantMessageEvent } from "@mariozechner/pi-ai";

// Full streaming with provider-specific options
async function* streamAnthropic(
  model: Model<"anthropic-messages">,
  context: Context,
  options?: AnthropicOptions
): AsyncIterable<AssistantMessageEvent>;

// Simplified streaming with common options
async function* streamSimple(
  model: Model<any>,
  context: Context,
  options?: SimpleStreamOptions
): AsyncIterable<AssistantMessageEvent>;
```

**Standardized Events:**
```typescript
type AssistantMessageEvent =
  | { type: "text"; delta: string }
  | { type: "tool_call"; toolCall: ToolCall }
  | { type: "thinking"; text: string }
  | { type: "usage"; usage: Usage }
  | { type: "stop"; reason: string }
  | { type: "error"; error: Error };
```

### Provider Registration API

**Interface:** `registerApiProvider()`

```typescript
import { registerApiProvider } from "@mariozechner/pi-ai";

registerApiProvider({
  api: "custom-api",
  stream: async (model, context, options) => {
    // Return AsyncIterable<AssistantMessageEvent>
  },
  streamSimple: async (model, context, options) => {
    // Simplified version
  },
  optionsSchema: {
    // TypeBox schema for options validation
  }
});
```

### Model Discovery API

**Interface:** `getModels()`, `findModels()`

```typescript
import { getModels, findModels } from "@mariozechner/pi-ai";

// Get all models for a provider
const anthropicModels = getModels("anthropic");

// Search models by pattern
const claudeModels = findModels("claude-*");

// Find specific model
const model = findModels("claude-sonnet-4-20250514")[0];
```

### OAuth Provider Interface

**Interface:** `OAuthProviderInterface`

```typescript
interface OAuthProviderInterface {
  providerId: string;
  name: string;
  login(credentials: OAuthCredentials): Promise<OAuthAuthInfo>;
  refresh(credentials: OAuthCredentials): Promise<OAuthAuthInfo>;
  validate(token: string): Promise<boolean>;
}
```

---

## Key Dependencies and Configuration

### Runtime Dependencies

**LLM Provider SDKs:**
```json
{
  "@anthropic-ai/sdk": "^0.91.1",
  "openai": "6.26.0",
  "@google/genai": "^1.40.0",
  "@mistralai/mistralai": "^2.2.0",
  "@aws-sdk/client-bedrock-runtime": "^3.1030.0"
}
```

**Utilities:**
```json
{
  "typebox": "^1.1.24",
  "zod-to-json-schema": "^3.24.6",
  "chalk": "^5.6.2",
  "undici": "^7.19.1",
  "proxy-agent": "^6.5.0",
  "partial-json": "^0.1.7"
}
```

### Configuration Files

**`package.json` Exports:**
```json
{
  "exports": {
    ".": "./dist/index.js",
    "./anthropic": "./dist/providers/anthropic.js",
    "./google": "./dist/providers/google.js",
    "./openai-responses": "./dist/providers/openai-responses.js",
    "./bedrock-provider": "./dist/bedrock-provider.js",
    "./oauth": "./dist/oauth.js"
  }
}
```

**Build Scripts:**
```json
{
  "scripts": {
    "clean": "shx rm -rf dist",
    "generate-models": "npx tsx scripts/generate-models.ts",
    "build": "npm run generate-models && tsgo -p tsconfig.build.json",
    "dev": "tsgo -p tsconfig.build.json --watch --preserveWatchOutput",
    "test": "vitest --run"
  }
}
```

### Environment Variables

**API Key Detection:**
```bash
# Provider-specific keys (checked in order)
ANTHROPIC_API_KEY              # Anthropic
OPENAI_API_KEY                 # OpenAI
GEMINI_API_KEY                 # Google
GROQ_API_KEY                   # Groq
CEREBRAS_API_KEY               # Cerebras
XAI_API_KEY                    # xAI
MISTRAL_API_KEY                # Mistral
BEDROCK_EXTENSIVE_MODEL_TEST   # Enable Bedrock tests
AWS_*                          # AWS credentials for Bedrock
```

**Model Configuration:**
```bash
# Custom models location
PI_MODELS_JSON=/path/to/models.json

# Offline mode (disable network operations)
PI_OFFLINE=1
```

---

## Data Models

### Core Type System

**Provider API Types:**
```typescript
// All supported provider APIs
type Api =
  | "anthropic-messages"
  | "openai-responses"
  | "openai-completions"
  | "openai-codex-responses"
  | "google-generative-ai"
  | "google-vertex"
  | "amazon-bedrock"
  | "mistral"
  | "azure-openai-responses"
  | "custom-api";  // User-defined

// Provider-specific options mapping
type ApiOptionsMap = {
  "anthropic-messages": AnthropicOptions;
  "openai-responses": OpenAIResponsesOptions;
  "google-generative-ai": GoogleOptions;
  // ... (all providers)
};
```

**Model Definition:**
```typescript
interface Model<TApi extends Api> {
  id: string;
  api: TApi;
  provider: KnownProvider | string;
  thinking: boolean;
  maxTokens: number;
  supportsImages: boolean;
  supportsTools: boolean;
  supportsStreaming: boolean;
  supportedThinkingLevels?: ThinkingLevel[];
}
```

**Context and Messages:**
```typescript
interface Context {
  messages: Message[];
  tools?: Tool[];
  system?: string;
  temperature?: number;
  maxTokens?: number;
  topP?: number;
}

type Message =
  | { role: "user"; content: Content[] }
  | { role: "assistant"; content: Content[]; toolCalls?: ToolCall[] }
  | { role: "tool"; toolCallId: string; content: string };

type Content =
  | { type: "text"; text: string }
  | { type: "image"; image: ImageContent };
```

### Provider Configuration

**Model Registry:**
```typescript
// Generated from scripts/generate-models.ts
// NEVER modify packages/ai/src/models.generated.ts directly

interface ModelRegistry {
  anthropic: ModelDefinition[];
  openai: ModelDefinition[];
  google: ModelDefinition[];
  bedrock: ModelDefinition[];
  mistral: ModelDefinition[];
  // ... (all providers)
}
```

**OAuth Credentials:**
```typescript
interface OAuthCredentials {
  providerId: string;
  accessToken?: string;
  refreshToken?: string;
  expiresAt?: number;
  userId?: string;
}
```

---

## Testing and Quality

### Test Organization

**Test Structure:**
```
packages/ai/test/
├── providers/
│   ├── anthropic.test.ts       # Anthropic provider tests
│   ├── openai-responses.test.ts # OpenAI provider tests
│   ├── google.test.ts          # Google provider tests
│   ├── bedrock.test.ts         # Bedrock provider tests
│   └── ...
├── models.test.ts              # Model discovery tests
├── stream.test.ts              # Streaming API tests
└── utils/
    ├── json-parse.test.ts      # JSON parsing utilities
    └── validation.test.ts      # TypeBox validation tests
```

### Test Guidelines

**From AGENTS.md:**
- Use mock provider tests - NO real API calls
- Use test harness from `packages/coding-agent/test/suite/harness.ts`
- Do NOT use real API keys or paid tokens in tests
- Run tests from package root, not repo root
- Name regression tests: `<issue-number>-<short-slug>.test.ts`

**Test Command:**
```bash
cd packages/ai
npx tsx ../../node_modules/vitest/dist/cli.js --run test/providers/anthropic.test.ts
```

### Quality Tools

**Linting and Formatting:**
```bash
# Biome for linting and formatting
biome check --write --error-on-warnings .

# Type checking
tsc --noEmit
```

**Pre-commit Hooks:**
```bash
# Husky runs before commit
npm run check  # Must pass before committing
```

---

## Common Issues and Solutions

### Provider Registration

**Issue:** Custom provider not showing up in model list

**Solution:**
1. Check provider is registered in `register-builtins.ts`
2. Verify lazy loader is not importing provider module directly
3. Add models to `scripts/generate-models.ts`
4. Run `npm run generate-models` to regenerate `models.generated.ts`

### Type Errors

**Issue:** Type mismatches between provider options

**Solution:**
```typescript
// Import specific option types
import type { AnthropicOptions } from "@mariozechner/pi-ai/anthropic";

// Use proper generic types
const model: Model<"anthropic-messages"> = {
  id: "claude-sonnet-4-20250514",
  api: "anthropic-messages",
  // ...
};
```

### API Key Detection

**Issue:** Environment variable not detected

**Solution:**
```bash
# Check environment variable is set
echo $ANTHROPIC_API_KEY

# Verify detection order in env-api-keys.ts
# Order: specific key -> generic key -> OAuth

# Test API key manually
pi-ai --list-models anthropic
```

### Streaming Issues

**Issue:** Streaming events not received in order

**Solution:**
```typescript
// Use AssistantMessageEventStream for proper ordering
import { AssistantMessageEventStream } from "@mariozechner/pi-ai";

const stream = AssistantMessageEventStream.from(events);
for await (const event of stream) {
  // Events are properly ordered and buffered
}
```

---

## File Structure

### Core Files

```
packages/ai/src/
├── index.ts                    # Main entry point
├── api-registry.ts             # Provider registration system
├── models.ts                   # Model discovery API
├── env-api-keys.ts             # Environment variable detection
├── stream.ts                   # Unified streaming API
├── types.ts                    # Core type definitions
├── cli.ts                      # CLI entry point
├── providers/
│   ├── register-builtins.ts    # Lazy provider registration
│   ├── anthropropic.ts         # Anthropic provider
│   ├── openai-responses.ts     # OpenAI provider
│   ├── google.ts               # Google provider
│   ├── amazon-bedrock.ts       # Bedrock provider
│   ├── mistral.ts              # Mistral provider
│   └── ... (other providers)
├── utils/
│   ├── event-stream.ts         # Event stream utilities
│   ├── json-parse.ts           # JSON parsing
│   ├── oauth/                  # OAuth implementation
│   ├── overflow.ts             # Context overflow detection
│   ├── validation.ts           # TypeBox validation
│   └── typebox-helpers.ts      # TypeBox utilities
└── scripts/
    └── generate-models.ts      # Model generation script
```

### Generated Files

```
packages/ai/src/
└── models.generated.ts         # Auto-generated model definitions
                                 # NEVER edit directly - use generate-models.ts
```

---

## Related Files

### Dependencies
- `packages/agent/src/index.ts` - Agent framework uses pi-ai for LLM calls
- `packages/coding-agent/src/` - Main CLI uses pi-ai for all provider interactions
- `packages/web-ui/src/` - Web UI uses pi-ai for chat functionality

### Configuration
- `tsconfig.build.json` - TypeScript build configuration
- `tsconfig.json` - TypeScript base configuration
- `vitest.config.ts` - Test configuration

### Documentation
- `README.md` - Package overview and usage
- `CHANGELOG.md` - Version history and changes

---

## Development Notes

### Adding a New Provider

**Files to Modify:**
1. `src/types.ts` - Add API identifier to `Api` union
2. `src/providers/<provider>.ts` - Create provider implementation
3. `src/providers/register-builtins.ts` - Register provider
4. `src/env-api-keys.ts` - Add credential detection
5. `scripts/generate-models.ts` - Add model definitions
6. `src/index.ts` - Export provider option types
7. `package.json` - Add subpath export
8. `test/providers/<provider>.test.ts` - Add tests

**Provider Implementation Template:**
```typescript
import type {
  Model,
  Context,
  StreamOptions,
  AssistantMessageEvent
} from "../types.js";

export interface CustomProviderOptions extends StreamOptions {
  // Provider-specific options
  customParam?: string;
}

export async function streamCustomProvider(
  model: Model<"custom-api">,
  context: Context,
  options?: CustomProviderOptions
): AsyncIterable<AssistantMessageEvent> {
  // Implementation
  // Must emit: text, tool_call, thinking, usage, stop, error events
}

export async function streamSimpleCustomProvider(
  model: Model<"custom-api">,
  context: Context,
  options?: SimpleStreamOptions
): AsyncIterable<AssistantMessageEvent> {
  // Simplified implementation
}
```

### Model Generation

**Generate Models Script:**
```bash
# Regenerate models from generate-models.ts
npm run generate-models

# This updates models.generated.ts
# DO NOT edit models.generated.ts directly
```

---

## Change Log

### 2026-05-04 08:50:13 CST

**Initial Documentation:**
- Created comprehensive package-level documentation
- Documented all 35+ provider interfaces
- Added streaming API examples
- Included testing guidelines and quality standards
- Documented file structure and development workflows

**Key Features Documented:**
- Unified multi-provider LLM API
- Automatic model discovery
- OAuth integration
- Environment variable detection
- Type-safe streaming interface

---

*For project-level documentation, see [root CLAUDE.md](../CLAUDE.md)*
